# CCNP Skills — ENCOR + ENARSI

A personal CCNP knowledge base, captured as I study and structured as installable
[Claude Code](https://claude.com/claude-code) skills rather than static notes.

Two catalogs live here:

| Catalog | Exam | Status | Path |
|---|---|---|---|
| **ENCOR** | 350-401 | **Complete** — all 6 domains, 32 topic skills, OCG Chapters 1–29 | `1.0.0/skills/` |
| **ENARSI** | 300-410 | **In progress** — just started | `1.0.0/skills-enarsi/` |

Each topic I study gets turned into a `SKILL.md` file — purpose, key concepts, real
IOS-XE config patterns, verification commands, and an ordered troubleshooting
checklist — generated live via a custom `/ccnp-note <topic>` (ENCOR) or
`/enarsi-note <topic>` (ENARSI) slash command as I work through the material. The result
is a knowledge base that's both human-readable and directly usable by an AI coding agent
for config review, troubleshooting walkthroughs, or lab work.

I'm a network engineer (NOC tech → network engineer) moving deeper into network
security and automation (NetDevOps). This repo is part of that work in public —
more written up on [LinkedIn](https://www.linkedin.com/in/stefan-c-reed/) as I go.

## What's next — a troubleshooting agent

This catalog is phase 1: one self-contained skill per topic. Phase 2 is combining
these into a single agent that can diagnose real network problems across multiple
topic areas at once — the way an experienced engineer actually troubleshoots, not
one isolated skill at a time.

That phase 2 work converges with another project of mine,
[netmiko-config-audit](https://github.com/stefcharreed/netmiko-config-audit) — a
config drift tool that already exposes an MCP server. Rather than building a
separate AI layer for each project, phase 2 composes this skill catalog's
knowledge with that tool's MCP server to reason over real config drift, not just
static topics. It's happening in a private repo since it'll likely involve real
device interaction patterns.

If that's interesting to you, [message me on LinkedIn](https://www.linkedin.com/in/stefan-c-reed/) — happy to talk through it.

## Design update (2026-07-16) — Design Baseline + intent-first troubleshooting

Every topic skill gained two sections, prompted by feedback on LinkedIn from
people with real production scar tissue:

- **Intent Questions + checklist step 0** — credit to **Stephan (Steve) M.,
  Sr. Network Engineer**: troubleshooting starts with "what is it supposed to
  do, and what isn't it doing?" Every troubleshooting checklist now opens
  with intent vs. observed, before any `show` command. He also suggested
  integrating Cisco validated designs as a best-practice layer — that became
  the Design Baseline section.
- **Design Baseline with a "legitimate reasons to deviate" column** — credit
  to **Phil Lafontaine, Solutions Engineer @ Cisco**: in real networks there
  are reasons, both good and bad, why something isn't configured to best
  practice. So a deviation from the baseline is a *question* ("is this
  intentional here?"), never automatically a finding. Every baseline row
  cites a named source — no source, no row.

This is exactly what working in public is for — thanks to both.

## Design update (2026-09-13) — Exam Preparation Tasks

Topic skills built from OCG chapters now carry an **Exam Preparation Tasks**
section with two parts:

- **Key topics coverage map** — the chapter's own "Key Topics" table reproduced
  with a fourth column pointing at where in the skill each element is covered.
  The mapping is deliberately not 1:1: one table or bullet often satisfies
  several key topics, and one key topic can be split across sections. A row
  that can't be mapped is marked `gap`, so the map doubles as a coverage audit
  of the skill itself.
- **"Do I Know This Already?" question analysis** — for each quiz question,
  what it's actually testing underneath the wording and which misconception the
  distractors were built to catch. The valuable ones are distractors naming a
  real technology that belongs to a *different* solution (EVPN/MP-BGP offered
  as an SD-Access control plane; "aggregation layer" offered for the access
  layer); those get promoted to Common Pitfalls too.

Older skills in this catalog predate the section and don't carry it yet — it
gets added as each chapter's source material is revisited.

## How it works

- `/ccnp-note <topic>` (a Claude Code skill) takes raw study notes, textbook excerpts,
  config blocks, or `show` command output and writes a structured skill file to
  `1.0.0/skills/<topic>/SKILL.md`.
- `/enarsi-note <topic>` does the same for ENARSI, writing to
  `1.0.0/skills-enarsi/<topic>/SKILL.md`. It deliberately reuses `ccnp-note`'s template by
  reference rather than copying it, so the two catalogs can't drift apart — and it checks
  for an existing ENCOR skill on the same protocol first, so ENARSI captures the delta
  instead of silently duplicating.
- Every skill follows the same template: Purpose, Key Concepts, Config Patterns,
  Design Baseline (best practices with sources — and legitimate reasons to
  deviate), Verification Commands, Intent Questions, Troubleshooting Checklist
  (step 0: intent vs. observed), Common Pitfalls, and Exam Preparation Tasks.
- Every new or updated skill is committed and pushed immediately — see
  [CLAUDE.md](CLAUDE.md) for the repo's working rules.

## Roadmap — ENCOR exam domains

**Status: complete.** The full ENCOR (350-401) Official Cert Guide has been read and captured,
Chapters 1–29 (Chapter 30 is Final Preparation — no technical content). All six blueprint
domains below are covered by skills in this catalog.

Two topics appear in Cisco's blueprint but have **no standalone OCG chapter**, so there is no
chapter to capture them from — they are noted here rather than tracked as outstanding work:

- **DHCP** — appears only incidentally in the OCG (PnP onboarding, ZBFW ACLs, CWA step 4).
- **First-hop security (port security, DHCP snooping, DAI)** — CCNA/SWITCH-level material the
  OCG assumes. Adding it would need a non-OCG source.

**ENCOR v1.2 (live 2026-03-19).** The OCG above targets v1.1. v1.2 **removed all wireless**
(the `wireless-*` skills below, marked *not on v1.2*) and hardware/software switching, and
**added** multicast SSM/bidir/MSDP and Catalyst Center AI-powered workflows. Those additions are
captured from official Cisco docs in:

- [x] [ENCOR v1.2 gap](1.0.0/skills/encor-v1-2-gap/SKILL.md) — v1.1→v1.2 blueprint diff, MSDP, bidir-PIM, SSM & SSM mapping, Catalyst Center AI (AI Network Analytics, AI Endpoint Analytics, AI Assistant), Catalyst renames

### 1. Architecture (15%)
- [x] [Enterprise Network Architecture](1.0.0/skills/enterprise-network-architecture/SKILL.md) — hierarchical LAN design (access/distribution/core), network blocks & PINs, HA design, SSO/NSF with GR and NSR, two-tier vs three-tier, Layer 2 vs routed access, simplified campus design (VSS/SWV/StackWise)
- [x] [Fabric Technologies](1.0.0/skills/fabric-technologies/SKILL.md) — SD-Access (LISP control plane, VXLAN-GPO data plane, TrustSec policy plane, fabric roles, DNA Center layers & workflows) and SD-WAN (vManage/vSmart/vBond/edge, OMP, AAR, Cloud OnRamp)

### 2. Virtualization (10%)
- [x] [Overlay Tunnels](1.0.0/skills/overlay-tunnels/SKILL.md) — GRE, IPsec (AH/ESP, IKEv1/IKEv2), GRE over IPsec & VTI, LISP, VXLAN
- [x] [Virtualization](1.0.0/skills/virtualization/SKILL.md) — server virtualization (VMs, Type 1/2 hypervisors, containers & container engines, vSwitches and distributed switching, Docker0 bridging), the ETSI NFV framework (NFVI, VNF, VIM, EM, MANO, OSS/BSS, service chaining), VNF I/O performance (the 11-step OVS path, OVS-DPDK, PCI passthrough, SR-IOV with VFs/PFs and VEB/VEPA), and Cisco ENFV (DNA Center MANO, NFVIS, ENCS / Catalyst 8200 uCPE)

### 3. Infrastructure (30%)
- [x] [VLANs](1.0.0/skills/VLANS/SKILL.md) — 802.1Q, access/trunk ports, native/allowed VLANs
- [x] [MAC addresses & Layer 2 forwarding](1.0.0/skills/mac-addresses/SKILL.md) — MAC table, CAM, collision domains
- [x] [Layer 3 forwarding](1.0.0/skills/layer-3-forwarding/SKILL.md) — ARP, routing table, SVIs, routed ports
- [x] [Spanning Tree Protocol (802.1D)](1.0.0/skills/spanning-tree-protocol/SKILL.md) — port states/roles, root election, TCNs
- [x] [Rapid Spanning Tree (802.1W)](1.0.0/skills/rapid-spanning-tree-protocol/SKILL.md) — discarding state, proposal/agreement
- [x] [Advanced STP tuning](1.0.0/skills/advanced-stp-tuning/SKILL.md) — root guard, BPDU guard/filter, loop guard, UDLD
- [x] [Multiple Spanning Tree (802.1S)](1.0.0/skills/mst-protocol/SKILL.md) — MSTIs, IST, MST regions, PVST simulation
- [x] [VLAN Trunks & EtherChannel bundles](1.0.0/skills/vlan-trunks-and-etherchannel/SKILL.md) — VTP, DTP, LACP/PAgP, load-balancing hashes
- [x] [IP Routing Essentials](1.0.0/skills/ip-routing-essentials/SKILL.md) — RIB/FIB, AD, static routes, PBR, VRF
- [x] [EIGRP](1.0.0/skills/eigrp/SKILL.md) — DUAL, successor/feasible successor, wide metrics, query/reply convergence
- [x] [OSPF](1.0.0/skills/ospf/SKILL.md) — LSDB, SPF, DR/BDR election, network types, neighbor adjacency
- [x] [Advanced OSPF](1.0.0/skills/advanced-ospf/SKILL.md) — multi-area design, ABRs, LSA types 1-7, discontiguous networks, summarization, route filtering
- [x] [OSPFv3](1.0.0/skills/ospfv3/SKILL.md) — OSPF for IPv6, link-local adjacencies, link/intra-area prefix LSAs, IPv4 support
- [x] [IS-IS](1.0.0/skills/is-is/SKILL.md) — *(no standalone OCG chapter; SD-Access underlay IGP, tested in practice exams)* NET/system ID/areas, L1/L2/L1-L2 adjacency matrix, is-type vs circuit-type, DIS, hello padding & MTU mismatch (INIT/ES-IS), metric-style narrow/wide/transition, ATT bit & L2→L1 route leaking, LDP-IGP sync, interface/area/domain authentication & HMAC-MD5, troubleshooting ordered by a field sample of real failures
- [x] [BGP](1.0.0/skills/bgp/SKILL.md) — eBGP/iBGP, path attributes, AS_Path loop prevention, route summarization, MP-BGP for IPv6
- [x] [Advanced BGP](1.0.0/skills/bgp-advanced/SKILL.md) — multihoming, transit AS avoidance, route maps, AS_Path filtering, BGP communities, best-path selection
- [x] [Multicast](1.0.0/skills/multicast/SKILL.md) — IGMP, IGMP snooping, PIM dense/sparse mode, RPF, rendezvous points, Auto-RP, BSR
- [x] [Quality of Service (QoS)](1.0.0/skills/qos/SKILL.md) — DiffServ/IntServ, DSCP & PHBs, MQC, trust boundary, policing/shaping, token buckets, srTCM/trTCM, CBWFQ/LLQ, WRED
- [x] [IP Services](1.0.0/skills/ip-services/SKILL.md) — NTP stratums & peers, PTP/IEEE 1588, FHRP (HSRP/VRRP/GLBP), object tracking, NAT/PAT
- [x] *(not on v1.2)* [Wireless Signals & Modulation](1.0.0/skills/wireless-signals-and-modulation/SKILL.md) — RF fundamentals, dB/dBm/EIRP, free space path loss, RSSI/SNR, modulation, MIMO, DRS
- [x] *(not on v1.2)* [Wireless Infrastructure](1.0.0/skills/wireless-infrastructure/SKILL.md) — autonomous/centralized/cloud/distributed/EWC deployments, AP modes, CAPWAP discovery & join, profiles and tags, antennas
- [x] *(not on v1.2)* [Wireless Roaming & Location Services](1.0.0/skills/wireless-roaming-and-location-services/SKILL.md) — intra/intercontroller roaming, Layer 2 vs Layer 3 roams, anchor/foreign controllers, mobility groups & domains, RSS trilateration, RF fingerprinting
- [x] *(not on v1.2)* [Troubleshooting Wireless Connectivity](1.0.0/skills/wireless-troubleshooting/SKILL.md) — scoping by report pattern, Client 360 View, Radioactive Trace, AP join statistics, channel utilization, AP operational config

### 4. Network Assurance (10%)
- [x] [Network Assurance](1.0.0/skills/network-assurance/SKILL.md) — ping/traceroute/debug discipline, SNMP (MIB structure, OIDs, communities, traps), syslog severities & destinations, NetFlow and Flexible NetFlow (records/exporters/monitors/samplers), SPAN/RSPAN/ERSPAN, IP SLA, Cisco DNA Center Assurance (Network Time Travel, Client 360, Path Trace)

### 5. Security (20%)
- [x] *(not on v1.2)* [Authenticating Wireless Clients](1.0.0/skills/wireless-client-authentication/SKILL.md) — Open Auth, PSK/WPA personal, WPA3 SAE, EAP & 802.1x roles, four-way EAPOL handshake, key hierarchy, WebAuth (LWA/CWA)
- [x] [Secure Network Access Control](1.0.0/skills/secure-network-access-control/SKILL.md) — Cisco SAFE (PINs, secure domains, attack continuum), the Cisco Secure portfolio (Talos, Secure Malware Analytics, AMP, Secure Client, Umbrella, WSA, ESA, Secure IPS/NGFW, Secure Network & Cloud Analytics, ISE/pxGrid), and NAC (802.1x/EAP methods, EAP chaining, MAB, LWA/CWA WebAuth, IBNS 2.0, TrustSec SGT/SXP/SGACL, MACsec)
- [x] [Network Device Access Control & Infrastructure Security](1.0.0/skills/network-device-access-control/SKILL.md) — ACLs on interfaces & vty (`access-class`), CLI access methods, password types 0/5/7/8/9, privilege levels & RBAC, `transport input`, SSH v1/v2, EXEC & absolute timeouts, AAA framework, TACACS+ vs RADIUS, the 11-step TACACS+ device-access config, ZBFW (zones, self/default zones, drop/pass/inspect, zone pairs), CoPP, device hardening

### 6. Automation (15%)
- [x] [Foundational Network Programmability](1.0.0/skills/network-programmability-foundations/SKILL.md) — CLI limits, APIs (Northbound/Southbound, REST, HTTP methods & CRUD, status codes), XML vs JSON, Postman, Cisco DNA Center & vManage APIs (Token API, X-Auth-Token, JSESSIONID, limit/offset), data models (YANG, NETCONF, RESTCONF), DevNet, GitHub, and basic Python (modules, dictionaries, functions, conditions)
- [x] [Automation Tools](1.0.0/skills/automation-tools/SKILL.md) — on-box EEM (event detectors, applets, events/actions, Tcl scripts, email variables) and off-box config management: agent-based Puppet (modules/manifests, PuppetDB, Forge), Chef (cookbooks/recipes, knife, OHAI, kitchen) and SaltStack (masters/minions, pillars/grains, reactors/beacons, 0MQ); agentless Ansible (playbooks/plays/tasks, YAML, inventory, ansible-vault, PPDIOO), Puppet Bolt and Salt SSH

## Roadmap — ENARSI exam domains

**Status: just started.** Source: *CCNP Enterprise Advanced Routing ENARSI 300-410 Official
Cert Guide*, **2nd edition** (Lacoste & Edgeworth, 2024) — 23 technical chapters, plus Ch. 24
(Final Preparation) and Ch. 25 (Exam Updates). The book targets **ENARSI v1.1**, which matches
Cisco's exam page (checked 2026-09-29). Blueprint numbers below follow the book's own
**Table I-1** exam-topic → chapter map. Skills land
in `1.0.0/skills-enarsi/`, one per chapter, generated with `/enarsi-note`.

ENARSI revisits protocols ENCOR already covers, at greater depth and with a troubleshooting
slant. Where a chapter overlaps an ENCOR skill (marked **↔**), the ENARSI skill captures **the
delta** and cross-references the ENCOR skill for the fundamentals rather than duplicating it.

### 1. Layer 3 Technologies (35%)
- [ ] **Ch. 1 — IPv4/IPv6 addressing & routing review** — DHCPv4, SLAAC / stateful / stateless
  DHCPv6, packet forwarding, AD, static routes · blueprint 1.1, 4.4 · ↔ `ip-routing-essentials`,
  `ip-services` (DHCP client / PPPoE note moves here)
- [ ] **Ch. 2 — EIGRP** — classic vs named mode, path metric calculation · 1.9.a–b, 1.9.e–f (load balancing, metrics) · ↔ `eigrp`
- [ ] **Ch. 3 — Advanced EIGRP** — timers & failure detection, summarization, WAN (stubs),
  route manipulation · 1.9.a, 1.9.c, 1.5
- [ ] **Ch. 4 — Troubleshooting EIGRP for IPv4** — adjacencies, routes, trouble tickets · 1.9, 1.9.c–d (stubs), 1.5
- [ ] **Ch. 5 — EIGRPv6 & named EIGRP troubleshooting** · 1.9.a
- [ ] **Ch. 6 — OSPF** — DR/BDR, network types, failure detection, authentication · 1.10 · ↔ `ospf`
- [ ] **Ch. 7 — Advanced OSPF** — LSAs, stubby areas, path selection, summarization,
  discontiguous networks, virtual links · 1.10.c–d · ↔ `advanced-ospf`
- [ ] **Ch. 8 — Troubleshooting OSPFv2** — adjacencies, routes, trouble tickets · 1.10
- [ ] **Ch. 9 — OSPFv3** — configuration, LSA flooding scope · 1.10.a · ↔ `ospfv3`
- [ ] **Ch. 10 — Troubleshooting OSPFv3** — incl. address families · 1.10
- [ ] **Ch. 11 — BGP** — session types & behaviors, MP-BGP for IPv6 · 1.11.a–b · ↔ `bgp`
- [ ] **Ch. 12 — Advanced BGP** — summarization, filtering & manipulation, communities, maximum
  prefix, configuration scalability (peer groups) · 1.11.e · ↔ `bgp-advanced`
- [ ] **Ch. 13 — BGP path selection** — best path, multipath · 1.11.c · ↔ `bgp-advanced`
- [ ] **Ch. 14 — Troubleshooting BGP** — neighbors, routes, path selection, IPv6, trouble tickets · 1.11
- [ ] **Ch. 15 — Route maps & conditional forwarding** — conditional matching, route maps, PBR ·
  1.6 · ↔ `ip-routing-essentials` (PBR)
- [ ] **Ch. 16 — Route redistribution** — protocol-specific configuration · 1.4
- [ ] **Ch. 17 — Troubleshooting redistribution** — IPv4/IPv6, trouble tickets · 1.2 (route maps), 1.3 (loop
  prevention), 1.4

### 2. VPN Technologies (20%)
- [ ] **Ch. 18 — VRF, MPLS & MPLS L3 VPNs** — VRF-Lite, MPLS operations, FEC, L3VPN · 1.7, 2.1, 2.2
  · ↔ `ip-routing-essentials` (VRF), `virtualization`
- [ ] **Ch. 19 — DMVPN tunnels** — GRE, NHRP, DMVPN, spoke-to-spoke, overlay problems, failure
  detection, IPv6 DMVPN · 2.3.a–b, 2.3.d–e · ↔ `overlay-tunnels`
- [ ] **Ch. 20 — Securing DMVPN tunnels** — IPsec fundamentals, tunnel protection · 2.3.c ·
  ↔ `overlay-tunnels`

### 3. Infrastructure Security (20%)
- [ ] **Ch. 21 — Troubleshooting ACLs & prefix lists** — IPv4/IPv6 ACLs, prefix lists · 3.2.a–b
- [ ] **Ch. 22 — Infrastructure security** — AAA, uRPF, CoPP, IPv6 first-hop security ·
  3.1, 3.2.c, 3.3, 3.4 · ↔ `network-device-access-control`

### 4. Infrastructure Services (25%)
- [ ] **Ch. 23 — Device management & management tools troubleshooting** — console/VTY,
  Telnet/SSH/HTTP(S)/SCP/(T)FTP, SNMP, logging & debugs, IP SLA & object tracking, NetFlow,
  **BFD**, DNA Center assurance · 1.8, 4.1–4.3, 4.5–4.7 · ↔ `network-assurance`, `automation-tools`
- DHCP (4.4) is captured in Ch. 1 above.

### Not a skill
- **Ch. 24 — Final Preparation** — exam-day advice, no technical content.
- **Ch. 25 — Exam Updates** — the printed version (June 2023) has no updates. Before booking the
  exam, check the companion site (ciscopress.com/register) for a newer PDF and fold any new
  technical content into the matching chapter skill.

### Notes from the book's Table I-1
- **1.8 BFD lives in Ch. 23** (Device Management and Management Tools), not in the routing
  chapters' failure-detection sections.
- **1.2 route maps and 1.3 loop prevention map to Ch. 17** (troubleshooting redistribution), even
  though Ch. 15 teaches route maps.
- **1.5 summarization** is spread across Ch. 3, 4, 5, 7, 8, 9, 10 and 12.
- **Likely typo in the book:** Table I-1 maps BGP items 1.11.a, 1.11.b and 1.11.d to **Ch. 10**
  (OSPFv3 troubleshooting). The BGP chapters are 11–14; treat those as Ch. 11.

## Repo layout

```
1.0.0/
  package.json            plugin manifest (declares both skill directories)
  skills/                 ENCOR 350-401 — complete
    <topic>/SKILL.md        one structured skill per topic
    ccnp-note/SKILL.md      generates new ENCOR topic skills (template source of truth)
  skills-enarsi/          ENARSI 300-410 — in progress
    <topic>/SKILL.md        one structured skill per topic
    enarsi-note/SKILL.md    generates new ENARSI topic skills (reuses ccnp-note's template)
```

## License

[CC BY-NC-SA 4.0](LICENSE) — share and adapt with attribution, non-commercial
use only, same license for derivatives.

Study notes derived from CCNP ENCOR coursework, restructured in my own words for
reuse as an AI agent skill catalog. Shared for transparency into how I study and
build tooling — not a substitute for the original course material.
