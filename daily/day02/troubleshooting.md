# CKA Day 2 — Troubleshooting Notes

## 1. Pod is Pending or not starting

Start with:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

Check:

- `PodScheduled`
- Node assignment
- Container state
- Events

If `PodScheduled=False`, investigate scheduling constraints such as resources, taints/tolerations, selectors or affinity.

If the Pod is scheduled but the container is not running, inspect image/runtime/startup issues.

## 2. Deployment has fewer Ready Pods than desired

Inspect the hierarchy:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods -o wide
```

Remember:

```text
Deployment controls rollout/version state.
ReplicaSet controls replica count for a template.
```

If a managed Pod is deleted, the ReplicaSet should create a replacement.

## 3. Rollout appears stuck

Useful commands:

```bash
kubectl rollout status deployment/<name>
kubectl get rs
kubectl get pods -l app=<label>
kubectl describe deployment <name>
kubectl describe pod <pod-name>
```

During rollout, old and new ReplicaSets can both have non-zero desired replicas. New Pods may temporarily be `ContainerCreating` while old Ready Pods are retained.

## 4. Service exists but application is unreachable

Check in this order:

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get pods --show-labels
kubectl get endpointslice
```

Questions:

```text
Does Service selector match Pod labels?
Are there backend endpoints?
Is targetPort correct?
Are Pods Ready?
Is the application listening on targetPort?
```

A Service can exist successfully while matching zero Pods.

## 5. ClusterIP vs Pod IP confusion

Service ClusterIP should remain stable while backend Pod addresses can change after Pod recreation.

Example from the lab:

```text
Service ClusterIP: 10.109.112.172
Backend Pods:      10.244.x.x
```

Do not hard-code Pod IPs for normal service discovery.

## 6. NodePort troubleshooting

Inspect:

```bash
kubectl get svc <service>
kubectl describe svc <service>
kubectl get nodes -o wide
```

Identify:

```text
Node internal IP
NodePort
Service selector
Endpoints
ExternalTrafficPolicy
```

With `ExternalTrafficPolicy: Cluster`, the backend Pod can be on a different Node from the Node whose IP receives the request.

## 7. ReplicaSet new/old confusion

Do not assume a hash such as `766544d56d` is newer because its value looks larger.

Use:

```bash
kubectl rollout history deployment/<name>
kubectl get rs
```

and inspect the revision annotation if needed.

## 8. Declarative manifest debugging

After changing a YAML file:

```bash
kubectl apply -f <file>
kubectl get deployment
kubectl get rs
kubectl get pods
```

If only `replicas` changes, expect scaling of the existing active ReplicaSet.

If `spec.template` changes (image, env, Pod labels, etc.), expect a rollout and another ReplicaSet.
