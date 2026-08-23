# CKA Day 2 — Workloads, Services and Declarative YAML

Date: 2026-08-23

## Day 2 goals

Move from cluster bootstrap into day-to-day Kubernetes workload administration on the working 3-node kubeadm lab.

## Completed

- [x] Understand Pod as the smallest schedulable unit
- [x] Trace Pod lifecycle through API server, scheduler, kubelet and containerd
- [x] Create and inspect a bare Pod
- [x] Understand why a bare Pod is not self-healing
- [x] Create a Deployment and observe its ReplicaSet and Pods
- [x] Scale a Deployment
- [x] Delete a managed Pod and observe self-healing
- [x] Perform a rolling image update
- [x] Inspect old/new ReplicaSets during rollout
- [x] Roll back a Deployment
- [x] Understand Deployment vs ReplicaSet responsibilities
- [x] Create ClusterIP and NodePort Services
- [x] Understand Service selectors, Pod labels and Endpoints
- [x] Understand Service port, targetPort and nodePort
- [x] Generate Deployment YAML with --dry-run=client -o yaml
- [x] Modify replicas and image declaratively with kubectl apply
- [x] Understand when a template change creates a new ReplicaSet
- [x] Review apply vs edit vs replace

## End-of-day mental model

```text
Deployment
  -> manages rollout / rollback / versions
  -> owns ReplicaSets

ReplicaSet
  -> maintains desired Pod replica count
  -> provides self-healing for managed Pods

Pod
  -> smallest schedulable workload unit
  -> contains one or more containers

Service
  -> provides a stable network entry point
  -> selects Pods using labels
```

## Lab checkpoint

The cluster now has working examples of:

- Deployments with multiple replicas
- ReplicaSet self-healing
- rolling updates and rollback
- ClusterIP Services
- NodePort Services
- label/selector based backend discovery
- declarative workload management through YAML and kubectl apply

## Day 3 starting point

Continue from workload administration into configuration, health and scheduling topics, including ConfigMap, Secret, environment variables, probes, resources and related troubleshooting.
