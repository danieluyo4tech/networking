# XYZ Company - Boma Ibo Branch Office Network Design

This repository contains the complete network design, structural physical infrastructure planning, IP subnetting calculations, and Cisco IOS deployment scripts for the new standalone branch office of XYZ Company near the local village of Boma Ibo, Eastern Nigeria.

---

## 1. Project Requirements & Scope

XYZ Company manages over 2 million global customers. The Lagos branch operates independently from the corporate Headquarters (HQ) network and supports three primary departments with hardwired and wireless network access:

*   **Admin / IT**
*   **Finance / HR**
*   **Customer Service / Reception**

### Hardware Constraints
*   **1 x Core Router** (Cisco Systems Hardware)
*   **1 x Layer 2 Distribution Switch** (Cisco Systems Hardware)
*   **1 x Wireless Access Point** (For departmental SSID distribution)

---

## 2. Physical Infrastructure & Structured Cabling

### Cable Type Selection
*   **Horizontal Media:** Category 6 (Cat6) Unshielded Twisted Pair (UTP) supporting bandwidth speeds up to 10 Gbps for workstations.
*   **Backbone Media:** Cat6A Shielded Twisted Pair (STP) providing high electromagnetic interference (EMI) protection for the core distribution uplinks.

### Drop Infrastructure Allocation
*   **Admin / IT Department:** 4 RJ-45 Structural Wall Drops
*   **Finance / HR Department:** 4 RJ-45 Structural Wall Drops
*   **Customer Service / Reception:** 6 RJ-45 Structural Wall Drops

### Surveillance & Wireless Infrastructure
*   **Wireless AP Deployment:** 1 Centrally situated Cisco Business Access Point.
*   **Physical Security Assets:** 4 Power-over-Ethernet (PoE) IP Cameras connected back to the core switch.

### Head-End Rack Assembly
The main equipment stack is mounted inside a wall-enclosed **9U Network Rack** featuring a ventilation-friendly spacer configuration:
1. **1U** Blank Spacer Panel
2. **1U** Cisco Integrated Services Router
3. **1U** 24-Port Cat6 Punchdown Patch Panel
4. **1U** Cisco Catalyst 24-Port PoE Switch
5. **1U** Horizontal Cable Manager
6. **2U** Uninterruptible Power Supply (UPS) 1500VA

---

## 3. Subnetting & IP Addressing Scheme

The Internet Service Provider (ISP) assigned a Class C base network allocation of **192.168.1.0**.

### Subnet Derivation Formula
*   **Required Segments:** 3
*   **Mathematical Formula:** $2^n \ge 3 \implies 2^2 = 4 \ge 3$
*   **Borrowed Bits ($n$):** 2 Bits
*   **Resulting Mask:** 255.255.255.192 (`/26`)
*   **Block Increment Increment Size:** $256 - 192 = 64$

### Network Allocation Matrix

| Department / Segment | VLAN ID | Network ID | Usable Host Range | Broadcast ID | Subnet Mask |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Admin / IT** | VLAN 10 | `192.168.1.0` | `192.168.1.1` - `192.168.1.62` | `192.168.1.63` | `255.255.255.192` (`/26`) |
| **Finance / HR** | VLAN 20 | `192.168.1.64` | `192.168.1.65` - `192.168.1.126` | `192.168.1.127` | `255.255.255.192` (`/26`) |
| **Customer Service** | VLAN 30 | `192.168.1.128` | `192.168.1.129` - `192.168.1.190` | `192.168.1.191` | `255.255.255.192` (`/26`) |
| *Future Spare Expansion*| *VLAN 40* | *192.168.1.192*| *192.168.1.193* - *192.168.1.254*| *192.168.1.255*| `255.255.255.192` (`/26`) |

---

## 4. Cisco IOS Configuration Scripts

The implementation uses **Router-on-a-Stick (RoaS)** logical topology to process secure, multi-departmental communications across a single 802.1Q hardware link.

### A. Layer 2 Switch Setup
```ios
enable
configure terminal

! Create Logical VLAN Networks
vlan 10
 name Admin_IT
vlan 20
 name Finance_HR
vlan 30
 name Customer_Service
exit

! Access Port Assignments for Workstations
interface range fa 0/2 - 4
 switchport mode access
 switchport access vlan 10
exit

interface range fa 0/5 - 7
 switchport mode access
 switchport access vlan 20
exit

interface range fa 0/8 - 10
 switchport mode access
 switchport access vlan 30
exit

! Uplink Core Trunk Connection to Router
interface fa 0/1
 switchport mode trunk
 description Link_to_Router_Trunk
exit

do write
```

### B. Router Sub-Interface Setup (Inter-VLAN Gateway)
```ios
enable
configure terminal

! Bring up primary interface link
interface gi 0/0
 no shutdown
exit

! Sub-Interface Deployment Matrix
interface gi 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.192
exit

interface gi 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.65 255.255.255.192
exit

interface gi 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.1.129 255.255.255.192
exit
```

### C. Dynamic Allocation Engine (DHCP Services)
```ios
configure terminal
service dhcp

! Exclude Gateways from Distribution Pools
ip dhcp excluded-address 192.168.1.1
ip dhcp excluded-address 192.168.1.65
ip dhcp excluded-address 192.168.1.129

! Pool Generation Sequence
ip dhcp pool Admin-Pool
 network 192.168.1.0 255.255.255.192
 default-router 192.168.1.1
 dns-server 192.168.1.1
 domain-name Admin.com
exit

ip dhcp pool Finance-Pool
 network 192.168.1.64 255.255.255.192
 default-router 192.168.1.65
 dns-server 192.168.1.65
 domain-name Finance.com
exit

ip dhcp pool Customer-Pool
 network 192.168.1.128 255.255.255.192
 default-router 192.168.1.129
 dns-server 192.168.1.129
 domain-name Customer.com
exit

do write
```

---

## 5. Verification Framework

To audit the environment configuration, log in to the diagnostic plane and run the following verification commands:

```bash
# Check physical links and logical sub-interface states
show ip interface brief

# Confirm switch VLAN assignment ports
show vlan brief

# Verify that the trunk link is functioning
show interfaces trunk

# Verify dynamic client leases
show ip dhcp binding
```
