# CKA Day 2 — Knowledge Notes

## 1. Pod lifecycle and component responsibilities

A Pod is the smallest unit Kubernetes schedules onto a Node.

Typical flow:

```text
kubectl
  -> kube-apiserver
  -> Pod object created
  -> kube-scheduler chooses a Node
  -> kubelet on that Node observes the assignment
  -> kubelet asks the container runtime (containerd) to start containers
  -> CNI configures Pod networking
```

In the lab, an nginx Pod was scheduled to `k8s-worker-02`, where kubelet and containerd started the container and Calico assigned a Pod IP.

Important distinction:

```text
scheduler = choose Node
kubelet   = make assigned Pod run on Node
containerd = run the container
```

## 2. Bare Pod vs managed Pod

A Pod created directly with:

```bash
kubectl run nginx --image=nginx
```

is a bare Pod. If it is deleted, no higher-level workload controller necessarily recreates it.

A Pod managed by a ReplicaSet is different. If one managed Pod disappears, the ReplicaSet controller notices that actual replicas are below desired replicas and creates a replacement.

## 3. Deployment vs ReplicaSet

Mental model:

```text
Deployment
   -> manages versions, rollout and rollback
   -> manages ReplicaSets

ReplicaSet
   -> maintains a desired number of Pods
   -> creates replacement Pods when needed
```

For a Deployment named `nginx-deploy`, names looked like:

```text
Deployment: nginx-deploy
ReplicaSet: nginx-deploy-8b9dbd8c9
Pod:        nginx-deploy-8b9dbd8c9-xxxxx
```

The final Pod suffix is unique/random for each new Pod object.

Do not interpret the ReplicaSet hash as a version number. Hash magnitude does not indicate which ReplicaSet is newer.

## 4. Scaling vs rollout

Changing only:

```yaml
spec:
  replicas: 3
```

normally scales the existing ReplicaSet.

Changing the Pod template, for example:

```yaml
spec:
  template:
    spec:
      containers:
      - image: nginx:1.27
```

changes the desired Pod template and causes Deployment to create/use a different ReplicaSet for a rollout.

Useful memory rule:

```text
Scale   = change replica count
Rollout = change Pod template
```

## 5. Rolling Update

During a rolling update, old and new ReplicaSets can coexist.

Example observed:

```text
old RS: desired decreases
new RS: desired increases
```

The total number of Pods may temporarily exceed `spec.replicas` because RollingUpdate can use `maxSurge`. Kubernetes also controls how many replicas may be unavailable using `maxUnavailable`.

Default strategy is commonly conceptually similar to:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

## 6. Rollback

Deployment keeps older ReplicaSets at zero replicas so it can reuse a previous template during rollback.

```bash
kubectl rollout undo deployment/nginx-deploy
```

Rollback is a Deployment responsibility, not a ReplicaSet decision.

Memory rule:

```text
Deployment = version management
ReplicaSet = replica management
```

## 7. How to identify new vs old ReplicaSets

Quick clue:

```text
AGE
```

A smaller age usually means the ReplicaSet was created more recently.

More reliable method: inspect Deployment revisions.

```bash
kubectl rollout history deployment/web-demo
```

ReplicaSets carry an annotation similar to:

```yaml
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
```

Do not decide new/old based on the hash text in the ReplicaSet name.

## 8. Service purpose

Pod IPs are ephemeral. A Service gives clients a stable logical entry point for a changing set of Pods.

```text
Client
  -> Service
  -> matching Pods
```

A Service typically discovers backend Pods using labels and a selector.

## 9. Labels and selectors

Pod template:

```yaml
template:
  metadata:
    labels:
      app: web-demo
```

These labels are placed on Pods created from the template.

Deployment selector:

```yaml
selector:
  matchLabels:
    app: web-demo
```

is used for workload ownership/reconciliation.

Service has its own independent selector:

```yaml
spec:
  selector:
    app: web-demo
```

Both workload controllers and Services can select the same Pods for different reasons.

If a Service selector does not match any Pod labels, the Service object still exists but has no useful backends/Endpoints.

## 10. ClusterIP and NodePort

ClusterIP:

```text
stable virtual Service IP
primarily for cluster-internal access
```

NodePort:

```text
NodeIP:nodePort
  -> Service
  -> backend Pod
```

A NodePort Service is not limited to forwarding only to Pods on the Node that received the traffic when external traffic policy is `Cluster`.

Observed lab example:

```text
NodePort: 32209
Service port: 80
Target port: 80
```

Conceptually:

```text
NodeIP:32209
  -> Service port 80
  -> Pod targetPort 80
```

NodePort and ClusterIP are two entry mechanisms on the same Service model; traffic does not need to be understood as literally traversing the ClusterIP first.

## 11. Service Endpoints

`kubectl describe svc` showed backend addresses such as:

```text
10.244.x.x:80
```

These are the current Pod backends. If a Pod is deleted and recreated with a new IP, the Service ClusterIP stays stable while backend endpoint data changes.

Modern Kubernetes also represents this information through EndpointSlice resources.

## 12. Declarative YAML

A generated Deployment skeleton:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-demo
  template:
    metadata:
      labels:
        app: web-demo
    spec:
      containers:
      - name: nginx
        image: nginx
```

Key distinction:

```text
metadata.labels
  = labels on the Deployment object itself

spec.template.metadata.labels
  = labels on Pods created from the template
```

`spec` expresses desired state. `status` is normally populated by Kubernetes and generally should not be manually maintained in normal manifests.

## 13. apply, edit and replace

```bash
kubectl apply -f file.yaml
```

Best mental model: keep a manifest describing desired state and repeatedly reconcile the existing object to it.

```bash
kubectl edit deployment web-demo
```

Useful for direct interactive edits to the live object; changes are not automatically written back into a separate local manifest.

```bash
kubectl replace -f file.yaml
```

Replaces the object definition with the supplied complete definition; know it exists, but `apply`, `edit` and imperative commands are generally more common in this study workflow.
