# Day 0 — Troubleshooting Record

## Outcome

No blocking errors were encountered during Day 0.

The following areas were validated successfully:

- VM connectivity
- swap configuration
- kernel modules
- sysctl networking settings
- containerd installation
- systemd cgroup configuration
- Kubernetes package installation
- `kubeadm init --dry-run`

## Preventive checks worth remembering

Even without a failure, Day 0 established a useful troubleshooting checklist for future kubeadm/node problems:

```text
1. Host identity
   hostname
   product UUID

2. Network
   hostname -I
   ip route
   ping

3. Resources
   nproc
   free -h
   df -h

4. Swap
   swapon --show

5. Kernel / networking
   lsmod
   sysctl net.ipv4.ip_forward

6. Container runtime
   systemctl status containerd

7. kubelet
   systemctl status kubelet
   journalctl -u kubelet

8. kubeadm
   kubeadm init --dry-run
```

## Security note

The dry-run output contained a generated bootstrap token and discovery information. Those ephemeral credentials are intentionally not copied into this public repository.

## Day 0 troubleshooting takeaway

A Kubernetes cluster problem should not immediately be treated as a `kubectl` problem. The failure may live below the Kubernetes API layer:

```text
Linux host
   ↓
systemd / kernel / networking
   ↓
containerd
   ↓
kubelet
   ↓
control-plane / Kubernetes API
```

Future troubleshooting should identify which layer is failing before changing configuration.
