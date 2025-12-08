# 📧 MAIL01 Complete Setup Guide - Healthcare Clinic Project

**Complete step-by-step guide for MAIL01 (Email Server with Mailcow)**  
**Manitoba Institute of Trades and Technology (M.I.T.T.) - Network & Systems Administrator Capstone**

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [VM Specifications](#vm-specifications)
- [Part 1: VM Creation in Proxmox](#part-1-vm-creation-in-proxmox)
- [Part 2: Debian 12 Installation](#part-2-debian-12-installation)
- [Part 3: Network Configuration](#part-3-network-configuration)
- [Part 4: Desktop GUI Installation](#part-4-desktop-gui-installation)
- [Part 5: System Preparation](#part-5-system-preparation)
- [Part 6: Docker Installation](#part-6-docker-installation)
- [Part 7: Mailcow Installation](#part-7-mailcow-installation)
- [Part 8: Mailcow Configuration](#part-8-mailcow-configuration)
- [Part 9: Testing & Verification](#part-9-testing--verification)
- [Part 10: Troubleshooting](#part-10-troubleshooting)
- [Completion Checklist](#completion-checklist)
- [Quick Reference](#quick-reference)

---

## Project Overview

**MAIL01** is an email server running Mailcow Dockerized - a complete email solution with SMTP, IMAP, webmail, and spam filtering for Healthcare Clinic.

### Technology Stack
- **Operating System:** Debian 12 (Bookworm)
- **Desktop Environment:** XFCE Desktop (lightweight)
- **Containerization:** Docker + Docker Compose
- **Email Platform:** Mailcow Dockerized
- **Services:** Postfix (SMTP), Dovecot (IMAP), SOGo (Webmail), Rspamd (Spam Filter)

### Network Details
- **VM Name:** MAIL01
- **IP Address:** 10.10.40.20/24
- **Gateway:** 10.10.40.1 (HSRP VIP on R1/R2)
- **DNS Servers:** 10.10.40.10 (DC01), 10.10.40.11 (DC02)
- **VLAN:** 40 (IT/Servers)
- **Domain:** healthclinic.local
- **Email Domain:** @healthclinic.local

---

## VM Specifications
```yaml
Proxmox Configuration:
  VM ID: 120
  Name: MAIL01
  Node: pve-main (10.10.40.15)
  
Hardware:
  CPU: 4 cores (host type)
  RAM: 8192 MB (8 GB)
  Disk: 80 GB (local-lvm storage)
  Network: VirtIO on vmbr0 (VLAN 40)
  
Operating System:
  Distribution: Debian 12 (Bookworm)
  Architecture: x86_64 (amd64)
  Kernel: 6.1.x
  Desktop: XFCE (lightweight)
  
Network Configuration:
  IP: 10.10.40.20/24 (static)
  Gateway: 10.10.40.1
  DNS: 10.10.40.10, 10.10.40.11
  Hostname: mail01.healthclinic.local
```

---

## Part 1: VM Creation in Proxmox

### Step 1.1: Access Proxmox Web Interface
```
URL: https://10.10.40.15:8006
Username: root
Password: Clinic@2025!
```

### Step 1.2: Upload Debian ISO (if not already uploaded)
```bash
# Via Proxmox console:
cd /var/lib/vz/template/iso/

# Download Debian 12 netinst ISO
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso

# Verify
ls -lh debian-12.8.0-amd64-netinst.iso
```

**Or via Web UI:**
1. Click **pve-main** → **local** → **ISO Images**
2. Click **Download from URL**
3. URL: `https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso`
4. Click **Query URL** → **Download**

### Step 1.3: Create New VM

Click **Create VM** button

#### Tab 1: General
```
Node: pve-main
VM ID: 120
Name: MAIL01
☑ Start at boot
```
Click **Next**

#### Tab 2: OS
```
☑ Use CD/DVD disc image file (iso)
Storage: local
ISO image: debian-12.8.0-amd64-netinst.iso

Guest OS:
  Type: Linux
  Version: 6.x - 2.6 Kernel
```
Click **Next**

#### Tab 3: System
```
Graphic card: Default
Machine: Default (i440fx)
BIOS: Default (SeaBIOS)
SCSI Controller: VirtIO SCSI single
☑ Qemu Agent
```
Click **Next**

#### Tab 4: Disks
```
Bus/Device: SCSI 0
Storage: local-lvm
Disk size (GiB): 80
Cache: Default
☑ Discard
☑ IO thread
```
Click **Next**

#### Tab 5: CPU
```
Sockets: 1
Cores: 4
Type: host
```
Click **Next**

#### Tab 6: Memory
```
Memory (MiB): 8192
☑ Ballooning Device
```
Click **Next**

#### Tab 7: Network
```
Bridge: vmbr0
Model: VirtIO (paravirtualized)
☐ Firewall
```
Click **Next**

#### Tab 8: Confirm
```
☑ Start after created
```
Click **Finish**

---

## Part 2: Debian 12 Installation

### Step 2.1: Open VM Console

1. Click **MAIL01 (120)**
2. Click **Console**
3. VM boots from Debian ISO

### Step 2.2: Installation Wizard

#### Boot Menu
```
Debian GNU/Linux installer menu (BIOS mode)

> Install
```
Select **Install**, press **Enter**

#### Language
```
Select a language: English
```
Press **Enter**

#### Location
```
Select your location: Canada
```
Press **Enter**

#### Keyboard
```
Configure the keyboard: American English
```
Press **Enter**

#### Network Configuration - Hostname
```
Hostname: mail01
```
Press **Enter**

#### Network Configuration - Domain
```
Domain name: healthclinic.local
```
Press **Enter**

#### Root Password
```
Root password: Clinic@2025!
Re-enter: Clinic@2025!
```
Press **Enter**

#### User Account
```
Full name: Mail Admin
Username: mailadmin
Password: Clinic@2025!
Re-enter: Clinic@2025!
```
Press **Enter**

#### Partition Disks
```
Partitioning method:
> Guided - use entire disk
```
Press **Enter**
```
Select disk: SCSI (0,0,0) (sda) - 85.9 GB
```
Press **Enter**
```
Partitioning scheme:
> All files in one partition
```
Press **Enter**
```
Finish partitioning and write changes to disk
Write changes? Yes
```
Press **Enter**

**Wait 2-5 minutes**

#### Package Manager
```
Scan extra media? No
Mirror country: Canada
Mirror: deb.debian.org
HTTP proxy: (leave blank)
```

#### Popularity Contest
```
Participate? No
```

#### Software Selection
```
Choose software to install:

[*] SSH server
[*] standard system utilities
[ ] Debian desktop environment
[ ] (everything else unchecked)
```

**Only select SSH server and standard utilities**

Press **Tab** to Continue, press **Enter**

**Wait 5-10 minutes**

#### GRUB Boot Loader
```
Install GRUB? Yes
Device: /dev/sda
```
Press **Enter**

#### Installation Complete
```
Select: Continue
```
Press **Enter**

**VM reboots**

---

## Part 3: Network Configuration

### Step 3.1: First Login
```
mail01 login: root
Password: Clinic@2025!
```

### Step 3.2: Check Network Interface
```bash
ip link show
```

**Note interface name (usually `ens18`)**

### Step 3.3: Configure Static IP
```bash
nano /etc/network/interfaces
```

**Replace DHCP section with:**
```
auto ens18
iface ens18 inet static
    address 10.10.40.20/24
    gateway 10.10.40.1
    dns-nameservers 10.10.40.10 10.10.40.11
    dns-search healthclinic.local
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 3.4: Restart Network
```bash
systemctl restart networking
# Or
reboot
```

### Step 3.5: Verify Network
```bash
# Login again
ip addr show ens18
# Should show: 10.10.40.20/24

# Test connectivity
ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
ping -c 3 8.8.8.8
```

---

## Part 4: Desktop GUI Installation

### Step 4.1: Update System
```bash
apt update
apt upgrade -y
```

### Step 4.2: Install XFCE Desktop
```bash
apt install xfce4 xfce4-goodies lightdm -y
```

**Takes 5-10 minutes**

**Select `lightdm` when prompted**

### Step 4.3: Reboot
```bash
reboot
```

### Step 4.4: Login to Desktop

**Graphical login appears**

1. Click **mailadmin**
2. Password: **Clinic@2025!**

**XFCE Desktop loads!** 🖥️

### Step 4.5: Open Terminal

**Applications → Terminal Emulator**

---

## Part 5: System Preparation

### Step 5.1: Update and Install Tools
```bash
sudo apt update
sudo apt upgrade -y

sudo apt install -y \
    curl \
    wget \
    git \
    vim \
    nano \
    net-tools \
    htop \
    dnsutils \
    ca-certificates \
    gnupg \
    lsb-release
```

### Step 5.2: Set Hostname
```bash
sudo hostnamectl set-hostname mail01.healthclinic.local

hostname -f
# Should show: mail01.healthclinic.local
```

### Step 5.3: Edit /etc/hosts
```bash
sudo nano /etc/hosts
```

**Add:**
```
127.0.0.1       localhost
10.10.40.20     mail01.healthclinic.local mail01

::1             localhost ip6-localhost ip6-loopback
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 5.4: Disable IPv6
```bash
sudo nano /etc/sysctl.conf
```

**Add at end:**
```
# Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`
```bash
sudo sysctl -p

# Verify
ip addr show | grep inet6
# Should return nothing
```

---

## Part 6: Docker Installation

### Step 6.1: Remove Old Docker
```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

### Step 6.2: Add Docker Repository
```bash
# Add GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
```

### Step 6.3: Install Docker
```bash
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

### Step 6.4: Start Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

Press **Q**

### Step 6.5: Verify Docker
```bash
docker --version
docker compose version
sudo docker run hello-world
```

### Step 6.6: Add User to Docker Group
```bash
sudo usermod -aG docker mailadmin
newgrp docker
docker ps
```

---

## Part 7: Mailcow Installation

## Log into your **MAIL01** terminal (via XFCE or SSH) and run these one by one – this corrects the repository file and forces a clean setup.

### 7.1. Fix DNS First (if curl fails – points to your domain controllers)

```bash
sudo nano /etc/resolv.conf
```
Replace the contents with:
```bash
textnameserver 10.10.40.10  # DC01 DNS
nameserver 10.10.40.11  # DC02 DNS
nameserver 8.8.8.8      # Fallback Google DNS
```

### 7.2 Clean Up the Broken Repository File
```bash
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo apt update  # This should now run without Docker errors
```

### 7.3. Re-Run the Corrected Docker Repository Addition (Fixed Echo Command)
Copy-paste this exact block (the missing backslash is now fixed – properly multi-line):
```bash
# Add GPG key (safe to re-run)
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
# Add repository (CORRECTED: proper backslashes for multi-line)
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
```
### 7. 4. Verify the Fix (Check What Was Written)
Bashcat /etc/apt/sources.list.d/docker.list
### Step 7.5: Generate Configuration
```bash
sudo ./generate_config.sh
```

**Enter hostname:**
```
Mail server hostname (FQDN):
mail01.healthclinic.local
```

### Step 7.5: Edit Configuration
```bash
sudo nano mailcow.conf
```

**Verify/change:**
```bash
MAILCOW_HOSTNAME=mail01.healthclinic.local
MAILCOW_IP=10.10.40.20
TZ=America/Winnipeg

HTTP_PORT=80
HTTP_BIND=0.0.0.0

HTTPS_PORT=443
HTTPS_BIND=0.0.0.0

API_ALLOW_FROM=127.0.0.1,10.10.40.0/24

# Optional: Disable Solr to save RAM
SKIP_SOLR=y

# Keep ClamAV for demonstration
SKIP_CLAMD=n
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.4: Pull Docker Images
```bash
sudo docker compose pull
```

**Takes 5-15 minutes to download ~2-3 GB**

### Step 7.5: Start Mailcow
```bash
sudo docker compose up -d
```

**First start takes 5-10 minutes**

### Step 7.6: Check Status
```bash
sudo docker compose ps
```

**All containers should show State: Up** ✅

---

## Part 8: Mailcow Configuration

### Step 8.1: Access Web UI

**Open browser:**
```
URL: https://10.10.40.20
```

**Accept SSL warning**

### Step 8.2: First Login
```
Username: admin
Password: moohoo
```

### Step 8.3: Change Admin Password

1. Click **Admin** (top-right)
2. Click **Edit administrator**
3. New password: `Clinic@2025!`
4. Confirm: `Clinic@2025!`
5. Click **Save**

### Step 8.4: Add Domain

1. **Configuration** → **Mail setup**
2. **Domains** tab
3. Click **Add domain**
```
Domain: healthclinic.local
Max mailboxes: 10
Max aliases: 10
Max quota: 3072 MB
```
<img width="1563" height="363" alt="image" src="https://github.com/user-attachments/assets/9a54c3a9-a7ab-41ba-8d3e-2e7529b443c3" />

4. Click **Add domain and restart SOGo**

### Step 8.5: Create Mailboxes

**Add mailbox:**

1. **Mailboxes** tab
2. Click **Add mailbox**
```
Username: admin
Domain: healthclinic.local
Full name: Administrator
Password: Clinic@2025!
Confirm: Clinic@2025!
Quota: 3072 MB
Active: ☑
```
<img width="1629" height="526" alt="image" src="https://github.com/user-attachments/assets/87a27245-2164-4967-a3c6-b06c11c7e172" />

3. Click **Add**

**Create additional mailboxes:**
- `doctor` - Dr. Sarah Johnson
- `reception` - Reception Desk
- `it` - IT Support

**Email addresses:**
- admin@healthclinic.local
- doctor@healthclinic.local
- reception@healthclinic.local
- it@healthclinic.local

<img width="1908" height="908" alt="image" src="https://github.com/user-attachments/assets/1cc172e5-8480-4e89-a842-6849e5385723" />


<img width="1875" height="968" alt="image" src="https://github.com/user-attachments/assets/1fad60af-1146-4c1c-a434-b395ed1b35ab" />



### Step 8.6: Test Webmail

1. Click **Apps** → **Webmail**
2. Login:
```
   Username: admin@healthclinic.local
   Password: Clinic@2025!
```

**SOGo webmail opens!** ✅

---

## Part 9: Testing & Verification

### Step 9.1: Send Test Email

**In SOGo:**

1. **Mail** → **New Message**
2. To: `admin@healthclinic.local`
3. Subject: `Test Email`
4. Body: `Test from Mailcow`
5. Click **Send**

### Step 9.2: Check Receipt

1. Refresh inbox
2. Test email appears ✅

### Step 9.3: Test Between Accounts

1. Send from `admin@` to `doctor@`
2. Logout
3. Login as `doctor@healthclinic.local`
4. Check inbox ✅

### Step 9.4: Check Containers
```bash
sudo docker compose ps
# All should show Up
```

### Step 9.5: Check Services

**In Web UI:**

1. **System** → **Service Status**
2. All services green ✅

### Step 9.6: Test External Access

**From another computer:**
```
URL: https://10.10.40.20/SOGo
Username: admin@healthclinic.local
Password: Clinic@2025!
```

**Should work!** ✅

---

## Part 10: Troubleshooting

### Cannot Access Webmail
```bash
sudo docker compose ps
sudo docker compose restart
sudo docker compose logs nginx-mailcow
```

### Containers Not Starting
```bash
sudo systemctl status docker
free -h
df -h

cd /opt/mailcow-dockerized
sudo docker compose down
sudo docker compose up -d
sudo docker compose logs
```

### Cannot Send/Receive Email
```bash
sudo docker compose logs postfix
sudo docker compose logs dovecot

sudo ss -tulpn | grep :25
sudo ss -tulpn | grep :587
sudo ss -tulpn | grep :993

sudo docker compose restart postfix-mailcow
sudo docker compose restart dovecot-mailcow
```

### High RAM Usage
```bash
free -h
sudo docker stats

# Disable ClamAV if needed
sudo nano /opt/mailcow-dockerized/mailcow.conf
# SKIP_CLAMD=y
sudo docker compose up -d
```

---

## Completion Checklist
```
✅ VM created (ID 120, MAIL01)
✅ Debian 12 installed
✅ Static IP: 10.10.40.20/24
✅ XFCE Desktop installed
✅ Network verified
✅ Docker installed
✅ Docker Compose installed
✅ Mailcow downloaded
✅ Mailcow configured
✅ All containers running
✅ Web UI accessible
✅ Admin password changed
✅ Domain added (healthclinic.local)
✅ Mailboxes created
✅ Webmail accessible
✅ Emails send/receive
✅ All services green
```

---

## Quick Reference

### Mailcow Management
```bash
cd /opt/mailcow-dockerized

# Start
sudo docker compose up -d

# Stop
sudo docker compose down

# Restart
sudo docker compose restart

# Status
sudo docker compose ps

# Logs
sudo docker compose logs -f

# Update
sudo ./update.sh
```

### Access URLs
```
Admin: https://10.10.40.20
Webmail: https://10.10.40.20/SOGo

Default Login:
  Username: admin
  Password: moohoo (change to Clinic@2025!)
```

### Email Accounts
```
admin@healthclinic.local - Administrator
doctor@healthclinic.local - Dr. Sarah Johnson
reception@healthclinic.local - Reception Desk
it@healthclinic.local - IT Support

All passwords: Clinic@2025!
```

### Service Management
```bash
# Docker
sudo systemctl status docker
sudo systemctl restart docker

# Check specific container
sudo docker compose logs postfix-mailcow
sudo docker compose logs dovecot-mailcow
sudo docker compose logs sogo-mailcow
```

### Network Testing
```bash
ip addr show ens18

ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
ping -c 3 10.10.40.30

# Check ports
sudo ss -tulpn | grep :25
sudo ss -tulpn | grep :587
sudo ss -tulpn | grep :993
sudo ss -tulpn | grep :443
```

### Useful Commands
```bash
# Mailcow version
cd /opt/mailcow-dockerized
cat VERSION

# Disk usage
df -h
sudo du -sh /var/lib/docker/

# RAM usage
free -h
sudo docker stats

# Backup
sudo ./helper-scripts/backup_and_restore.sh backup all

# View logs
sudo docker logs mailcowdockerized-postfix-mailcow-1
```

---

## File Structure
```
/opt/mailcow-dockerized/
├── mailcow.conf
├── docker-compose.yml
├── data/
│   ├── assets/
│   ├── conf/
│   └── web/
├── generate_config.sh
├── update.sh
└── helper-scripts/
```

---

## Email Ports
```
SMTP:
  25   - Incoming
  587  - Submission (auth)
  465  - SMTPS

IMAP:
  143  - IMAP
  993  - IMAPS (SSL)

POP3:
  110  - POP3
  995  - POP3S (SSL)

Web:
  80   - HTTP → HTTPS
  443  - HTTPS
```

---

## Screenshots for Documentation

1. VM creation
2. Debian installation
3. Network config
4. XFCE desktop
5. Docker version
6. Containers running
7. Web UI login
8. Dashboard
9. Domain config
10. Mailbox creation
11. Webmail interface
12. Test email
13. Service status
14. Container status

---

## Integration Notes

### With WEB01

**Send appointment confirmations:**
```python
# In Flask app
import smtplib
from email.mime.text import MIMEText

def send_confirmation(email, appointment):
    msg = MIMEText(f"Confirmed: {appointment.date}")
    msg['Subject'] = 'Appointment Confirmation'
    msg['From'] = 'appointments@healthclinic.local'
    msg['To'] = email
    
    with smtplib.SMTP('10.10.40.20', 587) as server:
        server.starttls()
        server.login('appointments@healthclinic.local', 'Clinic@2025!')
        server.send_message(msg)
```

### With ZABBIX01

**Configure monitoring alerts:**
- SMTP: 10.10.40.20
- Port: 587
- From: zabbix@healthclinic.local
- Auth: yes

---

## Important Notes

### For School Project

**This provides:**
- ✅ Working email server
- ✅ Multiple mailboxes
- ✅ Webmail interface
- ✅ Internal communication
- ✅ Good demonstration

**Limitations:**
- Self-signed SSL
- No external email
- No DNS MX records
- Internal use only
- Perfect for capstone! ✅

### For Production

**Would need:**
- DNS records (MX, SPF, DKIM, DMARC)
- Real SSL certificates
- Backup strategy
- Spam filtering
- Retention policies
- 2FA

---

## Next Steps

1. ✅ MAIL01 complete
2. 🔜 Configure ZABBIX01
3. 🔜 Create DC02 on ESXi
4. 🔜 Integration testing
5. 🔜 ACL configuration
6. 🔜 Test VMs

---

## Credits

**Project:** Healthcare Clinic IT Infrastructure  
**Institution:** M.I.T.T.  
**Student:** Paul Santos  
**Date:** December 2024

---

**END OF DOCUMENT - Copy from line 1 to here, paste to GitHub as MAIL01_Complete_Setup_Guide.md**
