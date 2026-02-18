# Version History - Cisco Nexus Student Learning Guide

**Author:** Asad Yaseen (asad4ntrp2@gmail.com)

## Document Versioning

This document follows semantic versioning: **MAJOR.MINOR.PATCH**
- **MAJOR**: Significant restructuring or new sections added
- **MINOR**: New content within existing sections
- **PATCH**: Corrections, typo fixes, formatting improvements

---

### v2.1.0 (2026-02-18) - Private VLANs (PVLANs)

**New Content Added:**

**Section 3.8: Private VLANs (~300 lines)**
- What are Private VLANs — full theory with architecture diagram
- Three port types explained: Promiscuous, Community, Isolated
- How PVLANs maintain security (hardware-enforced L2 isolation, ARP spoofing defense, lateral movement prevention)
- Complete communication matrix with ASCII diagram
- How PVLANs communicate with other VLANs (inter-VLAN routing, ip local-proxy-arp for hairpin)
- Full NX-OS configuration example (9 steps)
- Verification commands
- Real-world use cases: multi-tenant colocation, DMZ server isolation, hotel WiFi, PCI-DSS compliance
- PVLANs in VXLAN environment: VNI mapping, NVE config, EVPN config, anycast gateway integration
- Benefits and limitations of PVLAN + VXLAN

**New Diagram:**
- `09_Private_VLAN_Architecture.drawio` — PVLAN architecture with port types, communication rules, and use cases

**Other Updates:**
- Glossary expanded with 5 new PVLAN terms
- continue.md updated with PVLAN topics and practice questions
- README.md updated with PVLAN section and diagram entry

---

### v2.0.0 (2026-02-18) - DAD, EVPN Guide, Stacking vs VPC

**New Content Added:**

**Section 3.7: Dual-Active Detection (DAD) (~200 lines)**
- Full theory of DAD: detection, recovery, restoration lifecycle
- All DAD detection methods (DAD Link, Peer-Keepalive, ePAgP, BFD, Fast Hello)
- DAD in vPC (Nexus) with peer-link/keepalive decision matrix and ASCII diagrams
- DAD in StackWise Virtual (Catalyst 9000) with SVL vs DAD link detection flow
- DAD in VSS (Catalyst 6500) for legacy reference
- Cross-technology comparison table (vPC vs SVL vs VSS)
- Best practices and verification commands

**Part 15: EVPN Comprehensive Guide (~350 lines)**
- What is EVPN — full theory with architecture diagram
- 4 Types of EVPN: EVPN-VXLAN, EVPN-MPLS, EVPN-SR, EVPN-PBB with comparison table
- Where EVPN is used: DC fabrics, DCI, service provider, campus, multi-cloud
- 6 Real-world use cases: multi-tenant cloud, financial trading, 5G mobile backhaul, enterprise DC migration, disaster recovery, Kubernetes/container networking
- EVPN service types (E-LAN, E-LINE, E-TREE) with diagrams

**Part 16: Stacking vs VPC — Complete Comparison (~350 lines)**
- What is stacking — all StackWise generations table
- What is vPC — architecture recap
- 15-aspect detailed comparison table
- Use cases: where stacking is used (4 scenarios with diagrams)
- Use cases: where vPC is used (4 scenarios with diagrams)
- Pros and cons for both technologies
- Limitations (9 for stacking, 9 for vPC)
- Device compatibility tables (which platforms support stacking vs vPC)
- Decision matrix flowchart
- Real-world coexistence architecture diagram

**Other Updates:**
- Table of Contents updated with Parts 15-16
- Glossary expanded with 17 new terms (DAD, SVL, MEC, ePAgP, VSS, VPLS, VPWS, E-LAN, E-LINE, E-TREE, SR, SRv6, PBB, DCI, MEF)
- Study plan expanded from 5 days to 6 days
- continue.md updated with all new topics and practice questions
- README.md updated with new parts, study plan, and interview topics

**Stats:** Guide grew from 2,036 lines (v1.3.1) to 3,054 lines (v2.0.0)

---

### v1.3.1 (2026-02-14) - Professional Branding Badges

**Changes:**
- Redesigned all 16 diagram branding badges with professional styling
- Purple-blue gradient panel background (`#667eea` → `#764ba2`)
- Bold 17px white author name with gold accent separator bar
- 13px email and GitHub link text (gold highlight for GitHub URL)
- Vibrant dual-tone gradient fills on all diagram elements
  - Green-to-teal (current), Blue-to-indigo (fabric), Orange-to-gold (legacy)
  - Pink-to-purple (specialized), Red-to-orange (failures)
- Drop shadows, rounded corners, and diagonal gradient backgrounds
- Expanded SVG viewBox dimensions for proper badge display

---

### v1.3.0 (2026-02-14) - Diagrams Gallery + Branding

**Changes:**
- Added dedicated **Architecture Diagrams Gallery** section with all 8 diagrams full-page
- Full-screen lightbox viewer (click any diagram to enlarge, press Escape to close)
- SVG download button for each diagram
- Author branding watermark on all diagrams (name, email, GitHub link)
- GitHub repository link added to landing page with GitHub icon
- Author metadata added to all Draw.io source files
- Inline diagrams in study sections are clickable to open lightbox
- Sidebar navigation updated with Diagrams Gallery link

---

### v1.2.0 (2026-02-14) - Embedded Interactive Diagrams

**Changes:**
- Embedded all 8 Draw.io architecture diagrams directly into `index.html`
  - Part 1: Nexus Family Overview diagram
  - Part 3: vPC Architecture diagram
  - Part 7: VXLAN Packet Format diagram
  - Part 8: EVPN Route Types diagram
  - Part 9: Anycast Gateway + L2 VNI vs L3 VNI Traffic Flow diagrams
  - Part 11: NDFC Day-0/Day-1/Day-2 Operations diagram
  - Part 13: Leaf-Spine Clos Architecture diagram
- Diagrams render as interactive draw.io viewers (zoom, pan, layers)
- Each diagram placed in its corresponding study section

---

### v1.1.0 (2026-02-14) - Interactive HTML + GitHub Release

**Changes:**
- Added interactive HTML learning page (`index.html`) with:
  - Collapsible section-wise navigation
  - Fixed sidebar with topic grouping
  - Syntax-highlighted NX-OS configuration blocks with copy button
  - Search functionality (Ctrl+K)
  - Scroll progress bar
  - Side-by-side comparison cards
  - Self-check progress tracker with localStorage persistence
  - Responsive mobile design
  - Dark theme optimized for readability
- Updated README.md with author information and quick start guide
- Updated author attribution across all files
- Published to GitHub as public repository

---

### v1.0.0 (2026-02-14) - Initial Release

**Contents:**
- 14-part comprehensive interview preparation guide
- 8 Draw.io architecture diagrams
- Complete NX-OS configuration examples (leaf, spine, border leaf)
- vPC deep dive with all failure scenarios
- VXLAN/EVPN coverage with all 5 route types
- BGP EVPN control plane explanation
- Clos fabric architecture overview
- Nexus Dashboard / NDFC operations guide
- Legacy VLAN to VXLAN migration concepts
- Interview tips and common questions
- 5-day study plan

**Diagrams Included:**
1. `01_Nexus_Family_Overview.drawio` - Product family hierarchy
2. `02_Leaf_Spine_Clos_Architecture.drawio` - Full fabric topology
3. `03_VXLAN_Packet_Format.drawio` - Encapsulation details
4. `04_vPC_Architecture.drawio` - Components and failure scenarios
5. `05_EVPN_Route_Types.drawio` - All 5 route types visualized
6. `06_Anycast_Gateway.drawio` - HSRP vs anycast comparison
7. `07_L2VNI_vs_L3VNI_Traffic_Flow.drawio` - Bridging vs routing flows
8. `08_NDFC_Day0_Day1_Day2.drawio` - NDFC lifecycle operations

---

### Planned for v3.0.0
- ACI mode deep dive
- VXLAN Multi-Site configuration
- Additional lab exercises with topology files
- Troubleshooting flowcharts
- Ansible/Terraform automation examples
- Streaming telemetry configuration
- EVPN-MPLS lab configuration
- StackWise Virtual lab exercises
