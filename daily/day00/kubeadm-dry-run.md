# Day 0 — `kubeadm init --dry-run` Walkthrough

## Purpose

The Day 0 control-plane VM successfully ran:

```bash
sudo kubeadm init --dry-run
```

A dry-run is useful because it exposes much of the control-plane bootstrap sequence without actually creating the real cluster.

> Important: the final success message in a dry-run means the simulated initialization completed. It does **not** mean a persistent Kubernetes cluster has already been created.

## Observed environment

The output showed:

```text
Kubernetes version: v1.35.7
control-plane hostname: k8s-control
control-plane IP: 192.168.32.131
API server port: 6443
service subnet: 10.96.0.0/12
cluster domain: cluster.local
CoreDNS service IP: 10.96.0.10
container runtime endpoint: /var/run/containerd/containerd.sock
kubelet cgroup driver: systemd
```

The command noticed a newer remote Kubernetes version but stayed on the stable `1.35` release line, matching the installed kubeadm minor version.

## What `kubeadm init` plans to do

A useful high-level sequence from the output is:

```text
1. Preflight checks
        ↓
2. Pull / validate required images
        ↓
3. Generate PKI certificates
        ↓
4. Generate kubeconfig files
        ↓
5. Create local etcd static Pod manifest
        ↓
6. Create control-plane static Pod manifests
        ↓
7. Configure kubelet
        ↓
8. Start / wait for the control plane
        ↓
9. Store cluster and kubelet configuration
        ↓
10. Configure bootstrap token + RBAC
        ↓
11. Mark and taint the control-plane node
        ↓
12. Install CoreDNS
        ↓
13. Install kube-proxy
        ↓
14. Print kubeconfig and worker join instructions
```

This is one of the most important Day 0 takeaways: `kubeadm init` is not one mysterious action. It orchestrates many smaller Kubernetes and Linux configuration steps.

---

## 1. Preflight checks

The output begins with:

```text
[preflight] Running pre-flight checks
```

This validates whether the host is suitable for initialization. In future troubleshooting, a failed `kubeadm init` should first be classified as either:

```text
Host prerequisite failure
vs.
Control-plane startup failure
```

They require different investigation paths.

---

## 2. PKI and certificates

`kubeadm` plans to generate certificates and keys for components including:

- Kubernetes CA
- kube-apiserver
- apiserver → kubelet client
- front proxy
- etcd CA/server/peer/client
- apiserver → etcd client
- ServiceAccount signing keys

The API server certificate includes the control-plane host/IP plus Kubernetes service DNS names.

Mental model:

```text
Kubernetes control-plane components
        ↓
communicate over authenticated TLS
        ↓
/etc/kubernetes/pki
```

This directory will be important later during certificate and control-plane troubleshooting.

---

## 3. kubeconfig files

The dry-run generates kubeconfigs such as:

```text
admin.conf
super-admin.conf
kubelet.conf
controller-manager.conf
scheduler.conf
```

A kubeconfig answers three practical questions:

```text
Which cluster/API server?
Which identity/credentials?
Which context should be used?
```

For example, after a real initialization, the administrator normally copies `admin.conf` into the regular user's `$HOME/.kube/config`.

---

## 4. etcd as a static Pod

The output plans to create:

```text
/etc/kubernetes/manifests/etcd.yaml
```

Important observations:

```text
etcd client port: 2379
etcd peer port:   2380
data directory:   /var/lib/etcd
```

`etcd` is not installed here as an ordinary systemd service. In this kubeadm topology, kubelet runs etcd from a **static Pod manifest**.

This relationship is central to future CKA troubleshooting:

```text
/etc/kubernetes/manifests/etcd.yaml
            ↓
kubelet watches staticPodPath
            ↓
kubelet asks containerd to run etcd
```

---

## 5. Control-plane components are static Pods

The dry-run plans to create manifests for:

```text
kube-apiserver
kube-controller-manager
kube-scheduler
```

under:

```text
/etc/kubernetes/manifests/
```

This gives an important architecture chain:

```text
/etc/kubernetes/manifests/*.yaml
              ↓
           kubelet
              ↓
          containerd
              ↓
   control-plane containers
```

This explains why a broken manifest in this directory can take down an individual control-plane component even though there is no Deployment managing it.

### kube-apiserver

Observed configuration includes:

```text
advertise address: 192.168.32.131
secure port: 6443
etcd endpoint: https://127.0.0.1:2379
authorization: Node,RBAC
service CIDR: 10.96.0.0/12
```

Architecture relationship:

```text
kubectl / clients
      ↓ HTTPS :6443
kube-apiserver
      ↓
     etcd
```

### kube-controller-manager

Runs the controllers that continually reconcile actual state toward desired state.

### kube-scheduler

Selects a suitable node for Pods that do not yet have `spec.nodeName` assigned.

---

## 6. kubelet configuration

The generated kubelet configuration revealed several useful settings:

```text
cgroupDriver: systemd
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
clusterDNS: 10.96.0.10
clusterDomain: cluster.local
staticPodPath: /etc/kubernetes/manifests
```

This connects several Day 0 setup choices:

```text
kubelet
  ├── systemd cgroups
  ├── talks to containerd
  ├── watches static Pod manifests
  └── configures Pods to use cluster DNS
```

This is why configuring containerd with `SystemdCgroup = true` was relevant rather than an arbitrary installation tweak.

---

## 7. Cluster configuration stored in Kubernetes

The dry-run plans to create configuration in `kube-system`, including the `kubeadm-config` and `kubelet-config` ConfigMaps.

This is useful later during:

- cluster upgrades
- inspecting kubeadm configuration
- node bootstrap
- troubleshooting configuration drift

---

## 8. Control-plane labels and taint

The output plans to mark `k8s-control` with the control-plane role and add:

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

Meaning:

```text
Control-plane node
      ↓
NoSchedule taint
      ↓
ordinary workload Pods are not scheduled there
unless they have a matching toleration
```

This is an early preview of the later CKA Scheduling topic: **taints repel Pods; tolerations allow Pods to tolerate those taints.**

---

## 9. Worker bootstrap token and RBAC

The dry-run generated a temporary bootstrap token and the RBAC objects required for a new kubelet to bootstrap its identity.

For security, the actual token from the terminal output is intentionally **not stored in this public repository**.

High-level flow:

```text
worker runs kubeadm join
        ↓
bootstrap token
        ↓
contacts API server
        ↓
requests certificate (CSR)
        ↓
RBAC / bootstrap approval
        ↓
node obtains long-term kubelet credentials
```

This will be revisited when the worker nodes are joined for real.

---

## 10. CoreDNS

The dry-run plans to create CoreDNS as Kubernetes resources rather than static control-plane Pods.

Observed design:

```text
CoreDNS Deployment
        ↓
2 replicas
        ↓
kube-dns Service
ClusterIP: 10.96.0.10
Ports: 53 UDP/TCP
```

This matches the kubelet's generated `clusterDNS` value:

```text
kubelet clusterDNS → 10.96.0.10
                         ↓
                    kube-dns Service
                         ↓
                    CoreDNS Pods
```

This connection will become important during the Networking and DNS troubleshooting days.

---

## 11. kube-proxy

The dry-run plans to install kube-proxy as a `DaemonSet`.

Mental model:

```text
DaemonSet
   ↓
one kube-proxy Pod per eligible node
```

Its job is related to implementing Kubernetes Service networking on nodes. The exact networking mechanism will be explored later rather than memorized on Day 0.

---

## 12. What the final dry-run message actually means

The dry-run ended with a success message and printed instructions for:

1. configuring the user's kubeconfig
2. installing a Pod network
3. joining worker nodes

However, because this was `--dry-run`, those instructions refer to the simulated output location and generated temporary credentials.

Do **not** reuse the dry-run bootstrap token for the real cluster.

For the real bootstrap, the expected workflow is:

```text
real kubeadm init
      ↓
copy real admin.conf
      ↓
install CNI / Pod network
      ↓
use the real kubeadm join command on workers
      ↓
kubectl get nodes
```

---

## CKA takeaways from this one command

`kubeadm init --dry-run` already exposes multiple exam-relevant concepts:

- kubeadm cluster lifecycle
- PKI/certificates
- kubeconfig
- etcd
- static Pods
- kubelet configuration
- container runtime endpoint
- cgroup driver
- API server port and service CIDR
- RBAC
- bootstrap tokens / node join
- taints
- CoreDNS
- kube-proxy / DaemonSet

The important goal is not to memorize the entire output. It is to know where each responsibility lives and which files/components to inspect when something breaks.
