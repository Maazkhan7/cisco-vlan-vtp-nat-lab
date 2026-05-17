# 🔧 Cisco Packet Tracer — VLAN, VTP, DHCP & NAT/PAT Full Lab

> A complete enterprise-style network simulation built in Cisco Packet Tracer, covering VLAN segmentation, VTP domain propagation, per-VLAN DHCP, inter-VLAN routing, and NAT/PAT internet access.

📹 **Full configuration walkthrough on YouTube:** [Watch Here](#) *(replace with your link)*

---

## 📋 Table of Contents

- [Topology Overview](#topology-overview)
- [Network Design](#network-design)
- [Technologies Used](#technologies-used)
- [Configuration Summary](#configuration-summary)
  - [VTP Setup](#1-vtp-setup)
  - [VLAN Configuration](#2-vlan-configuration)
  - [Trunk Links](#3-trunk-links)
  - [DHCP Server](#4-dhcp-server)
  - [Inter-VLAN Routing](#5-inter-vlan-routing)
  - [NAT/PAT](#6-natpat)
- [Verification](#verification)
- [Files in This Repo](#files-in-this-repo)
- [How to Open](#how-to-open)
- [Author](#author)

---

## Topology Overview

```
                          DHCP-SERVER
                               |
  [PC0–PC19]              [Router0] ── NAT/PAT ── [Router1/ISP]
      |                        |
  [SW0]──[SW1]──[SW2]──[SW3]──[SW4]──[SW5]──[SW6]──[SW7]──[SW8]──[SW9/CORE]
   (Client)                                                          (VTP Server)

  VLAN 10 = Cyan PCs  — 192.168.1.0/24
  VLAN 20 = Yellow PCs — 192.168.2.0/24
```

- **10 × Cisco 2950-24 switches** in a daisy-chain trunk topology
- **20 PCs** — alternating VLAN 10 and VLAN 20 across all switches
- **1 DHCP Server** — serving both VLANs with separate pools
- **2 Routers (Cisco 2911)** — Router0 handles inter-VLAN routing + NAT; Router1 simulates ISP

---

## Network Design

| Component | Details |
|-----------|---------|
| Switches | 10 × Cisco 2950-24 (SW0–SW9) |
| Routers | 2 × Cisco 2911 (Router0, Router1/ISP) |
| PCs | 20 × PC-PT (PC0–PC19) |
| Server | 1 × Server-PT (DHCP Server) |
| VTP Domain | `core` |
| VTP Version | 2 |
| VTP Server | Switch9 (hostname: CORE) |
| VTP Clients | Switch0–Switch8 |
| VLAN 10 | 192.168.1.0/24 — Gateway: 192.168.1.1 |
| VLAN 20 | 192.168.2.0/24 — Gateway: 192.168.2.1 |
| DNS Server | 8.8.8.8 |

---

## Technologies Used

- **VTP (VLAN Trunking Protocol)** — centralized VLAN management
- **IEEE 802.1Q Trunking** — VLAN tagging across switch links
- **DHCP** — automatic IP assignment per VLAN
- **Inter-VLAN Routing** — router-on-a-stick or routed subinterfaces
- **NAT/PAT** — overloaded NAT for internet simulation
- **Cisco IOS CLI** — all configuration via command line

---

## Configuration Summary

### 1. VTP Setup

**On Switch9 (VTP Server):**
```bash
vtp domain core
vtp mode server
vtp version 2
```

**On Switch0–Switch8 (VTP Clients):**
```bash
vtp domain core
vtp mode client
vtp version 2
```

**Verify:**
```bash
show vtp status
show vtp counters
```

Expected output — Configuration Revision: 11, Number of existing VLANs: 7, zero config/digest errors.

---

### 2. VLAN Configuration

*Created only on the VTP Server (Switch9) — propagated automatically to all clients.*

```bash
vlan 10
 name VLAN10
vlan 20
 name VLAN20
```

**Assign access ports:**
```bash
interface FastEthernet0/X
 switchport mode access
 switchport access vlan 10   ! or vlan 20
```

---

### 3. Trunk Links

*On every inter-switch link:*
```bash
interface FastEthernet0/X
 switchport mode trunk
```

---

### 4. DHCP Server

Configured on the Server-PT (DHCP-SERVER):

| Pool Name | Default Gateway | Start IP | Subnet Mask | Max Users |
|-----------|----------------|----------|-------------|-----------|
| VLAN 10 | 192.168.1.1 | 192.168.1.1 | 255.255.255.0 | 20 |
| VLAN 20 | 192.168.2.1 | 192.168.2.1 | 255.255.255.0 | 20 |

---

### 5. Inter-VLAN Routing

*On Router0 — subinterfaces for each VLAN:*
```bash
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.2.1 255.255.255.0
```

---

### 6. NAT/PAT

```bash
ip access-list standard NAT_ACL
 permit 192.168.1.0 0.0.0.255
 permit 192.168.2.0 0.0.0.255

ip nat inside source list NAT_ACL interface GigabitEthernet0/1 overload

interface GigabitEthernet0/0
 ip nat inside

interface GigabitEthernet0/1
 ip nat outside
```

---

## Verification

### VTP propagation confirmed
```
show vtp status
→ Configuration Revision: 11
→ Number of existing VLANs: 7
→ VTP Operating Mode: Server / Client
→ Zero config errors, zero digest errors
```

### DHCP + connectivity confirmed
```
C:\>ping 192.168.1.2    ! Same-VLAN ping
→ 4/4 packets received, TTL=128

C:\>ping 192.168.2.2    ! Cross-VLAN ping (TTL=127 confirms routing hop)
→ 4/4 packets received, TTL=127
```

---

## Files in This Repo

```
cisco-vlan-vtp-nat-lab/
├── README.md
├── lab.pkt                  # Cisco Packet Tracer file
├── screenshots/
│   ├── topology.png
│   ├── vtp-status-server.png
│   ├── vtp-status-client.png
│   ├── vtp-counters.png
│   ├── dhcp-pools.png
│   └── ping-test.png
```

---

## How to Open

1. Download and install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco NetAcad account)
2. Clone this repo:
   ```bash
   git clone https://github.com/YOUR_USERNAME/cisco-vlan-vtp-nat-lab.git
   ```
3. Open `lab.pkt` in Packet Tracer
4. Explore the topology — all configurations are already applied

---

## Author

**Your Name**
- YouTube: [Your Channel](#)
- LinkedIn: [Your Profile](#)
- GitHub: [@YourUsername](#)

---

*If this lab helped you prepare for your CCNA, drop a ⭐ on the repo!*
