# CKA Day 2 — Commands and Lab Record

## 1. Bootstrap the control plane

The lab used kubeadm to initialize the control-plane node.

```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers
```

Why these options mattered in this lab:

```text
--pod-network-cidr=10.244.0.0/16
    Defines the Pod network range expected by the lab networking setup.

--image-repository registry.aliyuncs.com/google_containers
    Uses an alternative image source because the default Kubernetes registry
    was not reachable reliably from the lab environment.
```

## 2. Configure kubectl on the control-plane node

Typical kubeadm post-init configuration:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verification:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

## 3. Install Calico

Calico was installed as the cluster CNI so Pods can communicate across nodes.

Useful verification commands:

```bash
kubectl get pods -n kube-system
kubectl get ds -n kube-system
kubectl get nodes -o wide
```

During CNI startup, inspect unhealthy Pods with:

```bash
kubectl describe pod <pod-name> -n kube-system
kubectl logs <pod-name> -n kube-system
```

## 4. Join the two worker nodes

`kubeadm init` prints a join command containing the control-plane endpoint, token and discovery CA hash.

Run that generated command with `sudo` on each worker, conceptually:

```bash
sudo kubeadm join <control-plane-ip>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

This was completed on both:

```text
worker1
worker2
```

Verify from the control-plane node:

```bash
kubectl get nodes
kubectl get nodes -o wide
```

Expected end state:

```text
control-plane   Ready
worker1         Ready
worker2         Ready
```

## 5. Inspect cluster system workloads

```bash
kubectl get pods -n kube-system -o wide
kubectl get ds -n kube-system
kubectl get ds kube-proxy -n kube-system
```

These commands help connect API objects to the Pods actually running on each VM.

## 6. Static Pod inspection

On the control-plane node:

```bash
sudo ls -l /etc/kubernetes/manifests/
```

Typical kubeadm static Pod manifests include:

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Useful local inspection:

```bash
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
sudo systemctl status kubelet
sudo journalctl -u kubelet --no-pager -n 100
```

## 7. Runtime troubleshooting commands

On the node where an image/Pod problem occurs:

```bash
sudo systemctl status containerd
sudo crictl images
sudo crictl ps -a
sudo journalctl -u containerd --no-pager -n 100
sudo journalctl -u kubelet --no-pager -n 100
```

## End-of-day cluster checkpoint

Before shutting down the VMs, the important checkpoint is that the cluster has:

- one initialized control-plane node,
- Calico installed,
- two joined workers,
- system Pods running across the nodes,
- working `kubectl` administration from the control-plane node.
