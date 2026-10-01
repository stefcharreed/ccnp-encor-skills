---
name: ccnp-ip-routing-essentials
description: >
  Use this skill when troubleshooting or configuring ip-routing-essentials on IOS-XE.
  Invoke when the user asks about: RIB, FIB, CEF, Cisco Express Forwarding, FIB vs CEF,
  adjacency table, glean adjacency, punt adjacency, null adjacency, process switching,
  fast switching, route cache, dCEF, show ip cef, show adjacency, administrative distance, prefix
  length, ECMP, unequal-cost load balancing, CEF load balancing, CEF
  load sharing, per-packet, per-destination, ip load-sharing, out-of-order
  packets, static route, floating static
  route, null route, policy-based routing, VRF, ARP resolution, incomplete
  ARP entry, proxy ARP, local vs remote forwarding decision, ICMP
  destination host unreachable.
---

## Purpose
IP routing essentials cover how a router decides which path wins (prefix length, administrative distance, metric), how static routes are built and protected from loops, how PBR overrides destination-based forwarding, and how VRF segments a single router into isolated virtual routers.

## Key Concepts
- A **subnet/prefix** is a location on the network; a **path** is one of possibly several series of links between two prefixes; a **route** is the specific path a routing source (static or dynamic protocol) advertises to reach a destination — these terms are often used loosely but mean different things.
- An **autonomous system (AS)** is a routing domain under common administration. IGPs (RIPv2, EIGRP, OSPF, IS-IS) route within an AS; EGPs (BGP) route between autonomous systems — BGP can also run within an AS as iBGP (vs. eBGP between ASes).
- **Distance vector** protocols (RIPv2) advertise routes as a distance (metric, e.g. hop count) and vector (next-hop direction), trusting neighbor-advertised information without a full network map — analogous to trusting a road sign. They don't account for link speed, only distance.
- **Enhanced distance vector** (EIGRP, via DUAL) is a hybrid: distance-vector-style advertisement but with link-state-like neighbor relationships (hellos), event-triggered updates instead of periodic full updates, and metrics based on bandwidth/delay/reliability/load instead of just hop count — can pick a longer-hop but higher-bandwidth path over a short low-bandwidth one.
- **Link-state** protocols (OSPF, IS-IS) flood unmodified link-state info to every router, building an identical synchronized map (LSDB) everywhere, then each router independently runs Dijkstra SPF — analogous to a GPS with a full map. Costs more CPU/memory than distance vector but avoids loops and makes better decisions; supports extensions like OSPF opaque LSAs / IS-IS TLVs for MPLS-TE.
- **Path vector** (BGP) evaluates path attributes (AS_Path, MED, origin, next hop, local preference, atomic aggregate, aggregator) rather than a simple distance metric, and guarantees loop-freedom by rejecting any advertisement that already contains the local AS in its AS_Path.
- Path selection happens in this priority order: **prefix length** (longest match always wins regardless of source) → **administrative distance** (lower AD wins when multiple sources offer the same prefix length) → **metric** (lower wins when AD ties, e.g. two sources from the same protocol).
- The RIB only ever holds the *single best* route a routing process submits per prefix; if a lower-AD route is later removed, the RIB asks the other process(es) that lost the AD comparison to resubmit their route — meaning the lowest-AD route in absolute terms isn't always what gets submitted (e.g. BGP may submit an iBGP path at AD 200 instead of an available eBGP path at AD 20, because BGP's own best-path algorithm decided it first).
- **FIB vs CEF: CEF is the switching method, and the FIB is one of its two tables.** Cisco: "The
  two main components of Cisco Express Forwarding operation are the **forwarding information
  base (FIB)** and the **adjacency tables**." So the question isn't FIB *or* CEF. The FIB is
  *part of* CEF.
  - **FIB** = reachability. "The FIB contains the prefixes from the IP routing table structured in
    a way that is optimized for forwarding." It is "conceptually similar to a routing table" and
    "maintains a **mirror image** of the forwarding information in an IP routing table." There
    is a **one-to-one correlation** between FIB entries and RIB entries, and RIB changes are
    reflected in the FIB.
  - **Adjacency table** = the Layer 2 rewrite. "A node is said to be adjacent to another node if
    the node can be reached with a single hop across a link layer." The table stores the
    **outbound interface and MAC header rewrite** for each adjacent node, and "maintain[s] Layer 2
    next-hop addresses for all FIB entries." It is populated dynamically, e.g. by **ARP**. Each
    time an adjacency is created, the link-layer header is **pre-computed and stored**.
  - **Why split them:** the FIB doesn't store the MAC rewrite. It *points to* the adjacency
    entry. So both tables are **pre-built from the RIB and ARP before any packet arrives**: no
    packet is process-switched to build an entry, an ARP change doesn't invalidate the FIB, and
    recursive routes resolve by pointing straight at the adjacency.
  - **Where CEF sits among the switching paths:** **process switching** = an IOS process
    forwards each packet from the RIB + ARP cache (slowest, every packet hits the CPU).
    **Fast switching** = the *first* packet is process-switched to build a **route cache**
    entry, then later packets use the cache. The cache is demand-built, ages out (1/20th
    invalidated randomly every minute), and must be partly invalidated whenever ARP changes.
    **CEF** = the FIB contains *all* known routes, so there is "no route cache maintenance"
    and nothing is built on demand.
  - **Central vs distributed CEF:** in central CEF, the FIB and adjacency tables live on the
    **RP** and the RP forwards. In **dCEF**, "line cards maintain **identical copies** of the FIB
    and adjacency tables" (kept in sync over IPC) and forward without the RP. That copy is
    what lets NSF keep forwarding through an RP switchover (see `enterprise-network-architecture`).
  - *Sources: [CEF Overview, IP Switching CEF Configuration Guide (IOS XE 16)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipswitch_cef/configuration/xe-16/isw-cef-xe-16-book/isw-cef-overview.html);
    [How to Choose the Best Router Switching Path for Your Network](https://www.cisco.com/c/en/us/support/docs/ip/express-forwarding-cef/13706-20.html).*
- **ECMP** (equal-cost multipathing): when a protocol has multiple equal-metric paths and supports it, all are installed and traffic load-shares evenly. **Unequal-cost load balancing**: EIGRP-only, not default-enabled, installs multiple different-metric paths and ratios traffic proportional to each path's metric (lower metric gets more traffic share).
- **⚠ CEF LOAD BALANCING — PER-DESTINATION vs PER-PACKET (high-yield exam trap).** Once
  ECMP installs several paths, **CEF** decides how packets are spread across them. The
  choice is set on the **outbound interface** (`ip load-sharing ...`).
  - **Per-destination is the DEFAULT.** CEF hashes source + destination, so **every packet of
    a given flow takes the same path**. Packets stay in order, so this is **safe for VoIP**.
    Distribution is **statistical**: it only evens out across many flows, and one heavy flow
    can load a single link.
  - **Per-packet** round-robins **each packet** across the links, ignoring sessions and
    sequence. Links load evenly, but packets of one flow take different paths and can
    **arrive at the destination out of order**. For **VoIP** that means choppy, degraded
    calls, and TCP sees duplicate ACKs and retransmits.
  - **Where the reordering happens:** out of order **at the destination**, never at the
    source. The balancing decision is made on the router after the packets have left the
    sender and are in transit.
  - **Exam answer pattern:** "per-packet + VoIP" → *quality could suffer because packets can
    **arrive at their destination** out of order*. The distractors are "sent from the source
    out of order" (wrong place), "improves because balanced over multiple links" (reordering
    outweighs it), and "distributed statistically" (that's per-destination, not per-packet).
- Static route types: **Directly attached** (`ip route <net> <mask> <interface>`) — only valid on point-to-point interfaces without ARP (e.g. serial); using it on an Ethernet/ARP-capable interface forces ARP for every destination matching the route and can cause instability. **Recursive** (`ip route <net> <mask> <next-hop-ip>`) — requires a second RIB lookup to resolve the next-hop IP to an interface; cannot resolve via a default route (0.0.0.0/0) entry. **Fully specified** (`ip route <net> <mask> <interface> <next-hop-ip>`) — both interface and next-hop IP given, avoids the recursive lookup and ARP issues, and the route is pulled from the RIB if the named interface goes down.
- **Floating static routes** use a deliberately higher AD than the primary route so they only get installed as backup when the primary is withdrawn — common pattern for backup links behind a preferred dynamic-routing or lower-AD static path.
- **Null route** (`ip route <summary-net> <mask> Null0`) drops any traffic matching a summarized range that doesn't match a more specific real route — prevents routing loops on a router that's advertising (or receiving) a summarized block it doesn't fully use, without needing an ACL.
- IPv6 static routing mirrors IPv4: requires `ipv6 unicast-routing` enabled globally, then `ipv6 route <prefix>/<length> {interface-id | [interface-id] next-hop-ip}` — if the next hop is a link-local address, the route must be fully specified (interface + next-hop) since link-locals aren't globally routable on their own.
- **Policy-based routing (PBR)** overrides destination-based forwarding using packet characteristics (protocol, source/destination IP, etc.) to set a different next hop — verifies next-hop reachability in the RIB before using it, supports a prioritized list of fallback next hops, and silently fails closed (normal RIB forwarding) if none of the specified next hops are reachable. PBR does not modify the RIB itself — `show ip route` looks unchanged even with active PBR policies, which complicates troubleshooting since the conditional next hop isn't visible there.
- **Forwarding decides local vs. remote BEFORE any ARP happens.** The sender ANDs the destination IP against its own mask: same subnet → deliver locally, ARP for the destination itself; different subnet → ARP for the **default gateway** instead, and the destination IP never gets ARPed for at all. Getting this order right explains most "why isn't it ARPing for that?" confusion.
- **When ARP resolution fails on-subnet**, the sequence is: ARP cache miss → the IP packet is **queued** (most stacks hold only *one* packet per pending entry; the rest are dropped) → ARP Request broadcast (`FF:FF:FF:FF:FF:FF`, opcode 1, target HW `00:00:00:00:00:00`) → switch floods it VLAN-wide and learns only the *sender's* MAC → no reply → retries (Linux ~3× at 1s, IOS every 2s) → entry marked **INCOMPLETE** → queued packet dropped.
- **The "Destination Host Unreachable" you see for a same-subnet failure is generated locally by your own IP stack** — nothing ever left the wire, and no router was involved. For a **different-subnet** failure the router does the ARPing and returns a genuine **ICMP Type 3 Code 1**. Same message, completely different origin — a classic exam distinction.
- **Proxy ARP** (`ip proxy-arp`, historically **on by default** on Cisco router interfaces) makes a router answer an ARP for an IP it has a route to, supplying its own MAC. It silently masks host misconfiguration — a wrong subnet mask still "works" until proxy ARP is disabled elsewhere. **The tell: `arp -a` on the host shows several different IPs sharing one MAC.** It's also an MITM vector, so `no ip proxy-arp` is standard hardening.
- A **directly attached static route on an Ethernet interface** forces an ARP for *every* destination matching that route (see Common Pitfalls) — this is the same ARP machinery, which is why fully specified statics are preferred on multi-access links.
- **Two generations of VRF syntax, both exam-fair game.** Legacy `ip vrf <name>` + `ip vrf forwarding <name>` is **VRF-Lite, IPv4 only** — no `address-family` sub-mode exists under it. The newer `vrf definition <name>` + `vrf forwarding <name>` is multi-address-family (hence the required `address-family ipv4`/`ipv6` step). Same concept, and the interface behavior is identical: applying either form strips the interface's IP.
- **A static route is VRF-scoped by the `vrf` keyword, in global config:** `ip route vrf <name> <net> <mask> <next-hop>`. Without it the route lands in the global table and the VRF never sees it. Same pattern for the other VRF-aware commands (`ping vrf`, `traceroute vrf`, `show ip route vrf`).
- **An interface belongs to exactly one VRF** — including virtual ones. A tunnel interface takes `vrf forwarding` like any physical port.
- **VRF (Virtual Routing and Forwarding)** creates isolated logical routers on one physical box — separate routing/forwarding tables per VRF, allowing overlapping IP address ranges across VRFs with no conflict. All interfaces default to the **global VRF** (the standard routing table) until explicitly assigned elsewhere. Conceptually similar to VLANs on a switch, but VRF segmentation operates at Layer 3 with full per-VRF dynamic routing rather than 802.1Q tagging at Layer 2.

## Procedure
BGP path vector loop avoidance (illustrative 4-AS example: AS1–AS2–AS4–AS3, prefix 10.1.1.0/24 originated in AS1):
1. R1 (AS 1) advertises 10.1.1.0/24 to R2 (AS 2), adding AS 1 to the AS_Path.
2. R2 advertises the prefix to R4 (AS 4), adding AS 2 to the AS_Path (now "2 1").
3. R4 advertises the prefix to R3 (AS 3), adding AS 4 to the AS_Path (now "4 2 1").
4. R3 advertises the prefix back toward R1 and R2, adding AS 3 to the AS_Path (now "3 4 2 1").
5. R1 receives this advertisement, detects its own AS (1) already present in the AS_Path, and rejects it as a loop; R2 does the same upon detecting AS 2 in the path.

ARP resolution failure, same subnet (Host A 10.1.1.10/24 → 10.1.1.99, which doesn't exist):
1. **L3 decision:** 10.1.1.99 ANDs into 10.1.1.0/24 — same subnet, so local delivery. No gateway involved.
2. **ARP cache lookup** — miss.
3. **Packet queued**, not sent — L2 can't frame it without a destination MAC. One packet held; further packets to that IP are dropped.
4. **ARP Request broadcast** — Ethernet dst `FF:FF:FF:FF:FF:FF`, src A's MAC, EtherType `0x0806`, opcode 1, target proto 10.1.1.99, target HW all zeros.
5. **Switch floods** it out every port in the VLAN except ingress, and learns **A's** MAC on the ingress port. It learns nothing about .99 — switches only learn from frames that arrive.
6. **No reply** — nothing owns .99.
7. **Retries**, then the entry is marked **INCOMPLETE** (`Internet 10.1.1.99 - Incomplete ARPA`).
8. **Queued packet dropped**; the local stack reports Destination Host Unreachable.

Creating a VRF and assigning it to an interface:
1. Create the VRF routing table: `vrf definition <vrf-name>`.
2. Initialize the address family: `address-family {ipv4 | ipv6}` (configure both if dual-stack).
3. Enter the target interface's configuration mode: `interface <interface-id>`.
4. Associate the interface to the VRF: `vrf forwarding <vrf-name>`.
5. Configure the interface's IP address(es) — `ip address <ip-address> <subnet-mask> [secondary]` and/or `ipv6 address <ipv6-address>/<prefix-length>`. (Note: the IP address must be (re)configured after `vrf forwarding`, since assigning a VRF to an interface clears any previously configured IP address on it.)

## Reference Tables
Default administrative distances by route source:

| Route Origin | Default Administrative Distance |
|---|---|
| Directly connected interface | 0 |
| Static route (incl. directly attached static route) | 1 |
| EIGRP summary route | 5 |
| External BGP (eBGP) route | 20 |
| EIGRP (internal) route | 90 |
| OSPF route | 110 |
| IS-IS route | 115 |
| RIPv2 route | 120 |
| EIGRP (external) route | 170 |
| Internal BGP (iBGP) route | 200 |

**RIB → FIB → adjacency: who builds what** *(Cisco CEF Overview)*

| Table | Built from | Holds | Show command |
|---|---|---|---|
| RIB (routing table) | Connected, static, routing protocols | Best route per prefix | `show ip route` |
| FIB (CEF table) | The RIB, one-to-one | Prefix → next hop, optimized for lookup | `show ip cef` |
| Adjacency table | ARP (or routing protocol neighbors / manual config) | Next hop → outbound interface + pre-built MAC rewrite | `show adjacency [detail]` |

**Switching paths compared** *(Cisco "How to Choose the Best Router Switching Path")*

| Path | How a forwarding entry is built | Weakness |
|---|---|---|
| Process switching | Never cached. Every packet is looked up in the RIB + ARP cache by an IOS process | Every packet costs CPU |
| Fast switching | First packet process-switched, result stored in a **route cache** (binary tree) | Cache ages out, is invalidated by ARP changes, can't resolve recursion in-cache |
| Optimum switching | Same as fast, but a 256-way mtree (at most 4 lookups) | Still cache aging and invalidation |
| **CEF** | **Pre-built** FIB (256-way trie) + separate adjacency table, from the RIB and ARP | None of the above. No packet is process-switched to build an entry |

**Special adjacency types** *(Cisco CEF Overview, Table 1, and the switching-path doc)*

| Adjacency | What the device does |
|---|---|
| **Glean** | Next hop is directly connected, but **no MAC rewrite yet**. The FIB holds the subnet prefix and points to glean; CEF triggers ARP and then builds the host adjacency |
| **Punt** | Packet needs special handling or a feature CEF doesn't support, so it is **sent to the next higher switching level** (e.g. the CPU) |
| **Null** | Destined to **Null0**, so it is dropped. Usable as access filtering |
| **Drop** / **Discard** | The packet is dropped / discarded |
| **Receive** | Destined to the router itself (its own addresses, broadcasts) |

**CEF load balancing — per-destination vs per-packet**

| | Per-destination (default) | Per-packet |
|---|---|---|
| Unit balanced | Flow (source + destination hash) | Individual packet |
| Packet order | Preserved: one flow uses one path | Can arrive **out of order** at the destination |
| Link utilization | Statistical, uneven with few large flows | Even across links |
| VoIP / real-time | Safe | Degrades (choppy audio, jitter) |
| Configured with | `ip load-sharing per-destination` (default) | `ip load-sharing per-packet` |

## Config Patterns
```ios-xe
! Directly attached static route (point-to-point, non-ARP interface only)
ip route 10.22.22.0 255.255.255.0 Serial1/0

! Recursive static route (next-hop IP, requires a second RIB lookup)
ip route 10.22.22.0 255.255.255.0 10.12.1.2

! Fully specified static route (interface + next-hop IP, no recursive lookup)
ip route 10.22.22.0 255.255.255.0 GigabitEthernet0/0 10.12.1.2

! Floating static route — higher AD (210) as backup to a preferred path (AD 10)
ip route 10.22.22.0 255.255.255.0 10.12.1.2 10
ip route 10.22.22.0 255.255.255.0 Serial1/0 210

! Static route to Null0 — prevents routing loops on an unused summarized block
ip route 172.16.0.0 255.255.240.0 Null0

! IPv6 static routing
ipv6 unicast-routing
ipv6 route 2001:db8:22::/64 2001:db8:12::2

! VRF creation and interface assignment
vrf definition MGMT
 address-family ipv4
interface GigabitEthernet0/3
 vrf forwarding MGMT
 ip address 10.0.3.1 255.255.255.0
! VRF-scoped static route (global config mode, note the vrf keyword)
ip route vrf MGMT 10.0.9.0 255.255.255.0 10.0.3.2

! CEF load sharing, set on each outbound interface of the equal-cost paths
interface GigabitEthernet0/1
 ip load-sharing per-destination   ! default: flows stay on one path
! ip load-sharing per-packet        ! even links, but reorders packets; avoid for VoIP
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Dynamic routing preferred over static sprawl; every static documented with a purpose | Statics don't converge on failure and rot silently | Stub sites, last-resort defaults, deliberate security boundaries | ENCOR 350-401 OCG |
| Floating static ADs chosen deliberately relative to the primary source | Predictable failover order instead of accidental preference | — | ENCOR 350-401 OCG |
| Null0 discard route paired with every locally originated summary | Prevents loops for unused space inside the summary | — | ENCOR 350-401 OCG |
| Fully specified statics (interface + next-hop) on multi-access interfaces | Directly attached statics on Ethernet force ARP for every matching destination | Point-to-point interfaces, where interface-only statics are fine | ENCOR 350-401 OCG |
| PBR only with a documented business need | Invisible in `show ip route`; overrides destination routing in ways the next engineer won't expect | Compliance or source-based egress steering requirements | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip route` | Installed routes in the global RIB — source code letter (C/S/O/D/B/etc.), AD/metric in brackets `[AD/metric]` (absent for directly attached static/connected routes), next hop and outbound interface |
| `show ip route <prefix>` | Full descriptor block for one route — source protocol, distance, metric, and per-path traffic share count (useful for confirming ECMP or unequal-cost ratios) |
| `show ip route vrf <vrf-name>` | The routing table for one specific VRF — entries here never appear in the global `show ip route` output |
| `show ipv6 route` | IPv6 equivalent of `show ip route`, same code letters with IPv6-specific additions (O, OI, OE1/2, D, etc.) |
| `traceroute <dest> source <src>` | Confirms the actual forwarding path hop-by-hop — essential for verifying PBR is steering traffic differently than the plain RIB path would |
| `show ip arp` / `show ip arp <ip>` | IP→MAC bindings and age. **`Incomplete` means ARP was attempted and nobody answered** — the host is absent, off, or on the wrong VLAN |
| `show ip interface <id> \| include Proxy` | Whether proxy ARP is enabled — check this before concluding a host's mask is correct |
| `show arp timeout` / `show ip interface <id>` | ARP cache timeout (default 4hr) — compare against the switch's 300s MAC aging when diagnosing sustained unicast flooding |
| `show ip cef` / `show ip cef <prefix>` | The FIB itself: prefix, next hop, outbound interface. A route in `show ip route` but missing here is not being forwarded by CEF |
| `show adjacency detail` | The adjacency table: per next hop, the outbound interface and the pre-built MAC rewrite string. An entry stuck as glean/incomplete means ARP never resolved |
| `show ip cef <prefix> internal` | The CEF entry's load-sharing paths and hash buckets for the prefix: confirms ECMP made it into the FIB, not just the RIB |
| `show ip cef exact-route <src-ip> <dst-ip>` | The single path CEF picks for that source/destination pair. Under per-destination it is the same every time, which is how you prove a flow sticks to one link |

## Intent Questions
- Which routes should be in the RIB, from which source (connected/static/protocol), at which AD?
- Are statics or floating statics supposed to exist here — and is each one's purpose still documented?
- Is any traffic supposed to bypass destination-based routing (PBR), and does anyone remember why?
- Which VRF is this interface or route supposed to live in?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1/2: confirm the outbound interface for a directly attached or fully specified static route is physically up — the route is pulled from the RIB the moment that interface goes down.
2. Layer 3 next-hop reachability: for recursive static routes, confirm the next-hop IP actually resolves via the RIB (and not solely through a default route, which recursive statics can't use) — `show ip route <next-hop-ip>`.
3. Route not installed as expected: compare prefix length first, then AD (`show ip route <prefix>` to see which source actually won) — remember the RIB only sees the single best route each protocol submits, so a protocol's own internal best-path choice (e.g. BGP preferring iBGP) can mean a numerically lower AD elsewhere doesn't win.
4. PBR producing unexpected forwarding behavior: remember `show ip route` will look completely normal even when PBR is actively redirecting traffic — use `traceroute` with the relevant source address to see the real path, and confirm the PBR route-map's next hop(s) are actually present in the RIB.
5. Config error: directly attached static route configured on an ARP-capable (Ethernet) interface instead of a true point-to-point link — causes excessive ARP processing and potential instability; convert to a recursive or fully specified route instead.
6. Config error: floating static route never taking over after a primary failure — confirm its AD is genuinely higher than the primary's, and that the primary route is actually being withdrawn from the RIB (not just the interface flapping while the route stays installed).
7. Config error: VRF interface losing its IP address after `vrf forwarding <vrf-name>` is applied — this is expected behavior (assigning a VRF clears the interface's IP), not a bug; the address must be reconfigured afterward.
8. Software/platform bug (rare) — only after prefix/AD/metric logic, static route type, PBR route-map, and VRF assignment are all confirmed correct.

## Common Pitfalls
- **Treating FIB and CEF as competing answers.** CEF is the *switching method*; the FIB and the adjacency table are the *two tables it uses*. FIB is built from the RIB, adjacency from ARP. "Which two tables does CEF use?" = FIB + adjacency, never "FIB + RIB."
- **Assuming the route cache still exists under CEF.** Demand-built caches, aging, and first-packet process switching belong to fast switching. CEF pre-builds everything from the RIB and ARP.
- Forgetting that PBR does not modify or appear in the RIB at all — `show ip route` is the wrong tool for verifying PBR behavior; use `traceroute`/`ping` with the relevant source, or PBR-specific route-map hit counters.
- Using a directly attached static route on an Ethernet (ARP) interface — forces a fresh ARP lookup for every destination matching the route, unlike serial/point-to-point links where this pattern is safe.
- Trying to resolve a recursive static route's next hop purely via a default route entry — recursive statics explicitly cannot use 0.0.0.0/0 as their resolving route and will fail to install.
- Assuming the lowest AD overall always wins in the RIB — it's actually the lowest AD *among routes a process actually submits*, and a routing protocol's internal best-path selection (e.g. BGP) can submit a higher-AD path (iBGP at 200) even when a lower-AD option (eBGP at 20) exists elsewhere in that protocol's table.
- Mixing up ECMP (automatic, equal-metric, protocol-default-enabled) with EIGRP's unequal-cost load balancing (manual, different-metric, must be explicitly configured) — they produce very different traffic ratios and only EIGRP supports the unequal-cost variant.
- **Enabling CEF per-packet load balancing on links that carry VoIP.** Packets of one call take different paths and **arrive out of order at the destination**, causing choppy audio. Keep the default per-destination, or keep voice off per-packet links. Reordering happens in transit/at the destination, **not** at the source.
- Forgetting that assigning `vrf forwarding <vrf-name>` to an interface strips its previously configured IP address — always re-apply the IP address afterward, in that order.
- **Configuring a static route for a VRF destination and omitting the `vrf` keyword.** The route installs cleanly in the global table, the VRF's table still has no path, and `show ip route` looks fine — you have to run `show ip route vrf <name>` to see the hole.
- **Expecting a host to ARP for an off-subnet destination** — it never will. It ARPs for its default gateway and sends the frame there with the destination IP unchanged. If you're capturing and don't see an ARP for the far-end host, that's correct behavior, not a fault.
- **Reading "Destination Host Unreachable" as proof a router replied.** On a same-subnet ARP failure that message is generated locally and no packet ever left the host; only the different-subnet case produces a real ICMP Type 3 Code 1 from a router.
- **Letting proxy ARP hide a bad subnet mask.** Connectivity works, the config is wrong, and it breaks later for reasons that look unrelated. Multiple IPs mapping to one MAC in `arp -a` is the signature.
- Diagnosing an `Incomplete` ARP entry as a routing problem — it's purely L2/L1: wrong VLAN, host down, or port in the wrong access VLAN. The RIB is irrelevant when the destination is on-subnet.
