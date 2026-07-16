---
name: ccnp-ip-routing-essentials
description: >
  Use this skill when troubleshooting or configuring ip-routing-essentials on IOS-XE.
  Invoke when the user asks about: RIB, FIB, administrative distance, prefix
  length, ECMP, unequal-cost load balancing, static route, floating static
  route, null route, policy-based routing, VRF.
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
- **ECMP** (equal-cost multipathing): when a protocol has multiple equal-metric paths and supports it, all are installed and traffic load-shares evenly. **Unequal-cost load balancing**: EIGRP-only, not default-enabled, installs multiple different-metric paths and ratios traffic proportional to each path's metric (lower metric gets more traffic share).
- Static route types: **Directly attached** (`ip route <net> <mask> <interface>`) — only valid on point-to-point interfaces without ARP (e.g. serial); using it on an Ethernet/ARP-capable interface forces ARP for every destination matching the route and can cause instability. **Recursive** (`ip route <net> <mask> <next-hop-ip>`) — requires a second RIB lookup to resolve the next-hop IP to an interface; cannot resolve via a default route (0.0.0.0/0) entry. **Fully specified** (`ip route <net> <mask> <interface> <next-hop-ip>`) — both interface and next-hop IP given, avoids the recursive lookup and ARP issues, and the route is pulled from the RIB if the named interface goes down.
- **Floating static routes** use a deliberately higher AD than the primary route so they only get installed as backup when the primary is withdrawn — common pattern for backup links behind a preferred dynamic-routing or lower-AD static path.
- **Null route** (`ip route <summary-net> <mask> Null0`) drops any traffic matching a summarized range that doesn't match a more specific real route — prevents routing loops on a router that's advertising (or receiving) a summarized block it doesn't fully use, without needing an ACL.
- IPv6 static routing mirrors IPv4: requires `ipv6 unicast-routing` enabled globally, then `ipv6 route <prefix>/<length> {interface-id | [interface-id] next-hop-ip}` — if the next hop is a link-local address, the route must be fully specified (interface + next-hop) since link-locals aren't globally routable on their own.
- **Policy-based routing (PBR)** overrides destination-based forwarding using packet characteristics (protocol, source/destination IP, etc.) to set a different next hop — verifies next-hop reachability in the RIB before using it, supports a prioritized list of fallback next hops, and silently fails closed (normal RIB forwarding) if none of the specified next hops are reachable. PBR does not modify the RIB itself — `show ip route` looks unchanged even with active PBR policies, which complicates troubleshooting since the conditional next hop isn't visible there.
- **VRF (Virtual Routing and Forwarding)** creates isolated logical routers on one physical box — separate routing/forwarding tables per VRF, allowing overlapping IP address ranges across VRFs with no conflict. All interfaces default to the **global VRF** (the standard routing table) until explicitly assigned elsewhere. Conceptually similar to VLANs on a switch, but VRF segmentation operates at Layer 3 with full per-VRF dynamic routing rather than 802.1Q tagging at Layer 2.

## Procedure
BGP path vector loop avoidance (illustrative 4-AS example: AS1–AS2–AS4–AS3, prefix 10.1.1.0/24 originated in AS1):
1. R1 (AS 1) advertises 10.1.1.0/24 to R2 (AS 2), adding AS 1 to the AS_Path.
2. R2 advertises the prefix to R4 (AS 4), adding AS 2 to the AS_Path (now "2 1").
3. R4 advertises the prefix to R3 (AS 3), adding AS 4 to the AS_Path (now "4 2 1").
4. R3 advertises the prefix back toward R1 and R2, adding AS 3 to the AS_Path (now "3 4 2 1").
5. R1 receives this advertisement, detects its own AS (1) already present in the AS_Path, and rejects it as a loop; R2 does the same upon detecting AS 2 in the path.

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
- Forgetting that PBR does not modify or appear in the RIB at all — `show ip route` is the wrong tool for verifying PBR behavior; use `traceroute`/`ping` with the relevant source, or PBR-specific route-map hit counters.
- Using a directly attached static route on an Ethernet (ARP) interface — forces a fresh ARP lookup for every destination matching the route, unlike serial/point-to-point links where this pattern is safe.
- Trying to resolve a recursive static route's next hop purely via a default route entry — recursive statics explicitly cannot use 0.0.0.0/0 as their resolving route and will fail to install.
- Assuming the lowest AD overall always wins in the RIB — it's actually the lowest AD *among routes a process actually submits*, and a routing protocol's internal best-path selection (e.g. BGP) can submit a higher-AD path (iBGP at 200) even when a lower-AD option (eBGP at 20) exists elsewhere in that protocol's table.
- Mixing up ECMP (automatic, equal-metric, protocol-default-enabled) with EIGRP's unequal-cost load balancing (manual, different-metric, must be explicitly configured) — they produce very different traffic ratios and only EIGRP supports the unequal-cost variant.
- Forgetting that assigning `vrf forwarding <vrf-name>` to an interface strips its previously configured IP address — always re-apply the IP address afterward, in that order.
