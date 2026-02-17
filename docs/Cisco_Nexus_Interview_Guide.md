# Cisco Nexus Product Family - Comprehensive Interview Preparation Guide

> **Purpose:** Deep-dive study guide for a senior-level Cisco Nexus/Data Center interview
> **Author:** Compiled from Cisco official documentation and technical resources
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
15. [Part 15: EVPN Comprehensive Guide](#part-15-evpn-comprehensive-guide)
16. [Part 16: Stacking vs VPC — Complete Comparison](#part-16-stacking-vs-vpc)

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

## 3.7 Dual-Active Detection (DAD) — Preventing Split-Brain

### What is Dual-Active Detection?

Dual-Active Detection (DAD) is a **safety mechanism** that prevents the most dangerous failure scenario in any multi-chassis redundancy system: **split-brain** (also called dual-active). In a split-brain scenario, both chassis believe they are the active/primary device and independently forward traffic, causing:
- **Duplicate IP addresses** on the network
- **MAC address flapping** across ports
- **Layer 2 loops** and broadcast storms
- **Black-holed traffic** and inconsistent forwarding
- **Data corruption** in stateful applications

DAD exists in multiple Cisco technologies: **vPC** (Nexus), **StackWise Virtual** (Catalyst 9000), and **VSS** (Catalyst 6500/4500). While each implementation differs slightly, the core concept is the same: **detect when both devices become active and immediately shut one down to restore a single control plane**.

### How DAD Works — Theory

The DAD mechanism operates through a series of steps:

1. **Normal Operation:** One device is Active/Primary, the other is Standby/Secondary. A control link (peer-link, StackWise Virtual link, or VSL) synchronizes state between them.

2. **Control Link Failure:** The connection between the two devices breaks. The Standby/Secondary device can no longer hear from the Active/Primary.

3. **Secondary Promotion:** The Standby/Secondary device assumes the Active/Primary has failed and promotes itself to Active. Now both devices are Active — this is the **dual-active** condition.

4. **DAD Detection:** A separate, independent detection mechanism (DAD link) discovers that the other device is still alive and also Active.

5. **Recovery Action:** The device that detects the dual-active condition (typically the one that was originally Standby) enters **recovery mode** — it shuts down all its user-facing interfaces, keeping only the DAD link and management interfaces alive. This restores a single Active device on the network.

6. **Restoration:** When the original control link is restored, the recovering device reloads or resynchronizes and resumes its Standby role.

### DAD Detection Methods

There are multiple ways to detect a dual-active condition. Each serves as an independent channel to verify the peer's status:

| Method | How It Works | Used In |
|--------|-------------|---------|
| **DAD Link (Dedicated L2)** | A direct Ethernet link between the two chassis carrying BPDUs or special DAD frames. If the peer's BPDUs/frames are received, dual-active is detected | StackWise Virtual, VSS |
| **Peer-Keepalive (L3)** | A Layer 3 heartbeat (UDP) over mgmt0 or a routed interface. If keepalive is received but peer-link is down, the Secondary knows the Primary is alive | vPC |
| **Enhanced PAgP (EPAGP)** | Uses PAgP messages on port channels to downstream switches. The downstream switch relays the Active ID; if two different Active IDs appear, dual-active is detected | VSS |
| **IP-based (BFD/ICMP)** | Fast heartbeat probes over a separate IP path to detect peer liveness | Some StackWise Virtual configs |
| **Fast Hello (UDLD-like)** | Rapid L2 hello frames on a dedicated link, detecting peer within milliseconds | StackWise Virtual |

### DAD in vPC (Nexus)

In vPC, the **Peer-Keepalive link** serves as the DAD mechanism. It is a Layer 3 heartbeat between the two vPC peers, completely independent of the peer-link.

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    vPC Domain                                    │
    │                                                                  │
    │   ┌──────────────┐                    ┌──────────────┐          │
    │   │   Nexus A     │    Peer-Link      │   Nexus B     │          │
    │   │  (PRIMARY)    │◄════════════════►│  (SECONDARY)  │          │
    │   │              │  (Port-Channel)    │              │          │
    │   │   10.1.1.1   │                    │   10.1.1.2   │          │
    │   └──┬──┬──┬──┬──┘                    └──┬──┬──┬──┬──┘          │
    │      │  │  │  │    Peer-Keepalive        │  │  │  │             │
    │      │  │  │  │◄ ─ ─ (DAD) ─ ─ ─ ─ ─ ─►│  │  │  │             │
    │      │  │  │  │    (mgmt0: UDP 3200)     │  │  │  │             │
    │      │  │  │  │                           │  │  │  │             │
    │      │  │  │  │   ┌─────────────────┐    │  │  │  │             │
    │      │  │  │  └───┤  vPC Member 10   ├───┘  │  │  │             │
    │      │  │  │      │  (LACP to host)  │      │  │  │             │
    │      │  │  │      └─────────────────┘      │  │  │             │
    │      │  │  └───── vPC Member 20 ───────────┘  │  │             │
    │      │  └──────── vPC Member 30 ──────────────┘  │             │
    │      └─────────── Orphan Port ───────────────────┘             │
    └─────────────────────────────────────────────────────────────────┘

    DAD Decision Matrix (vPC):
    ┌──────────────────┬───────────────┬─────────────────────────────┐
    │   Peer-Link      │  Keepalive    │  Action                     │
    ├──────────────────┼───────────────┼─────────────────────────────┤
    │   UP             │  UP           │  Normal operation           │
    │   UP             │  DOWN         │  Warning only (no impact)   │
    │   DOWN           │  UP           │  Secondary suspends vPC     │
    │                  │               │  ports (DAD triggered)      │
    │   DOWN           │  DOWN         │  SPLIT-BRAIN! Both active   │
    │                  │               │  (most dangerous scenario)  │
    └──────────────────┴───────────────┴─────────────────────────────┘
```

**vPC DAD Behavior Step-by-Step:**

1. Both peers exchange keepalive messages every 1 second (default) via mgmt0
2. If the **peer-link fails** but keepalive is **still received**:
   - Primary knows: "My peer is alive but our data path is down"
   - Secondary knows: "My peer is alive, I must suspend my vPC ports"
   - **Secondary shuts down all vPC member ports** (NOT orphan ports on itself)
   - This prevents duplicate forwarding — only the Primary serves traffic
3. If the **peer-link fails** AND keepalive **also fails**:
   - Both switches assume the other has crashed
   - Both keep all ports UP — **SPLIT-BRAIN** occurs
   - This is why peer-link and keepalive MUST use **physically independent paths**

**vPC Keepalive Configuration (DAD):**

```
vpc domain 100
  peer-keepalive destination 10.201.182.26 source 10.201.182.25 \
    vrf management                         ! Use mgmt VRF
  peer-keepalive interval 1000             ! 1 second (default)
  peer-keepalive timeout 5                 ! 5 seconds to declare dead

  ! Auto-recovery: If both switches reboot simultaneously,
  ! one will recover vPC after this delay
  auto-recovery reload-delay 240
```

### DAD in StackWise Virtual (Catalyst 9000)

In StackWise Virtual (SVL), a **dedicated DAD link** provides split-brain detection. This is a separate physical connection from the StackWise Virtual Link (SVL).

```
    ┌─────────────────────────────────────────────────────────────────┐
    │             StackWise Virtual Domain                             │
    │                                                                  │
    │   ┌──────────────┐                    ┌──────────────┐          │
    │   │  Catalyst A   │  StackWise Virtual │  Catalyst B   │          │
    │   │  (ACTIVE)     │  Link (SVL)        │  (STANDBY)    │          │
    │   │              │◄══════════════════►│              │          │
    │   │  Switch 1    │  (40G/100G)         │  Switch 2    │          │
    │   │              │                     │              │          │
    │   │              │    DAD Link          │              │          │
    │   │              │◄─ ─ ─ ─ ─ ─ ─ ─ ─►│              │          │
    │   │              │  (Dedicated L2/L3)   │              │          │
    │   └──┬──┬──┬─────┘                     └─────┬──┬──┬──┘          │
    │      │  │  │         Looks like ONE          │  │  │             │
    │      │  │  │         switch to the            │  │  │             │
    │      │  │  │         network                  │  │  │             │
    │      │  │  └──────── MEC ────────────────────┘  │  │             │
    │      │  └─────────── MEC ───────────────────────┘  │             │
    │      └──────────── Single-attached ────────────────┘             │
    └─────────────────────────────────────────────────────────────────┘

    MEC = Multi-chassis EtherChannel (like vPC member)

    DAD Detection Flow:
    ┌──────────┐     SVL Fails     ┌──────────┐
    │ Active   │ ──────X────────── │ Standby  │
    │ Switch 1 │                   │ Switch 2 │
    │          │    DAD Link       │          │
    │          │◄─────────────────►│  → NOW   │
    │          │  "Are you alive?" │   ACTIVE │
    │          │  "Yes, I am!"     │          │
    │          │                   │ RECOVERY │
    │  Stays   │                   │ MODE:    │
    │  Active  │                   │ Shuts    │
    │          │                   │ all user │
    │          │                   │ ports    │
    └──────────┘                   └──────────┘
```

**StackWise Virtual DAD Methods:**

1. **DAD Link (Recommended):** A direct physical connection between chassis. Carries special DAD messages. If both chassis detect each other as Active through this link, the one with the lower priority (or the one that was Standby) enters recovery mode.

2. **Enhanced PAgP (ePAgP):** Uses downstream switches as witnesses. The downstream switch sees PAgP messages from both chassis; if it detects two different Active chassis IDs, it reports back.

3. **Fast Hello:** Rapid hellos on a dedicated interface for sub-second detection.

**StackWise Virtual DAD Configuration (Catalyst 9000):**

```
! === StackWise Virtual Link ===
stackwise-virtual
  domain 10
  dual-active detection pagp           ! Enable ePAgP-based DAD
  dual-active detection pagp trust channel-group 1

! === Dedicated DAD Link ===
interface TenGigabitEthernet1/0/24
  dual-active fast-hello                ! Fast hello DAD on this interface

! === OR: IP-based DAD ===
dual-active recovery-reload-delay 120   ! Wait before reloading
```

### DAD in VSS (Legacy Catalyst 6500 — For Reference)

VSS (Virtual Switching System) was the predecessor to StackWise Virtual on Catalyst 6500/4500:

```
    ┌──────────────┐    VSL    ┌──────────────┐
    │  Cat 6500 A  │◄════════►│  Cat 6500 B  │
    │   (Active)   │          │  (Standby)   │
    │              │  DAD     │              │
    │              │◄── ── ──►│              │
    └──────────────┘          └──────────────┘

    DAD Methods in VSS:
    1. Enhanced PAgP (ePAgP) via downstream switches
    2. Fast Hello on dedicated interfaces
    3. BFD (Bidirectional Forwarding Detection)
```

### DAD Comparison Across Technologies

| Feature | vPC (Nexus) | StackWise Virtual (Cat 9K) | VSS (Cat 6500) |
|---------|------------|---------------------------|----------------|
| **DAD Mechanism** | Peer-Keepalive (L3 UDP) | DAD Link / ePAgP / Fast Hello | ePAgP / Fast Hello / BFD |
| **Detection Speed** | 3-5 seconds (default) | Sub-second (Fast Hello) | 1-3 seconds |
| **Recovery Action** | Secondary suspends vPC ports | Standby shuts all user ports + reloads | Standby shuts all ports + reloads |
| **Control Link** | Peer-Link (Port Channel) | SVL (40G/100G) | VSL (10G) |
| **Appears as One Switch** | No (two mgmt planes) | Yes (single mgmt plane) | Yes (single mgmt plane) |
| **Platforms** | Nexus 3000/5000/7000/9000 | Catalyst 9300/9400/9500/9600 | Catalyst 6500/4500 (EOL) |
| **Recommended DAD** | mgmt0 keepalive in VRF management | Fast Hello on dedicated link | ePAgP via downstream |

### DAD Best Practices

1. **Always configure DAD** — Never rely solely on the main control link (peer-link/SVL)
2. **Use physically independent paths** — DAD link must NOT share the same cable bundle, line card, or power domain as the control link
3. **vPC:** Use mgmt0 for keepalive (completely out-of-band from peer-link)
4. **StackWise Virtual:** Use a dedicated DAD link on a separate line card/module from SVL ports
5. **Test DAD regularly** — Simulate peer-link failures during maintenance windows to verify DAD triggers correctly
6. **Monitor DAD state** — Include DAD link status in your monitoring/alerting system

### DAD Verification Commands

```
! === vPC DAD Verification ===
show vpc peer-keepalive            ! Keepalive status, timers, last heard
show vpc role                      ! Current role (Primary/Secondary)
show vpc                           ! Overall vPC status including keepalive

! === StackWise Virtual DAD Verification ===
show stackwise-virtual dual-active-detection  ! DAD method and status
show stackwise-virtual                         ! SVL and DAD link status
show switch                                    ! Switch roles (Active/Standby)
show redundancy                                ! Redundancy state

! === VSS DAD Verification ===
show switch virtual dual-active summary
show switch virtual role
```

## 3.8 Private VLANs (PVLANs) — Micro-Segmentation at Layer 2

### What are Private VLANs?

**Private VLANs (PVLANs)** are a Layer 2 isolation technology defined in **RFC 5765** that subdivides a single **primary VLAN** into multiple **secondary VLANs** to restrict communication between hosts on the same broadcast domain. PVLANs provide **micro-segmentation** without consuming additional VLAN IDs from the global VLAN pool.

**The problem PVLANs solve:**

In a traditional VLAN, all hosts can communicate freely with each other. But what if you have 200 servers in VLAN 100 and you want:
- Servers to reach their default gateway (router)
- Servers to NOT talk to each other directly
- Some groups of servers to communicate within their group only

Without PVLANs, you'd need a separate VLAN + subnet for each server or group — consuming hundreds of VLANs and IP subnets. PVLANs solve this by keeping everyone in the **same IP subnet** while enforcing **L2 isolation**.

### PVLAN Architecture — Three Port Types

PVLANs work by defining three types of secondary VLANs within a single primary VLAN:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    PRIVATE VLAN ARCHITECTURE                             │
│                                                                          │
│    Primary VLAN 100 (10.10.10.0/24)                                     │
│    ┌─────────────────────────────────────────────────────────────────┐   │
│    │                                                                 │   │
│    │   ┌─────────────────┐    The primary VLAN is the "umbrella"    │   │
│    │   │ PROMISCUOUS PORT │    that contains all secondary VLANs.   │   │
│    │   │   (Router/GW)    │    Only promiscuous ports can talk      │   │
│    │   │   10.10.10.1     │    to ALL hosts.                        │   │
│    │   └──────┬──────────┘                                          │   │
│    │          │                                                      │   │
│    │          │ Can talk to ALL ports (promiscuous → any)            │   │
│    │          │                                                      │   │
│    │    ┌─────┴─────────────────────────────────┐                   │   │
│    │    │                                       │                   │   │
│    │    ▼                                       ▼                   │   │
│    │  ┌─────────────────────────┐  ┌──────────────────────────┐    │   │
│    │  │  COMMUNITY VLAN 201     │  │   ISOLATED VLAN 301      │    │   │
│    │  │  (Secondary)            │  │   (Secondary)             │    │   │
│    │  │                         │  │                           │    │   │
│    │  │  ┌──────┐  ┌──────┐   │  │  ┌──────┐  ┌──────┐     │    │   │
│    │  │  │Host A│  │Host B│   │  │  │Host D│  │Host E│     │    │   │
│    │  │  │ .10  │  │ .11  │   │  │  │ .30  │  │ .31  │     │    │   │
│    │  │  └──┬───┘  └──┬───┘   │  │  └──┬───┘  └──┬───┘     │    │   │
│    │  │     │◄════════►│       │  │     │    X    │          │    │   │
│    │  │     Can talk to        │  │     CANNOT talk to       │    │   │
│    │  │     each other         │  │     each other           │    │   │
│    │  │     (same community)   │  │     (isolated = alone)   │    │   │
│    │  └─────────────────────────┘  └──────────────────────────┘    │   │
│    │          │                                       │             │   │
│    │          │          ┌──────────────┐             │             │   │
│    │          │          │ COMMUNITY    │             │             │   │
│    │          │          │ VLAN 202     │             │             │   │
│    │          │          │ (Secondary)  │             │             │   │
│    │          │          │ ┌──────┐     │             │             │   │
│    │          │          │ │Host C│     │             │             │   │
│    │          │          │ │ .20  │     │             │             │   │
│    │          │          │ └──┬───┘     │             │             │   │
│    │          │          └────┼─────────┘             │             │   │
│    │          │               │                       │             │   │
│    │          │    X          │          X            │             │   │
│    │     Community 201  CANNOT talk to         Isolated 301        │   │
│    │     CANNOT talk    Community 202          CANNOT talk to      │   │
│    │     to Community   or Isolated 301        any Community       │   │
│    │     202 or                                or other Isolated   │   │
│    │     Isolated 301                                              │   │
│    │                                                                 │   │
│    │   ALL hosts share the same subnet: 10.10.10.0/24               │   │
│    │   ALL hosts use the same gateway: 10.10.10.1 (promiscuous)     │   │
│    └─────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
```

### PVLAN Port Types Explained

| Port Type | VLAN Assignment | Can Talk To | Cannot Talk To | Use Case |
|-----------|----------------|-------------|----------------|----------|
| **Promiscuous** | Primary VLAN | ALL ports (community, isolated, and other promiscuous) | — | Default gateway, router, firewall, management server |
| **Community** | Community VLAN (secondary) | Same community members + promiscuous ports | Isolated ports, other community VLANs | Group of servers that need to communicate (e.g., web farm, DB cluster) |
| **Isolated** | Isolated VLAN (secondary) | Promiscuous ports ONLY | ALL other isolated ports, ALL community ports | Individual servers that should only reach the gateway (e.g., customer VMs, untrusted hosts) |

### How PVLANs Maintain Security

PVLANs enforce security at the **switch hardware level** (ASIC/TCAM), not through software ACLs:

**1. Traffic Isolation is Enforced in Hardware**
- The switch's forwarding table prevents isolated ports from sending frames to any port except promiscuous
- Even if a host spoofs its MAC or IP, L2 forwarding rules block the frame at the ASIC level
- No ACL processing overhead — isolation is wire-speed

**2. Same Subnet, Different Access Levels**
- All hosts share 10.10.10.0/24 — no IP address waste
- But Host D (isolated) cannot ARP for Host E (isolated) — the switch drops the frame
- Host A (community 201) can ARP for Host B (community 201) — same community, allowed
- Host A (community 201) cannot ARP for Host C (community 202) — different community, blocked

**3. Defense Against Common L2 Attacks**
- **ARP spoofing:** Isolated hosts cannot see each other's ARP requests
- **MAC flooding:** Only the promiscuous port receives traffic from all secondaries
- **Lateral movement:** A compromised server in an isolated VLAN cannot reach adjacent servers
- **Broadcast storms:** Broadcasts from isolated ports only reach the promiscuous port, not other isolated ports

### How PVLANs Communicate — Traffic Flow Rules

```
PVLAN Communication Matrix:

                    TO:
           ┌──────────────┬──────────────┬──────────────┬──────────────┐
           │ Promiscuous  │ Community A  │ Community B  │  Isolated    │
FROM:      │              │              │              │              │
┌──────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│Promisc.  │     YES      │     YES      │     YES      │     YES      │
├──────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│Comm. A   │     YES      │     YES      │      NO      │      NO      │
├──────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│Comm. B   │     YES      │      NO      │     YES      │      NO      │
├──────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│Isolated  │     YES      │      NO      │      NO      │      NO      │
└──────────┴──────────────┴──────────────┴──────────────┴──────────────┘

Key rules:
✓ Promiscuous can talk to EVERYONE
✓ Community can talk to SAME community + promiscuous
✗ Community CANNOT talk to different community or isolated
✗ Isolated can ONLY talk to promiscuous (nothing else)
```

### How PVLANs Communicate with Other VLANs (Inter-VLAN Routing)

Hosts in a PVLAN reach other VLANs the same way as any host — through the **default gateway** (promiscuous port). The key is that the router/L3 switch connected to the promiscuous port performs inter-VLAN routing:

```
                     ┌─────────────────────┐
                     │    L3 Switch / Router│
                     │                     │
                     │  SVI VLAN 100       │
                     │  10.10.10.1/24      │
                     │  (Promiscuous)      │
                     │                     │
                     │  SVI VLAN 200       │
                     │  10.20.20.1/24      │
                     │  (Regular VLAN)     │
                     └───┬──────────┬──────┘
                         │          │
              PVLAN 100  │          │  Regular VLAN 200
              (Primary)  │          │
         ┌───────────────┘          └──────────────────┐
         │                                              │
    ┌────┴────────────────────────┐          ┌─────────┴──────────┐
    │   PVLAN Domain              │          │  Regular VLAN 200   │
    │                             │          │                     │
    │  ┌────────┐  ┌────────┐   │          │  ┌────────┐        │
    │  │Host A  │  │Host D  │   │          │  │Host X  │        │
    │  │Comm 201│  │Isol 301│   │          │  │10.20.  │        │
    │  │.10     │  │.30     │   │          │  │20.10   │        │
    │  └────────┘  └────────┘   │          │  └────────┘        │
    └─────────────────────────────┘          └────────────────────┘

    Traffic flow: Host D (isolated, .30) → Host X (VLAN 200, 10.20.20.10)
    1. Host D sends packet to default GW 10.10.10.1 (promiscuous port) ✓
    2. L3 switch routes from VLAN 100 SVI → VLAN 200 SVI
    3. Packet delivered to Host X on VLAN 200
    4. Reply: Host X → 10.10.10.30 → routed back to PVLAN 100 → promiscuous → Host D ✓

    Traffic flow: Host D (isolated, .30) → Host A (community 201, .10)
    1. Host D sends frame to Host A — BLOCKED at L2 (isolated cannot reach community)
    2. Host D sends to default GW 10.10.10.1 — L3 switch receives it
    3. L3 switch tries to route back to 10.10.10.10 on same subnet (VLAN 100)
    4. WITHOUT "ip local-proxy-arp": Traffic is dropped (same-subnet, no routing needed)
    5. WITH "ip local-proxy-arp": L3 switch responds to ARP on behalf of Host A,
       routes the packet hairpin-style back through promiscuous port → Host A ✓
```

**Key command for inter-host communication within PVLAN via router:**

```
interface Vlan 100
  ip address 10.10.10.1/24
  private-vlan mapping 201,301          ! Map secondary VLANs to primary SVI
  ip local-proxy-arp                     ! Enable hairpin routing within same subnet
```

> **Critical:** Without `ip local-proxy-arp` on the SVI, hosts in different secondary VLANs within the same primary VLAN **cannot** communicate even through the router, because the router sees them on the same subnet and won't route between them.

### PVLAN Configuration on NX-OS

```
! === Step 1: Enable PVLAN feature ===
feature private-vlan

! === Step 2: Create Primary VLAN ===
vlan 100
  name PVLAN-PRIMARY
  private-vlan primary

! === Step 3: Create Community VLANs (Secondary) ===
vlan 201
  name PVLAN-COMMUNITY-WEBFARM
  private-vlan community

vlan 202
  name PVLAN-COMMUNITY-DBCLUSTER
  private-vlan community

! === Step 4: Create Isolated VLAN (Secondary) ===
vlan 301
  name PVLAN-ISOLATED-CUSTOMERS
  private-vlan isolated

! === Step 5: Associate Secondary VLANs to Primary ===
vlan 100
  private-vlan association 201,202,301

! === Step 6: Configure Promiscuous Port (to Router/Gateway) ===
interface Ethernet1/1
  description To-Router-GW (Promiscuous)
  switchport mode private-vlan promiscuous
  switchport private-vlan mapping 100 201,202,301
  no shutdown

! === Step 7: Configure Community Ports ===
interface Ethernet1/10
  description WebServer-1 (Community 201)
  switchport mode private-vlan host
  switchport private-vlan host-association 100 201
  no shutdown

interface Ethernet1/11
  description WebServer-2 (Community 201)
  switchport mode private-vlan host
  switchport private-vlan host-association 100 201
  no shutdown

interface Ethernet1/20
  description DBServer-1 (Community 202)
  switchport mode private-vlan host
  switchport private-vlan host-association 100 202
  no shutdown

! === Step 8: Configure Isolated Ports ===
interface Ethernet1/30
  description Customer-VM-1 (Isolated 301)
  switchport mode private-vlan host
  switchport private-vlan host-association 100 301
  no shutdown

interface Ethernet1/31
  description Customer-VM-2 (Isolated 301)
  switchport mode private-vlan host
  switchport private-vlan host-association 100 301
  no shutdown

! === Step 9: Configure SVI for Inter-VLAN Routing ===
interface Vlan 100
  no shutdown
  ip address 10.10.10.1/24
  private-vlan mapping 201,202,301
  ip local-proxy-arp                     ! Hairpin routing for same-subnet hosts
```

### PVLAN Verification Commands

```
show vlan private-vlan                    ! Show PVLAN associations
show vlan private-vlan type              ! Show primary/community/isolated types
show interface switchport                 ! Show port PVLAN mode and associations
show interface Ethernet1/10 switchport   ! Specific port PVLAN details
show private-vlan                        ! Summary of all PVLAN domains
show mac address-table vlan 100          ! MAC table for primary VLAN
show mac address-table vlan 201          ! MAC table for community VLAN
```

### Real-World Use Cases

#### Use Case 1: Multi-Tenant Colocation / Hosting

**Scenario:** A colocation provider has 50 customer servers in the same rack, all needing internet access through a shared firewall, but no customer should see another customer's traffic.

```
    ┌───────────────────────┐
    │  Firewall / Router    │
    │  (Promiscuous Port)   │
    │  GW: 10.100.1.1/24   │
    └──────────┬────────────┘
               │
    ┌──────────┴────────────────────────────────────┐
    │              Nexus Switch                      │
    │              PVLAN Primary: 100                │
    │              Isolated VLAN: 301                │
    │                                                │
    │   Eth1/1    Eth1/2    Eth1/3    ...  Eth1/50  │
    │   Isol      Isol      Isol           Isol     │
    │    │          │          │              │       │
    └────┼──────────┼──────────┼──────────────┼──────┘
         │          │          │              │
    ┌────┴──┐  ┌───┴───┐  ┌──┴────┐   ┌────┴──┐
    │Cust A │  │Cust B │  │Cust C │   │Cust N │
    │.10    │  │.11    │  │.12    │   │.59    │
    └───────┘  └───────┘  └───────┘   └───────┘

    All customers: Same subnet 10.100.1.0/24
    All customers: Can reach firewall (promiscuous) ✓
    All customers: CANNOT reach each other ✗ (isolated)
    Benefit: One subnet, one VLAN, 50 isolated tenants
```

**Why PVLANs:** Without PVLANs, you'd need 50 VLANs and 50 subnets. With PVLANs, one primary VLAN and one isolated secondary VLAN handles all 50 customers.

#### Use Case 2: DMZ Server Isolation

**Scenario:** A company's DMZ has web servers, mail servers, and DNS servers. Web servers need to talk to each other (load-balanced cluster), but no DMZ server should reach another group.

```
    ┌────────────────────────┐
    │  Firewall              │
    │  (Promiscuous)         │
    │  DMZ GW: 172.16.1.1   │
    └──────────┬─────────────┘
               │
    ┌──────────┴─────────────────────────────────────────┐
    │   Primary VLAN 10 (DMZ: 172.16.1.0/24)             │
    │                                                     │
    │   Community 101:        Community 102:  Isolated 199│
    │   Web Servers           Mail Servers    DNS Servers │
    │   ┌─────┐ ┌─────┐     ┌─────┐         ┌─────┐    │
    │   │Web-1│ │Web-2│     │Mail │         │DNS  │    │
    │   │ .10 │ │ .11 │     │ .20 │         │ .30 │    │
    │   └──┬──┘ └──┬──┘     └─────┘         └─────┘    │
    │      │◄═════►│                                     │
    │      Can talk                                      │
    └────────────────────────────────────────────────────┘

    Web-1 ←→ Web-2: ✓ (same community 101)
    Web-1 → Mail:   ✗ (different community)
    DNS → anything: ✗ (isolated, gateway only)
    All → Firewall: ✓ (promiscuous)
```

**Security benefit:** If an attacker compromises Web-1, they can reach Web-2 (same cluster) but CANNOT pivot to the mail server or DNS server. Lateral movement is blocked at L2.

#### Use Case 3: Hotel / Conference WiFi Isolation

**Scenario:** A hotel provides WiFi to guests. Each room's AP connects to an isolated port. Guests can reach the internet gateway but not other rooms.

**Why PVLANs:** Simple, scalable guest isolation without per-room VLANs. Thousands of rooms share one subnet.

#### Use Case 4: PCI-DSS Compliance

**Scenario:** Payment card processing servers must be isolated from non-PCI systems per PCI-DSS requirements, but share the same physical switch infrastructure.

**Why PVLANs:** PCI servers in an isolated VLAN can only reach the PCI gateway (promiscuous). Non-PCI servers in a community VLAN can communicate within their group but never reach PCI servers. Auditors accept PVLAN isolation as a valid segmentation control.

### Private VLANs in a VXLAN Environment

PVLANs can be integrated with VXLAN EVPN fabrics to extend micro-segmentation across the entire fabric. This is supported on Nexus 9000 with NX-OS 9.3(3)+ and is particularly useful for large-scale multi-tenant data centers.

#### How PVLANs Work with VXLAN

In a VXLAN fabric, PVLANs are mapped to VNIs just like regular VLANs. The primary VLAN and its secondary VLANs each get their own VNI:

```
    ┌──────────────────────────────────────────────────────────────────┐
    │              PVLAN over VXLAN EVPN Fabric                        │
    │                                                                  │
    │     Leaf 1 (VTEP 1)                    Leaf 2 (VTEP 2)         │
    │   ┌──────────────────┐              ┌──────────────────┐        │
    │   │ Primary VLAN 100 │              │ Primary VLAN 100 │        │
    │   │   VNI: 10100     │              │   VNI: 10100     │        │
    │   │                  │              │                  │        │
    │   │ Community 201    │   VXLAN      │ Community 201    │        │
    │   │   VNI: 10201     │◄═══════════►│   VNI: 10201     │        │
    │   │                  │   Tunnel     │                  │        │
    │   │ Isolated 301     │              │ Isolated 301     │        │
    │   │   VNI: 10301     │              │   VNI: 10301     │        │
    │   └──┬───────┬───────┘              └───────┬──────┬───┘        │
    │      │       │                              │      │            │
    │   ┌──┴──┐ ┌──┴──┐                      ┌───┴──┐ ┌─┴────┐      │
    │   │Web-1│ │VM-A │                      │Web-3│ │VM-B  │      │
    │   │C:201│ │I:301│                      │C:201│ │I:301 │      │
    │   └─────┘ └─────┘                      └─────┘ └──────┘      │
    │                                                                  │
    │   Web-1 (Leaf 1) ←→ Web-3 (Leaf 2): ✓ Community across fabric  │
    │   VM-A (Leaf 1) → VM-B (Leaf 2):    ✗ Isolated across fabric   │
    │   VM-A → Web-1:                      ✗ Isolated → Community     │
    └──────────────────────────────────────────────────────────────────┘
```

#### PVLAN + VXLAN Configuration on NX-OS

```
! === PVLAN VNI Mapping ===
vlan 100
  name PVLAN-PRIMARY
  private-vlan primary
  vn-segment 10100                       ! Primary VLAN → VNI

vlan 201
  name PVLAN-COMMUNITY-WEB
  private-vlan community
  vn-segment 10201                       ! Community VLAN → VNI

vlan 301
  name PVLAN-ISOLATED-VMS
  private-vlan isolated
  vn-segment 10301                       ! Isolated VLAN → VNI

vlan 100
  private-vlan association 201,301

! === NVE Interface with PVLAN VNIs ===
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10100
    ingress-replication protocol bgp
  member vni 10201
    ingress-replication protocol bgp
  member vni 10301
    ingress-replication protocol bgp

! === EVPN Configuration for PVLAN VNIs ===
evpn
  vni 10100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10201 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10301 l2
    rd auto
    route-target import auto
    route-target export auto

! === SVI with Anycast Gateway + PVLAN Mapping ===
fabric forwarding anycast-gateway-mac 0000.2222.3333

interface Vlan 100
  no shutdown
  vrf member TENANT-A
  ip address 10.10.10.1/24
  private-vlan mapping 201,301
  fabric forwarding mode anycast-gateway
  ip local-proxy-arp
```

#### Benefits of PVLAN + VXLAN

| Benefit | Description |
|---------|-------------|
| **Fabric-wide isolation** | Isolated/community rules enforced across all leaf switches, not just one |
| **Scalable micro-segmentation** | Thousands of isolated VMs across the fabric, one subnet |
| **Anycast gateway + PVLAN** | Every leaf is the gateway AND enforces PVLAN rules |
| **No ACL overhead** | Isolation is hardware-enforced at each VTEP |
| **Multi-tenant security** | Each tenant in isolated VLAN, shared infrastructure |
| **Complements EVPN** | PVLAN VNIs use standard EVPN Type 2/3 routes |

#### Limitations of PVLAN in VXLAN

- **NX-OS version:** Requires NX-OS 9.3(3) or later on Nexus 9000
- **ACI mode:** PVLANs work differently in ACI (uses micro-segmentation contracts instead)
- **vPC + PVLAN:** Supported but requires consistent PVLAN configuration on both peers
- **Not all platforms:** Check the Cisco compatibility matrix for PVLAN + VXLAN support per ASIC generation
- **Complexity:** Adds another layer of configuration on top of VXLAN EVPN

### PVLAN Quick Reference

```
PVLAN Hierarchy:
    Primary VLAN (e.g., 100)
    ├── Community VLAN 201  → Hosts can talk within group + promiscuous
    ├── Community VLAN 202  → Hosts can talk within group + promiscuous
    └── Isolated VLAN 301   → Hosts can ONLY talk to promiscuous

Port Types:
    Promiscuous → Talks to all (gateway/router)
    Community   → Talks to same community + promiscuous
    Isolated    → Talks to promiscuous only

One-liner rule:
    "Isolated = alone. Community = friends. Promiscuous = talks to everyone."
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

# PART 15: EVPN COMPREHENSIVE GUIDE

## 15.1 What is EVPN?

**EVPN (Ethernet Virtual Private Network)** is a standards-based **control-plane technology** (RFC 7432) that uses **BGP** to distribute Layer 2 (MAC) and Layer 3 (IP) reachability information across a network. It provides a unified framework for delivering both L2 and L3 VPN services over various transport technologies.

**In simple terms:** EVPN is a way for network devices to share MAC address tables and IP routing information using BGP, instead of relying on traditional flooding and learning. Think of it as upgrading your network's "address book" from a phone chain (everyone calls everyone) to a centralized database (BGP distributes entries to exactly who needs them).

### Why EVPN Was Created

Before EVPN, there were several technologies for extending Layer 2 across a network, but each had significant limitations:

| Legacy Technology | Problem |
|-------------------|---------|
| **VPLS (Virtual Private LAN Service)** | Flood-and-learn, no control-plane MAC learning, poor multihoming, full-mesh required |
| **OTV (Overlay Transport Virtualization)** | Cisco proprietary, limited to DCI, no integrated L3 |
| **FabricPath** | Cisco proprietary, L2 only, no L3 integration |
| **SPB (Shortest Path Bridging)** | Limited vendor adoption, no BGP integration |
| **Traditional VXLAN (flood-and-learn)** | No control plane, multicast dependency, no ARP suppression |

EVPN solves all of these by providing:
- **Control-plane MAC learning** (no flooding)
- **Integrated L2 + L3** services in one framework
- **Active-active multihoming** with fast convergence
- **ARP/ND suppression** (reduces broadcast traffic dramatically)
- **MAC mobility** for workload migration
- **Multi-vendor interoperability** (standards-based RFC)
- **Works over multiple transports** (VXLAN, MPLS, GRE, Segment Routing)

### EVPN Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                         EVPN Architecture                            │
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │                    Control Plane (BGP)                        │   │
│   │                                                              │   │
│   │   MP-BGP with AFI 25 (L2VPN), SAFI 70 (EVPN)               │   │
│   │   Carries: MAC addresses, IP addresses, IP prefixes          │   │
│   │   Uses: Route Distinguishers (RD), Route Targets (RT)       │   │
│   │   Route Types: 1 (EAD), 2 (MAC/IP), 3 (IMET),             │   │
│   │                4 (ES), 5 (IP Prefix)                        │   │
│   └──────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                    ┌─────────┴─────────┐                            │
│                    │  Data Plane        │                            │
│                    │  (Encapsulation)   │                            │
│                    └─────────┬─────────┘                            │
│                              │                                       │
│            ┌─────────────────┼─────────────────┐                    │
│            │                 │                 │                     │
│      ┌─────┴─────┐   ┌─────┴─────┐   ┌──────┴─────┐              │
│      │   VXLAN    │   │   MPLS    │   │  Segment   │              │
│      │           │   │           │   │  Routing   │              │
│      │ UDP 4789  │   │ Label     │   │  SRv6/     │              │
│      │ VNI       │   │ Switched  │   │  SR-MPLS   │              │
│      └───────────┘   └───────────┘   └────────────┘              │
│                                                                      │
│   Same EVPN control plane works over ANY of these transports!       │
└──────────────────────────────────────────────────────────────────────┘
```

## 15.2 Types of EVPN

EVPN is not a single technology — it is a framework with multiple deployment models. The type depends on the **transport encapsulation** used in the data plane:

### Type 1: EVPN-VXLAN (Most Common Today)

**What it is:** EVPN as the control plane for VXLAN tunnels. This is what Cisco Nexus, Arista, Juniper QFX, and most modern data center switches use.

**Transport:** VXLAN (UDP port 4789)

**Where used:** Data center leaf-spine fabrics, campus fabrics

```
  ┌────────┐                                    ┌────────┐
  │ Leaf 1 │═══════ VXLAN Tunnel ══════════════│ Leaf 2 │
  │ (VTEP) │   Outer IP + UDP + VXLAN Header   │ (VTEP) │
  └───┬────┘        VNI = 10010                 └───┬────┘
      │         BGP EVPN Control Plane              │
  ┌───┴───┐                                     ┌───┴───┐
  │Host A │                                     │Host B │
  └───────┘                                     └───────┘
```

**Key characteristics:**
- VNI (24-bit) identifies the EVPN instance in the data plane
- Uses ingress replication or multicast for BUM traffic
- Symmetric IRB for inter-subnet routing
- Anycast gateway for distributed default gateway
- Defined in RFC 8365

### Type 2: EVPN-MPLS (Service Provider Networks)

**What it is:** EVPN as the control plane for MPLS-based L2VPN services. Replaces legacy VPLS.

**Transport:** MPLS Label Switching

**Where used:** Service provider metro/core networks, enterprise WAN

```
  ┌────────┐                                    ┌────────┐
  │  PE 1  │═══════ MPLS LSP ═════════════════│  PE 2  │
  │        │   MPLS Labels (Transport + VPN)   │        │
  └───┬────┘        BGP EVPN signaling          └───┬────┘
      │                                              │
  ┌───┴───┐                                     ┌───┴───┐
  │ CE 1  │    Customer Edge                    │ CE 2  │
  └───────┘                                     └───────┘
```

**Key characteristics:**
- MPLS labels identify the EVPN instance in the data plane
- Provider Edge (PE) routers perform encapsulation
- Full MPLS infrastructure required (LDP or Segment Routing)
- Supports both VPWS (point-to-point) and VPLS (multipoint)
- Superior convergence and multihoming vs legacy VPLS

### Type 3: EVPN-SR (Segment Routing)

**What it is:** EVPN combined with Segment Routing (SR-MPLS or SRv6) for the data plane. The next evolution for service providers.

**Transport:** SR-MPLS labels or SRv6 (IPv6 encapsulation)

**Where used:** Next-gen service provider networks, 5G transport, large enterprise WANs

```
  ┌────────┐                                    ┌────────┐
  │  PE 1  │═══════ SR Path ══════════════════│  PE 2  │
  │        │   SID List (Segment Routing)      │        │
  └───┬────┘        BGP EVPN + SR-TE           └───┬────┘
      │                                              │
  ┌───┴───┐                                     ┌───┴───┐
  │ CE 1  │                                     │ CE 2  │
  └───────┘                                     └───────┘
```

**Key characteristics:**
- No LDP required — uses Segment Routing for label distribution
- Traffic Engineering (SR-TE) for path optimization
- SRv6 variant uses IPv6 headers (no MPLS labels at all)
- Simplified operations compared to EVPN-MPLS
- Supported on Cisco NCS, ASR, 8000 series

### Type 4: EVPN-PBB (Provider Backbone Bridging)

**What it is:** EVPN with PBB (802.1ah MAC-in-MAC) encapsulation. Mostly used in service provider metro Ethernet.

**Transport:** PBB (MAC-in-MAC)

**Where used:** Carrier Ethernet, metro networks (niche)

**Key characteristics:**
- MAC address scalability through MAC-in-MAC encapsulation
- Separates customer MACs from backbone MACs
- Limited deployment — largely superseded by EVPN-VXLAN and EVPN-MPLS

### EVPN Type Comparison

| Feature | EVPN-VXLAN | EVPN-MPLS | EVPN-SR | EVPN-PBB |
|---------|-----------|-----------|---------|----------|
| **Primary Use** | Data Center | Service Provider | Next-gen SP | Metro Ethernet |
| **Transport** | UDP/VXLAN | MPLS Labels | SR-MPLS/SRv6 | MAC-in-MAC |
| **Encapsulation Overhead** | ~50 bytes | ~8-16 bytes | ~8-40 bytes | ~22 bytes |
| **Scalability** | 16M VNIs | Limited by labels | High (SID space) | Very high |
| **TE Support** | Limited | RSVP-TE | Native SR-TE | None |
| **Infrastructure** | IP-only underlay | Full MPLS network | SR-enabled routers | PBB-capable |
| **Multivendor** | Excellent | Good | Growing | Limited |
| **Typical Platforms** | Nexus, Arista, QFX | ASR, NCS, Juniper MX | NCS, 8000, XRd | 7600, ASR |

## 15.3 Where is EVPN Used?

EVPN is used across virtually every segment of modern networking:

### 1. Data Center Fabrics (EVPN-VXLAN)

The most common deployment. Every modern data center fabric uses EVPN-VXLAN:

```
                    ┌─────────────────────────────────┐
                    │      Data Center Fabric          │
                    │                                  │
                    │   ┌───────┐    ┌───────┐        │
                    │   │Spine 1│    │Spine 2│  (RR)  │
                    │   └─┬─┬─┬─┘    └─┬─┬─┬─┘        │
                    │     │ │ │        │ │ │           │
                    │   ┌─┴─┴─┴────────┴─┴─┴─┐        │
                    │   │     EVPN-VXLAN      │        │
                    │   │     Overlay          │        │
                    │   └─┬─────┬─────┬──────┘        │
                    │     │     │     │                │
                    │  ┌──┴──┐ ┌┴──┐ ┌┴──┐            │
                    │  │Leaf1│ │L2 │ │L3 │  (VTEPs)   │
                    │  └─────┘ └───┘ └───┘            │
                    │    │       │      │              │
                    │  Servers  VMs  Containers        │
                    └─────────────────────────────────┘
```

**Use cases:**
- Multi-tenant cloud hosting (AWS, Azure, GCP use EVPN-VXLAN internally)
- Enterprise private data centers
- Hyperconverged infrastructure (HCI)
- Container/Kubernetes networking overlay

### 2. Data Center Interconnect (DCI)

Extending L2/L3 services between multiple data centers:

```
    ┌──────────────┐                        ┌──────────────┐
    │  Data Center  │    EVPN Multi-Site     │  Data Center  │
    │     East      │◄═══════════════════►│     West      │
    │              │    (VXLAN or MPLS)     │              │
    │  EVPN Fabric  │                        │  EVPN Fabric  │
    └──────────────┘                        └──────────────┘
         │                                        │
    Border Gateway ◄──── BGP EVPN ────► Border Gateway
```

**Use cases:**
- Disaster recovery (DR) — extend VLANs between sites for VM migration
- Active-active data centers
- Workload mobility (VMware vMotion across DCs)
- Geo-redundant applications

### 3. Service Provider Networks (EVPN-MPLS / EVPN-SR)

Replacing legacy L2VPN services (VPLS, H-VPLS):

```
    ┌──────┐         ┌──────────────────────┐         ┌──────┐
    │ CE-A │─────────│    SP MPLS/SR Core    │─────────│ CE-B │
    │      │   PE-1  │                      │  PE-2   │      │
    └──────┘         │    EVPN Signaling     │         └──────┘
                     │    MPLS/SR Transport  │
                     └──────────────────────┘
```

**Use cases:**
- Enterprise L2VPN services (E-LAN, E-LINE, E-TREE)
- Mobile backhaul (5G/LTE cell tower connectivity)
- Wholesale Ethernet services
- Cloud interconnect services (connecting enterprises to cloud providers)

### 4. Campus Networks

Emerging use case — extending EVPN-VXLAN to the campus:

**Use cases:**
- Cisco SD-Access (uses LISP+VXLAN but EVPN-VXLAN is gaining traction)
- Arista Campus (uses EVPN-VXLAN natively)
- Juniper campus (EVPN-VXLAN with Mist)
- Multi-site campus with consistent policy across buildings

### 5. Multi-Cloud Networking

Connecting on-premises data centers to public clouds:

**Use cases:**
- Extending EVPN to AWS (Transit Gateway + VXLAN)
- Azure ExpressRoute with EVPN peering
- Hybrid cloud workload placement
- Consistent network policy across on-prem and cloud

## 15.4 Real-World Use Cases and Examples

### Use Case 1: Multi-Tenant Cloud Provider

**Scenario:** A cloud hosting provider operates 3 data centers serving 500+ tenants. Each tenant needs isolated L2/L3 networks that can span across data centers.

**Why EVPN:** VLAN-based approach fails at 4,094 VLANs. EVPN-VXLAN provides 16 million VNIs, VRF-based tenant isolation, and native multi-site capability.

```
    Tenant A: VRF-A, VNI 10001-10050 (50 subnets)
    Tenant B: VRF-B, VNI 20001-20030 (30 subnets)
    Tenant C: VRF-C, VNI 30001-30100 (100 subnets)
    ...
    Tenant N: VRF-N, VNI N0001-N00xx

    All tenants isolated via VRF + Route Targets
    Each tenant's traffic VXLAN-encapsulated with unique VNI
    Inter-tenant routing only when explicitly configured via RT import
```

**Real example:** Major cloud providers and large colocation facilities (Equinix, Digital Realty) use EVPN-VXLAN for tenant isolation.

### Use Case 2: Financial Trading Floor

**Scenario:** A global investment bank needs sub-millisecond latency between trading applications across two data centers (primary and DR), with instant workload mobility.

**Why EVPN:**
- Anycast gateway eliminates HSRP failover delay
- L2 extension via VXLAN Multi-Site enables live VM migration between DCs
- ECMP across all spine links maximizes available bandwidth
- ARP suppression reduces broadcast storms that could add latency jitter

**Real example:** Major banks and exchanges use EVPN-VXLAN fabrics with Nexus 9000 or Arista 7000 series for trading infrastructure.

### Use Case 3: 5G Mobile Backhaul (Service Provider)

**Scenario:** A mobile carrier needs to connect thousands of 5G cell sites to regional aggregation points, providing both L2 (Ethernet backhaul) and L3 (IP transport) services.

**Why EVPN-MPLS/SR:**
- EVPN replaces legacy VPLS for cell site connectivity
- Active-active multihoming ensures zero-downtime if one aggregation router fails
- Segment Routing provides traffic engineering for SLA guarantees
- Single control plane (BGP EVPN) for both L2 and L3 services

**Real example:** Tier-1 mobile operators deploying EVPN-SR for 5G transport on platforms like Cisco NCS 5500 and Nokia 7750-SR.

### Use Case 4: Enterprise Data Center Migration

**Scenario:** A large enterprise is migrating from a legacy 3-tier network (Catalyst 6500 + 4500 with VLANs, STP, HSRP) to a modern EVPN-VXLAN fabric.

**Migration approach:**
```
    Phase 1: Deploy leaf-spine fabric alongside legacy
    ┌──────────────┐        ┌──────────────────┐
    │  Legacy 3-Tier│        │ New EVPN-VXLAN   │
    │  (VLANs+STP)  │◄══════►│ Fabric (Nexus)   │
    │              │  Trunk  │                  │
    └──────────────┘  Link   └──────────────────┘

    Phase 2: Migrate servers rack-by-rack to new leaf switches
    Phase 3: Border leaf provides L3 connectivity between old and new
    Phase 4: Decommission legacy switches
```

**Real example:** Enterprises across healthcare, manufacturing, and retail are actively migrating from Catalyst 6500/4500 networks to Nexus 9000 EVPN-VXLAN fabrics.

### Use Case 5: Disaster Recovery with Stretched VLANs

**Scenario:** A hospital network requires that critical healthcare applications (Epic, Cerner) can failover between two campuses 15km apart with minimal downtime.

**Why EVPN:**
- EVPN Multi-Site extends L2 domains between campuses over an IP backbone
- VMs can vMotion between sites without IP address changes
- Anycast gateway ensures no gateway failover delay
- Type 5 routes advertise summary prefixes between sites for efficient routing

```
    Campus A (Primary)                    Campus B (DR)
    ┌────────────────┐   WAN / Dark     ┌────────────────┐
    │ EVPN Fabric    │   Fiber (L3)     │ EVPN Fabric    │
    │                │◄════════════════►│                │
    │ Border GW      │   EVPN Multi-    │ Border GW      │
    │ (Nexus 9300)   │   Site BGP       │ (Nexus 9300)   │
    └────────────────┘                  └────────────────┘
    Epic App Server ─── vMotion ───────► Epic App Server
    (10.1.1.50)         (same IP!)       (10.1.1.50)
```

### Use Case 6: Kubernetes/Container Networking

**Scenario:** A tech company runs Kubernetes clusters across 200+ servers and needs network segmentation between different microservice environments (dev, staging, production).

**Why EVPN:**
- Each Kubernetes namespace maps to a VRF + set of VNIs
- CNI plugins (Calico, Cilium) can integrate with hardware EVPN-VXLAN
- Network policies enforced at both the fabric level (hardware ACLs) and pod level
- Bare-metal Kubernetes nodes dual-homed via EVPN multihoming (ESI)

## 15.5 EVPN Service Types (E-LAN, E-LINE, E-TREE)

EVPN supports multiple Ethernet service types as defined by the MEF (Metro Ethernet Forum):

| Service Type | MEF Name | EVPN Implementation | Topology |
|-------------|----------|---------------------|----------|
| **E-LINE** | Ethernet Virtual Private Line | EVPN-VPWS (RFC 8214) | Point-to-Point |
| **E-LAN** | Ethernet Virtual Private LAN | EVPN (RFC 7432) | Multipoint-to-Multipoint |
| **E-TREE** | Ethernet Virtual Private Tree | EVPN E-TREE (RFC 8317) | Hub-and-Spoke (rooted multipoint) |

```
    E-LINE (Point-to-Point):
    Site A ◄════════════════► Site B
    (One-to-one connection, like a leased line)

    E-LAN (Multipoint):
    Site A ◄════►┌────────┐◄════► Site C
                 │  EVPN  │
    Site B ◄════►│  Core  │◄════► Site D
                 └────────┘
    (Any-to-any, like a LAN switch)

    E-TREE (Hub-and-Spoke):
                 ┌── Site B (Leaf — can talk to Hub only)
    Site A ◄════►│
    (Hub/Root)   ├── Site C (Leaf — can talk to Hub only)
                 │
                 └── Site D (Leaf — can talk to Hub only)
    (Hub can talk to all, Leaves cannot talk to each other)
```

---

# PART 16: STACKING VS VPC — COMPLETE COMPARISON

## 16.1 What is Stacking?

**Stacking** is a technology that combines **multiple physical switches into a single logical switch** with a **unified management plane**. All member switches share one configuration, one IP address, one CLI session, one MAC address table, and one routing table. From the network's perspective, a stack of 8 switches looks and behaves like a single, very large switch.

```
    ┌─────────────────────────────────────────────────────────────┐
    │                   Switch Stack (Single Logical Switch)       │
    │                   One IP, One Config, One Mgmt Session       │
    │                                                              │
    │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
    │   │Switch 1 │◄══►│Switch 2 │◄══►│Switch 3 │◄══►│Switch 4 │ │
    │   │(Active) │    │(Standby)│    │(Member) │    │(Member) │ │
    │   └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘ │
    │        │              │              │              │       │
    │   48 ports       48 ports       48 ports       48 ports    │
    │        = 192 total ports managed as ONE switch              │
    │                                                              │
    │   Stack Ring (Dedicated high-speed interconnect):            │
    │   SW1 ══► SW2 ══► SW3 ══► SW4 ══► SW1 (ring topology)     │
    └─────────────────────────────────────────────────────────────┘
```

### Stacking Technologies by Platform

| Technology | Platform | Max Stack | Stack Bandwidth | Stack Cable |
|-----------|----------|-----------|----------------|-------------|
| **StackWise** | Catalyst 3750 (EOL) | 9 switches | 32 Gbps | Dedicated stack cable |
| **StackWise Plus** | Catalyst 3750-X (EOL) | 9 switches | 64 Gbps | Dedicated stack cable |
| **StackWise-160** | Catalyst 3850 | 9 switches | 160 Gbps | Dedicated stack cable |
| **StackWise-480** | Catalyst 9200/9300 | 8 switches | 480 Gbps | Dedicated stack cable |
| **StackWise-1T** | Catalyst 9300L | 8 switches | 1 Tbps | Dedicated stack cable |
| **StackWise Virtual** | Catalyst 9400/9500/9600 | 2 switches | Via 10G/25G/40G/100G uplinks | Standard network cables |
| **FlexStack** | Catalyst 2960-X/XR (EOL) | 8 switches | 80 Gbps | FlexStack module |

> **Key distinction:** Traditional stacking (StackWise) uses **proprietary stack cables** on fixed switches, while **StackWise Virtual (SVL)** uses **standard Ethernet uplinks** on modular switches and only supports **2 members**.

## 16.2 What is vPC?

**vPC (Virtual Port Channel)** allows two physical Nexus switches to present **a single logical port channel** to downstream devices. Unlike stacking, the two switches maintain **independent management planes** — each has its own IP address, configuration, and CLI. They coordinate through a **peer-link** and **peer-keepalive** to forward traffic without loops.

```
    ┌──────────────┐                    ┌──────────────┐
    │   Nexus A    │    Peer-Link       │   Nexus B    │
    │  (Primary)   │◄══════════════════►│  (Secondary) │
    │              │                    │              │
    │  Own IP:     │   Keepalive (L3)   │  Own IP:     │
    │  10.1.1.1    │◄─ ─ ─ ─ ─ ─ ─ ─ ►│  10.1.1.2    │
    │              │                    │              │
    │  Own Config  │                    │  Own Config  │
    │  Own CLI     │                    │  Own CLI     │
    └──┬──┬──┬─────┘                    └─────┬──┬──┬──┘
       │  │  │                                │  │  │
       │  │  └──────── vPC 10 ───────────────┘  │  │
       │  └─────────── vPC 20 ──────────────────┘  │
       └──────────── Orphan port                    │
                                           Orphan port
```

## 16.3 What is the Difference Between Stacking and vPC?

This is the core question. The fundamental difference is:

- **Stacking = One brain, many bodies.** Multiple physical switches merge into a single logical device with a single control plane.
- **vPC = Two brains, cooperating.** Two independent switches coordinate to present a unified port channel to downstream devices, but each maintains its own control plane.

### Detailed Comparison

| Aspect | Stacking (StackWise) | vPC (Nexus) |
|--------|---------------------|-------------|
| **Management Plane** | Single (one IP, one config, one CLI) | Dual (each switch has own IP, config, CLI) |
| **Control Plane** | Single (one STP root, one routing instance) | Dual (each runs STP, routing independently) |
| **Data Plane** | Unified (stack ring) | Coordinated (peer-link for cross-switch traffic) |
| **Appears as** | One switch to the entire network | One port channel to downstream devices only |
| **Number of Devices** | 2-9 (platform dependent) | Always exactly 2 |
| **Interconnect** | Proprietary stack cables or SVL links | Standard port channel (peer-link) |
| **Configuration** | One config file for entire stack | Two separate config files (must be kept consistent) |
| **Upgrades** | Single ISSU (one switch at a time in stack) | Independent upgrades (ISSU per switch) |
| **Failure Domain** | Entire stack (shared fate for control plane bugs) | Each switch independent (bug on one doesn't crash other) |
| **L3 Routing** | One routing process, one RIB | Each switch has own routing process and RIB |
| **STP** | One STP instance for entire stack | Each switch runs STP independently |
| **Supported on** | Catalyst switches (campus) | Nexus switches (data center) |
| **Scale** | Hundreds of ports (campus access/distribution) | Thousands of ports (data center spine/leaf) |
| **Split-Brain Protection** | DAD (Dual-Active Detection) | Peer-Keepalive (split-brain prevention) |

### How They Differ — Visual

```
    STACKING:                              vPC:
    ┌─────────────────────┐              ┌────────┐    ┌────────┐
    │   ONE Logical Switch │              │Switch A│    │Switch B│
    │                     │              │(Own IP)│    │(Own IP)│
    │  ┌────┐ ┌────┐     │              │Own Conf│    │Own Conf│
    │  │ SW1│ │ SW2│     │              └───┬────┘    └───┬────┘
    │  │    │ │    │     │                  │              │
    │  │ Active│ │ Stby │     │              │   Peer-Link  │
    │  └──┬─┘ └──┬┘     │              │◄═══════════►│
    │     │      │       │                  │              │
    │  One IP: 10.1.1.1  │              │   vPC 10    │
    │  One Config         │              │◄────────────►│
    │  One STP Root       │                  │   LACP      │
    │  One Routing Table  │              ┌───┴────────────┴───┐
    └─────────────────────┘              │   Server (sees     │
                                         │   one port channel) │
    Upstream sees ONE switch.            └────────────────────┘
    Port channels terminate to
    one logical entity.                  Upstream sees TWO switches
                                         but one port channel.
```

## 16.4 Use Cases

### Where Stacking is Used

**1. Campus Access Layer (Most Common)**

The primary use case for stacking is building high-density access layer closets:

```
    IDF/Wiring Closet:
    ┌─────────────────────────────────────┐
    │  Stack of 4x Catalyst 9300-48U      │
    │  = 192 ports, one management IP     │
    │  Connected to 2x distribution       │
    │  switches via MEC (Multi-chassis    │
    │  EtherChannel)                      │
    └─────────────────────────────────────┘
```

- Reduce management overhead: 4 switches = 1 IP to monitor
- Simplified cabling: MEC port channels span stack members
- PoE for IP phones, APs, cameras across all stack members

**2. Campus Distribution Layer**

StackWise Virtual on Catalyst 9500 for distribution:

```
    ┌────────────┐     ┌────────────┐
    │  Cat 9500  │ SVL │  Cat 9500  │  = ONE logical distribution switch
    │  (Active)  │◄══►│  (Standby) │    with dual-active detection
    └──────┬─────┘     └─────┬──────┘
           │                 │
    MEC to access stacks below
```

- Two physical chassis appear as one for STP simplicity
- DAD link prevents split-brain
- ISSU support for hitless upgrades

**3. Small Branch Offices**

Stack 2-3 Catalyst 9200 switches for a branch with 100-150 ports:
- Single management point for remote branches
- Reduces need for on-site IT expertise
- Centrally managed via Catalyst Center / DNA Center

**4. Server Room / Small Data Center (Fixed switches)**

Stack Catalyst 9300 for server connectivity in small DC:
- Cost-effective alternative to Nexus for small-scale
- 480 Gbps stack bandwidth sufficient for 1G/10G server access

### Where vPC is Used

**1. Data Center Leaf Pairs (Most Common)**

vPC is the standard for leaf switch redundancy in VXLAN EVPN fabrics:

```
    ┌─────────┐    ┌─────────┐
    │ Spine 1 │    │ Spine 2 │
    └──┬──┬───┘    └───┬──┬──┘
       │  │            │  │
    ┌──┴──┴────────────┴──┴──┐
    │   vPC Leaf Pair          │
    │  ┌────────┐ ┌────────┐  │
    │  │ Leaf 1 │ │ Leaf 2 │  │
    │  │(Primary)│ │(Second)│  │
    │  └──┬──┬──┘ └──┬──┬──┘  │
    │     │  └──vPC──┘  │     │
    │     │             │     │
    └─────┼─────────────┼─────┘
          │             │
       ┌──┴──┐       ┌──┴──┐
       │ ESXi│       │ ESXi│   Servers dual-homed via LACP
       └─────┘       └─────┘
```

- Servers dual-homed with LACP across two leaf switches
- Active-active forwarding (no STP blocking)
- vPC + VXLAN EVPN is the standard modern DC design

**2. Data Center Aggregation Layer**

vPC on Nexus 9500 for aggregation:
- Connect legacy L2 switches to VXLAN fabric
- Bridge between old and new during migration
- FCoE environments requiring active-active SAN paths

**3. Network Services Integration**

vPC to connect firewalls, load balancers, and other appliances:

```
    ┌────────┐  vPC  ┌────────┐
    │ Leaf 1 │◄════►│ Leaf 2 │
    └──┬─────┘      └─────┬──┘
       │    vPC Member     │
       └──────┬────────────┘
              │
         ┌────┴────┐
         │Firewall │   (Dual-homed via LACP to vPC pair)
         │  HA     │
         └─────────┘
```

**4. Storage Network (FCoE)**

vPC for Fibre Channel over Ethernet environments:
- Active-active SAN paths
- No STP blocking on storage traffic
- Nexus 5000/7000/9000 with FCoE support

## 16.5 Pros and Cons

### Stacking: Pros and Cons

| Pros | Cons |
|------|------|
| **Single management** — One IP, one config, one CLI | **Shared failure domain** — Control plane bug crashes entire stack |
| **Simplified STP** — One STP instance, one root bridge | **Stack bandwidth limitation** — Stack ring can bottleneck |
| **Easy scaling** — Add switch, it joins the stack automatically | **Proprietary cables** — Stack cables are vendor-specific |
| **Single routing instance** — One OSPF/BGP process, one RIB | **Software upgrades** — Typically requires all members on same version |
| **No consistency checks** — One config means no mismatch issues | **Physical proximity** — Stack cables limit distance (typically ~3m) |
| **Lower cost** — No need for peer-link bandwidth | **Member failure cascading** — Stack master failure causes reconvergence for all members |
| **Port channel across members** — MEC spans all members transparently | **Limited to campus platforms** — Not available on DC-grade Nexus switches |

### vPC: Pros and Cons

| Pros | Cons |
|------|------|
| **Independent failure domains** — One switch crash doesn't affect the other | **Configuration complexity** — Two configs must be kept in sync manually |
| **Independent upgrades** — ISSU one switch while other serves traffic | **Consistency checks** — Type 1 mismatches will suspend vPC ports |
| **No distance limitation** — Peer-link can be as long as any fiber run | **Peer-link overhead** — Requires dedicated high-bandwidth links |
| **Dual management** — If one switch's mgmt fails, the other is independent | **Only 2 switches** — Cannot extend vPC to 3+ switches |
| **Independent routing** — Each switch runs own routing, own RIB | **Orphan port complexity** — Single-attached devices need special handling |
| **Data center grade** — Built for high-density, high-throughput environments | **Split-brain risk** — If both keepalive and peer-link fail |
| **Works with VXLAN** — Native integration with EVPN overlay | **Not transparent** — Downstream devices must support LACP |

## 16.6 Limitations

### Stacking Limitations

1. **Maximum stack size:** 2-9 members (varies by platform) — cannot build large-scale fabrics with stacking alone
2. **Stack bandwidth:** Even StackWise-480 (480 Gbps) can bottleneck under heavy east-west traffic across members
3. **Distance:** Traditional stack cables limited to ~3 meters (StackWise Virtual allows longer via standard links)
4. **Single software version:** All members must run the same NX-OS/IOS-XE version
5. **Blast radius:** A software bug or crash on the active member takes down the entire stack
6. **No multi-tenancy:** No VRF-heavy designs or VXLAN overlay (campus-focused)
7. **Stack renumbering:** Adding/removing members can cause member number changes and port ID shifts
8. **Priority election:** Stack master election can be unpredictable if priorities aren't set correctly
9. **Not for spine-leaf:** Stacking is for access/distribution layers, not suitable for modern DC spine-leaf architectures

### vPC Limitations

1. **Only 2 peers:** vPC domain is always exactly 2 switches — no 3-way vPC
2. **Configuration sync:** No automatic config sync (operator must ensure matching configs on both peers)
3. **Peer-link bandwidth:** Peer-link must be large enough for orphan port traffic and CFS messages
4. **L3 over vPC:** Some limitations with L3 routing over vPC peer-link (e.g., routing adjacency on VLAN carried over peer-link)
5. **Orphan port traffic:** If peer-link fails, orphan ports on secondary become unreachable
6. **vPC+ (ACI):** vPC in ACI mode has additional restrictions compared to NX-OS mode
7. **Supported features:** Some NX-OS features are not supported with vPC (check release notes for compatibility matrix)
8. **No dynamic negotiation:** vPC domain ID, role priority, and keepalive must be manually configured and matched
9. **Peer-link is mandatory:** Even if no traffic needs to cross between peers, the peer-link must exist for control messages

## 16.7 Which Devices Use Stacking? Which Use vPC?

### Stacking Devices

| Platform | Stacking Technology | Typical Role |
|----------|-------------------|-------------|
| **Catalyst 9200** | StackWise-480 (up to 8) | Campus access (SMB/branch) |
| **Catalyst 9300** | StackWise-480/1T (up to 8) | Campus access (enterprise) |
| **Catalyst 9400** | StackWise Virtual (2 only) | Campus distribution/core (modular) |
| **Catalyst 9500** | StackWise Virtual (2 only) | Campus distribution/core (fixed) |
| **Catalyst 9600** | StackWise Virtual (2 only) | Campus core (modular, high perf) |
| **Catalyst 3850** (EOL) | StackWise-160 (up to 9) | Campus access (legacy) |
| **Catalyst 3650** (EOL) | StackWise-160 (up to 9) | Campus access (legacy) |
| **Catalyst 2960-X** (EOL) | FlexStack (up to 8) | Campus access (legacy, L2) |

### vPC Devices

| Platform | vPC Support | Typical Role |
|----------|------------|-------------|
| **Nexus 9300** (all ASICs) | Yes (primary use) | Data center leaf |
| **Nexus 9500** | Yes | Data center spine/aggregation |
| **Nexus 9400** | Yes | Data center aggregation |
| **Nexus 7000/7700** (EOL) | Yes | Legacy data center core |
| **Nexus 5000/5500/5600** (EOL) | Yes | Legacy access/aggregation |
| **Nexus 3000** | Yes (some models) | Low-latency ToR |

### Quick Decision Matrix

```
    "Should I use Stacking or vPC?"

    ┌──────────────────────────┐
    │ Is it a DATA CENTER?     │
    │ (Nexus, high density,    │
    │  10G-400G, VXLAN/EVPN)   │
    └──────────┬───────────────┘
               │
          ┌────┴────┐
          │  YES    │ ──────► Use vPC (Nexus switches)
          └─────────┘

          ┌─────────┐
          │   NO    │
          └────┬────┘
               │
    ┌──────────┴──────────────┐
    │ Is it a CAMPUS network?  │
    │ (Catalyst, 1G/10G/25G,   │
    │  PoE, access/distrib)    │
    └──────────┬───────────────┘
               │
          ┌────┴────┐
          │  YES    │ ──────► Use Stacking (Catalyst switches)
          └─────────┘
               │
          ┌────┴────┐
          │ BOTH?   │ ──────► Campus: Stacking
          └─────────┘         Data Center: vPC
                              They can coexist in the same network!
```

### How They Coexist (Real-World Architecture)

In a typical enterprise, both technologies are used simultaneously in different parts of the network:

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    ENTERPRISE NETWORK                           │
    │                                                                 │
    │   DATA CENTER                    │    CAMPUS                    │
    │   (vPC on Nexus)                 │    (Stacking on Catalyst)   │
    │                                  │                              │
    │   ┌────────┐    ┌────────┐      │    ┌─────────────────────┐  │
    │   │Spine 1 │    │Spine 2 │      │    │  Distribution       │  │
    │   └──┬──┬──┘    └──┬──┬──┘      │    │  Cat 9500 SVL Pair  │  │
    │      │  │          │  │         │    └──────┬──────────────┘  │
    │   ┌──┴──┴──────────┴──┴──┐      │           │                 │
    │   │     vPC Leaf Pairs    │      │    ┌──────┴──────────────┐  │
    │   │  ┌──────┐ ┌──────┐  │      │    │  Access Stack       │  │
    │   │  │Leaf 1│ │Leaf 2│  │      │    │  Cat 9300 x 4       │  │
    │   │  │(vPC) │ │(vPC) │  │      │    │  192 ports, 1 IP    │  │
    │   │  └──────┘ └──────┘  │      │    │  PoE for phones/APs │  │
    │   │                      │      │    └─────────────────────┘  │
    │   └──────────────────────┘      │                              │
    │      │                           │         │                    │
    │   Servers, VMs, Storage          │    Phones, PCs, APs, IoT    │
    │   (10G-100G, VXLAN/EVPN)        │    (1G PoE, SD-Access)      │
    └─────────────────────────────────┴──────────────────────────────┘
```

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
| **DAD** | Dual-Active Detection — prevents split-brain in multi-chassis systems |
| **SVL** | StackWise Virtual Link — interconnect between two Catalyst chassis forming one logical switch |
| **MEC** | Multi-chassis EtherChannel — port channel spanning stack members |
| **ePAgP** | Enhanced PAgP — DAD method using downstream switches as witnesses |
| **VSS** | Virtual Switching System — legacy Catalyst 6500 chassis virtualization (EOL) |
| **VPLS** | Virtual Private LAN Service — legacy L2VPN replaced by EVPN |
| **VPWS** | Virtual Private Wire Service — point-to-point L2VPN (EVPN E-LINE) |
| **E-LAN** | Ethernet Virtual Private LAN — multipoint-to-multipoint EVPN service |
| **E-LINE** | Ethernet Virtual Private Line — point-to-point EVPN service |
| **E-TREE** | Ethernet Virtual Private Tree — hub-and-spoke EVPN service |
| **SR** | Segment Routing — modern MPLS alternative using SIDs for path programming |
| **SRv6** | Segment Routing over IPv6 — SR using IPv6 encapsulation instead of MPLS labels |
| **PBB** | Provider Backbone Bridging — IEEE 802.1ah MAC-in-MAC encapsulation |
| **DCI** | Data Center Interconnect — extending L2/L3 between data centers |
| **MEF** | Metro Ethernet Forum — standards body defining Ethernet services |
| **PVLAN** | Private VLAN — L2 micro-segmentation subdividing a primary VLAN into secondary VLANs |
| **Primary VLAN** | The parent VLAN in a PVLAN domain that contains all secondary VLANs |
| **Community VLAN** | Secondary VLAN where hosts can talk to same community + promiscuous ports |
| **Isolated VLAN** | Secondary VLAN where hosts can ONLY talk to promiscuous ports |
| **Promiscuous Port** | PVLAN port that can communicate with all secondary VLANs (gateway/router) |

---

# APPENDIX B: STUDY PLAN (5-DAY)

| Day | Focus | Topics |
|-----|-------|--------|
| **Day 1** | Platform + Architecture | Nexus families, 9000 sub-series, Cloud Scale ASIC, NX-OS vs IOS |
| **Day 2** | L2 Technologies | vPC deep dive (all failure scenarios), DAD, STP on NX-OS, port channels |
| **Day 3** | L3 Technologies + VXLAN Basics | OSPF/BGP on NX-OS, VRF, VXLAN fundamentals, packet format |
| **Day 4** | EVPN Deep Dive | All 5 route types, EVPN types (VXLAN/MPLS/SR), anycast gateway, L2/L3 VNI, IRB, ARP suppression |
| **Day 5** | Design + Operations | Clos architecture, NDFC, Stacking vs vPC, configuration examples, troubleshooting commands |
| **Day 6** | Advanced Topics + Review | EVPN real-world use cases (DCI, 5G, multi-tenant), Stacking vs vPC deep dive, DAD scenarios, interview Q&A practice |

---

*Good luck with your interview! Focus on being able to explain WHY each technology exists (the problem it solves), HOW it works at the packet level, and WHEN to use it.*
