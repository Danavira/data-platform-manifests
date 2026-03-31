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