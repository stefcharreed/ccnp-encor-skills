---
name: ccnp-eigrp
description: >
  Use this skill when troubleshooting or configuring eigrp on IOS-XE.
  Invoke when the user asks about: EIGRP, DUAL, successor, feasible
  successor, feasibility condition, EIGRP topology table, wide metrics,
  EIGRP query, EIGRP summarization, variance.
---

## Purpose
EIGRP (an enhanced distance vector protocol using DUAL) precalculates loop-free backup paths and converges fast by querying only as far into the network as needed, instead of recalculating from scratch on every topology change.

## Key Concepts
- An EIGRP **autonomous system (AS)** is a routing domain — not the same concept as a BGP AS. A router can run multiple EIGRP processes, each tied to its own AS, with its own independent topology table; routes don't cross between AS numbers by default even on a router participating in both.
- The **EIGRP topology table** (distinct from the RIB) is what makes EIGRP more than a "true" distance vector protocol — it holds every advertised prefix, which neighbors advertised it, their reported metrics, and the raw values (bandwidth, delay, load, reliability) used to compute those metrics. This is what lets DUAL precompute backup paths instead of only knowing the single best path.
- Core DUAL terms: **successor route** — the lowest-metric path to a destination; **successor** — the next-hop router for that route; **feasible distance (FD)** — the locally calculated metric for the best path; **reported distance (RD)** — the metric the advertising neighbor calculated for itself (becomes the FD from that neighbor's perspective); **feasibility condition** — a candidate backup route's RD must be lower than the local FD, guaranteeing it can't loop back through the current best path; **feasible successor** — any route that satisfies the feasibility condition, kept as an instantly-usable backup.
- EIGRP exchanges full routing tables only at adjacency formation, then sends incremental updates only when topology actually changes — true distance-vector style advertisement, but layered with hello-based neighbor relationships like link-state protocols (this hybrid behavior is why EIGRP is called "enhanced" distance vector).
- IP protocol number 88, multicast group 224.0.0.10 used where possible, unicast only when necessary.
- Route states: **Passive (P)** = stable, no active recomputation; **Active (A)** = DUAL is actively recalculating this prefix because the successor was lost and no feasible successor exists.
- **Classic metric formula** uses K-values to weight bandwidth, load, delay, and reliability — default K1=1, K3=1, K2=K4=K5=0, which collapses the full formula down to `Metric = 256 * (10^7 / Min.Bandwidth(kbps) + Total Delay(tens of µs))`. Min. Bandwidth is the slowest link in the path; Total Delay accumulates hop by hop.
- **Wide metrics** scale the classic formula by 65,536 (vs. 256 for classic) to support interface speeds up to 655 Tbps without losing precision between very fast links (classic metrics can't distinguish an 11 Gbps link from a 50 Gbps one). Wide metrics measure latency in picoseconds instead of delay in tens of microseconds, and add a K6 value for extended attributes (jitter, energy, etc.).
- Metric backward compatibility: as long as K1–K5 match between classic and wide-metric routers and K6 is unset, the two metric styles can still form an adjacency — EIGRP detects a classic-metric peer and "unscales" the metric to interoperate.
- **ECMP**: multiple successor routes with the same metric all get installed, load-sharing evenly. **Unequal-cost load balancing** (EIGRP-specific, not default-enabled): controlled by the **variance multiplier** — any feasible successor whose FD is below (local FD × variance) also gets installed, with traffic split proportionally via the traffic share count shown in `show ip route`. The variance multiplier itself is the feasible successor's metric divided by the successor's metric, rounded up to a whole number.
- **Hello/hold timers**: hello every 5s by default (60s on T1-or-slower interfaces); hold time defaults to 3× the hello interval (15s default, 180s on slow interfaces) — resets on every received hello, and reaching 0 declares the neighbor down and triggers DUAL recomputation for any prefix where that neighbor was the successor.
- **Convergence on neighbor loss**: if a feasible successor exists, it's promoted to successor instantly (no recomputation needed) and the router advertises the new metric via an update packet — downstream routers then run their own DUAL against the new info, which can itself change their successor/feasible successor. If no feasible successor exists, the prefix goes Active and the router queries all EIGRP neighbors for alternate paths.
- **EIGRP summarization** is configured per-interface (not under the routing process) — once a summary range is applied, component routes within that range are suppressed and only the summary is advertised; the summary itself isn't advertised until at least one component route exists. Summarization both shrinks routing tables and creates a query boundary, limiting how far an Active-state query has to propagate during convergence.

## Procedure
Query/reply convergence when no feasible successor exists (example: R2 loses successor for 10.1.1.0/24, queries R3 and R4):
1. R2 detects the link failure, finds no feasible successor for the prefix, marks the route Active, and sends query packets (with delay set to infinity) to all its EIGRP neighbors for that prefix — here, R3 and R4.
2. R3 receives the query, has no route to the prefix at all, and replies to R2 immediately saying so.
3. R4 receives the query; since the query came from its successor (R2) and R4 has no feasible successor either, R4 marks the prefix Active and sends its own query downstream (to R5).
4. R5 receives R4's query; since the query did not come from R5's own successor and R5 has a feasible successor on a different interface, R5 replies to R4 with its EIGRP attributes for the prefix (no need to go Active itself).
5. R4 receives R5's reply — its last outstanding query — computes its new best path, marks the prefix Passive again, and replies to R2 with its new EIGRP metrics.
6. R2 receives R4's reply — its last outstanding query — computes its new best path and marks the prefix Passive.

A query boundary forms wherever a router answers a query without going Active itself (no route to the prefix, or it answers from a non-successor without needing to recompute) — this is what keeps a topology change from cascading through the entire AS.

## Reference Tables
EIGRP terminology, referenced against a topology where R1 reaches 10.4.4.0/24 via R3 (path metric 3328) with R1→R4 as a feasible successor (path metric 5376, RD 2816):

| Term | Definition |
|---|---|
| Successor route | The route with the lowest path metric to reach a destination |
| Successor | The first next-hop router for the successor route |
| Feasible distance (FD) | The locally calculated metric value for the lowest-metric path to a destination |
| Reported distance (RD) | The distance reported by the advertising neighbor — that neighbor's own FD for the route |
| Feasibility condition | A backup route's RD must be less than the locally calculated FD — guarantees a loop-free path |
| Feasible successor | A route that satisfies the feasibility condition; retained as an instant-use backup route |

EIGRP packet types:

| Opcode | Packet Type | Function |
|---|---|---|
| 1 | Update | Transmits routing/reachability information to EIGRP neighbors |
| 2 | Request | Requests specific information from one or more neighbors |
| 3 | Query | Sent to search for an alternate path during convergence |
| 4 | Reply | Sent in response to a query packet |
| 5 | Hello | Discovers EIGRP neighbors and detects when a neighbor is no longer available |

Default EIGRP classic-metric values by interface type (default K1=1, K3=1; K2=K4=K5=0):

| Interface Type | Link Speed (kbps) | Delay | Metric |
|---|---|---|---|
| Serial | 64 | 20,000 µs | 40,512,000 |
| T1 | 1,544 | 20,000 µs | 2,170,031 |
| Ethernet | 10,000 | 1,000 µs | 281,600 |
| FastEthernet | 100,000 | 100 µs | 28,160 |
| GigabitEthernet | 1,000,000 | 10 µs | 2,816 |
| 10 GigabitEthernet | 10,000,000 | 10 µs | 512 |

## Config Patterns
```ios-xe
! Basic EIGRP process with classic metrics (AS 100)
router eigrp 100
 network 10.0.0.0 0.255.255.255
 eigrp router-id 192.168.1.1

! Switch to EIGRP wide metrics (required on high-speed interfaces, e.g. 10G+)
router eigrp 100
 metric version 64-bit

! Tune hello/hold timers (must match neighbor or risk instability)
interface GigabitEthernet0/1
 ip hello-interval eigrp 100 5
 ip hold-time eigrp 100 15

! Unequal-cost load balancing — variance multiplier of 2
router eigrp 100
 variance 2

! Per-interface route summarization
interface GigabitEthernet0/2
 ip summary-address eigrp 100 172.16.0.0 255.255.0.0
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip eigrp topology` | Per-prefix entries: P (Passive/stable) vs A (Active/recomputing), successor count, FD, and each candidate path's metric/RD pair — confirms which routes have a feasible successor backup |
| `show ip eigrp topology <prefix>` (or `all-links`) | Detailed view of one prefix including paths that didn't pass the feasibility condition |
| `show ip eigrp neighbors` | Neighbor adjacency table — confirm expected neighbors are up, check hold time countdown for flapping suspicion |
| `show ip route eigrp` | Installed EIGRP routes — `[AD/metric]`, and multiple `via` lines indicate ECMP or unequal-cost load balancing |
| `show ip route <prefix>` | Full descriptor block per path — traffic share count (ratio for unequal-cost load balancing), total delay, minimum bandwidth, reliability, hop count |
| `show ip protocols` | Configured K-values, metric version (classic vs wide), variance, and summarization state |

## Troubleshooting Checklist
1. Layer 1/2: confirm the interface toward the expected neighbor is up and the link isn't flapping — `show interfaces` and `show ip eigrp neighbors` hold-time countdown.
2. Layer 3 adjacency: `show ip eigrp neighbors` — missing expected neighbor means hellos aren't being exchanged; check AS number match, K-value match, and that both sides are in the `network` statement's range (or otherwise enabled on that interface).
3. Route stuck Active / slow convergence: `show ip eigrp topology` — a prefix stuck in Active (sometimes reported as SIA, Stuck-In-Active) means a query never got a reply; trace the query path toward the non-responding neighbor.
4. No feasible successor where one was expected: check whether a path's RD is actually lower than the local FD — a path can have a worse (higher) metric than the successor and still fail the feasibility condition, meaning it requires a full DUAL recomputation instead of an instant failover.
5. Config error: K-value mismatch between neighbors (including classic vs. wide metric mismatch beyond K1–K5) — prevents adjacency from forming at all.
6. Config error: summarization configured under the wrong interface, or the summary route not advertising because no component route currently falls within its range.
7. Config error: variance not configured (or set to default 1) when unequal-cost load balancing was expected — `show ip route <prefix>` will show only the single successor path.
8. Software/platform bug (rare) — only after adjacency state, topology table entries, and metric/summarization config are all confirmed correct.

## Common Pitfalls
- Assuming a backup path with a *better* (lower) metric than the successor always becomes a feasible successor — feasibility is judged purely by the feasibility condition (RD < local FD), not by metric ranking; a numerically worse path can pass while a numerically better one fails.
- Forgetting EIGRP summarization is per-interface, not under the `router eigrp` process — looking for it in the wrong configuration context wastes troubleshooting time.
- Mixing classic and wide metrics without checking K1–K5 alignment — adjacency silently fails to form rather than erroring obviously; always verify `show ip protocols` K-values on both sides.
- Treating Active (A) state in `show ip eigrp topology` as inherently a problem — it's the expected, normal state *during* convergence; it's only a problem if a prefix stays Active far longer than expected (SIA).
- Confusing ECMP (automatic, same metric, default-enabled) with EIGRP's unequal-cost load balancing (manual, different metrics, requires `variance` > 1) — checking `show ip route` without realizing variance is still at its default of 1 will make unequal-cost paths look like they're "not working" when they were simply never enabled.
