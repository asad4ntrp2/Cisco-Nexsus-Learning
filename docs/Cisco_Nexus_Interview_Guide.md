# Cisco Nexus Product Family - Comprehensive Interview Preparation Guide

> **Purpose:** Deep-dive study guide for a senior-level Cisco Nexus/Data Center interview
> **Author:** Asad Yaseen (asad4ntrp2@gmail.com)
> **Date:** February 2026

---

# TABLE OF CONTENTS

1. [Part 1: Cisco Nexus Product Family Overview](#part-1-cisco-nexus-product-family-overview)
2. [Part 2: Nexus Architecture Deep Dive](#part-2-nexus-architecture-deep-dive)
3. [Part 3: Layer 2 Technologies](#part-3-layer-2-technologies)
4. [Part 4: Layer 3 Technologies](#part-4-layer-3-technologies)
5. [Part 5: NX-OS vs IOS Differences](#part-5-nx-os-vs-ios-differences)
6. [Part 6: Nexus vs Catalyst Family Comparison](#part-6-nexus-vs-catalyst-family-comparison)
7. [Part 7: VXLAN Fundamentals (For Legacy Engineers)](#part-7-vxlan-fundamentals)
8. [Part 8: BGP EVPN Deep Dive](#part-8-bgp-evpn-deep-dive)
9. [Part 9: VXLAN Fabric Operations](#part-9-vxlan-fabric-operations)
10. [Part 10: VXLAN Configuration Examples](#part-10-vxlan-configuration-examples)
11. [Part 11: Cisco Nexus Dashboard and NDFC](#part-11-cisco-nexus-dashboard-and-ndfc)
12. [Part 12: VXLAN vs Legacy Networks](#part-12-vxlan-vs-legacy-networks)
13. [Part 13: Clos Fabric Architecture](#part-13-clos-fabric-architecture)
14. [Part 14: Interview Tips and Common Questions](#part-14-interview-tips)

---

# PART 1: CISCO NEXUS PRODUCT FAMILY OVERVIEW

## 1.1 The Nexus Family at a Glance

The Cisco Nexus family is purpose-built for **data center** environments. Unlike Catalyst switches (campus/enterprise), Nexus switches run **NX-OS** and are optimized for:
- High-density 10/25/40/50/100/200/400 GbE
- Low-latency forwarding
- VXLAN/EVPN overlay fabrics
- ACI (Application Centric Infrastructure) mode
- Programmability (NX-API, OpenConfig, gRPC)

## 1.2 Current Nexus Switch Series

### Nexus 9000 Series (Flagship - Data Center Spine/Leaf)

The Nexus 9000 is the **primary** Nexus platform today. It can operate in two modes:
- **NX-OS Standalone Mode** — Traditional CLI-based operation with VXLAN/EVPN
- **ACI Mode** — Policy-driven SDN fabric controlled by APIC

**Sub-families:**

| Sub-Series | Form Factor | Role | Key Specs |
|-----------|------------|------|-----------|
| **9200** | Fixed 1RU | Leaf / Aggregation | Up to 7.2 Tbps, 5.35 bpps, 1/10/25/40/50/100G ports, 40 MB shared buffer |
| **9300** | Fixed 1RU/2RU | Leaf (most popular) | Up to 3.6 Tbps, 1.2 bpps (FX3), 48x 10/25G + 6x 100G typical, Cloud Scale ASIC |
| **9400** | Modular 4RU | Aggregation / Spine | Up to 25.6 Tbps, 8 expansion slots, 64x 400G, MACsec on all ports, PTP Class C |
| **9500** | Modular 4/8/16 slot | Spine / Core | Up to 16 line cards, 6 fabric modules, 1/10/25/40/50/100/200/400G line cards |
| **9800** | Fixed | Ultra-high density 400G | Latest generation for AI/ML workloads, 400/800G readiness |

**Nexus 9300 ASIC Generations (Critical Interview Knowledge):**

| ASIC Suffix | Generation | Key Feature |
|------------|-----------|-------------|
| **-EX** | 1st gen Cloud Scale | 2600 Mpps, VXLAN/EVPN support |
| **-FX** | 2nd gen Cloud Scale | 1200 Mpps, added MACsec, improved telemetry |
| **-FX2** | 3rd gen Cloud Scale | Enhanced buffer, improved ACI features |
| **-FX3** | 4th gen Cloud Scale | 1200 Mpps, cost-effective cloud-scale, increased endpoints |
| **-GX** | 5th gen Cloud Scale | 400G support, deeper buffers |
| **-GX2** | 6th gen Cloud Scale | Latest, highest density, 400G native |

> **Interview Tip:** The differences between ASIC generations come down to the number of **slices**, **forwarding TCAMs**, and **flex tiles** each ASIC supports, which are resources combined in what Cisco calls **routing modes** or **routing templates**.

### Nexus 7000/7700 Series (Legacy Modular)

| Feature | Detail |
|---------|--------|
| Role | Core / Aggregation (legacy DCI, large campus DC) |
| Architecture | Supervisor + I/O modules (M-series, F-series) |
| Key Feature | **VDC (Virtual Device Contexts)** — partition one physical switch into multiple logical switches |
| Technologies | OTV, FabricPath, vPC, LISP, MPLS |
| Status | **End-of-sale announced** — migrate to Nexus 9000 |

> **When to use:** Only in existing deployments or where VDC is absolutely required.

### Nexus 5000/5500/5600 Series (Legacy Access/Aggregation)

| Feature | Detail |
|---------|--------|
| Role | Access / Aggregation layer |
| Port Speeds | 1/10/40 GbE, FCoE (Fibre Channel over Ethernet) |
| Key Feature | Unified fabric (LAN + SAN on same wire), FabricPath |
| Status | **End-of-sale** — replaced by Nexus 9300 |

> **When to use:** Legacy environments with FCoE requirements.

### Nexus 3000/3500 Series (Low-Latency / ToR)

| Feature | Detail |
|---------|--------|
| Role | Top-of-Rack, HPC, financial trading, leaf |
| Key Feature | Ultra-low latency (~1 microsecond), L3 at wire speed |
| Port Speeds | 1/10/25/40/50/100 GbE |
| OS | NX-OS (same as 9000 in many cases) |

> **When to use:** When sub-microsecond latency is critical (HFT, HPC clusters).

### Nexus 2000 Series (Fabric Extenders - FEX)

| Feature | Detail |
|---------|--------|
| Role | Remote line card (extends parent Nexus 5k/7k/9k) |
| Architecture | **NOT an independent switch** — managed as part of parent |
| Key Feature | Simplifies management — appears as line card of parent |
| Status | Still supported but declining in new designs |

> **When to use:** When you want to extend port density without adding management complexity. Being replaced by small leaf switches in modern designs.

---

# PART 2: NEXUS ARCHITECTURE DEEP DIVE

## 2.1 Cloud Scale ASIC Architecture (Nexus 9000)

This is the heart of modern Nexus switching. You MUST understand this for an architectural interview.

### Slice-Based Architecture

The Cloud Scale ASIC uses a **slice-based architecture**:
- Each **slice** is a self-contained forwarding complex (an "SoC" — Switch on Chip)
- Multiple slices run in parallel to achieve higher throughput
- Each slice handles both **ingress** and **egress** functions for a subset of ports
- Slices are interconnected via a **slice interconnect** providing non-blocking any-to-any connectivity

```
                    ┌─────────────────────────────────┐
                    │        Cloud Scale ASIC          │
                    │                                  │
                    │  ┌────────┐  ┌────────┐         │
   Ports 1-12 ─────│──│ Slice 0│──│ Slice 1│──────────│───── Ports 13-24
                    │  └───┬────┘  └────┬───┘         │
                    │      │   Slice    │              │
                    │      │ Interconnect│              │
                    │  ┌───┴────┐  ┌────┴───┐         │
   Ports 25-36 ────│──│ Slice 2│──│ Slice 3│──────────│───── Ports 37-48
                    │  └────────┘  └────────┘         │
                    └─────────────────────────────────┘
```

### Forwarding Pipeline (Per Slice)

Each slice has a forwarding pipeline with these stages:

1. **Input Forwarding Controller (IFC)**
   - Receives packet from MAC layer
   - Parses packet headers
   - Performs lookup series (L2, L3, ACL, QoS)
   - Decides accept/deny and forwarding path

2. **Input Data-Path Controller (IDC)**
   - Handles buffering and scheduling
   - Manages ingress queuing

3. **Egress Data-Path Controller (EDC)**
   - Handles output buffering
   - Egress queuing and scheduling

4. **Egress Forwarding Controller (EFC)**
   - Final header modifications (rewrite)
   - Encapsulation (VXLAN, etc.)
   - Sends packet to MAC layer for transmission

### Programmable Forwarding Tables (Flex Tiles)

This is what makes Cloud Scale powerful:
- **Flex Tiles** provide a fungible (flexible) pool of table entries
- Tables can be **resized** based on deployment use case
- Supported table types:
  - IPv4/IPv6 unicast LPM (Longest Prefix Match)
  - IPv4/IPv6 unicast Host Route Table (HRT)
  - IPv4/IPv6 multicast (*, G) and (S, G)
  - MAC address / adjacency tables
  - ECMP tables
  - ACI policy tables
  - ACL/QoS TCAM entries

> **Interview Key Point:** Unlike older fixed-table ASICs, Cloud Scale can dynamically reallocate forwarding resources. A spine switch can maximize LPM entries while a leaf switch maximizes MAC/host entries.

## 2.2 Nexus 9500 Internal Architecture

The 9500 is a **modular chassis** with an internal **Clos fabric**:

```
   ┌──────────────────────────────────────────────────┐
   │              Nexus 9508 Chassis                    │
   │                                                    │
   │  ┌──────┐ ┌──────┐ ┌──────┐ ... ┌──────┐        │
   │  │ LC 1 │ │ LC 2 │ │ LC 3 │     │ LC 8 │  Line  │
   │  └──┬───┘ └──┬───┘ └──┬───┘     └──┬───┘  Cards  │
   │     │        │        │             │              │
   │  ┌──┴────────┴────────┴─────────────┴──┐          │
   │  │         Fabric Module Mesh           │          │
   │  │   FM1   FM2   FM3   FM4   FM5   FM6  │          │
   │  └─────────────────────────────────────┘          │
   │                                                    │
   │  ┌──────────┐  ┌──────────┐                       │
   │  │  SUP A   │  │  SUP B   │  (Active/Standby)    │
   │  └──────────┘  └──────────┘                       │
   └──────────────────────────────────────────────────┘
```

**Key components:**
- **Chassis options:** 9504 (4-slot), 9508 (8-slot), 9516 (16-slot)
- **Supervisor modules:** Active/Standby pair, state-synchronized
- **Line cards:** Cloud Scale (EX/FX/GX generations) providing 1-400G ports
- **Fabric modules:** Up to 6, providing internal Clos interconnect
  - FM-G: Up to 1.6 Tbps per line card slot
  - FM-E2: Up to 800 Gbps per line card slot
  - FM-E: Up to 800 Gbps per line card slot
- All traffic between line cards is **load-balanced across all fabric modules**

## 2.3 Nexus 9400 Architecture

The 9400 is a **new centralized modular** design:
- **4RU chassis** with 8 expansion slots
- Supports up to 64x 400G, or 128x 200G, or 176x 10/25/50G
- Up to **25.6 Tbps** switching capacity
- Single supervisor with expansion modules
- **MACsec** on all ports
- **PTP Class C** timing accuracy (important for telecom/5G)
- Supports both NX-OS and ACI modes

---

# PART 3: LAYER 2 TECHNOLOGIES

## 3.1 VLANs on NX-OS

VLANs work the same conceptually as IOS but with NX-OS syntax:

```
! Create VLANs
vlan 10
  name SERVERS
vlan 20
  name STORAGE

! Trunk interface
interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20

! Access interface
interface Ethernet1/2
  switchport mode access
  switchport access vlan 10
```

**NX-OS VLAN Differences from IOS:**
- VLAN 1 cannot be deleted but can be disabled
- VTP is supported but **not recommended** in data centers (use manual VLAN provisioning)
- VLAN ranges can be configured system-wide with `system vlan reserve`
- VN-Segment mapping available for VXLAN: `vlan 10` → `vn-segment 10010`

## 3.2 vPC (Virtual Port Channel) — CRITICAL INTERVIEW TOPIC

### What is vPC?

vPC allows two physical Nexus switches to appear as **one logical switch** to a downstream device. The downstream device forms a regular port channel (LACP) that spans two physical upstream switches.

### Why vPC Exists

In traditional STP networks:
- Redundant links are **blocked** (wasted bandwidth)
- Convergence is slow (seconds)

With vPC:
- **All links are active** (no STP blocking)
- Instant convergence
- Double the available bandwidth

### vPC Architecture Diagram

```
         ┌─────────────────────────────────┐
         │        vPC Domain               │
         │                                  │
    ┌────┴─────┐    Peer-Link    ┌─────────┴───┐
    │  Nexus A  │◄══════════════►│   Nexus B    │
    │ (Primary) │    (10G/40G+)  │ (Secondary)  │
    └─┬───┬──┬─┘                 └─┬──┬───┬────┘
      │   │  │    Peer-Keepalive   │  │   │
      │   │  │◄ ─ ─ ─ ─ ─ ─ ─ ─ ►│  │   │
      │   │  │   (mgmt0 or L3)    │  │   │
      │   │  │                     │  │   │
      │   │  └──────┐   ┌─────────┘  │   │
      │   │         │   │            │   │
      │   │    ┌────┴───┴────┐      │   │
      │   │    │  Server /   │      │   │
      │   │    │  Switch     │      │   │
      │   │    │  (LACP)     │      │   │
      │   │    └─────────────┘      │   │
      │   │                         │   │
      │   └──── vPC member ─────────┘   │
      │                                  │
      └──── Orphan ports (non-vPC) ─────┘
```

### vPC Components

| Component | Purpose |
|-----------|---------|
| **vPC Domain** | Logical grouping (must match on both peers) |
| **Peer-Link** | Port channel carrying control + data between peers. Carries BPDUs, HSRP, ARP, and data frames during failover |
| **Peer-Keepalive** | Heartbeat link (L3) — detects if peer is alive. Uses mgmt0 or dedicated L3 link |
| **vPC Member Port** | The actual port channel facing downstream |
| **Orphan Port** | Non-vPC interface on a vPC switch (single-attached device) |

### vPC Role Election

1. Lower **role priority** becomes Primary (default 32667)
2. If priority ties, lower **system MAC** wins
3. Role is **sticky** — does not change unless peer-link fails

### Consistency Checks (CRITICAL)

vPC performs consistency checks to ensure both peers have matching configurations:

**Type 1 (Suspend) — Will shut down vPC member ports if mismatched:**
- STP mode (PVST/MST/Rapid-PVST)
- STP global settings
- VLAN allowed on trunk
- STP interface parameters
- Port mode (access/trunk)
- MTU
- Storm control settings

**Type 2 (Graceful) — Logs warning but does NOT suspend ports:**
- STP priority
- STP port type
- BPDU filter/guard
- LACP mode

### vPC Failure Scenarios

**Scenario 1: Single vPC Member Link Failure**
- Traffic shifts to surviving member link
- LACP handles convergence
- Impact: Minimal — standard port channel behavior

**Scenario 2: Peer-Link Failure (Keepalive UP)**
- Secondary switch **shuts down all vPC member ports**
- Primary keeps all interfaces up
- Orphan ports on secondary become unreachable
- Impact: Significant — half bandwidth, orphan port outage

**Scenario 3: Keepalive Link Failure Only**
- No immediate impact — roles already elected
- Just heartbeat is lost
- **Danger:** If peer-link then fails → split-brain

**Scenario 4: SPLIT-BRAIN (Keepalive fails THEN Peer-link fails)**
- Both switches think they are Primary
- Both keep vPC ports UP
- Results in: **duplicate traffic, MAC flapping, L2 loops, black-holing**
- **This is the most dangerous scenario**
- Prevention: Use completely independent paths for peer-link and keepalive

### vPC Configuration Example

```
! === Step 1: Enable features ===
feature vpc
feature lacp

! === Step 2: Create vPC Domain ===
vpc domain 100
  role priority 2000            ! Lower = Primary
  peer-keepalive destination 10.201.182.26 source 10.201.182.25
  peer-gateway                  ! Routes traffic destined to peer's MAC
  peer-switch                   ! Both switches share STP root role
  delay restore 150             ! Wait 150s after reload before bringing up vPC
  ip arp synchronize            ! Sync ARP between peers
  ipv6 nd synchronize           ! Sync IPv6 ND between peers
  auto-recovery                 ! Auto-recover if both peers reboot

! === Step 3: Create Peer-Link ===
interface port-channel 1
  switchport mode trunk
  switchport trunk allowed vlan 1-1000
  vpc peer-link                 ! Designate as peer-link

interface Ethernet1/49-50
  channel-group 1 mode active   ! LACP active to peer
  no shutdown

! === Step 4: Create vPC Member ===
interface port-channel 10
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10                        ! vPC number (must match across domain)

interface Ethernet1/1
  channel-group 10 mode active
  no shutdown
```

### vPC Best Practices

1. **Peer-link:** Use at least 2x 10G or 40G links in a port channel
2. **Keepalive:** Use mgmt0 (out-of-band) or a dedicated VRF — NEVER use peer-link
3. **Enable peer-gateway** — allows routing of packets destined to peer's HSRP MAC
4. **Enable peer-switch** — both switches act as STP root (eliminates root bridge issues)
5. **delay restore** — prevents black-holing after reload (wait for routing to converge)
6. **auto-recovery** — recovers vPC if both switches reboot simultaneously

### When to Use vPC

| Use It When | Don't Use It When |
|------------|-------------------|
| Active-active L2 redundancy needed | Pure L3 routed design (no L2 extension) |
| Servers with LACP dual-homed | Single-attached devices (orphan ports add complexity) |
| STP-free topology desired | VXLAN EVPN with ESI multihoming available |
| FCoE environments | Small deployments where STP is acceptable |

## 3.3 Spanning Tree Protocol on NX-OS

NX-OS supports:
- **Rapid-PVST+** (default) — Per-VLAN Rapid STP
- **MST** (Multiple Spanning Tree) — IEEE 802.1s

NX-OS does **NOT** support classic STP (802.1D) or PVST.

```
! Enable Rapid-PVST+ (default)
spanning-tree mode rapid-pvst

! Set root bridge
spanning-tree vlan 1-100 root primary

! Configure bridge priority
spanning-tree vlan 10 priority 4096

! Edge ports (PortFast equivalent)
interface Ethernet1/1
  spanning-tree port type edge
  spanning-tree bpduguard enable

! Network ports (uplinks between switches)
interface Ethernet1/49
  spanning-tree port type network

! STP best practices with vPC
spanning-tree port type edge default    ! All access ports default to edge
spanning-tree port type network default ! All trunks default to network
```

> **Interview Tip:** With vPC + peer-switch, both vPC peers share the STP root role. This eliminates suboptimal STP paths that would occur if only one switch were root.

## 3.4 FabricPath (Legacy)

FabricPath was Cisco's proprietary L2 multipath technology before VXLAN:

- Replaced STP with IS-IS-based L2 routing
- Allowed ECMP at Layer 2
- Each switch gets a unique **switch-id**
- Frames get a FabricPath header (source/destination switch-id)
- **Deprecated** — replaced entirely by VXLAN/EVPN

> **When to use:** Never for new designs. Only exists in legacy Nexus 7000/5000 environments.

## 3.5 Port Channels and LACP

```
! Static port channel
interface port-channel 5
  switchport mode trunk

interface Ethernet1/1-2
  channel-group 5 mode on

! LACP port channel (recommended)
interface port-channel 10
  switchport mode trunk

interface Ethernet1/3-4
  channel-group 10 mode active

! LACP configuration options
interface port-channel 10
  lacp min-links 2             ! Minimum links to keep channel up
  lacp max-bundle 8            ! Maximum active links
  lacp fast-select-hot-standby ! Fast activation of standby links
```

## 3.6 Storm Control

```
interface Ethernet1/1
  storm-control broadcast level 5.00    ! 5% threshold
  storm-control multicast level 5.00
  storm-control unicast level 10.00
  storm-control action trap              ! Send SNMP trap
  storm-control action shutdown          ! Shut interface on violation
```

---

# PART 4: LAYER 3 TECHNOLOGIES

## 4.1 OSPF on NX-OS

### Key NX-OS OSPF Differences from IOS

| Feature | NX-OS | IOS |
|---------|-------|-----|
| Activation | `feature ospf` required | Always available |
| Instance naming | Alphanumeric (up to 20 chars) | Numeric only (1-65536) |
| Network statement | Interface-level `ip router ospf` | `network` under router process |
| Passive interface | Interface-level config | Router process config |
| Default reference BW | **40 Gbps** | 100 Mbps |
| Route redistribution | **Route-map REQUIRED** | Route-map optional |
| VRF support | Multiple VRFs per OSPF instance | One-to-one |
| Max equal-cost paths | 8 (expandable to 16) | 4 (expandable) |
| Router-ID selection | Loopback0 > first loopback > first physical | Different priority |
| Logging adjacency | **OFF by default** | ON by default |
| Key encryption | Auto 3DES encryption | Requires `service password` |

### OSPF Configuration on NX-OS

```
! === Enable OSPF ===
feature ospf

! === Create OSPF instance ===
router ospf UNDERLAY
  router-id 10.1.0.1
  log-adjacency-changes
  auto-cost reference-bandwidth 400000   ! 400 Gbps for modern DC

! === Configure interfaces (NX-OS style: under interface) ===
interface loopback 0
  ip address 10.1.0.1/32
  ip router ospf UNDERLAY area 0.0.0.0

interface Ethernet1/1
  no switchport
  ip address 10.1.1.0/31
  ip ospf network point-to-point          ! Always use p2p in DC
  ip router ospf UNDERLAY area 0.0.0.0
  no ip ospf passive-interface

! === Passive interface (default all, then selectively activate) ===
router ospf UNDERLAY
  passive-interface default

interface Ethernet1/1
  no ip ospf passive-interface            ! Activate OSPF on this link
```

### OSPF Area Types (Interview Must-Know)

| Area Type | Blocks | Allows | Use Case |
|-----------|--------|--------|----------|
| **Normal (Area 0)** | Nothing | All LSA types | Backbone |
| **Stub** | Type 5 (External) | Type 1,2,3 | Edge areas, no external routes needed |
| **Totally Stubby** | Type 3,4,5 | Type 1,2 only | Simplest edge area, only default route |
| **NSSA** | Type 5 | Type 7 (converted at ABR) | Edge area that needs limited redistribution |
| **Totally NSSA** | Type 3,4,5 | Type 7 | Strictest: only default + local external |

```
! Stub area
router ospf UNDERLAY
  area 1 stub

! Totally stubby area
router ospf UNDERLAY
  area 1 stub no-summary

! NSSA area
router ospf UNDERLAY
  area 2 nssa
```

### OSPF Authentication on NX-OS

```
! MD5 Authentication (per interface)
interface Ethernet1/1
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 MySecret

! SHA Authentication (more secure, RFC 5709)
key chain OSPF-KEY
  key 1
    key-string MySecureKey
    cryptographic-algorithm HMAC-SHA-256

interface Ethernet1/1
  ip ospf authentication key-chain OSPF-KEY
```

## 4.2 BGP on NX-OS

### BGP Features on NX-OS

- Multi-protocol BGP (MP-BGP) with multiple address families
- **L2VPN EVPN** address family (critical for VXLAN)
- Peer templates (session, policy, peer)
- Route reflectors
- Additional paths (advertise multiple paths per prefix)
- BFD integration for fast failure detection
- Graceful restart / NSF
- Conditional advertisement

### BGP Configuration on NX-OS

```
! === Enable BGP ===
feature bgp

! === iBGP with Route Reflector (Spine) ===
router bgp 65000
  router-id 10.1.0.1
  log-neighbor-changes

  ! Address family for underlay
  address-family ipv4 unicast
    maximum-paths 4              ! ECMP

  ! L2VPN EVPN for VXLAN overlay
  address-family l2vpn evpn
    retain route-target all      ! Route reflector: keep all RTs

  ! Template for leaf peers
  template peer LEAF-PEER
    remote-as 65000
    update-source loopback0
    address-family ipv4 unicast
      send-community both
      route-reflector-client
    address-family l2vpn evpn
      send-community both
      route-reflector-client

  ! Apply template to neighbors
  neighbor 10.1.0.11
    inherit peer LEAF-PEER
    description LEAF-1
  neighbor 10.1.0.12
    inherit peer LEAF-PEER
    description LEAF-2

! === eBGP (Leaf to Spine) ===
router bgp 65001
  router-id 10.1.0.11

  neighbor 10.1.1.0
    remote-as 65000
    address-family ipv4 unicast
    address-family l2vpn evpn
      send-community both
```

### BGP Peer Templates

Templates simplify configuration when you have many peers with identical settings:

| Template Type | Scope | Contains |
|--------------|-------|----------|
| **peer-session** | Session-level | remote-as, timers, transport, update-source |
| **peer-policy** | Address-family level | Prefix-lists, route-maps, send-community, route-reflector |
| **peer** | Combined | Both session and address-family attributes |

Templates support **single-level inheritance** — local config overrides inherited values.

## 4.3 HSRP/VRRP on NX-OS

### HSRP (Hot Standby Router Protocol)

```
feature hsrp

interface Vlan 10
  ip address 10.10.10.2/24
  hsrp version 2
  hsrp 10
    preempt
    priority 110
    ip 10.10.10.1                ! Virtual IP
    authentication md5 key-string MyKey
    timers 1 3                   ! Hello 1s, Hold 3s
```

### VRRP

```
feature vrrp

interface Vlan 20
  ip address 10.20.20.2/24
  vrrp 20
    priority 110
    address 10.20.20.1
    preempt
```

> **Critical Note for VXLAN:** In VXLAN EVPN fabrics, traditional HSRP/VRRP is **replaced** by **Distributed Anycast Gateway**. This is a fundamental architectural shift covered in Part 9.

## 4.4 VRF and VRF-Lite

```
! Create VRF
vrf context TENANT-A
  rd auto
  address-family ipv4 unicast
    route-target both auto

! Assign interface to VRF
interface Vlan 10
  vrf member TENANT-A            ! Must be BEFORE ip address
  ip address 10.10.10.1/24

! VRF-aware routing
router ospf TENANT-A
  vrf TENANT-A
    router-id 10.10.10.1

! VRF-aware BGP
router bgp 65001
  vrf TENANT-A
    address-family ipv4 unicast
    neighbor 10.10.10.2
      remote-as 65002
      address-family ipv4 unicast
```

## 4.5 PIM (Protocol Independent Multicast) on NX-OS

**NX-OS does NOT support PIM Dense Mode.** Only Sparse Mode and SSM.

```
! === Enable PIM ===
feature pim

! === Configure RP (Rendezvous Point) ===
! Static RP
ip pim rp-address 10.1.0.100 group-list 239.0.0.0/8

! Anycast RP (for redundancy)
interface loopback 100
  ip address 10.1.0.100/32
  ip pim sparse-mode
ip pim anycast-rp 10.1.0.100 10.1.0.1    ! RP source 1
ip pim anycast-rp 10.1.0.100 10.1.0.2    ! RP source 2

! === Enable PIM on interfaces ===
interface Ethernet1/1
  ip pim sparse-mode

! === SSM (Source-Specific Multicast) ===
ip pim ssm range 232.0.0.0/8

! === BSR (Bootstrap Router) — alternative to static RP ===
ip pim bsr bsr-candidate Ethernet1/1 hash-len 24 priority 64
ip pim rp-candidate Ethernet1/1 group-list 239.0.0.0/24
```

> **Important:** Do NOT configure both Auto-RP and BSR in the same network.

> **vPC + PIM:** PIM SSM is supported on vPC (NX-OS 7.0(3)I4(1)+), but PIM adjacency between SVI on vPC VLAN and downstream device is NOT supported.

## 4.6 Policy-Based Routing (PBR)

```
! Create route-map
route-map PBR-POLICY permit 10
  match ip address ACL-WEB-TRAFFIC
  set ip next-hop 10.1.1.1

! Apply to interface
interface Vlan 10
  ip policy route-map PBR-POLICY
```

---

# PART 5: NX-OS vs IOS DIFFERENCES

This is a guaranteed interview topic. Key differences at a glance:

| Feature | NX-OS | IOS/IOS-XE |
|---------|-------|------------|
| **Feature activation** | `feature <name>` required | Features always available |
| **Process model** | Linux kernel, separate processes | Monolithic (IOS) / Linux (IOS-XE) |
| **VDC support** | Yes (Nexus 7000) | No (VSS/StackWise instead) |
| **ISSU** | Non-disruptive upgrades | Limited/disruptive |
| **Configuration mode** | Interface-centric (OSPF/PIM under interface) | Router-process centric |
| **Routing instance names** | Alphanumeric | Numeric only |
| **Default STP** | Rapid-PVST+ | PVST+ |
| **Redistribution** | Route-map **mandatory** | Route-map optional |
| **NX-API** | REST API built-in | Limited (RESTCONF/NETCONF) |
| **Password encryption** | Auto 3DES | Manual `service password-encryption` |
| **VRF per OSPF** | Multiple VRFs per instance | One VRF per instance |
| **Reference bandwidth** | 40 Gbps | 100 Mbps |
| **Adjacency logging** | OFF by default | ON by default |

### NX-OS Unique Concepts

1. **Feature-based licensing:** Features must be explicitly enabled (`feature vpc`, `feature bgp`, etc.)
2. **Checkpointing & Rollback:** `checkpoint` saves config state; `rollback` reverts
3. **NX-API (NXAPI):** REST/JSON API for programmability
4. **Scheduler:** Built-in job scheduler for automated tasks
5. **PowerOn Auto Provisioning (POAP):** Zero-touch deployment
6. **EEM (Embedded Event Manager):** Same as IOS but with Python scripting

---

# PART 6: NEXUS VS CATALYST FAMILY COMPARISON

| Aspect | Nexus | Catalyst |
|--------|-------|---------|
| **Target** | Data Center | Campus / Enterprise / Branch |
| **OS** | NX-OS or ACI | IOS-XE (9000), IOS (legacy) |
| **Fabric Technology** | VXLAN/EVPN, ACI | SD-Access (LISP/VXLAN) |
| **Controller** | APIC (ACI), Nexus Dashboard | Cisco DNA Center / Catalyst Center |
| **Port Speeds** | 10G-400G dominant | 1G-25G dominant, some 100G |
| **Low Latency** | Sub-microsecond (N3K) | Not optimized |
| **FCoE/Storage** | Yes (N5K/N7K) | No |
| **VDC** | Yes (N7K) | No |
| **StackWise** | No | Yes (C9200/9300) |
| **Typical Scale** | 100s-1000s of ports per fabric | 10s-100s of ports per closet |
| **Redundancy** | vPC, VXLAN EVPN | StackWise Virtual, VSS |
| **Power** | DC power options | PoE/PoE+ for endpoints |
| **VXLAN Support** | Native, optimized | Supported (Catalyst 9000) |

> **Key Interview Point:** Catalyst 9000 now supports VXLAN/EVPN (IOS-XE), blurring the line. However, Nexus remains the choice for high-density DC with ACI, advanced telemetry, and sub-microsecond latency requirements.

---

# PART 7: VXLAN FUNDAMENTALS (For Legacy Engineers)

## 7.1 Why VXLAN Exists — The Problem with VLANs

As a legacy VLAN engineer, here's what you already know and what changes:

**VLAN Limitations:**
1. **Only 4,094 VLANs** (12-bit VLAN ID) — not enough for multi-tenant cloud
2. **STP dependency** — blocks redundant links, slow convergence
3. **No mobility** — VLANs bound to physical topology
4. **L2 domain = broadcast domain** — doesn't scale beyond campus
5. **No multi-tenancy** — VLANs are flat, no isolation between tenants

**VXLAN Solves All of These:**
1. **16 million segments** (24-bit VNI)
2. **No STP** — uses L3 ECMP routing in the underlay
3. **Workload mobility** — extend L2 anywhere over L3
4. **Scalable** — overlay/underlay separation
5. **Multi-tenancy** — VRF + VNI provides full isolation

## 7.2 What is VXLAN?

**VXLAN (Virtual Extensible LAN)** is defined in RFC 7348. It is a **MAC-in-UDP encapsulation** technology that creates Layer 2 overlay networks on top of a Layer 3 underlay network.

In simple terms: VXLAN takes a regular Ethernet frame, wraps it in a UDP packet with a VXLAN header, and sends it across the IP network. The receiving end unwraps it and delivers the original Ethernet frame.

## 7.3 VXLAN Packet Format

```
┌──────────────────────────────────────────────────────────────────┐
│                    Outer Ethernet Header                         │
│  (Src MAC = Egress VTEP MAC, Dst MAC = Next-hop MAC)           │
├──────────────────────────────────────────────────────────────────┤
│                    Outer IP Header (20 bytes)                    │
│  (Src IP = Source VTEP IP, Dst IP = Destination VTEP IP)        │
├──────────────────────────────────────────────────────────────────┤
│                    Outer UDP Header (8 bytes)                    │
│  Src Port = Hash of inner headers    Dst Port = 4789 (IANA)    │
├──────────────────────────────────────────────────────────────────┤
│                    VXLAN Header (8 bytes)                        │
│  Flags: I = 1 (valid VNI)    VNI: 24-bit identifier            │
│  Reserved bits (must be 0)                                       │
├──────────────────────────────────────────────────────────────────┤
│                    Inner Ethernet Frame (Original)               │
│  (Original Src MAC, Dst MAC, Payload)                           │
└──────────────────────────────────────────────────────────────────┘
```

**Key Details:**
- **Outer UDP Dst Port:** 4789 (IANA standard)
- **Outer UDP Src Port:** Hash of inner frame headers → enables ECMP load balancing
- **VNI (VXLAN Network Identifier):** 24-bit field → 16,777,216 possible segments
- **MTU overhead:** ~50 bytes (Outer Eth 14 + IP 20 + UDP 8 + VXLAN 8) — set underlay MTU to **9216** (jumbo frames)

## 7.4 VTEP (VXLAN Tunnel Endpoint)

A VTEP is the device that performs VXLAN encapsulation and decapsulation.

```
                       Underlay Network (L3)
                    ┌──────────────────────────┐
                    │    IP Routed Fabric       │
                    │                            │
   ┌────────┐      │      ┌────────┐           │      ┌────────┐
   │ VTEP 1 │◄─────┼──────│ Spine  │───────────┼─────►│ VTEP 2 │
   │(Leaf 1)│      │      └────────┘           │      │(Leaf 2)│
   └───┬────┘      │                            │      └───┬────┘
       │            └──────────────────────────┘           │
   ┌───┴───┐           VXLAN Tunnel                    ┌───┴───┐
   │Server │◄ ═ ═ ═ ═ (Overlay L2) ═ ═ ═ ═ ═ ═ ═ ═ ►│Server │
   │  A    │           VNI 10010                       │  B    │
   └───────┘                                           └───────┘
   VLAN 10                                              VLAN 10
```

- VTEP has a **loopback IP** used as the tunnel source (NVE interface)
- Each VTEP maps local VLANs to VNIs
- VTEP encapsulates egress frames and decapsulates ingress frames

### NVE Interface (Network Virtualization Edge)

The NVE interface is the VTEP on NX-OS:

```
interface nve1
  no shutdown
  host-reachability protocol bgp        ! Use BGP EVPN as control plane
  source-interface loopback1            ! VTEP IP address
  member vni 10010                      ! L2 VNI for VLAN 10
    ingress-replication protocol bgp    ! BUM via ingress replication
  member vni 50001 associate-vrf        ! L3 VNI for inter-VNI routing
```

## 7.5 How VLANs Map to VXLANs

This is the bridge from your legacy VLAN knowledge:

```
VLAN 10  ──────►  VNI 10010  (L2 VNI)
VLAN 20  ──────►  VNI 10020  (L2 VNI)
VLAN 30  ──────►  VNI 10030  (L2 VNI)
     └──── All in Tenant VRF-A ──────►  VNI 50001  (L3 VNI)
```

**Configuration mapping:**

```
! VLAN to VNI mapping
vlan 10
  vn-segment 10010           ! This VLAN = VNI 10010

vlan 20
  vn-segment 10020           ! This VLAN = VNI 10020

! L3 VNI (for inter-VLAN routing across fabric)
vlan 2001
  vn-segment 50001           ! Transit VLAN for L3 VNI
```

**Think of it this way:**
- **L2 VNI** = Your VLAN extended across the fabric (same broadcast domain everywhere)
- **L3 VNI** = The "router" that connects different VNIs within a tenant (VRF)

## 7.6 How VXLAN Maps to EVPN

EVPN (Ethernet VPN) is the **control plane** for VXLAN. Without EVPN, VXLAN uses flood-and-learn (like traditional Ethernet) which doesn't scale.

```
VXLAN VNI 10010  ──────►  EVPN Instance (EVI)  ──────►  RD + RT
                           │
                           ├── Route Type 2: MAC/IP learned on this VNI
                           ├── Route Type 3: VTEP discovery for BUM
                           └── Route Type 5: IP prefix routes (inter-VNI)
```

**Mapping relationships:**
- Each **VNI** maps to an **EVPN Instance (EVI)**
- Each EVI has a **Route Distinguisher (RD)** making it unique in BGP
- Each EVI has **Route Targets (RT)** controlling import/export between VTEPs

```
! EVPN Instance configuration
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```

---

# PART 8: BGP EVPN DEEP DIVE

## 8.1 What is BGP EVPN?

BGP EVPN (Ethernet Virtual Private Network) is an extension to BGP that allows it to carry **Layer 2 (MAC) and Layer 3 (IP) reachability information** using the Multi-Protocol BGP (MP-BGP) framework.

**Defined in:** RFC 7432 (EVPN), RFC 8365 (EVPN as NVO solution with VXLAN)

**Key Properties:**
- Uses **MP-BGP** as transport mechanism
- Address Family Identifier (AFI): **25** (L2VPN)
- Subsequent Address Family Identifier (SAFI): **70** (EVPN)
- Carries MAC addresses, IP addresses, and IP prefixes in BGP NLRI
- Uses **extended communities** for Route Targets
- Provides a **control-plane based** MAC learning mechanism (replaces flood-and-learn)

## 8.2 Why BGP EVPN? (vs Flood-and-Learn)

| Feature | Flood-and-Learn (No EVPN) | BGP EVPN |
|---------|--------------------------|----------|
| MAC Learning | Data-plane flooding | Control-plane (BGP) |
| BUM Handling | Multicast underlay required | Ingress replication OR multicast |
| ARP Handling | Flood | **ARP suppression** (answer locally) |
| Convergence | Slow (relearn MACs) | Fast (BGP withdrawal) |
| Multi-tenancy | Limited | Full (VRF + RT/RD) |
| Mobility | Complex | Built-in MAC mobility |
| Security | Open | Only VTEPs known to control plane |

## 8.3 EVPN NLRI Format

```
┌─────────────────────────────────────────────┐
│  Route Type (1 byte)                         │
├─────────────────────────────────────────────┤
│  Length (1 byte)                              │
├─────────────────────────────────────────────┤
│  Route Type Specific Data (variable)         │
└─────────────────────────────────────────────┘
```

## 8.4 EVPN Route Types — MUST KNOW ALL FIVE

### Route Type 1: Ethernet Auto-Discovery (EAD)

**Purpose:** Multihoming support — aliasing and fast convergence

**Two sub-types:**
- **EAD per ES (Ethernet Segment):** Signals membership in an Ethernet Segment
- **EAD per EVI:** Signals per-VNI membership for aliasing

**How it works:**
- When a host is multihomed to two leaf switches (via same Ethernet Segment)
- Both leaves advertise Type 1 routes with the same ESI
- Remote VTEPs learn that traffic to this host can go via **either** leaf (ECMP)
- If one leaf loses connectivity, it **withdraws** its Type 1 route
- Remote VTEPs immediately remove that leaf from ECMP (fast convergence)

**Analogy for legacy engineers:** Think of it as "advertising that I'm a member of a port channel group, and here's my group ID (ESI)."

### Route Type 2: MAC/IP Advertisement

**Purpose:** The most important route type — advertises MAC addresses and optionally IP addresses

**Contains:**
- Ethernet Segment Identifier (ESI)
- Ethernet Tag ID
- MAC Address
- IP Address (optional — can carry both MAC and IP)
- MPLS Label / VNI

**How it works:**
1. Host A (MAC aa:bb:cc:dd:ee:ff, IP 10.10.10.5) connects to Leaf 1
2. Leaf 1 learns the MAC via data-plane (or ARP)
3. Leaf 1 generates a Type 2 route: "MAC aa:bb:cc:dd:ee:ff / IP 10.10.10.5 is reachable via me (VTEP 10.1.0.11) on VNI 10010"
4. BGP distributes this to all VTEPs in the fabric
5. Other VTEPs now know how to reach this MAC **without flooding**

**Analogy:** This is like a CAM table entry being advertised via BGP instead of being learned by flooding.

### Route Type 3: Inclusive Multicast Ethernet Tag (IMET)

**Purpose:** VTEP discovery and BUM (Broadcast, Unknown unicast, Multicast) handling

**How it works:**
1. Each VTEP advertises a Type 3 route for each VNI it participates in
2. The Type 3 route says: "I (VTEP 10.1.0.11) am interested in VNI 10010"
3. This builds the **ingress replication list** — the list of VTEPs that should receive BUM traffic for that VNI
4. When a VTEP needs to flood a frame (unknown unicast, broadcast, multicast), it sends individual copies to each VTEP in the list

**Analogy:** This is like saying "subscribe me to the multicast group for VLAN 10."

### Route Type 4: Ethernet Segment Route

**Purpose:** Designated Forwarder (DF) election in multihoming scenarios

**How it works:**
1. When multiple VTEPs share the same Ethernet Segment (ESI), they all advertise Type 4 routes
2. VTEPs exchange Type 4 routes to discover who else is on the same segment
3. DF election occurs: one VTEP is elected as **Designated Forwarder** per VLAN
4. Only the DF forwards BUM traffic for that VLAN on the shared segment (prevents loops)
5. Non-DF VTEPs forward known unicast only

**Analogy:** Think of it as the STP root election, but for a multi-homed Ethernet Segment. Only one switch forwards broadcasts.

### Route Type 5: IP Prefix Route

**Purpose:** Advertise IP subnets/prefixes (not just host routes) into the EVPN fabric

**Defined in:** RFC 9136

**How it works:**
1. A border leaf learns external routes (from WAN, internet, other DC)
2. It advertises these as Type 5 routes: "Prefix 172.16.0.0/16 is reachable via me"
3. All VTEPs in the fabric install these prefixes in the appropriate VRF routing table
4. Traffic is VXLAN-encapsulated to the border leaf and then forwarded externally

**Use cases:**
- Advertising external prefixes into the fabric
- Inter-subnet routing between data centers
- Advertising subnet routes for silent hosts

**Analogy:** This is like redistributing external routes into your fabric's BGP EVPN control plane.

### Route Type Summary Table

| Type | Name | Purpose | When Generated |
|------|------|---------|---------------|
| 1 | Ethernet Auto-Discovery | Multihoming aliasing + fast convergence | Multihomed leaf joins ES |
| 2 | MAC/IP Advertisement | MAC + IP learning (most common) | Host MAC/ARP learned |
| 3 | Inclusive Multicast | VTEP discovery, BUM replication list | VTEP joins a VNI |
| 4 | Ethernet Segment | DF election for multihoming | Multihomed leaf on shared ES |
| 5 | IP Prefix | External/subnet route advertisement | External route learned at border |

## 8.5 Route Distinguisher (RD) and Route Target (RT)

**Route Distinguisher (RD):**
- Makes routes globally unique in BGP
- Format: `<IP>:<number>` or `<ASN>:<number>`
- `rd auto` generates from router-id + VNI
- Each VTEP should have a **unique RD** (critical for border nodes)

**Route Target (RT):**
- Controls route import/export between VTEPs
- Export RT: "Tag this route with this community"
- Import RT: "Accept routes with this community tag"
- `route-target both auto` generates from ASN + VNI

```
! Example: routes flow like this
VTEP-1 exports: RD 10.1.0.11:10010, RT 65000:10010
    └──► BGP carries to Route Reflector
         └──► VTEP-2 imports routes matching RT 65000:10010
```

## 8.6 ARP Suppression

One of EVPN's most powerful features:

**Without ARP Suppression:**
1. Host A sends ARP broadcast "Who has 10.10.10.5?"
2. ARP flooded to ALL VTEPs in the VNI
3. Every VTEP processes and floods to local segments
4. Massive unnecessary broadcast traffic

**With ARP Suppression:**
1. Host A sends ARP broadcast "Who has 10.10.10.5?"
2. Local VTEP checks its EVPN database (learned from Type 2 routes)
3. VTEP already knows: "10.10.10.5 = MAC aa:bb:cc:dd:ee:ff"
4. VTEP generates ARP reply **locally** → no flooding!
5. Massive reduction in BUM traffic

```
! Enable ARP suppression
interface nve1
  member vni 10010
    suppress-arp
```

---

# PART 9: VXLAN FABRIC OPERATIONS

## 9.1 Distributed Anycast Gateway — Replacing HSRP/VRRP

### The Legacy Problem

In traditional networks with HSRP/VRRP:

```
    ┌────────┐         ┌────────┐
    │Switch A│         │Switch B│
    │HSRP Act│         │HSRP Sby│
    │.1 (VIP)│         │.3      │
    └───┬────┘         └───┬────┘
        │                  │
    ════╪══════════════════╪════  VLAN 10: 10.10.10.0/24
        │                  │
    ┌───┴───┐          ┌───┴───┐
    │Host A │          │Host B │
    │GW:.1  │          │GW:.1  │
    └───────┘          └───────┘
```

**Problems:**
1. Only the **Active** router forwards traffic — Standby wastes capacity
2. Traffic from Host B must cross to Switch A (**tromboning**)
3. If Active fails, convergence takes seconds (HSRP hello timers)
4. VIP moves with the Active role — not truly distributed

### The VXLAN Solution: Distributed Anycast Gateway

In VXLAN EVPN, **every leaf switch is the default gateway** with the **same IP and MAC**:

```
    ┌────────┐                    ┌────────┐
    │ Leaf 1 │                    │ Leaf 2 │
    │GW: .1  │                    │GW: .1  │
    │MAC:0001│                    │MAC:0001│  ← Same IP AND MAC!
    └───┬────┘                    └───┬────┘
        │        Spine Layer          │
    ════╪═══════════════════════════╪════  VNI 10010
        │                            │
    ┌───┴───┐                    ┌───┴───┐
    │Host A │                    │Host B │
    │GW:.1  │                    │GW:.1  │
    └───────┘                    └───────┘
```

**How it works:**
1. **Every** leaf switch configured with the same SVI IP (e.g., 10.10.10.1/24)
2. **Every** leaf uses the same **anycast gateway MAC** (e.g., 0000.2222.3333)
3. The host's default gateway is always its **closest leaf switch**
4. No tromboning — traffic is routed at the first hop
5. No failover delay — if one leaf fails, the host's gateway is still at another leaf

**Configuration:**

```
! Global anycast gateway MAC (same on ALL leaves)
fabric forwarding anycast-gateway-mac 0000.2222.3333

! SVI with anycast gateway (same IP on ALL leaves)
interface Vlan 10
  no shutdown
  vrf member TENANT-A
  ip address 10.10.10.1/24
  fabric forwarding mode anycast-gateway    ! Enable anycast gateway
```

> **Interview Critical Point:** The `fabric forwarding anycast-gateway-mac` must be the **same** on every leaf in the fabric. The SVI IP address is also the **same** on every leaf. This is fundamentally different from HSRP where only one switch owns the VIP.

## 9.2 L2 VNI vs L3 VNI

### L2 VNI (Bridging)
- Maps to a single VLAN / broadcast domain
- Provides **Layer 2 connectivity** between hosts in the same subnet across the fabric
- Used for MAC learning and BUM forwarding
- One L2 VNI per VLAN

### L3 VNI (Routing)
- Maps to a **VRF** (tenant)
- Provides **inter-subnet routing** across the fabric
- One L3 VNI per VRF (not per VLAN)
- Uses a **transit VLAN** that doesn't carry user traffic

```
! L2 VNI: VLAN 10 = VNI 10010 (bridging for subnet 10.10.10.0/24)
vlan 10
  vn-segment 10010

! L2 VNI: VLAN 20 = VNI 10020 (bridging for subnet 10.20.20.0/24)
vlan 20
  vn-segment 10020

! L3 VNI: Transit VLAN 2001 = VNI 50001 (routing for VRF TENANT-A)
vlan 2001
  vn-segment 50001

interface Vlan 2001
  no shutdown
  vrf member TENANT-A
  ip forward
  no ip redirects
  ipv6 address use-link-local-only
```

### How Inter-Subnet Routing Works

When Host A (VLAN 10, 10.10.10.5) wants to reach Host B (VLAN 20, 10.20.20.5):

**Symmetric IRB (Integrated Routing and Bridging):**
1. Host A sends packet to default gateway (Leaf 1, anycast gateway)
2. Leaf 1 routes the packet from VNI 10010 to VRF TENANT-A
3. Leaf 1 looks up destination: EVPN Type 2 says Host B is on Leaf 2
4. Leaf 1 encapsulates with **L3 VNI 50001** and sends to Leaf 2
5. Leaf 2 decapsulates, routes from VRF into VNI 10020
6. Leaf 2 delivers to Host B on VLAN 20

> **Symmetric IRB** means both ingress and egress VTEPs perform routing. This is the **recommended** and most common mode on Nexus.

## 9.3 ECMP in VXLAN Fabric

### How Underlay ECMP Works

The underlay network uses **IP routing** (OSPF or IS-IS) between all switches. In a leaf-spine design:

```
              ┌─────────┐    ┌─────────┐
              │ Spine 1 │    │ Spine 2 │
              └──┬──┬───┘    └───┬──┬──┘
                 │  │            │  │
         ┌───────┘  └──┐  ┌─────┘  └───────┐
         │             │  │                  │
    ┌────┴───┐    ┌────┴──┴───┐    ┌────────┴┐
    │ Leaf 1 │    │  Leaf 2   │    │  Leaf 3  │
    └────────┘    └───────────┘    └──────────┘
```

- Leaf 1 has **two equal-cost paths** to reach Leaf 3: via Spine 1 and Spine 2
- The underlay IGP (OSPF/IS-IS) installs both paths as ECMP
- VXLAN outer UDP source port is a **hash of inner headers** → enables per-flow load balancing
- Adding more spines = more ECMP paths = more bandwidth

### How ECMP Is Maintained

```
! OSPF with ECMP
router ospf UNDERLAY
  maximum-paths 16

! BGP with ECMP
router bgp 65000
  address-family ipv4 unicast
    maximum-paths 4          ! eBGP ECMP
    maximum-paths ibgp 4     ! iBGP ECMP
```

## 9.4 Multicast Underlay vs Ingress Replication

For handling BUM (Broadcast, Unknown Unicast, Multicast) traffic:

### Option 1: Multicast Underlay
- Assign a multicast group per VNI (or shared across VNIs)
- BUM traffic is sent to the multicast group
- Spines act as PIM Rendezvous Points
- **Pro:** Efficient for large-scale BUM
- **Con:** Requires multicast configuration in underlay (complex)

### Option 2: Ingress Replication (Head-End Replication)
- VTEP sends individual unicast copies to every other VTEP in the VNI
- VTEP list built from EVPN Type 3 routes
- **Pro:** No multicast needed — simpler underlay
- **Con:** Higher load on sending VTEP for BUM traffic

```
! Ingress Replication (Recommended for most deployments)
interface nve1
  member vni 10010
    ingress-replication protocol bgp

! Multicast-based
interface nve1
  member vni 10010
    mcast-group 239.1.1.10
```

> **Interview Guidance:** Modern deployments overwhelmingly use **Ingress Replication** because it simplifies the underlay. Multicast underlay is only used in very large fabrics with high BUM traffic.

---

# PART 10: VXLAN CONFIGURATION EXAMPLES

## 10.1 Complete Leaf Switch Configuration

```
! ============================================================
! LEAF SWITCH - Full VXLAN BGP EVPN Configuration
! ============================================================

! === Step 1: Enable Required Features ===
feature ospf
feature bgp
feature pim
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

nv overlay evpn

! === Step 2: Configure Anycast Gateway MAC (SAME ON ALL LEAVES) ===
fabric forwarding anycast-gateway-mac 0000.2222.3333

! === Step 3: Create VLANs with VNI Mapping ===
vlan 10
  name SERVERS
  vn-segment 10010              ! L2 VNI

vlan 20
  name DATABASE
  vn-segment 10020              ! L2 VNI

vlan 2001
  name L3VNI-TENANT-A
  vn-segment 50001              ! L3 VNI (transit)

! === Step 4: Create VRF with L3 VNI ===
vrf context TENANT-A
  vni 50001
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
  address-family ipv6 unicast
    route-target both auto
    route-target both auto evpn

! === Step 5: Configure Loopbacks ===
interface loopback 0
  description Router-ID / BGP Peering
  ip address 10.1.0.11/32
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback 1
  description VTEP Source (NVE)
  ip address 10.1.1.11/32
  ip router ospf UNDERLAY area 0.0.0.0

! === Step 6: Configure Underlay (OSPF) ===
router ospf UNDERLAY
  router-id 10.1.0.11

interface Ethernet1/49
  description To Spine-1
  no switchport
  mtu 9216
  ip address 10.1.100.1/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/50
  description To Spine-2
  no switchport
  mtu 9216
  ip address 10.1.100.3/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

! === Step 7: Configure BGP with EVPN ===
router bgp 65000
  router-id 10.1.0.11
  log-neighbor-changes
  address-family ipv4 unicast
  address-family l2vpn evpn

  neighbor 10.1.0.1
    remote-as 65000
    description Spine-1 (RR)
    update-source loopback0
    address-family ipv4 unicast
      send-community both
    address-family l2vpn evpn
      send-community both

  neighbor 10.1.0.2
    remote-as 65000
    description Spine-2 (RR)
    update-source loopback0
    address-family ipv4 unicast
      send-community both
    address-family l2vpn evpn
      send-community both

  vrf TENANT-A
    address-family ipv4 unicast
      advertise l2vpn evpn
      redistribute direct route-map PERMIT-ALL
      maximum-paths ibgp 2

! === Step 8: Configure NVE (VTEP) Interface ===
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 50001 associate-vrf

! === Step 9: Configure SVIs with Anycast Gateway ===
interface Vlan 10
  no shutdown
  vrf member TENANT-A
  ip address 10.10.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan 20
  no shutdown
  vrf member TENANT-A
  ip address 10.20.20.1/24
  fabric forwarding mode anycast-gateway

! L3 VNI SVI (transit — no user traffic)
interface Vlan 2001
  no shutdown
  vrf member TENANT-A
  ip forward
  no ip redirects
  ipv6 address use-link-local-only
  no ipv6 redirects

! === Step 10: Configure EVPN ===
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto

! === Step 11: Access Ports ===
interface Ethernet1/1
  description Server-1
  switchport mode access
  switchport access vlan 10
  no shutdown

interface Ethernet1/2
  description DB-Server-1
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  no shutdown

! === Route-map for redistribution ===
route-map PERMIT-ALL permit 10
```

## 10.2 Complete Spine Switch Configuration

```
! ============================================================
! SPINE SWITCH - Route Reflector for VXLAN EVPN
! ============================================================

feature ospf
feature bgp
feature pim

! === Loopback ===
interface loopback 0
  description Router-ID
  ip address 10.1.0.1/32
  ip router ospf UNDERLAY area 0.0.0.0

! === Underlay ===
router ospf UNDERLAY
  router-id 10.1.0.1

interface Ethernet1/1
  description To Leaf-1
  no switchport
  mtu 9216
  ip address 10.1.100.0/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description To Leaf-2
  no switchport
  mtu 9216
  ip address 10.1.100.2/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description To Leaf-3
  no switchport
  mtu 9216
  ip address 10.1.100.4/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

! === BGP Route Reflector ===
router bgp 65000
  router-id 10.1.0.1
  log-neighbor-changes

  address-family ipv4 unicast
  address-family l2vpn evpn
    retain route-target all          ! Keep all RTs (even non-local)

  ! Template for leaf peers
  template peer LEAF
    remote-as 65000
    update-source loopback0
    address-family ipv4 unicast
      send-community both
      route-reflector-client         ! Reflect routes to leaves
    address-family l2vpn evpn
      send-community both
      route-reflector-client

  neighbor 10.1.0.11
    inherit peer LEAF
    description Leaf-1

  neighbor 10.1.0.12
    inherit peer LEAF
    description Leaf-2

  neighbor 10.1.0.13
    inherit peer LEAF
    description Leaf-3
```

> **Note:** The spine does NOT run NVE. It has NO VTEPs, NO VNIs, NO VLANs. It is purely an underlay router and BGP route reflector.

## 10.3 Border Leaf Configuration (External Connectivity)

```
! ============================================================
! BORDER LEAF - External Connectivity via VRF-Lite
! ============================================================

! (All standard leaf config above, PLUS:)

! === External-facing sub-interface ===
interface Ethernet1/3
  no switchport
  no shutdown

interface Ethernet1/3.2
  description To External Router (VRF-Lite)
  encapsulation dot1q 2
  vrf member TENANT-A
  ip address 10.31.95.31/24
  no shutdown

! === VRF Context ===
vrf context TENANT-A
  vni 50001
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

! === BGP Peering to External Router ===
router bgp 65000
  vrf TENANT-A
    router-id 10.31.95.31
    address-family ipv4 unicast
      advertise l2vpn evpn
      maximum-paths 2
      maximum-paths ibgp 2
    neighbor 10.31.95.95
      remote-as 65099
      description External-Router
      address-family ipv4 unicast
        route-map EXTCON-FILTER out

! === Route Filtering (prevent host routes and default route leaking) ===
ip prefix-list DEFAULT-ROUTE seq 5 permit 0.0.0.0/0 le 1
ip prefix-list HOST-ROUTE seq 5 permit 0.0.0.0/0 eq 32

route-map EXTCON-FILTER deny 10
  match ip address prefix-list DEFAULT-ROUTE
route-map EXTCON-FILTER deny 20
  match ip address prefix-list HOST-ROUTE
route-map EXTCON-FILTER permit 1000
```

---

# PART 11: CISCO NEXUS DASHBOARD AND NDFC

## 11.1 What is Nexus Dashboard?

Cisco Nexus Dashboard is a **unified operations platform** that hosts multiple services for data center network management. Think of it as the "operating system" for DC network operations.

**Key Services on Nexus Dashboard:**
- **NDFC (Nexus Dashboard Fabric Controller)** — Fabric deployment and management (formerly DCNM)
- **NDI (Nexus Dashboard Insights)** — Analytics, assurance, and troubleshooting
- **NDO (Nexus Dashboard Orchestrator)** — Multi-site and multi-domain orchestration

## 11.2 NDFC (Nexus Dashboard Fabric Controller)

NDFC is the primary tool for deploying and managing VXLAN EVPN fabrics on Nexus switches.

### Day-0 Operations: Building the Fabric

**Step 1: Create Fabric**
- Select template: **Data Center VXLAN EVPN**
- Provide: Fabric name, BGP ASN
- NDFC generates underlay configuration templates

**Step 2: Discover Switches**
- Methods: Seed IP discovery or POAP (Power-On Auto Provisioning)
- **Greenfield:** NDFC performs write-erase + reload, pushes fresh config
- **Brownfield:** NDFC preserves existing config, learns topology

**Step 3: Assign Switch Roles**
- Leaf, Spine, Border Leaf, Border Gateway, Super Spine, ToR
- NDFC auto-detects topology and connections

**Step 4: Configure vPC Pairing**
- One-click vPC pair automation
- Generates domain, peer-link, keepalive configuration

**Step 5: Deploy Underlay**
- **Recalculate & Deploy** → NDFC generates and pushes:
  - OSPF/IS-IS routing
  - Loopback addresses
  - P2P links
  - PIM/RP configuration (if multicast mode)
  - BGP sessions with route reflectors

### Day-1 Operations: Overlay Provisioning

**Network Creation:**
- Create L2 networks (VNIs) with gateway
- Create L3 VRFs for multi-tenancy
- Attach networks to leaf interfaces and ports
- Configure VRF-Lite for external connectivity

**Key Workflow:**
1. Define VRF (tenant) with VNI
2. Create networks (subnets) within VRF
3. Attach networks to switches and interfaces
4. Preview generated config
5. Deploy

**Deployment modes:**
- **Config-profile:** Bundled overlay configurations
- **CLI mode:** Raw NX-OS CLI deployment

### Day-2 Operations: Ongoing Management

- **Configuration Compliance:** Periodic checks for drift from intended state
- **Change Control:** Rollback capability at ticket level
- **Performance Monitoring:** SNMP-based CPU, memory, traffic metrics
- **Image Management:** NX-OS upgrades across fabric
- **Topology Visualization:** Real-time fabric topology view

### NDFC Automation Capabilities

| Feature | Description |
|---------|-------------|
| Template Framework | Out-of-box policies + custom freeform CLI |
| Resource Management | Automatic IP, VLAN, VNI, port-channel allocation |
| Save-Preview-Deploy | Review generated config before pushing |
| Multi-Site (MSD) | Manage multiple VXLAN fabrics as one domain |
| Brownfield Import | Migrate existing networks into NDFC management |
| REST API | Full programmatic access to all NDFC functions |

### NDFC Fabric Templates

| Template | Use Case |
|----------|----------|
| Data Center VXLAN EVPN | Primary — Nexus 3000/9000 VXLAN fabric |
| Enhanced Classic LAN | Non-VXLAN, traditional L2/L3 management |
| External Fabric | WAN/external device management |
| LAN Classic | Legacy DCNM compatibility |
| MSD (Multi-Site Domain) | Multi-fabric orchestration |

---

# PART 12: VXLAN VS LEGACY NETWORKS

## 12.1 Side-by-Side Comparison

| Aspect | Legacy VLAN Network | VXLAN EVPN Fabric |
|--------|-------------------|-------------------|
| **Scalability** | 4,094 VLANs | 16 million VNIs |
| **Architecture** | 3-tier (Core/Dist/Access) | Leaf-Spine (Clos) |
| **Loop Prevention** | STP (blocks links) | L3 ECMP (all links active) |
| **MAC Learning** | Flood-and-learn | BGP EVPN control plane |
| **Gateway** | HSRP/VRRP (active/standby) | Distributed Anycast (all active) |
| **Multi-Tenancy** | VRF-Lite (complex) | VRF + VNI (built-in) |
| **Workload Mobility** | Limited to VLAN span | Anywhere in fabric |
| **ARP Handling** | Broadcast flooding | ARP suppression |
| **Bandwidth Utilization** | ~50% (STP blocking) | ~100% (ECMP) |
| **Convergence** | Seconds (STP) | Sub-second (BGP) |
| **DCI (Data Center Interconnect)** | OTV, VPLS (complex) | VXLAN Multi-Site (native) |
| **Automation** | Manual / limited | NDFC full lifecycle |
| **Troubleshooting** | show mac address-table | show bgp l2vpn evpn, show nve peers |

## 12.2 Migration Path: VLAN to VXLAN

For a legacy engineer, the conceptual mapping:

```
Legacy Concept          →    VXLAN Equivalent
──────────────────────────────────────────────────
VLAN 10                 →    VNI 10010 (L2 VNI)
Trunk                   →    VXLAN Tunnel (NVE)
STP Root Bridge         →    Not needed (L3 ECMP)
HSRP Virtual IP         →    Anycast Gateway (same IP on ALL leaves)
HSRP Virtual MAC        →    Anycast Gateway MAC (same on ALL leaves)
Access Layer Switch     →    Leaf Switch
Distribution Switch     →    Not needed (leaf routes directly)
Core Switch             →    Spine Switch
Inter-VLAN Routing      →    L3 VNI + Symmetric IRB
VRF-Lite to WAN         →    Border Leaf + VRF-Lite
Spanning Tree           →    Nothing (ECMP handles redundancy)
Port Channel to server  →    Still port channel (or vPC at leaf pair)
```

## 12.3 Key Mindset Shifts

1. **Stop thinking in L2 paths** — The underlay is all L3. Even "L2" traffic (same VLAN) is IP-routed between VTEPs.

2. **Every leaf is a router AND a gateway** — No more dedicated distribution layer for routing.

3. **No more STP** — Redundancy comes from multiple spines and ECMP. All links forward traffic.

4. **MAC tables are distributed via BGP** — You don't learn MACs by flooding anymore.

5. **MTU matters** — VXLAN adds 50 bytes overhead. Set underlay to 9216 (jumbo frames).

6. **Troubleshooting changes** — Instead of `show mac address-table`, you use:
   - `show bgp l2vpn evpn`
   - `show nve peers`
   - `show nve vni`
   - `show l2route evpn mac all`

---

# PART 13: CLOS FABRIC ARCHITECTURE

## 13.1 What is a Clos Fabric?

Named after Charles Clos (1953), a Clos network is a **multi-stage non-blocking** switching architecture. In modern data centers, it's implemented as a **leaf-spine** topology.

## 13.2 Leaf-Spine vs Traditional 3-Tier

### Traditional 3-Tier

```
              ┌──────────┐
              │   Core    │    ← L3 routing
              └──┬───┬───┘
         ┌───────┘   └───────┐
    ┌────┴─────┐        ┌────┴─────┐
    │  Dist 1  │        │  Dist 2  │    ← L2/L3, STP root
    └──┬───┬───┘        └──┬───┬───┘
   ┌───┘   └───┐      ┌───┘   └───┐
┌──┴──┐  ┌──┴──┐  ┌──┴──┐  ┌──┴──┐
│Acc 1│  │Acc 2│  │Acc 3│  │Acc 4│    ← L2 access
└─────┘  └─────┘  └─────┘  └─────┘
```

**Problems:**
- STP blocks redundant links (wasted bandwidth)
- North-South optimized (bad for East-West)
- Oversubscribed uplinks
- Limited scalability

### Leaf-Spine (Clos)

```
       ┌─────────┐   ┌─────────┐   ┌─────────┐
       │ Spine 1 │   │ Spine 2 │   │ Spine 3 │
       └──┬─┬─┬──┘   └──┬─┬─┬──┘   └──┬─┬─┬──┘
          │ │ │          │ │ │          │ │ │
    ┌─────┘ │ └────┐ ┌──┘ │ └──┐ ┌────┘ │ └─────┐
    │       │      │ │    │    │ │      │       │
┌───┴──┐ ┌─┴────┐ ┌┴─┴──┐ ┌──┴─┴┐ ┌────┴─┐ ┌──┴───┐
│Leaf 1│ │Leaf 2│ │Leaf 3│ │Leaf 4│ │Leaf 5│ │Leaf 6│
└──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘
   ▲         ▲        ▲        ▲        ▲        ▲
 Servers   Servers  Servers  Servers  Servers  Servers
```

**Every leaf connects to EVERY spine.** Every spine connects to EVERY leaf.

**Benefits:**
- **Predictable latency** — exactly 2 hops between any two leaves (leaf→spine→leaf)
- **Full bandwidth** — all links active (ECMP, no STP)
- **East-West optimized** — perfect for server-to-server traffic
- **Linear scalability** — add spines for more bandwidth, add leaves for more ports
- **No oversubscription** — can be designed 1:1

## 13.3 Switch Roles in a Clos Fabric

| Role | Function | Example Hardware |
|------|----------|-----------------|
| **Leaf** | Connects to servers/hosts, performs VTEP functions | Nexus 9300-FX3 |
| **Spine** | Interconnects all leaves, acts as BGP Route Reflector | Nexus 9500, 9364C |
| **Border Leaf** | Connects fabric to external networks (WAN, Internet, other DC) | Nexus 9300 |
| **Border Gateway** | Connects two VXLAN fabrics (Multi-Site) | Nexus 9300/9500 |
| **Super Spine** | Additional tier for very large fabrics (5-stage Clos) | Nexus 9500 |

## 13.4 Traffic Flow in a Clos Fabric

**East-West Traffic (Server to Server within fabric):**
```
Server A → Leaf 1 → Spine (any) → Leaf 3 → Server B
            (2 hops, ~2-5 microseconds)
```

**North-South Traffic (Server to External/WAN):**
```
Server A → Leaf 1 → Spine (any) → Border Leaf → External Router
            (3 hops)
```

## 13.5 Why Leaf-Spine Wins for Data Centers

| Metric | 3-Tier | Leaf-Spine |
|--------|--------|-----------|
| Hop count (server-to-server) | 2-6 hops (variable) | Always 2 hops |
| Latency predictability | Variable | Deterministic |
| Bandwidth efficiency | ~50% (STP) | ~100% (ECMP) |
| Scalability | Limited (STP domain) | Linear (add spine/leaf) |
| East-West optimization | Poor | Excellent |
| Operational complexity | STP tuning, root placement | Simple (L3 everywhere) |

---

# PART 14: INTERVIEW TIPS AND COMMON QUESTIONS

## 14.1 Likely Interview Questions

### Nexus Platform Questions
1. "What are the differences between Nexus 9300, 9400, and 9500?"
2. "Explain Cloud Scale ASIC architecture and flex tiles"
3. "When would you choose Nexus over Catalyst?"
4. "What is the difference between NX-OS mode and ACI mode?"

### vPC Questions
5. "Explain vPC and all its failure scenarios"
6. "What happens during a split-brain scenario? How do you prevent it?"
7. "What are Type 1 vs Type 2 consistency checks?"
8. "Why is peer-gateway important?"

### VXLAN/EVPN Questions
9. "Walk me through the VXLAN packet format"
10. "Explain all 5 EVPN route types"
11. "What is the difference between L2 VNI and L3 VNI?"
12. "How does the anycast gateway work and why is it better than HSRP?"
13. "Explain symmetric vs asymmetric IRB"
14. "How does ARP suppression work?"
15. "What is the role of the spine in a VXLAN EVPN fabric?"

### Architecture Questions
16. "Design a VXLAN fabric for 500 servers across 2 data centers"
17. "Explain the underlay design choices (OSPF vs IS-IS, multicast vs ingress replication)"
18. "How do you handle external connectivity in a VXLAN fabric?"
19. "What is ECMP and how does it work in a leaf-spine fabric?"

### Operations Questions
20. "How do you troubleshoot a MAC not being learned in a VXLAN fabric?"
21. "What commands would you use to verify VXLAN EVPN is working?"
22. "How does NDFC automate Day-0/Day-1/Day-2 operations?"

## 14.2 Key Verification Commands

```
! VXLAN/EVPN Verification
show nve peers                          ! Show VXLAN tunnel peers
show nve vni                            ! Show VNI status
show nve interface nve1                 ! Show NVE interface status
show bgp l2vpn evpn summary            ! BGP EVPN neighbor status
show bgp l2vpn evpn                     ! All EVPN routes
show bgp l2vpn evpn route-type 2       ! MAC/IP routes only
show bgp l2vpn evpn route-type 3       ! IMET routes only
show bgp l2vpn evpn route-type 5       ! IP prefix routes
show l2route evpn mac all              ! All learned MACs via EVPN
show l2route evpn mac-ip all           ! All MAC-IP bindings
show fabric forwarding anycast-gateway ! Anycast gateway status

! Underlay Verification
show ip ospf neighbors                  ! OSPF adjacencies
show ip route                           ! Routing table
show ip route ospf                      ! OSPF routes specifically

! vPC Verification
show vpc                                ! vPC domain status
show vpc brief                          ! Quick vPC summary
show vpc consistency-parameters global  ! Type 1 checks
show vpc consistency-parameters interface port-channel 10  ! Type 2 checks
show vpc peer-keepalive                 ! Keepalive status
show vpc role                           ! Primary/Secondary role
show vpc orphan-ports                   ! Orphan port status
```

## 14.3 Quick Reference: Design Best Practices

1. **Underlay MTU:** 9216 (jumbo frames for VXLAN overhead)
2. **Underlay Routing:** OSPF with point-to-point links (or IS-IS for large scale)
3. **BGP Design:** iBGP with spines as Route Reflectors (or eBGP with unique ASN per leaf)
4. **BUM Handling:** Ingress Replication (simpler) unless very high BUM requires multicast
5. **Anycast Gateway:** Same MAC and IP on ALL leaves (no HSRP/VRRP)
6. **L3 VNI:** One per VRF, uses a transit VLAN with no user traffic
7. **Symmetric IRB:** Use for inter-subnet routing (recommended over asymmetric)
8. **vPC:** Still used at leaf pairs for server dual-homing (vPC + VXLAN)
9. **Border Leaf:** VRF-Lite (eBGP) for external connectivity
10. **Spine:** No VTEPs, no VNIs — pure routing + route reflection

---

# APPENDIX A: GLOSSARY

| Term | Definition |
|------|-----------|
| **VTEP** | VXLAN Tunnel Endpoint — device that encaps/decaps VXLAN |
| **VNI** | VXLAN Network Identifier — 24-bit segment ID |
| **NVE** | Network Virtualization Edge — VTEP interface on NX-OS |
| **L2 VNI** | Layer 2 VNI for bridging within a VLAN |
| **L3 VNI** | Layer 3 VNI for inter-VNI routing within a VRF |
| **EVPN** | Ethernet VPN — BGP-based control plane for VXLAN |
| **EVI** | EVPN Instance — maps to a VNI |
| **IRB** | Integrated Routing and Bridging — routing between VNIs |
| **RD** | Route Distinguisher — makes routes unique in BGP |
| **RT** | Route Target — controls import/export policy |
| **ESI** | Ethernet Segment Identifier — identifies multihomed segment |
| **DF** | Designated Forwarder — elected to forward BUM in multihoming |
| **IMET** | Inclusive Multicast Ethernet Tag — Type 3 route |
| **BUM** | Broadcast, Unknown Unicast, Multicast traffic |
| **ACI** | Application Centric Infrastructure — Cisco SDN fabric |
| **APIC** | Application Policy Infrastructure Controller — ACI brain |
| **NDFC** | Nexus Dashboard Fabric Controller — VXLAN automation tool |
| **NDI** | Nexus Dashboard Insights — analytics and assurance |
| **POAP** | Power-On Auto Provisioning — zero-touch deployment |
| **Clos** | Multi-stage non-blocking switching architecture |

---

# APPENDIX B: STUDY PLAN (5-DAY)

| Day | Focus | Topics |
|-----|-------|--------|
| **Day 1** | Platform + Architecture | Nexus families, 9000 sub-series, Cloud Scale ASIC, NX-OS vs IOS |
| **Day 2** | L2 Technologies | vPC deep dive (all failure scenarios), STP on NX-OS, port channels |
| **Day 3** | L3 Technologies + VXLAN Basics | OSPF/BGP on NX-OS, VRF, VXLAN fundamentals, packet format |
| **Day 4** | EVPN Deep Dive | All 5 route types, anycast gateway, L2/L3 VNI, IRB, ARP suppression |
| **Day 5** | Design + Operations | Clos architecture, NDFC, configuration examples, troubleshooting commands |

---

*Good luck with your interview! Focus on being able to explain WHY each technology exists (the problem it solves), HOW it works at the packet level, and WHEN to use it.*
