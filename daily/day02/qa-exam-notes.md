# CKA Day 2 — Q&A and Exam Notes

## Q1. Who makes the assigned container actually run on a Node?

Question: after Scheduler assigns a Pod to `worker-02`, who is responsible for making it run there?

Answer:

```text
kubelet
```

Reason: kubelet on the assigned Node observes the Pod assignment and works with the container runtime to start the containers.

## Q2. If a bare Pod is deleted, who recreates it?

Answer:

```text
Nobody, unless a higher-level controller owns it.
```

A directly created bare Pod has no ReplicaSet maintaining its desired replica count.

## Q3. Who recreates a Pod belonging to a Deployment?

Important correction from the discussion:

```text
Deployment does not directly maintain individual Pod count.
ReplicaSet maintains the replicas for a particular Pod template.
```

The relevant ReplicaSet controller notices the replica deficit and creates a replacement Pod.

## Q4. Why does a recreated Pod get a different name?

For example:

```text
ReplicaSet: nginx-deploy-8b9dbd8c9
Pod:        nginx-deploy-8b9dbd8c9-xxxxx
```

A replacement is a new Pod object, so it receives a new unique suffix and UID. The ReplicaSet portion of the name stays the same while that ReplicaSet remains the owner.

## Q5. Why are there two ReplicaSets after changing an image?

Changing the image changes the Deployment's Pod template. Deployment creates/uses a new ReplicaSet for the new template and gradually scales between old and new ReplicaSets to implement RollingUpdate.

## Q6. Who manages rollback, Deployment or ReplicaSet?

Correct answer:

```text
Deployment
```

Memory rule:

```text
Deployment = rollout / rollback / version management
ReplicaSet = Pod replica count
```

## Q7. Why keep an old ReplicaSet at 0 replicas?

Old ReplicaSets preserve previous Pod templates/revisions and can be scaled back up during rollback.

## Q8. Will Service ClusterIP change if a backend Pod is replaced?

Correct answer:

```text
No.
```

The Service provides a stable virtual IP. The backend Pod IP can change; EndpointSlice/endpoint data is updated accordingly.

## Q9. Can NodePort on worker-01 send traffic to a Pod on worker-02?

Correct answer:

```text
Yes, with the normal Cluster external traffic policy.
```

NodePort does not require the destination Pod to live on the Node whose IP was used by the client.

## Q10. What happens if a Service selector does not match any Pods?

Correct answer:

```text
The Service object still exists, but it has no matching backend Pods.
```

Troubleshooting clue:

```bash
kubectl describe svc <service>
kubectl get endpointslice
kubectl get pods --show-labels
```

## Q11. What are the two appearances of `app: web-demo` in Deployment YAML?

```yaml
spec:
  selector:
    matchLabels:
      app: web-demo

  template:
    metadata:
      labels:
        app: web-demo
```

The selector identifies Pods managed by the workload; the Pod template labels are placed on the Pods that the Deployment creates.

A Service has its own independent selector and can also select these Pod labels for networking.

## Q12. What happens when the same manifest is applied again with replicas changed?

Example:

```yaml
replicas: 3
```

to:

```yaml
replicas: 5
```

then:

```bash
kubectl apply -f web-demo.yaml
```

Correct mental model:

```text
The existing Deployment is updated/reconciled to 5 replicas.
```

It does not create another Deployment simply because `apply` was run again.

## Q13. How do I know which ReplicaSet is newest?

Fast clue:

```text
AGE
```

More reliable:

```bash
kubectl rollout history deployment/<name>
```

and inspect:

```text
deployment.kubernetes.io/revision
```

Do not compare the numeric-looking hash in ReplicaSet names.

## Q14. Why can there temporarily be more Pods than `replicas` during rollout?

Because RollingUpdate can use `maxSurge`, allowing extra new Pods while old Pods are still serving traffic. `maxUnavailable` controls how many desired replicas may be unavailable during the rollout.

## CKA high-value memory rules

```text
scheduler          = choose Node
kubelet            = run assigned Pod on Node
Deployment         = versions / rollout / rollback
ReplicaSet         = maintain Pod replica count
Service            = stable network entry
label              = tag on object/Pod
selector           = condition used to find matching objects
ClusterIP          = cluster-internal Service entry
NodePort           = NodeIP:nodePort external-style entry
scale              = change replicas
rollout            = change Pod template
```

## Service troubleshooting checklist

```text
1. Does the Service exist?
2. Is its selector correct?
3. Do Pod labels match?
4. Are EndpointSlices/endpoints populated?
5. Is targetPort correct?
6. Is the application actually listening on that port?
7. Are the Pods Ready?
```
