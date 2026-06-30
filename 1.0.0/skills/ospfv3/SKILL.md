---
name: ccnp-ospfv3
description: >
  Use this skill when troubleshooting or configuring OSPFv3 on IOS-XE.
  Invoke when the user asks about: OSPFv3, OSPF for IPv6, IPv6 link-local
  adjacency, OSPFv3 instance ID, link LSA, intra-area prefix LSA, OSPFv3
  passive-interface, OSPFv3 summarization, OSPFv3 IPv4 support.
---

## Purpose
OSPFv3 is the IPv6-capable evolution of OSPF — it strips IP addressing out of the core protocol so the same link-state mechanics from OSPFv2 (areas, LSA flooding, SPF) can run address-family independent, supporting IPv6, and optionally IPv4, side by side.

## Key Concepts
- OSPFv3 is **not backward compatible** with OSPFv2 — they are separate processes/protocols — but the underlying mechanisms (areas, ABRs, LSA flooding scope, SPF calculation, path selection, summarization) covered for OSPFv2 in earlier chapters carry over essentially unchanged.
- Core differences from OSPFv2:
  - **Multiple address family support** — OSPFv3 supports both IPv6 and IPv4 address families, not just one protocol per address family.
  - **New LSA types** were added specifically to carry IPv6 prefix information.
  - **Removal of addressing semantics** — IP prefix information is no longer carried in the OSPF packet headers at all; it's carried only as LSA payload data. This makes the protocol essentially address-family independent, similar in spirit to IS-IS. Because of this shift, OSPFv3 uses the term **link** instead of **network**, since SPT (shortest path tree) calculations are performed per link rather than per subnet.
  - **LSA flooding** uses a new link-state type field in the LSA header to determine flooding scope and how to handle unrecognized LSA types.
  - **Packet format** changed: OSPFv3 runs directly over IPv6 (not encapsulated the way OSPFv2 rides over IPv4), and the OSPF packet header itself has fewer fields than OSPFv2's.
  - **Router ID**: still used to identify neighbors regardless of network type, but on IOS routers the OSPFv3 router ID must always be manually assigned in the routing process — OSPFv3 does fall back to the same RID auto-detection algorithm as OSPFv2 (lowest/highest IPv4 address depending on platform), but if no IPv4 interface exists at all, the RID defaults to 0.0.0.0 and adjacencies will not form.
  - **Authentication** has been removed from the OSPF protocol itself; it's instead handled through IPsec extension headers in the IPv6 packet.
  - **Neighbor adjacencies** are built using IPv6 link-local addressing for inter-router communication. Neighbors are not auto-discovered over NBMA interfaces — they must be manually specified by link-local address. Because IPv6 allows multiple subnets on one interface, OSPFv3 adjacencies can form even when the two routers don't share a common subnet (as long as they share link-local connectivity).
  - **Multiple instances**: OSPFv3 packets carry an instance ID field, which can be used to control which routers on a shared segment are allowed to form adjacencies with each other.
- **New LSA structure**: OSPFv3 restructures the router LSA (type 1) so it's only responsible for announcing interface parameters — interface type (point-to-point, broadcast, NBMA, point-to-multipoint, virtual link) and metric/cost — not IP prefix data. It also renames OSPFv2's network summary LSA to the **inter-area prefix LSA**, and the ASBR summary LSA to the **inter-area router LSA**.
- IP address (prefix) information is now carried independently by two new LSA types: the **intra-area prefix LSA** and the **link LSA**. Splitting prefix data out this way means OSPF no longer has to run a full SPF recalculation every time an IP address is added or changed on an interface — the Dijkstra SPT calculation only examines router and network LSAs, so prefix-only changes trigger a lighter-weight update instead of a full SPF run.
- **OSPFv3 communication**: uses IPv6 protocol number 89, sourced from the local interface's IPv6 link-local address. It uses the same five OSPF packet types and logic as OSPFv2. Destination address is either a unicast link-local address or one of two multicast link-local scoped addresses depending on packet type (see Reference Tables).
- The DR and BDR send link-state update and flooding-acknowledgment messages to **FF02::5 (AllSPFRouters)**; non-DR/BDR routers send updates/acks to the DR and BDR via **FF02::6 (AllDRouters)** instead of flooding to all routers.
- **OSPFv3 does not use the `network` statement** to enable interfaces (unlike OSPFv2) — it's enabled directly under the interface with `ospfv3 process-id ipv6 area area-id`. The address family is auto-enabled on the interface as soon as OSPFv3 is configured on it; explicitly initializing `address-family {ipv6 | ipv4} unicast` under the process is optional.
- Legacy IOS commands `ipv6 router ospf` (process init) and `ipv6 ospf process-id area area-id` (interface enablement) are deprecated/legacy — current syntax is `router ospfv3 [process-id]` and `ospfv3 process-id ipv6 area area-id`.
- **Passive interfaces**: configured under the OSPFv3 process (`passive-interface interface-id` or `passive-interface default` for all interfaces) or under a specific address family — placing it under the global process cascades the setting to both IPv4 and IPv6 address families. Re-enable a specific interface with `no passive-interface interface-id`. Marking a previously-adjacent interface passive immediately tears down that neighbor relationship (FULL → DOWN).
- **Summarization**: internal OSPFv3 route summarization follows the same area-boundary rule as OSPFv2 — it must occur on ABRs. The config command differs in syntax: `area area-id range prefix/prefix-length`, configured under the `address-family ipv6 unicast` submode of the OSPFv3 process (not directly under the process like OSPFv2's `area range`).
- **Common IPv6-summarization mistake**: don't confuse hex and decimal when computing a summary boundary. Summarization math is still done in decimal, but the first and third hex digits of a hextet aren't decimal digits — e.g., the first hextet of `2001::1/128` is `2001` in hex, which is decimal 32 and 1 (not "20" and "1").
- **OSPFv3 network types**: same set as OSPFv2 (broadcast, point-to-point, NBMA, point-to-multipoint). Changed per-interface with `ospfv3 network {point-to-point | broadcast}`; verified in `show ospfv3 interface` output's "Network Type" line, or as P2P/DR/BDR state in `show ospfv3 interface brief`.
- **IPv4 support in OSPFv3** (per RFC 5838): OSPFv3 can carry IPv4 routes by setting the instance ID field in link LSAs into the IPv4-reserved range (64–95) instead of the IPv6-reserved range. Enabling it requires no separate IPv4-specific routing process — it rides the same OSPFv3 process already running for IPv6.
- **Verification command mapping**: OSPFv3 commands largely mirror OSPFv2 by replacing `ip ospf` with `ospfv3 ipv6` (or `ospfv3` generically when the address family is implied). Interfaces are identified by an **interface ID** value (not an IP address, since address semantics were removed) — e.g., `Interface ID 3` rather than an IP address tied to that link.

## Procedure
Enabling OSPFv3 on a router and interface (IPv6 address family):
1. Enable `ipv6 unicast-routing` globally — this is a prerequisite; OSPFv3 will not initialize without it.
2. Initialize the OSPFv3 process: `router ospfv3 [process-id]`.
3. Manually assign the router ID under the process: `router-id router-id` (any unique 32-bit value in IPv4-address format — it does not need to correspond to an actual IPv4 address on the router).
4. (Optional) Explicitly initialize the address family under the process: `address-family {ipv6 | ipv4} unicast` — not required, since the address family activates automatically once OSPFv3 is enabled on an interface, but required as the submode if you need to configure address-family-specific features like `area range` summarization.
5. Enable OSPFv3 directly on each interface and assign its area: `ospfv3 process-id ipv6 area area-id` (no `network` statement is used).

Adding IPv4 support to an already-running OSPFv3 (IPv6) deployment:
1. Confirm the interface already has an IPv6 address (global or link-local) configured — OSPFv3 for IPv4 still rides the existing IPv6-enabled OSPFv3 process and requires IPv6 reachability on the link.
2. Enable the OSPFv3 process for IPv4 on that same interface: `ospfv3 process-id ipv4 area area-id`. This is the only interface-level step needed — IPv4 addressing on the interface plus this command is sufficient; no separate `router ospfv3` IPv4-specific block is required.

## Reference Tables
OSPFv3 well-known multicast destination addresses:

| Address | Name | Used for |
|---|---|---|
| FF02::5 | AllSPFRouters | Every router sends hello messages here for neighbor discovery and down-detection; DR/BDR also use it to flood link-state updates and acknowledgments to all routers |
| FF02::6 | AllDRouters | Non-DR/BDR routers send link-state updates and acknowledgments here, addressed to the DR and BDR only |

OSPFv3 vs. OSPFv2 LSA naming and responsibility changes:

| OSPFv2 term | OSPFv3 term | Change |
|---|---|---|
| Router LSA (type 1) | Router LSA | Restructured to carry only interface type and metric — no IP prefix data |
| Network summary LSA | Inter-area prefix LSA | Renamed; same inter-area role |
| ASBR summary LSA | Inter-area router LSA | Renamed; same role for locating ASBRs across areas |
| (n/a — prefix data was in router/network LSA) | Intra-area prefix LSA | New — carries IP prefix info independently of router LSA |
| (n/a) | Link LSA | New — carries link-local address and prefix info for a single link |

OSPFv3 command reference (compare against OSPFv2 equivalents):

| Task | Command |
|---|---|
| Configure OSPFv3 on a router and enable it on an interface | `router ospfv3 [process-id]` / `interface interface-id` / `ospfv3 process-id {ipv4 \| ipv6} area area-id` |
| Configure a specific interface as passive | `passive-interface interface-id` |
| Configure all interfaces as passive | `passive-interface default` |
| Summarize an IPv6 network range on an ABR | `area area-id range prefix/prefix-length` (under `address-family ipv6 unicast`) |
| Set an interface's OSPFv3 network type | `ospfv3 network {point-to-point \| broadcast}` |
| Display OSPFv3 interface settings | `show ospfv3 interface [interface-id]` |
| Display OSPFv3 IPv6 neighbors | `show ospfv3 ipv6 neighbor` |

## Config Patterns
```ios-xe
! Prerequisite for any OSPFv3 deployment
ipv6 unicast-routing

! Basic OSPFv3 (IPv6) on an ABR — interfaces in two areas
interface Loopback0
 ipv6 address 2001:DB8::3/128
 ospfv3 1 ipv6 area 0
!
interface GigabitEthernet0/2
 ipv6 address FE80::3 link-local
 ipv6 address 2001:DB8:0:23::3/64
 ospfv3 1 ipv6 area 0
!
interface GigabitEthernet0/4
 ipv6 address FE80::3 link-local
 ipv6 address 2001:DB8:0:34::3/64
 ospfv3 1 ipv6 area 34
!
router ospfv3 1
 router-id 192.168.3.3

! Passive interface — one specific interface, then default-all with an exception
router ospfv3 1
 passive-interface GigabitEthernet0/1
!
router ospfv3 1
 passive-interface default
 no passive-interface GigabitEthernet0/3

! IPv6 summarization on an ABR
router ospfv3 1
 address-family ipv6 unicast
 area 0 range 2001:db8:0:0::/65

! Changing network type to point-to-point
interface GigabitEthernet0/3
 ospfv3 network point-to-point

! Adding IPv4 support to an existing OSPFv3 (IPv6) interface
interface GigabitEthernet0/1
 ospfv3 1 ipv4 area 0
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ospfv3 ipv6 neighbor` | Adjacency state (FULL/DR, FULL/BDR), dead timer countdown, interface ID and name |
| `show ospfv3 interface [interface-id]` | Link-local address, interface ID (not an IP), area/process/instance ID, router ID, network type, DR/BDR identity, hello/dead timers, `Passive interface` flag if applicable |
| `show ospfv3 interface brief` | Per-interface PID, area, address family (ipv4/ipv6 shown as separate rows when both are enabled), cost, state, neighbor count |
| `show ipv6 route ospf` | `O` (intra-area) vs `OI` (inter-area) IPv6 routes, next hop shown as the neighbor's link-local address |
| `show ip route ospfv3` | IPv4 routes learned via OSPFv3 when IPv4 support is enabled — same `O`/`O IA` notation as OSPFv2 |
| `show ospfv3 neighbor` | Neighbors broken out per address family (ipv4 vs ipv6) when a router runs both under the same process |

## Troubleshooting Checklist
1. Layer 1/2: confirm the physical/data-link path to the expected neighbor is up — `show interfaces`.
2. Prerequisite check: confirm `ipv6 unicast-routing` is enabled globally — without it, OSPFv3 will not initialize at all.
3. Layer 3 adjacency: `show ospfv3 ipv6 neighbor` — missing expected neighbor; confirm both sides have a link-local address (auto-assigned or static) and matching area on the link.
4. Router ID issue: confirm a router ID was manually assigned under `router ospfv3` — if no IPv4 interface exists anywhere on the router and the RID wasn't set manually, it defaults to 0.0.0.0 and adjacencies will not form.
5. Interface unexpectedly not participating: check `show ospfv3 interface interface-id` for a `Passive interface` indication — passive interfaces send no hellos and won't form adjacencies, and toggling an interface passive on an already-adjacent neighbor immediately tears the adjacency down.
6. NBMA-specific: confirm neighbors are manually specified by link-local address — OSPFv3 does not auto-discover neighbors over NBMA interfaces the way it does over broadcast media.
7. Instance ID mismatch: if two routers on the same segment unexpectedly fail to peer (or unexpectedly do peer when they shouldn't), check the OSPFv3 instance ID configured on each interface — it controls adjacency eligibility per RFC 5838's instance ID partitioning.
8. Summarization not suppressing component routes: confirm `area range` is configured under `address-family ipv6 unicast` on the ABR (not directly under the OSPFv3 process the way OSPFv2 does it) — wrong submode is the most common config-location mistake here.
9. IPv4-over-OSPFv3 not working: confirm the interface has IPv6 reachability (global or link-local) configured first — `ospfv3 process-id ipv4 area area-id` rides the existing IPv6-enabled OSPFv3 process and won't function without it.
10. Config error: area mismatch between two ends of a link, or `network` statement used by habit instead of the required interface-level `ospfv3` command — OSPFv3 silently ignores `network` statements since they aren't part of how OSPFv3 enables interfaces.
11. Software/platform bug (rare) — only after adjacency state, router ID, passive-interface config, and area/instance ID assignment are all confirmed correct.

## Common Pitfalls
- Trying to enable OSPFv3 on an interface using a `network` statement under the routing process, the way OSPFv2 works — OSPFv3 has no `network` statement; interfaces are enabled directly with `ospfv3 process-id ipv6 area area-id`.
- Forgetting to manually configure the router ID — OSPFv3 doesn't require this to be a real IPv4 address, but it does require it to be set; omitting it and having no IPv4 interfaces anywhere on the router silently leaves the RID at 0.0.0.0, blocking all adjacencies.
- Assuming OSPFv3 and OSPFv2 can interoperate or share configuration syntax — they are separate, non-backward-compatible protocols that happen to share LSA flooding/SPF concepts; troubleshooting habits transfer, commands and packet structure do not.
- Confusing hex and decimal digits when calculating an IPv6 summary boundary — the first/third digits of a hextet are hex digits, not decimal place values, and treating them as decimal produces a wrong summary range.
- Configuring `area range` directly under `router ospfv3` instead of under `address-family ipv6 unicast` — IPv6 summarization in OSPFv3 requires the address-family submode, unlike OSPFv2's `area range` which sits directly under the process.
- Assuming IPv4 support in OSPFv3 needs its own separate routing process or `network` statements — it only needs an IPv6-addressed interface plus `ospfv3 process-id ipv4 area area-id`; no parallel IPv4 process configuration is involved.
