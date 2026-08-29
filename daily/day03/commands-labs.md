# Day 3 Commands & Labs

## Lab 1 — Create and inspect a ConfigMap

```bash
kubectl create configmap web-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

kubectl get cm
kubectl describe configmap web-config
kubectl get configmap web-config -o yaml
```

Observed data:

```yaml
data:
  APP_ENV: production
  LOG_LEVEL: info
```

## Lab 2 — Inject one ConfigMap key

`config-demo.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: web-config
          key: APP_ENV
```

Run:

```bash
kubectl apply -f config-demo.yaml
kubectl get pod config-demo
kubectl exec config-demo -- printenv APP_ENV
```

Observed:

```text
production
```

## Lab 3 — Inject the entire ConfigMap

`config-demo-all.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo-all
spec:
  containers:
  - name: nginx
    image: nginx
    envFrom:
    - configMapRef:
        name: web-config
```

Run:

```bash
kubectl apply -f config-demo-all.yaml
kubectl get pod config-demo-all
kubectl exec config-demo-all -- printenv APP_ENV
kubectl exec config-demo-all -- printenv LOG_LEVEL
```

Observed:

```text
production
info
```

## Lab 4 — Mount ConfigMap as files

Create ConfigMap:

```bash
kubectl create configmap app-file-config \
  --from-literal=config.txt="hello from configmap"
```

`config-volume-demo.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-file-config
```

Run:

```bash
kubectl apply -f config-volume-demo.yaml
kubectl get pod config-volume-demo
kubectl exec config-volume-demo -- ls -l /etc/config
kubectl exec config-volume-demo -- cat /etc/config/config.txt
```

Observed:

```text
config.txt -> ..data/config.txt
hello from configmap
```

## Lab 5 — Compare ConfigMap update behavior

Update the environment-variable ConfigMap:

```bash
kubectl edit configmap web-config
```

Change:

```text
APP_ENV=production
```

to:

```text
APP_ENV=staging
```

Check the already-running Pod:

```bash
kubectl exec config-demo -- printenv APP_ENV
```

Observed:

```text
production
```

The existing environment variable did not refresh.

Update the file-backed ConfigMap:

```bash
kubectl edit configmap app-file-config
```

Change file content to:

```text
hello from updated configmap
```

Then:

```bash
kubectl exec config-volume-demo -- cat /etc/config/config.txt
```

Observed after Kubernetes refreshed the projection:

```text
hello from updated configmap
```

## Lab 6 — Create a Secret

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=mysecret123

kubectl get secret db-secret -o yaml
```

Observed:

```yaml
data:
  DB_PASSWORD: bXlzZWNyZXQxMjM=
  DB_USER: YWRtaW4=
type: Opaque
```

Decode manually for learning:

```bash
echo 'YWRtaW4=' | base64 -d
echo 'bXlzZWNyZXQxMjM=' | base64 -d
```

## Lab 7 — Inject one Secret key

`secret-demo.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_PASSWORD
```

Run:

```bash
kubectl apply -f secret-demo.yaml
kubectl get pod secret-demo
kubectl exec secret-demo -- printenv DATABASE_PASSWORD
```

Observed:

```text
mysecret123
```

Kubernetes decoded the stored Base64 value before exposing it as the environment variable.

## Secret patterns to remember

Whole Secret as environment variables:

```yaml
envFrom:
- secretRef:
    name: db-secret
```

Secret Volume:

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /etc/secret

volumes:
- name: secret-volume
  secret:
    secretName: db-secret
```

Then a Secret key `DB_PASSWORD` appears as:

```text
/etc/secret/DB_PASSWORD
```

## Useful commands

```bash
kubectl get cm
kubectl get secret
kubectl get cm <name> -o yaml
kubectl get secret <name> -o yaml
kubectl describe pod <pod>
kubectl exec <pod> -- printenv <VAR>
kubectl exec <pod> -- cat <path>
kubectl edit configmap <name>
```
