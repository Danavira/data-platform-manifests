# k8s-vps-cluster-lab
Documentation of setting up Kubernetes (K3s) on a multi-node cluster, from server hardening to installations.
```
kubernetes-on-vps/
│
├── README.md
├── architecture/
│   ├── cluster-architecture.md
│   ├── network-design.md
│   └── diagrams/
│
├── docs/
│   ├── 01-vps-setup.md
│   ├── 02-server-hardening.md
│   ├── 03-install-container-runtime.md
│   ├── 04-install-kubernetes.md
│   ├── 05-create-cluster.md
│   ├── 06-networking.md
│   ├── 07-storage.md
│   └── troubleshooting.md
│
├── scripts/
│   ├── init-server.sh
│   ├── install-containerd.sh
│   ├── install-kubernetes.sh
│   └── join-node.sh
│
├── manifests/
│   ├── nginx-deployment.yaml
│   ├── ingress.yaml
│   └── storage-class.yaml
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── ansible/
│   ├── inventory
│   ├── playbook.yml
│   └── roles/
│
└── images/
    └── architecture-diagram.png
```


Steps to secure VPS.

1. Update the system
```
# update package lists
apt update

# upgrade installed packages
apt upgrade -y

# install useful base tools
apt install -y curl wget git vim htop unzip
```

2. Set Hostname (Optional but recommended)
```
# set hostname
hostnamectl set-hostname my-vps

# verify
hostnamectl
```

3. Create a non-root sudo user (Never operate servers as root)
```
# create new user
adduser devuser

# add the user to sudo group
usermod -aG sudo devuser

# verify
groups devuser
```

4. Copy SSH Key to new user

```
# on local machine, run this
ssh-copy-id devuser@SERVER_IP

# if ssh-copy-id not available
mkdir -p /home/devuser/.ssh
nano /home/devuser/.ssh/authorized_keys
# paste your public key
chmod 700 /home/devuser/.ssh
chmod 600 /home/devuser/.ssh/authorized_keys
chown -R devuser:devuser /home/devuser/.ssh
```

5. Test login before continuing
```
# on local machine, on another terminal
ssh devuser@SERVER_IP
```

6. Harden SSH Configuration
```
# edit sshd config
sudo nano /etc/ssh/sshd_config

# change default SSH port
Port 2222

# disable root login
PermitRootLogin no

# disable password authentication
PasswordAuthentication no

# enable key authentication
PubkeyAuthentication yes

# disable challenge auth
ChallengeResponseAuthentication no

# optional: restrict SSH to specific users
AllowUsers devuser
```

7. Restart SSH
```
# restart ssh service
sudo systemctl restart sshd

ssh devuser@SERVER_IP -p 2222
```

8. Install and Configure Firewall (UFW)
```
# install ufw
sudo apt install ufw -y

# allow new SSH port
sudo ufw allow 2222/tcp

# allow other necessary ports (optional)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# deny all by default
sudo ufw default deny incoming
sudo ufw default allow outgoing

# rate limiting for SSH
sudo ufw limit 2222/tcp

# enable firewall
sudo ufw enable

# verify
sudo ufw status verbose
```

9. Install Fail2Ban (Brute Force Protection)
```
sudo apt install -y fail2ban

# create config
sudo nano /etc/fail2ban/jail.local

# add this configuration
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 1h

# restart fail2ban
sudo systemctl restart fail2ban

# verify
sudo fail2ban-client status sshd
```

10. Enable Automatic security updates
```
sudo apt install -y unattended-upgrades

# enable automatic updates
sudo dpkg-reconfigure unattended-upgrades

# verify
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

11. Install Time synchronization (NTP)
```
sudo apt install -y chrony

# verify
sudo systemctl enable chrony
sudo systemctl start chrony

# check sync status
chronyc tracking
```

12. Remove unnecessary services
```
# check open ports
sudo ss -tulpn

# check running services
systemctl list-units --type=service

# disable unnecessary services
sudo systemctl disable --now apache2
sudo systemctl disable --now cups
sudo systemctl disable --now avahi-daemon
```

13. Install and Configure Monitoring (Optional)
```
# install node_exporter
sudo apt install -y prometheus-node-exporter

# enable and start
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# verify
sudo systemctl status node_exporter


# another way
sudo apt install -y htop logwatch
```

14. Install Auditd (Security Logging)
```
sudo apt install -y auditd audispd-plugins

# enable and start
sudo systemctl enable auditd
sudo systemctl start auditd

# verify
sudo systemctl status auditd
```

15. Secure shared memory
```
sudo nano /etc/fstab

# add this line
tmpfs /run/shm tmpfs defaults,noexec,nosuid,nodev,size=512M 0 0

# verify
sudo mount -o remount /run/shm
```

16. Disable Swap (Important for kubernetes nodes)
```
# check swap
swapon --show

# disable swap
sudo swapoff -a

# remove swap from fstab
sudo nano /etc/fstab

# verify
swapon --show
```

17. Final system check
```
# check open ports
ss -tulpn

# check firewall
sudo ufw status

# check fail2ban
sudo fail2ban-client status

# check updates
apt list --upgradable


# check disk usage
df -h

# check memory usage
free -h

# check cpu usage
htop
```

18. Snapshot the VPS
At this point:

base system hardened

SSH secured

firewall active

brute force protection active

Now create a provider snapshot.

This becomes your golden VPS template.
```



k8s setup

1. The "Daily Driver" Utilities
These aren't "deployments" in your cluster; they are tools you install on your laptop to manage the VPS cluster efficiently.

k9s: A terminal UI for Kubernetes. It’s significantly faster than typing kubectl commands and is a favorite among SREs for real-time monitoring.

kubectx & kubens: Simple scripts to switch between clusters and namespaces. When you start managing multiple environments (e.g., dev vs. prod), these are lifesavers.

Lens / OpenLens: If you prefer a GUI, this is the "VS Code of Kubernetes." It gives you a beautiful bird's-eye view of your VPS resources.

2. GitOps: The "Data Engineer" Way
In a professional setup, you should never use kubectl apply -f. Instead, you use GitOps. You push code to GitHub, and the cluster "pulls" it.

Argo CD: This is the gold standard. It provides a web UI that shows exactly what is running in your cluster vs. what is in your Git repo.

FluxCD: A more lightweight, "Kubernetes-native" alternative to Argo. It’s often used in smaller clusters or edge computing (like K3s).

3. The Observability Stack (Loki-Stack)
As a Data Engineer, if your pipeline fails, you need to know why. You need the LGTM stack (Loki, Grafana, Tempo, Mimir) or a simpler version:

Prometheus & Grafana: The industry standard for metrics (CPU, Memory, Disk usage).

Loki: Like "ELK" but much lighter. It collects logs from all your pods and lets you query them in Grafana.

Prometheus Operator: Since you're learning about Operators, this is the best one to start with. It manages the entire monitoring setup for you.

4. Storage & Secret Management
Since you're on a VPS, you don't have "Google/AWS Cloud Storage" by default.

Longhorn: Created by the Rancher team (the same people who made K3s). It provides highly available persistent storage across your VPS nodes. If one node dies, your data is still safe on the others.

External Secrets Operator: Instead of putting passwords in YAML files, this tool pulls secrets from a secure vault (like Bitwarden, HashiCorp Vault, or even AWS Secrets Manager) and injects them into Kubernetes.



blog topics
kubernetes single node setup vs multi node cluster setup
upgrading kubernetes (lens, kubectx, k9s)