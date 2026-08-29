# Day 3 Knowledge — ConfigMap and Secret

## 1. Why ConfigMap exists

Hardcoding configuration in a Pod or Deployment template couples workload definition and application configuration.

A Deployment-managed Pod normally does **not** lose hardcoded configuration merely because the Pod is recreated: the Deployment template recreates the Pod with the same values. The real benefit of ConfigMap is **decoupling** configuration from the workload definition.

Typical ConfigMap data:

```text
APP_ENV=production
LOG_LEVEL=info
API_URL=https://api.example.com
```

## 2. ConfigMap is namespaced

A ConfigMap and a Pod that references it must be in the same namespace.

If YAML omits `metadata.namespace`, kubectl uses the namespace selected by the current context or the namespace provided by `-n`; in the usual default context this is `default`.

A reference such as:

```yaml
configMapKeyRef:
  name: web-config
```

is effectively resolved as:

```text
<Pod namespace>/web-config
```

`default/web-config` and `test/web-config` are different objects.

## 3. Inject one ConfigMap key as an environment variable

```yaml
env:
- name: APP_ENV
  valueFrom:
    configMapKeyRef:
      name: web-config
      key: APP_ENV
```

Important distinction:

- `env[].name` = name of the environment variable inside the container
- `configMapKeyRef.name` = name of the ConfigMap object
- `configMapKeyRef.key` = key to read from that ConfigMap

The environment variable name does not have to match the ConfigMap key.

Example:

```yaml
env:
- name: MY_ENVIRONMENT
  valueFrom:
    configMapKeyRef:
      name: web-config
      key: APP_ENV
```

With `APP_ENV: production`, the container receives:

```text
MY_ENVIRONMENT=production
```

## 4. Inject an entire ConfigMap

```yaml
envFrom:
- configMapRef:
    name: web-config
```

This injects all valid keys as environment variables.

Useful comparison:

```text
env + configMapKeyRef
  -> select one key

envFrom + configMapRef
  -> import the whole ConfigMap
```

If a Pod should receive only `APP_ENV` and not `LOG_LEVEL`, prefer `configMapKeyRef`.

## 5. Mount ConfigMap as files

Example ConfigMap data:

```yaml
data:
  config.txt: hello from configmap
```

Pod:

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config

volumes:
- name: config-volume
  configMap:
    name: app-file-config
```

The mapping is:

```text
ConfigMap key   -> filename
ConfigMap value -> file content
```

So:

```text
config.txt: hello from configmap
```

becomes:

```text
/etc/config/config.txt
```

with file content:

```text
hello from configmap
```

### volumes vs volumeMounts

`volumes` defines the storage source at Pod level:

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-file-config
```

`volumeMounts` determines where a particular container sees that Volume:

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

The `name` must match.

## 6. Environment variable vs Volume update behavior

This was verified in the lab.

After changing a ConfigMap value:

```text
Environment variable injection
  -> an already-running Pod keeps the old environment variable value
  -> recreate/restart the Pod to consume the new value

ConfigMap Volume
  -> the mounted file is normally refreshed by Kubernetes after a short delay
  -> the application itself may still need to reload/re-read the file
```

Observed lab result:

```text
ConfigMap environment variable: production
ConfigMap Volume file: hello from updated configmap
```

The mounted ConfigMap file appeared as a symbolic link such as:

```text
config.txt -> ..data/config.txt
```

This is related to Kubernetes' managed projection/update mechanism.

## 7. Secret

Secret is intended for sensitive data such as:

- passwords
- tokens
- API keys
- certificates

Generic Secret example:

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=mysecret123
```

The common generic type is:

```yaml
type: Opaque
```

## 8. Base64 is encoding, not encryption

A Secret may show:

```yaml
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: bXlzZWNyZXQxMjM=
```

These values are Base64-encoded.

```bash
echo 'YWRtaW4=' | base64 -d
# admin
```

Base64:

- changes representation
- is trivially reversible
- requires no secret key

Therefore:

```text
Base64 != encryption
```

Secret security additionally depends on controls such as RBAC, API access and etcd encryption-at-rest configuration.

## 9. Inject one Secret key

```yaml
env:
- name: DATABASE_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: DB_PASSWORD
```

Although the Secret YAML contains Base64 data, Kubernetes supplies the decoded value to the container.

Verified result:

```bash
kubectl exec secret-demo -- printenv DATABASE_PASSWORD
# mysecret123
```

## 10. Whole Secret and Secret Volume

Whole Secret as environment variables:

```yaml
envFrom:
- secretRef:
    name: db-secret
```

Secret as files:

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /etc/secret

volumes:
- name: secret-volume
  secret:
    secretName: db-secret
```

A key named `DB_PASSWORD` becomes:

```text
/etc/secret/DB_PASSWORD
```

and the file contains the decoded Secret value.

## 11. Key reference summary

```text
configMapKeyRef -> one ConfigMap key
configMapRef    -> whole ConfigMap

secretKeyRef    -> one Secret key
secretRef       -> whole Secret

env/envFrom     -> environment variables
Volume          -> files
```

## 12. YAML `|` literal block scalar

Example:

```yaml
data:
  nginx.conf: |
    server {
      listen 80;
    }
```

`|` tells YAML to treat the indented block as a multiline literal string while preserving line breaks.

This is useful for configuration files, scripts and certificate-style content.

By contrast, YAML `>` is a folded block scalar and normally folds line breaks into spaces.
