# CKA Day 1 — Kubernetes Architecture, kubeadm Bootstrap, CNI and Worker Join

Date: 2026-08-16

## Day 1 goals

Day 1 covered both Kubernetes architecture fundamentals and the complete bootstrap of the first real 3-node kubeadm cluster.

### Completed

- [x] Understand the main Kubernetes control-plane and node components
- [x] Understand the responsibilities of kube-apiserver, kube-controller-manager, kube-scheduler and kubelet
- [x] Bootstrap the control-plane node with `kubeadm init`
- [x] Troubleshoot Kubernetes image pull / registry reachability problems
- [x] Use an alternative image repository for kubeadm bootstrap images
- [x] Configure the Pod CIDR used by the lab networking setup
- [x] Configure `kubectl` on the control-plane node
- [x] Understand kubeconfig, `admin.conf`, `super-admin.conf` and RBAC bootstrap concepts
- [x] Install Calico as the cluster CNI
- [x] Join `worker1` and `worker2` to the cluster
- [x] Understand DaemonSets such as `kube-proxy`
- [x] Understand why `kubectl` on a worker is separate from the worker being successfully joined
- [x] Practice CKA-style component responsibility questions
- [x] Understand static Pods and `/etc/kubernetes/manifests`

## Cluster state at the end of Day 1

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

The control-plane was created with `kubeadm init`, Calico provides Pod networking, and both worker nodes successfully joined the cluster.

## Key outcome

Day 1 moved from Kubernetes architecture theory into a functioning multi-node cluster. This gives the rest of the course a real environment for workload deployment, administration and troubleshooting.

## Suggested Day 2 starting point

Continue with workload primitives and everyday Kubernetes administration:

- Pods
- ReplicaSets
- Deployments
- Services
- imperative vs declarative operations
- YAML practice
- `kubectl create/apply/edit/delete`
- CKA-style workload troubleshooting
