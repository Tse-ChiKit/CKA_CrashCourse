# Day 0 — Execution Notes

## Status

Day 0 completed successfully with no installation or configuration errors reported.

## What was completed

The three-node VMware lab was prepared for kubeadm-based Kubernetes installation. The environment work covered:

- validating the Ubuntu VMs
- checking hostnames, network connectivity and node identity
- disabling swap
- loading `overlay` and `br_netfilter`
- enabling IPv4 forwarding and bridge packet processing
- installing and configuring `containerd`
- setting containerd to use the systemd cgroup driver
- installing `kubeadm`, `kubelet` and `kubectl`
- running `kubeadm init --dry-run` on the control-plane VM

No errors were encountered during the Day 0 setup.

## Mental model established

```text
Kubernetes node
│
├── kubelet
│    └── Node agent that reconciles the node with the desired state
│
├── containerd
│    └── Container runtime that actually runs containers
│
└── kubeadm
     └── Bootstrap/lifecycle tool used to initialize and join nodes
```

A useful distinction from Day 0:

```text
Installing kubelet != creating a Kubernetes cluster
```

Before `kubeadm init`, kubelet may not yet have the complete cluster configuration it needs. This becomes important later when diagnosing kubelet startup failures.

## Linux foundation observations

Kubernetes depends heavily on the underlying Linux host. Day 0 touched several host-level prerequisites that will be revisited later:

- kernel modules
- cgroups
- systemd
- packet forwarding
- Linux bridge/netfilter behavior
- container runtime sockets

These are not separate from Kubernetes administration; they are part of the CKA troubleshooting surface.

## Day 0 result

The lab is ready for the next phase: performing a real `kubeadm init`, configuring kubectl, installing a CNI / Pod network, and joining the worker nodes.
