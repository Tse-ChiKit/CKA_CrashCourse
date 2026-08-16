# CKA Day 1 — Commands and Lab Record

## 1. Bootstrap the control plane

```bash
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers
```

Why these options mattered:

```text
--pod-network-cidr=10.244.0.0/16
    Defines the Pod network range used by the lab networking setup.

--image-repository registry.aliyuncs.com/google_containers
    Uses an alternative source for kubeadm-managed Kubernetes images because
    the default registry was not reachable reliably from the lab environment.
```

## 2. Configure kubectl on the control-plane node

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

## 3. RBAC recovery command discussed

```bash
sudo KUBECONFIG=/etc/kubernetes/super-admin.conf \
  kubectl create clusterrolebinding kubeadm-cluster-admins \
  --clusterrole=cluster-admin \
  --group=kubeadm:cluster-admins
```

## 4. Install Calico

Calico was installed as the cluster CNI so Pods can communicate across nodes.

Useful checks:

```bash
kubectl get pods -n kube-system
kubectl get ds -n kube-system
kubectl get nodes -o wide
```

For unhealthy system Pods:

```bash
kubectl describe pod <pod-name> -n kube-system
kubectl logs <pod-name> -n kube-system
```

## 5. Join worker1 and worker2

`kubeadm init` prints a join command containing the control-plane endpoint, token and discovery CA hash.

Run the generated command on each worker:

```bash
sudo kubeadm join <control-plane-ip>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

Completed on:

```text
worker1
worker2
```

Verify from the control-plane:

```bash
kubectl get nodes
kubectl get nodes -o wide
```

Expected checkpoint:

```text
control-plane   Ready
worker1         Ready
worker2         Ready
```

## 6. Inspect DaemonSets and system workloads

```bash
kubectl get pods -n kube-system -o wide
kubectl get ds -n kube-system
kubectl get ds kube-proxy -n kube-system
```

## 7. Inspect static Pod manifests

On the control-plane node:

```bash
sudo ls -l /etc/kubernetes/manifests/
```

Typical kubeadm manifests:

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Useful inspection:

```bash
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
sudo systemctl status kubelet
sudo journalctl -u kubelet --no-pager -n 100
```

## 8. Runtime troubleshooting commands

```bash
sudo systemctl status containerd
sudo crictl images
sudo crictl ps -a
sudo journalctl -u containerd --no-pager -n 100
sudo journalctl -u kubelet --no-pager -n 100
```

## End-of-Day 1 checkpoint

The lab now has:

- one initialized control-plane node,
- Calico installed,
- two joined worker nodes,
- system Pods running across the nodes,
- working `kubectl` administration from the control-plane node,
- a practical understanding of the first major cluster bootstrap troubleshooting paths.
