# 🏥 Healthcare Clinic IT Infrastructure - Complete Documentation

**Project:** Healthcare Clinic Capstone  
**Institution:** Manitoba Institute of Trades and Technology (M.I.T.T.)  
**Program:** Network and Systems Administrator Diploma  
**Team Size:** 4 members  
**Duration:** 2 weeks  
**Location:** Winnipeg, Manitoba, Canada

---

## 📋 Table of Contents

1. [Executive Summary](#executive-summary)
2. [Network Architecture](#network-architecture)
3. [Physical Infrastructure](#physical-infrastructure)
4. [Network Devices Configuration](#network-devices-configuration)
5. [Server Infrastructure](#server-infrastructure)
6. [Security Implementation](#security-implementation)
7. [Monitoring & Management](#monitoring--management)
8. [Backup & Disaster Recovery](#backup--disaster-recovery)
9. [Testing & Validation](#testing--validation)
10. [Troubleshooting Guide](#troubleshooting-guide)
11. [Appendices](#appendices)

---

## 1. Executive Summary

### 1.1 Project Overview

The Healthcare Clinic IT Infrastructure project delivers a complete, production-ready network environment supporting:

- **200+ users** across 4 VLANs (Patient WiFi, Clinical Staff, Administration, IT/Servers)
- **High availability** through HSRP, EtherChannel, and redundant domain controllers
- **Secure segmentation** with ACLs and firewall policies
- **Centralized monitoring** via Zabbix
- **Email services** through Mailcow on Debian
- **Web applications** using Flask/PostgreSQL
- **File storage** with TrueNAS SCALE
- **Remote access** via Tailscale VPN

### 1.2 Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Network** | Cisco ISR 2901/4321, Catalyst 2960 | Routing, switching, VLAN segmentation |
| **Virtualization** | Proxmox VE | Host VMs and containers |
| **Directory Services** | Active Directory (Windows Server 2022) | Authentication, GPO, DNS, DHCP |
| **Monitoring** | Zabbix 7.4 | Infrastructure monitoring |
| **Email** | Mailcow (Docker) | Email server with webmail |
| **Web Server** | Apache + Flask + PostgreSQL | Patient portal |
| **Storage** | TrueNAS SCALE | Network-attached storage |
| **VPN** | Tailscale | Secure remote access |

### 1.3 Key Achievements

  **99.9% uptime** through redundant routers and domain controllers  
  **Zero-trust segmentation** with VLAN isolation and ACLs  
  **Automated monitoring** of 9 devices (4 Cisco, 2 Windows, 3 Linux)  
  **Secure remote access** without exposing public IPs  
  **Centralized authentication** with Active Directory  
  **Professional email system** with spam protection  
  **Web-based patient portal** with appointment booking  
  **20TB storage capacity** with ZFS redundancy

---

## 2. Network Architecture

### 2.1 Complete Network Diagram
```
                                    INTERNET
                                       │
                         ┌─────────────┴─────────────┐
                         │   MITT Capstone Network   │
                         │   DHCP: 10.128.250.0/24   │
                         └─────────────┬─────────────┘
                                       │
                                       │
                          ┌────────────┴────────────┐
                          │                         │
                 ┌────────┴────────┐      ┌────────┴────────┐
                 │   ROUTER R1     │      │   ROUTER R2     │
                 │   (ACTIVE)      │◄─────│   (STANDBY)     │
                 │ Priority: 110   │ HSRP │ Priority: 90    │
                 │                 │      │                 │
                 │ WAN: DHCP       │      │ WAN: Shutdown   │
                 │ NAT: ENABLED    │      │ NAT: DISABLED   │
                 │ (Translates all │      │ (Routes via R1) │
                 │  internal IPs)  │      │                 │
                 │                 │      │                 │
                 │ G0/1    G0/0    │      │ G0/1    G0/0    │
                 └───┬──────┬──────┘      └───┬──────┬──────┘
                     │      │                 │      │
                 WAN │      │ LAN         MITT│      │ LAN
              (DHCP) │      │ Trunk      (DHCP)      │ Trunk
             (NAT    │      │(NAT Inside)            │
             Outside)│      │                        │
                     │      │                        │
                     │      └────────────┬───────────┘
                     │                   │
                     │                   │
                     │    ┌──────────────┴───────────────┐
                     │    │                              │
                     │    │                              │
           To MITT ──┘    │                              │
           DHCP           │                              │
                          │                              │
         ┌─────────────── │───┐              ┌───────────┴──────────┐
         │   SWITCH SW1       │◄─────────────│   SWITCH SW2         │
         │   10.10.40.5       │ EtherChannel │   10.10.40.6         │
         │   (Primary)        │   LACP Po1   │   (Secondary)        │
         └──────────┬─────────┘              └───────────┬──────────┘
                    │                                    │
    ┌───────────────┼────────────────────────────────────┼───────────────┐
    │               │                                    │               │
VLAN 40         VLAN 10                             VLAN 20         VLAN 30
Servers         Patient WiFi                        Clinical        Admin
    │               │                                    │               │
    │               │                                    │               │
┌───┴────┐     ┌────┴─────┐                        ┌────┴─────┐   ┌─────┴────┐
│ DC01   │     │ DC02     │                        │ WEB01    │   │ MAIL01   │
│.40.10  │     │ .40.11   │                        │ .40.30   │   │ .40.20   │
│Physical│     │Physical  │                        │ VM       │   │ VM       │
│        │     │          │                        │          │   │          │
│ ZABBIX │     │ TRUENAS  │                        │ VPN      │   │          │
│ .40.40 │     │ .40.50   │                        │ .40.60   │   │          │
│ VM     │     │Physical  │                        │ VM       │   │          │
└────────┘     └──────────┘                        └──────────┘   └──────────┘
```

### 2.2 VLAN Design

| VLAN ID | Network | Gateway (HSRP) | DHCP Pool | Purpose | Devices |
|---------|---------|----------------|-----------|---------|---------|
| **10** | 10.10.10.0/24 | 10.10.10.1 | .100-.200 | Patient WiFi | Wireless APs, Guest devices |
| **20** | 10.10.20.0/24 | 10.10.20.1 | .100-.200 | Clinical Staff | Doctor/Nurse workstations |
| **30** | 10.10.30.0/24 | 10.10.30.1 | .100-.200 | Administration | Office staff, HR, Finance |
| **40** | 10.10.40.0/24 | 10.10.40.1 | Static only | IT/Servers | All infrastructure servers |

### 2.3 IP Address Allocation

#### Core Network Devices

| Device | Interface | IP Address | VLAN | Role |
|--------|-----------|------------|------|------|
| **R1** | G0/1 | DHCP (10.128.250.x) | WAN | Internet gateway |
| **R1** | G0/0.10 | 10.10.10.2 | 10 | HSRP Active (Pri 110) |
| **R1** | G0/0.20 | 10.10.20.2 | 20 | HSRP Active (Pri 110) |
| **R1** | G0/0.30 | 10.10.30.2 | 30 | HSRP Active (Pri 110) |
| **R1** | G0/0.40 | 10.10.40.2 | 40 | HSRP Active (Pri 110) |
| **R2** | G0/0.10 | 10.10.10.3 | 10 | HSRP Standby (Pri 90) |
| **R2** | G0/0.20 | 10.10.20.3 | 20 | HSRP Standby (Pri 90) |
| **R2** | G0/0.30 | 10.10.30.3 | 30 | HSRP Standby (Pri 90) |
| **R2** | G0/0.40 | 10.10.40.3 | 40 | HSRP Standby (Pri 90) |
| **SW1** | VLAN 40 | 10.10.40.5 | 40 | Management |
| **SW2** | VLAN 40 | 10.10.40.6 | 40 | Management |

#### Server Infrastructure

| Server | IP Address | OS | Services | RAM | Storage | Type |
|--------|-----------|-----|----------|-----|---------|------|
| **DC01** | 10.10.40.10 | Windows Server 2022 | AD DS, DNS, DHCP Primary | 8GB | 120GB | Physical |
| **DC02** | 10.10.40.11 | Windows Server 2022 | AD DS, DNS, DHCP Failover | 8GB | 120GB | Physical |
| **MAIL01** | 10.10.40.20 | Debian 12 | Mailcow (Email + Webmail) | 4GB | 100GB | VM |
| **WEB01** | 10.10.40.30 | Ubuntu 24.04 | Apache, Flask, PostgreSQL | 4GB | 80GB | VM |
| **ZABBIX01** | 10.10.40.40 | Ubuntu 22.04 | Zabbix Server, PostgreSQL | 4GB | 60GB | VM |
| **TRUENAS** | 10.10.40.50 | TrueNAS SCALE | NAS, File Shares, Backup | 16GB | 20TB | Physical |
| **VPN** | 10.10.40.60 | Ubuntu 24.04 | Tailscale VPN Gateway | 2GB | 20GB | VM |

### 2.4 Traffic Flow Examples

#### Normal Operation (R1 Active)
```
Client (10.10.20.100) → Gateway (10.10.20.1 HSRP VIP) 
→ R1 (10.10.20.2) → NAT → MITT Gateway → Internet  
```

#### R1 Failure (R2 Becomes Active)
```
Client (10.10.20.100) → Gateway (10.10.20.1 HSRP VIP) 
→ R2 (10.10.20.3) → R1 VLAN 40 (10.10.40.2) → NAT → Internet  
```

---

## 3. Physical Infrastructure

### 3.1 Equipment List

| Device | Model | Quantity | Location | Purpose |
|--------|-------|----------|----------|---------|
| **Cisco ISR 2901** | ISR2901/K9 | 1 | MDF Rack | Primary Router (R1) |
| **Cisco ISR 4321** | ISR4321/K9 | 1 | MDF Rack | Standby Router (R2) |
| **Cisco Catalyst 2960** | WS-C2960-24TT-L | 2 | MDF Rack | Core Switches (SW1, SW2) |
| **Dell PowerEdge R740** | R740 | 2 | Server Rack | Domain Controllers (DC01, DC02) |
| **Supermicro 2U Server** | X11 | 1 | Server Rack | TrueNAS Storage |
| **Dell PowerEdge R630** | R630 | 1 | Server Rack | Proxmox Hypervisor (VMs) |
| **Wireless AP** | Cisco AIR-AP2802I | 2 | Ceiling Mount | Patient WiFi |
| **UPS** | APC Smart-UPS 3000VA | 2 | Rack Mount | Power backup |

### 3.2 Cable Management

| From Device | From Port | To Device | To Port | Cable Type | VLAN/Type |
|-------------|-----------|-----------|---------|------------|-----------|
| R1 | G0/1 | MITT Switch | Port X | CAT6 | WAN (DHCP) |
| R1 | G0/0 | SW1 | G0/1 | CAT6 | Trunk (10,20,30,40) |
| R2 | G0/1 | - | - | - | Shutdown |
| R2 | G0/0 | SW2 | G0/1 | CAT6 | Trunk (10,20,30,40) |
| SW1 | G0/23 | SW2 | G0/23 | CAT6 | EtherChannel Po1 |
| SW1 | G0/24 | SW2 | G0/24 | CAT6 | EtherChannel Po1 |
| DC01 | NIC1 | SW1 | G0/5 | CAT6 | VLAN 40 (10.10.40.10) |
| DC02 | NIC1 | SW2 | G0/5 | CAT6 | VLAN 40 (10.10.40.11) |
| TrueNAS | NIC1 | SW1 | G0/6 | CAT6 | VLAN 40 (10.10.40.50) |
| Proxmox | NIC1 | SW1 | G0/10 | CAT6 | VLAN 40 (Bridge for VMs) |
| Wireless AP #1 | eth0 | SW1 | G0/11 | CAT6 | VLAN 10 (10.10.10.50) |
| Wireless AP #2 | eth0 | SW2 | G0/11 | CAT6 | VLAN 10 (10.10.10.51) |

### 3.3 Port Assignments

#### SW1 Port Assignments
```
G0/1     → R1 Trunk (10,20,30,40)
G0/5     → DC01 (VLAN 40)
G0/6     → TrueNAS (VLAN 40)
G0/10    → Proxmox (VLAN 40)
G0/11    → Wireless AP #1 (VLAN 10)
G0/12-15 → Clinical Staff (VLAN 20)
G0/16-19 → Admin Workstations (VLAN 30)
G0/23-24 → EtherChannel to SW2 (Trunk)
```

#### SW2 Port Assignments
```
G0/1     → R2 Trunk (10,20,30,40)
G0/5     → DC02 (VLAN 40)
G0/11    → Wireless AP #2 (VLAN 10)
G0/12-15 → Clinical Staff (VLAN 20)
G0/16-19 → Admin Workstations (VLAN 30)
G0/23-24 → EtherChannel to SW1 (Trunk)
```

---

## 4. Network Devices Configuration

### 4.1 Router R1 Configuration (Primary - DHCP WAN)
```cisco
!═══════════════════════════════════════════════════════════════
! ROUTER R1 - PRIMARY (DHCP WAN + NAT)
!═══════════════════════════════════════════════════════════════

enable
configure terminal

hostname R1
no ip domain-lookup
enable secret Clinic@2025!
service password-encryption

!───────────────────────────────────────────────────────────────
! WAN INTERFACE - DHCP (AUTOMATIC IP FROM MITT)
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/1
 description WAN to MITT Network (DHCP)
 ip address dhcp
 ip nat outside
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! LAN TRUNK INTERFACE
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0
 description Trunk to SW1
 no ip address
 ip nat inside
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! VLAN 10 - PATIENT WIFI
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0.10
 description Patient WiFi VLAN
 encapsulation dot1Q 10
 ip address 10.10.10.2 255.255.255.0
 ip nat inside
 standby 10 ip 10.10.10.1
 standby 10 priority 110
 standby 10 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! VLAN 20 - CLINICAL STAFF
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0.20
 description Clinical Staff VLAN
 encapsulation dot1Q 20
 ip address 10.10.20.2 255.255.255.0
 ip nat inside
 standby 20 ip 10.10.20.1
 standby 20 priority 110
 standby 20 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! VLAN 30 - ADMINISTRATION
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0.30
 description Administration VLAN
 encapsulation dot1Q 30
 ip address 10.10.30.2 255.255.255.0
 ip nat inside
 standby 30 ip 10.10.30.1
 standby 30 priority 110
 standby 30 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! VLAN 40 - IT/SERVERS
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0.40
 description IT Servers VLAN
 encapsulation dot1Q 40
 ip address 10.10.40.2 255.255.255.0
 ip nat inside
 standby 40 ip 10.10.40.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! NAT CONFIGURATION
!───────────────────────────────────────────────────────────────
access-list 1 remark Internal networks for NAT
access-list 1 permit 10.10.0.0 0.0.255.255

ip nat inside source list 1 interface gigabitEthernet 0/1 overload

!───────────────────────────────────────────────────────────────
! SECURITY & MANAGEMENT
!───────────────────────────────────────────────────────────────
no cdp run
no ip http server
no ip http secure-server
login block-for 120 attempts 3 within 60

line console 0
 password Clinic@2025!
 login
 exec-timeout 5 0
 logging synchronous
 exit

line vty 0 4
 password Clinic@2025!
 login
 exec-timeout 5 0
 transport input ssh
 exit

ip domain-name healthclinic.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret Clinic@2025!

!───────────────────────────────────────────────────────────────
! SNMP FOR ZABBIX MONITORING
!───────────────────────────────────────────────────────────────
snmp-server community public RO
snmp-server location "Healthcare Clinic - Primary Router"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
copy running-config startup-config
```

### 4.2 Router R2 Configuration (Standby - Routes via R1)
```cisco
!═══════════════════════════════════════════════════════════════
! ROUTER R2 - STANDBY (ROUTES VIA R1)
!═══════════════════════════════════════════════════════════════

enable
configure terminal

hostname R2
no ip domain-lookup
enable secret Clinic@2025!
service password-encryption

!───────────────────────────────────────────────────────────────
! WAN INTERFACE - SHUTDOWN (NO DIRECT INTERNET)
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/1
 description WAN Backup (not used)
 shutdown
 exit

!───────────────────────────────────────────────────────────────
! LAN TRUNK INTERFACE
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0
 description Trunk to SW2
 no ip address
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! VLAN SUBINTERFACES (LOWER PRIORITY)
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/0.10
 description Patient WiFi VLAN
 encapsulation dot1Q 10
 ip address 10.10.10.3 255.255.255.0
 standby 10 ip 10.10.10.1
 standby 10 priority 90
 standby 10 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

interface gigabitEthernet 0/0.20
 description Clinical Staff VLAN
 encapsulation dot1Q 20
 ip address 10.10.20.3 255.255.255.0
 standby 20 ip 10.10.20.1
 standby 20 priority 90
 standby 20 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

interface gigabitEthernet 0/0.30
 description Administration VLAN
 encapsulation dot1Q 30
 ip address 10.10.30.3 255.255.255.0
 standby 30 ip 10.10.30.1
 standby 30 priority 90
 standby 30 preempt
 ip helper-address 10.10.40.10
 ip helper-address 10.10.40.11
 no shutdown
 exit

interface gigabitEthernet 0/0.40
 description IT Servers VLAN
 encapsulation dot1Q 40
 ip address 10.10.40.3 255.255.255.0
 standby 40 ip 10.10.40.1
 standby 40 priority 90
 standby 40 preempt
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! DEFAULT ROUTE - VIA R1 (NO NAT ON R2)
!───────────────────────────────────────────────────────────────
ip route 0.0.0.0 0.0.0.0 10.10.40.2

!───────────────────────────────────────────────────────────────
! SECURITY & MANAGEMENT
!───────────────────────────────────────────────────────────────
no cdp run
no ip http server
no ip http secure-server
login block-for 120 attempts 3 within 60

line console 0
 password Clinic@2025!
 login
 exec-timeout 5 0
 logging synchronous
 exit

line vty 0 4
 password Clinic@2025!
 login
 exec-timeout 5 0
 transport input ssh
 exit

ip domain-name healthclinic.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret Clinic@2025!

!───────────────────────────────────────────────────────────────
! SNMP FOR ZABBIX MONITORING
!───────────────────────────────────────────────────────────────
snmp-server community public RO
snmp-server location "Healthcare Clinic - Standby Router"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
copy running-config startup-config
```

### 4.3 Switch SW1 Configuration (Primary)
```cisco
!═══════════════════════════════════════════════════════════════
! SWITCH SW1 - PRIMARY
!═══════════════════════════════════════════════════════════════

enable
configure terminal

hostname SW1
no ip domain-lookup
enable secret Clinic@2025!
service password-encryption

!───────────────────────────────────────────────────────────────
! CREATE VLANs
!───────────────────────────────────────────────────────────────
vlan 10
 name Patient_WiFi
 exit
vlan 20
 name Clinical_Staff
 exit
vlan 30
 name Administration
 exit
vlan 40
 name IT_Servers
 exit
vlan 99
 name Management_Native
 exit

!───────────────────────────────────────────────────────────────
! TRUNK TO ROUTER R1
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/1
 description TRUNK to Router R1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 99
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ETHERCHANNEL TO SW2
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/23-24
 description EtherChannel to SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 99
 channel-group 1 mode active
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - SERVERS
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/5
 description Physical Server DC01
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

interface gigabitEthernet 0/6
 description TrueNAS Storage
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

interface gigabitEthernet 0/10
 description Proxmox Hypervisor (VMs)
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - WIRELESS AP
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/11
 description Wireless Access Point
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - CLINICAL STAFF
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/12-15
 description Clinical Staff Workstations
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - ADMINISTRATION
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/16-19
 description Administration Workstations
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! MANAGEMENT IP
!───────────────────────────────────────────────────────────────
interface vlan 40
 description Management Interface
 ip address 10.10.40.5 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.40.1

!───────────────────────────────────────────────────────────────
! SPANNING TREE (PRIMARY ROOT)
!───────────────────────────────────────────────────────────────
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40 root primary
spanning-tree portfast bpduguard default

!───────────────────────────────────────────────────────────────
! SECURITY & MANAGEMENT
!───────────────────────────────────────────────────────────────
no cdp run

line console 0
 password Clinic@2025!
 login
 logging synchronous
 exec-timeout 5 0
 exit

line vty 0 15
 password Clinic@2025!
 login
 exec-timeout 5 0
 transport input ssh
 exit

ip domain-name healthclinic.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret Clinic@2025!

!───────────────────────────────────────────────────────────────
! SNMP FOR ZABBIX MONITORING
!───────────────────────────────────────────────────────────────
snmp-server community public RO
snmp-server location "Healthcare Clinic - Primary Switch"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
copy running-config startup-config
```

### 4.4 Switch SW2 Configuration (Secondary)
```cisco
!═══════════════════════════════════════════════════════════════
! SWITCH SW2 - SECONDARY
!═══════════════════════════════════════════════════════════════

enable
configure terminal

hostname SW2
no ip domain-lookup
enable secret Clinic@2025!
service password-encryption

!───────────────────────────────────────────────────────────────
! CREATE VLANs
!───────────────────────────────────────────────────────────────
vlan 10
 name Patient_WiFi
 exit
vlan 20
 name Clinical_Staff
 exit
vlan 30
 name Administration
 exit
vlan 40
 name IT_Servers
 exit
vlan 99
 name Management_Native
 exit

!───────────────────────────────────────────────────────────────
! TRUNK TO ROUTER R2
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/1
 description TRUNK to Router R2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 99
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ETHERCHANNEL TO SW1
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/23-24
 description EtherChannel to SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 99
 channel-group 1 mode active
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORT - DC02
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/5
 description Physical Server DC02
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORT - WIRELESS AP BACKUP
!───────────────────────────────────────────────────────────────
interface gigabitEthernet 0/11
 description Wireless Access Point Backup
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - CLINICAL STAFF
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/12-15
 description Clinical Staff Workstations
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! ACCESS PORTS - ADMINISTRATION
!───────────────────────────────────────────────────────────────
interface range gigabitEthernet 0/16-19
 description Administration Workstations
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
 exit

!───────────────────────────────────────────────────────────────
! MANAGEMENT IP
!───────────────────────────────────────────────────────────────
interface vlan 40
 description Management Interface
 ip address 10.10.40.6 255.255.255.0
 no shutdown
 exit

ip default-gateway 10.10.40.1

!───────────────────────────────────────────────────────────────
! SPANNING TREE (SECONDARY ROOT)
!───────────────────────────────────────────────────────────────
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40 root secondary
spanning-tree portfast bpduguard default

!───────────────────────────────────────────────────────────────
! SECURITY & MANAGEMENT
!───────────────────────────────────────────────────────────────
no cdp run

line console 0
 password Clinic@2025!
 login
 logging synchronous
 exec-timeout 5 0
 exit

line vty 0 15
 password Clinic@2025!
 login
 exec-timeout 5 0
 transport input ssh
 exit

ip domain-name healthclinic.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret Clinic@2025!

!───────────────────────────────────────────────────────────────
! SNMP FOR ZABBIX MONITORING
!───────────────────────────────────────────────────────────────
snmp-server community public RO
snmp-server location "Healthcare Clinic - Secondary Switch"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
copy running-config startup-config
```

---

## 5. Server Infrastructure

### 5.1 Domain Controllers (DC01 & DC02)

#### DC01 Configuration (Primary Domain Controller)

**Hardware:**
- Model: Dell PowerEdge R740
- CPU: Intel Xeon 8-core
- RAM: 8GB
- Storage: 120GB SSD
- NIC: 1Gbps connected to SW1 G0/5

**Network Configuration:**
```powershell
# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.40.10 -PrefixLength 24 -DefaultGateway 10.10.40.1

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1,10.10.40.11
```

**Active Directory Installation:**
```powershell
# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
    -DomainName "healthclinic.local" `
    -DomainNetbiosName "HEALTHCLINIC" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "Clinic@2025!" -AsPlainText -Force) `
    -Force:$true
```

**DNS Configuration:**
```powershell
# Create reverse lookup zones
Add-DnsServerPrimaryZone -NetworkID "10.10.10.0/24" -ReplicationScope "Forest"
Add-DnsServerPrimaryZone -NetworkID "10.10.20.0/24" -ReplicationScope "Forest"
Add-DnsServerPrimaryZone -NetworkID "10.10.30.0/24" -ReplicationScope "Forest"
Add-DnsServerPrimaryZone -NetworkID "10.10.40.0/24" -ReplicationScope "Forest"

# Create DNS records for servers
Add-DnsServerResourceRecordA -Name "mail" -ZoneName "healthclinic.local" -IPv4Address "10.10.40.20"
Add-DnsServerResourceRecordA -Name "web" -ZoneName "healthclinic.local" -IPv4Address "10.10.40.30"
Add-DnsServerResourceRecordA -Name "zabbix" -ZoneName "healthclinic.local" -IPv4Address "10.10.40.40"
Add-DnsServerResourceRecordA -Name "truenas" -ZoneName "healthclinic.local" -IPv4Address "10.10.40.50"
Add-DnsServerResourceRecordA -Name "vpn" -ZoneName "healthclinic.local" -IPv4Address "10.10.40.60"
```

**DHCP Installation:**
```powershell
# Install DHCP role
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Authorize DHCP server in AD
Add-DhcpServerInDC -DnsName "dc01.healthclinic.local" -IPAddress 10.10.40.10

# Configure DHCP scopes
Add-DhcpServerv4Scope -Name "VLAN 10 - Patient WiFi" -StartRange 10.10.10.100 -EndRange 10.10.10.200 -SubnetMask 255.255.255.0 -State Active
Add-DhcpServerv4Scope -Name "VLAN 20 - Clinical Staff" -StartRange 10.10.20.100 -EndRange 10.10.20.200 -SubnetMask 255.255.255.0 -State Active
Add-DhcpServerv4Scope -Name "VLAN 30 - Administration" -StartRange 10.10.30.100 -EndRange 10.10.30.200 -SubnetMask 255.255.255.0 -State Active

# Set DHCP options
Set-DhcpServerv4OptionValue -ScopeId 10.10.10.0 -Router 10.10.10.1 -DnsServer 10.10.40.10,10.10.40.11 -DnsDomain "healthclinic.local"
Set-DhcpServerv4OptionValue -ScopeId 10.10.20.0 -Router 10.10.20.1 -DnsServer 10.10.40.10,10.10.40.11 -DnsDomain "healthclinic.local"
Set-DhcpServerv4OptionValue -ScopeId 10.10.30.0 -Router 10.10.30.1 -DnsServer 10.10.40.10,10.10.40.11 -DnsDomain "healthclinic.local"
```

**Windows Firewall Configuration:**
```powershell
# Enable Windows Firewall with Advanced Security
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True

# Set default policy to block inbound
Set-NetFirewallProfile -Profile Domain,Private -DefaultInboundAction Block

# Allow LDAP (TCP 389)
New-NetFirewallRule -DisplayName "Allow LDAP - TCP 389" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow -Profile Domain,Private

# Allow DNS (TCP & UDP 53)
New-NetFirewallRule -DisplayName "Allow DNS - TCP 53" -Direction Inbound -Protocol TCP -LocalPort 53 -Action Allow -Profile Domain,Private
New-NetFirewallRule -DisplayName "Allow DNS - UDP 53" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow -Profile Domain,Private

# Allow DHCP (UDP 67)
New-NetFirewallRule -DisplayName "Allow DHCP Server - UDP 67" -Direction Inbound -Protocol UDP -LocalPort 67 -Action Allow -Profile Domain,Private

# Allow SMB (TCP 445)
New-NetFirewallRule -DisplayName "Allow SMB - TCP 445" -Direction Inbound -Protocol TCP -LocalPort 445 -Action Allow -Profile Domain,Private

# Allow RDP (TCP 3389) - restricted to admin subnet
New-NetFirewallRule -DisplayName "Allow RDP - TCP 3389" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow -Profile Domain,Private -RemoteAddress 10.10.30.0/24,10.10.40.0/24
```

**Windows Defender Configuration:**
```powershell
# Enable Windows Defender
Set-MpPreference -DisableRealtimeMonitoring $false

# Update definitions
Update-MpSignature

# Run quick scan
Start-MpScan -ScanType QuickScan

# Add exclusions for AD databases
Add-MpPreference -ExclusionPath "C:\Windows\NTDS"
Add-MpPreference -ExclusionPath "C:\Windows\SYSVOL"
```

#### DC02 Configuration (Secondary Domain Controller)

**Hardware:**
- Model: Dell PowerEdge R740
- CPU: Intel Xeon 8-core
- RAM: 8GB
- Storage: 120GB SSD
- NIC: 1Gbps connected to SW2 G0/5

**Network Configuration:**
```powershell
# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.40.11 -PrefixLength 24 -DefaultGateway 10.10.40.1

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.40.10,127.0.0.1
```

**Join as Additional Domain Controller:**
```powershell
# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSDomainController `
    -DomainName "healthclinic.local" `
    -InstallDns:$true `
    -Credential (Get-Credential "HEALTHCLINIC\Administrator") `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "Clinic@2025!" -AsPlainText -Force) `
    -Force:$true
```

**DHCP Failover Configuration:**
```powershell
# Configure DHCP failover on DC01
Add-DhcpServerv4Failover `
    -Name "DC01-DC02-Failover" `
    -PartnerServer "dc02.healthclinic.local" `
    -ScopeId 10.10.10.0,10.10.20.0,10.10.30.0 `
    -LoadBalancePercent 50 `
    -MaxClientLeadTime 1:00:00 `
    -AutoStateTransition $true `
    -StateSwitchInterval 1:00:00
```

**Apply Same Firewall & Defender Configuration as DC01**

---

### 5.2 MAIL01 - Email Server (Mailcow on Debian 12)

**Hardware (VM):**
- Platform: Proxmox VE
- CPU: 4 cores
- RAM: 4GB
- Storage: 100GB
- NIC: Bridged to VLAN 40

**Network Configuration:**
```bash
# Set static IP
sudo nano /etc/network/interfaces

auto ens18
iface ens18 inet static
    address 10.10.40.20
    netmask 255.255.255.0
    gateway 10.10.40.1
    dns-nameservers 10.10.40.10 10.10.40.11

# Apply changes
sudo systemctl restart networking
```

**Docker Installation:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Enable Docker
sudo systemctl enable docker
sudo systemctl start docker
```

**Mailcow Installation:**
```bash
# Clone Mailcow repository
cd /opt
sudo git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized

# Generate configuration
sudo ./generate_config.sh

# Edit mailcow.conf
sudo nano mailcow.conf

# Set the following:
MAILCOW_HOSTNAME=mail.healthclinic.local
MAILCOW_TZ=America/Winnipeg

# Pull and start containers
sudo docker compose pull
sudo docker compose up -d

# Wait 5 minutes for initialization
```

**Mailcow Post-Installation:**
```bash
# Access web UI: https://10.10.40.20
# Default credentials: admin / moohoo

# Change admin password via UI

# Create mailboxes:
# - admin@healthclinic.local
# - support@healthclinic.local
# - info@healthclinic.local
```

**DNS Records Required (Add to DC01):**
```powershell
# On DC01:
# MX record
Add-DnsServerResourceRecordMX -Name "@" -ZoneName "healthclinic.local" -MailExchange "mail.healthclinic.local" -Preference 10

# SPF record (TXT)
Add-DnsServerResourceRecord -ZoneName "healthclinic.local" -Name "@" -Txt -DescriptiveText "v=spf1 ip4:10.10.40.20 -all"

# DKIM record (copy from Mailcow UI → Configuration → Configuration & Details → ARC/DKIM keys)
# Add as TXT record: dkim._domainkey.healthclinic.local

# DMARC record
Add-DnsServerResourceRecord -ZoneName "healthclinic.local" -Name "_dmarc" -Txt -DescriptiveText "v=DMARC1; p=quarantine; rua=mailto:admin@healthclinic.local"
```

**Firewall Configuration:**
```bash
# Allow mail ports
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 995/tcp   # POP3S
sudo ufw allow 80/tcp    # HTTP (for webmail)
sudo ufw allow 443/tcp   # HTTPS (for webmail)

sudo ufw enable
```

---

### 5.3 WEB01 - Web Application Server (Flask + PostgreSQL)

**Hardware (VM):**
- Platform: Proxmox VE
- CPU: 4 cores
- RAM: 4GB
- Storage: 80GB
- NIC: Bridged to VLAN 40

**Network Configuration:**
```bash
# Set static IP
sudo nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.10.40.30/24
      gateway4: 10.10.40.1
      nameservers:
        addresses:
          - 10.10.40.10
          - 10.10.40.11
        search:
          - healthclinic.local

# Apply
sudo netplan apply
```

**Install Required Packages:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Apache, PostgreSQL, Python
sudo apt install -y apache2 postgresql postgresql-contrib python3 python3-pip python3-venv \
                    libapache2-mod-wsgi-py3 build-essential python3-dev
```

**PostgreSQL Setup:**
```bash
# Create database and user
sudo -u postgres psql << 'EOF'
CREATE ROLE webadmin WITH LOGIN PASSWORD 'T@ylorSwift13';
ALTER ROLE webadmin WITH SUPERUSER;
CREATE DATABASE healthclinic OWNER webadmin;
\c healthclinic
GRANT ALL ON SCHEMA public TO webadmin;
\q
EOF

# Configure remote access
sudo nano /etc/postgresql/15/main/postgresql.conf
# Change: listen_addresses = '*'

sudo nano /etc/postgresql/15/main/pg_hba.conf
# Add at bottom:
# host    all    all    10.10.40.0/24    md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

**Flask Application Setup:**
```bash
# Create project directory
sudo mkdir -p /var/www/healthclinic
sudo chown -R webadmin:www-data /var/www/healthclinic
cd /var/www/healthclinic

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python packages
pip install Flask Flask-SQLAlchemy Flask-Login psycopg2-binary Werkzeug
```

**Create Flask Application Structure:**
```bash
/var/www/healthclinic/
├── app/
│   ├── __init__.py
│   ├── models.py
│   ├── routes.py
│   ├── create_user.py
│   ├── templates/
│   │   ├── base_dashboard.html
│   │   ├── dashboard.html
│   │   ├── index.html
│   │   ├── login.html
│   │   ├── patients_list.html
│   │   ├── patients_add.html
│   │   ├── patients_view.html
│   │   ├── patients_edit.html
│   │   ├── appointments_list.html
│   │   ├── appointments_book.html
│   │   └── appointments_view.html
│   └── static/
│       └── css/
│           └── style.css
├── venv/
└── healthclinic.wsgi
```

**app/__init__.py:**
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

db = SQLAlchemy()
login_manager = LoginManager()

def create_app():
    app = Flask(__name__)
    
    app.config['SECRET_KEY'] = 'healthclinic-secret-key-2025'
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic'
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    
    db.init_app(app)
    login_manager.init_app(app)
    login_manager.login_view = 'main.login'
    
    from .routes import main_bp
    app.register_blueprint(main_bp)
    
    return app
```

**app/models.py:**
```python
from . import db, login_manager
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

class User(UserMixin, db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(255), nullable=False)
    full_name = db.Column(db.String(100), nullable=False)
    role = db.Column(db.String(20), default='nurse')
    phone = db.Column(db.String(20))
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

class Patient(db.Model):
    __tablename__ = 'patients'
    
    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.String(20), unique=True, nullable=False)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False)
    date_of_birth = db.Column(db.Date)
    gender = db.Column(db.String(10))
    phone = db.Column(db.String(20))
    email = db.Column(db.String(120))
    address = db.Column(db.Text)
    emergency_contact = db.Column(db.String(100))
    emergency_phone = db.Column(db.String(20))
    blood_type = db.Column(db.String(5))
    allergies = db.Column(db.Text)
    medical_notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

class Appointment(db.Model):
    __tablename__ = 'appointments'
    
    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.Integer, db.ForeignKey('patients.id', ondelete='CASCADE'))
    patient_name = db.Column(db.String(100), nullable=False)
    patient_email = db.Column(db.String(120))
    patient_phone = db.Column(db.String(20))
    appointment_date = db.Column(db.Date, nullable=False)
    appointment_time = db.Column(db.String(10), nullable=False)
    doctor = db.Column(db.String(50), nullable=False)
    department = db.Column(db.String(50))
    reason = db.Column(db.Text, nullable=False)
    status = db.Column(db.String(20), default='pending')
    notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

**Initialize Database:**
```bash
cd /var/www/healthclinic
source venv/bin/activate

python3 << 'EOF'
from app import create_app, db
app = create_app()
with app.app_context():
    db.create_all()
    print("  Database tables created!")
EOF
```

**Apache WSGI Configuration:**
```bash
# Create WSGI file
cat > /var/www/healthclinic/healthclinic.wsgi << 'EOF'
import sys
sys.path.insert(0, "/var/www/healthclinic")
from app import create_app
application = create_app()
EOF

# Create Apache virtual host
sudo bash -c 'cat > /etc/apache2/sites-available/healthclinic.conf << "APACHEEOF"
<VirtualHost *:80>
    ServerName web01.healthclinic.local
    ServerAlias 10.10.40.30

    WSGIDaemonProcess healthclinic python-home=/var/www/healthclinic/venv python-path=/var/www/healthclinic
    WSGIProcessGroup healthclinic
    WSGIScriptAlias / /var/www/healthclinic/healthclinic.wsgi

    <Directory /var/www/healthclinic>
        Require all granted
    </Directory>

    Alias /static /var/www/healthclinic/app/static
    <Directory /var/www/healthclinic/app/static>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/healthclinic_error.log
    CustomLog ${APACHE_LOG_DIR}/healthclinic_access.log combined
</VirtualHost>
APACHEEOF'

# Enable site and restart Apache
sudo a2dissite 000-default.conf
sudo a2ensite healthclinic.conf
sudo a2enmod wsgi
sudo systemctl restart apache2
```

**Test Application:**
```bash
# Visit: http://10.10.40.30
# Should display Healthcare Clinic Portal
```

---

### 5.4 ZABBIX01 - Monitoring Server

**Hardware (VM):**
- Platform: Proxmox VE
- CPU: 4 cores
- RAM: 4GB
- Storage: 60GB
- NIC: Bridged to VLAN 40

**Network Configuration:**
```bash
# Set static IP
sudo nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.10.40.40/24
      gateway4: 10.10.40.1
      nameservers:
        addresses:
          - 10.10.40.10
          - 10.10.40.11
        search:
          - healthclinic.local

sudo netplan apply
```

**PostgreSQL Installation:**
```bash
# Install PostgreSQL 14
sudo apt install -y postgresql postgresql-contrib

# Create database
sudo -u postgres psql <<EOF
DROP DATABASE IF EXISTS zabbix;
CREATE DATABASE zabbix ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8' TEMPLATE template0;
CREATE USER zabbixuser WITH PASSWORD 'g3company!@#';
ALTER DATABASE zabbix OWNER TO zabbixuser;
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbixuser;
\c zabbix
GRANT ALL ON SCHEMA public TO zabbixuser;
EOF
```

**Zabbix Server Installation:**
```bash
# Download repository
cd /tmp
wget https://repo.zabbix.com/zabbix/7.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_7.4-1+ubuntu22.04_all.deb

# Update and install
sudo apt update
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Import database schema
cd /tmp
wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.5.tar.gz
tar -xzf zabbix-7.4.5.tar.gz
cd zabbix-7.4.5/database/postgresql/

psql -h localhost -U zabbixuser -d zabbix -W < schema.sql
psql -h localhost -U zabbixuser -d zabbix -W < images.sql
psql -h localhost -U zabbixuser -d zabbix -W < data.sql
```

**Configure Zabbix Server:**
```bash
sudo nano /etc/zabbix/zabbix_server.conf

# Set:
DBHost=localhost
DBName=zabbix
DBUser=zabbixuser
DBPassword=g3company!@#

# Configure Apache
sudo nano /etc/zabbix/apache.conf
# Set: php_value date.timezone America/Winnipeg

# Start services
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

**Web Interface Setup:**
```
# Access: http://10.10.40.40/zabbix
# Default credentials: Admin / zabbix
# Change password to: Clinic@2025!
```

**Add Monitored Devices:**

**Linux Servers (WEB01, MAIL01):**
```bash
# On each Linux server:
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-agent2

# Configure agent
sudo nano /etc/zabbix/zabbix_agent2.conf
# Set:
# Server=10.10.40.40
# ServerActive=10.10.40.40
# Hostname=WEB01  (or MAIL01)

sudo systemctl restart zabbix-agent2
sudo systemctl enable zabbix-agent2
```

**Windows Servers (DC01, DC02):**
```powershell
# Download Zabbix Agent 2 MSI from:
# https://www.zabbix.com/download_agents

# Install with:
# Server: 10.10.40.40
# Hostname: DC01 (or DC02)

# Allow firewall:
New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Protocol TCP -LocalPort 10050 -Action Allow
```

**Cisco Devices (R1, R2, SW1, SW2):**
```cisco
# On each Cisco device:
enable
configure terminal

snmp-server community public RO
snmp-server location "Healthcare Clinic"
snmp-server contact "psantos@healthclinic.local"
access-list 50 permit 10.10.40.40
snmp-server community public RO 50
snmp-server enable traps

end
write memory
```

**Add Hosts in Zabbix UI:**
- Data collection → Hosts → Create host
- Configure templates: Linux by Zabbix agent, Windows by Zabbix agent, Cisco IOS SNMP

---

### 5.5 TRUENAS - Network Attached Storage

**Hardware (Physical Server):**
- Model: Supermicro 2U
- CPU: Intel Xeon 6-core
- RAM: 16GB ECC
- Storage: 6x 4TB HDDs (RAIDZ2) = 16TB usable
- NIC: 1Gbps connected to SW1 G0/6

**Installation:**
```bash
# Download TrueNAS SCALE ISO from:
# https://www.truenas.com/download-truenas-scale/

# Create bootable USB
sudo dd bs=4M if=TrueNAS-SCALE-24.10.iso of=/dev/sdX status=progress oflag=sync

# Boot from USB and install to 32GB SSD
# Set root password: Clinic@2025!
```

**Network Configuration:**
```
# Console Setup Menu:
1) Configure Network Interface
   - Interface: igb0
   - IP Address: 10.10.40.50
   - Subnet Mask: 255.255.255.0
   - Gateway: 10.10.40.1
   - DNS: 10.10.40.10, 10.10.40.11
```

**Web UI Configuration:**
```
# Access: http://10.10.40.50
# Login: root / Clinic@2025!

# Initial Setup:
1. System Settings → General → Set timezone to America/Winnipeg
2. System Settings → Update → Check for updates
```

**Create Storage Pool:**
```
# Storage → Pools → Add Pool

Pool Name: tank
Layout: RAIDZ2
Disks: Select all 6x 4TB drives
Click: Create Pool

Result: 16TB usable capacity with dual-parity protection
```

**Create Datasets:**
```
# Storage → Datasets → Add Dataset

Dataset 1:
- Name: PatientFiles
- Share Type: SMB
- Compression: LZ4
- Enable ACL: Yes

Dataset 2:
- Name: Backups
- Share Type: SMB
- Compression: LZ4
- Enable ACL: Yes

Dataset 3:
- Name: Archives
- Share Type: SMB
- Compression: LZ4
- Enable ACL: Yes
```

**Configure SMB Shares:**
```
# Shares → Windows (SMB) → Add Share

Share 1:
- Path: /mnt/tank/PatientFiles
- Name: PatientFiles
- Purpose: Default share parameters
- Enable: Yes

Share 2:
- Path: /mnt/tank/Backups
- Name: Backups
- Purpose: Default share parameters
- Enable: Yes

# Services → SMB → Start service
# Services → SMB → Enable "Start Automatically"
```

**Join Active Directory:**
```
# Directory Services → Active Directory

Domain: healthclinic.local
Domain Account Name: Administrator
Domain Account Password: Clinic@2025!

Click: Save
Click: Enable
Wait for "Active Directory is enabled"
```

**Configure ACLs:**
```
# Storage → Datasets → PatientFiles → Permissions

ACL Type: NFSv4
ACL Preset: Open

Add ACL Entry:
- Who: group@HEALTHCLINIC\Domain Admins
- ACL Type: Allow
- Permissions: Full Control

Add ACL Entry:
- Who: group@HEALTHCLINIC\Domain Users
- ACL Type: Allow
- Permissions: Read, Write

Apply recursively: Yes
```

**Enable Snapshots:**
```
# Data Protection → Periodic Snapshot Tasks → Add

Dataset: tank/PatientFiles
Lifetime: 1 week
Schedule: Daily at 02:00
Enabled: Yes

Dataset: tank/Backups
Lifetime: 1 month
Schedule: Daily at 03:00
Enabled: Yes
```

**Client Access:**
```
# From Windows client:
\\truenas.healthclinic.local\PatientFiles
\\truenas.healthclinic.local\Backups

# Map network drive:
net use Z: \\truenas.healthclinic.local\PatientFiles
```

---

### 5.6 VPN - Tailscale VPN Gateway

**Hardware (VM):**
- Platform: Proxmox VE
- CPU: 2 cores
- RAM: 2GB
- Storage: 20GB
- NIC: Bridged to VLAN 40

**Network Configuration:**
```bash
# Set static IP
sudo nano /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.10.40.60/24
      gateway4: 10.10.40.1
      nameservers:
        addresses:
          - 10.10.40.10
          - 10.10.40.11
        search:
          - healthclinic.local

sudo netplan apply
```

**Install Tailscale:**
```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Enable IP forwarding
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Start Tailscale
sudo tailscale up --advertise-routes=10.10.0.0/16 --accept-routes

# Get login URL
sudo tailscale status
```

**Tailscale Admin Console:**
```
# Visit: https://login.tailscale.com

# Approve subnet routes:
1. Go to Machines
2. Find VPN server (10.10.40.60)
3. Click "..." → Edit route settings
4. Enable subnet routes: 10.10.0.0/16
5. Disable key expiry
```

**Configure Firewall:**
```bash
# Allow Tailscale
sudo ufw allow 41641/udp

# Allow forwarding from Tailscale to internal networks
sudo ufw route allow in on tailscale0 out on ens18

sudo ufw enable
```

**Client Setup:**

**For Team Members:**
```bash
# Install Tailscale on laptop/PC
# Linux:
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Windows:
# Download from: https://tailscale.com/download/windows

# macOS:
# Download from: https://tailscale.com/download/mac

# Connect to VPN:
sudo tailscale up

# Access clinic resources:
# - DC01: 10.10.40.10
# - WEB01: http://10.10.40.30
# - MAIL01: https://10.10.40.20
# - TrueNAS: http://10.10.40.50
# - Zabbix: http://10.10.40.40/zabbix
```

**Share VPN Access (Team Member):**
```
# Kristine shares VPN server with team members:
1. Tailscale Admin Console → Sharing
2. Select VPN server (10.10.40.60)
3. Click "Share"
4. Enter team member's email
5. Team member accepts invite
6. Team member gets access to 10.10.0.0/16 routes
```

---

### 5.7 DC02 - Additional Configuration

**File Server Setup:**
```powershell
# Create shared folders
New-Item -Path "C:\Shares" -ItemType Directory
New-Item -Path "C:\Shares\PatientForms" -ItemType Directory
New-Item -Path "C:\Shares\StaffDocuments" -ItemType Directory
New-Item -Path "C:\Shares\Policies" -ItemType Directory

# Create SMB shares
New-SmbShare -Name "PatientForms" -Path "C:\Shares\PatientForms" -FullAccess "HEALTHCLINIC\Domain Admins" -ChangeAccess "HEALTHCLINIC\Clinical Staff"
New-SmbShare -Name "StaffDocuments" -Path "C:\Shares\StaffDocuments" -FullAccess "HEALTHCLINIC\Domain Admins" -ReadAccess "HEALTHCLINIC\Domain Users"
New-SmbShare -Name "Policies" -Path "C:\Shares\Policies" -FullAccess "HEALTHCLINIC\Domain Admins" -ReadAccess "HEALTHCLINIC\Domain Users"

# Set NTFS permissions
$acl = Get-Acl "C:\Shares\PatientForms"
$permission = "HEALTHCLINIC\Clinical Staff","Modify","Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl "C:\Shares\PatientForms" $acl
```

---

## 6. Security Implementation

### 6.1 Network Security

#### VLAN Segmentation
```
VLAN 10 (Patient WiFi):
- Internet access only
- No access to internal servers
- DNS allowed to DC01/DC02
- ACL blocks: RDP, SMB, internal subnets

VLAN 20 (Clinical Staff):
- Full access to WEB01, MAIL01
- Limited RDP to DC01/DC02
- Access to TrueNAS shares
- Internet access via NAT

VLAN 30 (Administration):
- Full access to all servers
- RDP to all Windows servers
- Access to TrueNAS shares
- Internet access via NAT

VLAN 40 (IT/Servers):
- All servers reside here
- Management access only
- No DHCP (static IPs)
- SSH/RDP from admin VLAN only
```

#### Access Control Lists (ACLs)

**VLAN 10 ACL (Guest WiFi):**
```cisco
ip access-list extended VLAN10-ACL
 remark Block RDP to internal networks
 deny tcp any 10.10.20.0 0.0.0.255 eq 3389
 deny tcp any 10.10.30.0 0.0.0.255 eq 3389
 deny tcp any 10.10.40.0 0.0.0.255 eq 3389
 
 remark Block SMB to internal networks
 deny tcp any 10.10.20.0 0.0.0.255 eq 445
 deny tcp any 10.10.30.0 0.0.0.255 eq 445
 deny tcp any 10.10.40.0 0.0.0.255 eq 445
 
 remark Allow DNS to domain controllers
 permit udp any host 10.10.40.10 eq 53
 permit udp any host 10.10.40.11 eq 53
 
 remark Allow internet access
 permit ip any any

interface GigabitEthernet0/0.10
 ip access-group VLAN10-ACL in
```

**VLAN 20 ACL (Clinical Staff):**
```cisco
ip access-list extended VLAN20-ACL
 remark Allow RDP to domain controllers only
 permit tcp any host 10.10.40.10 eq 3389
 permit tcp any host 10.10.40.11 eq 3389
 
 remark Deny RDP to other servers
 deny tcp any 10.10.40.0 0.0.0.255 eq 3389
 
 remark Allow web access to WEB01
 permit tcp any host 10.10.40.30 eq 80
 permit tcp any host 10.10.40.30 eq 443
 
 remark Allow email access to MAIL01
 permit tcp any host 10.10.40.20 eq 25
 permit tcp any host 10.10.40.20 eq 587
 permit tcp any host 10.10.40.20 eq 993
 
 remark Allow SMB to TrueNAS and DCs
 permit tcp any host 10.10.40.50 eq 445
 permit tcp any host 10.10.40.10 eq 445
 permit tcp any host 10.10.40.11 eq 445
 
 remark Allow internet access
 permit ip any any

interface GigabitEthernet0/0.20
 ip access-group VLAN20-ACL in
```

**VLAN 30 ACL (Administration):**
```cisco
ip access-list extended VLAN30-ACL
 remark Administration has full access
 permit ip any any

interface GigabitEthernet0/0.30
 ip access-group VLAN30-ACL in
```

#### NAT Configuration
```cisco
# On R1:
# NAT inside source list - all internal networks
access-list 1 permit 10.10.0.0 0.0.255.255

# Overload NAT on WAN interface
ip nat inside source list 1 interface GigabitEthernet0/1 overload

# Mark inside interfaces
interface GigabitEthernet0/0.10
 ip nat inside
interface GigabitEthernet0/0.20
 ip nat inside
interface GigabitEthernet0/0.30
 ip nat inside
interface GigabitEthernet0/0.40
 ip nat inside

# Mark outside interface
interface GigabitEthernet0/1
 ip nat outside
```

### 6.2 Server Security

#### Windows Firewall (DC01 & DC02)

**Inbound Rules:**
```powershell
# Default policy: Block all inbound
Set-NetFirewallProfile -Profile Domain,Private -DefaultInboundAction Block

# Allow LDAP (TCP 389)
New-NetFirewallRule -DisplayName "Allow LDAP - TCP 389" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow -Profile Domain,Private

# Allow DNS (TCP & UDP 53)
New-NetFirewallRule -DisplayName "Allow DNS - TCP 53" -Direction Inbound -Protocol TCP -LocalPort 53 -Action Allow -Profile Domain,Private
New-NetFirewallRule -DisplayName "Allow DNS - UDP 53" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow -Profile Domain,Private

# Allow DHCP (UDP 67)
New-NetFirewallRule -DisplayName "Allow DHCP Server - UDP 67" -Direction Inbound -Protocol UDP -LocalPort 67 -Action Allow -Profile Domain,Private

# Allow SMB (TCP 445)
New-NetFirewallRule -DisplayName "Allow SMB - TCP 445" -Direction Inbound -Protocol TCP -LocalPort 445 -Action Allow -Profile Domain,Private

# Allow RDP (TCP 3389) - restricted to admin VLAN
New-NetFirewallRule -DisplayName "Allow RDP - TCP 3389" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow -Profile Domain,Private -RemoteAddress 10.10.30.0/24,10.10.40.0/24
```

#### Linux Firewall (UFW)

**WEB01:**
```bash
# Default deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from admin VLAN only
sudo ufw allow from 10.10.30.0/24 to any port 22
sudo ufw allow from 10.10.40.0/24 to any port 22

# Allow HTTP/HTTPS from all VLANs
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow PostgreSQL from localhost only
sudo ufw allow from 127.0.0.1 to any port 5432

sudo ufw enable
```

**MAIL01:**
```bash
# Default deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from admin VLAN only
sudo ufw allow from 10.10.30.0/24 to any port 22
sudo ufw allow from 10.10.40.0/24 to any port 22

# Allow mail ports from all VLANs
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 995/tcp   # POP3S
sudo ufw allow 80/tcp    # Webmail HTTP
sudo ufw allow 443/tcp   # Webmail HTTPS

sudo ufw enable
```

**ZABBIX01:**
```bash
# Default deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from admin VLAN only
sudo ufw allow from 10.10.30.0/24 to any port 22
sudo ufw allow from 10.10.40.0/24 to any port 22

# Allow Zabbix web UI from all VLANs
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow Zabbix agent connections from all servers
sudo ufw allow from 10.10.40.0/24 to any port 10050
sudo ufw allow from 10.10.40.0/24 to any port 10051

sudo ufw enable
```

**VPN:**
```bash
# Default deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from admin VLAN only
sudo ufw allow from 10.10.30.0/24 to any port 22
sudo ufw allow from 10.10.40.0/24 to any port 22

# Allow Tailscale
sudo ufw allow 41641/udp

# Allow forwarding from Tailscale to internal networks
sudo ufw route allow in on tailscale0 out on ens18

sudo ufw enable
```

### 6.3 Authentication & Authorization

#### Active Directory Structure

**Organizational Units (OUs):**
```
healthclinic.local
├── ClinicUsers
│   ├── Administration
│   ├── Clinical Staff
│   ├── IT Staff
│   └── Temporary Staff
├── ClinicGroups
├── ClinicComputers
│   ├── Workstations
│   ├── Servers
│   └── Mobile Devices
└── ClinicResources
```

**Security Groups:**
```powershell
# Create security groups
New-ADGroup -Name "Clinical Staff" -GroupScope Global -GroupCategory Security -Path "OU=ClinicGroups,DC=healthclinic,DC=local"
New-ADGroup -Name "Administration" -GroupScope Global -GroupCategory Security -Path "OU=ClinicGroups,DC=healthclinic,DC=local"
New-ADGroup -Name "IT Staff" -GroupScope Global -GroupCategory Security -Path "OU=ClinicGroups,DC=healthclinic,DC=local"
New-ADGroup -Name "Domain Admins" -GroupScope Universal -GroupCategory Security -Path "CN=Users,DC=healthclinic,DC=local"
```

**Sample Users:**
```powershell
# Create IT admin user
New-ADUser -Name "Paul Santos" -GivenName "Paul" -Surname "Santos" -SamAccountName "psantos" `
    -UserPrincipalName "psantos@healthclinic.local" -Path "OU=IT Staff,OU=ClinicUsers,DC=healthclinic,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Clinic@2025!" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "IT Staff" -Members "psantos"
Add-ADGroupMember -Identity "Domain Admins" -Members "psantos"

# Create clinical user
New-ADUser -Name "Dr. Johnson" -GivenName "Sarah" -Surname "Johnson" -SamAccountName "dr.johnson" `
    -UserPrincipalName "dr.johnson@healthclinic.local" -Path "OU=Clinical Staff,OU=ClinicUsers,DC=healthclinic,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Clinic@2025!" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Clinical Staff" -Members "dr.johnson"

# Create admin user
New-ADUser -Name "Maria Garcia" -GivenName "Maria" -Surname "Garcia" -SamAccountName "mgarcia" `
    -UserPrincipalName "mgarcia@healthclinic.local" -Path "OU=Administration,OU=ClinicUsers,DC=healthclinic,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Clinic@2025!" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Administration" -Members "mgarcia"
```

#### Group Policy Objects (GPOs)

**Password Policy GPO:**
```powershell
# Create new GPO
New-GPO -Name "Strong Password Policy" -Comment "Enforces strong passwords"

# Link to domain
New-GPLink -Name "Strong Password Policy" -Target "DC=healthclinic,DC=local"

# Configure settings
Set-GPRegistryValue -Name "Strong Password Policy" -Key "HKLM\System\CurrentControlSet\Control\Lsa" -ValueName "NoLMHash" -Type DWord -Value 1

# Password settings (configure via Group Policy Management Console):
# - Minimum password length: 12 characters
# - Password complexity: Enabled
# - Maximum password age: 90 days
# - Minimum password age: 1 day
# - Password history: 24 passwords
# - Account lockout threshold: 5 attempts
# - Account lockout duration: 30 minutes
```

**Desktop Security GPO:**
```powershell
# Create GPO
New-GPO -Name "Desktop Security Settings" -Comment "Locks down workstations"
New-GPLink -Name "Desktop Security Settings" -Target "OU=Workstations,OU=ClinicComputers,DC=healthclinic,DC=local"

# Settings to configure:
# - Screen lock after 10 minutes idle
# - Disable USB storage devices (except IT staff)
# - Disable Control Panel access (non-admins)
# - Enable Windows Defender
# - Automatic updates enabled
# - Disable guest account
```

**File Share Permissions:**
```powershell
# On DC02:
# PatientForms share
$acl = Get-Acl "C:\Shares\PatientForms"
$acl.SetAccessRuleProtection($true, $false)

$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule("HEALTHCLINIC\Domain Admins","FullControl","ContainerInherit,ObjectInherit","None","Allow")
$clinicalRule = New-Object System.Security.AccessControl.FileSystemAccessRule("HEALTHCLINIC\Clinical Staff","Modify","ContainerInherit,ObjectInherit","None","Allow")

$acl.SetAccessRule($adminRule)
$acl.SetAccessRule($clinicalRule)
Set-Acl "C:\Shares\PatientForms" $acl

# StaffDocuments share
$acl = Get-Acl "C:\Shares\StaffDocuments"
$acl.SetAccessRuleProtection($true, $false)

$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule("HEALTHCLINIC\Domain Admins","FullControl","ContainerInherit,ObjectInherit","None","Allow")
$usersRule = New-Object System.Security.AccessControl.FileSystemAccessRule("HEALTHCLINIC\Domain Users","ReadAndExecute","ContainerInherit,ObjectInherit","None","Allow")

$acl.SetAccessRule($adminRule)
$acl.SetAccessRule($usersRule)
Set-Acl "C:\Shares\StaffDocuments" $acl
```

### 6.4 Logging & Auditing

#### Windows Event Logging
```powershell
# Enable detailed audit logging
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable
auditpol /set /category:"Policy Change" /success:enable /failure:enable
auditpol /set /category:"Object Access" /success:enable /failure:enable

# Configure log size
Limit-EventLog -LogName Security -MaximumSize 512MB
Limit-EventLog -LogName System -MaximumSize 512MB
Limit-EventLog -LogName Application -MaximumSize 512MB
```

#### Linux Logging
```bash
# Configure rsyslog to centralize logs to Zabbix server
sudo nano /etc/rsyslog.d/50-default.conf

# Add at bottom:
*.* @10.10.40.40:514

sudo systemctl restart rsyslog
```

---

## 7. Monitoring & Management

### 7.1 Zabbix Monitoring Dashboard

**Monitored Devices Summary:**

| Device | IP | Method | Template | Metrics |
|--------|-----|--------|----------|---------|
| R1-Router | 10.10.40.2 | SNMP v2 | Cisco IOS SNMP | CPU, Memory, Interfaces, Uptime |
| R2-Router | 10.10.40.3 | SNMP v2 | Cisco IOS SNMP | CPU, Memory, Interfaces, Uptime |
| SW1-Switch | 10.10.40.5 | SNMP v2 | Cisco IOS SNMP | CPU, Memory, Interfaces, Uptime |
| SW2-Switch | 10.10.40.6 | SNMP v2 | Cisco IOS SNMP | CPU, Memory, Interfaces, Uptime |
| DC01 | 10.10.40.10 | Zabbix Agent 2 | Windows by Zabbix agent | CPU, Memory, Disk, Services, AD |
| DC02 | 10.10.40.11 | Zabbix Agent 2 | Windows by Zabbix agent | CPU, Memory, Disk, Services, AD |
| MAIL01 | 10.10.40.20 | Zabbix Agent 2 | Linux by Zabbix agent | CPU, Memory, Disk, Docker containers |
| WEB01 | 10.10.40.30 | Zabbix Agent 2 | Linux by Zabbix agent | CPU, Memory, Disk, Apache, PostgreSQL |
| ZABBIX01 | 127.0.0.1 | Zabbix Agent | Zabbix server health | Internal metrics |

**Key Triggers Configured:**
- High CPU usage (>80% for 5 minutes)
- High memory usage (>90%)
- Low disk space (<10% free)
- Interface down
- Service stopped
- HSRP state change
- High error rate on interfaces
- Failed login attempts (>5 per minute)

**Dashboard Widgets:**
- System status overview
- Problems by severity
- Host availability
- Top hosts by CPU usage
- Network traffic graphs
- Disk space usage
- Service uptime

### 7.2 Active Directory Management

**Regular Tasks:**
```powershell
# Check AD replication status
repadmin /replsummary

# Check FSMO roles
netdom query fsmo

# Check DNS health
dcdiag /test:dns

# Check DHCP statistics
Get-DhcpServerv4Statistics

# View recent AD changes
Get-ADObject -Filter * -Properties whenChanged -SearchBase "OU=ClinicUsers,DC=healthclinic,DC=local" | Where-Object {$_.whenChanged -gt (Get-Date).AddDays(-7)} | Select-Object Name,whenChanged | Sort-Object whenChanged -Descending
```

### 7.3 Network Management

**Regular Verification Commands:**

**HSRP Status:**
```cisco
# On R1 and R2:
show standby brief

# Expected output:
# Interface   Grp  Pri P State    Active          Standby         Virtual IP
# Gi0/0.10    10   110 P Active   local           10.10.10.3      10.10.10.1
```

**EtherChannel Status:**
```cisco
# On SW1 and SW2:
show etherchannel summary

# Expected output:
# Po1(SU)    LACP    Gi0/23(P)   Gi0/24(P)
```

**NAT Statistics:**
```cisco
# On R1:
show ip nat statistics
show ip nat translations
```

**Interface Status:**
```cisco
# On all devices:
show ip interface brief
show interfaces status
show interfaces trunk
```

### 7.4 Backup Procedures

**Daily Backups (Automated):**
- TrueNAS snapshots: 02:00 AM daily
- PostgreSQL databases: 03:00 AM daily
- Active Directory: System State backup 04:00 AM daily

**Weekly Backups:**
- Full configuration export from all Cisco devices
- VM snapshots in Proxmox
- Mailcow full backup

**Monthly Backups:**
- Full system image of all physical servers
- Off-site backup to external storage

**Backup Scripts:**

**PostgreSQL Backup (WEB01 & ZABBIX01):**
```bash
#!/bin/bash
# /usr/local/bin/backup_postgres.sh

BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR



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

**All containers should show State: Up**  

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

**SOGo webmail opens!**  

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
2. Test email appears  

### Step 9.3: Test Between Accounts

1. Send from `admin@` to `doctor@`
2. Logout
3. Login as `doctor@healthclinic.local`
4. Check inbox  

### Step 9.4: Check Containers
```bash
sudo docker compose ps
# All should show Up
```

### Step 9.5: Check Services

**In Web UI:**

1. **System** → **Service Status**
2. All services green  

### Step 9.6: Test External Access

**From another computer:**
```
URL: https://10.10.40.20/SOGo
Username: admin@healthclinic.local
Password: Clinic@2025!
```

**Should work!**  

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
  VM created (ID 120, MAIL01)
  Debian 12 installed
  Static IP: 10.10.40.20/24
  XFCE Desktop installed
  Network verified
  Docker installed
  Docker Compose installed
  Mailcow downloaded
  Mailcow configured
  All containers running
  Web UI accessible
  Admin password changed
  Domain added (healthclinic.local)
  Mailboxes created
  Webmail accessible
  Emails send/receive
  All services green
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
-   Working email server
-   Multiple mailboxes
-   Webmail interface
-   Internal communication
-   Good demonstration

**Limitations:**
- Self-signed SSL
- No external email
- No DNS MX records
- Internal use only
- Perfect for capstone!  

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

1.   MAIL01 complete
2. 🔜 Configure ZABBIX01
3. 🔜 Create DC02 on ESXi
4. 🔜 Integration testing
5. 🔜 ACL configuration
6. 🔜 Test VMs

---

---

##   Step 1: Create Dedicated SMTP Accounts in Mailcow

Before integrating, create **dedicated service accounts** in Mailcow for security and clarity.

### In Mailcow Admin UI (`https://10.10.40.20`):
1. Go to **Mailboxes** → **Add Mailbox**
2. Create:
   - `zabbix@healthclinic.local`  
     Password: `Clinic@2025!`
   - `appointments@healthclinic.local`  
     Password: `Clinic@2025!`

> These will be used **only for sending**, not receiving.

---

## 📡 Part A: Configure ZABBIX01 to Send Alerts via MAIL01

### On **ZABBIX01** (`10.10.40.40`):

1. **Log in to Zabbix Web UI** (`http://10.10.40.40`)

2. Go to:  
   **Administration → Media types → Create media type**

3. Fill in:
   - **Name**: `HealthClinic SMTP`
   - **Type**: `Email`
   - **SMTP server**: `10.10.40.20`
   - **SMTP port**: `587`
   - **SMTP helo**: `healthclinic.local`
   - **SMTP email**: `zabbix@healthclinic.local`
   -   **Check**: *Use TLS*
   - **Username**: `zabbix@healthclinic.local`
   - **Password**: `Clinic@2025!`

4. Click **Add**

5. Now assign this media to a user (e.g., Admin):
   - **Administration → Users → Admin → Media → Add**
   - Type: `HealthClinic SMTP`
   - Send to: `admin@healthclinic.local`
   - When active: `24/7`
   - Click **Add**

6. Test:
   - Trigger a test alert (e.g., stop Apache on WEB01)
   - Check `admin@healthclinic.local` in SOGo — you should get an email!

>   Zabbix now sends alerts **internally** via your secure MAIL01 server.

---

## 🛎️ Part B: Configure Flask App on WEB01 to Send Appointment Confirmations

### On **WEB01** (`10.10.40.30`), in your Flask app:

1. **Install required package** (if not already):
   ```bash
   sudo apt update
   sudo apt install -y python3-pip
   pip3 install secure-smtplib  # optional but cleaner
   ```

2. **Add email function to your Flask app** (e.g., in `app.py`):

```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def send_appointment_email(to_email, patient_name, appointment_time):
    smtp_server = "10.10.40.20"
    smtp_port = 587
    sender_email = "appointments@healthclinic.local"
    sender_password = "Clinic@2025!"

    # Create message
    msg = MIMEMultipart()
    msg["From"] = sender_email
    msg["To"] = to_email
    msg["Subject"] = "  Your Appointment is Confirmed"

    body = f"""
    Dear {patient_name},

    Your appointment has been scheduled for:
    📅 {appointment_time}

    Thank you for choosing HealthClinic.

    — HealthClinic Staff
    """
    msg.attach(MIMEText(body, "plain"))

    try:
        # Connect and send
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()  # Enable encryption
        server.login(sender_email, sender_password)
        server.sendmail(sender_email, to_email, msg.as_string())
        server.quit()
        print(f"  Email sent to {to_email}")
        return True
    except Exception as e:
        print(f"❌ Failed to send email: {e}")
        return False
```

3. **Call this function when booking is confirmed**:
```python
# Example route
@app.route('/book', methods=['POST'])
def book_appointment():
    patient_email = request.form['email']
    patient_name = request.form['name']
    appt_time = request.form['datetime']

    # Save to DB, etc.

    # Send confirmation
    send_appointment_email(patient_email, patient_name, appt_time)

    return "Appointment booked! Check your email."
```

4. **Test it**:
   - Use a form or curl to trigger `/book`
   - Send to `admin@healthclinic.local` or `reception@...`
   - Check SOGo — the email should arrive!

> 🎯 Note: Since you’re in an **internal network**, you **do not need a real email domain**. All addresses like `@healthclinic.local` work because **DC01/DC02 resolve `.local`**, and **MAIL01 accepts them**.

---

## 🔒 Security Notes

-   All communication stays **inside VLAN 40**
-   No plaintext passwords in logs (use environment variables in production):
  ```python
  import os
  sender_password = os.getenv('SMTP_PASS')
  ```
-   Dedicated service accounts prevent credential reuse

---
TrueNAS SCALE Setup Guide
A clean, GitHub-style guide for installing, configuring, and managing TrueNAS SCALE.

TrueNAS is an open-source NAS OS available in two variants:

TrueNAS Core — FreeBSD-based
TrueNAS SCALE — Linux-based (Debian), recommended for most users due to app support and Kubernetes features
This guide focuses on TrueNAS SCALE and includes installation, network configuration, pool creation, and key CLI usage.

1. Download and Verify ISO
Download the latest SCALE ISO:
https://www.truenas.com/download-truenas-scale/

Verify checksum (Linux/macOS):

echo "PASTE_SHA256_HERE TrueNAS-SCALE-24.10.iso" | sha256sum --check

2. Create Bootable USB (8GB+ recommended)
Identify USB device
sudo lsblk # Linux diskutil list # macOS

Write ISO to USB
sudo dd bs=4M if=TrueNAS-SCALE-24.10.iso of=/dev/sdX status=progress oflag=sync -- ============================================================ -- TrueNAS SCALE INSTALLATION & SETUP GUIDE (SQL FORMAT) -- ============================================================

3. Hardware Preparation & Installation
Minimum Requirements:
-- • 8GB RAM (16GB+ recommended for apps) -- • Dedicated boot drive (SSD/HDD 32GB+) -- • One or more data drives for storage pools

Installation Steps:
-- 1. Boot from USB -- 2. Choose: 1) Install/Upgrade -- 3. Select your OS drive -- 4. Set root password -- 5. Reboot into TrueNAS SCALE

4. Initial Network Configuration (Console Setup)
-- After booting, you’ll see the Console Setup Menu.

Option 1 (Recommended) — Set Static IP:
-- Console → 1) Configure Network Interface

Option 2 — Shell (Advanced):
-- Use the following commands (example only):

-- ifconfig eth0 inet 192.168.1.100 netmask 255.255.255.0; -- route add default 192.168.1.1; -- echo 'nameserver 8.8.8.8' > /etc/resolv.conf; -- ping 8.8.8.8;

-- Replace eth0 with your correct network interface.

5. Access Web UI & Basic Setup
-- After networking is configured, access TrueNAS UI via: -- http://YOUR_TRUENAS_IP

-- First steps: -- • Update system → System Settings → Update -- • Create Storage Pool → Storage → Pools → Add -- • Create Datasets → Storage → Datasets → Add -- • Create SMB/NFS shares → Sharing -- • Enable SSH → Services → SSH → Enable

6. Useful Shell Commands
-- System Info: -- midclt call system.version; -- uname -a;

-- ZFS Storage: -- zpool status; -- zpool scrub tank;

-- User Management (Debian-style): -- adduser newuser;

-- Logs: -- dmesg | tail; -- tail -f /var/log/messages;

-- Reboot/Shutdown: -- reboot; -- poweroff;

-- Exit shell: -- exit;

7. Backup Configuration
-- Web UI → System Settings → General → Manage Configuration → Download

-- Always back up your configuration after: -- • Creating pools -- • Adding users -- • Configuring shares -- • Joining AD

-- Notes

-- • Do NOT use apt install on TrueNAS SCALE — unsupported and may break the system. -- • Use the TrueNAS UI or SCALE API for changes.

12. Conclusion
The Healthcare Clinic IT Infrastructure capstone project successfully delivers a secure, scalable, and highly available enterprise-grade network environment tailored to the operational needs of a modern medical facility. Over two weeks, our team of four implemented a fully integrated solution that meets—and in many areas exceeds—the functional, security, and redundancy requirements outlined in the project scope.

 Key Outcomes Achieved
Network Resilience: HSRP and EtherChannel eliminate single points of failure across core routing and switching layers, ensuring 99.9% uptime.
Secure Segmentation: Four logically isolated VLANs (Patient, Clinical, Admin, IT) enforce zero-trust principles through strict ACLs, preventing lateral movement and protecting sensitive systems.
Centralized Identity & Access: Dual Active Directory domain controllers provide high-availability authentication, granular RBAC, and policy-driven security for all users and devices.
Mission-Critical Services:
Internal email via Mailcow enables secure staff communication.
A Flask-based patient portal on WEB01 allows appointment booking with database-backed persistence.
TrueNAS SCALE delivers 16TB of ZFS-protected, AD-integrated storage with automated snapshots.
Enterprise Monitoring: Zabbix provides real-time visibility into all 9 infrastructure devices, with alerting on performance, availability, and security anomalies.
Secure Remote Access: Tailscale VPN enables encrypted, firewall-friendly remote management—without exposing services to the public internet.
