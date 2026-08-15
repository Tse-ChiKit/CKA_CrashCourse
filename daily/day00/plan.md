# Day 0 — Lab Environment Setup

## Objective

Prepare three Ubuntu VMs as a clean kubeadm-based Kubernetes lab for CKA study.

Target topology:

```text
Windows 10
└── VMware Workstation
    ├── k8s-control
    ├── k8s-worker01
    └── k8s-worker02
```

## Focus

Day 0 intentionally focuses on the host and node prerequisites rather than Kubernetes workloads.

Core components introduced:

- `containerd` — container runtime
- `kubelet` — node agent
- `kubeadm` — cluster bootstrap and lifecycle tool

## Tasks

### 1. Validate each VM

Check:

```bash
hostname
cat /etc/os-release
uname -r
nproc
free -h
df -h /
hostname -I
ip route
cat /sys/class/dmi/id/product_uuid
swapon --show
```

Validate:

- unique hostname
- unique VM identity / UUID
- enough CPU, RAM and disk
- stable IP connectivity between nodes
- outbound Internet connectivity

### 2. Disable swap

```bash
sudo swapoff -a
swapon --show
```

Also disable the swap entry in `/etc/fstab` so the change survives reboot.

### 3. Prepare required kernel modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Persist them:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

### 4. Configure networking sysctl settings

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### 5. Install and configure containerd

```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Use the systemd cgroup driver:

```text
SystemdCgroup = true
```

Then restart and enable containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
```

### 6. Install Kubernetes v1.35 packages

Install:

- `kubeadm`
- `kubelet`
- `kubectl`

Then hold package versions to avoid accidental upgrades during the crash course.

### 7. Run kubeadm dry-run on the control-plane node

```bash
sudo kubeadm init --dry-run
```

Purpose:

- validate prerequisites
- inspect what `kubeadm init` plans to generate
- learn the bootstrap sequence before creating the real cluster

## Day 0 completion criteria

- [x] Three Ubuntu VMs available
- [x] Nodes can communicate
- [x] Unique host identities
- [x] Swap disabled
- [x] Required kernel modules configured
- [x] Packet forwarding configured
- [x] containerd installed and running
- [x] systemd cgroup driver configured
- [x] kubeadm installed
- [x] kubelet installed
- [x] kubectl installed
- [x] `kubeadm init --dry-run` completed successfully

## Exam mindset

Day 0 establishes the Linux and kubeadm foundation needed later for CKA topics such as:

- cluster installation
- control-plane troubleshooting
- kubelet troubleshooting
- container runtime troubleshooting
- networking prerequisites
- cluster upgrade and maintenance
