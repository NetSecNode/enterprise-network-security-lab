# Enterprise network security lab — Corporate Infrastructure Assessment

> A hands-on network security project simulating the design, 
> deployment and assessment of a corporate IT infrastructure 
> in a controlled lab environment.

## Overview

This project covers the end-to-end process of designing a 
secure corporate network for a mid-sized company, followed 
by active security testing of the deployed infrastructure.

The lab environment replicates a realistic 6-floor office 
building with 120 endpoints, a segmented DMZ, perimeter 
firewall, NAS storage, and IDS/IPS nodes.

All security tools used for testing were written from scratch 
in Python — no pre-built scanners or frameworks were used 
for the assessment phase.

---

## Network Architecture

| Component         | Details                                      |
|-------------------|----------------------------------------------|
| Floors            | 6                                            |
| Endpoints         | 20 workstations per floor (120 total)        |
| Switches          | 1 per floor (6 total)                        |
| Core Router       | 1 central router aggregating all floors      |
| Perimeter Firewall| 1 (between internal network and Internet)    |
| DMZ Firewall      | 1 (isolating the web server)                 |
| Web Server        | DVWA on Metasploitable2 (simulated target)   |
| NAS               | 1 (connected to ground floor switch)         |
| IDS/IPS           | 3 nodes on internal perimeter                |

→ Full topology diagram: [`diagrams/network-topology.png`](diagrams/network-topology.png)  
→ Design rationale: [`docs/network-design.md`](docs/network-design.md)

---

## IP Addressing & Subnetting

The network address space was designed using VLSM (Variable 
Length Subnet Masking) to optimize IP allocation across the 
6-floor infrastructure, minimizing address waste while 
maintaining clear segment boundaries.

→ Full subnetting design: [`docs/subnetting.md`](docs/subnetting.md)

---

## Security Tools (custom-built)

### 1. Port Scanner
A lightweight TCP port scanner written in Python.  
Accepts an IP address and port range as input, returns 
open/closed status for each port.
```
Usage: python scanner.py --host <IP> --start <port> --end <port>
```

→ [`tools/port-scanner/`](tools/port-scanner/)

---

### 2. HTTP Method Checker
A Python tool to enumerate which HTTP methods (GET, POST, 
PUT, DELETE, OPTIONS...) are enabled on a given URL path.
```
Usage: python http_checker.py --url <URL> --path <path>
```

→ [`tools/http-method-checker/`](tools/http-method-checker/)

---

### 3. Socket Sniffer
A raw socket packet capture tool written in Python.  
Captures and displays network-level traffic for analysis.

→ [`tools/socket-sniffer/`](tools/socket-sniffer/)

---

## Assessment Results

- **Port Scan:** open ports identified on the target, 
  with security recommendations  
- **HTTP Methods:** methods enumerated on `/phpmyadmin` 
  and other paths, with risk analysis

→ Full report: [`docs/security-report.md`](docs/security-report.md)  
→ Raw outputs: [`results/`](results/)

---

## Lab Environment

| Role              | System                  |
|-------------------|-------------------------|
| Attacker machine  | Kali Linux              |
| Target / Web Server | Metasploitable2 (DVWA)|
| Network simulation| [Cisco Packet Tracer / GNS3 / draw.io] |

> ⚠️ All tests were performed in an isolated lab environment. 
> No real systems were targeted.

---

## Tech Stack

- Python 3.x
- `socket` (stdlib)
- `requests`
- Kali Linux
- Metasploitable2

---

## Author

**[LC]** — [NetSecNode](https://github.com/NetSecNode)
