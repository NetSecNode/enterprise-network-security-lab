# IP Addressing & Subnetting

## Requirements Analysis

Before selecting subnet sizes, the host requirements 
for each network segment were assessed:

| Segment | Required hosts | Notes |
|---|---|---|
| Floor 1–6 (each) | 20 | Workstations per floor |
| Management | ~15 | Network appliances only |
| NAS | 1 | Single storage device |
| DMZ | 1 | Web server only |

The chosen address space is **192.168.0.0/16** 
(private RFC 1918), subdivided using VLSM (Variable 
Length Subnet Masking) to allocate the minimum 
necessary address space per segment.

---

## Subnet Selection Rationale

### Floor VLANs — why /26?

Each floor requires 20 usable host addresses.

| Mask | Usable hosts | Assessment |
|---|---|---|
| /27 | 30 | Sufficient but leaves minimal growth margin |
| /26 | 62 | Covers current load with ~3× headroom |
| /25 | 126 | Excessive — wastes ~100 addresses per floor |

**/26 was selected** as the optimal balance between 
current requirements and future scalability. With 20 
workstations currently deployed, the subnet accommodates 
up to 62 hosts — allowing floor capacity to triple 
without requiring renumbering.

### Management VLAN — why /26?

The management segment hosts approximately 15 devices 
(routers, switches, firewalls, IDS/IPS nodes). A /26 
provides 62 usable addresses, sufficient for all current 
appliances with ample room for infrastructure expansion.

### NAS VLAN — why /26?

The NAS segment currently hosts a single device. A /26 
is deliberately over-provisioned to accommodate future 
storage nodes or backup appliances without subnet 
redesign.

---

## Address Space Allocation

Full map of assigned subnets within the 192.168.0.0/16 
space, ordered by network address:

| Subnet | Mask | VLAN ID | Segment | Type |
|---|---|---|---|---|
| 192.168.50.0/26 | 255.255.255.192 | 50 | Management | Static |
| 192.168.90.0/26 | 255.255.255.192 | 90 | NAS | Static |
| 192.168.100.0/26 | 255.255.255.192 | — | DMZ / Perimeter FW | Static |
| 192.168.101.0/26 | 255.255.255.192 | 101 | Floor 1 | DHCP |
| 192.168.102.0/26 | 255.255.255.192 | 102 | Floor 2 | DHCP |
| 192.168.103.0/26 | 255.255.255.192 | 103 | Floor 3 | DHCP |
| 192.168.104.0/26 | 255.255.255.192 | 104 | Floor 4 | DHCP |
| 192.168.105.0/26 | 255.255.255.192 | 105 | Floor 5 | DHCP |
| 192.168.106.0/26 | 255.255.255.192 | 106 | Floor 6 | DHCP |
| 192.168.150.0/26 | 255.255.255.192 | — | Internal FW | Static |

---

## VLAN Address Tables

### Floor VLANs — DHCP

| VLAN | ID | Subnet | Gateway | DHCP Range |
|---|---|---|---|---|
| Floor 1 | 101 | 192.168.101.0/26 | 192.168.101.1 | .20 → .62 |
| Floor 2 | 102 | 192.168.102.0/26 | 192.168.102.1 | .20 → .62 |
| Floor 3 | 103 | 192.168.103.0/26 | 192.168.103.1 | .20 → .62 |
| Floor 4 | 104 | 192.168.104.0/26 | 192.168.104.1 | .20 → .62 |
| Floor 5 | 105 | 192.168.105.0/26 | 192.168.105.1 | .20 → .62 |
| Floor 6 | 106 | 192.168.106.0/26 | 192.168.106.1 | .20 → .62 |

DHCP ranges start at .20, reserving .1–.19 for 
potential static assignments (printers, access points, 
local servers).

### Management VLAN — static addressing

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
| IPS 1 | NAS inline protection | 192.168.50.62 |

### NAS VLAN — static addressing

| Device | Role | IP Address |
|---|---|---|
| NAS | Network storage | 192.168.90.10 |
