# CKA Day 2 — Knowledge Notes

## 1. Static Pods and the control plane

In a kubeadm-based control-plane node, core control-plane components such as the API server, scheduler and controller manager are commonly run as **static Pods**.

Their manifests are stored under:

```bash
/etc/kubernetes/manifests/
```

The kubelet watches this directory. If a valid manifest such as:

```text
/etc/kubernetes/manifests/kube-scheduler.yaml
```

is recreated after accidental deletion, the kubelet can detect it and recreate the corresponding static Pod automatically.

### Important mental model

The kubelet does not need a Deployment or ReplicaSet to manage a static Pod. It directly watches the manifest file on that node.

---

## 2. Controller Manager vs Scheduler

CKA-style scenario:

```text
Deployment replicas = 3
Actual running Pods = 2
```

The main component responsible for detecting that the desired replica count and actual replica count differ is:

```text
kube-controller-manager
```

More specifically, the relevant controllers reconcile the desired state and create a new Pod object when necessary.

After the new Pod exists but has no node assigned, the component that decides whether it should run on `worker1`, `worker2`, etc. is:

```text
kube-scheduler
```

### Responsibility split

```text
Controller Manager:
"We need another Pod."

Scheduler:
"This Pod should run on worker2."

Kubelet on worker2:
"I will make the assigned Pod actually run on this node."
```

This distinction is very important for CKA troubleshooting questions.

---

## 3. DaemonSet mental model

A DaemonSet is a **cluster-level Kubernetes API object**. It does not live inside one VM.

For example:

```bash
kubectl get ds kube-proxy -n kube-system
```

shows one DaemonSet object stored in the Kubernetes cluster.

The DaemonSet controller ensures that matching nodes run a corresponding Pod. Therefore one `kube-proxy` DaemonSet can create kube-proxy Pods across the control-plane and worker nodes, depending on its selectors/tolerations and node eligibility.

Conceptually:

```text
One DaemonSet object
        |
        +--> kube-proxy Pod on control-plane
        +--> kube-proxy Pod on worker1
        +--> kube-proxy Pod on worker2
```

The DaemonSet itself is not "running on worker1" or "stored on worker2". The object is part of cluster state; the resulting Pods run on nodes.

---

## 4. Why kubectl failed on a worker node

Running:

```bash
kubectl get ds kube-proxy -n kube-system
```

on a worker produced an error similar to:

```text
Get "http://localhost:8080/api?...":
dial tcp 127.0.0.1:8080: connect: connection refused
```

The important point is that `kubectl` is only a client. It needs a kubeconfig containing information such as:

- Kubernetes API server address
- cluster CA/certificate information
- user/client credentials
- context

The control-plane setup normally prepares an admin kubeconfig, for example:

```text
/etc/kubernetes/admin.conf
```

A worker node does not automatically receive an administrator kubeconfig just because it joined the cluster.

Therefore, a worker can successfully run kubelet and Kubernetes workloads while an interactive `kubectl` command from that worker still fails.

### CKA takeaway

Do not confuse:

```text
Node successfully joined cluster
```

with:

```text
Current shell has kubectl credentials to administer cluster
```

They are separate concerns.

---

## 5. Calico and Pod networking

A Kubernetes cluster needs a CNI implementation so Pods on different nodes can communicate using Pod IPs.

For this lab, Calico is used as the CNI.

The cluster bootstrap and CNI configuration must agree on networking assumptions, especially the Pod address range.

The control plane was initialized with a Pod network CIDR:

```bash
--pod-network-cidr=10.244.0.0/16
```

The important principle is not the exact number itself but that the kubeadm cluster configuration and the selected CNI configuration must be compatible.

After the CNI is installed and healthy, nodes can transition to `Ready` and normal Pod networking becomes available.

---

## 6. Image repository changes are resource-specific

Changing an image repository in one Kubernetes workload does not globally rewrite every image in the cluster.

For example, modifying the image used by the `kube-proxy` DaemonSet affects the Pods created from that DaemonSet template. It does not automatically change the image registry for Calico, CoreDNS, or arbitrary future workloads.

Likewise, the kubeadm option:

```bash
--image-repository registry.aliyuncs.com/google_containers
```

changes where kubeadm pulls the Kubernetes bootstrap/control-plane images from. It does not act as a universal cluster-wide container registry setting.

### Mental model

Always ask:

```text
Which Kubernetes object or tool owns this image reference?
```

Then change the image source at that level.
