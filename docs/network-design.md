# Network Design — Corporate IT Infrastructure

## Overview

This document describes the network architecture designed 
for a mid-sized corporate environment distributed across a 
6-floor office building, hosting 120 workstations across 
two operational profiles.

The infrastructure is designed around three core principles:
**security**, **scalability**, and **manageability** — with 
clear separation between public-facing services, internal 
LAN, and management traffic.

→ Network topology diagram: [`../diagrams/network-topology.png`](../diagrams/network-topology.png)

---

## Infrastructure Overview

| Parameter | Value |
|---|---|
| Building floors | 6 |
| Total workstations | 120 (20 per floor) |
| Network zones | WAN / DMZ / Internal LAN |
| Segmentation | VLAN-based (one per floor + management + NAS) |
| Firewall architecture | Dual firewall (perimeter + internal) |
| Cabling | Cat6 structured cabling |

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
perimeter firewall.

**DMZ (Demilitarized Zone)** — isolated segment hosting 
the public-facing web server. Positioned between the 
perimeter firewall and the internal firewall, accessible 
from the internet only on port 80. The DMZ switch also 
connects IDS 1 for traffic monitoring.

**Internal LAN** — private corporate network, segmented 
into per-floor VLANs. Accessible only through the internal 
firewall. Contains the core switch, floor switches, NAS, 
IDS 2, and IPS 1.

### Traffic flow
```
Internet
    │
Perimeter firewall
    │
DMZ switch ──── Web server (DVWA)
    │       └── IDS 1 (DMZ monitor)
    │
Internal firewall
    │
Core router
    │
Core switch ──── IDS 2 (LAN monitor)
    │       └── IPS 1 ──── NAS
    │
Floor switches (×6)
    │
Workstations (20 per floor)
```

---

## VLAN Design

### Floor VLANs (user traffic — DHCP)

| VLAN Name | VLAN ID | Subnet | Netmask | Gateway | DHCP Range |
|---|---|---|---|---|---|
| Floor 1 | 101 | 192.168.101.0/26 | 255.255.255.192 | 192.168.101.1 | .20 → .62 |
| Floor 2 | 102 | 192.168.102.0/26 | 255.255.255.192 | 192.168.102.1 | .20 → .62 |
| Floor 3 | 103 | 192.168.103.0/26 | 255.255.255.192 | 192.168.103.1 | .20 → .62 |
| Floor 4 | 104 | 192.168.104.0/26 | 255.255.255.192 | 192.168.104.1 | .20 → .62 |
| Floor 5 | 105 | 192.168.105.0/26 | 255.255.255.192 | 192.168.105.1 | .20 → .62 |
| Floor 6 | 106 | 192.168.106.0/26 | 255.255.255.192 | 192.168.106.1 | .20 → .62 |

Floor switch ports are configured in **Access mode**, 
uplinks to the core switch in **Trunk mode**.

### Management VLAN (static addressing)

| VLAN Name | VLAN ID | Subnet | Netmask | Gateway |
|---|---|---|---|---|
| Management | 50 | 192.168.50.0/26 | 255.255.255.192 | 192.168.50.1 |

| Device | Role | IP Address |
|---|---|---|
| Core router | Management gateway | 192.168.50.1 |
| Core switch | Core management | 192.168.50.2 |
| Floor switch 1 | Access management | 192.168.50.3 |
| Floor switch 2 | Access management | 192.168.50.4 |
| Floor switch 3 | Access management | 192.168.50.5 |
| Floor switch 4 | Access management | 192.168.50.6 |
| Floor switch 5 | Access management | 192.168.50.7 |
| Floor switch 6 | Access management | 192.168.50.8 |
| NAS | Storage management | 192.168.50.9 |
| Perimeter firewall | Security management | 192.168.100.1 |
| Internal firewall | Security management | 192.168.150.1 |
| IDS 1 | DMZ monitoring | 192.168.50.60 |
| IDS 2 | LAN monitoring | 192.168.50.61 |
| IPS 1 | NAS protection | 192.168.50.62 |

Access to the management VLAN is restricted to authorized 
hosts via ACL. All user VLANs are blocked from reaching 
the management segment.

### NAS VLAN (static addressing)

| VLAN Name | VLAN ID | Subnet | Netmask | Gateway | IP |
|---|---|---|---|---|---|
| NAS | 90 | 192.168.90.0/26 | 255.255.255.192 | 192.168.90.1 | 192.168.90.10 (static) |

Users access the NAS exclusively via SMB for backup 
operations. Administrative access is permitted only from 
the management VLAN.

---

## Security Design

### Firewall policy — Default Deny

All traffic is blocked by default. Rules are defined 
explicitly for each permitted flow following the 
**Least Privilege** principle.

| Direction | Source | Destination | Action |
|---|---|---|---|
| Inbound (WAN → DMZ) | Any | Web server (port 80) | Allow |
| Inbound (WAN → LAN) | Any | Any | Block |
| Internal (Management → DMZ) | Management net | DMZ | Allow (TCP/UDP) |
| Internal (LAN → DMZ) | User VLANs | DMZ | Block |
| Internal (LAN → Internet) | User VLANs | Any | Allow (filtered) |
| DMZ → LAN | Web server | Any | Block |
| DMZ → Firewall mgmt | Any | Firewall (443/80/22) | Block |

### IDS/IPS placement

Three nodes are deployed at strategic points:

- **IDS 1** — connected to the DMZ switch, monitors 
  traffic between the perimeter firewall and the internal 
  network
- **IDS 2** — connected to the core switch, monitors 
  inbound traffic from the core router to the internal LAN
- **IPS 1** — deployed inline between the core switch and 
  the NAS, providing active traffic filtering and 
  protection for storage access

> In the lab environment, IDS/IPS nodes are simulated 
> using network sniffers due to hardware constraints in 
> Cisco Packet Tracer.

---

## Physical Infrastructure

### Structured cabling

- Cat6 Ethernet cabling throughout all 6 floors
- Patch panel termination per floor
- Trunk uplinks between floor switches and core switch

### Hardware bill of materials

| Component | Qty |
|---|---|
| 48-port unmanaged switch | 8 |
| Core router | 1 |
| Firewall appliance | 2 |
| Web server | 1 |
| NAS | 1 |
| IDS/IPS nodes | 3 |
| Cat6 cable (305m rolls) | 12 |
