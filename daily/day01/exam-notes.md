# CKA Day 1 — Exam Notes

## 1. Static scheduler Pod disappeared

Scenario:

```text
kubectl get pods -n kube-system
```

no longer shows a healthy scheduler and:

```text
/etc/kubernetes/manifests/kube-scheduler.yaml
```

was deleted.

If the manifest is recreated correctly, the scheduler can return because the **kubelet watches the static Pod manifest directory** and reconciles the local static Pods from those files.

Memory association:

```text
/etc/kubernetes/manifests
    -> static Pod
    -> kubelet watches local manifest
```

## 2. Deployment wants 3 replicas but only 2 exist

Who mainly notices the desired/actual mismatch?

```text
Answer: kube-controller-manager
```

Who decides whether the newly created unscheduled Pod should run on `worker1` or `worker2`?

```text
Answer: kube-scheduler
```

Fast rule:

```text
controller-manager = maintain desired state
scheduler          = choose a node
kubelet            = run it on that node
```

## 3. DaemonSet question pattern

If asked where a DaemonSet "runs", be precise:

- the DaemonSet is a Kubernetes API object,
- its controller reconciles desired per-node Pods,
- the resulting Pods run on eligible nodes.

Typical use cases include networking agents, monitoring agents, log collectors and `kube-proxy`.

Useful commands:

```bash
kubectl get ds -A
kubectl get ds <name> -n <namespace> -o wide
kubectl describe ds <name> -n <namespace>
```

## 4. kubectl authentication question pattern

If `kubectl` tries `localhost:8080` or cannot reach the API server from a worker, check kubeconfig before blaming Pod networking.

Useful checks:

```bash
echo $KUBECONFIG
kubectl config current-context
kubectl config view
ls -l ~/.kube/config
```

Do not assume worker nodes automatically receive administrator kubeconfig credentials.

## 5. kubeadm / image pull question pattern

If kubeadm is stuck pulling images:

1. Check container runtime health.
2. Identify the exact failing image.
3. Check registry/network connectivity.
4. Inspect containerd/kubelet logs.
5. Change mirrors or image repositories only after identifying the layer.

Useful commands:

```bash
sudo systemctl status containerd
sudo crictl images
sudo journalctl -u containerd --no-pager -n 100
sudo journalctl -u kubelet --no-pager -n 100
```

## High-value Day 1 distinctions

```text
Static Pod manifest     != Deployment manifest
DaemonSet object        != one daemon process on one VM
Node joined cluster     != kubectl configured on that node
kubeadm image repo      != global image registry setting
Controller Manager      != Scheduler
Scheduler               != Kubelet
```

These distinctions are especially valuable in CKA troubleshooting questions.
