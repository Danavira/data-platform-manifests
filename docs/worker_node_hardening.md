Steps to prepare and secure a k3s worker node before joining the cluster.

Start with general_server_hardening.md first, then apply the steps below.

---

1. Open Required Ports in UFW (k3s worker)
```
# API server (worker -> control plane)
sudo ufw allow 6443/tcp

# kubelet API
sudo ufw allow 10250/tcp

# Flannel VXLAN (if using Flannel CNI)
sudo ufw allow 8472/udp

# WireGuard IPv4 (if using WireGuard CNI)
sudo ufw allow 51820/udp

# WireGuard IPv6 (if using WireGuard CNI)
sudo ufw allow 51821/udp

# verify
sudo ufw status verbose
```

2. Disable Automatic Reboots (unattended-upgrades)
A surprise reboot will evict pods. Make sure this line is set in `/etc/apt/apt.conf.d/50unattended-upgrades`:
```
Unattended-Upgrade::Automatic-Reboot "false";
```
Drain the node manually before rebooting for kernel updates:
```
# run from control plane
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

3. Load Required Kernel Modules
```
# load modules immediately
sudo modprobe br_netfilter overlay ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh

# persist across reboots
cat <<EOF | sudo tee /etc/modules-load.d/k3s.conf
br_netfilter
overlay
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
EOF

# verify
lsmod | grep -e br_netfilter -e overlay -e ip_vs
```

4. Configure sysctl for Kubernetes Networking
```
cat <<EOF | sudo tee /etc/sysctl.d/k3s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# apply immediately
sudo sysctl --system

# verify
sysctl net.ipv4.ip_forward
```

5. Disable Swap (Required for kubernetes nodes)
```
# check swap
swapon --show

# disable swap
sudo swapoff -a

# remove swap from fstab (comment out or delete the swap line)
sudo nano /etc/fstab

# verify
swapon --show
```

6. Skip apt node_exporter
Do NOT install `prometheus-node-exporter` via apt — it will conflict with the node-exporter
DaemonSet already deployed in the cluster. The cluster manages that deployment.

7. Join the cluster
```
# on the control plane, get the node token
sudo cat /var/lib/rancher/k3s/server/node-token

# on the worker node
curl -sfL https://get.k3s.io | K3S_URL=https://<CONTROL_PLANE_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -

# verify from control plane
kubectl get nodes
```
