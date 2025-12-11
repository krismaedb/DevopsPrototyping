# Physical Topology

## Routers

**R1 (Primary Router)**
- Gi0/0 → SW1 (trunk)
- Gi0/1 → WAN (DHCP)

**R2 (Standby Router)**
- Gi0/0/0 → SW2 (trunk)
- Gi0/0/1 → WAN backup (DHCP)

## Switches

**SW1 (Primary Switch)**
- Gi0/1 → R1 Gi0/0 (trunk)
- Fa0/5 → DC01 (access VLAN40)
- Fa0/10 → Laptop/VM (VLAN40)
- Fa0/11 → Wireless AP (VLAN10)
- Fa0/12–15 → Clinical staff (VLAN20)
- Fa0/16–19 → Admin workstations (VLAN30)
- Fa0/23–24 → SW2 (EtherChannel trunk, VLAN10/20/30/40)

**SW2 (Secondary Switch)**
- Fa0/5 → DC02 (VLAN40)
- Fa0/11 → Wireless AP backup (VLAN10)
- Fa0/12–15 → Clinical staff (VLAN20)
- Fa0/16–19 → Admin workstations (VLAN30)
- Fa0/23–24 → SW1 (EtherChannel trunk)

# Logical Topology

## VLANs
- VLAN10: Patient WiFi → 10.10.10.0/24
- VLAN20: Clinical Staff → 10.10.20.0/24
- VLAN30: Admin → 10.10.30.0/24
- VLAN40: IT Servers → 10.10.40.0/24

## HSRP(Gateway Redundancy)
- VLAN10: 10.10.10.1 (R1 priority 110, R2 priority 90)
- VLAN20: 10.10.20.1
- VLAN30: 10.10.30.1
- VLAN40: 10.10.40.1

## Routing / NAT
- Both R1 and R2 do NAT for internal networks → WAN
- Access lists restrict RDP and internal traversal based on VLAN

## Servers & End Devices
- DC01 → SW1 VLAN40
- DC02 → SW VLAN10
- Clinical staf2 VLAN40
- Wireless AP →f PCs → VLAN20
- Admin PCs → VLAN30

# Healthcare Clinic Network Documentation

## Design
**Topology:** Two routers (R1 primary, R2 standby), two core switches (SW1 primary, SW2 secondary), VLAN segmentation, redundant paths via EtherChannel and VRRP.  
**Infrastructure:**  
- Routers: ISR2901 (R1), ISR4321 (R2)  
- Switches: Cisco 2960 (SW1, SW2)  
- VLANs: 10 (Patient WiFi), 20 (Clinical Staff), 30 (Admin), 40 (IT Servers)  
- Redundancy: EtherChannel, VRRP, dual routers  
**IP Addressing:** Static for servers and VLANs, DHCP relay for clients, NAT overload for internet access.

## WLAN
- VLAN10: Patient WiFi  
- APs connected to SW1/SW2  
- ACLs limit internal access, allow DNS/internet

## End-to-End Connectivity
**Internet:** NAT configured on WAN interfaces (R1 Gi0/1, R2 Gi0/0/1)  
**Host-to-Host:**  
- VLAN10: Deny RDP to internal VLANs  
- VLAN20: Limited RDP to servers  
- VLAN30: Full access  
- DHCP relay configured on all VLANs

## Segmentation
**VLANs & Subnets:**  
- VLAN10: 10.10.10.0/24  
- VLAN20: 10.10.20.0/24  
- VLAN30: 10.10.30.0/24  
- VLAN40: 10.10.40.0/24  
**ACLs:** Applied per VLAN to control access (110 = Guest, 120 = Clinical, 130 = Admin)

## Redundancy
**EtherChannel:** Port-channel1 between SW1 & SW2, trunking VLANs 10,20,30,40  
**FHRP:** R1 priority 110, R2 priority 90, VLAN interfaces with standby IP  
**Backup Routes:** WAN on both routers with NAT failover

## Management
- SNMP configured with traps (VRRP, VLAN changes, port security, system events)  
- SSH enabled on all devices for secure access

