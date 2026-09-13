---
name: ccnp-enterprise-network-architecture
description: >
  Use this skill when designing, reviewing, or troubleshooting enterprise campus
  architecture on IOS-XE — the hierarchical LAN design model, high availability at the
  supervisor/route-processor level, and the campus design options built on top of them.
  Invoke when the user asks about: enterprise network architecture, campus design,
  hierarchical LAN design model, access layer, network edge, distribution layer,
  aggregation layer, core layer, backbone, building block, network block, place in the
  network, PIN, modular design, two-tier design, collapsed core, three-tier design,
  Layer 2 access layer, STP-based access, Layer 3 access layer, routed access,
  simplified campus design, SD-Access design, leaf-spine, north-south traffic,
  east-west traffic, WAN edge block, Internet edge block, data center block,
  network services edge, high availability network design, network-level HA,
  system-level HA, redundant supervisor, route processor, RP switchover, dual RP,
  SSO, Stateful Switchover, NSF, Nonstop Forwarding, GR, Graceful Restart, RFC 4724,
  NSR, Nonstop Routing, SSO/NSF with GR, SSO/NSF with NSR, GR-aware, GR Helper,
  NSF-aware, GR-unaware, SSO/NSF-capable, checkpointing, FIB preservation, BFD, UDLD,
  FHRP at the distribution layer, HSRP VRRP GLBP uplink utilization, odd/even VLAN
  load balancing, VSS, Virtual Switching System, StackWise, StackWise Virtual, SWV,
  switch clustering, MEC, Multichassis EtherChannel, cross-stack EtherChannel,
  interchassis SSO/NSF, VLAN spanning access switches, loop-free topology,
  looped topology, route summarization at the distribution layer.
---

## Purpose
Defines how a campus network is carved into repeatable modular layers and blocks, how
much redundancy each layer carries, and which supervisor-level HA mechanism keeps
traffic forwarding when a route processor fails — the design decisions that determine
whether a single failure is invisible or an outage.

## Key Concepts

**The hierarchical LAN design model**
- Divides the enterprise network into **modular layers**, each implementing a specific
  set of functions. Layers are replicated throughout the network, which gives a
  consistent deployment method and a predictable way to scale.
- It exists to avoid a **flat, fully meshed** network where every node connects to every
  other node. In a full mesh, a change tends to affect a large number of systems.
- Hierarchical design provides **fault containment**: changes are constrained to a
  subset of the network, so fewer systems are affected. Components can be placed in or
  taken out of service with little or no impact to the rest of the network.
- Benefits the exam cares about: easier troubleshooting, faster problem isolation,
  high scalability, simplified design, improved performance. It is **not** the best
  design for modern data centers — that is leaf–spine.
- The three layers are **access**, **distribution**, and **core**. Not every site needs
  all three: a small single-building campus may need only access and distribution
  (collapsed core); a multi-building campus will need all three. Modularity means each
  layer provides the same services and the same design methods regardless of how many
  layers exist at a given location.

**Access layer**
- Also called the **network edge**. Where end-user devices and endpoints attach —
  PCs, IP phones, printers, wireless APs, personal telepresence, IP cameras.
- Provides high-bandwidth connectivity over wired and wireless access technologies
  (Gigabit Ethernet, 802.11n/ac/ax). Endpoints rarely use the full capacity for long,
  but the ability to **burst** improves quality of experience and productivity.
- APs and IP phones extend the access layer **one more layer out** from the switch.
- Segmented with VLANs so different device types land in different logical networks for
  performance, management, and security reasons.
- **Access switches are not interconnected to each other.** Communication between
  endpoints on different access switches goes up through the distribution layer.
- Plays a big role in security — preventing users and endpoints from reaching services
  they are not authorized for — and is where the **QoS trust boundary** and QoS
  mechanisms are typically enforced so QoS is delivered end to end.
- For business-critical endpoints that can only connect to a single access switch, use
  access switches with **redundant supervisor engines**.

**Distribution layer**
- Primary function: **aggregate access layer switches** in a given building or campus.
- It is the **boundary between the Layer 2 domain of the access layer and the Layer 3
  domain of the core**, and that boundary provides two key LAN functions:
  - On the **Layer 2 side**: a boundary for STP, limiting propagation of Layer 2 faults.
  - On the **Layer 3 side**: a logical point to **summarize IP routing information** as
    it enters the core. Summarization shrinks routing tables (easier troubleshooting)
    and reduces protocol overhead for faster recovery from failures.
- Distribution switches are deployed **in pairs** for redundancy, interconnected to each
  other with either a Layer 2 or Layer 3 link.
- **A hierarchical LAN design building block holds a maximum of two distribution
  switches.**
- In a large campus, multiple distribution layers are often required when access
  switches sit in multiple geographically dispersed buildings — putting distribution
  switches in each building reduces the number of expensive fiber-optic runs between
  buildings.

**Core layer**
- Also called the **backbone**. Once a location grows **beyond three distribution
  layers**, add a core to optimize the design.
- Provides scalability, high availability, and fast convergence; acts as the aggregation
  point for multiple networks.
- Interconnects the end-user/endpoint campus access layer with the other **network
  blocks** — data center, private cloud, public cloud, WAN, Internet edge, network
  services.
- Reduces complexity from **N × (N − 1)** links to **N** links for N distributions.

**High availability — network level vs system level**
- *Network-level* guidelines: add redundant devices and links at different layers;
  ensure no single points of failure and a fault-tolerant design; simplify the design
  using virtual network clustering technologies; implement network monitoring systems
  that analyze capacity, faulty hardware, and security threats.
- *System-level* guidelines: use routers with redundant hardware (power supplies,
  fan trays, modular line cards, dual route processors / supervisor engines); use
  hot-swappable / OIR-capable components; enable SSO and NSF with GR or NSR; enable
  link-failure detection with **BFD** and **UDLD**; enable FHRPs (HSRP, VRRP, GLBP).

**Why RP redundancy needs help from the control plane**
- In a router with redundant RPs (supervisor engines on some platforms), one is
  **active** and one is **standby**. The active RP handles the control plane and the
  RIB, and in centralized forwarding architectures it also handles the **FIB and
  adjacency table**.
- So an active RP failure can drop routing protocol adjacencies, causing packet loss
  and instability. HA technologies let the router keep forwarding non-stop using the
  current CEF entries in the FIB instead of dropping packets while the standby RP
  rebuilds adjacencies, the routing table, and the FIB.
- Four supported combinations: **SSO and NSF**, **SSO/NSF with GR**, **SSO/NSF with
  NSR**, **SSO/NSF with NSR and GR**.

**SSO and NSF**
- **SSO** is an *internal* router redundancy feature: it checkpoints (synchronizes /
  mirrors) the **router configuration, line card operation, and Layer 2 protocol state**
  from the active RP to the standby RP.
- **NSF** is an *internal* Layer 3 **data forwarding plane** redundancy feature: it
  checkpoints and frequently updates the **FIB** from the active RP to the standby RP.
- SSO does **not** checkpoint any Layer 3 control plane information about neighbor
  routers — so existing routing adjacencies go down and must re-establish after a
  switchover. Because NSF checkpointed the FIB, the **data plane is unaffected** and
  traffic keeps flowing while that happens. Once convergence completes, the FIB is
  updated from the RIB.
- **NSF is not a configurable feature — it is enabled when SSO is enabled.** That is why
  the pair is written **SSO/NSF**.
- Downside of SSO/NSF alone: the *neighbors* only see adjacencies going down, so they
  stop forwarding traffic toward the failing router. GR, NSR, or both fix that.

**SSO/NSF with GR**
- **Graceful Restart** is standards-based (**RFC 4724**) and is the **only one of the
  three that interacts with neighbor routers**. It is deployed with SSO/NSF to protect
  the Layer 3 forwarding plane during an RP switchover.
- Neighbors must support the **GR routing protocol extensions**. Those extensions let a
  neighbor know *in advance* that the restarting router can continue forwarding packets
  even though it may briefly bring down its routing adjacency.
- **Naming trap:** *NSF (SSO/NSF)* is the term Cisco originally used for Graceful
  Restart, and it is still prevalent in Cisco documentation and the IOS XE command line.
  **Any `nsf` command or keyword you find in the docs or CLI is referring to Graceful
  Restart.** SSO/NSF is an *internal* capability; GR is an *external* one that interacts
  with neighbors.

**SSO/NSF with NSR**
- **Nonstop Routing** is an internal Cisco feature that uses **no routing protocol
  extensions**. The active RP constantly checkpoints all relevant routing **control
  plane** information to the standby RP — including routing adjacencies and **TCP
  sockets**.
- During a switchover the new active RP uses that checkpointed state to maintain the
  existing adjacencies and recalculate the routing table **without alerting the
  neighbor** that a switchover happened.
- Primary benefit over GR: a completely self-contained "in box" solution. There is no
  disruption to routing protocol adjacencies, so the neighbor does **not** need to be
  NSR-aware or GR-aware. Use it when the neighbor is **GR-unaware**.

**SSO/NSF with NSR and GR**
- NSR's downside is that constant checkpointing of routing and forwarding information to
  the standby RP **increases the workload on the router**. For scaled deployments, the
  recommendation is **GR for neighbors that are GR-aware, NSR for peers that are
  GR-unaware**.

**Architecture options**
- Six options, and because campus networks are modular, one enterprise can deploy a
  **mixture** of them: two-tier (collapsed core), three-tier, Layer 2 access (STP based),
  Layer 3 access (routed access), simplified campus design, and SD-Access. Each option
  is evaluated against business requirements — size, reliability, resiliency,
  availability, performance, security, scalability.

**Two-tier (collapsed core)**
- For smaller campuses — multiple departments across multiple floors of a building.
  A core layer may not be needed, so the core function is **collapsed into the
  distribution layer**. Cost-effective (no core layer devices) with **no sacrifice of
  most of the benefits** of the three-tier hierarchical model.
- Before choosing it, weigh future **scale, expansion, and manageability**.
- The same core/distribution pair provides LAN aggregation to the access layer *and*
  connectivity to the WAN edge, Internet edge, data center, and network services blocks.

**Three-tier**
- Separates core and distribution. Recommended when **more than two pairs of
  distribution switches** are required, which typically happens when:
  - a large enterprise campus has multiple buildings, each needing a dedicated
    distribution layer;
  - the density of WAN routers, Internet edge devices, data center servers, and network
    services grows to where it affects network performance and throughput;
  - geographic dispersion of access switches across many buildings would otherwise
    require more fiber-optic interconnects back to a single collapsed core.
- Each **building block** / **place in the network (PIN)** uses the hierarchical model
  with a pair of distribution switches connected to the core block. The **data center
  block is the exception** — it commonly uses **leaf–spine**, which suits the
  predominantly **east–west** traffic between servers. Hierarchical LAN design is more
  appropriate for **north–south** flows: endpoints reaching the WAN edge, data center,
  Internet, or network services blocks.

**Layer 2 access layer (STP based)**
- Traditional design: Layer 2 access, Layer 3 distribution. The **distribution layer is
  the Layer 3 IP gateway** for access layer hosts.
- Whenever possible, **restrict a VLAN to a single access switch** to eliminate topology
  loops — even when STP is enabled, loops are a common point of failure in LANs. That
  gives a loop-free design at the cost of flexibility, since all hosts in a VLAN are
  confined to one access switch.
- Some organizations require the same VLAN extended to multiple access switches for an
  application or service. That **looped design** makes STP block links, which reduces
  available bandwidth and slows convergence.
- The distribution pair runs an **FHRP** to give hosts a consistent MAC and gateway IP
  per VLAN. **HSRP and VRRP** are most common; their downside is that hosts can only
  send data out through the *active* FHRP router, leaving one access-to-distribution
  uplink unused. Load balancing requires **manual configuration** — making one
  distribution switch active for odd VLANs and the other active for even VLANs.
- **GLBP** gives greater uplink utilization by load balancing hosts across multiple
  uplinks — but it works **only on loop-free topologies**.
- All of these redundancy protocols need their default timers **fine-tuned** to reach
  sub-second convergence, which can impact switch CPU.

**Layer 3 access layer (routed access)**
- Layer 3 is extended all the way to the access switches. Access switches become full
  Layer 3 routed nodes doing both Layer 2 and Layer 3 switching, and the
  access-to-distribution Layer 2 trunks are replaced with **Layer 3 point-to-point
  routed links**. The **Layer 2 / Layer 3 demarcation moves from the distribution switch
  to the access switch**.
- Advantages: **no FHRP required**; **no STP required** (no Layer 2 links to block);
  **increased uplink utilization** (both uplinks usable); **easier troubleshooting**
  (common end-to-end tools like `ping` and `traceroute`); **faster convergence** using
  EIGRP or OSPF.
- Limitations: same as the loop-free Layer 2 design — **VLANs cannot span multiple
  access switches**; and it may not be the most cost-effective option, since Layer 3
  capable access switches can cost more than Layer 2 switches.

**Simplified campus design**
- Relies on **switch clustering**: **VSS** and **StackWise Virtual (SWV)** cluster two
  physical switches into a single logical switch; **StackWise** stacks two or more
  switches (platform-dependent maximum) into a single logical switch — one management
  and control plane, managed as if it were one physical switch.
- Platform dependent: **StackWise** is supported on access-layer-targeted switches;
  **VSS and SWV** on distribution- and core-targeted switches — though any of them can
  be used at any layer as necessary.
- VSS and SWV support EtherChannels spanning both physical switches: **Multichassis
  EtherChannel (MEC)**. StackWise supports **cross-stack EtherChannels**. Both let a
  device connect across all the physical switches as if connecting to a single switch.
- Advantages: simplified design (fewer boxes to manage); **no FHRP required** (the
  default gateway is on a single logical interface); **reduced STP dependence** —
  EtherChannel removes the need for STP in a Layer 2 access design, though **STP is
  still required as a failsafe** if multiple access switches are interconnected;
  increased uplink utilization; easier troubleshooting (a logical hub-and-spoke from
  distribution to access); faster convergence (all links forwarding, sub-second failover
  within the uplink bundle); **distributed VLANs** — VLANs can span multiple access
  switches **without blocking any links**; and high availability via **interchassis
  SSO/NSF** when one switch in the cluster fails.

**SD-Access design**
- The industry's first **intent-based networking** solution for the enterprise, built on
  the principles of Cisco Digital Network Architecture (DNA). It is the combination of
  the **campus fabric design** and **DNA Center (DNAC)**.
- Adds fabric capabilities via automation, providing automated **end-to-end segmentation**
  to separate user, device, and application traffic **without requiring a network
  redesign**. Adds host mobility and enhanced security on top of normal switching and
  routing. (Covered in depth in ENCOR Ch. 23, Fabric Technologies.)

## Procedure

**RP switchover with SSO/NSF only (no GR, no NSR):**
1. The active RP fails. The standby RP takes over as the new active RP.
2. The new active RP uses the SSO-learned checkpoint information to keep interfaces from
   flapping and the router and/or line cards from reloading.
3. Layer 3 control plane state was **not** checkpointed, so existing routing protocol
   adjacencies go down and begin to re-establish.
4. Because NSF checkpointed the FIB, the data plane is unaffected — the router keeps
   forwarding on the existing CEF entries while the adjacencies come back.
5. Neighbor routers, seeing the adjacencies drop, **stop forwarding traffic toward the
   restarting router** — this is the gap SSO/NSF alone leaves open.
6. After routing convergence completes, the FIB is updated with new routing or topology
   information from the RIB if necessary.

**RP switchover with SSO/NSF + GR:**
1. Before any failure, the restarting router advertises the GR routing protocol
   extensions so GR-aware neighbors know in advance it can keep forwarding through a
   switchover.
2. The active RP fails; the standby takes over exactly as above, preserving the FIB.
3. The routing adjacency may go down briefly, but **GR-aware neighbors preserve the
   routes and adjacency state** and keep forwarding traffic to the restarting router.
4. Adjacencies are re-established upon completion of the RP switchover, and normal
   routing resumes with no traffic loss at the neighbors.
5. A **GR-unaware** neighbor ignores all of this and behaves as in the SSO/NSF-only
   case — which is why GR buys nothing against a GR-unaware peer.

**RP switchover with SSO/NSF + NSR:**
1. During normal operation the active RP constantly checkpoints routing control plane
   state — adjacencies and TCP sockets — to the standby RP.
2. The active RP fails; the standby takes over.
3. The new active RP uses that checkpointed state to **maintain the existing routing
   adjacencies** and recalculate the routing table.
4. The neighbor is never alerted that a switchover occurred and needs no GR or NSR
   awareness of its own — the entire event is contained "in box."

## Reference Tables

**Hierarchical layer roles**

| Layer | Also called | Primary role | Key functions |
|---|---|---|---|
| Access | Network edge | Direct network access for endpoints and users | High-bandwidth wired/wireless attachment, VLAN segmentation, QoS trust boundary, endpoint security; access switches are **not** interconnected to each other |
| Distribution | Aggregation layer | Aggregation point for access switches; services and control boundary between access and core | STP boundary on the Layer 2 side, route summarization on the Layer 3 side, FHRP gateway for Layer 2 access, deployed in pairs (**max 2 per building block**) |
| Core | Backbone | Connects distribution layers in large environments | Scalability, high availability, fast convergence; interconnects network blocks; reduces N × (N − 1) links to N links |

**High availability technologies**

| Technology | Scope | What it checkpoints | Routing protocol extensions? | Neighbor must support it? | Separately configurable? |
|---|---|---|---|---|---|
| SSO | Internal | Router config, line card operation, Layer 2 protocol state | No | No | Yes |
| NSF | Internal | FIB / CEF entries (Layer 3 **data** plane) | No | No | **No** — enabled automatically with SSO |
| GR (RFC 4724) | **External** — interacts with neighbors | Nothing locally; signals neighbors to preserve routes and adjacency state | **Yes** | **Yes** — neighbor must be GR-aware | Yes |
| NSR | Internal | Routing **control** plane: adjacencies and TCP sockets | No | No | Yes |

**Graceful Restart router categories**

| Category | Dual RPs required? | Supports GR extensions? | Role during an RP switchover |
|---|---|---|---|
| SSO/NSF-capable | **Yes** | Yes (is also GR-aware) | The restarting router — uses SSO/NSF to preserve the FIB; adjacencies re-establish on completion of the switchover |
| GR-aware (GR Helper; misnomer "NSF-aware") | **No** | Yes | The neighbor — keeps forwarding to the restarting router by preserving its routes and adjacency state; does not need to be SSO/NSF capable |
| GR-unaware | No | **No** | Ignores GR; tears down the adjacency and stops forwarding to the restarting router — the case that calls for NSR |

**Architecture options**

| Option | Use when | Key tradeoff |
|---|---|---|
| Two-tier / collapsed core | Smaller campus; multiple departments across floors of a building; core not yet warranted | Cost-effective with most three-tier benefits retained, but future scale, expansion, and manageability must be considered up front |
| Three-tier | More than two pairs of distribution switches; multiple buildings; growing WAN/Internet/DC/services density; dispersed access switches | More devices and cost, but avoids fiber sprawl back to one collapsed core and keeps blocks independent |
| Layer 2 access (STP based) | Traditional designs; VLANs may need to span access switches | STP loops are a common failure point; one uplink idle under HSRP/VRRP unless manually odd/even load balanced |
| Layer 3 access (routed access) | Fast convergence and full uplink utilization wanted; VLANs confined to one switch is acceptable | No STP or FHRP needed, but VLANs cannot span access switches and L3-capable access switches cost more |
| Simplified campus design | VSS / SWV / StackWise available; want fewer logical boxes and distributed VLANs | Loop-free, highly available, VLANs span without blocking — STP still needed as a failsafe if access switches are interconnected |
| SD-Access | Want intent-based networking with automated end-to-end segmentation | Campus fabric + DNA Center; segmentation without a network redesign, at the cost of a new operating model |

**Network blocks reached from the core (or collapsed core)**

| Block | What lives there | Notes |
|---|---|---|
| WAN edge | WAN/cloud routers | Remote data centers, remote branches, other campuses, and cloud connectivity to AWS/Azure/GCP over **dedicated interconnections** |
| Data center / server room | Business-critical servers, server farm | Websites, corporate email, business apps, storage, big data, e-commerce, backups; commonly leaf–spine rather than hierarchical |
| Internet edge | Internet routers, firewalls, ESA/WSA | Regular Internet access, e-commerce, remote branches, remote VPN access, and cloud connectivity that does **not** require dedicated interconnections |
| Network services edge | WLC, ISE, TelePresence Manager, CUCM | Shared services consumed across the campus |

## Config Patterns
```ios-xe
! ---- System-level HA: Stateful Switchover on a dual-RP / dual-supervisor platform ----
! NSF is NOT configured separately - it is enabled automatically when SSO is enabled.
redundancy
 mode sso

! ---- Graceful Restart (Cisco's CLI calls it "nsf") ----
router ospf 1
 nsf ietf                          ! RFC 3623 IETF GR; "nsf cisco" for the Cisco variant
 nsr                               ! Nonstop Routing - internal, no neighbor awareness needed
!
router eigrp CAMPUS
 address-family ipv4 unicast autonomous-system 100
  nsf
!
router bgp 65000
 bgp graceful-restart
 neighbor 10.0.0.1 remote-as 65001
 neighbor 10.0.0.1 ha-mode sso     ! BGP NSR toward a GR-unaware peer

! ---- Link-failure detection (system-level HA guidelines) ----
udld enable aggressive
!
interface TenGigabitEthernet1/0/1
 bfd interval 50 min_rx 50 multiplier 3
 udld port aggressive
!
router ospf 1
 bfd all-interfaces

! ---- Layer 2 access layer: distribution is the L3 gateway, FHRP + STP root aligned ----
! Odd VLANs active on this distribution switch, even VLANs active on the peer.
spanning-tree vlan 10,30 root primary
spanning-tree vlan 20,40 root secondary
!
interface Vlan10
 ip address 10.1.10.2 255.255.255.0
 standby version 2
 standby 10 ip 10.1.10.1
 standby 10 priority 110
 standby 10 preempt delay minimum 180
!
interface GigabitEthernet1/0/1
 description --- to access switch ---
 switchport mode trunk
 switchport trunk allowed vlan 10,20

! ---- Layer 3 access layer (routed access): uplink is a routed P2P link, no trunk ----
interface GigabitEthernet1/0/49
 description --- routed uplink to distribution ---
 no switchport
 ip address 10.255.1.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf 1 area 10
!
interface Vlan10
 description --- local SVI, gateway now lives on the ACCESS switch ---
 ip address 10.1.10.1 255.255.255.0

! ---- Simplified campus design: StackWise Virtual pair + Multichassis EtherChannel ----
stackwise-virtual
 domain 1
!
interface range TenGigabitEthernet1/0/1-2
 stackwise-virtual link 1
!
interface TenGigabitEthernet1/0/3
 stackwise-virtual dual-active-detection
!
! MEC toward an access switch - member links land on both physical chassis
interface range TenGigabitEthernet1/0/10, TenGigabitEthernet2/0/10
 channel-group 10 mode active
!
interface Port-channel10
 switchport mode trunk
 switchport trunk allowed vlan 10,20
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Restrict a VLAN to a single access switch whenever possible | Spanning a VLAN across access switches creates a looped topology; loops are a common point of failure in LANs even when STP is enabled, and STP blocking removes bandwidth and slows convergence | An application or service genuinely requires the same Layer 2 VLAN on multiple access switches; a simplified campus design (VSS/SWV/StackWise + MEC) where VLANs can span with no blocked links | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Layer 2 Access Layer") |
| Deploy distribution switches in pairs, interconnected by a Layer 2 or Layer 3 link | A single distribution switch is a single point of failure for every access switch aggregated behind it | Very small remote sites where the cost of the second switch is a deliberate accepted risk; a VSS/SWV/StackWise cluster that already presents redundant hardware as one logical device | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Distribution Layer") |
| Summarize IP routing information at the distribution layer as it enters the core | Summarization reduces routing table size (easier troubleshooting) and protocol overhead, giving faster recovery from failures | Addressing that was never allocated contiguously per building block; designs where specific prefixes are deliberately carried for traffic engineering or visibility | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Distribution Layer") |
| Introduce a core layer once a location grows beyond three distribution layers | Without a core, interconnecting N distributions takes N × (N − 1) links; the core reduces that to N and becomes the aggregation point for the other network blocks | A small single-building campus with bounded, known scale where a two-tier collapsed core is the cost-effective answer | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Core Layer") |
| Enforce the QoS trust boundary at the access layer | The access layer is the first point of policy enforcement; marking or re-marking there is what makes end-to-end QoS (and the user's QoE) achievable | Endpoints that cannot be trusted to mark at all (trust is moved deeper); unmanaged or third-party access switches where the boundary has to sit at the distribution layer | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Access Layer") |
| Use access switches with redundant supervisor engines for business-critical endpoints that can only connect to a single access switch | A single-homed critical endpoint has no network-level redundancy, so the only remaining protection is system-level redundancy inside the switch | The endpoint is dual-homed to two access switches, or the application itself provides redundancy across sites; non-critical endpoints where the cost is not justified | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("Access Layer") |
| Pair SSO/NSF with either GR or NSR, never leave it standing alone | SSO/NSF preserves the local FIB but does not stop *neighbors* from tearing down adjacencies and halting traffic toward the restarting router | Single-RP platforms where none of this applies; a deliberately isolated device with no routing neighbors to protect | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("SSO and NSF") |
| In scaled deployments, use GR toward GR-aware neighbors and NSR toward GR-unaware peers | NSR increases the router's workload through constant checkpointing of routing and forwarding information to the standby RP; GR offloads that to a cooperating neighbor | Small deployments where enabling NSR everywhere is operationally simpler and the CPU headroom exists; mixed-vendor peers whose GR support is unreliable in practice | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 22 ("SSO/NSF with NSR and GR") |
| Align the active FHRP gateway with the STP root bridge for each VLAN at the distribution layer | If the active gateway and the root bridge sit on different distribution switches, every off-VLAN packet climbs to the wrong switch and is tromboned across the inter-distribution link | Routed-access designs with no Layer 2 between distribution switches; a VSS/SWV/StackWise cluster that presents one logical device and one logical gateway | [Campus Network for High Availability Design Guide (CVD)](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Campus/HA_campus_DG/hacampusdg.html) |

*A deviation from this table is a question for the network's operator — "is
this intentional here?" — never automatically a finding.*

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show redundancy states` | `my state = ACTIVE`, `peer state = STANDBY HOT`. Anything else (DISABLED, COLD) means SSO is not operational — and therefore NSF is not protecting the FIB |
| `show redundancy` | `Operating Redundancy Mode = sso`; confirms the configured mode actually took effect on this hardware |
| `show redundancy history` | Whether an RP switchover actually occurred, and when — the first thing to check when someone reports an unexplained blip |
| `show module` / `show platform` | Both supervisors physically present and in `Ok`/`Active`/`Standby` state; a missing or failed standby means no SSO at all |
| `show ip ospf` | `Non-Stop Forwarding enabled` / graceful restart helper status; confirms GR is actually running, not just configured |
| `show ip ospf neighbor detail` | Per-neighbor GR/helper capability — tells you whether the neighbor is GR-aware or GR-unaware |
| `show ip bgp neighbors 10.0.0.1` | `Graceful Restart Capability: advertised and received` — both directions must be present for GR to do anything |
| `show ip protocols` | Per-protocol NSF/GR status in one place, including restart interval and helper mode |
| `show spanning-tree root` | Which switch is root for each VLAN — compare against the design and against the FHRP active switch |
| `show standby brief` | Active/standby FHRP role per VLAN; misalignment with the STP root produces tromboned traffic |
| `show interfaces trunk` | Whether the access-to-distribution link is a trunk (Layer 2 access design) — its presence in a routed-access design is a design mismatch |
| `show ip interface brief` | Routed uplinks with IPs and `up/up` in a routed-access design; SVIs on the access switch rather than the distribution switch |
| `show etherchannel summary` | Port-channel in `(SU)` with member links `(P)` on **both** chassis for an MEC or cross-stack EtherChannel |
| `show stackwise-virtual` / `show stackwise-virtual link` | Domain number, SVL link members, and link state `U` — a down SVL risks a dual-active event |
| `show switch` | StackWise members in `Ready` state with the expected Active/Standby/Member roles and priorities |
| `show bfd neighbors` | Session state `Up` with the expected interval/multiplier — proves fast link-failure detection is actually running |
| `show udld neighbors` | Bidirectional neighbors detected on fiber uplinks; missing entries mean UDLD is not protecting that link |
| `show ip route summary` | Route count at the core — a large number of specifics from one building block means distribution summarization is missing or broken |

## Intent Questions
- Which tier is this device *supposed* to be — access, distribution, core, or a collapsed
  core — and does its configuration match that role (Layer 2 vs Layer 3 boundary,
  summarization, FHRP gateway, QoS trust boundary)?
- Where is the Layer 2 / Layer 3 demarcation supposed to sit: at the distribution layer
  (Layer 2 access) or at the access switch (routed access)? Every subsequent finding
  depends on that answer.
- Are VLANs supposed to span multiple access switches here — and if so, what is supposed
  to make that safe: an STP-blocked looped topology, or a VSS/SWV/StackWise cluster with
  MEC?
- What convergence target is this design supposed to hit, and which mechanism is supposed
  to deliver it — SSO/NSF with GR or NSR, tuned FHRP timers, EtherChannel member
  failover, or a fast IGP?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then
   write the one-line symptom ("should ___, isn't ___") — before running any show
   command.
1. **Hardware / Layer 1**: `show module` or `show platform` — is the standby supervisor
   actually present and healthy? Check fiber uplinks and the inter-distribution link for
   errors before blaming design.
2. **Redundancy state**: `show redundancy states` — is the peer `STANDBY HOT`? If SSO is
   not operational, NSF is not either, and no amount of GR/NSR configuration will protect
   a switchover. Then `show redundancy history` to see whether a switchover actually
   happened.
3. **Layer 2 topology**: `show spanning-tree` — is the root bridge where the design says
   it should be? Is a VLAN spanning access switches unintentionally, creating a looped
   topology with blocked uplinks?
4. **Uplink utilization**: `show interfaces` / `show etherchannel summary` — if one
   access-to-distribution uplink is idle, decide which cause it is: STP blocking, or an
   HSRP/VRRP active gateway sitting on the other distribution switch with no odd/even
   VLAN load balancing configured.
5. **FHRP alignment**: `show standby brief` — active gateway and STP root on the same
   switch for each VLAN? Misalignment trombones every off-VLAN packet across the
   inter-distribution link.
6. **Layer 3 boundary mismatch**: `show interfaces trunk` and `show ip interface brief` —
   does the access-to-distribution link match the intended design? A trunk where a routed
   link was designed (or vice versa) shows up as failed adjacencies or unexpected Layer 2
   flooding.
7. **Routing and summarization**: `show ip route` / `show ip route summary` at the core —
   are the distribution summaries present, or are specifics leaking and inflating the
   core's table?
8. **HA neighbor capability**: `show ip ospf neighbor detail` / `show ip bgp neighbors` —
   is the neighbor GR-aware? Against a GR-unaware peer, GR buys nothing and NSR is the
   answer.
9. **Fast-failure detection**: `show bfd neighbors` and `show udld neighbors` — are the
   mechanisms the design relies on for sub-second detection actually up on the links that
   matter?
10. **Configuration errors**: redundancy mode left at the platform default; `nsf`
    configured under some routing protocols but not others; FHRP preempt delay too short
    for the switch's boot time; SVL or dual-active-detection links not configured.
11. **Software/bugs**: check platform release notes for SSO/NSF/GR/NSR caveats on the
    running train, and for known StackWise Virtual dual-active or MEC issues.

## Common Pitfalls
- **NSF cannot be enabled without SSO.** It is not a separately configurable feature — it
  comes on when SSO is enabled. That is exactly why the pair is written SSO/NSF.
- **Any `nsf` keyword in IOS XE or Cisco documentation means Graceful Restart**, not the
  internal NSF feature. Cisco used "NSF" for GR first, and the CLI never caught up.
- **GR is the only one of the three that talks to neighbors.** SSO/NSF and NSR are
  internal. Enabling GR against a GR-unaware neighbor protects nothing.
- **"NSF-aware" is a misnomer** for GR-aware / GR Helper — and a **GR-aware router does
  not need dual RPs** or SSO/NSF capability of its own.
- SSO by itself does **not** preserve Layer 3 control plane state; adjacencies still drop.
  It is NSF's FIB checkpoint that keeps the data plane forwarding while they rebuild.
- **NSR increases router workload** through constant checkpointing — blanket-enabling it
  across a scaled deployment is a real CPU cost, which is why GR is preferred toward
  GR-aware peers.
- **The collapsed core is the two-tier design** — not the "simplified campus design,"
  which is the VSS/SWV/StackWise clustering option. Different questions, different answers.
- **A building block holds a maximum of two distribution switches**, and a core is
  warranted once a location exceeds three distribution layers.
- The hierarchical LAN design model is **not** the best design for modern data centers.
  Leaf–spine is, because data center traffic is predominantly **east–west**; hierarchical
  design suits **north–south** flows.
- **Access layer switches are not interconnected to each other** in the hierarchical
  model — traffic between endpoints on different access switches goes through the
  distribution layer.
- **Cloud connectivity lives in two different blocks**: dedicated interconnections to
  AWS/Azure/GCP go through the **WAN edge**, while cloud access that does not require a
  dedicated interconnect goes through the **Internet edge**.
- **GLBP fixes uplink utilization only on loop-free topologies.** On a looped Layer 2
  access design it is not an option, and HSRP/VRRP odd/even VLAN load balancing has to be
  configured by hand.
- **Routed access removes STP and FHRP but cannot span VLANs** across access switches —
  the same limitation as the loop-free Layer 2 design — and the L3-capable switches cost
  more.
- **Simplified campus design still needs STP as a failsafe** if multiple access switches
  end up interconnected, even though EtherChannel removes the day-to-day dependence.
- **Daisy-chaining is not a simplified campus design technology.** Clustering, stacking,
  VSS, and StackWise Virtual are.
- All FHRP-based redundancy needs its **default timers tuned** for sub-second
  convergence, and that tuning costs switch CPU — it is a tradeoff, not free.
