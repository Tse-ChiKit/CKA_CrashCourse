# CKA Day 2 — Commands and Lab Record

## Pod inspection

```bash
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods -o wide
kubectl describe pod nginx
kubectl get pod nginx -o yaml
```

Important fields inspected:

- Node
- Status
- Pod IP
- Container ID / runtime
- Conditions
- Events

Bare Pod self-healing test:

```bash
kubectl delete pod nginx
kubectl get pods
```

Expected result: the bare Pod does not automatically return.

## Deployment and ReplicaSet

```bash
kubectl create deployment nginx-deploy --image=nginx
kubectl get deployment
kubectl get rs
kubectl get pods -o wide
```

Scale to 3 replicas:

```bash
kubectl scale deployment nginx-deploy --replicas=3
kubectl get deployment
kubectl get rs
kubectl get pods -o wide
```

Self-healing test:

```bash
kubectl delete pod <pod-name>
kubectl get pods -o wide
```

Observe a new Pod object with a different final name suffix.

## Rolling update and rollback

Update image:

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.27
kubectl get deployment
kubectl get rs
kubectl get pods -o wide
kubectl rollout status deployment/nginx-deploy
```

Inspect history:

```bash
kubectl rollout history deployment/nginx-deploy
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deploy
kubectl rollout status deployment/nginx-deploy
kubectl get rs
kubectl get pods -o wide
```

Inspect ReplicaSet revisions:

```bash
kubectl get rs -l app=web-demo \
  -o custom-columns='NAME:.metadata.name,REVISION:.metadata.annotations.deployment\.kubernetes\.io/revision,DESIRED:.spec.replicas,AGE:.metadata.creationTimestamp'
```

## Service — ClusterIP

Create Service:

```bash
kubectl expose deployment nginx-deploy \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP
```

Inspect:

```bash
kubectl get svc
kubectl describe svc nginx-service
kubectl get endpoints nginx-service
kubectl get endpointslice
```

Observed lab Service:

```text
nginx-service
ClusterIP: 10.109.112.172
Selector: app=nginx-deploy
Port: 80
TargetPort: 80
```

Observed backend endpoints:

```text
10.244.118.79:80
10.244.36.203:80
10.244.118.80:80
```

## Service — NodePort

Create NodePort Service:

```bash
kubectl expose deployment nginx-deploy \
  --name=nginx-nodeport \
  --port=80 \
  --target-port=80 \
  --type=NodePort
```

Inspect:

```bash
kubectl get svc nginx-nodeport
kubectl describe svc nginx-nodeport
kubectl get nodes -o wide
```

Observed lab NodePort:

```text
ClusterIP: 10.104.191.0
Service port: 80
NodePort: 32209
ExternalTrafficPolicy: Cluster
```

Node IPs:

```text
k8s-control     192.168.32.131
k8s-worker-01   192.168.32.129
k8s-worker-02   192.168.32.130
```

Conceptual external access:

```text
http://192.168.32.129:32209
http://192.168.32.130:32209
```

## Labels and selectors

```bash
kubectl get pods --show-labels
kubectl get pods -l app=nginx-deploy
kubectl get pods -l app=web-demo -o wide
```

Observed nginx Pod labels:

```text
app=nginx-deploy
pod-template-hash=<hash>
```

## Generate YAML quickly

```bash
kubectl create deployment web-demo \
  --image=nginx \
  --replicas=2 \
  --dry-run=client \
  -o yaml > web-demo.yaml

cat web-demo.yaml
```

Then edit the manifest and apply:

```bash
kubectl apply -f web-demo.yaml
kubectl get deployment web-demo
kubectl get rs
kubectl get pods -l app=web-demo -o wide
kubectl describe deployment web-demo
```

The lab later scaled `web-demo` to 5 replicas and changed its Pod template image, showing that changing replicas scales an existing ReplicaSet while changing the template creates a new ReplicaSet and starts a rollout.

## Useful rollout troubleshooting

```bash
kubectl rollout status deployment/web-demo
kubectl rollout history deployment/web-demo
kubectl get rs
kubectl get pods -l app=web-demo
kubectl describe deployment web-demo
```
