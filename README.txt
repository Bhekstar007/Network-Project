# MASA Project – Enterprise Network Infrastructure

## Overview

This project designs and implements a multi-site, multi‑VLAN enterprise network for **MASAproject.com**.  
The network serves R&D, Engineering, Science, IT, Security groups and a server farm, using advanced routing, switching, and security features. All devices are configured with SSH, role‑based access, and dynamic IP allocation.

**Key objectives:**
- Efficient **VLSM** addressing using a `10.4.0.0/16` backbone.
- Inter‑VLAN routing via **router‑on‑a‑stick** and **OSPF**.
- High availability through **EtherChannel** and trunking.
- Secure remote management (SSH, local AAA, banners).
- Automatic IP assignment via **DHCP pools** on the router.

---

## Network Topology

![Topology](Topology.png)

The topology includes:
- **Headquarters (HQ)** – management VLAN 80, security group, admin devices.
- **R&D_S1 / R&D_S2 / R&D_S3** – access switches for IT, Engineering, Science groups.
- **Server_Sw** – dedicated switch for Web, Database, File, Mail, Print servers.
- **R&D_Router** – central router connecting all VLANs, WAN (Frame Relay), and providing DHCP.
- **HQ_Router** - central router connecting all VLANs, WAN (Frame Relay), and providing DHCP.

---

## IP Addressing Scheme (VLSM)

The `10.4.0.0/16`  address space is subnetted using **Variable Length Subnet Masking** to meet host requirements with minimal waste.

| Subnet Name       | Required Hosts | CIDR | Subnet Mask         | Network Address   | Usable Range                |
|-------------------|----------------|------|---------------------|-------------------|-----------------------------|
| R&D‑Sci           | 198            | /24  | 255.255.255.0       | 10.4.0.0/24       | 10.4.0.1   – 10.4.0.254     |
| OPS‑ENG           | 154            | /24  | 255.255.255.0       | 10.4.1.0/24       | 10.4.1.1   – 10.4.1.254     |
| Train‑Eng         | 132            | /24  | 255.255.255.0       | 10.4.2.0/24       | 10.4.2.1   – 10.4.2.254     |
| HQ (Admin)        | 110            | /25  | 255.255.255.128     | 10.4.3.0/25       | 10.4.3.1   – 10.4.3.126     |
| SEC (Security)    | 110            | /25  | 255.255.255.128     | 10.4.3.128/25     | 10.4.3.129 – 10.4.3.254     |
| R&D‑ENG           | 66             | /25  | 255.255.255.128     | 10.4.4.0/25       | 10.4.4.1   – 10.4.4.126     |
| IT                | 22             | /27  | 255.255.255.224     | 10.4.4.128/27     | 10.4.4.129 – 10.4.4.158     |
| **Serial links**  | 2              | /24  | 255.255.255.0       | 10.4.5.0/24       | point‑to‑point assignments  |
| **Server Farm**   | Various        | /24  | 255.255.255.0       | 10.4.6.0/24 – 10.4.10.0/24  | 10.4.X.1 – 10.4.X.254      |

> Full VLSM details are in `Project VSLM.xlsx`.

---

## Technologies Used

| Category           | Implementation                                                                 |
|--------------------|--------------------------------------------------------------------------------|
| **Switching**      | VLANs (5, 25, 60, 80, 100, 180, 200, 300, 400, 500)                           |
|                    | Access / Trunk ports                                                           |
|                    | EtherChannel (LACP – active/passive) on port‑channels 1,2,3                    |
| **Routing**        | Router‑on‑a‑stick (802.1Q subinterfaces)                                       |
|                    | OSPF area 0 (passive interfaces on LAN, active point‑to‑point on serial links) |
| **WAN**            | Frame Relay (IETF encapsulation, static maps, no inverse‑ARP)                  |
| **Management**     | SSH (RSA 1024), local username/password, privileged secret, banner MOTD        |
| **DHCP**           | Excluded addresses + pools for all subnets (Science, Engineering, IT, Security, server VLANs) |

---

## Configuration Highlights

### 1. VLAN & Access Mapping
- **IT Group** → VLAN 5  – ports f0/1-2 on all R&D_Switches
- **Engineering Group** → VLAN 60 – ports f0/7-10
- **Science Group** → VLAN 180 – ports f0/11-14
- **Security Group** → VLAN 25 – port g0/2 (R&D_S3)
- **Server_Sw** → ports f0/1-5 (Web‑Server VLAN 100), f0/6-10 (DB‑Server VLAN 200), etc.
- **Management** → VLAN 80 (SVI: 10.4.4.145,146,147,148)

### 2. EtherChannel Trunks
- Between R&D_S1 – R&D_S3: ports f0/3-4 → **channel‑group 1**
- Between R&D_S2 – R&D_S3: ports f0/3-4 → **channel‑group 3**
- Between R&D_S1 – R&D_S2: ports f0/5-6 → **channel‑group 2**
- Modes: `active`/`passive` (LACP)

### 3. Router (R&D_Router) – Subinterfaces