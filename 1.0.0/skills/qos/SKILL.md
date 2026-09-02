---
name: ccnp-qos
description: >
  Use this skill when troubleshooting or configuring QoS on IOS-XE.
  Invoke when the user asks about: QoS, DSCP, DiffServ, IntServ, PHB, EF,
  AF, class selector, CoS, 802.1p, ToS byte, IP Precedence, MQC, class-map,
  policy-map, service-policy, NBAR2, trust boundary, policing, shaping,
  token bucket, CIR, Bc, Be, PIR, Tc, srTCM, trTCM, markdown, LLQ, CBWFQ,
  priority queue, bandwidth remaining, WRED, RED, tail drop, ECN,
  queue-limit, random-detect, TCP global synchronization.
---

## Purpose
QoS decides who loses when a link is oversubscribed — it classifies traffic
into aggregates, marks each aggregate so every downstream hop can act on it
without re-inspecting the packet, and then applies bandwidth, delay, jitter
and loss treatment per class. **QoS never creates bandwidth; it only chooses
what gets hurt first.**

## Key Concepts

### The three models
- **Best Effort:** no QoS at all — a single FIFO queue, all traffic equal.
  The default on every interface until you apply a policy.
- **IntServ (RFC 1633):** per-flow bandwidth *reservation* signalled with
  RSVP before the flow starts. A genuine guarantee, but the network holds
  **per-flow state at every hop**, so it does not scale. Think circuit
  emulation on a packet network.
- **DiffServ (RFC 2475):** classify and mark at the **edge**, then every hop
  in the core acts on the mark alone via a **per-hop behavior (PHB)**. No
  per-flow state, no signalling — scales to the whole internet, which is why
  it is what you will actually configure.

> **The tradeoff in one line:** IntServ guarantees a flow and cannot scale;
> DiffServ scales and guarantees only a *class*, not any single flow.

### MQC — the three-object model
Every IOS-XE QoS policy is the same three objects, and they are always used
in this order:
1. **`class-map`** — *what is this traffic?* (classification)
2. **`policy-map`** — *what do I do to it?* (marking, policing, shaping, queuing)
3. **`service-policy`** — *where does it apply?* (interface, direction)

- A `class-map` is **`match-all` by default** (logical AND). `match-any` is
  logical OR and must be stated explicitly. **This is a very common config
  bug** — a `match-all` class with two mutually exclusive matches silently
  matches nothing.
- **`class-default`** catches everything not matched by a user class. It
  always exists, even when you do not type it. Unclassified traffic lands
  here, so **whatever you do (or fail to do) to `class-default` is your
  policy for the majority of traffic.**
- Policy maps can be **nested** (`service-policy` under a class) to build
  hierarchical policies — the basis of shape-then-queue (see Config Patterns).

### Classification
Traffic descriptors available, roughly by layer:
- **Internal:** `qos-group` (locally significant, never leaves the device)
- **Layer 1:** incoming interface / subinterface
- **Layer 2:** source/destination MAC, **CoS (802.1p)**
- **Layer 2.5:** MPLS EXP bits
- **Layer 3:** **DSCP**, IP Precedence, source/destination IP, ACLs
- **Layer 4:** TCP/UDP port numbers
- **Layer 7:** **NBAR2** — deep packet inspection that recognises the
  application rather than the port, which is the only reliable way to handle
  applications that hop ports or ride inside HTTPS.

### Marking
- **CoS / 802.1p** lives in the **802.1Q tag**, so it exists *only on trunk
  links*. The Tag Control Information (TCI) field is
  **PCP (3 bits, CoS 0-7) + DEI/CFI (1 bit) + VLAN ID (12 bits)**.
  🚨 **CoS is destroyed the moment a frame leaves a trunk** (access port,
  or any routed hop). It cannot survive end-to-end — which is precisely why
  DSCP exists.
- **The ToS byte** is the second byte of the IPv4 header (Traffic Class in
  IPv6). Originally 3 bits of **IP Precedence**; redefined by RFC 2474 as
  **6 bits DSCP + 2 bits ECN**.
- **DSCP survives end to end** because it lives in the IP header — it is the
  only marking that crosses routed boundaries intact.
- **PHB** is what a hop *does* with a mark. The mark is a label, not a
  behavior — **a DSCP value does nothing until some device is configured to
  act on it.** This is the single most misunderstood point in QoS.

### Policing vs shaping
- **Policer:** meters traffic and, for anything over the rate, **drops or
  re-marks it immediately.** Works **ingress or egress**. No buffering, so
  no added delay — but dropping TCP causes **retransmissions**.
- **Shaper:** **buffers and delays** traffic above the rate until the rate
  falls back under. **Egress only.** Fewer TCP retransmissions, but it adds
  **delay and jitter**, and it needs memory for the buffer.

> **Choose by what the traffic can tolerate:** TCP bulk data prefers shaping
> (delay is cheap, retransmission is expensive). Real-time voice prefers
> policing (a dropped packet is gone anyway; a delayed one is worse than
> useless).

- **Placement:** policers **inbound at the network edge**, so junk never
  wastes core bandwidth. Shapers **outbound on SP-facing interfaces**, to
  stay under a contracted rate the SP is policing — or under an SLA whose
  violation costs money.
- **Markdown:** rather than dropping excess, re-mark it *down* within its AF
  class (AFx1 → AFx2, or → AFx3 with three-color policing), then let **WRED
  downstream drop AFx3 more aggressively than AFx2, and AFx2 more than
  AFx1.** Markdown only works if the rest of the network is configured to
  respect it — otherwise you have relabelled traffic and changed nothing.

### Token buckets
- **Token = 1 byte = 8 bits.** A packet takes tokens equal to its size.
- **CIR** — the policed rate in bps. **Bc** — bucket size in bytes, the most
  that can be sent in one **Tc**. **Tc** — the interval in ms.
- Tokens refill at the CIR. **A full bucket discards new tokens** — they are
  not banked for later.
- ⚠️ **Bc must be ≥ the largest possible IP packet.** If it is not, large
  packets can *never* accumulate enough tokens and will exceed the rate
  100% of the time — a policy that silently drops all big packets.
- **Recommended Tc is 8–125 ms.** Use **8–10 ms for voice**: a long Tc means
  the whole Bc is transmitted in a burst at line rate and then the link goes
  silent for the rest of the interval, which is exactly the interpacket
  delay that wrecks real-time traffic.

### Congestion management (queuing)
- **Congestion is detected when the Tx-ring (TxQ) — the Layer 1 hardware
  queue — is full.** Queuing activates then and deactivates when it drains.
  **Scheduling, by contrast, is always running**, congestion or not.
- Two causes only: **speed mismatch** (fast ingress → slow egress) and
  **aggregation** (many ingress → one egress).
- **CBWFQ:** up to 256 queues, each with a **minimum bandwidth guarantee**
  during congestion. **No latency guarantee** — fine for data, not for voice.
- **LLQ = CBWFQ + a strict-priority queue.** The PQ is serviced first, and
  is **policed during congestion** so it cannot starve the other classes the
  way legacy PQ did.
- 🚨 **An unpoliced priority queue starves everything else.** Always pair
  `priority` with an explicit or conditional policing rate.

### Congestion avoidance (dropping)
- **Tail drop** is the default: when the buffer is full, everything arriving
  is dropped regardless of class.
- **TCP global synchronization** is why that is bad — many TCP senders on a
  link back off *simultaneously*, the link goes underutilised, they all ramp
  up *simultaneously*, congest it again, and the sawtooth repeats forever.
- **RED** drops random packets *before* the buffer fills, desynchronising
  the senders.
- **WRED** is Cisco's RED weighted by **IPP or DSCP** — lower values dropped
  more aggressively. `random-detect dscp-based` is the recommended form when
  you classify with DSCP; **precedence-based is the default if you do not
  say otherwise.**
- **ECN** extends WRED: instead of dropping, *mark* the ECN bits so the
  endpoints slow down without losing a packet.

## Procedure

**Applying any MQC policy (the order never changes):**
1. `class-map` — define what the traffic is (match on DSCP, ACL, NBAR2, …).
2. `policy-map` — under each `class`, define the action (set / police /
   shape / priority / bandwidth / random-detect).
3. `service-policy input|output` on the interface — attach it and pick the
   direction.
4. Verify with `show policy-map interface` — **counters, not config**.

**Single-rate two-color policer (one bucket):**
1. Tokens accumulate in the Bc bucket at the CIR.
2. Packet arrives; compare its size (in tokens) to tokens available.
3. Enough tokens → **conform** → transmit (or re-mark and transmit).
4. Not enough → **exceed** → drop, or mark down and transmit.

**srTCM — single-rate three-color, RFC 2697 (two buckets, one rate):**
1. Tokens fill the **Bc** bucket at the CIR.
2. **Leftover** tokens at the end of each interval **overflow into the Be
   bucket** instead of being discarded. Be is fed only by Bc's spillover.
3. `B < Tc` (enough in Bc) → **conform (green)** → transmit or re-mark.
4. Else `B < Te` (enough in Be) → **exceed (yellow)** → drop, or mark down.
5. Else → **violate (red)** → usually drop, optionally mark down.

**trTCM — two-rate three-color, RFC 2698 (two buckets, two rates):**
1. **Bc is filled at the CIR; Be is filled independently at the PIR.** No
   spillover between them — this is the structural difference from srTCM.
2. 🚨 **The check order is REVERSED:** `B > Tp` (over PIR) → **violate**
   first.
3. Else `B > Tc` (over CIR) → **exceed**.
4. Else → **conform**.

> **Why trTCM exists:** srTCM's excess capacity depends on *random leftover
> tokens*, so its burst behavior is unpredictable. trTCM's PIR is a
> **sustained, configured** second rate — you can tell a customer exactly
> what their peak is, and drop violations at a defined rate.

## Reference Tables

### DSCP PHBs — the values worth memorising

| PHB | DSCP name | Decimal | Binary | Typical use |
|---|---|---|---|---|
| **Default** | DF / BE | **0** | 000000 | Best effort, `class-default` |
| **Expedited Forwarding** (RFC 3246) | **EF** | **46** | 101110 | **Voice bearer** — one queue, low latency |
| Class Selector | CS1 | 8 | 001000 | Scavenger / bulk |
| Class Selector | CS2 | 16 | 010000 | Network management |
| Class Selector | CS3 | 24 | 011000 | **Call signalling** |
| Class Selector | CS4 | 32 | 100000 | Realtime interactive |
| Class Selector | CS5 | 40 | 101000 | Broadcast video |
| Class Selector | CS6 | 48 | 110000 | **Network control (routing protocols)** |
| Class Selector | CS7 | 56 | 111000 | Reserved — do not use for user traffic |

**Class Selector = backward compatibility with IP Precedence.** CSx is
always `xxx000`, so its top 3 bits equal the old IPP value — a CS-marked
packet is read correctly by IPP-only gear. **CSx decimal = IPP × 8.**

### Assured Forwarding — AFxy

**Formula: `DSCP = 8x + 2y`**, where **x = class (1–4, which queue)** and
**y = drop probability (1–3, higher = dropped first)**.

| | Drop 1 (low) | Drop 2 (med) | Drop 3 (high) |
|---|---|---|---|
| **AF1x** | AF11 = **10** | AF12 = **12** | AF13 = **14** |
| **AF2x** | AF21 = **18** | AF22 = **20** | AF23 = **22** |
| **AF3x** | AF31 = **26** | AF32 = **28** | AF33 = **30** |
| **AF4x** | AF41 = **34** | AF42 = **36** | AF43 = **38** |

⚠️ **x and y do different jobs.** x picks the *queue* (bandwidth); y picks
the *drop order within that queue*. AF13 and AF11 sit in the same queue —
AF13 is simply thrown overboard first. That is exactly what markdown +
WRED exploit.

### Marking field sizes

| Field | Location | Bits | Values | Survives a routed hop? |
|---|---|---|---|---|
| **CoS / PCP** | 802.1Q tag (TCI) | 3 | 0–7 | ❌ **No — trunk-local only** |
| DEI / CFI | 802.1Q tag (TCI) | 1 | 0–1 | ❌ No |
| VLAN ID | 802.1Q tag (TCI) | 12 | 0–4095 | ❌ No |
| IP Precedence | IPv4 ToS byte | 3 | 0–7 | ✅ Yes (legacy) |
| **DSCP** | IPv4 ToS / IPv6 Traffic Class | **6** | **0–63** | ✅ **Yes** |
| ECN | IPv4 ToS / IPv6 Traffic Class | 2 | 0–3 | ✅ Yes |
| MPLS EXP | MPLS shim header | 3 | 0–7 | Within the MPLS domain |

### Token bucket formulas

| Quantity | Formula |
|---|---|
| **Tc (ms)** | `(Bc [bits] / CIR [bps]) × 1000` |
| **Bc (bits)** | `CIR × (Tc / 1000)` |
| Tcs per second | `1000 / Tc` |
| Packets conforming per Tc | `Bc / packet size in bits` (round down) |
| Packets per second | `packets per Tc × Tcs per second` |
| Time to send Bc at line rate | `(Bc [bits] / interface speed [bps]) × 1000` |

**Worked example (ENCOR OCG Ch.14):** 1 Gbps interface, CIR 120 Mbps, Bc 12 Mb.
`Tc = (12,000,000 / 120,000,000) × 1000 = 100 ms` → **10 Tcs/sec**.
1500-byte (12,000-bit) packets: `12,000,000 / 12,000 = 1000 packets per Tc`
→ `1000 × 10 = 10,000 pps` → `10,000 × 12,000 = 120 Mbps` ✅ (matches CIR).
At line rate those 1000 packets take `(12,000,000 / 1,000,000,000) × 1000 = 12 ms`,
leaving **88 ms of interpacket delay** in every 100 ms interval — which is
why a 100 ms Tc is far too long for voice.

### Policer types compared

| | Buckets | Rates | Colors | Excess source | Check order |
|---|---|---|---|---|---|
| **Single-rate two-color** | 1 | CIR | conform / exceed | none | conform → exceed |
| **srTCM** (RFC 2697) | 2 | CIR | conform / exceed / violate | **Bc overflow** (random) | conform → exceed → violate |
| **trTCM** (RFC 2698) | 2 | **CIR + PIR** | conform / exceed / violate | **PIR** (sustained) | 🚨 **violate → exceed → conform** |

### `police` command defaults

| Parameter | Default |
|---|---|
| **Bc** | **1500 bytes or CIR/32, whichever is larger** |
| **Be** (srTCM) | equal to Bc |
| **Be** (trTCM, `pir` present) | **1500 bytes or PIR/32, whichever is larger** |
| `conform-action` | `transmit` |
| `exceed-action` | `drop` |
| `violate-action` | `drop` |

### Queuing algorithms

| Algorithm | Queues | Bandwidth guarantee | Latency guarantee | Problem |
|---|---|---|---|---|
| FIFO | 1 | ❌ | ❌ | All traffic equal |
| Round robin | n | equal share | ❌ | No prioritisation at all |
| WRR | n | weighted | ❌ | Still no latency bound |
| CQ (Cisco WRR) | 16 | weighted | ❌ | Long delays; FIFO inside each queue |
| PQ | 4 | ❌ | high queue only | 🚨 **Starves lower queues** |
| WFQ | per-flow | fair, IPP-weighted | ❌ | **No fixed guarantee for any flow** |
| **CBWFQ** | **256** | ✅ minimum | ❌ | Not for real-time |
| **LLQ** | CBWFQ + PQ | ✅ | ✅ | **PQ must be policed** |

### CBWFQ / LLQ commands

| Command | What it does |
|---|---|
| `priority` | LLQ strict priority. **Pair with `police` or it starves other classes** |
| `priority <kbps> [burst]` | LLQ with a **conditional** policer — only enforced during congestion |
| `priority percent <n>` | Same, as a % of interface bandwidth (or of the parent shaper) |
| `priority level {1 \| 2}` | Multilevel strict priority — e.g. voice at level 1, video at level 2 |
| `bandwidth <kbps>` | Minimum bandwidth guarantee, absolute |
| `bandwidth percent <n>` | Minimum guarantee as % of interface bandwidth |
| `bandwidth remaining percent <n>` | Guarantee as % of **what is left after the priority queues** |
| `bandwidth remaining ratio <n>` | Same, expressed as a ratio |
| `fair-queue` | Flow-based queuing *within* a class |
| `shape average <bps>` | Shape to a mean rate, bursting to Bc each Tc (**the normal choice**) |
| `shape peak <bps>` | Shapes at `mean × (1 + Be/Bc)` — rarely used |
| `queue-limit <n>` | Change the tail-drop depth for the class |
| `random-detect [dscp-based \| precedence-based \| cos-based]` | Enable WRED (**precedence-based is the default**) |

### Queuing policy rules (these are exam answers and real config errors)

| Rule | Detail |
|---|---|
| `random-detect` / `fair-queue` need a partner | Require `bandwidth` or `shape` in the **same user class** (n/a to `class-default`) |
| `queue-limit` needs a partner | Requires `bandwidth`, `shape`, or `priority` in the same user class |
| **`priority` is exclusive** | `random-detect`, `shape`, `fair-queue`, `bandwidth` **cannot coexist with `priority` in the same class** |
| Unpoliced PQ blocks `bandwidth` | `bandwidth`/`bandwidth percent` cannot coexist in a policy map with **unpoliced** priority queues |
| The escape hatch | **`bandwidth remaining percent` CAN coexist with `priority`** — use it when the PQ is unconstrained |
| No mixing | All `bandwidth` commands in a policy map must be the **same type** |
| No explicit policer | `priority percent` / `priority level {1\|2} percent` do **not** support a separate `police` |
| Sum rule | Total of all class bandwidths **should not exceed 100%** |
| Direction | **Class-based shaping is outbound only** |

## Config Patterns

```ios-xe
! ---------- 1. CLASSIFY (match-all is the default; say match-any if you mean OR)
class-map match-any VOIP-TELEPHONY
 match dscp ef
class-map match-any VIDEO
 match dscp af41 af42 af43
class-map match-all CRITICAL-DATA
 match dscp af31
class-map match-any SCAVENGER
 match dscp cs1
class-map match-any LYNC-APP
 match protocol lync                    ! NBAR2 — Layer 7 classification

! ---------- 2. MARK at the trust boundary (ingress, closest to the source)
policy-map MARKING
 class VOIP-TELEPHONY
  set dscp ef
 class VIDEO
  set dscp af41
 class SCAVENGER
  set dscp cs1
 class class-default
  set dscp default                      ! explicitly bleach unclassified traffic

! ---------- 3a. POLICER — single-rate two-color
policy-map OUTBOUND-POLICY
 class VOIP-TELEPHONY
  police 50000000 conform-action transmit exceed-action drop
 class VIDEO
  police 25000000 conform-action transmit exceed-action set-dscp-transmit af21

! ---------- 3b. POLICER — srTCM (single rate, three colors)
policy-map SRTCM-POLICY
 class VOIP-TELEPHONY
  police 50000000 conform-action set-dscp-transmit af31 exceed-action set-dscp-transmit af32 violate-action drop

! ---------- 3c. POLICER — trTCM (two rates: violate is checked FIRST)
policy-map TRTCM-POLICY
 class VOIP-TELEPHONY
  police cir 50000000 pir 100000000 conform-action transmit exceed-action set-dscp-transmit af31 violate-action drop

! ---------- 4. QUEUE — LLQ + CBWFQ, priority queues CONDITIONALLY POLICED
policy-map QUEUING
 class VOIP
  priority level 1 percent 30           ! policed -> other classes keep their guarantees
 class VIDEO
  priority level 2 percent 30
 class CRITICAL
  bandwidth percent 10
 class SCAVENGER
  bandwidth percent 5
 class TRANSACTIONAL
  bandwidth percent 15
 class class-default
  bandwidth percent 10
  fair-queue
  random-detect dscp-based
  queue-limit 64

! ---------- 5. QUEUE — UNCONSTRAINED priority queues
! With no policer on the PQs, 'bandwidth percent' is illegal here.
! 'bandwidth remaining percent' is the only legal way to divide what's left.
policy-map QUEUING-UNCONSTRAINED
 class VOIP
  priority level 1
 class VIDEO
  priority level 2
 class CRITICAL
  bandwidth remaining percent 40
 class SCAVENGER
  bandwidth remaining percent 20
 class TRANSACTIONAL
  bandwidth remaining percent 20
 class class-default
  bandwidth remaining percent 20
  fair-queue
  random-detect dscp-based

! ---------- 6. HIERARCHICAL — shape to the SP rate, queue INSIDE the shaper
! The child policy's percentages are calculated against the shaper's
! 100 Mbps mean rate, NOT the interface's 1 Gbps.
policy-map SHAPING
 class class-default
  shape average 100000000
  service-policy QUEUING

! ---------- 7. APPLY (the PARENT policy is the one attached)
interface GigabitEthernet1
 service-policy output SHAPING
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| **Mark as close to the source as possible; classify/mark at the access edge** | Core devices then act on the mark alone — no deep inspection at high speed | Endpoints that cannot mark, or a merged/acquired network with an untrusted access layer | RFC 2475 (DiffServ architecture); ENCOR OCG Ch.14 "Trust Boundary" |
| **Do not trust endpoint markings by default; use conditional trust for IP phones** | Any PC can set DSCP EF and help itself to the priority queue | Managed, locked-down endpoints under the same admin control | ENCOR OCG Ch.14 "Trust Boundary" |
| **Voice bearer = EF; call signalling = CS3; routing protocols = CS6** | Standard service-class mapping every vendor implements the same way | An SP contract that mandates a different marking scheme at the handoff | RFC 4594 (Configuration Guidelines for DiffServ Service Classes) |
| **Total LLQ priority bandwidth ≤ 33% of link capacity** | An oversized PQ starves data classes and defeats the point of CBWFQ | Purpose-built voice/video links where data is genuinely secondary | Cisco Enterprise QoS Solution Reference Network Design Guide (QoS SRND) |
| **Always police the priority queue** | Unpoliced strict priority reproduces the legacy PQ starvation problem | None in production — this is what LLQ was invented to fix | ENCOR OCG Ch.14, Table 14-10 (`priority` description) |
| **Use WRED (`random-detect`) instead of tail drop on TCP classes** | Tail drop causes TCP global synchronization: sawtooth utilisation | Real-time/UDP classes — WRED on voice is pointless, drops are drops | RFC 2597 §; ENCOR OCG Ch.14 "Congestion-Avoidance Tools" |
| **`random-detect dscp-based` when classifying with DSCP** | The default is precedence-based and will ignore your AF drop precedence | A precedence-only legacy domain | ENCOR OCG Ch.14, Table 14-11 |
| **Keep Bc ≥ largest expected IP packet; target Tc 8–125 ms (8–10 ms for voice)** | Undersized Bc drops every large packet; oversized Tc creates burst-then-silence interpacket delay | High-throughput bulk transfer where jitter does not matter | ENCOR OCG Ch.14 "Token Bucket Algorithms" |
| **Sum of all class bandwidth guarantees ≤ 100%** | Over-subscription of guarantees is not a guarantee | None — IOS will generally reject it anyway | ENCOR OCG Ch.14 "Queuing policy guidelines" |
| **Use three-color policing only when the three actions actually differ** | If conform and exceed do the same thing, the second bucket is complexity for nothing | — | ENCOR OCG Ch.14 "Single-Rate Three-Color Markers/Policers" |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show policy-map interface <intf>` | **The single most useful QoS command.** Per-class packet/byte counters, conform/exceed/violate counts, drops, queue depth. **If a class shows 0 packets, your classification is wrong — not your policy** |
| `show policy-map <name>` | Resolved config **including the defaults IOS filled in** — this is how you read the real Bc/Be it chose |
| `show class-map [<name>]` | Match criteria and, critically, whether it is **match-all or match-any** |
| `show policy-map interface <intf> output class <class>` | Narrow to one class when the full output is unreadable |
| `show mls qos interface <intf>` | Switch trust state (trust dscp / trust cos / untrusted) on Catalyst platforms |
| `show table-map <name>` | Marking translation tables (CoS↔DSCP mapping) |
| `show interfaces <intf>` | `txload`, `output drops`, and the **output queue depth** — confirms congestion is real |
| `show queueing interface <intf>` | Active queuing strategy and per-queue depth |
| `show run interface <intf> \| include service-policy` | Confirm the policy is attached **and in the right direction** |

## Intent Questions
- **What is this network's service-class model?** Which applications are
  meant to be EF, which AF class, and what is the intended fate of everything
  in `class-default`?
- **Where is the trust boundary supposed to be**, and is it being enforced
  there — or is an endpoint marking its own traffic EF?
- **Is this link actually congested?** QoS only acts under congestion; a
  policy on an uncongested link changes nothing and is not the fault.
- **Is there a contracted rate (SP SLA) here** the egress must stay under,
  and is the shaper's mean rate set to that number?

## Troubleshooting Checklist
0. **State intent vs. observed:** answer the Intent Questions above for this
   network, then write the one-line symptom ("voice should be in the priority
   queue, isn't — jitter on calls through the WAN edge") — before running any
   show command.
1. **Is the link actually congested?** `show interfaces` for `txload` and
   `output drops`. **QoS is inert without congestion** — if the link is at
   5%, the problem is elsewhere (duplex, MTU, routing, the app itself).
2. **Is the policy even attached, and in the right direction?**
   `show run interface … | include service-policy`. **Shaping is
   outbound-only** — an inbound `shape` will not be there.
3. **Is traffic hitting the class you think?** `show policy-map interface`.
   **A class with 0 packets is a classification bug**, and the traffic is
   silently in `class-default`.
4. **Is `match-all` doing what you meant?** It is the default. Two matches
   that cannot both be true = a class that never matches.
5. **Are the markings what you expect on arrival?** If DSCP is 0 at the far
   end, something bleached it — a trust boundary set to untrusted, or a
   policy re-marking `class-default`. Remember **CoS never survives a routed
   hop**; only DSCP does.
6. **Is the priority queue policed?** An unpoliced PQ starves everything
   else — the symptom is "voice is perfect, all other traffic is dying."
7. **Check the policer's real Bc/Be**, not the configured ones:
   `show policy-map <name>`. **Bc smaller than the largest packet drops every
   large packet** while small packets pass — an intermittent-looking failure
   that is actually deterministic.
8. **High exceed/violate counters with correct config** = the CIR is genuinely
   too low, or Bc is too small for the burst profile. Distinguish "policy is
   broken" from "policy is working and the traffic is over the contract."
9. **Sawtooth throughput on TCP** = tail drop and global synchronization →
   apply WRED (`random-detect dscp-based`) to the affected class.
10. **Hierarchical policy percentages look wrong?** A child policy's
    percentages are relative to the **parent shaper's mean rate**, not the
    interface speed.
11. Platform differences — the same command may behave differently or not
    exist on a given switch/router. Verify against the platform's own
    configuration guide before calling it a bug.

## Common Pitfalls
- 🚨 **Marking is not behavior.** Setting DSCP EF does nothing unless a
  downstream device has a policy that acts on EF. Marking without a queuing
  policy is the most common "I configured QoS and nothing changed."
- 🚨 **`class-map` is `match-all` by default.** Everyone assumes OR; the
  default is AND.
- 🚨 **CoS does not survive a routed hop** — it lives in the 802.1Q tag.
  Anything that must hold end-to-end has to be DSCP.
- 🚨 **trTCM checks violate FIRST**, the reverse of srTCM. Exam favourite,
  and a real source of surprise when tuning a policy.
- **srTCM's Be comes from Bc's leftover tokens** (random and bursty); trTCM's
  comes from the PIR (sustained and predictable). They are not interchangeable.
- **`Tc` the interval and `Tc` the Bc bucket token count are different
  things** and the OCG uses the same letters for both. Read from context.
- **A full token bucket discards new tokens.** You cannot save up quiet time
  to fund a later burst beyond Bc (or Be).
- **Bc default is `max(1500 bytes, CIR/32)`** — not simply 1500. On a fast
  CIR, IOS picks a much larger Bc than people expect.
- **`priority` cannot share a class with `bandwidth`, `shape`, `fair-queue`,
  or `random-detect`.** When the PQ is unpoliced, `bandwidth remaining
  percent` is the *only* legal way to divide the rest.
- **QoS only helps under congestion.** It cannot fix a slow application, a
  duplex mismatch, or an undersized link — it only decides who suffers when
  the link is genuinely full.
- **Policing an uncongested link still drops traffic.** A policer enforces its
  rate whether or not there is congestion; LLQ's conditional policer is the
  one that only bites during congestion. Do not confuse the two.
- **`class-default` is where all your unclassified traffic actually is.**
  Leaving it untouched is a policy decision, usually an unintended one.
