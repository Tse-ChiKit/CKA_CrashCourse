# CKA Day 2 — Cluster Bootstrap, CNI and Worker Join

Date: 2026-08-16

## Day 2 goals

Today focused on turning the prepared Linux VMs into a working 3-node Kubernetes cluster.

### Completed

- [x] Bootstrap the control-plane node with `kubeadm init`
- [x] Use an alternative Kubernetes image repository when the default registry is unreachable
- [x] Configure the Pod CIDR for the chosen CNI
- [x] Install Calico as the cluster CNI
- [x] Join `worker1` to the cluster
- [x] Join `worker2` to the cluster
- [x] Understand why `kubectl` behaves differently on the control-plane and worker nodes
- [x] Understand the purpose and execution model of DaemonSets such as `kube-proxy`
- [x] Review component responsibilities with CKA-style questions

## Cluster state at the end of Day 2

The lab is now a real multi-node Kubernetes cluster:

```text
control-plane
├── kube-apiserver
├── kube-controller-manager
├── kube-scheduler
├── etcd
├── kubelet
├── kube-proxy
└── Calico components

worker1
├── kubelet
├── kube-proxy
└── Calico components

worker2
├── kubelet
├── kube-proxy
└── Calico components
```

The control-plane was created with `kubeadm init`, networking was provided by Calico, and both workers successfully joined the cluster.

## Key outcome

Day 2 completes the basic infrastructure needed for the rest of the course. From Day 3 onward, labs can focus on Kubernetes workloads and administration rather than VM/bootstrap setup.

## Suggested Day 3 starting point

Move into workload primitives and daily administration:

- Pods
- Deployments
- ReplicaSets
- Services
- imperative vs declarative operations
- `kubectl create/apply/edit/delete`
- YAML practice
- CKA-style workload troubleshooting
