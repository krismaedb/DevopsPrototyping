# ZABBIX MONITORING SETUP - Healthcare Clinic IT Infrastructure

**Project:** Healthcare Clinic IT Infrastructure Capstone  

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [ZABBIX01 Server Installation](#zabbix01-server-installation)
3. [Linux Servers (WEB01, MAIL01)](#linux-servers-web01-mail01)
4. [Windows Servers (DC01, DC02)](#windows-servers-dc01-dc02)
5. [Cisco Network Devices (R1, R2, SW1, SW2)](#cisco-network-devices-r1-r2-sw1-sw2)
6. [Verification & Monitoring](#verification--monitoring)
7. [Troubleshooting](#troubleshooting)
8. [Credentials](#credentials)

---

## Overview

### Infrastructure Summary

**Monitoring Server:**
- **ZABBIX01:** Ubuntu 22.04, Zabbix 7.4, PostgreSQL 14, Apache
- **IP:** 10.10.40.40/24
- **Purpose:** Centralized monitoring for all network devices and servers

**Monitored Devices (9 total):**

| Device | Role | IP Address | OS/Platform | Monitoring Method |
|--------|------|------------|-------------|-------------------|
| **WEB01** | Web Application Server | 10.10.40.30 | Ubuntu 24.04 | Zabbix Agent 7.0 |
| **MAIL01** | Email Server | 10.10.40.20 | Debian 12 | Zabbix Agent 7.0 |
| **DC01** | Primary Domain Controller | 10.10.40.10 | Windows Server 2022 | Zabbix Agent 2 |
| **DC02** | Secondary Domain Controller | 10.10.40.11 | Windows Server 2022 | Zabbix Agent 2 |
| **R1** | Primary Router | 10.10.40.2 | Cisco IOS | SNMP v2 |
| **R2** | Standby Router | 10.10.40.3 | Cisco IOS | SNMP v2 |
| **SW1** | Primary Switch | 10.10.40.5 | Cisco IOS | SNMP v2 |
| **SW2** | Secondary Switch | 10.10.40.6 | Cisco IOS | SNMP v2 |
| **Zabbix server** | Self-monitoring | 127.0.0.1 | Ubuntu 22.04 | Zabbix Agent |

**Network Details:**
- VLAN: 40 (IT/Servers)
- Gateway: 10.10.40.1 (HSRP VIP)
- DNS: 10.10.40.10, 10.10.40.11
- Domain: healthclinic.local

---

## ZABBIX01 Server Installation

### System Preparation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y vim wget curl net-tools htop

# Set timezone
timedatectl set-timezone America/Winnipeg

# Verify network
ip addr show
ping -c 4 10.10.40.1
ping -c 4 10.10.40.10
ping -c 4 8.8.8.8
```

### PostgreSQL Database Setup
```bash
# Install PostgreSQL 14
sudo apt install -y postgresql postgresql-contrib

# Start and enable service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database and user
sudo -u postgres psql <<EOF
DROP DATABASE IF EXISTS zabbix;
CREATE DATABASE zabbix ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8' TEMPLATE template0;
CREATE USER zabbixuser WITH PASSWORD 'g3company!@#';
ALTER DATABASE zabbix OWNER TO zabbixuser;
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbixuser;
\c zabbix
GRANT ALL ON SCHEMA public TO zabbixuser;
EOF

# Verify database created
sudo -u postgres psql -c "\l zabbix"
```

### Zabbix Server Installation
```bash
# Download Zabbix 7.4 repository
cd /tmp
wget https://repo.zabbix.com/zabbix/7.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.4-1+ubuntu22.04_all.deb

# Install repository
sudo dpkg -i zabbix-release_7.4-1+ubuntu22.04_all.deb

# Update package list
sudo apt update

# Install Zabbix components
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Verify installation
dpkg -l | grep zabbix
```

### Database Schema Import
```bash
# Download Zabbix source for SQL files
cd /tmp
wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.5.tar.gz
tar -xzf zabbix-7.4.5.tar.gz
cd zabbix-7.4.5/database/postgresql/

# Import schema as zabbixuser
psql -h localhost -U zabbixuser -d zabbix -W < schema.sql
# Password: g3company!@#

psql -h localhost -U zabbixuser -d zabbix -W < images.sql
# Password: g3company!@#

psql -h localhost -U zabbixuser -d zabbix -W < data.sql
# Password: g3company!@#

# Verify tables created (should return 207)
sudo -u postgres psql -d zabbix -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public';"

# Check dbversion
sudo -u postgres psql -d zabbix -c "SELECT * FROM public.dbversion;"
```

### Database Permissions
```bash
# Grant permissions
sudo -u postgres psql -d zabbix <<EOF
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO zabbixuser;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;
EOF
```

### PostgreSQL Authentication
```bash
# Edit pg_hba.conf
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Change authentication methods to md5:
# local   all             all                                     md5
# host    all             all             127.0.0.1/32            md5
# host    all             all             ::1/128                 md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

### Zabbix Server Configuration
```bash
# Edit Zabbix server config
sudo nano /etc/zabbix/zabbix_server.conf

# Add/modify these lines:
# DBHost=localhost
# DBName=zabbix
# DBUser=zabbixuser
# DBPassword=g3company!@#

# Verify configuration
grep -E "^DB" /etc/zabbix/zabbix_server.conf

# Test configuration
zabbix_server -c /etc/zabbix/zabbix_server.conf -T
# Should output: "Validation successful"
```

### Apache Configuration
```bash
# Edit Apache config
sudo nano /etc/zabbix/apache.conf

# Verify timezone line exists:
# php_value date.timezone America/Winnipeg

# Enable Zabbix Apache config
sudo a2enconf zabbix

# Restart Apache
sudo systemctl restart apache2
sudo systemctl enable apache2
```

### Start Zabbix Services
```bash
# Start Zabbix server
sudo systemctl start zabbix-server
sudo systemctl enable zabbix-server

# Start Zabbix agent
sudo systemctl start zabbix-agent
sudo systemctl enable zabbix-agent

# Verify services
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
sudo systemctl status apache2

# Check ports
sudo ss -tunlp | grep 10051  # Zabbix server
sudo ss -tunlp | grep 10050  # Zabbix agent
sudo ss -tunlp | grep :80     # Apache
```

### Web Interface Setup

1. **Access URL:** http://10.10.40.40/zabbix

2. **Setup Wizard:**
   - Welcome → Click "Next step"
   - Prerequisites → Verify all green → Click "Next step"
   - Configure DB connection:
     - Database type: PostgreSQL
     - Database host: localhost
     - Database port: 5432
     - Database name: zabbix
     - User: zabbixuser
     - Password: g3company!@#
     - Click "Next step"
   - Zabbix server details:
     - Host: localhost
     - Port: 10051
     - Name: Healthcare Clinic Zabbix
     - Click "Next step"
   - Pre-installation summary → Review → Click "Next step"
   - Congratulations → Click "Finish"

3. **Login:**
   - Username: Admin
   - Password: zabbix

4. **Change Password:**
   - Administration → Users → Admin → Change password
   - New password: Clinic@2025!
   - Click "Update"

---

## Linux Servers (WEB01, MAIL01)

### WEB01 - Ubuntu 24.04 (10.10.40.30)
```bash
# SSH to WEB01
ssh webadmin@10.10.40.30

# Download Zabbix 7.0 repository
cd /tmp
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb

# Install repository
sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb

# Update and install agent
sudo apt update
sudo apt install -y zabbix-agent2

# Configure agent
sudo nano /etc/zabbix/zabbix_agent2.conf

# Set these values:
# Server=10.10.40.40
# ServerActive=10.10.40.40
# Hostname=WEB01

# Verify configuration
sudo grep -E "^(Server|ServerActive|Hostname)" /etc/zabbix/zabbix_agent2.conf

# Start and enable agent
sudo systemctl start zabbix-agent2
sudo systemctl enable zabbix-agent2

# Verify status
sudo systemctl status zabbix-agent2
sudo ss -tunlp | grep 10050

# Test from ZABBIX01
exit
zabbix_get -s 10.10.40.30 -k agent.ping
# Should return: 1
```

**Add in Zabbix Web UI:**
- Data collection → Hosts → Create host
- Host name: `WEB01`
- Visible name: `WEB01 - Web Application Server`
- Templates: `Linux by Zabbix agent`
- Host groups: `Linux servers`
- Interfaces: Agent, IP: 10.10.40.30, Port: 10050
- Click "Add"

---

### MAIL01 - Debian 12 (10.10.40.20)
```bash
# SSH to MAIL01
ssh root@10.10.40.20

# Download Zabbix 7.0 repository
cd /tmp
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb

# Install repository
dpkg -i zabbix-release_latest_7.0+debian12_all.deb

# Update and install agent
apt update
apt install -y zabbix-agent2

# Configure agent
nano /etc/zabbix/zabbix_agent2.conf

# Set these values:
# Server=10.10.40.40
# ServerActive=10.10.40.40
# Hostname=MAIL01

# Verify configuration
grep -E "^(Server|ServerActive|Hostname)" /etc/zabbix/zabbix_agent2.conf

# Start and enable agent
systemctl start zabbix-agent2
systemctl enable zabbix-agent2

# Verify status
systemctl status zabbix-agent2
ss -tunlp | grep 10050

# Test from ZABBIX01
exit
zabbix_get -s 10.10.40.20 -k agent.ping
# Should return: 1
```

**Add in Zabbix Web UI:**
- Data collection → Hosts → Create host
- Host name: `MAIL01`
- Visible name: `MAIL01 - Email Server`
- Templates: `Linux by Zabbix agent`
- Host groups: `Linux servers`
- Interfaces: Agent, IP: 10.10.40.20, Port: 10050
- Click "Add"

---

## Windows Servers (DC01, DC02)

### DC01 - Windows Server 2022 (10.10.40.10)

**Install Zabbix Agent:**

1. **Download Agent:**
   - URL: https://www.zabbix.com/download_agents
   - Download: Zabbix Agent 2 for Windows (64-bit) MSI

2. **Install Agent:**
   - Run: `zabbix_agent2-7.4.x-windows-amd64-openssl.msi`
   - Server: `10.10.40.40`
   - Hostname: `DC01`
   - Listen port: `10050`
   - Click "Install"

3. **Configure Firewall (PowerShell as Administrator):**
```powershell
# Allow Zabbix agent through firewall
New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allow

# Verify firewall rule
Get-NetFirewallRule -DisplayName "Zabbix Agent"

# Check service status
Get-Service "Zabbix Agent 2"
# Should show: Status = Running

# Restart service if needed
Restart-Service "Zabbix Agent 2"
```

**Add in Zabbix Web UI:**
- Data collection → Hosts → Create host
- Host name: `DC01`
- Visible name: `DC01 - Primary Domain Controller`
- Templates: `Windows by Zabbix agent`
- Host groups: `Windows servers`
- Interfaces: Agent, IP: 10.10.40.10, Port: 10050
- Click "Add"

**Test from ZABBIX01:**
```bash
zabbix_get -s 10.10.40.10 -k agent.ping
# Should return: 1
```

---

### DC02 - Windows Server 2022 (10.10.40.11)

**Repeat same steps as DC01:**

1. Install Zabbix Agent 2 MSI
   - Server: `10.10.40.40`
   - Hostname: `DC02`

2. Configure Firewall:
```powershell
New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allow
Get-Service "Zabbix Agent 2"
```

**Add in Zabbix Web UI:**
- Host name: `DC02`
- Visible name: `DC02 - Secondary Domain Controller`
- Templates: `Windows by Zabbix agent`
- Host groups: `Windows servers`
- Interfaces: Agent, IP: 10.10.40.11, Port: 10050

**Test:**
```bash
zabbix_get -s 10.10.40.11 -k agent.ping
```

---

## Cisco Network Devices (R1, R2, SW1, SW2)

### Enable SNMP on All Cisco Devices

**R1 - Primary Router (10.10.40.2):**
```bash
# SSH to R1
ssh admin@10.10.40.2
# Password: Clinic@2025!

enable
# Password: Clinic@2025!

configure terminal

# Enable SNMP
snmp-server community public RO
snmp-server location "MITT Healthcare Clinic - Primary Router"
snmp-server contact "psantos@healthclinic.local"

# Allow SNMP access from Zabbix only
access-list 50 permit 10.10.40.40
snmp-server community public RO 50

# Enable SNMP traps
snmp-server enable traps

# Save configuration
end
write memory

# Verify SNMP
show snmp community
```

**Add in Zabbix Web UI:**
- Data collection → Hosts → Create host
- Host name: `R1-Router`
- Visible name: `R1 - Primary Router`
- Templates: `Cisco IOS SNMP`
- Host groups: `Network devices`
- Interfaces: SNMP
  - IP address: 10.10.40.2
  - Port: 161
  - SNMP version: SNMPv2
  - SNMP community: `{$SNMP_COMMUNITY}`
- Macros tab:
  - Macro: `{$SNMP_COMMUNITY}`
  - Value: `public`
- Click "Add"

---

**R2 - Standby Router (10.10.40.3):**
```bash
# SSH to R2
ssh admin@10.10.40.3

enable
configure terminal

snmp-server community public RO
snmp-server location "MITT Healthcare Clinic - Standby Router"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
write memory
```

**Add in Zabbix Web UI:**
- Host name: `R2-Router`
- Visible name: `R2 - Standby Router`
- Templates: `Cisco IOS SNMP`
- Host groups: `Network devices`
- Interfaces: SNMP, IP: 10.10.40.3, Port: 161, Community: `public`

---

**SW1 - Primary Switch (10.10.40.5):**
```bash
# SSH to SW1
ssh admin@10.10.40.5

enable
configure terminal

snmp-server community public RO
snmp-server location "MITT Healthcare Clinic - Primary Switch"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
write memory
```

**Add in Zabbix Web UI:**
- Host name: `SW1-Switch`
- Visible name: `SW1 - Primary Switch`
- Templates: `Cisco IOS SNMP`
- Host groups: `Network devices`
- Interfaces: SNMP, IP: 10.10.40.5, Port: 161, Community: `public`

---

**SW2 - Secondary Switch (10.10.40.6):**
```bash
# SSH to SW2
ssh admin@10.10.40.6

enable
configure terminal

snmp-server community public RO
snmp-server location "MITT Healthcare Clinic - Secondary Switch"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
write memory
```

**Add in Zabbix Web UI:**
- Host name: `SW2-Switch`
- Visible name: `SW2 - Secondary Switch`
- Templates: `Cisco IOS SNMP`
- Host groups: `Network devices`
- Interfaces: SNMP, IP: 10.10.40.6, Port: 161, Community: `public`

---

## Verification & Monitoring

### Check All Hosts Status

**In Zabbix Web UI:**

1. **Go to:** Monitoring → Hosts

2. **Verify all 9 devices show:**

| Host | IP | Availability | Status |
|------|----|--------------|----|
| DC01 | 10.10.40.10 | 🟢 ZBX | Enabled |
| DC02 | 10.10.40.11 | 🟢 ZBX | Enabled |
| MAIL01 | 10.10.40.20 | 🟢 ZBX | Enabled |
| WEB01 | 10.10.40.30 | 🟢 ZBX | Enabled |
| R1-Router | 10.10.40.2 | 🟢 SNMP | Enabled |
| R2-Router | 10.10.40.3 | 🟢 SNMP | Enabled |
| SW1-Switch | 10.10.40.5 | 🟢 SNMP | Enabled |
| SW2-Switch | 10.10.40.6 | 🟢 SNMP | Enabled |
| Zabbix server | 127.0.0.1 | 🟢 ZBX | Enabled |

3. **View Latest Data:**
   - Click "Latest data" next to any host
   - Should see real-time metrics (CPU, memory, disk, network)

4. **View Graphs:**
   - Click "Graphs" next to any host
   - Should see pre-built graphs

5. **Check Problems:**
   - Go to: Monitoring → Problems
   - Should show any triggered alerts

### Command Line Verification
```bash
# On ZABBIX01

# Check Zabbix server status
sudo systemctl status zabbix-server

# Check server logs
sudo tail -50 /var/log/zabbix/zabbix_server.log

# Test all agents
zabbix_get -s 10.10.40.30 -k agent.ping  # WEB01
zabbix_get -s 10.10.40.20 -k agent.ping  # MAIL01
zabbix_get -s 10.10.40.10 -k agent.ping  # DC01
zabbix_get -s 10.10.40.11 -k agent.ping  # DC02

# Test SNMP devices
snmpwalk -v2c -c public 10.10.40.2 sysDescr  # R1
snmpwalk -v2c -c public 10.10.40.3 sysDescr  # R2
snmpwalk -v2c -c public 10.10.40.5 sysDescr  # SW1
snmpwalk -v2c -c public 10.10.40.6 sysDescr  # SW2

# Check ports
sudo ss -tunlp | grep 10051  # Zabbix server
sudo ss -tunlp | grep 10050  # Zabbix agent
```

### Create Dashboard (Optional)

1. **Go to:** Monitoring → Dashboard → Create dashboard
2. **Name:** Healthcare Clinic Overview
3. **Add widgets:**
   - System information
   - Problems (current problems)
   - Host availability
   - Graphs (CPU, memory, network traffic)
4. **Save dashboard**

---

## Troubleshooting

### Zabbix Server Issues

**Server not running:**
```bash
# Check status
sudo systemctl status zabbix-server

# View logs
sudo tail -100 /var/log/zabbix/zabbix_server.log
sudo journalctl -xeu zabbix-server.service -n 50

# Restart server
sudo systemctl restart zabbix-server

# Check configuration
zabbix_server -c /etc/zabbix/zabbix_server.conf -T
```

**Database connection failed:**
```bash
# Test database connection
psql -h localhost -U zabbixuser -d zabbix -W
# Password: g3company!@#

# Check PostgreSQL is running
sudo systemctl status postgresql

# Check PostgreSQL logs
sudo tail -50 /var/log/postgresql/postgresql-14-main.log
```

### Agent Issues (Gray ZBX Icon)

**On monitored server:**
```bash
# Check agent status
sudo systemctl status zabbix-agent2

# View agent logs
sudo tail -50 /var/log/zabbix/zabbix_agent2.log

# Verify configuration
sudo grep -E "^(Server|ServerActive|Hostname)" /etc/zabbix/zabbix_agent2.conf

# Check port is listening
sudo ss -tunlp | grep 10050

# Restart agent
sudo systemctl restart zabbix-agent2
```

**From ZABBIX01:**
```bash
# Test connectivity
zabbix_get -s <IP> -k agent.ping
zabbix_get -s <IP> -k agent.version
zabbix_get -s <IP> -k system.hostname

# Check network connectivity
ping <IP>
telnet <IP> 10050
```

**Common Issues:**
- **Hostname mismatch:** Agent hostname must match Zabbix host name exactly
- **Firewall blocking:** Allow port 10050 on monitored server
- **Wrong Server IP:** Check `Server=10.10.40.40` in agent config
- **Service not running:** `sudo systemctl start zabbix-agent2`

### SNMP Issues (Gray SNMP Icon)

**On Cisco device:**
```bash
# Verify SNMP configuration
show snmp community
show access-lists 50

# Test SNMP locally
show snmp
```

**From ZABBIX01:**
```bash
# Install snmp tools
sudo apt install -y snmp snmp-mibs-downloader

# Test SNMP connectivity
snmpwalk -v2c -c public <IP> sysDescr
snmpwalk -v2c -c public <IP> sysUpTime

# Check if device responds
ping <IP>
```

**Common Issues:**
- **Wrong community string:** Should be `public`
- **ACL blocking:** Verify `access-list 50 permit 10.10.40.40`
- **SNMP not enabled:** Run `snmp-server community public RO`

### Web UI Issues

**Cannot access http://10.10.40.40/zabbix:**
```bash
# Check Apache is running
sudo systemctl status apache2

# Check port 80 is listening
sudo ss -tunlp | grep :80

# Check Zabbix Apache config
sudo nano /etc/apache2/conf-enabled/zabbix.conf

# Restart Apache
sudo systemctl restart apache2

# Check Apache logs
sudo tail -50 /var/log/apache2/error.log
```

**"Zabbix server is not running" warning:**
```bash
# Check Zabbix server status
sudo systemctl status zabbix-server

# Check if port 10051 is listening
sudo ss -tunlp | grep 10051

# Restart Zabbix server
sudo systemctl restart zabbix-server

# Wait 30 seconds and refresh browser
```

---

## Credentials

### PostgreSQL Database

| Item | Value |
|------|-------|
| Superuser | postgres |
| Superuser Password | TaylorSwift13 |
| Database Name | zabbix |
| Database Owner | zabbixuser |
| Zabbix User | zabbixuser |
| Zabbix Password | g3company!@# |

### Zabbix Web Interface

| Item | Value |
|------|-------|
| URL | http://10.10.40.40/zabbix |
| Username | Admin |
| Password | Clinic@2025! |

### Monitored Servers

| Server | Username | Password | Notes |
|--------|----------|----------|-------|
| ZABBIX01 | zadmin | Clinic@2025! | SSH user |
| WEB01 | webadmin | (varies) | SSH user |
| MAIL01 | root | Clinic@2025! | SSH user |
| DC01 | Administrator | Clinic@2025! | Windows admin |
| DC02 | Administrator | Clinic@2025! | Windows admin |

### Cisco Devices

| Device | Username | Password | Enable Password |
|--------|----------|----------|-----------------|
| R1, R2, SW1, SW2 | admin | Clinic@2025! | Clinic@2025! |

### SNMP

| Parameter | Value |
|-----------|-------|
| Community String | public |
| Version | SNMPv2 |
| Port | 161 |

---

## Summary

**Monitoring Infrastructure:**
- ✅ ZABBIX01 server: Ubuntu 22.04, Zabbix 7.4, PostgreSQL 14, Apache
- ✅ Total monitored devices: 9
  - 2 Linux servers (WEB01, MAIL01)
  - 2 Windows servers (DC01, DC02)
  - 4 Cisco devices (R1, R2, SW1, SW2)
  - 1 Zabbix server (self-monitoring)

**Monitoring Methods:**
- Zabbix Agent: 5 devices (WEB01, MAIL01, DC01, DC02, Zabbix server)
- SNMP: 4 devices (R1, R2, SW1, SW2)

**Key Metrics Monitored:**
- System: CPU, memory, disk space, uptime
- Network: Interface traffic, packet loss, bandwidth
- Services: Process status, service availability
- Hardware: Temperature, power supply status (where supported)

**Access:**
- Web Interface: http://10.10.40.40/zabbix
- Monitoring → Hosts: View all devices
- Monitoring → Latest data: Real-time metrics
- Monitoring → Problems: Active alerts
