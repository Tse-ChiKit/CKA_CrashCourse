# CKA Day 1 — Knowledge Notes

## 1. Kubernetes component responsibilities

A useful mental model from today's study:

```text
kube-apiserver
    -> cluster API entry point

kube-controller-manager
    -> detects differences between desired state and actual state

kube-scheduler
    -> chooses which node an unscheduled Pod should run on

kubelet
    -> makes sure the Pods assigned to its node actually run

containerd
    -> container runtime used to pull images and run containers
```

Example:

```text
Deployment replicas = 3
Actual Pods = 2
```

The controller manager notices the desired-state mismatch and causes another Pod to be created. The scheduler then decides which node that Pod should run on. The kubelet on that node makes the Pod actually run.

## 2. Static Pods and the control plane

In a kubeadm control-plane node, core components such as the API server, scheduler and controller manager commonly run as static Pods.

Their manifests are stored under:

```bash
/etc/kubernetes/manifests/
```

The kubelet watches this directory. If a valid manifest such as `kube-scheduler.yaml` is recreated after accidental deletion, kubelet can detect it and recreate the corresponding static Pod automatically.

Important distinction:

```text
Static Pod -> kubelet watches a local manifest directly
Deployment -> controllers reconcile an API object stored in the cluster
```

## 3. DaemonSet mental model

A DaemonSet is a cluster-level Kubernetes API object. It is not a daemon process that lives inside one particular VM.

For example, one `kube-proxy` DaemonSet can result in kube-proxy Pods running across eligible nodes:

```text
One DaemonSet object
        |
        +--> kube-proxy Pod on control-plane
        +--> kube-proxy Pod on worker1
        +--> kube-proxy Pod on worker2
```

The DaemonSet controller is responsible for maintaining those per-node Pods.

## 4. kubectl and kubeconfig

`kubectl` is only a client. It needs a kubeconfig containing information such as:

- Kubernetes API server address
- cluster CA/certificate data
- user credentials
- context

On a kubeadm control-plane node, kubeconfig files are normally generated under `/etc/kubernetes/`, including `admin.conf` and `super-admin.conf`.

A worker joining the cluster does not automatically mean the interactive shell on that worker has administrative `kubectl` credentials.

Therefore:

```text
Node joined cluster != kubectl configured for the current user
```

## 5. RBAC and ClusterRoleBinding

The recovery command discussed during bootstrap was:

```bash
sudo KUBECONFIG=/etc/kubernetes/super-admin.conf \
  kubectl create clusterrolebinding kubeadm-cluster-admins \
  --clusterrole=cluster-admin \
  --group=kubeadm:cluster-admins
```

Breakdown:

```text
kubeadm-cluster-admins
    -> name of the ClusterRoleBinding object

--clusterrole=cluster-admin
    -> built-in ClusterRole being granted

--group=kubeadm:cluster-admins
    -> group receiving that role

KUBECONFIG=/etc/kubernetes/super-admin.conf
    -> tells kubectl which credentials/config to use
```

The ClusterRoleBinding command does not create `admin.conf` or `super-admin.conf`; kubeadm generates those kubeconfig files during bootstrap.

## 6. Calico and Pod networking

Kubernetes needs a CNI implementation for Pod networking across nodes. This lab uses Calico.

The control plane was initialized with:

```bash
--pod-network-cidr=10.244.0.0/16
```

The important concept is compatibility between the cluster Pod CIDR assumptions and the chosen CNI configuration. The exact CIDR is a configuration choice, not a universally required Kubernetes value.

## 7. Image repository settings are scoped

The kubeadm flag:

```bash
--image-repository registry.aliyuncs.com/google_containers
```

changes where kubeadm gets its bootstrap/control-plane images. It is not a universal registry setting for every future workload.

Likewise, changing the image in the `kube-proxy` DaemonSet only changes the Pods managed by that DaemonSet. It does not automatically rewrite Calico images or configure containerd globally.

Useful question when troubleshooting image pulls:

```text
Which object or tool owns this image reference?
```
