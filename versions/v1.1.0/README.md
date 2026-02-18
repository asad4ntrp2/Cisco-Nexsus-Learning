# Cisco Nexus Product Family - Student Learning Guide

**Author:** Asad Yaseen
**Email:** asad4ntrp2@gmail.com

---

## Overview

A comprehensive study guide for the Cisco Nexus product family, designed for network engineers preparing for senior-level data center interviews. Covers everything from platform hardware through VXLAN/EVPN fabric architecture.

## Quick Start

Open **`index.html`** in any browser for the interactive learning experience with:
- Collapsible section-wise navigation
- Syntax-highlighted NX-OS configurations with copy button
- ASCII network diagrams
- Search and progress tracking
- Side-by-side comparison tables

## Repository Structure

```
Cisco Nexsus Learning/
├── README.md                                    ← You are here
├── VERSION.md                                   ← Document version history
├── index.html                                   ← Interactive HTML learning page
├── docs/
│   └── Cisco_Nexus_Interview_Guide.md           ← Main study guide (all 14 parts)
└── diagrams/
    ├── 01_Nexus_Family_Overview.drawio
    ├── 02_Leaf_Spine_Clos_Architecture.drawio
    ├── 03_VXLAN_Packet_Format.drawio
    ├── 04_vPC_Architecture.drawio
    ├── 05_EVPN_Route_Types.drawio
    ├── 06_Anycast_Gateway.drawio
    ├── 07_L2VNI_vs_L3VNI_Traffic_Flow.drawio
    └── 08_NDFC_Day0_Day1_Day2.drawio
```

## Guide Contents (14 Parts)

| Part | Topic | Category |
|------|-------|----------|
| 01 | Nexus Product Family Overview | Platform |
| 02 | Architecture Deep Dive (Cloud Scale ASIC) | Platform |
| 03 | Layer 2 Technologies (vPC, STP, Port Channels) | L2 |
| 04 | Layer 3 Technologies (OSPF, BGP, HSRP, VRF, PIM) | L3 |
| 05 | NX-OS vs IOS Differences | Comparison |
| 06 | Nexus vs Catalyst Family | Comparison |
| 07 | VXLAN Fundamentals (For Legacy Engineers) | VXLAN |
| 08 | BGP EVPN Deep Dive (All 5 Route Types) | VXLAN |
| 09 | VXLAN Fabric Operations (Anycast GW, IRB) | VXLAN |
| 10 | Complete Configuration Examples | Config |
| 11 | Nexus Dashboard & NDFC | Operations |
| 12 | VXLAN vs Legacy Networks | Migration |
| 13 | Clos Fabric Architecture | Design |
| 14 | Interview Tips & Common Questions | Interview |

## 5-Day Study Plan

| Day | Focus | Sections |
|-----|-------|----------|
| **Day 1** | Platform + Architecture | Parts 1, 2, 5, 6 |
| **Day 2** | L2 Technologies | Part 3 (vPC deep dive) |
| **Day 3** | L3 + VXLAN Basics | Parts 4, 7 |
| **Day 4** | EVPN Deep Dive | Parts 8, 9 |
| **Day 5** | Design + Operations | Parts 10, 11, 12, 13, 14 |

## Diagrams

All diagrams are in `.drawio` format. Open with:
- **[draw.io Desktop App](https://www.drawio.com/)** (recommended)
- **VS Code** with Draw.io Integration extension
- **Browser** at [app.diagrams.net](https://app.diagrams.net/) (File > Open From > Device)

| # | Diagram | Covers |
|---|---------|--------|
| 01 | Nexus Family Overview | All switch series, roles, and status |
| 02 | Leaf-Spine Clos Architecture | Full fabric topology |
| 03 | VXLAN Packet Format | Encapsulation with VNI, UDP, IP headers |
| 04 | vPC Architecture | Components, failure scenarios, consistency checks |
| 05 | EVPN Route Types | All 5 route types visualized |
| 06 | Anycast Gateway | Legacy HSRP vs distributed anycast |
| 07 | L2 VNI vs L3 VNI | Bridging vs routing traffic flow |
| 08 | NDFC Operations | Day-0, Day-1, Day-2 lifecycle |

## Key Interview Topics

1. **Cloud Scale ASIC** - Slice-based architecture, flex tiles, forwarding pipeline
2. **vPC** - All failure scenarios (especially split-brain), consistency checks
3. **VXLAN/EVPN** - All 5 route types, packet format, VNI mapping
4. **Anycast Gateway** - Why it replaces HSRP/VRRP, how it works
5. **Clos Fabric** - Leaf-spine design, ECMP, 2-hop latency
6. **NX-OS vs IOS** - Feature activation, interface-centric config, route-map requirements

## Sources

This guide was compiled from Cisco official documentation, data sheets, and configuration guides available at [cisco.com](https://www.cisco.com/).

---

*Author: Asad Yaseen (asad4ntrp2@gmail.com)*
*Version: 1.1.0*
*Created: February 14, 2026*
