# Study Progress Tracker - Cisco Nexus Interview Preparation

## Current Status: IN PROGRESS

---

## Completed Topics

- [x] Part 1: Nexus Product Family Overview (all models)
- [x] Part 2: Architecture Deep Dive (Cloud Scale ASIC, 9500 chassis, 9400)
- [x] Part 3: Layer 2 Technologies (vPC, STP, FabricPath, port channels, **DAD**, **Private VLANs**)
- [x] Part 4: Layer 3 Technologies (OSPF, BGP, HSRP, VRF, PIM, PBR)
- [x] Part 5: NX-OS vs IOS Differences
- [x] Part 6: Nexus vs Catalyst Comparison
- [x] Part 7: VXLAN Fundamentals (packet format, VTEP, VNI mapping)
- [x] Part 8: BGP EVPN Deep Dive (all 5 route types, RD/RT, ARP suppression)
- [x] Part 9: VXLAN Fabric Operations (anycast GW, L2/L3 VNI, ECMP, IRB)
- [x] Part 10: Configuration Examples (leaf, spine, border leaf)
- [x] Part 11: Nexus Dashboard & NDFC (Day-0/1/2 operations)
- [x] Part 12: VXLAN vs Legacy Networks
- [x] Part 13: Clos Fabric Architecture
- [x] Part 14: Interview Tips & Common Questions
- [x] Part 15: EVPN Comprehensive Guide (types, use cases, real-world examples)
- [x] Part 16: Stacking vs VPC — Complete Comparison
- [x] Draw.io Diagrams (9 diagrams created)

## New in v2.1.0

### Section 3.8: Private VLANs (PVLANs)
- [x] What are Private VLANs — full theory with ASCII architecture diagram
- [x] Three port types: Promiscuous, Community, Isolated
- [x] How PVLANs maintain security (hardware-enforced, L2 attack defense)
- [x] Communication matrix (who can talk to whom)
- [x] How PVLANs communicate with other VLANs (inter-VLAN routing, ip local-proxy-arp)
- [x] Full NX-OS configuration example
- [x] Verification commands
- [x] Real-world use cases (colocation, DMZ, hotel WiFi, PCI-DSS)
- [x] PVLANs in VXLAN environment (VNI mapping, config, benefits, limitations)
- [x] Draw.io diagram: 09_Private_VLAN_Architecture.drawio

## New in v2.0.0

### Section 3.7: Dual-Active Detection (DAD)
- [x] DAD theory and detection flow
- [x] All DAD methods (DAD Link, Peer-Keepalive, ePAgP, BFD, Fast Hello)
- [x] DAD in vPC (Nexus) — decision matrix, step-by-step behavior, config
- [x] DAD in StackWise Virtual (Catalyst 9000) — SVL vs DAD link, detection flow
- [x] DAD in VSS (Catalyst 6500) — legacy reference
- [x] Comparison table across all three technologies
- [x] Best practices and verification commands
- [x] ASCII diagrams for all DAD scenarios

### Part 15: EVPN Comprehensive Guide
- [x] What is EVPN — full theory with architecture diagram
- [x] 4 Types of EVPN (VXLAN, MPLS, SR, PBB) with comparison table
- [x] Where EVPN is used (DC fabrics, DCI, SP, campus, multi-cloud)
- [x] 6 Real-world use cases (multi-tenant cloud, financial trading, 5G backhaul, enterprise migration, DR, Kubernetes)
- [x] EVPN service types (E-LAN, E-LINE, E-TREE) with diagrams

### Part 16: Stacking vs VPC — Complete Comparison
- [x] What is stacking (all StackWise generations table)
- [x] What is vPC (recap with diagram)
- [x] Detailed 15-aspect comparison table
- [x] Use cases: where stacking is used (4 scenarios with diagrams)
- [x] Use cases: where vPC is used (4 scenarios with diagrams)
- [x] Pros and cons for both technologies
- [x] Limitations (9 for stacking, 9 for vPC)
- [x] Device tables: which platforms support stacking vs vPC
- [x] Decision matrix flowchart
- [x] Real-world architecture showing both coexisting

## Topics for Further Study

### Priority 1 (Before Interview)
- [ ] Practice configuring VXLAN EVPN on NX-OS simulator (CML/VIRL or NX-OSv)
- [ ] Lab the vPC failure scenarios hands-on
- [ ] Lab DAD failure scenarios (peer-link down + keepalive up)
- [ ] Lab Private VLANs — configure promiscuous, community, and isolated ports
- [ ] Test PVLAN isolation: verify isolated hosts cannot reach each other
- [ ] Review `show` commands and troubleshooting workflows
- [ ] Practice explaining EVPN route types verbally (whiteboard practice)
- [ ] Practice explaining Stacking vs vPC differences verbally

### Priority 2 (Advanced Topics)
- [ ] ACI (Application Centric Infrastructure) mode vs NX-OS mode
- [ ] VXLAN Multi-Site with BGW (Border Gateway)
- [ ] VXLAN Multi-Site with EVPN Multi-Site extension
- [ ] Tenant Routed Multicast (TRM) in VXLAN
- [ ] CloudSec encryption for VXLAN
- [ ] Segment Routing (SRv6) with EVPN
- [ ] EVPN-MPLS configuration on service provider platforms
- [ ] Network as Code with Ansible/Terraform for NDFC

### Priority 3 (Operational Depth)
- [ ] ISSU (In-Service Software Upgrade) on Nexus
- [ ] NX-API and programmability
- [ ] Streaming telemetry (gNMI, gRPC)
- [ ] Nexus Dashboard Insights (NDI) for assurance
- [ ] Troubleshooting EVPN route-type issues in production
- [ ] StackWise Virtual troubleshooting and ISSU

## Lab Exercises (Recommended)

1. **Lab 1:** Build a 2-spine, 4-leaf VXLAN EVPN fabric
   - Configure underlay (OSPF, loopbacks)
   - Configure overlay (BGP EVPN, NVE, VNIs)
   - Verify with `show nve peers`, `show bgp l2vpn evpn`

2. **Lab 2:** Add anycast gateway and test inter-subnet routing
   - Configure distributed anycast gateway
   - Test symmetric IRB between VLANs
   - Verify L3 VNI traffic flow

3. **Lab 3:** Configure vPC pair at leaf layer
   - Set up vPC domain, peer-link, keepalive
   - Simulate failure scenarios
   - Verify consistency checks
   - **Test DAD:** Pull peer-link while keepalive is up, verify secondary suspends vPC ports

4. **Lab 4:** Add border leaf for external connectivity
   - Configure VRF-Lite with eBGP
   - Test north-south traffic flow
   - Verify Type 5 route propagation

5. **Lab 5:** StackWise Virtual on Catalyst 9500
   - Configure SVL and DAD link
   - Test dual-active detection
   - Compare behavior with vPC

## Interview Practice Questions

Work through these without looking at the guide:
1. Walk me through the VXLAN packet format end-to-end
2. Explain all 5 EVPN route types and when each is used
3. What happens in a vPC split-brain scenario?
4. How does anycast gateway differ from HSRP?
5. Design a VXLAN fabric for 1000 servers across 2 data centers
6. What are the key differences between NX-OS and IOS?
7. **NEW:** Explain Dual-Active Detection — what is it, why is it needed, and how does it differ between vPC and StackWise Virtual?
8. **NEW:** What are the different types of EVPN and where is each used?
9. **NEW:** What is the difference between stacking and vPC? When would you use each?
10. **NEW:** Name 3 real-world EVPN use cases and explain why EVPN is the right choice for each.
11. **NEW:** On which devices is vPC used vs stacking? What are the limitations of each?
12. **NEW:** What are Private VLANs? Explain the three port types and draw the communication matrix.
13. **NEW:** How would you use PVLANs in a VXLAN EVPN fabric? What are the benefits?

---

*Last Updated: February 18, 2026 (v2.1.0)*
