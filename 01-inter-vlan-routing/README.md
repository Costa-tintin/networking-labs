# Lab 01 — Inter-VLAN Routing: Router-on-a-Stick vs Layer 3 Switch (SVI)

**Author:** u/Trunked_Telemetry  
**Tools:** Cisco Packet Tracer 8.2+  
**Difficulty:** Beginner to Intermediate  
**Date:** May 2026  
---

## What This Lab Is About

I built the same SME network two different ways to understand what 
actually gets deployed in real offices — and what the tradeoffs are 
from a cost, performance, and management perspective.

The Africa/Kenya market context matters here. A used Cisco 2911 
router costs roughly $200-250 in Nairobi. A used Cisco 3650 
multilayer switch costs $500-600. That price difference shapes 
real deployment decisions in ways that the standard Cisco 
curriculum does not address.

---

## Topology 1 — Router-on-a-Stick (Small SME, under 30 users)

### Devices Used
| Device | Model | Role |
|--------|-------|------|
| Router | Cisco 2911 (R1-SME) | Inter-VLAN routing + DHCP + NAT + Internet |
| Switch | Cisco 2960 (SW1-ACCESS) | Layer 2 access switch with 802.1Q trunk |
| PCs | PC-PT x4 | HR-PC1, HR-PC2, Sales-PC1, Sales-PC2 |
| Servers | Server-PT x2 | DNS-SERVER (10.10.80.2), Web-Server (10.10.80.3) |
| ISP | Server-PT | Simulated internet — 203.0.113.1 |

### IP Addressing
| Device | Interface | IP Address | Subnet | Gateway |
|--------|-----------|------------|--------|---------|
| R1-SME | Gig0/0.10 | 10.10.10.1 | /24 | — |
| R1-SME | Gig0/0.20 | 10.10.20.1 | /24 | — |
| R1-SME | Gig0/0.80 | 10.10.80.1 | /24 | — |
| R1-SME | Gig0/1 (WAN) | 203.0.113.2 | /30 | 203.0.113.1 |
| HR-PC1 | Fa0 | 10.10.10.10 (DHCP) | /24 | 10.10.10.1 |
| HR-PC2 | Fa0 | 10.10.10.11 (DHCP) | /24 | 10.10.10.1 |
| Sales-PC1 | Fa0 | 10.10.20.10 (DHCP) | /24 | 10.10.20.1 |
| Sales-PC2 | Fa0 | 10.10.20.11 (DHCP) | /24 | 10.10.20.1 |
| DNS-SERVER | Fa0 | 10.10.80.2 (static) | /24 | 10.10.80.1 |
| Web-Server | Fa0 | 10.10.80.3 (static) | /24 | 10.10.80.1 |
| ISP Server | Fa0 | 203.0.113.1 (static) | /30 | — |

### VLAN Structure
| VLAN | Name | Subnet | Ports on SW1-ACCESS |
|------|------|--------|---------------------|
| 10 | HR | 10.10.10.0/24 | Fa0/1, Fa0/2 |
| 20 | SALES | 10.10.20.0/24 | Fa0/3, Fa0/4 |
| 80 | SERVERS | 10.10.80.0/24 | Fa0/5, Fa0/6 |

### How It Works
The switch sends all VLAN traffic up a single 802.1Q trunk to the 
router. The router uses subinterfaces — one per VLAN — to route 
between them. All traffic between HR and Sales physically leaves 
the switch, goes to the router, and comes back. This is the 
"stick" in router-on-a-stick.


