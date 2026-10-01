---
name: ccnp-encor-v1-2-gap
description: >
  Use this skill when studying the ENCOR 350-401 v1.2 blueprint changes that the
  v1.1-era Official Cert Guide (2nd edition) does not cover. Invoke when the user
  asks about: ENCOR v1.2, v1.1 vs v1.2, what changed in ENCOR, MSDP, Multicast
  Source Discovery Protocol, SA message, Source-Active, peer-RPF, mesh group,
  default peer, Anycast RP, ip msdp peer, TCP 639, bidirectional PIM, bidir-PIM,
  BIDIR, designated forwarder, DF election, RPA, phantom RP, ip pim bidir-enable,
  SSM, Source Specific Multicast, PIM-SSM, 232.0.0.0/8, ip pim ssm, SSM mapping,
  ip igmp ssm-map, Catalyst Center AI, AI-powered workflows, AI Network Analytics,
  AI Endpoint Analytics, Smart Grouping, trust score, Catalyst Center AI Assistant,
  Catalyst Center rename, Catalyst SD-WAN rename, Machine Reasoning Engine, Network
  Reasoner, AI-enhanced RRM, Auto-RP vs BSR vs Anycast RP, sink RP, ip domain multicast,
  SSM mapping DNS, ENCOR v1.2 exam experience, v1.2 study advice, chapter weights.
---

## Purpose
ENCOR **v1.2** went live **2026-03-19**. The ENCOR OCG 2nd edition, which this catalog was
built from, targets **v1.1**. This skill is the delta: every v1.2 objective the book does
not teach, sourced **only from official Cisco documentation** (listed per section and in
Sources). Nothing here is lab-validated; it is doc-sourced.

What v1.2 **removed** (in the book, no longer on the exam): all wireless (old 1.2, 3.3, 5.4),
1.6 hardware/software switching (CEF, CAM, TCAM, FIB, RIB, adjacency), 5.5.e network access
control with 802.1X/MAB/WebAuth, and "wired and wireless" QoS narrowed to "Interpret QoS
configurations." The `wireless-*` skills in this catalog are therefore **not v1.2 exam
material**.

## Key Concepts

### Blueprint diff: v1.1 → v1.2 (from Cisco's two blueprint PDFs)
| v1.2 objective | What changed | Covered by |
|---|---|---|
| **3.3.d Multicast:** "RPF check, PIM SM, IGMP v2/v3, **SSM, bidir, and MSDP**" | v1.1 was "RPF check, PIM and IGMP v2/v3". **SSM, bidir, MSDP are new** | This skill + `multicast` |
| **4.5 Catalyst Center:** "…using traditional and **AI-powered workflows**" | v1.1: "Describe Cisco DNA Center workflows…" | This skill + `network-assurance`, `fabric-technologies` |
| 1.2 Catalyst SD-WAN, 6.4 "Catalyst Center and SD-WAN Manager", 6.0 "Automation and **Artificial Intelligence**" | **Renames only** | Name table below |
| 3.3.d wording | "PIM" became **"PIM SM"** — PIM-DM is no longer named | `multicast` |
| 1.4 QoS | "Interpret **wired and wireless** QoS configurations" + sub-bullets (components, policy) → just "Interpret QoS configurations" | `qos` |
| 6.7 | "…orchestration tools, **such as Chef, Puppet, Ansible, and SaltStack**" → example list **deleted** (but see the test-taker sample: candidates still report questions on those four) | `automation-tools` |
| 6.0 title | "Automation" → "Automation **and Artificial Intelligence**" — **no new AI objective in section 6**; the only AI content in the whole blueprint is 4.5 | — |

*Verified line-by-line against both official PDFs (2026-10-01): everything else in v1.2 —
including STP enhancements, NTP **and PTP**, OSPF **area types**, and "local user
authentication" — is **unchanged from v1.1**. The genuinely new technical content is only
**SSM, bidir, MSDP** and **AI-powered Catalyst Center workflows**. Chapter-by-chapter study
weights: [`ENCOR-v1.2-CHAPTER-WEIGHTS.md`](../../../ENCOR-v1.2-CHAPTER-WEIGHTS.md).*

**Rename table:** DNA Center → **Catalyst Center** · Cisco SD-WAN → **Catalyst SD-WAN** ·
vManage → **SD-WAN Manager** · vSmart → **Controller** · vBond → **Validator** ·
vEdge/cEdge → **Edge**. (SD-WAN names are also in `fabric-technologies`.)
Per Cisco's release notes, the SD-WAN renames took effect in **Catalyst SD-WAN 20.12.1 /
IOS XE SD-WAN 17.12.1a**, and also include **vAnalytics → SD-WAN Analytics** and **"Cisco
Controllers" → Catalyst SD-WAN *Control Components*** (the umbrella term for Manager +
Controller + Validator).

### MSDP — Multicast Source Discovery Protocol
- **What it solves:** "a mechanism to connect multiple PIM-SM domains." It "allows a
  rendezvous point (RP) to dynamically discover active sources outside of its domain." Each
  domain keeps its own RP; MSDP lets the RPs tell each other about sources.
- **Transport:** RPs peer over **TCP port 639**, configured explicitly like BGP neighbors.
- **Source-Active (SA) messages** advertise active sources. They carry **the originating
  RP's IP address plus one or more (S,G) pairs**, and may encapsulate the first data packet.
  Four message types: SA, SA request, SA response, keepalive.
- **Timers:** keepalive every **60 s**; session reset if nothing is heard for **75 s**
  (hold); connection retry **30 s**.
- **Peer-RPF check:** an arriving SA is accepted only from the peer on the path back toward
  the originating RP. MSDP uses **(M)BGP routing data** to work that out, so MSDP normally
  depends on BGP. **BGP is not required** with a **mesh group**, a **default peer**, or when
  only **one MSDP peer** is configured.
- **Mesh group:** fully meshed MSDP peers. Optimizes SA flooding and **eliminates RPF checks
  on arriving SA messages** (members don't re-forward SAs to each other).
- **Default peer:** accepts all SAs **without the peer-RPF check**. Intended for a stub or
  nontransit AS.
- **Anycast RP:** several RPs share one RP address; MSDP between them (usually a mesh group)
  keeps their source lists in sync. `ip msdp originator-id` makes each RP originate SAs from
  its own unique interface address instead of the shared anycast address.
- **Security:** MD5 password authentication on the TCP session.

### Choosing an RP mechanism — where MSDP and Anycast RP fit
Cisco's RP white paper compares the four recommended PIM-SM RP designs:

| Design | How RP info reaches routers | RP-failure convergence | Notes |
|---|---|---|---|
| **Auto-RP** (multiple RPs) | Mapping agent floods via **224.0.1.39 / 224.0.1.40** in dense mode | **~3 minutes** | Needs sparse-dense mode, plus a **"sink RP"** (fictitious static RP) so groups don't fall back to dense |
| **BSR** (multiple RPs) | PIM messages, hop by hop (link-local) | **~3 minutes** | Pure sparse mode, **no dense-mode reversion risk**; **TTL scoping can't be used** |
| **Anycast static RP** | Same /32 RP address on several RPs; **MSDP** shares sources | **Seconds** (IGP convergence) | **IGP must carry host (/32) routes** |
| **Anycast RP + Auto-RP** | Auto-RP announces the anycast address; MSDP **mesh group** stops SA looping | **Seconds** | Still needs the sink RP |

**Anycast RP details (Cisco Anycast RP white paper):**
- Each source and receiver uses the **topologically closest** RP, so registrations spread
  across RPs. If an RP fails, unicast routing converges and the remaining RPs take over —
  **existing multicast sessions are unaffected**, because "the RP is normally needed only to
  start new sessions."
- **Router-ID trap:** a router may choose the shared anycast address as its router ID. Set
  the router ID manually (Cisco: to the MSDP peering address), and keep `ip msdp
  originator-id` pointing at a **unique** interface.
- The MSDP peering address must differ from the anycast RP address.

### Bidirectional PIM (bidir-PIM, RFC 5015)
- "A variant of PIM Sparse mode that builds **bidirectional** multicast trees between
  sources and receivers **without maintaining any source-specific state**." Built for
  **many-to-many** applications, where per-source (S,G) state would not scale.
- **Versus PIM-SM:** **no (S,G) state, no source trees, no SPT switchover, no register
  encapsulation** to the RP. One shared tree rooted at the RP carries traffic **both** up
  toward the RP and down to receivers.
- **Designated Forwarder (DF):** one DF **per RP, per link**. It is the router with the
  **best unicast route to the RP address** (compared via MRIB metrics). The DF forwards
  traffic downstream onto its link and upstream from its link toward the RP; non-DF routers
  discard. DF election is what keeps the bidirectional tree **loop-free**.
- **RP address (RPA) need not be a real router:** it "can be any unassigned IP address on a
  network that is reachable throughout the PIM domain" (the "phantom RP" design).
- In `show ip mroute`, bidir groups carry the **B** flag; `show ip pim neighbor` shows
  bidir-capable neighbors with mode **B**.

### SSM — Source Specific Multicast
- Receivers join a specific **(S,G) channel**, not just a group. "Only source-specific
  multicast distribution trees (**not shared trees**) are created."
- **Range:** IANA reserved **232.0.0.0–232.255.255.255** for SSM.
- **No RP at all.** PIM-SSM is derived from PIM-SM but "does not require an RP, so there is
  no need for an RP mechanism such as Auto-RP, **MSDP**, or BSR."
- **IGMPv3 is required** for native SSM: hosts signal the source with INCLUDE-mode reports,
  and "only **INCLUDE mode** reports are accepted by the last-hop router" for SSM groups.
- **Benefits:** different sources can **reuse** the same group address; traffic from a source
  crosses the network **only if requested** (DoS resistance); no source tracking to operate.
- **SSM mapping** — for hosts that can only do IGMPv1/v2: the last-hop router maps the group
  to a source statically (`ip igmp ssm-map static`) or by DNS lookup, then joins (S,G) on the
  host's behalf.
- **SSM mapping detail (IOS XE 17 SSM Mapping guide):**
  - Exists for **legacy set-top boxes and apps without IGMPv3**. Static mapping uses an ACL
    of groups → source; **DNS mapping is on by default** once `ip igmp ssm-map enable` is set.
  - DNS lookup name is the **reversed group address + domain**: group 232.1.1.4 →
    `4.1.1.232.<domain>`, returning A records (the sources). Domain prefix via
    `ip domain multicast <domain>` (default `in-addr.arpa`).
  - **Up to 20 sources per group**, and only **one application per group G** (an IGMPv1/v2
    report cannot say which source it wants).
  - SSM's DoS protection does **not** extend to the LAN behind the last-hop router, which
    still runs IGMPv1/v2. Cisco warns to enable IGMPv3 "with care" on a last-hop router that
    relies only on SSM mapping.
  - `ip igmp static-group <group> source ssm-map` statically forwards a mapped channel.
  - `show ip igmp groups detail` marks mapped sources with the **M** flag; `debug ip igmp`
    shows `Convert IGMPv2 report (*,G) to IGMPv3 with N source(s) using STATIC|DNS` or
    `DNS source lookup failed`.
- Legacy Cisco transition options on the older doc: **IGMP v3lite** and **URD** (URD
  intercepts TCP 465). Likely low exam value.

### Catalyst Center — traditional vs AI-powered workflows (4.5)
- **Traditional workflows** = the Design → Policy → Provision → Assurance menus, already in
  `fabric-technologies` (Catalyst Center workflow table) and `network-assurance`.
- **Cisco AI Network Analytics** (inside Assurance): "machine learning and machine reasoning"
  to **baseline** what is normal for *your* sites, detect **anomalies** (onboarding: DHCP,
  AAA, association failures; application throughput), show **trends and insights**,
  **network heatmaps / peer comparison**, and walk through **root-cause troubleshooting
  steps**. Event data is **de-identified** in Catalyst Center and sent **encrypted to Cisco's
  cloud** for the ML; results come back into Assurance. Needs the **Advantage** license and
  HTTPS cloud connectivity.
- **Cisco AI Endpoint Analytics:** endpoint and IoT **profiling/visibility**. Labels each
  endpoint by type, hardware model, manufacturer, OS. Telemetry from **NBAR deep packet
  inspection on Catalyst 9000 access switches**, **ISE** (via pxGrid), ServiceNow CMDB,
  Catalyst 9800 WLCs, and traffic telemetry appliances. AI parts: **Smart Grouping** (ML
  clusters unknown endpoints and proposes profiling rules), **AI spoofing detection**, and a
  **trust score 1–10**. Profiles and trust scores publish to **ISE**, which can use them in
  authorization policy and **ANC** actions (quarantine, port bounce, reauth).
- **Catalyst Center AI Assistant:** a **generative-AI conversational** interface for
  **monitoring, troubleshooting, and documentation** questions. Enabled by registering with
  Cisco Catalyst Cloud; available on **all license tiers**, though what it can do follows the
  licensed features underneath. Data is processed in the US and **not used to train** the
  model; Cisco says to **validate its suggestions**, especially for critical changes.

- **Machine Reasoning Engine (MRE) / Network Reasoner** — the other "AI-powered workflow"
  family. MRE "uses artificial intelligence to automate complex network operation
  workflows," running **knowledge packs** (step-by-step workflows from a **cloud-hosted
  knowledge base** of Cisco expertise) against live device output for **automated root-cause
  analysis**. Launched from **Tools > Network Reasoner**, or from an Assurance issue or the
  Inventory. Workflows include **CPU utilization, power supply, interface down, network /
  IP connectivity (ping), Layer 2 loop, PoE, wired client, wireless AP/client, controller
  HA/SSO, SD-Access readiness and fabric health, scale limits**. Requires the Machine
  Reasoning package and MRE write permission; devices typically **IOS XE 16.9.3+**.
- **Full AI feature list (Catalyst Center 3.3.1 data sheet):** AI Assistant · **AI-enhanced
  RRM** (wireless — learns RF trends) · AI Network Analytics · Machine Reasoning Engine ·
  **AI-driven predictive insights** (e.g. predicting Wi-Fi interference, onboarding delays,
  traffic load) · **AI agents (Cisco Cloud Control)** for cross-domain troubleshooting ·
  AI Endpoint Analytics. *Distinguish them by job:* **analytics** = learn a baseline and flag
  anomalies; **reasoning (MRE)** = run an expert's diagnostic steps; **assistant** = natural-
  language Q&A; **endpoint analytics** = identify what a device is.

## What v1.2 test-takers report (Reddit sample, 2026-10-01)
*Not documentation. 27 r/ccnp and r/ccna threads posted after v1.2 went live (2026-03-19), of
which ~9 contain **first-hand** v1.2 exam accounts. Self-reported, NDA-limited, one community —
treat as weak signal, and **the blueprint always wins**. Exam forms vary between candidates.*

| What candidates reported | How many | Notes |
|---|---|---|
| **Automation was the hardest / heaviest area** — Python script reading, JSON syntax, **EEM**, and "the difference between ansible and puppet" asked repeatedly | 3–4 | Chef/Puppet/Ansible/SaltStack were **removed from the 6.7 wording but still asked** |
| **SD-Access component functions, Catalyst Center APIs** (north/south/east/westbound), **SD-WAN, LISP/VXLAN** | 3–4 | "More of a memory game" |
| **Labs ("simlets") = routing & switching configuration**: OSPF (several ways to do the same task), BGP, EIGRP, ACL edits, STP, IP SLA, EtherChannel, SPAN, tunnels | 3 | One candidate: "**less than 60 questions and 6 labs**"; another: labs "dead simple… finished them all within 10 mins" |
| **Every blueprint item marked *configure*** must be buildable from a blank device (CoPP, RSPAN, AAA, OSPF summarization, eBGP via loopbacks) | 2 (top-voted) | Matches the blueprint verbs |
| **Wireless:** "zero wireless topics" vs "at least 5 wireless questions" | 1 vs 1 | **Contradictory** — the blueprint lists none; don't study it |
| Little **IPv6** depth compared with v1.1 | 1 | Single account |
| **Multicast SSM/bidir/MSDP** | 0 first-hand | Mentioned only as study advice ("greater depth to multicast… White papers are your best bet") — **no taker reported them appearing**. Absence of a report is not absence from the exam |
| **Boson ExSim is harder and less representative** than the real exam ("never got a passing score in EXSIM and still passed"; "not at all representative") | 3 | Consistent with Boson asking off-blueprint items (e.g. IS-IS — see the `is-is` skill) |
| **Score report from a fail (2026-09-29):** Automation 47%, **Virtualization 10%**, Infrastructure 60%, Assurance 100%, Security 65%, Architecture 67% | 1 (+1 "also low on virtualization") | Virtualization (VRF, GRE/IPsec, LISP, VXLAN) is a quiet failure point |

**Study resources candidates mention:** the OCG has **not** been updated for v1.2 ("just skip
the wireless sections"); Cisco U content also still v1.1 per one poster; **Cisco white papers**
for multicast depth; CML/EVE-NG over Packet Tracer; retake wait after a fail is **5 days**.

**Practical takeaways:** (1) treat automation (Python/JSON/EEM/orchestration comparisons) as a
full-weight domain; (2) drill every *configure* objective as a from-scratch lab verified with
show commands; (3) know SD-Access/SD-WAN component roles and Catalyst Center API directions
cold; (4) learn SSM/bidir/MSDP to the blueprint's verb — **describe** — not configuration depth.

## Config Patterns
```
! ---------- MSDP between two domains' RPs ----------
ip multicast-routing
ip msdp peer 192.0.2.2 connect-source Loopback0
ip msdp sa-filter in 192.0.2.2 list MSDP-SA-IN   ! optional SA filtering
! ip msdp default-peer 192.0.2.2                  ! stub AS: skip peer-RPF / no BGP

! ---------- Anycast RP: two RPs share 198.51.100.1 ----------
interface Loopback1
 ip address 198.51.100.1 255.255.255.255        ! shared anycast RP address
ip pim rp-address 198.51.100.1
ip msdp peer 192.0.2.2 connect-source Loopback0
ip msdp mesh-group ANYCAST-RP 192.0.2.2
ip msdp originator-id Loopback0                 ! SAs sourced from the unique loopback

! ---------- Bidirectional PIM ----------
ip pim bidir-enable
ip access-list standard BIDIR-GROUPS
 permit 225.0.0.0 0.255.255.255
ip pim rp-address 198.51.100.10 BIDIR-GROUPS bidir

! ---------- SSM ----------
ip pim ssm default                               ! 232.0.0.0/8
interface GigabitEthernet0/1
 ip pim sparse-mode
 ip igmp version 3                               ! required for SSM receivers
! SSM mapping for IGMPv1/v2-only hosts
ip igmp ssm-map enable
no ip igmp ssm-map query dns
ip igmp ssm-map static SSM-MAP-ACL 203.0.113.10
```
Addresses are RFC 5737 documentation ranges. Syntax is from Cisco's IOS XE 17 guides;
**not run on gear**.

## Verification Commands
| Command | What to look for |
|---|---|
| `show ip msdp peer [addr]` / `show ip msdp summary` | Peer state (Up), uptime, SA counts |
| `show ip msdp sa-cache` | (S,G) sources learned from other domains, with the originating RP |
| `show ip msdp count` | SA message counts per peer/AS |
| `show ip mroute` | Bidir groups flagged **B**; SSM groups have (S,G) entries and no (*,G)/RP |
| `show ip pim neighbor` | Mode **B** = bidir-capable neighbor |
| `show ip pim interface df` | DF winner per RP on each interface |
| `show ip igmp groups detail` | IGMPv3 INCLUDE-mode source lists on the receiver interface |
| `show ip igmp ssm-mapping [group]` | Which source a group maps to for v1/v2 hosts |

## Common Pitfalls
- **"SSM needs an RP / MSDP."** It needs neither. MSDP exists to share sources *between RPs*
  in PIM-SM; SSM has no RP.
- **Forgetting MSDP's BGP dependency.** Without BGP, SAs fail the peer-RPF check, unless
  it's a mesh group, default peer, or single peer.
- **Mixing up MSDP and Anycast RP.** Anycast RP is the *design* (shared RP address); MSDP is
  the *protocol* that keeps those RPs' source lists in sync.
- **Expecting (S,G) state or SPT switchover in bidir.** Bidir has only (*,G) on one shared
  tree; the DF, not an SPT, prevents loops.
- **Bidir DF vs PIM DR.** The DF is per RP per link and is chosen by best route to the RP; the
  DR is per LAN for PIM-SM joins/registers.
- **SSM groups outside 232/8.** `ip pim ssm default` only covers 232.0.0.0/8; other ranges need
  `ip pim ssm range <acl>`.
- **IGMPv2 receivers in SSM.** They can't name a source; use IGMPv3 or SSM mapping.
- **"AI Network Analytics runs on the appliance."** The ML runs in **Cisco's cloud** on
  de-identified data, and it needs the Advantage license.

## Sources (official Cisco)
- [ENCOR v1.2 blueprint (PDF)](https://learningcontent.cisco.com/documents/marketing/exam-topics/350-401-ENCORE-v1.2.pdf) · [ENCOR v1.1 blueprint (PDF)](https://learningcontent.cisco.com/documents/marketing/exam-topics/350-401-ENCORE-v1.1.pdf)
- [IP Multicast Configuration Guide, IOS XE 17.x — Using MSDP to Interconnect Multiple PIM-SM Domains](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-multicast/b-ip-multicast/m_imc_msdp_im_pim_sim.html)
- [Catalyst 9000 Multicast Configuration Guide — MSDP](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/multicast/multicast-configuration-guide/msdp.html)
- [IP Multicast Configuration Guide, IOS XE 17 (ASR 900) — Bidirectional PIM](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipmulti_pim/configuration/xe-3s/asr903/17-1-1/b-imc-pim-xe-17-1-asr900/m-bidirectional-pim.html) (platform restrictions on this page are ASR 900–specific)
- [IP Multicast Routing Configuration Guide, IOS XE 17.12 (Catalyst 9500) — Configuring PIM](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9500/software/release/17-12/configuration_guide/ip_mcast_rtng/b_1712_ip_mcast_rtng_9500_cg/configuring_pim.html) (bidir DF election on Catalyst)
- [IP Multicast Configuration Guide, IOS XE 17.x — Configuring Source Specific Multicast](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-multicast/b-ip-multicast/m_imc_ssm.html)
- [Catalyst 9000 Multicast Configuration Guide — SSM (incl. SSM mapping)](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/multicast/multicast-configuration-guide/ssm.html)
- [Catalyst Assurance User Guide 3.1.x — Cisco AI Network Analytics Overview](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/catalyst-center-assurance/3-1-x/b_cisco_catalyst_assurance_3_1_x_ug/b_cisco_catalyst_assurance_3_1_x_ug_chapter_010.html)
- [Catalyst Center User Guide 3.1.x — Cisco AI Endpoint Analytics](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/catalyst-center/3-1-x/user_guide/b_cisco_catalyst_center_user_guide_3_1_x/endpoint-analytics-1-0.html)
- [Cisco white paper — Anycast RP](https://www.cisco.com/c/en/us/td/docs/ios/solutions_docs/ip_multicast/White_papers/anycast.html)
- [Cisco white paper — Configuring a Rendezvous Point](https://www.cisco.com/c/en/us/td/docs/ios/solutions_docs/ip_multicast/White_papers/rps.html) (Auto-RP vs BSR vs Anycast RP)
- [IP Multicast Configuration Guide, IOS XE 17.x — SSM Mapping](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/ip-multicast/b-ip-multicast/m_imc_ssm_map-0.html)
- [Catalyst Center User Guide 3.1.x — Troubleshoot Network Devices Using Network Reasoner](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/catalyst-center/3-1-x/user_guide/b_cisco_catalyst_center_user_guide_3_1_x/m_troubleshoot_network_device_using_network_reasoner.html)
- [Cisco Catalyst Center 3.3.1 Data Sheet](https://www.cisco.com/c/en/us/products/collateral/networking/catalyst-center-3-3-1-ds.html) (AI feature list)
- [Release Notes, Catalyst SD-WAN Control Components 20.12.x](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/release/notes/controllers-20-12/rel-notes-controllers-20-12.pdf) (official rename list)
- [Cisco Catalyst Center AI Assistant](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/network-automation-and-management/catalyst-center/articles/cisco-catalyst-center-ai-assistant.html)
