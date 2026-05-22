# 02 — Enterprise 3-Tier Network Lab

A fully working enterprise network built in Cisco Packet Tracer.
Built by someone curious about how real networks work — not an expert.
Every error documented. Every fix explained.

![Full Topology](screenshots/01-full-topology.png)

---

## What This Lab Covers

| Layer | Devices | Purpose |
|-------|---------|---------|
| Core | 2x Cisco 3650 (Core-SW1, Core-SW2) | Routing, HSRP, OSPF |
| Distribution | 2x Cisco 2960 (Dist-SW1, Dist-SW2) | STP root, LACP uplinks |
| Access | 4x Cisco 2960 (Acc-SW1 to SW4) | End device connectivity |
| Security | ASA 5506-X | Firewall, NAT/PAT |
| WAN | Cisco 1941 Router | Simulated ISP |
| Wireless | WLC-3504 + Lightweight AP | Wi-Fi management |
| End Devices | PCs, IP Phones, Laptop, Servers | Users and services |

---

## Protocols Implemented

### VLANs + Trunking
Traffic segmented by department.
Each department gets its own broadcast domain.

| VLAN | Name | Subnet | Gateway (HSRP VIP) |
|------|------|--------|-------------------|
| 10 | SALES | 10.10.10.0/24 | 10.10.10.1 |
| 20 | HR | 10.20.20.0/24 | 10.20.20.1 |
| 30 | ENGINEERING | 10.30.30.0/24 | 10.30.30.1 |
| 50 | WIRELESS-MGMT | — | — |
| 99 | SERVERS | 10.99.99.0/24 | 10.99.99.254 |
| 100 | VOICE | — | — |
| 999 | NATIVE-UNUSED | — | security only |

### LACP — Link Aggregation
Two physical cables bundled into one logical link.
Double bandwidth + redundancy between Core and Distribution.
Core-SW1 Gig1/0/3 ─┐
├── Port-channel1 (2Gbps logical)
Core-SW1 Gig1/0/4 ─┘
│
Dist-SW1 Gig0/1  ─┐
├── Port-channel1
Dist-SW1 Gig0/2  ─┘

### HSRP — Gateway Redundancy
Two core switches share one virtual IP.
PCs use the virtual IP as default gateway.
If Core-SW1 fails, Core-SW2 takes over in under 2 seconds.
Core-SW1 (Active)  priority 110 — real IP 10.10.10.2
Core-SW2 (Standby) priority 90  — real IP 10.10.10.3
Virtual IP         10.10.10.1   — what PCs use as gateway

### OSPF — Dynamic Routing
Core switches learn each others routes automatically.
No static routes needed between VLANs.
Default route from ISP distributed to all switches.

### DHCP Relay — ip helper-address
One central DHCP server serves all VLANs.
Core switch SVIs relay broadcasts to server using
ip helper-address 10.99.99.1

### ASA Firewall — NAT/PAT
All inside traffic NATed to one public IP.
Inbound connections blocked by default.
ICMP, DNS, HTTP inspection enabled.

### Port Security
Access ports locked to one MAC address.
Sticky learning — first device that connects is remembered.
Violation mode restrict — drops and logs unauthorized MACs.

---

## How to Run This Lab

1. Download Cisco Packet Tracer free from
   https://www.netacad.com/cisco-packet-tracer
2. Clone this repo
3. Open topology/enterprise-lab.pkt
4. All devices are pre-configured
5. Test DHCP: click any PC → Desktop → IP Config → DHCP
6. Test routing: PC Command Prompt → ping 10.20.20.10
7. Test internet: ping 203.0.113.1

---

## CLI Configs

All device configurations saved in /cli-configs/

| File | Device | Key Config |
|------|--------|-----------|
| core-sw1.txt | Core-SW1 | HSRP active, OSPF, LACP active, ip helper |
| core-sw2.txt | Core-SW2 | HSRP standby, OSPF, LACP active |
| dist-sw1.txt | Dist-SW1 | STP root VLANs 10+30, LACP passive |
| dist-sw2.txt | Dist-SW2 | STP root VLANs 20+100, LACP passive |
| acc-sw1.txt | Acc-SW1 | VLANs, port security, WLC trunk |
| acc-sw2.txt | Acc-SW2 | VLANs, voice VLAN, HR ports |
| acc-sw3.txt | Acc-SW3 | VLANs, engineering ports |
| acc-sw4.txt | Acc-SW4 | Server ports VLAN 99 |
| asa-fw.txt | ASA-FW | NAT/PAT, inspection policy |
| router-isp.txt | Router-ISP | Simulated internet |

---

## Screenshots

| # | Screenshot | What it Shows |
|---|-----------|---------------|
| 01 | Full topology | All devices and connections |
| 02 | VLAN brief | VLANs created on Acc-SW1 |
| 03 | Trunk status | Po1 trunking with correct VLANs |
| 04 | STP root | Dist-SW1 is root for VLAN 10 |
| 05 | STP ports | Port states at access layer |
| 06 | LACP Core-SW1 | Po1(SU) bundled and in use |
| 07 | LACP Dist-SW1 | Po1(SU) bundled and in use |
| 08 | HSRP Active | Core-SW1 Active priority 110 |
| 09 | HSRP Standby | Core-SW2 Standby priority 90 |
| 10 | Route table SW1 | Connected + static + OSPF routes |
| 11 | Route table SW2 | Routes learned |
| 12 | OSPF interfaces | Active OSPF interfaces |
| 13 | DHCP pools | All 3 pools on Server-DHCP |
| 14 | PC-Sales DHCP | IP 10.10.10.10 auto-assigned |
| 15 | PC-HR DHCP | IP 10.20.20.10 auto-assigned |
| 16 | PC-ENG DHCP | IP 10.30.30.10 auto-assigned |
| 17 | Cross-VLAN ping | Sales pinging HR and Eng |
| 18 | Internet ping | PC pinging 203.0.113.1 |
| 19 | ASA NAT | PAT translations visible |
| 20 | Port security | Sticky MAC learned, secure-up |
| 21 | ASA interfaces | inside + outside up/up |
| 22 | WLC AP | AP registered via CAPWAP |
| 23 | Wireless client | Laptop connected to CorpWiFi |
| 24 | Final topology | All green links working |

---

## Troubleshooting Log

See docs/troubleshooting-log.md for every error hit
and how it was diagnosed and fixed.

Key errors documented:
- Native VLAN mismatch (99 vs 999)
- LACP suspended ports (both sides passive)
- DHCP failure (missing ip helper-address)
- OSPF no neighbors (passive-interface default)
- STP wrong root bridge (access layer winning)

---

## Tools

- Cisco Packet Tracer 9.0
- Cisco IOS-XE 16.3.2 (3650 switches)
- Cisco IOS 15.x (2960 switches)
- Cisco ASA OS 9.x

---

*Built as a learning exercise — not a professional deployment.
Curious about networking and documenting the journey.*
