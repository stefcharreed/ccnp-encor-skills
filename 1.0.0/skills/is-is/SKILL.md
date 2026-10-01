---
name: ccnp-is-is
description: >
  Use this skill when troubleshooting or configuring IS-IS on IOS-XE — fundamentals,
  adjacency troubleshooting, and authentication. Invoke when the user asks about: IS-IS,
  ISIS, NET, system ID, area address, Level 1, Level 2, L1/L2, is-type, isis circuit-type,
  IS-IS adjacency, adjacency stuck INIT, ES-IS, show clns neighbors, show clns interface,
  IS-IS MTU mismatch, hello padding, no isis hello padding, clns mtu, DIS, IS-IS priority,
  metric-style wide, narrow metrics, attached bit, ATT bit, route leaking, redistribute isis
  ip level-2 into level-1, MPLS LDP IGP sync, isis password, area-password, domain-password,
  isis authentication mode md5, key-chain, authentication send-only, authenticate snp, IIH,
  LSP, CSNP, PSNP, debug isis adj-packets, LSP authentication failed.
---

## Purpose
IS-IS is the link-state IGP that underpins SD-Access and most service-provider cores; it
decides which routers become neighbors (levels, areas, MTU, authentication) and which link
state each router will accept — and its failures split cleanly into "no adjacency" and
"adjacency up but routes missing," which need completely different checks.

> **Blueprint note:** IS-IS has **no standalone ENCOR OCG chapter**. It matters for ENCOR
> because it is the **SD-Access underlay IGP** (see `fabric-technologies`) and because
> practice exams test it. Everything here is cited to Cisco documentation, except the
> clearly labelled **field sample** of Cisco Community threads, which is evidence about what
> breaks in practice — not documentation.

## Key Concepts

**Addressing and levels**
- **NET (Network Entity Title)** = **area address + 6-byte system ID + NSEL**, and the NSEL
  is always **00** on a router. Example: `49.0001.0000.0000.0001.00` → area `49.0001`,
  system ID `0000.0000.0001`. Unlike OSPF, **the area is a property of the router (in the
  NET), not of a link** — "every single router belongs to an Area."
- **Level 1 (L1)** = routing **within** an area. **Level 2 (L2)** = routing **between** areas
  (the backbone). A router is L1, L2, or **L1/L2** (`is-type`).
- **Defaults (IOS XE):** the **first IS-IS instance is `level-1-2`**; later instances default
  to Level 1. Interface `isis circuit-type` defaults to **`level-1-2`**. Interface metric
  **0–63, default 10**. **Metric style defaults to narrow.**
- **`is-type`** (router mode) sets what the *router* can be; **`isis circuit-type`** (interface
  mode) sets which hellos the *interface* sends and which adjacencies it will form.

**Adjacencies**
- **States:** **Down** (no hellos received), **Init** (a hello was seen but the handshake is not
  complete — one side recognizes the neighbor), **Up** (bidirectional hello exchange confirmed).
- **Who forms an adjacency with whom** (see the matrix in Reference Tables): **L1 needs a
  matching area ID; L2 does not care about area ID**; L1 and L2-only routers never form an
  adjacency with each other.
- **Hello padding:** IS-IS **pads hellos (IIHs) to the full interface MTU by default** — "for
  early detection of errors due to transmission problems with large frames or due to
  mismatched MTUs." That is why an MTU mismatch blocks the adjacency instead of breaking
  something later.
- **MTU mismatch symptom is asymmetric:** the router with the **larger MTU** sends hellos the
  other side discards, so it **stays in INIT**; the router with the **smaller MTU** only
  receives unpadded **ES hellos** and shows an **ES-IS** adjacency instead of IS-IS.
- **`no isis hello padding`** without `always` still sends **full-MTU hellos until the
  adjacency is Up**, then shrinks them; **`no isis hello padding always`** never pads.
  Router-level form: `no hello padding [multi-point | point-to-point]`.
- **DIS (Designated IS)** on multi-access links: highest **priority wins (0–127, default
  64)**, tie broken by **highest SNPA (MAC address)**.
- **Hello timer default 10 s, hold multiplier 3** (30 s hold).

**Routing between levels**
- An **L1/L2 router sets the ATT (attached) bit** in its L1 LSP. L1 routers react by
  installing a **default route toward that L1/L2 router**. **L2 routes are NOT leaked into L1
  by default** — an L1 router seeing only a default route is *normal*, not a fault.
- To leak specific L2 routes into L1 (to avoid suboptimal routing):
  `redistribute isis ip level-2 into level-1 route-map <name>` on the L1/L2 router.
- `show isis database` marks the ATT bit in the **ATT/P/OL** column.

**Metrics**
- **Narrow** = old-style TLVs (6-bit interface metric, 0–63). **Wide** = new-style TLVs
  (**24-bit interface, 32-bit path metric**). **`transition`** generates and accepts both.
- Wide metrics are **required** for features such as prefix tags and MPLS TE, and Cisco's
  fast-convergence best-practice guide recommends `metric-style wide`.
- **Mismatched metric styles** between neighbors: the adjacency comes **up** but routes do not
  install — each side ignores the other's TLV style. In `show isis database detail`,
  **"IS-Extended"** marks a wide-metric neighbor; plain **"IS"** marks narrow.

**Interaction with MPLS**
- **MPLS LDP-IGP synchronization** (`mpls ldp sync` under `router isis`) makes IS-IS
  **advertise max-metric on a link until LDP converges** on it. If the LDP peer is
  unreachable, the adjacency proceeds after the holddown timer. A link "up in IS-IS but never
  used" on an MPLS core is often this feature — check `show mpls ldp igp sync`.

**Authentication — three scopes, three different failure modes**
- **PDUs:** **IIH** (hellos: LAN and point-to-point), **LSP** (routing information), **CSNP**
  (lists every LSP; the DIS sends them periodically on a LAN), **PSNP** (lists a subset;
  acknowledges and requests LSPs).
- **Interface authentication** — `isis password` (interface mode). Carried in **Hellos**; **no
  level keyword = both L1 and L2**. **Mismatch → no adjacency.**
- **Area authentication** — `area-password` (router mode). Carried in **L1 LSPs, CSNPs and
  PSNPs**. **Mismatch → adjacency up, L1 LSPs rejected.**
- **Domain authentication** — `domain-password` (router mode). Carried in **L2 LSPs**; L2
  SNPs carry it **only with `authenticate snp`** — by default "the IS-IS routing protocol does
  not insert the password into SNPs." **Mismatch → adjacency up, L2 LSPs rejected.**
- **HMAC-MD5** (IOS 12.2(13)T+) "adds an HMAC-MD5 digest to each IS-IS PDU," using a key chain
  with `authentication mode md5` per interface (`isis authentication ...`) or per process
  (`authentication ...` under `router isis`). **Enhanced clear text** uses the same commands
  with `mode text`.
- **Legacy and new modes cannot coexist** on the same scope and level; configuring MD5
  **overrides** `area-password` / `domain-password`. Key ID must be **numeric**; key-string
  **1–80 characters, first character not numeric**.

## Procedure
Non-disruptive migration to HMAC-MD5 authentication (Cisco config guide — "perform the task
steps in the order shown, which requires moving from router to router"):
1. Run an IOS image that supports the target authentication type on **every** router.
2. Configure **`authentication send-only`** on all routers — they **send** authenticated PDUs
   but still **accept** unauthenticated ones.
3. Configure the **authentication mode and key chain** on all routers.
4. Remove send-only (**`no authentication send-only`**) router by router — enforcement starts.

Migration from narrow to wide metrics (Cisco best-practice guide):
1. Configure **`metric-style transition`** on every router so each generates and accepts both
   TLV styles.
2. Once every router is in transition, move them to **`metric-style wide`**.

## Reference Tables

**Adjacency matrix (Cisco: Configure IS-IS Adjacency and Area Types)**

| Router ↓ / Neighbor → | L1 | L1/L2 | L2 |
|---|---|---|---|
| **L1** | L1 adjacency **if area ID matches**, else none | L1 adjacency **if area ID matches**, else none | **No adjacency** |
| **L1/L2** | L1 adjacency **if area ID matches**, else none | **L1 + L2 if area ID matches, else L2 only** | L2 adjacency, **area ID does not matter** |
| **L2** | **No adjacency** | L2 adjacency, area ID does not matter | L2 adjacency, area ID does not matter |

**Symptom → most likely cause**

| Symptom | Most likely cause | Confirm with |
|---|---|---|
| Neighbor stuck **INIT** on one side, **ES-IS** on the other | **MTU mismatch** (padded hellos dropped) | `show clns interface` MTU both sides; `debug isis adj-packets` hello lengths |
| Hellos sent, none received (`PTP Hellos (sent/rcvd): 15/11`, or no "Rec … IIH") | **Unidirectional link** or something in the path dropping hellos | `show clns traffic`, packet capture |
| No adjacency, MTU equal | **Level / circuit-type / area** mismatch, or **interface password** mismatch | `show clns interface`, `show run`, the matrix above |
| Adjacency flaps "**hold time expired**" then recovers | **Unreliable transport** — hellos delayed/lost in transit | Syslog `%CLNS-5-ADJCHANGE`, captures |
| Adjacency **up**, routes missing | **Area/domain password**, **metric-style mismatch**, or LDP-IGP sync max-metric | `show isis database detail`, `debug isis update-packets`, `show mpls ldp igp sync` |
| L1 router has only a **default route** to other areas | **Normal** — ATT bit default; L2 routes are not leaked by default | `show isis database` ATT column |

**Authentication: which command protects which PDUs**

| Scope | Command | Mode | PDUs protected | Level | Mismatch result |
|---|---|---|---|---|---|
| **Interface** | `isis password <pw> [level-1 \| level-2]` | Interface | **Hellos (IIH)** | Both if unspecified | **No adjacency** |
| **Area** | `area-password <pw>` | router isis | **L1 LSPs, CSNPs, PSNPs** | L1 only | Adjacency up, **L1 LSPs rejected** |
| **Domain** | `domain-password <pw> [authenticate snp {validate \| send-only}]` | router isis | **L2 LSPs** (+ L2 SNPs only with `authenticate snp`) | L2 only | Adjacency up, **L2 LSPs rejected** |
| **MD5 / enhanced text — per interface** | `isis authentication mode {md5 \| text} [level-x]` + `isis authentication key-chain <name> [level-x]` | Interface | Hellos on that interface | Both if unspecified | — |
| **MD5 / enhanced text — per process** | `authentication mode {md5 \| text} [level-x]` + `authentication key-chain <name> [level-x]` | router isis | LSPs, CSNPs, PSNPs | Both if unspecified | — |

**Field sample — what actually breaks (Cisco Community, 2026-10-01)**
*Not documentation. 17 problem threads from search-selected Cisco Community posts (2002–2026);
small, search-biased, measures who posts rather than true frequency. Used only to order the
troubleshooting checklist.*

| Root cause | Threads | Note |
|---|---|---|
| MTU / hello padding | 4 | 1 confirmed, 3 expert-diagnosed |
| Unresolved | 4 | — |
| Authentication | 2 | Both IOS-XE ↔ IOS-XR behavior |
| Level / is-type design | 2 | Incl. "L1 not getting L2 routes" (normal) |
| Other feature (LDP-IGP sync, QoS policy) | 2 | — |
| Interface mode / encapsulation | 2 | FabricPath members; OSI + IP IS-IS mixed |
| Metric-style mismatch | 1 | Cisco ↔ Junos |
| **Mixed-platform involvement (cross-cutting)** | **5 of 17** | XE↔XR or Cisco↔Junos defaults differ |

## Config Patterns
```ios-xe
! ===== Baseline L1/L2 router =====
router isis
 net 49.0001.0000.0000.0001.00
 is-type level-1-2                    ! default for the first instance
 metric-style wide                    ! recommended; default is narrow
!
interface GigabitEthernet0/1
 ip address 192.0.2.1 255.255.255.252
 ip router isis
 isis network point-to-point          ! no DIS election on a 2-router link
 isis circuit-type level-2-only       ! form only L2 adjacencies on this link

! ===== MTU mismatch — fix the MTU, not the padding =====
interface GigabitEthernet0/2
 clns mtu 1500                        ! IS-IS-specific MTU; no interface bounce needed
! (alternative: mtu 1500 + shutdown/no shutdown)

! ===== Leak selected L2 routes into L1 =====
ip prefix-list LEAK permit 198.51.100.0/24
route-map L2-TO-L1 permit 10
 match ip address prefix-list LEAK
router isis
 redistribute isis ip level-2 into level-1 route-map L2-TO-L1

! ===== MPLS LDP-IGP sync =====
router isis
 mpls ldp sync
interface GigabitEthernet0/3
 mpls ldp igp sync delay 10           ! per-interface delay

! ===== Authentication: legacy clear text, all three scopes =====
interface GigabitEthernet0/1
 isis password IfacePw1               ! Hellos, L1 + L2
router isis
 area-password AreaPw1                ! L1 LSP/CSNP/PSNP
 domain-password DomPw1 authenticate snp validate   ! L2 LSPs + L2 SNPs

! ===== Authentication: HMAC-MD5 with send-only migration =====
key chain ISIS-KEYS
 key 1
  key-string IsisMd5Key
interface GigabitEthernet0/1
 isis authentication mode md5
 isis authentication key-chain ISIS-KEYS
router isis
 authentication send-only             ! step 2 on EVERY router first
 authentication mode md5
 authentication key-chain ISIS-KEYS
 no authentication send-only          ! step 4, router by router
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Fix MTU mismatches by making MTU equal on both ends — do not disable hello padding to "fix" an adjacency | Padding exists to expose MTU problems early; disabling it can mask a mismatch that later causes traffic loss through LSP flooding failures | Bandwidth/CPU-constrained links where both ends' MTU is verified equal (Cisco's fast-convergence guide recommends disabling padding in that case) | [Cisco: IS-IS Hello Padding Behavior](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/119399-technote-isis-00.html); [Cisco: MTU Mismatch Problem in IS-IS](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/47201-isis-mtu.html) |
| Use `metric-style wide` (migrating via `transition`) | Narrow metrics cap interface cost at 63 and cannot carry prefix tags or MPLS TE information | A network still containing routers that only support narrow TLVs — run `transition` until they are gone | [Cisco: Setting Best Practice Parameters for IS-IS Fast Convergence](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-16-12/irs-xe-16-12-book/irs-fscbp.html) |
| Prefer HMAC-MD5 key-chain authentication over legacy clear-text passwords | Clear text sends the password in plain text; MD5 adds a digest to every PDU and supports key rotation | Interop with a device that only supports clear text — documented, time-boxed | [Cisco IOS XE: Configuring IS-IS Authentication](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-3e/irs-xe-3e-book/rs-scty-0.html) |
| Migrate authentication with `send-only` on all routers before enforcing | Enforcing on one router before its neighbors send authenticated PDUs drops adjacencies or LSPs | Greenfield build or a maintenance window where an outage is acceptable | [Cisco IOS XE: Configuring IS-IS Authentication](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-3e/irs-xe-3e-book/rs-scty-0.html) |

*A deviation from this table is a question for the network's operator — "is this intentional
here?" — never automatically a finding.*

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show clns neighbors [detail]` | State **Up/Init**, and **protocol IS-IS vs ES-IS** — ES-IS on one side + INIT on the other = MTU |
| `show isis neighbors [detail]` | Adjacency **type L1/L2/L1L2** and state |
| `show clns interface <int>` | **MTU**, **circuit type**, level priority, hello/hold timers |
| `show clns traffic` | **PTP Hellos (sent/rcvd)** — sent far exceeding received = one-way loss |
| `debug isis adj-packets [int]` | Hello **lengths** each side sends (MTU), and whether "Rec … IIH" ever appears |
| `show isis database [detail]` | LSPs actually accepted; **ATT bit**; **"IS" vs "IS-Extended"** (narrow vs wide) |
| `debug isis update-packets` | **"LSP authentication failed"** for area/domain password mismatches |
| `show mpls ldp igp sync` | Whether LDP sync is achieved; unsynced links advertise max-metric |

## Intent Questions
- What level should each router be (L1, L2, L1/L2), and which area is each in?
- Should L1 areas get only the ATT-bit default, or are specific L2 routes meant to be leaked?
- Which metric style is the network standardized on — and are any other vendors/OSes in the
  domain whose defaults may differ?
- Which authentication scopes are intended (interface, area, domain), clear text or MD5, and
  is a migration (`send-only`) in progress?

## Troubleshooting Checklist
*Ordered by the field sample: MTU/hello problems were the most common root cause, and
mixed-platform pairs were involved in about a third of cases.*

0. State intent vs. observed: answer the Intent Questions above for this network, then write
   the one-line symptom ("should ___, isn't ___") — before running any show command. Then
   decide which branch you are in: **no adjacency** (steps 1–5) or **adjacency up, routes
   missing** (steps 6–9).
1. **Layer 1 / one-way loss:** interface up/up; `show clns traffic` — are hellos received at
   all? Sent ≫ received means a unidirectional link or something in the path dropping them.
2. **MTU (most common):** INIT on one side and ES-IS on the other is the signature. Compare
   `show clns interface` MTU on both ends and the hello lengths in `debug isis adj-packets`.
   Check any **transit switch / provider path** too — it must pass full-MTU frames. Fix the MTU
   (`clns mtu` or `mtu` + bounce), don't just disable padding. **IOS-XR and IOS calculate MTU
   differently** — a field-reported cause on mixed links.
3. **Levels and areas:** apply the adjacency matrix — L1↔L2-only never forms; L1 needs a
   matching area ID. Check both `is-type` and `isis circuit-type`.
4. **Interface authentication:** `isis password` / `isis authentication` mismatch, or the
   level keyword covering only one level.
5. **Network type / encapsulation:** point-to-point vs broadcast on the two ends; on
   FabricPath or port-channels, confirm **member interfaces** are in the right mode.
6. **Adjacency up, routes missing — LSPs rejected?** Compare `show isis database` on both
   sides; `debug isis update-packets` for "LSP authentication failed" → **area/domain password**.
7. **Metric style:** LSPs present but routes not installed → compare narrow vs wide
   ("IS" vs "IS-Extended"). Very likely on Cisco ↔ other-vendor or XE ↔ XR pairs.
8. **L1 missing other-area routes:** expected — L1 uses the ATT-bit default. Leak only if
   suboptimal routing is a real problem.
9. **Other features:** MPLS **LDP-IGP sync** advertising max-metric (`show mpls ldp igp sync`);
   **QoS or other policies** applied to the IS-IS links (a field case: a Catalyst Center
   application policy on SD-Access links dropped IS-IS neighbors until removed).

## Common Pitfalls
- **Disabling hello padding to "fix" an adjacency.** It hides the MTU mismatch; LSP flooding can
  still fail later. And `no isis hello padding` without `always` still pads until Up.
- **Reading ES-IS in `show clns neighbors` as "IS-IS is sort of up."** It means IS-IS hellos are
  *not* arriving — usually MTU.
- **Treating "L1 router has no L2 routes" as a fault.** L2 routes are not leaked by default;
  the ATT-bit default route is the design.
- **Thinking the area is per-link like OSPF.** In IS-IS the area is in the router's NET; L1
  adjacency needs a matching area, L2 does not.
- **Mixing narrow and wide metrics.** Adjacency up, routes silently missing. Use `transition`
  during migration.
- **Assuming another platform has the same defaults.** XE↔XR and Cisco↔Junos pairs show up in
  about a third of the field sample (MTU math, metric style, authentication behavior).
- **Confusing the three authentication commands.** `isis password` = **interface** (Hellos).
  `area-password` = **area** (L1). `domain-password` = **domain** (L2). There is **no "router
  authentication"** in IS-IS — that distractor borrows OSPF-style wording.
- **Expecting every password mismatch to drop the adjacency.** Only the **interface** password
  does; area/domain mismatches leave adjacencies up and silently reject LSPs.
- **Assuming `domain-password` covers L2 CSNPs/PSNPs.** Only with `authenticate snp`.
- **Enforcing authentication before every neighbor sends it** — skip `send-only` and things drop.
- **Mixing legacy and new authentication modes on the same scope/level** — not allowed; MD5
  overrides the legacy passwords.

## Exam Preparation Tasks

### "Do I Know This Already?" question analysis
*(From a Boson ENCOR practice question, not the OCG.)*

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| Which type of authentication is configured with `isis password`? | Mapping each IS-IS auth command to its **scope** — and that `isis password` is the only **interface-mode** one | **C — interface authentication** | **Area** and **domain** are real IS-IS scopes but use `area-password` / `domain-password` under `router isis`. **"Router authentication"** is not an IS-IS scope at all — a plausible-sounding invented term. See Common Pitfalls |

*Sources: [Troubleshoot IS-IS Adjacency Issues](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/220649-troubleshoot-is-is-adjacency-issues.html) ·
[MTU Mismatch Problem in IS-IS](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/47201-isis-mtu.html) ·
[IS-IS Hello Padding Behavior](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/119399-technote-isis-00.html) ·
[Configure IS-IS Adjacency and Area Types](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/200293-IS-IS-Adjacency-and-Area-Types.html) ·
[Configure the Attach Bit](https://www.cisco.com/c/en/us/support/docs/ip/ip-routing/200472-Configure-the-Attach-bit-set.html) ·
[IS-IS Configuration Guide, IOS XE 17 (Catalyst 9000)](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr3-fwd/isis/is-is-configuration-guide/is-is.html) ·
[Setting Best Practice Parameters for IS-IS Fast Convergence](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-16-12/irs-xe-16-12-book/irs-fscbp.html) ·
[MPLS LDP IGP Synchronization, IOS XE 17](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/mpls/b-mpls/m_mp-ldp-igp-synch.html) ·
[IS-IS Authentication tech note](https://www.cisco.com/c/en/us/support/docs/ip/integrated-intermediate-system-to-intermediate-system-is-is/13792-isis-authent.html) ·
[Configuring IS-IS Authentication, IOS XE](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-3e/irs-xe-3e-book/rs-scty-0.html)*
