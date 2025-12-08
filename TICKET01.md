# 📋 TICKET01 Complete Setup Guide - Healthcare Clinic Project
**Complete step-by-step guide for TICKET01 (Ticketing System with UVDesk using PostgreSQL)**  
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
- [Part 6: PostgreSQL Installation](#part-6-postgresql-installation)
- [Part 7: UVDesk Installation](#part-7-uvdesk-installation)
- [Part 8: UVDesk Configuration](#part-8-uvdesk-configuration)
- [Part 9: Integration with Mailcow](#part-9-integration-with-mailcow)
- [Part 10: Testing & Verification](#part-10-testing--verification)
- [Part 11: Troubleshooting](#part-11-troubleshooting)
- [Completion Checklist](#completion-checklist)
- [Quick Reference](#quick-reference)

---

## Project Overview
**TICKET01** is a ticketing system running UVDesk - an open-source helpdesk platform for managing IT support tickets, customer inquiries, and clinic staff requests. It integrates with your Mailcow email server for automatic ticket creation from emails.

### Technology Stack
- **Operating System:** Debian 12 (Bookworm)
- **Desktop Environment:** XFCE Desktop (lightweight)
- **Database:** PostgreSQL 15 (as specified)
- **Web Server:** Apache 2.4
- **PHP Framework:** Symfony (UVDesk backend)
- **Services:** Helpdesk portal, email-to-ticket piping, knowledge base, SLAs, agent dashboards

### Network Details
- **VM Name:** TICKET01
- **IP Address:** 10.10.40.50/24
- **Gateway:** 10.10.40.1 (HSRP VIP on R1/R2)
- **DNS Servers:** 10.10.40.10 (DC01), 10.10.40.11 (DC02)
- **VLAN:** 40 (IT/Servers)
- **Domain:** healthclinic.local
- **Ticketing URL:** http://ticket.healthclinic.local (or http://10.10.40.50/helpdesk)
- **Admin Email:** admin@healthclinic.local

---

## VM Specifications
```yaml
Proxmox Configuration:
  VM ID: 105
  Name: TICKET01
  Node: pve-main (10.10.40.15)

Hardware:
  CPU: 2-4 cores (host type)
  RAM: 4096 MB (4 GB)
  Disk: 40 GB (local-lvm storage)
  Network: VirtIO on vmbr0 (VLAN 40)

Operating System:
  Distribution: Debian 12 (Bookworm)
  Architecture: x86_64 (amd64)
  Kernel: 6.1.x
  Desktop: XFCE (lightweight)

Network Configuration:
  IP: 10.10.40.50/24 (static)
  Gateway: 10.10.40.1
  DNS: 10.10.40.10, 10.10.40.11
  Hostname: ticket01.healthclinic.local
```

---

## Part 1: VM Creation in Proxmox
### Step 1.1: Access Proxmox Web Interface
```
URL: https://10.10.40.15:8006
Username: root
Password: Clinic@2025!
```

### Step 1.2: Upload Debian ISO (if not already done)
If you haven't uploaded the ISO yet (from MAIL01 setup):
```bash
# Via Proxmox console:
cd /var/lib/vz/template/iso/
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso
ls -lh debian-12.8.0-amd64-netinst.iso
```
Or via Web UI:
1. Click **pve-main** → **local** → **ISO Images**
2. Click **Download from URL**
3. URL: `https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso`
4. Click **Query URL** → **Download**

### Step 1.3: Create New VM
Click **Create VM** button

#### Tab 1: General
```
Node: pve-main
VM ID: 105
Name: TICKET01
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
Disk size (GiB): 40
Cache: Default
☑ Discard
☑ IO thread
```
Click **Next**

#### Tab 5: CPU
```
Sockets: 1
Cores: 2 (or 4 if available)
Type: host
```
Click **Next**

#### Tab 6: Memory
```
Memory (MiB): 4096
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
1. Click **TICKET01 (105)**
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
Hostname: ticket01
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
Full name: Ticket Admin
Username: ticketadmin
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
Select disk: SCSI (0,0,0) (sda) - 42.9 GB
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
ticket01 login: root
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
    address 10.10.40.50/24
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
# Should show: 10.10.40.50/24
# Test connectivity
ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
ping -c 3 8.8.8.8
ping -c 3 10.10.40.20  # MAIL01 for integration
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
1. Click **ticketadmin**  
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
    lsb-release \
    unzip \
    apache2 \
    php \
    php-curl \
    php-gd \
    php-intl \
    php-mbstring \
    php-xml \
    php-zip \
    php-bcmath \
    php-imap \
    php-dev \
    php-pear \
    composer
```

### Step 5.2: Set Hostname
```bash
sudo hostnamectl set-hostname ticket01.healthclinic.local
hostname -f
# Should show: ticket01.healthclinic.local
```

### Step 5.3: Edit /etc/hosts
```bash
sudo nano /etc/hosts
```
**Add:**
```
127.0.0.1 localhost
10.10.40.50 ticket01.healthclinic.local ticket01
::1 localhost ip6-localhost ip6-loopback
```
**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 5.4: Disable IPv6 (optional, for simplicity)
```bash
sudo nano /etc/sysctl.conf
```
**Add at end:**
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`
```bash
sudo sysctl -p
```

---

## Part 6: PostgreSQL Installation
### Step 6.1: Install PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql  # Should be active
```

### Step 6.2: Secure and Create Database
```bash
sudo -u postgres psql
```
In psql shell:
```sql
ALTER USER postgres PASSWORD 'Clinic@2025!';
CREATE USER uvdesk WITH PASSWORD 'Clinic@2025!';
CREATE DATABASE uvdesk OWNER uvdesk;
GRANT ALL PRIVILEGES ON DATABASE uvdesk TO uvdesk;
\q
```

### Step 6.3: Verify Postgres
```bash
sudo -u postgres psql -c "\l"  # List databases, should see uvdesk
```

---

## Part 7: UVDesk Installation
### Step 7.1: Install PHP Extensions for UVDesk
```bash
sudo apt install php-pgsql php-mailparse -y  # For Postgres and email parsing
sudo systemctl restart apache2
```

### Step 7.2: Download UVDesk
```bash
cd /var/www/html
sudo composer create-project uvdesk/community-skeleton helpdesk
cd helpdesk
sudo composer require doctrine/dbal  # Ensure DBAL for Postgres
```

### Step 7.3: Configure Environment
```bash
sudo cp .env.local.example .env.local
sudo nano .env.local
```
**Update database URL:**
```
DATABASE_URL="pgsql://uvdesk:Clinic@2025!@localhost:5432/uvdesk?serverVersion=15&charset=utf8"
```
**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.4: Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html/helpdesk
sudo chmod -R 755 /var/www/html/helpdesk
```

### Step 7.5: Install Dependencies and Setup
```bash
sudo composer install --no-dev --optimize-autoloader
sudo php bin/console uvdesk:configure
sudo php bin/console doctrine:database:create
sudo php bin/console doctrine:schema:create
sudo php bin/console doctrine:migrations:migrate
sudo php bin/console uvdesk:refresh-assets
sudo php bin/console cache:clear
```

### Step 7.6: Apache Virtual Host
```bash
sudo nano /etc/apache2/sites-available/helpdesk.conf
```
**Paste:**
```
<VirtualHost *:80>
    ServerName ticket.healthclinic.local
    DocumentRoot /var/www/html/helpdesk/public

    <Directory /var/www/html/helpdesk/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/helpdesk_error.log
    CustomLog ${APACHE_LOG_DIR}/helpdesk_access.log combined
</VirtualHost>
```
```bash
sudo a2ensite helpdesk.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## Part 8: UVDesk Configuration
### Step 8.1: Access Web Installer
Open browser:  
http://10.10.40.50/helpdesk/public  
(or http://ticket.healthclinic.local if DNS added)  

Follow the installer:
- Database: Already set in .env.local
- Admin User: admin@healthclinic.local / Clinic@2025!
- Complete setup → Login

### Step 8.2: Add Agents/Users
1. Login as admin
2. **Agents** → Add Agent  
   - doctor@healthclinic.local (Doctor)  
   - reception@healthclinic.local (Reception)  
   - it@healthclinic.local (IT Support)  
3. **Groups** → Create "Clinic IT" and assign agents

### Step 8.3: Configure Basic Settings
- **Settings** → **General** → Set site name: "Health Clinic Helpdesk"  
- Add your logo (from earlier)  
- **Workflows** → Set up SLAs (e.g., P1: 1 hour response)

---

## Part 9: Integration with Mailcow
### Step 9.1: Create Support Mailbox in Mailcow
In Mailcow Admin UI:  
1. **Mailboxes** → Add Mailbox  
   ```
   Username: support
   Domain: healthclinic.local
   Password: Clinic@2025!
   ```
2. Note IMAP settings: Server 10.10.40.20, Port 993 (SSL)

### Step 9.2: Configure Email Piping in UVDesk
In UVDesk Admin:  
1. **Channels** → **Email** → Add Mailbox  
   ```
   Name: Clinic Support
   Email: support@healthclinic.local
   IMAP Server: 10.10.40.20
   Port: 993
   Username: support@healthclinic.local
   Password: Clinic@2025!
   SSL: Yes
   ```
2. **Workflows** → Auto-create tickets from emails

### Step 9.3: Test Integration
Send email from doctor@healthclinic.local to support@healthclinic.local → Auto-creates ticket in UVDesk

---

## Part 10: Testing & Verification
### Step 10.1: Access Helpdesk
Browser: http://10.10.40.50/helpdesk/public  
Login as admin@healthclinic.local

### Step 10.2: Create Test Ticket
1. **New Ticket** → Submit as reception  
2. Assign to it@healthclinic.local  
3. Reply and close

### Step 10.3: Check Email Piping
Send test email to support@healthclinic.local → New ticket appears

### Step 10.4: Check Resources
```bash
free -h  # RAM usage
htop     # CPU/Processes
sudo systemctl status apache2 postgresql
```

### Step 10.5: External Access
From another VM (e.g., WEB01): Open http://10.10.40.50/helpdesk/public → Works

---

## Part 11: Troubleshooting
### Cannot Access Portal
```bash
sudo tail -f /var/log/apache2/helpdesk_error.log
sudo systemctl restart apache2
```

### Database Connection Issues
```bash
sudo -u postgres psql -d uvdesk -c "\dt"  # List tables
sudo nano /var/www/html/helpdesk/.env.local  # Check DATABASE_URL
sudo php bin/console doctrine:schema:validate
```

### Email Piping Fails
```bash
sudo php bin/console uvdesk:mailbox:check  # Test mailbox
Check Mailcow logs: sudo docker compose logs dovecot-mailcow
```

### High Resource Usage
```bash
free -h
sudo docker stats  # If any (though not used here)
Optimize: sudo php bin/console cache:clear
```

---

## Completion Checklist
```
✅ VM created (ID 105, TICKET01)
✅ Debian 12 installed
✅ Static IP: 10.10.40.50/24
✅ XFCE Desktop installed
✅ Network verified
✅ PostgreSQL installed and database created
✅ UVDesk downloaded and installed
✅ UVDesk configured via web installer
✅ Agents and groups added
✅ Integrated with Mailcow (email piping)
✅ Portal accessible
✅ Tickets create/send/reply/close
✅ All services running
```

---

## Quick Reference
### UVDesk Management
```bash
cd /var/www/html/helpdesk
# Clear cache
sudo php bin/console cache:clear
# Refresh assets
sudo php bin/console uvdesk:refresh-assets
# Check mailbox
sudo php bin/console uvdesk:mailbox:check
```

### Access URLs
```
Admin/Agent Portal: http://10.10.40.50/helpdesk/public/member
User Portal: http://10.10.40.50/helpdesk/public
Default Login:
  Username: admin@healthclinic.local
  Password: Clinic@2025!
```

### Service Management
```bash
# Apache
sudo systemctl status apache2
sudo systemctl restart apache2
# PostgreSQL
sudo systemctl status postgresql
sudo systemctl restart postgresql
# Logs
sudo tail -f /var/log/apache2/helpdesk_error.log
sudo -u postgres psql -d uvdesk -c "SELECT * FROM uv_ticket LIMIT 5;"
```

### Network Testing
```bash
ip addr show ens18
ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
ping -c 3 10.10.40.20  # MAIL01
# Check ports
sudo ss -tulpn | grep :80
sudo ss -tulpn | grep :5432  # Postgres
```

### Useful Commands
```bash
# UVDesk version
cd /var/www/html/helpdesk
cat composer.json | grep uvdesk
# Disk usage
df -h
sudo du -sh /var/www/html/helpdesk
# RAM usage
free -h
htop
# Backup (manual)
sudo tar -czf uvdesk_backup.tar.gz /var/www/html/helpdesk
sudo pg_dump uvdesk > uvdesk_db_backup.sql
```

---

## File Structure
```
/var/www/html/helpdesk/
├── .env.local
├── public/  # Web root
├── bin/     # Console commands
├── src/     # Symfony source
├── var/     # Cache/logs
└── vendor/  # Dependencies
```

---

## Ticketing Ports
```
HTTP: 80 (Apache)
Postgres: 5432
```

---

## Screenshots for Documentation
1. VM creation
2. Debian installation
3. Network config
4. XFCE desktop
5. Postgres status
6. UVDesk installer
7. Admin dashboard
8. Agents list
9. New ticket creation
10. Email piping test
11. SLA configuration
12. Service status

---

## Integration Notes
### With WEB01
Embed helpdesk link in Flask app:
```python
# In Flask template
<a href="http://10.10.40.50/helpdesk/public">Submit Ticket</a>
```

### With ZABBIX01
Configure alerts to email support@healthclinic.local → auto-ticket in UVDesk

---

## Important Notes
### For School Project
**This provides:**
- ✅ Modern helpdesk with portals
- ✅ Email-to-ticket automation
- ✅ Agent workflows
- ✅ Internal ticketing for clinic
- ✅ Good demo with Mailcow integration

**Limitations:**
- HTTP only (add HTTPS for production)
- No external access
- Basic SLAs (expand as needed)
- Perfect for capstone! ✅

### For Production
**Would need:**
- HTTPS/SSL
- Backups
- Custom branding
- Advanced workflows
- 2FA

---

## Next Steps
1. ✅ TICKET01 complete
2. 🔜 Configure ZABBIX01
3. 🔜 Integration testing (e.g., email from Mailcow to UVDesk)
4. 🔜 ACLs for VLAN access
5. 🔜 Test from client VMs

---

## Credits
**Project:** Healthcare Clinic IT Infrastructure
**Institution:** M.I.T.T.
**Student:** Paul Santos
**Date:** December 2025

---
**END OF DOCUMENT - Copy from line 1 to here, paste to GitHub as TICKET01_Complete_Setup_Guide.md**
