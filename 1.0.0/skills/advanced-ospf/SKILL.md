---
name: ccnp-advanced-ospf
description: >
  Use this skill when troubleshooting or configuring multi-area OSPF on
  IOS-XE. Invoke when the user asks about: OSPF areas, area border router,
  ABR, backbone area, Area 0, LSA types, type 3 summary LSA, discontiguous
  network, OSPF path selection, intra-area vs inter-area, OSPF
  summarization, OSPF route filtering, area range, area filter-list.
---

## Purpose
Multi-area OSPF segments a routing domain into smaller areas so the LSDB, SPF calculation, and flooding scope stay manageable as a network grows, while Area 0 (the backbone) ties every area together and prevents routing loops between them.

## Key Concepts
- An **OSPF area** is a logical grouping of router interfaces (not whole routers) — area membership is set per interface, and an interface belongs to only one area. All routers within the same area maintain an identical LSDB for that area.
- Splitting a domain into areas fixes three problems with one giant area: a link flap forces a full SPF recalculation for every router in the area, the LSDB grows unmanageably large (more memory, slower SPF), and no route summarization can occur. Routers outside a changed area only run a partial SPF (metric/prefix updates), not a full recalculation.
- **Area 0 (the backbone)** is special: by design every other area must connect to it, because OSPF expects all areas to inject routes into the backbone and the backbone to redistribute them back out to other areas. A backbone design that isn't contiguous breaks inter-area routing (see Discontiguous Networks below).
- An **area border router (ABR)** is an OSPF router connected to Area 0 and at least one other area (per Cisco/RFC 3509). An ABR must participate in Area 0 — otherwise its other-area routes never reach the backbone. ABRs compute a separate SPF tree for every area they participate in, and maintain a separate LSDB per area.
- Connecting to multiple areas does NOT automatically mean routes cross between them — only a proper ABR (one that also participates in Area 0) injects routes from one area into another.
- The **area ID** is a 32-bit field, written either in simple decimal (0–4,294,967,295) or dotted-decimal (0.0.0.0–255.255.255.255). One router can configure decimal and its neighbor dotted-decimal for the same area and still form an adjacency — OSPF always advertises the area ID in dotted-decimal format inside the hello packet.
- **OSPF route types**, shown in the routing table:
  - **Intra-area route (`O`)** — destination is in the same area as the calculating router.
  - **Inter-area route (`O IA`)** — destination is in a different area, reached via an ABR.
  - **External route** — learned from outside the OSPF domain via redistribution; beyond ENCOR exam scope.
- **LSA sequence numbers** (32-bit) version-control LSAs: a higher sequence number than what's in the LSDB means the LSA is newer and gets processed; a lower or equal sequence number means it's stale and gets discarded.
- **LSA age** increments by 1 every second once installed in the LSDB. Once a router's own LSA reaches 1800 seconds (30 minutes), the originating router re-floods it with age reset to 0 (periodic refresh, independent of topology changes). If an LSA's age reaches 3600 seconds without refresh, it's deemed invalid and purged from the LSDB — flooding age increments as it traverses routers act as a secondary consistency safety net.
- All routers within an area carry an identical set of LSAs for that area; ABRs maintain a separate LSA set per area they touch, and most LSAs in one area differ from those in another.
- **LSA Type 1 (router LSA)**: every OSPF router originates one. Contains an entry for each OSPF-enabled link/interface and its attached networks, with link type, neighbor-correlating info (neighbor RID, or DR interface address on multi-access segments with a DR), and interface metric. Type 1 LSAs are never flooded outside the originating area — the underlying topology of an area is invisible from outside it. Viewed with `show ip ospf database router`.
- **LSA Type 2 (network LSA)**: represents a multi-access network segment that has elected a DR. Only the DR advertises the type 2 LSA, listing every router attached to that segment. If no DR has been elected on a segment, no type 2 LSA exists (the corresponding type 1 transit-link entry is treated as a stub instead). Like type 1, type 2 LSAs stay inside the originating area. Viewed with `show ip ospf database network`.
- **LSA Type 3 (summary LSA)**: represents a network from a different area. ABRs build these — type 1/type 2 LSAs are never forwarded directly into another area. When an ABR receives a type 1 LSA, it creates a type 3 LSA referencing that network (using the type 2 LSA to determine the multi-access network's mask) and advertises the type 3 LSA into other areas. If an ABR receives a type 3 LSA from Area 0, it regenerates a new type 3 LSA for the non-backbone area, listing itself as the advertising router and adding its own cost. Viewed with `show ip ospf database summary` (can append a network prefix to restrict output).
- **LSA Type 4 (ASBR summary LSA)**: advertises a summary LSA for a specific ASBR. **LSA Type 5 (AS external LSA)**: advertises redistributed routes. **LSA Type 7 (NSSA external LSA)**: advertises redistributed routes inside an NSSA. Types 4/5/7 relate to external-route redistribution, which is beyond ENCOR exam scope.
- **Type 3 LSA metric logic**: if the type 3 LSA is built from a type 1 LSA, its metric is the total path cost to reach the originating router. If it's built from a type 3 LSA received from Area 0, its metric is the cost to reach that ABR plus the metric already in the original type 3 LSA. An ABR advertises only one type 3 LSA per prefix even if it knows of multiple paths (intra-area and/or inter-area) — the best path's metric is the one used.
- **Discontiguous network**: occurs when inter-area traffic must cross a non-backbone area to reach its destination (e.g., Area 12 → Area 23 → Area 34, where Area 23 isn't the backbone). The simplest fix is ensuring Area 0 stays contiguous; virtual links or GRE tunnels are other (more complex) fixes, both beyond ENCOR scope. In real networks, discontiguous backbones usually arise from hardware failures partitioning Area 0 — designing for path redundancy into the backbone matters.
- **OSPF path selection priority**: 1) intra-area, 2) inter-area, 3) external routes (extra logic, out of scope). Intra-area routes are *always* preferred over inter-area routes for the same destination, even if an inter-area path has a numerically lower total metric — this is a common exam trap. Ties in metric within the same route-type tier install multiple paths (ECMP).
- **Equal-Cost Multipathing (ECMP)**: when path selection finds multiple equal-metric best paths, all are installed (default max 4 paths) — overridable with `maximum-paths` under the OSPF process.
- **Summarization** happens only at ABRs (since every router in an area must hold an identical LSDB, summarization can't happen mid-area) and works only on type 1 LSAs being converted to type 3. It shrinks the LSDB on the far side of the ABR and can eliminate SPF recalculation outside the area entirely for the summarized range, since the more specific prefixes are hidden from routers beyond the ABR.
- **Inter-area summarization metric**: by default IOS XE sets the type 3 summary LSA's metric to the lowest metric among the component routes (RFC 1583 guidance) — same dynamic-recheck behavior as EIGRP: removing or adding a component route can shift the summary to a new lowest metric and trigger a re-advertisement. The metric can instead be statically pinned with the `cost` keyword.
- An ABR performing inter-area summarization installs a **discard route** to Null0 matching the summarized range, to prevent routing loops for any portion of the range that doesn't have a more specific route in the RIB. AD for the OSPF summary discard route is 110 for internal networks, 254 for external.
- **Route filtering** with link-state protocols is harder than with vector protocols, because every router in an area shares an identical LSDB — filtering generally has to happen as routes enter/leave an area at the ABR, not at arbitrary points in the flood path.

## Procedure
Type 3 LSA metric calculation as a prefix crosses multiple ABRs (example: R2 advertises 10.123.1.0/24 in Area 1234; it must reach R6 in Area 56, via ABRs R4 and R5):
1. R4 (ABR between Area 1234 and Area 0) receives R2's type 1 LSA for 10.123.1.0/24 and creates a new type 3 LSA using metric 65 — the cost of 1 for R2's LAN interface plus 64 for the serial link between R2 and R4.
2. R4 advertises that type 3 LSA (metric 65) into Area 0.
3. R5 (ABR between Area 0 and Area 56) receives the type 3 LSA from Area 0 and creates a new type 3 LSA for Area 56, using metric 66 — the cost of 1 for the R4–R5 link plus the original type 3 LSA's metric of 65.
4. R6, inside Area 56, receives R5's type 3 LSA. R6 adds its own cost to reach the ABR R5 (1) to the metric already in the type 3 LSA (66), giving a total path metric of 67 to reach 10.123.1.0/24.

Each ABR along the path only ever adds its own local cost to the metric it received — it never has visibility into the original topology beyond the previous ABR, which is exactly what keeps each area's internal topology hidden from the others.

## Reference Tables
OSPF LSA types used for IPv4 routing:

| Type | Name | Function |
|---|---|---|
| 1 | Router LSA | Advertises the LSAs that originate within an area (every OSPF-enabled link) |
| 2 | Network LSA | Advertises a multi-access network segment attached to a DR |
| 3 | Summary LSA | Advertises network prefixes that originated from a different area |
| 4 | ASBR summary LSA | Advertises a summary LSA for a specific ASBR |
| 5 | AS external LSA | Advertises LSAs for routes that have been redistributed |
| 7 | NSSA external LSA | Advertises redistributed routes in NSSAs |

Fundamental rules ABRs use for creating type 3 LSAs:

| Source of the LSA at the ABR | Resulting action |
|---|---|
| Type 1 LSA received from a non-backbone area | Creates type 3 LSAs into the backbone area AND non-backbone areas |
| Type 3 LSA received from Area 0 | Creates a type 3 LSA for the non-backbone area only |
| Type 3 LSA received from a non-backbone area | Inserted into the LSDB for the source area only — no new type 3 LSA is created for any other area (including a segmented Area 0) |

OSPF route filtering techniques:

| Technique | Command | Effect |
|---|---|---|
| Filtering with summarization | `area area-id range network subnet-mask not-advertise` | Suppresses type 3 LSA creation for every network inside the range — those component routes stay visible only within the originating area |
| Area filtering (inbound/outbound) | `area area-id filter-list prefix prefix-list-name {in \| out}` | Filters specific prefixes as type 3 LSAs are advertised into (`in`) or out of (`out`) an area on the ABR, without requiring the prefixes to summarize together |

## Config Patterns
```ios-xe
! Basic multi-area OSPF on an ABR — one interface per area, including Area 0
router ospf 1
 router-id 192.168.4.4
 network 10.24.1.0 0.0.0.255 area 1234
 network 10.45.1.0 0.0.0.255 area 0

! Inter-area route summarization on an ABR, with a static cost
router ospf 1
 router-id 192.168.2.2
 area 12 range 172.16.0.0 255.255.0.0 cost 45

! Filtering with summarization — block a range from being advertised at all
router ospf 1
 area 12 range 172.16.2.0 255.255.255.0 not-advertise

! Area filtering with a prefix-list, applied at the ABR
ip prefix-list PREFIX-FILTER seq 5 deny 172.16.1.0/24
ip prefix-list PREFIX-FILTER seq 10 permit 0.0.0.0/0 le 32
!
router ospf 1
 area 0 filter-list prefix PREFIX-FILTER in
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip ospf interface brief` | Per-interface area assignment, cost, state, and neighbor count — confirms an ABR has interfaces in both Area 0 and the non-backbone area |
| `show ip route \| begin Gateway` | `O` (intra-area) vs `O IA` (inter-area) prefixes — confirms which routes are local to the area vs learned via an ABR |
| `show ip ospf database router` | Type 1 LSAs for the current area — link IDs, link types, and per-link metrics |
| `show ip ospf database network` | Type 2 LSAs — DR pseudonode and attached router list for multi-access segments |
| `show ip ospf database summary [prefix]` | Type 3 LSAs — link-state ID, advertising ABR, and metric for inter-area prefixes |
| `show ip route ospf` | Confirms summarization: a single summary entry plus `is a summary ... Null0` discard route, with component routes suppressed beyond the ABR |
| `show ip protocols` | Configured `area range` and `area filter-list` statements under the OSPF process |

## Troubleshooting Checklist
1. Layer 1/2: confirm the physical/data-link path toward the expected ABR or neighbor is up — `show interfaces`.
2. Layer 3 adjacency: `show ip ospf neighbor` — confirm the expected neighbor is FULL; check that the interface is in the area you expect via `show ip ospf interface brief`.
3. Missing inter-area routes: confirm the router claiming to be an ABR actually has an interface in Area 0 — a router touching two non-backbone areas without an Area 0 interface will not inject routes between them.
4. Discontiguous network symptoms (routes missing between two non-backbone areas): verify Area 0 is contiguous end-to-end; a partitioned backbone (often from a hardware failure) silently breaks inter-area route propagation.
5. Unexpected path choice: remember intra-area always wins over inter-area regardless of metric — before assuming a routing bug, check `show ip route <prefix>` for `type intra area` vs `type inter area` rather than just comparing metrics.
6. Summarization not suppressing component routes: confirm the `area range` is configured on the ABR (not a non-ABR router) and that the range actually covers the component prefixes; also check for a `not-advertise` keyword that may be unintentionally blocking the summary itself.
7. Route filtered unexpectedly: check both `area range ... not-advertise` and `area filter-list` statements on every ABR along the path — filtering can occur at multiple points, and `in`/`out` direction is easy to get backwards.
8. Config error: area ID mismatch between two ends of a link (decimal vs. dotted-decimal formatting is fine, but the actual area number/ID must match) — neighbor won't form or settles in a non-FULL state.
9. Software/platform bug (rare) — only after adjacency, area assignment, and area-range/filter-list config are all confirmed correct.

## Common Pitfalls
- Assuming any router touching two areas automatically routes between them — it only does if it's a proper ABR with an Area 0 interface; a router in Area 1 and Area 2 but not Area 0 will not exchange routes between Area 1 and Area 2.
- Picking the lower-metric inter-area path over a higher-metric intra-area path — OSPF never does this; intra-area beats inter-area unconditionally, independent of metric.
- Expecting type 1 or type 2 LSAs to cross an area boundary — they never do; only type 3 (and 4/5/7 for external routes) cross between areas, which is also why an area's internal topology is invisible from outside it.
- Forgetting that `area range` (summarization) only works on type 1 LSAs — it can't summarize already-summarized type 3 LSAs arriving from another area, so summarization should be designed at the area boundary closest to the original source.
- Trying to filter a single prefix out of one specific downstream area using summarization-based filtering (`not-advertise`) — that technique blocks the prefix from *every* area beyond the ABR; selective per-area filtering requires `area filter-list` with a prefix-list instead.
- Designing a topology where inter-area traffic must transit a non-backbone area (discontiguous network) — looks like normal connectivity in the topology diagram but silently breaks OSPF's area-based path computation; always verify Area 0 is the only area routes traverse between two other areas.
