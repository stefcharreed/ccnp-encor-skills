---
name: ccnp-is-is
description: >
  Use this skill when troubleshooting or configuring IS-IS authentication on IOS-XE.
  Invoke when the user asks about: IS-IS, ISIS, isis password, area-password,
  domain-password, IS-IS interface authentication, IS-IS area authentication,
  IS-IS domain authentication, IS-IS HMAC-MD5, isis authentication mode md5,
  isis authentication key-chain, authentication send-only, authenticate snp, IIH,
  LSP, CSNP, PSNP, DIS, level-1, level-2, show clns neighbors, LSP authentication failed.
---

## Purpose
IS-IS authentication controls which neighbors may form adjacencies and which link-state
information a router will accept — and because it is split into three independent scopes
(interface, area, domain), a mismatch in one scope can break routing while adjacencies
still look healthy.

> **Blueprint note:** IS-IS has **no standalone ENCOR OCG chapter**. It matters for ENCOR
> because it is the **SD-Access underlay IGP** (see `fabric-technologies`) and because
> practice exams test the authentication commands directly. This skill is built from two
> Cisco documents, cited inline, plus one practice-exam question.

## Key Concepts
- **IS-IS PDU types** (what authentication protects):
  - **IIH (IS-IS Hello)** — LAN Hello and Point-to-Point Hello. Forms and maintains adjacencies.
  - **LSP (Link State PDU)** — carries the routing information flooded between routers.
  - **CSNP (Complete Sequence Number PDU)** — lists *every* LSP in the database. On a LAN,
    the **DIS** (Designated Intermediate System, IS-IS's equivalent of the OSPF DR) sends
    CSNPs periodically.
  - **PSNP (Partial Sequence Number PDU)** — lists a *subset* of LSPs; used to acknowledge
    received LSPs and to request missing ones.
- **Levels:** Level 1 (L1) = routing **within** an area. Level 2 (L2) = routing **between**
  areas (the backbone / domain). A router can be L1, L2, or L1/L2.
- **Three clear-text authentication scopes** (the original IS-IS authentication):
  - **Interface authentication** — `isis password` in **interface** mode. Carried in
    **Hellos** for the level specified; **no level keyword = both L1 and L2**.
  - **Area authentication** — `area-password` in **router isis** mode. Carried in **L1 LSPs,
    CSNPs and PSNPs**. No level keyword (it is L1 by definition).
  - **Domain authentication** — `domain-password` in **router isis** mode. Carried in **L2
    LSPs**. **L2 CSNPs/PSNPs carry it only if you add `authenticate snp`** — by default "the
    IS-IS routing protocol does not insert the password into SNPs."
- **All three can run on the same router at the same time** — they protect different PDUs.
- **Mismatch behavior is different per scope — this is the troubleshooting key:**
  - **Interface** password mismatch → **no adjacency forms**.
  - **Area** password mismatch → **adjacency forms**, but the router **does not accept L1 LSPs**
    from the unauthenticated neighbor.
  - **Domain** password mismatch → **adjacency forms**, but the router **rejects L2 LSPs** from
    routers without matching domain authentication.
- **HMAC-MD5 authentication** (introduced in IOS 12.2(13)T) "adds an HMAC-MD5 digest to each
  IS-IS PDU" — LSP, LAN Hello, P2P Hello, CSNP and PSNP. Configured with a **key chain** plus
  `authentication mode md5`, either per interface (`isis authentication ...`) or for the whole
  process (`authentication ...` under `router isis`), each optionally per level.
- **Enhanced clear-text** authentication uses the same new commands with `mode text`, adding
  key-chain management and encrypted display of the password.
- **New and legacy modes cannot coexist on the same scope and level:** "Either authentication
  mode or old password mode may be configured on a given scope … and level — but not both."
  Configuring MD5 **automatically overrides** `area-password` and `domain-password`.
- **Key chain rules:** key ID must be **numeric**; key-string is **1–80** characters, and the
  **first character cannot be numeric**.

## Procedure
Non-disruptive migration to HMAC-MD5 (Cisco config guide — "perform the task steps in the
order shown, which requires moving from router to router"):
1. Run an IOS image that supports the target authentication type on **every** router.
2. Configure **`authentication send-only`** on all routers — each router now **sends**
   authenticated PDUs but still **accepts** PDUs with no or different authentication.
3. Configure the **authentication mode and key chain** on all routers.
4. Remove send-only (**`no authentication send-only`**) router by router — only now does each
   router start **enforcing** authentication on received PDUs.

## Reference Tables
**Which command protects which PDUs**

| Scope | Command | Mode | PDUs protected | Level | Mismatch result |
|---|---|---|---|---|---|
| **Interface** | `isis password <pw> [level-1 \| level-2]` | Interface | **Hellos (IIH)** | Both if unspecified | **No adjacency** |
| **Area** | `area-password <pw>` | router isis | **L1 LSPs, CSNPs, PSNPs** | L1 only | Adjacency up, **L1 LSPs rejected** |
| **Domain** | `domain-password <pw> [authenticate snp {validate \| send-only}]` | router isis | **L2 LSPs** (+ L2 SNPs only with `authenticate snp`) | L2 only | Adjacency up, **L2 LSPs rejected** |
| **MD5 / enhanced text — per interface** | `isis authentication mode {md5 \| text} [level-1 \| level-2]` + `isis authentication key-chain <name> [level-x]` | Interface | Hellos on that interface | Both if unspecified | — |
| **MD5 / enhanced text — per process** | `authentication mode {md5 \| text} [level-1 \| level-2]` + `authentication key-chain <name> [level-x]` | router isis | LSPs, CSNPs, PSNPs | Both if unspecified | — |

## Config Patterns
```ios-xe
! ===== Legacy clear-text: all three scopes on one router =====
interface GigabitEthernet0/1
 ip address 192.0.2.1 255.255.255.0
 ip router isis
 isis password IfacePw1            ! interface auth: Hellos, L1 + L2
!
router isis
 net 49.0001.0000.0000.0001.00
 area-password AreaPw1             ! area auth: L1 LSP/CSNP/PSNP
 domain-password DomPw1 authenticate snp validate   ! domain auth: L2 LSPs + L2 SNPs

! ===== HMAC-MD5 with a key chain =====
key chain ISIS-KEYS
 key 1
  key-string IsisMd5Key
!
interface GigabitEthernet0/1
 isis authentication mode md5                 ! Hellos on this link
 isis authentication key-chain ISIS-KEYS
!
router isis
 authentication mode md5                      ! LSPs/SNPs for the process
 authentication key-chain ISIS-KEYS

! ===== Migration: send-only first, enforce last =====
router isis
 authentication send-only                     ! step 2 on EVERY router
 authentication mode md5                      ! step 3
 authentication key-chain ISIS-KEYS
! ...then, router by router:
 no authentication send-only                  ! step 4
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Prefer HMAC-MD5 key-chain authentication over the legacy clear-text `isis password` / `area-password` / `domain-password` | Clear-text authentication sends the password in plain text; HMAC-MD5 adds a digest to every IS-IS PDU and supports key-chain rotation | Interoperating with a router/IOS that only supports clear text — a documented, time-boxed exception | [Cisco IOS XE: Configuring IS-IS Authentication](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-3e/irs-xe-3e-book/rs-scty-0.html) |
| Migrate authentication with `send-only` on all routers first, enforcing last | Enabling enforcement on one router before its neighbors send authenticated PDUs drops adjacencies or LSPs mid-migration | A greenfield build or maintenance window where a full outage is acceptable | [Cisco IOS XE: Configuring IS-IS Authentication](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_isis/configuration/xe-3e/irs-xe-3e-book/rs-scty-0.html) |

*A deviation from this table is a question for the network's operator — "is this intentional
here?" — never automatically a finding.*

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show clns neighbors` | Adjacency state per neighbor. **Missing neighbor** on an authenticated link points at an **interface (Hello)** password mismatch |
| `show isis neighbors` | IS-IS adjacency type (L1/L2/L1L2) and state |
| `show isis database` | LSPs actually accepted. **Neighbor up but its LSPs missing** points at an **area or domain** password mismatch |
| `debug isis update-packets` | **"LSP authentication failed"** messages naming the rejecting level |
| `show run \| section router isis` | Which scope uses legacy passwords vs `authentication mode`, and whether `send-only` is still configured |

## Intent Questions
- Which scopes are supposed to be authenticated here — interface (adjacencies), area (L1
  LSPs), domain (L2 LSPs), or all three?
- Clear text or HMAC-MD5 — and is a migration in progress (is `send-only` intentional)?
- Is each router L1, L2, or L1/L2 — i.e. which passwords should it even need?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write
   the one-line symptom ("should ___, isn't ___") — before running any show command.
1. **Physical/Layer 2:** interface up/up and IS-IS enabled on it (`ip router isis`).
2. **No adjacency at all?** Check **interface authentication** first — an `isis password` or
   `isis authentication` mismatch (or one side unconfigured) prevents the adjacency.
   Remember the level: a password set for `level-1` only does nothing for L2 Hellos.
3. **Adjacency up but routes missing?** It is **not** the interface password. Compare
   `show isis database` on both sides: missing L1 LSPs → **area-password**; missing L2 LSPs
   → **domain-password**. Confirm with `debug isis update-packets`.
4. **Config conflicts:** legacy password and `authentication mode` on the same scope/level
   cannot coexist; MD5 silently overrides `area-password`/`domain-password`.
5. **Key chain errors:** key ID not numeric, key-string starting with a digit, or keys not
   matching on both ends.
6. **Leftover `send-only`:** authentication looks configured but is not being enforced.

## Common Pitfalls
- **Confusing the three commands.** `isis password` = **interface** (Hellos, interface mode).
  `area-password` = **area** (L1). `domain-password` = **domain** (L2). There is **no "router
  authentication"** in IS-IS — that distractor borrows OSPF-style wording.
- **Expecting a password mismatch to always drop the adjacency.** Only the **interface**
  password does that. Area and domain mismatches leave adjacencies **up** while LSPs are
  silently rejected — the failure looks like a routing problem, not an authentication one.
- **Assuming `domain-password` covers L2 CSNPs/PSNPs.** It covers L2 **LSPs**; SNPs carry the
  password only with `authenticate snp`. (Some practice-exam explanations state it covers
  L2 SNPs unconditionally.)
- **Forgetting the level keyword on `isis password`.** No keyword = both levels; specifying
  one level leaves the other unauthenticated.
- **Enabling enforcement before every neighbor sends authentication** — skip the
  `send-only` step and adjacencies or LSPs drop mid-change.
- **Mixing legacy and new modes on the same scope/level** — not allowed; MD5 overrides the
  legacy passwords.

## Exam Preparation Tasks

### "Do I Know This Already?" question analysis
*(From a Boson ENCOR practice question, not the OCG.)*

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| Which type of authentication is configured with `isis password`? | Mapping each IS-IS auth command to its **scope** — and that `isis password` is the only **interface-mode** one | **C — interface authentication** | **Area** and **domain** are real IS-IS scopes but use `area-password` / `domain-password` under `router isis`. **"Router authentication"** is not an IS-IS scope at all — a plausible-sounding invented term. See Common Pitfalls #1 |
