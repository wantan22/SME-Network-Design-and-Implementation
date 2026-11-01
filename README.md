# SME Network Design and Implementation

> Redesigning a flat small-to-medium enterprise network into a segmented, redundant and hardened hierarchical design using VLANs, Router-on-a-Stick, LACP EtherChannel, PVST+, DHCP and Layer 2 security.

**Course:** NET2201 Computer Networks (SRWE) · **Practical Group:** 2 · **Submitted:** 2 August 2026

**Demo video:** [youtu.be/uDCfK2LEvJc](https://youtu.be/uDCfK2LEvJc)

---

## Overview

The legacy network put every department, workstation and management endpoint in a single broadcast domain. That meant excess broadcast traffic, no isolation between Staff, Student and Guest users, no automated addressing, no link redundancy and no access-layer protection.

This project replaces it with a structured design, built first in **Cisco Packet Tracer** and then replicated on **physical Cisco lab equipment**.

## Features

- **VLAN segmentation:** Staff, Student, Guest and Management VLANs, plus a dedicated native VLAN and a blackhole VLAN
- **Inter-VLAN routing:** 802.1Q Router-on-a-Stick on the core router (R1)
- **Centralised DHCP:** per-VLAN pools on R1, with gateway and infrastructure addresses excluded
- **Redundancy:** two-member LACP EtherChannel between the switches, with S1 as primary and S2 as secondary STP root bridge
- **Internet connectivity:** static default route on R1 and a summarised return route on a simulated ISP router
- **Access-layer security (8 controls):**
  1. Sticky port security (max 2 MACs, violation shutdown)
  2. Unused ports administratively shut down
  3. Unused ports moved to dead-end VLAN 999
  4. DTP disabled (`switchport nonegotiate` and static modes)
  5. Native VLAN moved from VLAN 1 to VLAN 100
  6. PortFast with BPDU Guard
  7. Isolated management VLAN 99
  8. SSH v2 only with local authentication (Telnet disabled)

## Topology

| Device | Packet Tracer model | Physical lab model | Role |
|---|---|---|---|
| R1 | ISR 4331 | Cisco 2911 | Core gateway: ROAS, DHCP server, default route |
| ISP | ISR 4331 | Cisco 2911 | Simulated ISP and return route |
| S1 | Catalyst 2960 | Catalyst 2960 | Access switch, primary STP root |
| S2 | Catalyst 2960 | Catalyst 2960 | Access switch, secondary STP root |
| SRV-WEB | Server-PT | n/a (ISP loopback used) | Simulated public web and DNS server |
| PC-A to PC-E, ADMIN-PC | Generic PC | 4 PCs available | DHCP test hosts and static management workstation |

Topology diagrams (Packet Tracer, logical and physical) are in the project report (`Report PDF Version.pdf`).

## Addressing Plan

| VLAN | Name | Subnet | Gateway | Notes |
|---|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | DHCP |
| 20 | STUDENT | 192.168.20.0/24 | 192.168.20.1 | DHCP |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 | DHCP |
| 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | Static only; S1 `.11`, S2 `.12`, ADMIN-PC `.21` |
| 100 | NATIVE | none | none | Native VLAN on all trunks |
| 999 | BLACKHOLE | none | none | Unused ports (shut down) |

**WAN:** R1 `209.165.200.225/30` ⇄ ISP `209.165.200.226/30`. The ISP LAN is `209.165.201.0/27`, and SRV-WEB is at `209.165.201.10`.

**Routing:**

```
R1(config)#  ip route 0.0.0.0 0.0.0.0 209.165.200.226
ISP(config)# ip route 192.168.0.0 255.255.0.0 209.165.200.225
```

## Repository Contents

```
SME Network Design and Implementation/
├── Network Simulation.pkt
├── Presentation Slide.pptx
├── Report PDF Version.pdf
└── Report Word Version.docx
```

| File | Description |
|---|---|
| `Network Simulation.pkt` | Cisco Packet Tracer simulation file |
| `Report PDF Version.pdf` | Full project report (PDF): requirements, design, configs, testing, troubleshooting |
| `Report Word Version.docx` | Editable Word version of the report |
| `Presentation Slide.pptx` | Presentation slides |

## Getting Started

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (a free Networking Academy account is required).
2. Open `Network Simulation.pkt`.
3. Log in to a device with these lab credentials (**lab use only**):
   - Enable secret: `class`
   - Local user: `admin` / `admin`
   - Console/VTY line password: `cisco`
4. From **ADMIN-PC**, verify secure management:
   ```
   ssh -l admin 192.168.99.11
   ```

## Testing & Verification

Fifteen tests were run in Packet Tracer and repeated on physical hardware:

| Area | Checks |
|---|---|
| Addressing | DHCP assignment per VLAN, excluded ranges respected |
| Connectivity | Intra-VLAN across switches, inter-VLAN routing, internet via default route and ISP return route |
| Layer 2 | VLAN database, trunks and native VLAN, EtherChannel health (`SU`, LACP), STP root placement |
| Resilience | Link failover during continuous ping with no STP reconvergence |
| Security | Port security violation, BPDU Guard, SSH v2 (Telnet refused), unused port quarantine |

Evidence and screenshots are in the report appendix (`Report PDF Version.pdf`).

## Lab Notes & Limitations

- The physical lab's older Catalyst 2960 IOS image has no cryptographic feature set, so **SSH could not be enabled on the lab switches**. The management plane there is protected by console/VTY passwords, an enable secret, local accounts, `service password-encryption` and the isolated VLAN 99. Full SSH v2 is demonstrated in Packet Tracer.
- Physical routers use `G0/0` and subinterfaces `G0/0.10/.20/.30/.99` instead of the ISR 4331 naming.
- Only four PCs were available in the lab, so one host per VLAN was used.
- **Future work:** an ACL on the Guest subinterface to restrict VLAN 30 to internet-only access, and Rapid PVST+ if more redundant switch links are added.

## Key Lessons

- Sticky port security learns MACs only when a port forwards traffic, so `show port-security` must be read as what the device has learned, not just what it was configured to do.
- The Management VLAN has no DHCP pool by design. Predictable static addressing and no easy lease for intruders are worth the extra admin effort.

## Team (Practical Group 2)

| Name | Contribution |
|---|---|
| Chan Xiang Wei | 20% |
| Teo Wan Yin | 20% |
| Tey Yu Shan | 20% |
| Chia Li Ying | 20% |
| Arjoona Kumaran | 20% |

## Disclaimer

Built for academic purposes. The credentials above are lab placeholders and must never be reused in a production environment.
