# CKA Day 2 — Exam Notes

## Component responsibility questions

### Scenario 1 — Static scheduler Pod disappeared

`kubectl get pods -n kube-system` no longer shows a healthy scheduler and:

```text
/etc/kubernetes/manifests/kube-scheduler.yaml
```

was accidentally deleted.

If the manifest is recreated correctly, the scheduler can come back because the **kubelet watches the static Pod manifest directory** and reconciles the local static Pods from those files.

CKA keyword association:

```text
/etc/kubernetes/manifests
    -> static Pod
    -> kubelet watches local manifest
```

---

### Scenario 2 — Deployment wants 3 replicas but only 2 exist

Which component mainly notices the desired/actual state mismatch?

```text
Answer: kube-controller-manager
```

The controllers reconcile the Deployment/ReplicaSet state and cause another Pod object to be created.

Who chooses whether that new unscheduled Pod runs on `worker1` or `worker2`?

```text
Answer: kube-scheduler
```

Fast memory rule:

```text
controller-manager = maintain desired state
scheduler          = choose a node
kubelet            = run it on that node
```

---

## DaemonSet question pattern

If asked where a DaemonSet "runs", be precise:

- the DaemonSet is a Kubernetes API object,
- the DaemonSet controller reconciles it,
- Pods created from it run on matching nodes.

Typical use cases:

- networking agents,
- node monitoring agents,
- log collectors,
- `kube-proxy`.

Useful exam commands:

```bash
kubectl get ds -A
kubectl get ds <name> -n <namespace> -o wide
kubectl describe ds <name> -n <namespace>
```

---

## kubectl authentication question pattern

If `kubectl` attempts to use `localhost:8080` or otherwise cannot reach the API server from a node, check kubeconfig before blaming Kubernetes networking.

Useful checks:

```bash
echo $KUBECONFIG
kubectl config current-context
kubectl config view
ls -l ~/.kube/config
```

On kubeadm control-plane nodes, useful kubeconfig files may exist under:

```text
/etc/kubernetes/
```

Do not assume workers automatically have admin credentials.

---

## kubeadm / image pull question pattern

If kubeadm is stuck pulling images:

1. Check container runtime health.
2. Identify the exact image that cannot be pulled.
3. Check registry/network connectivity.
4. Inspect kubelet/containerd logs.
5. Only then change mirrors/repositories if appropriate.

Useful commands:

```bash
sudo systemctl status containerd
sudo crictl images
sudo journalctl -u containerd --no-pager -n 100
sudo journalctl -u kubelet --no-pager -n 100
```

---

## High-value Day 2 distinctions

Memorize these pairs:

```text
Static Pod manifest     != Deployment manifest
DaemonSet object        != one daemon process on one VM
Node joined cluster     != kubectl configured on that node
kubeadm image repo      != global image registry setting
Controller Manager      != Scheduler
Scheduler               != Kubelet
```

These distinctions are common sources of wrong answers in troubleshooting questions.
