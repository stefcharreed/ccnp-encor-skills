---
name: ccnp-multicast
description: >
  Use this skill when troubleshooting or configuring multicast on IOS-XE.
  Invoke when the user asks about: IP multicast, IGMP, IGMPv2, IGMPv3, IGMP
  snooping, PIM, PIM dense mode, PIM sparse mode, rendezvous point, RP,
  Auto-RP, BSR, RPF, (S,G), (*,G), multicast distribution trees.
---

## Purpose
Multicast lets one source send a single stream to many interested receivers
(one-to-many) without the bandwidth cost of unicast-per-receiver or the
flooding cost of broadcast — IGMP handles receiver-to-router signaling at
Layer 2/3, and PIM routes the stream across the network to wherever those
receivers are.

## Key Concepts
- **Three transmission models:** unicast (one-to-one), broadcast (one-to-all,
  not recommended — requires directed broadcast which is a DDoS vector, and
  burdens every NIC on the segment), multicast (one-to-many, replicated only
  where the tree forks).
- **IGMP vs PIM split:** IGMP operates between receivers and their local
  (last-hop) router — Layer 2/3 boundary. PIM operates router-to-router to
  build the distribution tree from source to receivers across Layer 3.
- **Multicast addressing (224.0.0.0/4, IANA):** key blocks — local network
  control (224.0.0.0/24, TTL 1, never forwarded — e.g. 224.0.0.5 OSPF,
  224.0.0.10 EIGRP, 224.0.0.13 all PIM routers), internetwork control
  (224.0.1.0/24 — e.g. 224.0.1.39/.40 Auto-RP), SSM block (232.0.0.0/8),
  GLOP block (233.0.0.0/8, 16-bit ASN mapped into octets 2-3), and the
  administratively-scoped/organization-local block (239.0.0.0/8 — like
  RFC 1918 for multicast, free to reuse internally).
- **Multicast MAC mapping:** first 24 bits always `01:00:5E`, 25th bit
  always 0, low-order 23 bits copied directly from the low-order 23 bits of
  the multicast IP. Because the IP has 28 host-significant bits but only 23
  map across, **32 different multicast IP addresses can collide on the same
  multicast MAC** (e.g. 239.255.1.1 and 239.127.1.1 both map to
  01:00:5E:7F:01:01) — a real source of unexpected traffic at the NIC level.
- **The I/G bit (individual/group), a.k.a. the unicast/multicast bit:** the
  **least significant bit of the first byte** of the destination MAC.
  `0` = individual (unicast), `1` = group (multicast or broadcast).
  **Shortcut: read the first byte in hex — odd = group, even = unicast.**
  `01:00:5E:...` odd → multicast · `FF:FF:FF:FF:FF:FF` odd → broadcast ·
  `00:12:34:...` even → unicast. **It sits there because Ethernet transmits
  each byte least-significant-bit first, so the I/G bit is literally the
  first bit on the wire** — a NIC can tell "is this for a group?" before the
  other 47 bits arrive.
- ⚠️ **"Last bit of the FIRST byte" — not the last byte.** This is the most
  common way to misread it. The I/G bit is **bit 8 of the 48**, near the
  front of the address, not bit 48 at the end:

```
MAC:   01  :  00  :  5E  :  7F  :  01  :  01
       ^^                                ^^
    byte 1                            byte 6

byte 1 = 01 = 0000 000[1]   <- I/G bit, bit 8 of 48
byte 6 = 01 = 0000 000[1]   <- bit 48, no special meaning at all
```

  **"Least significant" is always scoped to something** — here it means
  least significant *within byte 1*, so you look at that one byte and take
  its rightmost bit. Same scope discipline as the "low-order" trap above.
  **The odd/even shortcut therefore only ever applies to the first byte** —
  the last byte's value is irrelevant to unicast-vs-group.
- **Why bit 8 and not bit 48:** Ethernet sends bytes in order (byte 1 first)
  and sends the bits *within* each byte least-significant-first. Those two
  rules together put the I/G bit **first on the wire**. Had it been placed at
  the end of the last byte, every NIC would have to buffer all 48 bits before
  it could tell whether the frame was even addressed to a group.
- **U/L bit (universal/local)** is its neighbour, the *second* least
  significant bit of byte 1: `0` = globally unique OUI-assigned (burned in),
  `1` = locally administered. **This is the bit that flips when a MAC is
  randomised or spoofed** — a locally-administered MAC on a corporate
  segment is worth a second look.
- ⚠️ **Terminology trap — "low-order" is used at two different scopes one
  sentence apart** (the ENCOR OCG does exactly this and does not flag it):
  the "**low-order bit of the first byte**" is the I/G bit — scope is **one
  byte**. The "**low-order 23 bits**" are the copied group bits — scope is
  the whole **48-bit address**. Same phrase, one bit versus twenty-three.
- **Counting the fixed part two ways, both correct:** "24 bits of `01:00:5E`
  plus a 25th bit of 0" and "the high-order 25 bits are fixed" are the same
  statement. 48 − 25 = the 23 bits that carry the group.
- **Where the discarded bits actually go:** of the 32 IP bits, 9 never make
  the trip — the 4-bit `1110` multicast prefix (carries no group
  information) plus **5 high-order bits of the 28-bit group ID**. Only those
  5 are real information loss, which is exactly why the collision ratio is
  2^5 = 32 and not something larger.
- **IGMP versions:** IGMPv1 (RFC 1112, obsolete) — IGMPv2 (RFC 2236,
  standard) adds leave-group messages and querier election — IGMPv3
  (RFC 3376) adds source filtering (include/exclude mode) and is required
  for SSM. All sent with TTL 1 and the router-alert option so they never
  leave the local segment.
- **IGMPv2 message types:** v2 membership report (0x16, the "IGMP join"),
  v1 membership report (0x12, backward compat), leave group (0x17), general
  query (0x11, dest 224.0.0.1, group field 0.0.0.0, default 10s max response
  time), group-specific query (0x11, sent in response to a leave to confirm
  no other members remain).
- **IGMP querier election:** lowest interface IP on the segment wins; losers
  start a timer that resets on every query heard and re-trigger election
  (after 2x query interval, default 60s) if the querier goes silent.
- **IGMP snooping (RFC 4541):** switch inspects IGMP joins/leaves to learn
  which ports want which multicast group, avoiding flood-to-all-ports for
  multicast traffic (the default switch behavior, since multicast MACs are
  never source MACs and so are always "unknown"). Some groups (224.0.0.0/24)
  are still flooded everywhere regardless.
- **Distribution trees — source tree (SPT):** rooted at the source, state
  notation (S,G) — shortest path, but requires a separate SPT per source.
  **Shared tree (RPT):** rooted at the RP, state notation (*,G) — fewer
  table entries (scales better as sources increase) but receivers get
  traffic from *all* sources sending to that group, which wastes bandwidth
  and is a potential security issue (unintended sources can inject traffic).
- **PIM terminology:** RPF interface (lowest-cost path to source or RP, tie
  broken by highest IP), RPF neighbor (PIM neighbor on that interface),
  upstream/downstream (toward source vs toward receivers), IIF (incoming
  interface, same as RPF interface — only interface multicast can legally
  arrive on), OIF/OIL (outgoing interface / list of them), LHR (last-hop
  router, attached to receivers, sends PIM joins upstream), FHR (first-hop
  router, attached to source, sends register messages to RP), MRIB
  (topology table, aka mroute table — S, G, IIF, OIFs, RPF neighbor), MFIB
  (hardware forwarding table built from MRIB).
- **Five PIM modes:** Dense Mode (PIM-DM), Sparse Mode (PIM-SM), Sparse
  Dense Mode, Source Specific Multicast (PIM-SSM), Bidirectional (Bidir-PIM).
  PIM-DM/PIM-SM are jointly called any-source multicast (ASM). All PIM
  control messages use IP protocol 103, multicast to 224.0.0.13 with TTL 1
  (except register/register-stop, which are unicast).
- **PIM-DM flood-and-prune:** assumes receivers on every subnet. Floods
  traffic everywhere first (including non-RPF interfaces — packets arriving
  non-RPF are discarded), then routers with no interested downstream
  receivers send a prune back toward the source. Prunes expire every 3
  minutes, causing periodic reflooding — not recommended for production.
- **PIM-SM explicit join model:** assumes no one wants the stream until they
  ask. Receiver sends IGMP join to LHR → LHR sends PIM (*,G) join toward the
  RP, building the shared tree. Source's FHR encapsulates multicast data in
  unicast PIM register messages to the RP (via a register tunnel) until the
  RP either sends register-stop (no interested receivers) or builds a native
  (S,G) SPT back to the source and the FHR stops registering.
- **PIM-SM SPT switchover:** by default, as soon as the LHR receives the
  first packet down the shared tree from the RP, it checks the unicast
  table for the best path to the source and sends an (S,G) join directly
  toward the source — switching off the RPT once duplicate (now SPT-sourced)
  traffic arrives, by pruning the RPT path with an (S,G) prune to the RP.
  This switchover happens even if RPT and SPT paths are identical. Can be
  disabled per-group.
- 🔑 **How the LHR ever learns a source — the keystone most texts skip.** The
  `(*,G)` join cannot name a source, because the receiver does not know one.
  **The LHR learns S from the DATA PLANE, not from PIM:** the multicast
  packets arriving down the shared tree are ordinary IP packets, and every
  one carries a source address in its header. First packet arrives → LHR
  reads `src=10.1.1.1` → it can now send an `(S,G)` join. **The asterisk
  describes the join, never the traffic.**
- **Therefore the RPT is a bootstrap, not a path.** The RP's entire job is
  **source discovery** — it exists because receivers do not know sources.
  The moment the first packet lands at the LHR, discovery has *succeeded*
  and the RP's job is done. **That is why the SPT switchover exists at all.**
  ✅ **The proof is PIM-SSM:** IGMPv3 lets the receiver name the source, so
  discovery is unnecessary — and SSM has no RP, no shared tree and no
  Register process. Remove the discovery problem and the RP disappears.
- **"Why bother with the SPT if PIM-SM is built around the RP?"** Because the
  point of sparse mode is **not** the shared tree — it is **explicit join
  only, i.e. do not flood**. Explicit join needs a *known root*; a receiver
  knows no source; so it joins a known root (the RP) as scaffolding.
  **RPT solves discovery; SPT solves delivery.** The decisive argument is
  the third one below:
  1. **Path** — RPT is source→RP→receiver, SPT is source→receiver; via an
     intermediate can never be shorter, at best equal.
  2. **Latency** — the real workloads are live video, voice, market data.
  3. 🚨 **RP as bottleneck** — if every flow stayed on the RPT permanently,
     **all multicast in the domain would funnel through one router**, making
     the RP a data-plane chokepoint and single point of failure, not just a
     control-plane one. **The switchover is what keeps the RP a control-plane
     device.**
  **Cost shape:** PIM-SM front-loads signalling **once per source** to buy a
  near-zero steady state; PIM-DM is cheap to start and then **re-floods the
  whole domain every 3 minutes forever.**
- **The switchover is a TRIGGER, not a comparison.** With `spt-threshold 0`
  the LHR does not evaluate "is the SPT better?" — it switches on the single
  event *"a packet arrived from a source I did not know."* It cannot cheaply
  determine that the trees are identical, so it always switches. When the RP
  already sits on the natural path you therefore pay full `(S,G)` state cost
  on every router along the way **for zero path benefit**. ⚙️ **Design
  consequence: RP placement and `spt-threshold` interact — disable switchover
  precisely when you have placed the RP *well*,** because a well-placed RP
  leaves the SPT nothing to improve.
- ⚠️ **A DR is a PER-SEGMENT ROLE, not a position on a tree.** It is elected
  on every multi-access segment to be the **single** router that acts there,
  preventing duplicate joins (receiver side) and duplicate Registers (source
  side) when two PIM routers share a LAN. **Proof it is not tree-bound: the
  same DR sends the `(*,G)` join and later the `(S,G)` join** — it is the
  router that performs the switchover. On a point-to-point link the election
  still runs but is meaningless; there is no duplication to prevent.
  **FHR/LHR are positions; DR is an elected role.** They usually coincide.
- 🚨 **Two OSPF-shaped traps in the PIM DR:** PIM has **no BDR**, and PIM DR
  election **IS preemptive** — a higher-priority router takes over, where
  OSPF's DR is sticky. Tiebreak is highest **IP**, not RID.
- **DR vs Assert winner — two elections, different planes:** the **DR** is
  proactive and decides who sends **control** messages (joins, registers);
  the **Assert winner** is reactive, fires only when duplicate data actually
  appears on a segment, and decides who **forwards data**. **They can be
  different routers on the same segment.**
- 🔑 **Auto-RP vs BSR in one line: Auto-RP distributes an ANSWER; BSR
  distributes the CANDIDATES and every router computes the answer itself.**
  Auto-RP's Mapping Agent is an *authority* that resolves conflicts and
  publishes a verdict. BSR has no authority over the choice — identical
  RP-set plus identical deterministic hash on every router means they all
  reach the same conclusion independently. **Stateless agreement vs a
  published verdict.**
- 🚨 **Auto-RP has a chicken-and-egg problem; BSR does not.** Auto-RP's own
  announcements are multicast (224.0.1.39/.40), but forwarding sparse-mode
  multicast requires already knowing the RP. Two ways out:
  **`ip pim autorp listener`** — floods **only** .39/.40 dense while every
  other group stays sparse. **This is the correct answer.**
  **`ip pim sparse-dense-mode`** — older and hazardous: the dense fallback is
  **not scoped to Auto-RP**, so *any* group without an RP mapping floods the
  domain. One misconfiguration becomes a flood.
  ✅ **BSR is immune** — Bootstrap messages go to **224.0.0.13**, link-local
  (TTL 1), flooded hop-by-hop router to router. It never needs an RP to
  propagate itself. **This is the strongest practical argument for BSR.**
- ⚠️ **Priority direction INVERTS inside BSR — classic exam bait:**

| Election | Winner |
|---|---|
| **BSR election** (which router becomes BSR) | **Highest** priority |
| **C-RP selection** (which RP serves a group) | **Lowest** priority (like AD) |
| **Auto-RP conflict** | Priority unused — **highest IP** |

- **Hash mask length controls granularity, not preference.** A *longer* mask
  spreads consecutive groups across more RPs; a *shorter* mask clumps them
  onto fewer. Order of evaluation: **C-RP priority first (lower wins), then
  hash value (highest wins), then highest IP.**
- **Static RP beats both — but only with `override`.** By default a
  dynamically learned mapping (Auto-RP or BSR) **wins over** a static
  `ip pim rp-address`; adding `override` flips that precedence. Worth knowing
  before you "fix" a mapping with a static entry and watch it be ignored.
- **Designated Router (DR) election:** on multi-access PIM-SM segments,
  highest DR priority (default 1 on all routers) wins, tiebreak by highest
  IP. FHR-side DR encapsulates register messages; LHR-side DR sends PIM
  joins/prunes upstream and performs SPT switchover. Default DR hold time
  105s (3.5x hello interval).
- **RPF check:** if a multicast packet arrives on the interface a router
  would use to reach the source (SPT state) or the RP (shared-tree state),
  it's accepted and forwarded out the OIL; otherwise it's discarded to
  prevent loops. (S,G) state RPF-checks against the source; no source-state
  (shared-tree) RPF-checks against the RP.
- **PIM Assert / forwarder election:** when two routers both receive the
  same (S,G) traffic via their RPF interfaces and both forward it onto the
  same multi-access LAN (duplicate flow — common with PIM-DM, possible with
  PIM-SM under routing inconsistencies), they send assert messages carrying
  AD + metric back to the source/RP. Lowest AD wins, tie → lowest metric,
  tie → highest IP. Loser prunes its interface; if the winner goes down, the
  loser's prune times out (3 min) and it resumes forwarding.
- **Rendezvous Point (RP) — mandatory for PIM-SM:** single common root of
  the shared tree, configured statically or learned dynamically. **Static
  RP** — simplest for small/stable networks, but every router needs matching
  config and there's no automatic failover. **Auto-RP** (Cisco proprietary)
  — candidate RPs (C-RPs) advertise via RP announcements to 224.0.1.39 every
  60s (highest IP wins on conflict); RP mapping agents (MAs) listen, build a
  group-to-RP mapping cache, and re-advertise it to 224.0.1.40, which all
  PIM routers join to learn RP assignments — supports multiple RPs serving
  different group ranges and built-in failover. **BSR** (RFC 5059, IETF
  standard, part of PIM v2) — candidate BSRs (C-BSRs) elect one BSR (highest
  priority, tie → highest IP); C-RPs unicast advertisements to the BSR; the
  BSR floods the full RP set (group range, RP priority/address, hash mask
  length, SM/Bidir flag) to all routers via Bootstrap messages; each router
  independently runs the same hash algorithm against the same C-RP list to
  pick the active RP per group range, rather than the BSR electing a winner
  itself. **BSR and Auto-RP should not run together** — not designed to
  interoperate.

## Config Patterns
```ios-xe
! --- Enable multicast routing globally (required before any PIM config) ---
ip multicast-routing

! --- Enable PIM sparse mode on an interface (also enables IGMP) ---
interface GigabitEthernet0/1
 ip pim sparse-mode

! --- Enable PIM dense mode (legacy/lab use only) ---
interface GigabitEthernet0/1
 ip pim dense-mode

! --- Static RP, applied on every PIM router in the domain ---
ip pim rp-address 10.255.255.1

! --- Static RP scoped to a specific group range via ACL ---
ip access-list standard MCAST-GROUPS
 permit 239.1.1.0 0.0.0.255
ip pim rp-address 10.255.255.1 MCAST-GROUPS

! --- Auto-RP: configure a router as candidate RP ---
ip pim send-rp-announce Loopback0 scope 16 group-list MCAST-GROUPS

! --- Auto-RP: configure a router as mapping agent ---
ip pim send-rp-discovery Loopback0 scope 16

! --- BSR: configure a candidate RP ---
ip pim rp-candidate Loopback0 group-list MCAST-GROUPS priority 0

! --- BSR: configure a candidate BSR ---
ip pim bsr-candidate Loopback0 priority 0

! --- Enable IGMPv3 on an LHR-facing interface (required for SSM) ---
interface GigabitEthernet0/2
 ip pim sparse-mode
 ip igmp version 3

! --- Disable PIM SPT switchover for all groups (stay on shared tree) ---
ip pim spt-threshold infinity

! --- IGMP static join for testing without a real receiver ---
interface GigabitEthernet0/2
 ip igmp join-group 239.1.1.1
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| PIM sparse mode in production, never dense | Dense mode flood-and-prunes the entire network every ~3 minutes | Tiny lab demos of flood/prune behavior | ENCOR 350-401 OCG |
| RP placement and discovery method (static, Auto-RP, BSR) chosen deliberately and consistent domain-wide | RP disagreement means silent join failures — receivers just never get the stream | — | ENCOR 350-401 OCG |
| IGMP snooping left enabled on switches | Without it, every multicast frame floods the whole VLAN | Legacy hosts with broken IGMP implementations | ENCOR 350-401 OCG |
| Multicast topology congruent with unicast unless deliberately engineered | RPF checks follow the unicast table; incongruence silently drops traffic | Deliberate separate multicast topologies via static mroutes/mBGP | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip mroute` | (S,G) and (*,G) entries, incoming interface, OIL — confirms forwarding state exists |
| `show ip mroute <group>` | Detailed state for one group, flags (D=dense, S=sparse, J=SPT-join, T=SPT-flag, etc.) |
| `show ip pim neighbor` | Confirms PIM adjacency formed on each interface, DR column shows elected DR |
| `show ip pim rp mapping` | Group-to-RP mappings learned via static/Auto-RP/BSR |
| `show ip pim rp <group>` | Which RP is active for a specific group, and how it was learned |
| `show ip igmp groups` | Groups with active receivers on directly connected interfaces |
| `show ip igmp interface` | IGMP version, querier address, query interval per interface |
| `show ip mfib <group>` | Hardware forwarding state — compare against `show ip mroute` for MRIB/MFIB consistency |
| `show ip pim interface` | PIM mode (dense/sparse), DR, neighbor count per interface |

## Intent Questions
- Which groups are supposed to exist, sourced from where, received where?
- Where is the RP supposed to be, and how is every router supposed to learn about it?
- Should the multicast topology follow unicast (default RPF), or is it deliberately different here?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Confirm `ip multicast-routing` is enabled globally — nothing works
   without it.
2. Confirm `ip pim sparse-mode` (or dense-mode) is on every transit
   interface, including the RP-facing and receiver-facing interfaces — a
   missing PIM-enabled interface breaks the tree at that hop.
3. Check `show ip pim neighbor` — no neighbor means no PIM adjacency, which
   means no joins can propagate upstream.
4. For PIM-SM, confirm RP reachability and that `show ip pim rp mapping`
   agrees on every router — a static RP typo or missing group-list ACL line
   silently excludes a group range.
5. Check RPF: `show ip route` toward the source (or RP) must match the
   interface multicast traffic is actually arriving on, or it gets silently
   discarded as a non-RPF packet — this is the single most common
   "multicast just doesn't flow" cause.
6. If receivers exist but no traffic flows, check `show ip igmp groups` on
   the LHR — confirm the IGMP join was actually received and the group/
   version matches what the source expects (SSM requires IGMPv3).
7. If duplicate multicast traffic appears on a LAN segment, look for a PIM
   assert in progress (`show ip mroute` flags) — usually indicates an
   inconsistent RPF neighbor selection from mismatched IGP/route metrics.
8. If using PIM-DM, remember prunes expire every 3 minutes — periodic
   reflood-then-reprune is expected behavior, not a fault, unless it's
   happening on every subnet (no receivers anywhere = wrong mode choice).
9. If BSR and Auto-RP are both configured in the same domain, stop —
   they're not designed to coexist and will produce inconsistent RP
   mappings.
10. Check `ip pim spt-threshold` if a receiver seems "stuck" on the shared
    tree path through the RP when a shorter SPT should have taken over.

## Common Pitfalls
- Assuming broadcast video/data feeds are an efficient alternative to
  multicast — they require IP directed broadcasts (disabled by default,
  and a DDoS exposure if enabled) and burden every NIC on the segment.
- Forgetting that 32 different multicast group IPs can map to the same
  multicast MAC address (only 23 of 28 host bits transfer) — can cause a
  receiver to process traffic for a group it never joined.
- Treating PIM-DM as production-ready — it's flood-and-prune by design,
  reverts to flooding every 3 minutes, and only makes sense when receivers
  are genuinely on every subnet (rare).
- Forgetting that PIM-SM is mandatory-RP — no RP configured (static,
  Auto-RP, or BSR) means PIM-SM simply doesn't work, unlike PIM-DM.
- Running Auto-RP and BSR simultaneously in the same PIM domain — they
  weren't designed to interoperate and will conflict.
- Forgetting `ip igmp version 3` on receiver-facing interfaces when SSM is
  required — IGMPv2 doesn't support source filtering.
- Confusing RPF interface selection with normal unicast best-path thinking
  — RPF is checked against the *source* (or RP for shared-tree state), not
  the destination, and packets arriving on the "wrong" interface are
  dropped even if that interface has a perfectly valid unicast route.
- Expecting the SPT switchover to be optional/manual by default — on Cisco
  routers it happens automatically after the very first packet from the RP,
  even when the SPT path is identical to the RPT path.
