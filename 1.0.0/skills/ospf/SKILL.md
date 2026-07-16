---
name: ccnp-ospf
description: >
  Use this skill when troubleshooting or configuring ospf on IOS-XE.
  Invoke when the user asks about: OSPF, LSDB, SPF, designated router, DR,
  BDR, OSPF neighbor states, OSPF network type, router ID, OSPF area 0.
---

## Purpose
OSPF is a nonproprietary link-state IGP that floods identical link-state info to every router in an area, letting each independently run Dijkstra SPF to build a synchronized, loop-free topology — this chapter covers single-area (Area 0) fundamentals.

## Key Concepts
- OSPF floods link-state advertisements (LSAs) unchanged, router to router; every router in an area ends up with an identical link-state database (LSDB) — a full map of the area, not just next-hop distances like distance vector protocols.
- Each router runs Dijkstra SPF treating itself as the root, producing a shortest path tree (SPT) unique to that router's perspective, even though every router used the same underlying LSDB.
- OSPF areas provide a two-tier hierarchy: Area 0 is the mandatory backbone, and all other (non-backbone) areas must connect to it — non-backbone areas advertise their routes into the backbone, which then re-advertises into other non-backbone areas. A non-backbone area's internal topology is hidden from routers outside it, reducing LSA traffic outside that area.
- A router can run multiple independent OSPF processes; each has its own LSDB, and routes don't cross between processes without explicit redistribution. OSPF process IDs are locally significant — they don't need to match between neighboring routers for an adjacency to form.
- OSPF runs directly over IP (protocol 89), using multicast where possible: AllSPFRouters (224.0.0.5) reaches every OSPF router; AllDRouters (224.0.0.6) is used specifically for communication with the DR/BDR.
- Router ID (RID): a 32-bit value uniquely identifying a router within an OSPF domain, synonymous with "neighbor ID" in show output. Selected dynamically at process startup — highest IP among up loopback interfaces, or if none, highest IP among up physical interfaces — and does not change until the process restarts, even if the interface that provided it changes. Setting it statically (`router-id`) aids troubleshooting and avoids unexpected RID churn.
- DR/BDR: on multi-access segments (Ethernet), full mesh adjacency between every router scales terribly — `n(n-1)/2` adjacencies for n routers — so OSPF elects a Designated Router to act as a pseudonode; every other router forms a full adjacency only with the DR (and BDR), cutting a 4-router segment from 6 adjacencies down to 3. If the DR fails, the BDR (which already maintains full adjacencies with everyone, precisely to make this fast) is promoted, and a new BDR election follows.
- DR/BDR roles do not preempt once elected — a higher-priority router added later won't take over until the current DR/BDR fails or the OSPF process restarts. Priority 0 on an interface withdraws it from DR/BDR eligibility entirely; default priority is 1.
- Network statement (`network <ip> <wildcard> area <id>`) is commonly misunderstood as "advertising" networks — it actually just selects which interfaces run OSPF and in which area; the LSA is what actually advertises the interface. The most specific (longest) matching network statement wins when an interface's IP matches more than one. Interface-specific config (`ip ospf <process-id> area <area-id>`) is the alternative method and takes precedence over network statements when both are present (hybrid config).
- Passive interfaces still get added to the LSDB (the segment is still advertised) but never send hellos or form adjacencies on that interface — protects against a rogue device plugged into an access segment while still advertising the prefix.
- Neighbor adjacency requirements: unique RIDs, interfaces sharing a common subnet (network masks must match, except point-to-point network type or virtual links), matching MTU (OSPF doesn't support fragmentation), matching area ID, matching DR enablement for the segment, matching hello/dead timers, matching authentication type/credentials, and matching area type flags (stub, NSSA, etc).
- OSPF network types determine whether DR/BDR election happens and what default timers apply: Broadcast (Ethernet default, DR required), Non-broadcast (Frame Relay main/multipoint default, DR required), Point-to-point (serial HDLC/PPP, GRE tunnel, Frame Relay P2P subinterface default — no DR), Point-to-multipoint (never default, host routes, hub-and-spoke — no DR), Loopback (loopback default, always advertised as /32 regardless of configured mask). See Reference Tables for the full timer/DR matrix.
- Default route advertisement: `default-information originate [always] [metric <value>] [metric-type <value>]` injects a default route into OSPF as an external route. Without `always`, a default route must already exist in the local RIB (e.g., a static default route) before OSPF will advertise one; `always` forces advertisement regardless.
- OSPF link cost = reference bandwidth ÷ interface bandwidth; default reference bandwidth is 100 Mbps, which makes FastEthernet and 10GigabitEthernet indistinguishable (both cost 1) — raising the reference bandwidth (`auto-cost reference-bandwidth`) restores differentiation on high-speed links, and must be changed identically on every router in the domain to keep SPF calculations consistent. Interface cost is capped at 65,535 (16-bit LSA field), though the accumulated path metric can exceed that since it's calculated locally.
- Failure detection: hello timer controls how often hellos are sent; dead interval (default 4× the hello timer) is how long a router waits without a hello before declaring a neighbor down and triggering new LSAs + SPF recalculation. Hello and dead intervals must match between neighbors for an adjacency to form, and the dead interval must always be set higher than the hello interval.

## Procedure
LSA flooding through a DR-elected segment (DR and BDR already established, all DROTHERs fully adjacent to both):
1. A router that learns of a new/changed route sends the updated LSA to the AllDRouters address (224.0.0.6) — only the DR and BDR accept and process packets sent to this address.
2. The DR sends a unicast acknowledgment back to the router that originated the LSA update.
3. The DR floods the LSA to every router on the segment via the AllSPFRouters address (224.0.0.5), completing propagation across the segment.

## Reference Tables
OSPF packet types:

| Type | Packet Name | Functional Overview |
|---|---|---|
| 1 | Hello | Discovers and maintains neighbors; sent periodically on all OSPF interfaces |
| 2 | Database description (DBD/DDP) | Summarizes LSDB contents when an adjacency first forms |
| 3 | Link-state request (LSR) | Requests a portion of a neighbor's database believed to be stale |
| 4 | Link-state update (LSU) | Explicit LSA for a specific link, typically sent in direct response to an LSR |
| 5 | Link-state ack | Acknowledges flooded LSAs, making flooding a reliable transport |

OSPF hello packet fields:

| Data Field | Description |
|---|---|
| Router ID (RID) | Unique 32-bit ID within the OSPF domain |
| Authentication options | None, clear text, or MD5 |
| Area ID | 32-bit OSPF area the interface belongs to |
| Interface address mask | Network mask for the interface's primary IP |
| Interface priority | DR/BDR election priority for this interface |
| Hello interval | Seconds between hello packets on this interface |
| Dead interval | Seconds to wait for a hello before declaring the neighbor down |
| Designated router / backup designated router | IP addresses of the DR and BDR for the segment |
| Active neighbor | List of neighbors seen on the segment within the dead interval |

OSPF neighbor states:

| State | Description |
|---|---|
| Down | Initial state — no hello received yet |
| Attempt | NBMA-only — no recent info, but still attempting contact |
| Init | A hello was received, but bidirectional communication isn't confirmed |
| 2-Way | Bidirectional communication confirmed; DR/BDR election happens here if needed |
| ExStart | First adjacency-forming state — routers negotiate primary/secondary for LSDB sync |
| Exchange | DBD packets are exchanged to describe link states |
| Loading | LSR packets request the more recent LSAs discovered but not yet received |
| Full | Neighbors are fully adjacent |

OSPF network types:

| Type | Default Media | DR/BDR Field Used | Hello / Wait / Dead |
|---|---|---|---|
| Broadcast | Ethernet | Yes | 10 / 40 / 40 |
| Non-broadcast | Frame Relay main/multipoint | Yes | 30 / 120 / 120 |
| Point-to-point | Serial (HDLC/PPP), GRE, Frame Relay P2P subinterface | No | 10 / 40 / 40 |
| Point-to-multipoint | None by default (manual, hub-and-spoke) | No | 30 / 120 / 120 |
| Loopback | Loopback interfaces | N/A | N/A |

Default OSPF interface costs (default 100 Mbps reference bandwidth):

| Interface Type | OSPF Cost |
|---|---|
| T1 | 64 |
| Ethernet | 10 |
| FastEthernet | 1 |
| GigabitEthernet | 1 |
| 10 GigabitEthernet | 1 |

## Config Patterns
```ios-xe
! Basic OSPF process with static router ID and network statement (Area 0)
router ospf 1
 router-id 192.168.1.1
 network 10.0.0.0 0.255.255.255 area 0

! Interface-specific OSPF enablement (alternative to network statement)
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 ip ospf 1 area 0

! Passive interfaces — protect an access segment, all-passive then selectively enable
router ospf 1
 passive-interface default
 no passive-interface GigabitEthernet0/1

! Default route advertisement into OSPF
ip route 0.0.0.0 0.0.0.0 100.64.1.2
router ospf 1
 network 10.0.0.0 0.255.255.255 area 0
 default-information originate

! Tune reference bandwidth (must match across the entire OSPF domain)
router ospf 1
 auto-cost reference-bandwidth 10000

! Manual interface cost override
interface GigabitEthernet0/1
 ip ospf cost 50

! Hello/dead timer tuning (must match the neighbor)
interface GigabitEthernet0/1
 ip ospf hello-interval 5
 ip ospf dead-interval 20

! DR election control — exclude from DR/BDR, or bias toward winning
interface GigabitEthernet0/1
 ip ospf priority 0
interface GigabitEthernet0/2
 ip ospf priority 100

! Force network type when default doesn't fit the design
interface GigabitEthernet0/1
 ip ospf network point-to-point
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Static `router-id` on every OSPF router | Predictable RIDs; no churn when the source interface changes or the process restarts | Throwaway lab topologies | ENCOR 350-401 OCG |
| `passive-interface default`, then selectively un-passive transit links | No hellos or adjacencies on access/user segments while still advertising the prefix | Small networks where nearly every interface is transit | Cisco IOS hardening guide (doc 13608) |
| `ip ospf network point-to-point` on routed point-to-point links | Skips a pointless DR/BDR election; faster adjacency and convergence | Segments that genuinely have more than two routers | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| `auto-cost reference-bandwidth` raised identically domain-wide | Keeps Gig/10G links distinguishable in SPF (default 100 Mbps reference makes them all cost 1) | Brownfield where a partial rollout would skew costs worse than the default | ENCOR 350-401 OCG |
| OSPF authentication on all adjacencies | Blocks rogue or accidental adjacencies | Isolated out-of-band or lab segments | Cisco IOS hardening guide (doc 13608) |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip ospf interface [brief \| <id>]` | OSPF-enabled interfaces, area, cost, current DR/BDR/DROTHER state, neighbor counts (full/2-way) |
| `show ip ospf neighbor [detail]` | Neighbor ID (RID), priority, state (e.g. `FULL/BDR`, `2WAY/DROTHER`), dead time countdown, address/interface |
| `show ip route ospf` | Installed OSPF routes — `[110/metric]`, next hop, outbound interface |
| `show ip ospf database router` | Per-router LSA detail — useful for confirming exactly what prefix/mask a router is advertising (e.g. confirming loopback /32 vs configured mask) |
| `clear ip ospf process` | Restarts the OSPF process — required to pick up a new static RID, or to force DR/BDR re-election after a priority change |

## Intent Questions
- Which routers should be adjacent, on which interfaces, in which area?
- Who is supposed to be DR on each multi-access segment — and does the design actually care?
- Which prefixes should appear in the RIB via OSPF, learned from where?
- What is supposed to be passive, stub, or filtered on purpose?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1/2: confirm the physical link is up and both ends are in the same broadcast domain (`show interfaces`) before assuming an OSPF problem.
2. Layer 3 adjacency stuck below Full: `show ip ospf neighbor` — note exactly which state it's stuck at (Init = one-way hello only; 2-Way on a DR segment without election yet; ExStart/Exchange = MTU mismatch is the classic cause; Loading = LSA download still in progress).
3. No neighbor showing up at all: confirm matching subnet/mask (except point-to-point or virtual-link interfaces), matching area ID, and that the interface isn't accidentally passive (`passive-interface` blocks hello transmission/processing entirely while still advertising the prefix).
4. Config error: MTU mismatch between neighbors — OSPF doesn't fragment, so this specifically stalls adjacency in ExStart/Exchange rather than blocking it outright at Init.
5. Config error: hello/dead timer mismatch, or dead interval not set higher than hello interval — prevents adjacency or causes premature neighbor-down declarations.
6. Config error: wrong network statement wildcard/area, or interface-specific config conflicting with a network statement (interface-specific wins when both apply) — check `show ip ospf interface brief` for the actual enabled interfaces/areas versus intended.
7. DR/BDR placement not matching design intent: remember roles don't preempt — a priority change alone won't move an already-elected DR/BDR; `clear ip ospf process` is required to force re-election.
8. Reference bandwidth inconsistency: if path selection looks wrong on high-speed links, check `auto-cost reference-bandwidth` matches across every OSPF router in the domain — a mismatch breaks the assumption that cost ratios are consistent network-wide.
9. Software/platform bug (rare) — only after adjacency requirements, network type, timers, and cost configuration are all confirmed correct.

## Common Pitfalls
- Assuming the `network` statement advertises routes — it only selects which interfaces run OSPF and their area; the resulting LSA is what actually advertises the prefix.
- Expecting DR/BDR roles to preempt when a higher-priority router joins later — they never do; only a DR/BDR failure or process restart triggers re-election, so manual priority changes need `clear ip ospf process` to take effect immediately.
- Forgetting that a router's RID is fixed at process startup and won't change just because the interface that originally provided it (e.g., a loopback) changes — only a process restart re-evaluates it, which is why setting a static RID is recommended for stability.
- Treating OSPF point-to-point network type loopback advertisement as a bug when a /24-configured loopback shows up as /32 in the LSDB — that's expected default Loopback network type behavior; use `ip ospf network point-to-point` on the loopback if the full configured prefix needs to be advertised instead.
- Forgetting `auto-cost reference-bandwidth` must be changed on every router in the OSPF domain, not just one — an inconsistent reference bandwidth breaks the assumption that cost accurately reflects relative link speed across the whole topology.
- Mistaking an MTU mismatch (which stalls adjacency specifically in ExStart/Exchange) for a more generic connectivity problem — the neighbor state itself tells you which adjacency requirement is failing.
