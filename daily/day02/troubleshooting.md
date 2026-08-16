# CKA Day 2 — Troubleshooting Notes

## 1. kubeadm init stuck while pulling images

### Symptom

`kubeadm init` stayed at the image pulling stage for a long time.

### Diagnosis

The container runtime itself was working, but the machine could not reliably reach the default Kubernetes image registry. Direct network access such as `curl` also failed.

This made the problem primarily a registry/network reachability issue rather than a kubelet or containerd failure.

### Resolution used in the lab

Use an alternative Kubernetes image repository during cluster initialization:

```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers
```

### Lesson

When kubeadm hangs on image pulling, separate the problem into layers:

```text
1. Is containerd running?
2. Can the node reach the registry?
3. Can the required image actually be pulled?
4. Is the image name/tag correct?
```

Do not immediately assume kubeadm itself is broken.

---

## 2. kubeadm init reached control-plane creation but failed on RBAC bootstrap

### Symptom

`kubeadm init` progressed through static Pod creation and kubelet startup, then failed around creation of a `ClusterRoleBinding` / admin bootstrap step.

### Important concepts learned

RBAC bindings connect a user/group to a role.

Example recovery command discussed during the lab:

```bash
sudo KUBECONFIG=/etc/kubernetes/super-admin.conf \
  kubectl create clusterrolebinding kubeadm-cluster-admins \
  --clusterrole=cluster-admin \
  --group=kubeadm:cluster-admins
```

Breakdown:

```text
kubeadm-cluster-admins
    Name of the ClusterRoleBinding object.
    This name can be chosen as long as it is valid and does not conflict.

--clusterrole=cluster-admin
    Bind to the built-in cluster-admin ClusterRole.

--group=kubeadm:cluster-admins
    Grant that role to the named group.

KUBECONFIG=/etc/kubernetes/super-admin.conf
    Tell kubectl which kubeconfig/credentials to use for this command.
```

### Important clarification

`admin.conf` or `super-admin.conf` is **not created by the ClusterRoleBinding command**. These kubeconfig files are produced as part of kubeadm's cluster bootstrap process. The environment variable only tells `kubectl` which kubeconfig to use.

---

## 3. kube-proxy / Calico image pull confusion across nodes

### Question encountered

If a DaemonSet is changed, does every node automatically use a globally changed registry? Why can one worker still fail?

### Key lesson

A DaemonSet contains a Pod template. Updating the image in that template affects Pods managed by that DaemonSet, but only for that specific workload.

It does not configure containerd's registry behavior globally and does not alter unrelated workloads.

Therefore a worker may still fail to pull another component's image if:

- that component uses a different image reference,
- the registry is unreachable from that node,
- credentials/mirror configuration differ,
- the new DaemonSet rollout has not completed,
- or the failing image belongs to a different workload entirely.

### Troubleshooting commands

```bash
kubectl get pods -n kube-system -o wide
kubectl get ds -n kube-system
kubectl describe pod <pod-name> -n kube-system
kubectl describe node <node-name>
```

For container-runtime level checks on the affected node:

```bash
sudo crictl images
sudo crictl ps -a
sudo journalctl -u containerd --no-pager -n 100
sudo journalctl -u kubelet --no-pager -n 100
```

---

## 4. kubectl on worker tried localhost:8080

### Symptom

```text
couldn't get current server API group list
Get "http://localhost:8080/api?...":
dial tcp 127.0.0.1:8080: connect: connection refused
```

### Cause

The worker shell had no usable kubeconfig for `kubectl`.

Joining a worker with `kubeadm join` configures kubelet so the node can participate in the cluster. It does not automatically configure the current user's `kubectl` as a cluster administrator.

### Preferred lab workflow

Run administrative `kubectl` commands from the control-plane node where the admin kubeconfig is already configured.

If there is a deliberate reason to administer the cluster from another machine, copy/configure an appropriate kubeconfig securely rather than treating every worker as an admin workstation.

---

## Day 2 troubleshooting pattern

A useful troubleshooting hierarchy from today's work:

```text
kubectl / API object layer
        ↓
kubelet / controller behavior
        ↓
container runtime
        ↓
image registry / network
        ↓
OS / VM networking
```

First determine which layer is failing before changing configuration.
