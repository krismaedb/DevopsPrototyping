# Tailscale VPN Server - Documentation

**Project:** Healthcare Clinic IT Infrastructure - VPN Remote Access  
**Server:** VPN (10.10.40.60)  
**Platform:** Ubuntu 24.04 LTS (VM on Proxmox)  
**VPN Solution:** Tailscale (Free Plan)  
**Institution:** Manitoba Institute of Trades and Technology (M.I.T.T.)  
**Author:** Group 3

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Server Installation](#server-installation)
5. [Subnet Routing Configuration](#subnet-routing-configuration)
6. [Client Setup](#client-setup)
7. [Team Member Access](#team-member-access)
8. [Security Configuration](#security-configuration)
9. [Testing & Verification](#testing--verification)
10. [Troubleshooting](#troubleshooting)
11. [Management & Monitoring](#management--monitoring)

---

## 1. Overview

### 1.1 Purpose

The Tailscale VPN server provides **secure remote access** to the Healthcare Clinic IT infrastructure without exposing internal servers to the public internet.

**Key Capabilities:**
- Access Proxmox Web UI remotely
- Manage all internal servers (DC01, DC02, WEB01, MAIL01, ZABBIX01, TrueNAS)
- Collaborate with team members securely
- Zero public IP exposure
- No port forwarding required
- Encrypted end-to-end connections

### 1.2 Why Tailscale?

| Feature | Benefit |
|---------|---------|
| **WireGuard-based** | Modern, fast, secure VPN protocol |
| **Zero-configuration** | No complex routing or firewall rules |
| **Free for small teams** | Up to 100 devices, unlimited users |
| **Cross-platform** | Windows, macOS, Linux, iOS, Android |
| **Mesh networking** | Direct peer-to-peer connections |
| **No public IP needed** | Works behind NAT/firewalls |
| **Easy device management** | Web-based admin console |

### 1.3 Use Cases

✅ **Remote Lab Access** - Access Proxmox and VMs from home  
✅ **Team Collaboration** - Multiple students access shared lab  
✅ **Secure Management** - Manage servers without exposing SSH/RDP  
✅ **File Access** - Access TrueNAS shares remotely  
✅ **Monitoring** - Check Zabbix dashboards from anywhere  
✅ **Emergency Access** - Fix issues outside of campus

---

## 2. Architecture

### 2.1 Network Diagram
```
                        INTERNET
                            │
                            │ Encrypted WireGuard Tunnel
                            │
                ┌───────────┴────────────┐
                │   Tailscale Cloud      │
                │   (Coordination Server)│
                └───────────┬────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        │                   │                   │
    ┌───┴────┐         ┌────┴─────┐       ┌────┴─────┐
    │ Admin  │         │  Team    │       │  Team    │
    │ Laptop │         │ Member 1 │       │ Member 2 │
    │ (Home) │         │ (Home)   │       │ (Home)   │
    └────────┘         └──────────┘       └──────────┘
   100.64.1.1          100.64.1.2         100.64.1.3
        │                   │                   │
        │    Tailscale Network (100.64.0.0/10) │
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────┴────────┐
                    │   VPN SERVER   │
                    │   10.10.40.60  │
                    │ (ubuntu-vpn VM)│
                    │                │
                    │ Subnet Router: │
                    │ 10.10.0.0/16   │
                    └───────┬────────┘
                            │
                            │ VLAN 40 - IT/Servers
                            │
            ┌───────────────┴───────────────────────────────┐
            │        Healthcare Clinic Network              │
            │           10.10.0.0/16                        │
            │                                               │
    ┌───────┴────┐  ┌──────┴──────┐  ┌──────┴──────┐     │
    │   DC01     │  │   DC02      │  │   WEB01     │     │
    │ .40.10     │  │   .40.11    │  │   .40.30    │     │
    └────────────┘  └─────────────┘  └─────────────┘     │
                                                           │
    ┌────────────┐  ┌─────────────┐  ┌─────────────┐     │
    │   MAIL01   │  │  ZABBIX01   │  │  TRUENAS    │     │
    │ .40.20     │  │   .40.40    │  │   .40.50    │     │
    └────────────┘  └─────────────┘  └─────────────┘     │
                                                           │
    └───────────────────────────────────────────────────────┘
```

### 2.2 IP Address Scheme

| Device/Network | IP Address | Purpose |
|----------------|------------|---------|
| **VPN Server (ubuntu-vpn)** | 10.10.40.60 | Internal LAN IP |
| **VPN Server (Tailscale)** | 100.64.1.1 | Tailscale mesh IP |
| **Advertised Subnet** | 10.10.0.0/16 | All internal VLANs |
| **VLAN 10** | 10.10.10.0/24 | Patient WiFi |
| **VLAN 20** | 10.10.20.0/24 | Clinical Staff |
| **VLAN 30** | 10.10.30.0/24 | Administration |
| **VLAN 40** | 10.10.40.0/24 | IT/Servers |
| **Remote Clients** | 100.64.x.x | Dynamic Tailscale IPs |

### 2.3 Traffic Flow

**Before Tailscale:**
```
Remote User → ❌ NO ACCESS → Internal Network
```

**After Tailscale:**
```
Remote User (100.64.1.2) → Tailscale Cloud (encrypted) 
    → VPN Server (100.64.1.1 / 10.10.40.60) 
    → Internal Server (10.10.40.30) ✅
```

---

## 3. Prerequisites

### 3.1 Infrastructure Requirements

✅ **Proxmox VE** - Running and accessible  
✅ **Network Access** - VM can reach internet  
✅ **Internal Network** - VM on same VLAN as servers  
✅ **DNS Resolution** - VM can resolve domain names  
✅ **Firewall Rules** - Allow Tailscale UDP 41641

### 3.2 Accounts Required

✅ **Tailscale Account** - Free account at https://login.tailscale.com  
✅ **Admin Access** - Root/sudo on VPN server  
✅ **Network Access** - Permission to configure routing

### 3.3 VM Specifications

| Component | Specification |
|-----------|---------------|
| **Hostname** | vpn.healthclinic.local |
| **OS** | Ubuntu 24.04 LTS Server |
| **CPU** | 2 cores |
| **RAM** | 2 GB |
| **Storage** | 20 GB |
| **Network** | Bridged to VLAN 40 |
| **IP Address** | 10.10.40.60/24 (static) |
| **Gateway** | 10.10.40.1 (HSRP VIP) |
| **DNS** | 10.10.40.10, 10.10.40.11 |

---

## 4. Server Installation

### 4.1 Create VM in Proxmox
```bash
# On Proxmox host via SSH or Web UI:

# Create VM using Ubuntu 24.04 ISO
qm create 160 --name vpn --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0
qm importdisk 160 /var/lib/vz/template/iso/ubuntu-24.04-server-amd64.iso local-lvm
qm set 160 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-160-disk-0
qm set 160 --boot order=scsi0
qm set 160 --ide2 local-lvm:cloudinit
qm set 160 --serial0 socket --vga serial0

# Start VM
qm start 160
```

**Or via Proxmox Web UI:**
1. Right-click node → Create VM
2. General: VM ID = 160, Name = vpn
3. OS: Select Ubuntu 24.04 ISO
4. System: Default (UEFI if available)
5. Disks: 20 GB
6. CPU: 2 cores
7. Memory: 2048 MB
8. Network: Bridge to vmbr0 (VLAN 40)
9. Finish

### 4.2 Install Ubuntu Server
```bash
# Boot VM and follow Ubuntu installer:
# - Language: English
# - Keyboard: English (US)
# - Network: Configure manually

# Network Configuration:
IPv4: 10.10.40.60/24
Gateway: 10.10.40.1
DNS: 10.10.40.10, 10.10.40.11
Search domain: healthclinic.local

# Storage: Use entire disk (20 GB)
# Profile:
Name: VPN Admin
Server name: vpn
Username: vp nadmin
Password: g3company!@#

# SSH: Install OpenSSH server ✅
# Featured Server Snaps: None
# Complete installation and reboot
```

### 4.3 Initial System Configuration
```bash
# SSH into the new VM
ssh vp nadmin@10.10.40.60

# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget vim htop net-tools ufw

# Set timezone
sudo timedatectl set-timezone America/Winnipeg

# Verify network
ip addr show
ping -c 4 10.10.40.1      # Gateway
ping -c 4 10.10.40.10     # DC01
ping -c 4 8.8.8.8         # Internet
ping -c 4 google.com      # DNS resolution

# Set hostname
sudo hostnamectl set-hostname vpn.healthclinic.local

# Update /etc/hosts
sudo nano /etc/hosts
# Add:
10.10.40.60    vpn.healthclinic.local vpn

# Reboot to apply all changes
sudo reboot
```

### 4.4 Configure Static IP (Netplan)
```bash
# Edit netplan configuration
sudo nano /etc/netplan/00-installer-config.yaml
```

**Configuration:**
```yaml
network:
  version: 2
  ethernets:
    ens18:  # Your interface name (check with 'ip a')
      addresses:
        - 10.10.40.60/24
      routes:
        - to: default
          via: 10.10.40.1
      nameservers:
        addresses:
          - 10.10.40.10
          - 10.10.40.11
        search:
          - healthclinic.local
```

**Apply configuration:**
```bash
# Test configuration (won't break network)
sudo netplan try

# If successful (within 120 seconds):
# Press Enter to accept

# Or apply directly
sudo netplan apply

# Verify
ip addr show ens18
ip route show
cat /etc/resolv.conf
```

---

## 5. Subnet Routing Configuration

### 5.1 Enable IP Forwarding
```bash
# Enable IPv4 forwarding permanently
sudo nano /etc/sysctl.conf
```

**Add or uncomment these lines:**
```conf
# Enable IP forwarding for Tailscale subnet routing
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

**Apply changes:**
```bash
sudo sysctl -p

# Verify
sysctl net.ipv4.ip_forward
# Should output: net.ipv4.ip_forward = 1

sysctl net.ipv6.conf.all.forwarding
# Should output: net.ipv6.conf.all.forwarding = 1
```

### 5.2 Install Tailscale
```bash
# Download and run Tailscale installer
curl -fsSL https://tailscale.com/install.sh | sh

# Verify installation
tailscale version
# Should output: 1.xx.x (latest version)

# Check service status
sudo systemctl status tailscaled
# Should be: active (running)
```

### 5.3 Configure Tailscale with Subnet Routes
```bash
# Start Tailscale and advertise internal subnets
sudo tailscale up \
  --advertise-routes=10.10.0.0/16 \
  --accept-routes \
  --hostname=vpn-healthclinic

# A URL will appear, copy and paste into browser:
# https://login.tailscale.com/a/XXXXXXXX

# Login with your Tailscale account:
# - Google
# - Microsoft
# - Email
```

**What this command does:**
- `--advertise-routes=10.10.0.0/16` - Share ALL clinic VLANs (10-40)
- `--accept-routes` - Accept routes from other Tailscale devices
- `--hostname=vpn-healthclinic` - Friendly name in Tailscale

### 5.4 Approve Subnet Routes in Tailscale Admin
```bash
# 1. Open browser and go to:
https://login.tailscale.com/admin/machines

# 2. Find your VPN server:
#    Name: vpn-healthclinic
#    IP: 100.64.x.x
#    Internal IP: 10.10.40.60

# 3. Click the three dots (•••) next to the device

# 4. Click "Edit route settings..."

# 5. You'll see:
#    Advertised routes: 10.10.0.0/16
#    Status: Pending approval

# 6. Toggle ON the subnet route:
#    ✅ 10.10.0.0/16

# 7. Click "Save"

# 8. Verify approval:
#    Status should change to: ✅ Approved
```

### 5.5 Verify Subnet Routing
```bash
# Check Tailscale status
sudo tailscale status

# Expected output:
# 100.64.1.1   vpn-healthclinic     vp nadmin@  linux   -
#              10.10.0.0/16         advertised

# Check advertised routes
sudo tailscale status --json | grep -A 5 "AdvertisedRoutes"

# Should show:
# "AdvertisedRoutes": [
#   "10.10.0.0/16"
# ]

# Check if routes are approved
sudo tailscale status --json | grep -A 5 "AllowedIPs"

# Verify IP forwarding is working
cat /proc/sys/net/ipv4/ip_forward
# Should output: 1
```

### 5.6 Disable Key Expiry (Important!)
```bash
# By default, Tailscale keys expire after 180 days
# Disable expiry for VPN server:

# 1. Go to: https://login.tailscale.com/admin/machines
# 2. Find: vpn-healthclinic
# 3. Click three dots (•••)
# 4. Click "Disable key expiry"
# 5. Confirm
```

---

## 6. Client Setup

### 6.1 Install Tailscale on Remote Devices

#### Windows Client
```powershell
# Download installer:
# https://tailscale.com/download/windows

# Run installer: tailscale-setup-xxx.exe

# After installation:
# 1. Tailscale icon appears in system tray
# 2. Click icon → "Log in"
# 3. Browser opens → Login with Tailscale account
# 4. Click "Connect"

# Verify connection:
# Open PowerShell:
tailscale status

# Should show:
# 100.64.1.1   vpn-healthclinic     ...
# 100.64.1.2   LAPTOP-XXXX          ...
```

#### macOS Client
```bash
# Download installer:
# https://tailscale.com/download/mac

# Install: Tailscale.app
# Open Tailscale from Applications
# Click "Log in"
# Authenticate in browser

# Verify from Terminal:
tailscale status
```

#### Linux Client
```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Connect
sudo tailscale up

# Verify
tailscale status
tailscale ip -4  # Show your Tailscale IP
```

#### Mobile Devices
```
# iOS:
App Store → Search "Tailscale" → Install

# Android:
Play Store → Search "Tailscale" → Install

# Login and connect
```

### 6.2 Test Connection to Internal Network
```bash
# From remote client, test connectivity:

# Ping VPN server
ping 10.10.40.60

# Should succeed! (if not, see troubleshooting)

# Ping domain controllers
ping 10.10.40.10
ping 10.10.40.11

# Ping web server
ping 10.10.40.30

# Access Proxmox Web UI
# Open browser:
https://10.10.40.100:8006

# Access WEB01
http://10.10.40.30

# Access Zabbix
http://10.10.40.40/zabbix

# Access TrueNAS
http://10.10.40.50
```

### 6.3 Access Windows Servers via RDP
```powershell
# From Windows client:
mstsc

# Computer: 10.10.40.10
# Username: HEALTHCLINIC\Administrator
# Password: g3company!@#

# Click "Connect"
# Should open RDP session to DC01! ✅
```

### 6.4 Access File Shares
```bash
# Windows:
# Open File Explorer
# Address bar: \\10.10.40.50\PatientFiles
# Enter credentials: HEALTHCLINIC\username

# Linux/macOS:
# Install CIFS utils first
sudo apt install cifs-utils  # Ubuntu
brew install samba           # macOS

# Mount share
sudo mount -t cifs //10.10.40.50/PatientFiles /mnt/clinic -o username=your_ad_user,domain=HEALTHCLINIC
```

---

## 7. Team Member Access

### 7.1 How Tailscale Sharing Works

**Traditional VPN Problem:**
- Only one account can use the VPN
- Adding users requires complex configuration

**Tailscale Solution:**
- Each person creates FREE Tailscale account
- Admin shares the VPN server device
- Members get access WITHOUT joining admin's network
- No limit on shared devices (free plan)

### 7.2 Team Member Setup (Step-by-Step)

#### Step 1: Team Member Creates Tailscale Account
```
1. Go to: https://login.tailscale.com
2. Click "Sign up"
3. Choose:
   - Google
   - Microsoft
   - Email
4. Complete registration
5. You now have your own empty Tailscale network
```

#### Step 2: Team Member Installs Tailscale
```bash
# Windows:
Download from: https://tailscale.com/download/windows
Install and login with YOUR OWN account

# Linux:
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# macOS:
Download from: https://tailscale.com/download/mac
Install and login
```

#### Step 3: Admin Shares VPN Server Device
```
Admin (Paul Santos) does this:

1. Go to: https://login.tailscale.com/admin/machines
2. Find: vpn-healthclinic (100.64.1.1)
3. Click three dots (•••)
4. Click "Share..."
5. Enter team member's email: kristine@example.com
6. Click "Share"
7. Email sent to Kristine ✅
```

#### Step 4: Team Member Accepts Share
```
Team member (Kristine) does this:

1. Check email for Tailscale invitation
2. Click "Accept" in email
3. OR go to: https://login.tailscale.com/admin/machines
4. Click "Shared with you" tab
5. See: vpn-healthclinic (pending)
6. Click "Accept"
7. Device appears in "Machines" list ✅
```

#### Step 5: Team Member Tests Access
```bash
# From team member's laptop:

# Check Tailscale status
tailscale status

# Should show shared device:
# 100.64.1.1   vpn-healthclinic     shared   linux   active

# Test ping
ping 10.10.40.60  # VPN server
ping 10.10.40.10  # DC01
ping 10.10.40.30  # WEB01

# Access web resources
# Browser: https://10.10.40.100:8006  (Proxmox)
# Browser: http://10.10.40.30         (Patient Portal)
```

### 7.3 Adding Multiple Team Members
```bash
# Repeat sharing process for each member:

1. Admin shares vpn-healthclinic with:
   - kristine@example.com ✅
   - john@example.com ✅
   - maria@example.com ✅

2. Each member:
   - Creates own Tailscale account
   - Installs Tailscale client
   - Accepts shared device
   - Gets full access to 10.10.0.0/16

# No limit on number of shared devices (free plan)
```

### 7.4 Revoking Access
```bash
# Admin can revoke access anytime:

1. Go to: https://login.tailscale.com/admin/machines
2. Find: vpn-healthclinic
3. Click three dots (•••)
4. Click "Manage sharing..."
5. Find user to remove
6. Click "X" next to their email
7. Confirm removal
8. User immediately loses access ✅
```

---

## 8. Security Configuration

### 8.1 Configure UFW Firewall
```bash
# Reset firewall to default
sudo ufw --force reset

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from internal network only
sudo ufw allow from 10.10.30.0/24 to any port 22 comment 'SSH from Admin VLAN'
sudo ufw allow from 10.10.40.0/24 to any port 22 comment 'SSH from Server VLAN'

# Allow Tailscale
sudo ufw allow 41641/udp comment 'Tailscale'

# Allow forwarding from Tailscale to internal network
sudo ufw route allow in on tailscale0 out on ens18

# Enable firewall
sudo ufw enable

# Verify rules
sudo ufw status verbose
```

**Expected output:**
```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       10.10.30.0/24              # SSH from Admin VLAN
22                         ALLOW       10.10.40.0/24              # SSH from Server VLAN
41641/udp                  ALLOW       Anywhere                   # Tailscale

Anywhere on ens18          ALLOW FWD   Anywhere on tailscale0
```

### 8.2 Tailscale ACLs (Access Control Lists)
```bash
# Configure granular access control:

1. Go to: https://login.tailscale.com/admin/acls
2. Click "Edit ACLs"
3. Add policy:
```

**Example ACL Configuration:**
```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": [
        "10.10.40.60:*",
        "10.10.40.10:3389",
        "10.10.40.11:3389",
        "10.10.40.30:80,443",
        "10.10.40.40:80,443",
        "10.10.40.50:80,445"
      ]
    }
  ]
}
```

**What this does:**
- All members can access VPN server (10.10.40.60)
- RDP to DC01 and DC02 only
- HTTP/HTTPS to WEB01, ZABBIX01, TrueNAS
- Blocks other services for security

### 8.3 Enable MFA (Multi-Factor Authentication)
```bash
# Require MFA for all users:

1. Go to: https://login.tailscale.com/admin/settings
2. Click "Security"
3. Enable "Require 2FA"
4. All users must set up:
   - Authenticator app (Google Authenticator, Authy)
   - OR SMS
   - OR hardware key
```

### 8.4 Audit Logging
```bash
# View connection logs:

1. Go to: https://login.tailscale.com/admin/logs
2. See all connections:
   - Who connected
   - When
   - From where
   - To which device

# Export logs (optional):
# Settings → Logs → Enable log streaming
# Send to: Zabbix, Splunk, or SIEM
```

---

## 9. Testing & Verification

### 9.1 VPN Server Health Check
```bash
# On VPN server:

# Check Tailscale service
sudo systemctl status tailscaled
# Should be: active (running)

# Check Tailscale connection
tailscale status
# Should show devices and routes

# Check IP forwarding
cat /proc/sys/net/ipv4/ip_forward
# Should output: 1

# Check advertised routes
ip route show table 52
# Should show Tailscale routes

# Check firewall
sudo ufw status verbose
# Should show allow rules for Tailscale

# Monitor connections
sudo tailscale debug watch-ipn
# Shows real-time connection attempts
```

### 9.2 Client Connectivity Tests
```bash
# From remote client:

# Test 1: Ping VPN server
ping -c 4 10.10.40.60
# Expected: 0% packet loss ✅

# Test 2: Ping internal gateway
ping -c 4 10.10.40.1
# Expected: 0% packet loss ✅

# Test 3: Ping domain controller
ping -c 4 10.10.40.10
# Expected: 0% packet loss ✅

# Test 4: DNS resolution
nslookup dc01.healthclinic.local 10.10.40.10
# Expected: Returns 10.10.40.10 ✅

# Test 5: Web access
curl -I http://10.10.40.30
# Expected: HTTP/1.1 200 OK ✅

# Test 6: Port connectivity
telnet 10.10.40.10 3389
# Expected: Connection established ✅

# Test 7: Traceroute
traceroute 10.10.40.30
# Expected:
# 1  100.64.1.1 (VPN server)
# 2  10.10.40.30 (destination)
```

### 9.3 Performance Tests
```bash
# Install iperf3 on VPN server and client
sudo apt install iperf3

# On VPN server:
iperf3 -s

# On remote client:
iperf3 -c 10.10.40.60 -t 30

# Expected throughput:
# - Home internet: 20-100 Mbps
# - VPN overhead: ~10-20% reduction
# - Latency: +10-50ms

# Test DNS performance
time nslookup dc01.healthclinic.local 10.10.40.10
# Should complete in <0.1 seconds
```

### 9.4 Failover Tests
```bash
# Test 1: VPN server reboot
sudo reboot

# Client should:
# - Lose connection temporarily
# - Automatically reconnect after ~60 seconds
# - Resume normal access

# Test 2: Network interruption
# Disconnect client from internet for 30 seconds
# Reconnect

# Expected:
# - Tailscale automatically reconnects
# - No manual intervention needed

# Test 3: Route change
# On VPN server:
sudo tailscale down
sudo tailscale up --advertise-routes=10.10.0.0/16

# Client should:
# - Briefly lose access
# - Automatically recover
```

---

## 10. Troubleshooting

### 10.1 Common Issues

#### Issue 1: Cannot ping internal servers

**Symptoms:**
```bash
ping 10.10.40.10
# Request timeout
```

**Diagnosis:**
```bash
# Check Tailscale status
tailscale status
# Look for: vpn-healthclinic with advertised routes

# Check if routes are approved
# Go to: https://login.tailscale.com/admin/machines
# Verify: 10.10.0.0/16 route is ✅ enabled
```

**Fix:**
```bash
# On VPN server:
# Verify IP forwarding
sudo sysctl net.ipv4.ip_forward
# If 0, enable it:
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Restart Tailscale
sudo systemctl restart tailscaled
sudo tailscale up --advertise-routes=10.10.0.0/16

# On client:
# Restart Tailscale
sudo systemctl restart tailscaled  # Linux
# or close/open Tailscale app (Windows/Mac)
```

#### Issue 2: Tailscale shows "offline"

**Symptoms:**
```bash
tailscale status
# Shows no devices or "offline"
```

**Diagnosis:**
```bash
# Check service
sudo systemctl status tailscaled

# Check connectivity
ping -c 4 controlplane.tailscale.com
```

**Fix:**
```bash
# Restart Tailscale service
sudo systemctl restart tailscaled

# Re-authenticate
sudo tailscale up --advertise-routes=10.10.0.0/16

# Check firewall
sudo ufw status
# Ensure 41641/udp is allowed
sudo ufw allow 41641/udp
```

#### Issue 3: Slow performance

**Symptoms:**
```bash
# High latency (>100ms)
# Slow file transfers
```

**Diagnosis:**
```bash
# Check path
tailscale ping 100.64.1.1

# Expected output:
# pong from vpn-healthclinic (100.64.1.1) via DERP(ord) in 45ms
#                                           ^^^^
#                              DERP = relay server (not direct)

# Check if direct connection is possible
tailscale status --json | grep -i relay
```

**Fix:**
```bash
# Enable direct connections (if behind firewall):
sudo tailscale up \
  --advertise-routes=10.10.0.0/16 \
  --accept-routes \
  --advertise-exit-node

# Or configure router to allow UDP 41641
# Or use Tailscale's "Skip relay" option
```

#### Issue 4: Route not appearing on client

**Symptoms:**
```bash
# Client shows VPN server but can't reach 10.10.x.x
```

**Diagnosis:**
```bash
# On client:
tailscale status
# Check for: "10.10.0.0/16" in output

# Check client accepts routes
tailscale status --json | grep -i routes
```

**Fix:**
```bash
# On client, ensure --accept-routes is set
sudo tailscale up --accept-routes

# On admin panel:
# https://login.tailscale.com/admin/machines
# Find client device
# Edit route settings
# Enable: "Use exit node" and "Accept subnet routes"
```

#### Issue 5: Authentication errors

**Symptoms:**
```bash
tailscale up
# Error: "login token expired"
```

**Fix:**
```bash
# Re-authenticate
sudo tailscale down
sudo tailscale up --advertise-routes=10.10.0.0/16

# Follow URL to login again

# If still fails:
sudo tailscale logout
sudo tailscale up --advertise-routes=10.10.0.0/16
```

### 10.2 Diagnostic Commands
```bash
# Comprehensive diagnostics:

# View Tailscale logs
sudo journalctl -u tailscaled -f

# Check network interfaces
ip addr show tailscale0

# Check routing table
ip route show table 52

# Check NAT/firewall
sudo iptables -L -v -n
sudo iptables -t nat -L -v -n

# Test DNS
dig @10.10.40.10 dc01.healthclinic.local

# Network trace
sudo tcpdump -i tailscale0 -n icmp

# Tailscale debug info
sudo tailscale bugreport
```

### 10.3 Emergency Recovery
```bash
# If VPN server becomes unresponsive:

# Option 1: Access via Proxmox console
# 1. Log into Proxmox: https://10.10.40.100:8006
# 2. Select VM 160 (vpn)
# 3. Click "Console"
# 4. Login as vp nadmin
# 5. Run diagnostics

# Option 2: Rebuild VPN server
# 1. Snapshot current VM in Proxmox (if possible)
# 2. Create new VM from Ubuntu ISO
# 3. Follow installation steps in Section 4
# 4. Re-run Tailscale setup in Section 5
# 5. Clients automatically reconnect (same hostname)

# Option 3: Temporary access
# If VPN is down and you need immediate access:
# - Use Proxmox VNC console for servers
# - Or connect laptop directly to campus network
# - Fix VPN server from there
```

---

## 11. Management & Monitoring

### 11.1 Daily Operations

**Routine Checks:**
```bash
# Morning check (automated via cron):
#!/bin/bash
# /usr/local/bin/check_vpn.sh

# Check Tailscale status
if ! systemctl is-active --quiet tailscaled; then
    echo "Tailscale service is down!" | mail -s "VPN Alert" admin@healthclinic.local
    systemctl start tailscaled
fi

# Check connectivity
if ! ping -c 1 -W 2 100.100.100.100 > /dev/null 2>&1; then
    echo "Tailscale connectivity lost!" | mail -s "VPN Alert" admin@healthclinic.local
    tailscale up --advertise-routes=10.10.0.0/16
fi

# Log success
logger "VPN health check: OK"
```

**Schedule with cron:**
```bash
# Add to crontab
sudo crontab -e

# Run every 5 minutes
*/5 * * * * /usr/local/bin/check_vpn.sh
```

### 11.2 Monitoring with Zabbix
```bash
# Add VPN server to Zabbix:
# (Already done in Section 5.4 - ZABBIX01 setup)

# Custom monitoring items:

# 1. Tailscale service status
systemctl is-active tailscaled

# 2. Number of connected peers
tailscale status | grep -c "online"

# 3. Advertised route status
tailscale status --json | jq '.Peer[].PrimaryRoutes'

# 4. Bandwidth usage
ifconfig tailscale0 | grep "RX bytes"
```

### 11.3 User Management

**View Connected Users:**
```bash
# On VPN server:
tailscale status

# Output shows:
# 100.64.1.1   vpn-healthclinic     vp nadmin@  linux   -
# 100.64.1.2   laptop-kristine      kristine@   windows online
# 100.64.1.3   laptop-swathi        swathi@       windows     online

# Via web console:
# https://login.tailscale.com/admin/machines
# Shows all connected devices with:
# - Device name
# - User
# - OS
# - Last seen
# - IP addresses
```

**Add New User:**
```bash
# 1. User creates Tailscale account
# 2. Admin shares vpn-healthclinic device
# 3. User accepts and connects
# 4. Verify access:
tailscale status | grep username
```

**Remove User:**
```bash
# Method 1: Unshare device
# 1. https://login.tailscale.com/admin/machines
# 2. Find vpn-healthclinic
# 3. Manage sharing → Remove user

# Method 2: Revoke device
# If user's device is compromised:
# 1. Find user's device in admin panel
# 2. Click three dots → "Revoke"
# 3. User immediately disconnected
```

### 11.4 Logging & Auditing
```bash
# Tailscale logs
sudo journalctl -u tailscaled --since "1 hour ago"

# Connection history
# Web console: https://login.tailscale.com/admin/logs

# Export logs for compliance
# Settings → Logs → Configure log streaming
# Send to Zabbix or external SIEM

# Audit important events:
# - Device added/removed
# - Route changes
# - Failed connection attempts
# - ACL changes
```

### 11.5 Backup & Recovery
```bash
# Backup VPN server configuration:

# 1. Backup Tailscale identity
sudo cp -r /var/lib/tailscale /backup/tailscale_$(date +%Y%m%d)

# 2. Backup firewall rules
sudo ufw status numbered > /backup/ufw_rules_$(date +%Y%m%d).txt

# 3. Backup system configuration
sudo tar -czf /backup/vpn_config_$(date +%Y%m%d).tar.gz \
    /etc/netplan \
    /etc/sysctl.conf \
    /etc/ufw

# 4. Copy backups to TrueNAS
scp /backup/* root@10.10.40.50:/mnt/tank/Backups/VPN/

# Recovery:
# 1. Build new VM (Section 4)
# 2. Restore /var/lib/tailscale
# 3. sudo tailscale up --advertise-routes=10.10.0.0/16
# 4. Same device reappears in admin console
# 5. No client reconfiguration needed! ✅
```

### 11.6 Updates & Maintenance
```bash
# Update Tailscale:
sudo apt update
sudo apt upgrade tailscale

# Tailscale updates automatically by default
# Check version:
tailscale version

# Update Ubuntu system:
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y

# Reboot if kernel updated:
sudo reboot

# Tailscale automatically reconnects after reboot
```

---

## 12. Integration with Infrastructure

### 12.1 Active Directory Integration
```powershell
# Allow Tailscale IPs through Windows Firewall on DCs:

# On DC01 and DC02:
New-NetFirewallRule -DisplayName "Allow Tailscale VPN" `
  -Direction Inbound `
  -RemoteAddress 100.64.0.0/10 `
  -Action Allow `
  -Profile Domain,Private

# Test RDP from VPN client:
mstsc /v:10.10.40.10
# Should connect via Tailscale ✅
```

### 12.2 DNS Configuration
```bash
# Configure Tailscale to use internal DNS:

sudo tailscale up \
  --advertise-routes=10.10.0.0/16 \
  --accept-dns=false

# Set custom DNS on clients:
# Tailscale app → DNS → Override DNS
# Set to: 10.10.40.10, 10.10.40.11

# Or via systemd-resolved (Linux):
sudo nano /etc/systemd/resolved.conf

[Resolve]
DNS=10.10.40.10 10.10.40.11
Domains=healthclinic.local

sudo systemctl restart systemd-resolved
```

### 12.3 Zabbix Monitoring Integration
```bash
# Monitor VPN server from Zabbix:

# 1. VPN server already has Zabbix agent (Section 5.4)
# 2. Add custom items in Zabbix UI:

# Item: Tailscale service status
# Key: proc.num[tailscaled]
# Type: Numeric (integer)
# Trigger: < 1 (service down)

# Item: Connected peers
# Key: system.run["tailscale status | grep -c online"]
# Type: Numeric (integer)
# Trigger: = 0 (no peers)

# Item: Advertised route status
# Key: system.run["tailscale status | grep -q 10.10.0.0/16 && echo 1 || echo 0"]
# Type: Numeric (integer)
# Trigger: = 0 (route not advertised)
```

---

## 13. Security Best Practices

### 13.1 Security Checklist
```
✅ IP forwarding enabled only on VPN server
✅ UFW firewall configured (deny incoming by default)
✅ SSH access restricted to internal networks
✅ Tailscale key expiry disabled for server
✅ MFA enabled for all user accounts
✅ ACLs configured to limit access
✅ Regular security audits via logs
✅ Automatic updates enabled
✅ Backups configured and tested
✅ Disaster recovery plan documented
```

### 13.2 Network Segmentation
```bash
# VPN users should NOT have access to:
# - Cisco device management (10.10.40.2-6)
# - IPMI/iLO interfaces (if present)
# - Proxmox management (except admins)

# Configure Tailscale ACLs to enforce:
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:admin"],
      "dst": ["10.10.40.0/24:*"]
    },
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": [
        "10.10.40.10:3389,445",
        "10.10.40.11:3389,445",
        "10.10.40.20:25,587,993,80,443",
        "10.10.40.30:80,443",
        "10.10.40.40:80,443",
        "10.10.40.50:80,445"
      ]
    }
  ]
}
```

### 13.3 Incident Response
```bash
# If VPN is compromised:

# 1. Immediate actions:
# - Revoke all shared access
# - Disconnect VPN server: sudo tailscale down
# - Review logs: sudo journalctl -u tailscaled

# 2. Investigation:
# - Check Tailscale admin logs
# - Review firewall logs: sudo ufw status numbered
# - Check for unauthorized devices
# - Review ACL changes

# 3. Recovery:
# - Change Tailscale admin password
# - Rotate all shared access
# - Re-enable MFA if disabled
# - Rebuild VPN server if needed
# - Notify affected users

# 4. Prevention:
# - Enable stricter ACLs
# - Require device approval
# - Enable audit logging
# - Regular security reviews
```

---

## 14. Documentation & Training

### 14.1 User Documentation

**Quick Start Guide for Team Members:**
```markdown
# VPN Access - Quick Start

## 1. Create Tailscale Account
- Go to: https://login.tailscale.com
- Sign up (free)

## 2. Install Tailscale
- Windows: https://tailscale.com/download/windows
- Mac: https://tailscale.com/download/mac
- Linux: curl -fsSL https://tailscale.com/install.sh | sh

## 3. Accept VPN Invitation
- Check your email
- Click "Accept"
- Login to Tailscale

## 4. Connect
- Open Tailscale app
- Click "Connect"

## 5. Access Resources
- Proxmox: https://10.10.40.100:8006
- Web App: http://10.10.40.30
- File Shares: \\10.10.40.50\PatientFiles

## Help
- Can't connect? Email: admin@healthclinic.local
- Or check: https://docs.healthclinic.local/vpn
```

### 14.2 Administrator Documentation
```markdown
# VPN Administration Guide

## Daily Tasks
- Check connected users: tailscale status
- Review logs: sudo journalctl -u tailscaled --since today

## Weekly Tasks
- Review access logs in Tailscale admin console
- Check for system updates
- Verify backups completed

## Monthly Tasks
- Review ACL policies
- Audit user access
- Test disaster recovery
- Update documentation
```

---

## 15. Summary

### 15.1 What We Built

✅ **Secure VPN Gateway** - Ubuntu VM running Tailscale  
✅ **Subnet Routing** - Full access to 10.10.0.0/16 network  
✅ **Multi-User Access** - Team members can connect independently  
✅ **Zero Public Exposure** - No open ports, no public IP  
✅ **Encrypted Connections** - WireGuard encryption  
✅ **Easy Management** - Web-based admin console  
✅ **Automatic Failover** - Reconnects automatically  
✅ **Integrated Monitoring** - Zabbix monitoring configured

### 15.2 Key Features

| Feature | Status |
|---------|--------|
| **Remote Access** | ✅ Fully functional |
| **Team Collaboration** | ✅ Multiple users supported |
| **Security** | ✅ Firewall + ACLs configured |
| **Monitoring** | ✅ Zabbix integration |
| **Backup** | ✅ Automated backups |
| **Documentation** | ✅ Complete |

### 15.3 Access Summary

**From Remote Location, You Can:**
- ✅ Access Proxmox Web UI (https://10.10.40.100:8006)
- ✅ RDP to Windows servers (10.10.40.10, 10.10.40.11)
- ✅ SSH to Linux servers (10.10.40.20, 10.10.40.30, 10.10.40.40, 10.10.40.60)
- ✅ Access web applications (http://10.10.40.30)
- ✅ View Zabbix monitoring (http://10.10.40.40/zabbix)
- ✅ Access email webmail (https://10.10.40.20)
- ✅ Access TrueNAS UI (http://10.10.40.50)
- ✅ Access file shares (\\10.10.40.50\PatientFiles)
- ✅ Manage network devices via SSH

### 15.4 Critical Information

| Item | Value |
|------|-------|
| **VPN Server** | vpn.healthclinic.local |
| **Internal IP** | 10.10.40.60 |
| **Tailscale IP** | 100.64.1.1 |
| **Advertised Routes** | 10.10.0.0/16 |
| **Admin Console** | https://login.tailscale.com |
| **Server Username** | vp nadmin |
| **Server Password** | g3company!@# |

---

## 16. Appendices

### Appendix A: Command Reference
```bash
# Essential commands:

# Start Tailscale
sudo tailscale up --advertise-routes=10.10.0.0/16

# Stop Tailscale
sudo tailscale down

# View status
tailscale status

# View IP addresses
tailscale ip -4
tailscale ip -6

# Test connectivity
tailscale ping 100.64.1.1

# View routes
ip route show table 52

# Check service
sudo systemctl status tailscaled

# View logs
sudo journalctl -u tailscaled -f

# Restart service
sudo systemctl restart tailscaled

# Update Tailscale
sudo apt update && sudo apt upgrade tailscale
```

### Appendix B: Firewall Rules
```bash
# Complete UFW configuration:

sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 10.10.30.0/24 to any port 22
sudo ufw allow from 10.10.40.0/24 to any port 22
sudo ufw allow 41641/udp
sudo ufw route allow in on tailscale0 out on ens18
sudo ufw enable
```

### Appendix C: Troubleshooting Flowchart
```
Cannot access internal servers from VPN?
│
├─ Can you ping VPN server (100.64.1.1)?
│  ├─ NO → Check Tailscale connection on client
│  │      → tailscale status
│  │      → Restart Tailscale app
│  │
│  └─ YES → Can you ping internal server (10.10.40.10)?
│           ├─ NO → Check subnet routes approved
│           │      → https://login.tailscale.com/admin/machines
│           │      → Verify 10.10.0.0/16 is enabled
│           │      → Check IP forwarding: sysctl net.ipv4.ip_forward
│           │
│           └─ YES → Problem is with specific service
│                  → Check service status on server
│                  → Check firewall on server
│                  → Check application logs
```

---

## 17. Conclusion

You now have a **fully functional, secure VPN solution** that provides:

🔒 **Secure Remote Access** - Zero-trust WireGuard encryption  
🌐 **Full Network Access** - All VLANs accessible remotely  
👥 **Team Collaboration** - Multiple users, easy management  
📊 **Monitoring** - Integrated with Zabbix  
🛡️ **Security** - Firewall, ACLs, MFA enabled  
📚 **Documentation** - Complete setup and troubleshooting guide

**Total Setup Time:** ~2 hours  
**Monthly Cost:** $0 (Free plan)  
**Maintenance:** Minimal (automatic updates)

---

**Document Version:** 1.0  
**Maintained By:** Kristine Bagsican and Team  
**Project:** Healthcare Clinic IT Infrastructure Capstone

---

**End of Documentation** ✅
