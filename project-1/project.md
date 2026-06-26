# Project 1: Cross-Department Network Design (Cisco Packet Tracer)

## 📋 Project Prompt / Question
Design a network in Cisco Packet Tracer to connect **ACCOUNTS** and **DELIVERY** Departments through the following criteria:

*   **a.** Each Department should contain at least 2 PCs.
*   **b.** Appropriate number of switches and routers should be used in the network.
*   **c.** Using the given addresses `192.168.40.0`, all interfaces should be configured with appropriate IP addresses, subnet mask, and gateways.
*   **d.** All devices in the network should be connected using the appropriate cables.
*   **e.** Test the connectivity between ACCOUNTS and DELIVERY department. PCs in the DELIVERY department should be able to ping the PCs in the ACCOUNTS department.

---

## 🔢 Subnetting & IP Schema (Solution)
*   **Base Network Address**: `192.168.40.0`
*   **Required Subnets**: 2 (\(2^n = 2 \implies n = 1\) host bit borrowed)
*   **Custom Subnet Mask**: `255.255.255.128` (Binary: `11111111.11111111.11111111.10000000`)
*   **Block Size**: 128

### Subnet Allocation Table

| Department | Network ID | Valid Host Range | Broadcast ID | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **ACCOUNTS** | `192.168.40.0` | `192.168.40.1` – `192.168.40.126` | `192.168.40.127` | `192.168.40.1` |
| **DELIVERY** | `192.168.40.128` | `192.168.40.129` – `192.168.40.254` | `192.168.40.255` | `192.168.40.129` |

---

## 💻 Hardware Configuration Map

### 1. End Devices (PCs)
Assign static IP configurations under `Desktop > IP Configuration` in Packet Tracer:

*   **ACCOUNTS Department:**
    *   **PC-01**: `192.168.40.2` | Mask: `255.255.255.128` | Gateway: `192.168.40.1`
    *   **PC-02**: `192.168.40.3` | Mask: `255.255.255.128` | Gateway: `192.168.40.1`
*   **DELIVERY Department:**
    *   **PC-03**: `192.168.40.130` | Mask: `255.255.255.128` | Gateway: `192.168.40.129`
    *   **PC-04**: `192.168.40.131` | Mask: `255.255.255.128` | Gateway: `192.168.40.129`

### 2. Physical Cabling Map
*   **PCs to Switches**: Copper Straight-Through cable.
*   **Switches to Router**: Copper Straight-Through cable.

---

## 🛠️ Cisco IOS Router Commands

Apply these configuration steps in the Router **CLI** tab:

### Step 1: Initialize Interfaces
```text
Router> enable
Router# configure terminal
Router(config)# interface range gigabitEthernet 0/0 - 1
Router(config-if-range)# no shutdown
Router(config-if-range)# exit
```

### Step 2: Assign Layer 3 Addressing
```text
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.40.1 255.255.255.128
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 192.168.40.129 255.255.255.128
Router(config-if)# exit
```

### Step 3: Commit Configuration & Verify
```text
Router(config)# do write
Router(config)# do show startup-config
```

---

## 🧪 Verification & Testing
To confirm cross-department functionality, run the following verification checks:

1.  Open the Command Prompt on a **DELIVERY** node (e.g., `192.168.40.130`).
2.  Ping an **ACCOUNTS** host by running:
    ```cmd
    ping 192.168.40.2
    ```
3.  *Note: The initial packet may time out due to ARP resolution. Subsequent replies should return with a 100% success rate.*
