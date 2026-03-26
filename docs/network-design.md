# Network Design — Corporate IT Infrastructure

## Overview

This document describes the network architecture designed 
for a mid-sized corporate environment distributed across a 
6-floor office building, hosting 120 workstations across 
two operational profiles.

The infrastructure is built around three core principles:
**security**, **scalability**, and **manageability** — with 
clear separation between public-facing services, internal 
LAN, and management traffic.

→ Network topology: [`../diagrams/network-topology.jpg`](../diagrams/network-topology.jpg)  
→ IP addressing and subnetting: [`subnetting.md`](subnetting.md)

---

## Infrastructure Overview

| Parameter | Value |
|---|---|
| Building floors | 6 |
| Total workstations | 120 (20 per floor) |
| Network zones | WAN / DMZ / Internal LAN |
| Segmentation | VLAN-based (per floor + management + NAS) |
| Firewall architecture | Dual firewall (perimeter + internal) |
| Cabling standard | Cat6 structured cabling |

---

## Endpoint Profiles

Two hardware profiles are defined based on workload 
requirements:

| Profile | Use case | CPU | RAM | Storage | Qty |
|---|---|---|---|---|---|
| Administrative | Office, admin, secretarial | Intel i5 | 16 GB | 512 GB SSD | 90 |
| Engineering | Design, management, rendering | Intel i7 | 32 GB | 1 TB SSD + GPU | 30 |

All endpoints run Windows 11 Pro. Engineering workstations 
include a dedicated GPU for graphics acceleration.

---

## Network Architecture

### Zone segmentation

The infrastructure is divided into three distinct zones:

**WAN** — external internet connection, terminated at the 
perimeter firewall. No direct access to internal resources 
is permitted from this zone.

**DMZ (Demilitarized Zone)** — isolated segment hosting 
the public-facing web server. Positioned between the 
perimeter firewall and the internal firewall, reachable 
from the internet exclusively on port 80. IDS 1 is 
connected to the DMZ switch for traffic monitoring.

**Internal LAN** — private corporate network, segmented 
into per-floor VLANs. Accessible only through the internal 
firewall. Contains the core switch, floor switches, NAS 
storage, IDS 2, and IPS 1.

### Traffic flow
```
Internet
    │
Perimeter firewall
    │
DMZ switch ──── Web server
    │       └── IDS 1
    │
Internal firewall
    │
Core router
    │
Core switch ──── IDS 2
    │       └── IPS 1 ──── NAS
    │
Floor switches (×6)
    │
Workstations (20 per floor)
```

---

## VLAN Design

The internal LAN is logically segmented into 9 VLANs:

- 6 **user VLANs** (one per floor) with dynamic DHCP 
  addressing — isolate broadcast domains and limit 
  lateral movement in case of compromise
- 1 **management VLAN** with static addressing — 
  dedicated exclusively to network appliance administration
- 1 **NAS VLAN** with static addressing — separates 
  storage traffic from both user and management traffic
- 1 **DMZ segment** — isolates public-facing services 
  from the internal network

Floor switch ports are configured in **Access mode**. 
Uplinks to the core switch use **Trunk mode** to carry 
all required VLANs. Inter-VLAN routing is handled 
centrally by the core router.

→ Full IP addressing scheme and subnet calculations: 
[`subnetting.md`](subnetting.md)

---

## Security Design

### Firewall policy — Default Deny

All traffic is blocked by default. Permit rules are 
defined explicitly for each required flow, following the 
**Least Privilege** principle — every user and service 
has access only to the resources strictly necessary for 
its function.

| Direction | Source | Destination | Dst Port | Action |
|---|---|---|---|---|
| WAN → DMZ | Any | Web server | 80 | Allow |
| WAN → LAN | Any | Any | Any | Block |
| Management → DMZ | Management net | DMZ | Any | Allow |
| Developers → DMZ | Developers net | Web server only | Any | Allow |
| User VLANs → DMZ | User VLANs | Any | Any | Block |
| User VLANs → Internet | User VLANs | Any | Any | Allow (filtered) |
| DMZ → LAN | Any | Any | Any | Block |
| DMZ → Firewall mgmt | Any | Firewall | 443/80/22 | Block |

The DMZ-to-LAN block ensures that a compromised public 
service cannot initiate connections toward internal 
resources. The firewall self-protection rule prevents 
an attacker who gains access to the web server from 
reaching the firewall management interface.

### IDS/IPS placement

Three nodes are deployed at strategic points within 
the infrastructure:

**IDS 1** — connected to the DMZ switch. Monitors all 
traffic transiting the DMZ segment from three directions: 
inbound requests from the WAN through the perimeter 
firewall, outbound responses from the web server, and 
traffic originating from the internal LAN through the 
internal firewall. This positioning provides full 
visibility on the DMZ segment, covering both external 
attack attempts targeting public-facing services and 
anomalous lateral traffic initiated from within the 
internal network.

**IDS 2** — connected to the core switch. Monitors 
traffic entering the internal LAN from the core router, 
detecting anomalous patterns before they propagate 
across floor VLANs. Provides visibility on inter-VLAN 
traffic and any suspicious activity crossing segment 
boundaries within the internal network.

**IPS 1** — deployed inline between the core switch and 
the NAS. Unlike the IDS nodes, IPS 1 operates in active 
mode — it does not only detect but actively blocks 
unauthorized or anomalous connections before they reach 
the NAS, protecting stored data from both external 
threats that have bypassed the firewall and internal 
lateral movement attempts.

---

## Physical Infrastructure

### Structured cabling

The physical layer uses Cat6 Ethernet cabling throughout 
all 6 floors, with patch panel termination per floor and 
organised rack cabling. Trunk uplinks connect floor 
switches to the core switch.

### Hardware bill of materials

| Component | Qty |
|---|---|
| 48-port switch | 8 |
| Core router | 1 |
| Firewall appliance | 2 |
| Web server | 1 |
| NAS | 1 |
| IDS/IPS nodes | 3 |
| Cat6 cable (305 m rolls) | 12 |

> 8 switches are deployed across the infrastructure: 
> 6 floor switches (one per floor), 1 core switch, 
> and 1 DMZ switch.
