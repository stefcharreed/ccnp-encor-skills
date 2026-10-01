# ENCOR v1.2 — OCG Chapter Weights

*Logged 2026-10-01. Weights every chapter of the **ENCOR OCG, 2nd edition** (written for v1.1)
against the **ENCOR 350-401 v1.2** blueprint (live 2026-03-19), plus the v1.2 material the
book does not cover. Companion to [`encor-v1-2-gap`](1.0.0/skills/encor-v1-2-gap/SKILL.md).*

## How the weights were built

1. **Start from Cisco's official domain weights** (v1.2 blueprint): Architecture 15% ·
   Virtualization 10% · Infrastructure 30% · Network Assurance 10% · Security 20% ·
   Automation & AI 15%.
2. **Map every v1.2 objective to the chapter that teaches it**, and split each domain's
   percentage across those chapters in proportion to how many objectives they carry and how
   deep the verb goes (*configure/troubleshoot* > *interpret* > *describe/explain*). Chapter
   weights therefore **sum to 100%**.
3. **Assign a priority tier** from the weight **plus** two adjustments: (a) **blueprint verb** —
   anything marked *configure* or *troubleshoot* can appear as a lab, and (b) the **v1.2
   test-taker sample** (Reddit, 2026-10-01 — see the gap skill), which reports automation as the
   heaviest area, labs focused on routing/switching configuration, and virtualization as a
   quiet failure point.

> ⚠️ **Honest limits.** Cisco publishes weights **per domain only**; everything below that is
> an allocation by objective mapping — a reasoned estimate, not Cisco data. The test-taker
> sample is small (~9 first-hand accounts), self-reported and NDA-limited. Chapter numbers
> 14–29 are confirmed by this catalog's own citations; **chapters 1–13 are mapped from the
> 2nd-edition table of contents and were not re-checked against the book in this pass.**

## The table

| Ch | Chapter (skill) | v1.2 objectives | Verb | Weight | Tier |
|---|---|---|---|---|---|
| 28 | Foundational Network Programmability ([skill](1.0.0/skills/network-programmability-foundations/SKILL.md)) | 6.1 Python · 6.2 JSON · 6.3 YANG · 6.4 Catalyst Center / SD-WAN Manager APIs · 6.5 REST codes · 4.6 NETCONF/RESTCONF · 5.3 REST API security | interpret, construct, configure | **13.0%** | 🔴 1 |
| 26 | Network Device Access Control & Infrastructure Security ([skill](1.0.0/skills/network-device-access-control/SKILL.md)) | 5.1 lines, local users, AAA · 5.2 ACLs, CoPP | **configure** | **12.5%** | 🔴 1 |
| 24 | Network Assurance ([skill](1.0.0/skills/network-assurance/SKILL.md)) | 4.1 debugs/ping/traceroute/SNMP/syslog · 4.2 Flexible NetFlow · 4.3 SPAN/RSPAN/ERSPAN · 4.4 IP SLA · 4.5 Catalyst Center (traditional) | **configure**, diagnose | **7.0%** | 🔴 1 |
| 23 | Fabric Technologies ([skill](1.0.0/skills/fabric-technologies/SKILL.md)) | 1.2 Catalyst SD-WAN · 1.3 SD-Access | explain | **7.0%** | 🔴 1 |
| 16 | Overlay Tunnels ([skill](1.0.0/skills/overlay-tunnels/SKILL.md)) | 2.2.b GRE & IPsec · 2.3 LISP, VXLAN | **configure**, describe | **6.0%** | 🔴 1 |
| 29 | Introduction to Automation Tools ([skill](1.0.0/skills/automation-tools/SKILL.md)) | 6.6 EEM applets · 6.7 agent vs agentless | **construct**, compare | **5.0%** | 🔴 1 |
| 15 | IP Services ([skill](1.0.0/skills/ip-services/SKILL.md)) | 3.3.a NTP/PTP · 3.3.b NAT/PAT · 3.3.c HSRP/VRRP | **configure**, interpret | **4.5%** | 🔴 1 |
| 25 | Secure Network Access Control ([skill](1.0.0/skills/secure-network-access-control/SKILL.md)) | 5.4 threat defense, endpoint security, NGFW, TrustSec & MACsec *(802.1X/MAB/WebAuth NAC removed in v1.2)* | describe | **6.0%** | 🟠 2 |
| 14 | QoS ([skill](1.0.0/skills/qos/SKILL.md)) | 1.4 interpret QoS configurations | interpret | **4.0%** | 🟠 2 |
| 22 | Enterprise Network Architecture ([skill](1.0.0/skills/enterprise-network-architecture/SKILL.md)) | 1.1 2-/3-tier, fabric, cloud; HA (redundancy, FHRP, SSO) | explain | **4.0%** | 🟠 2 |
| 5 | VLAN Trunks & EtherChannel ([skill](1.0.0/skills/vlan-trunks-and-etherchannel/SKILL.md)) | 3.1.a 802.1Q trunking · 3.1.b EtherChannel | **troubleshoot** | **3.5%** | 🔴 1 |
| 8 | OSPF ([skill](1.0.0/skills/ospf/SKILL.md)) | 3.2.a compare · 3.2.b configure (adjacency, network types, passive-interface) | **configure** | **3.5%** | 🔴 1 |
| 6 | IP Routing Essentials ([skill](1.0.0/skills/ip-routing-essentials/SKILL.md)) | 2.2.a VRF · 3.2.d PBR · RIB/AD foundations | configure (VRF), describe | **3.0%** | 🟠 2 |
| 27 | Virtualization ([skill](1.0.0/skills/virtualization/SKILL.md)) | 2.1 hypervisors, VMs, virtual switching | describe | **2.5%** | 🟠 2 |
| 2 | Spanning Tree Protocol ([skill](1.0.0/skills/spanning-tree-protocol/SKILL.md), [RSTP](1.0.0/skills/rapid-spanning-tree-protocol/SKILL.md)) | 3.1.c RSTP | **configure** | **2.5%** | 🟠 2 |
| 9 | Advanced OSPF ([skill](1.0.0/skills/advanced-ospf/SKILL.md)) | 3.2.b multiple normal areas, summarization, filtering | **configure** | **2.5%** | 🔴 1 |
| 11 | BGP ([skill](1.0.0/skills/bgp/SKILL.md)) | 3.2.c eBGP between directly connected neighbors, best path | **configure** | **2.5%** | 🔴 1 |
| 3 | Advanced STP Tuning ([skill](1.0.0/skills/advanced-stp-tuning/SKILL.md)) | 3.1.c root guard, BPDU guard | **configure** | **1.5%** | 🟠 2 |
| 4 | Multiple Spanning Tree ([skill](1.0.0/skills/mst-protocol/SKILL.md)) | 3.1.c MST | **configure** | **1.5%** | 🟠 2 |
| 7 | EIGRP ([skill](1.0.0/skills/eigrp/SKILL.md)) | 3.2.a compare EIGRP vs OSPF only | compare | **1.5%** | 🟡 3 |
| 13 | Multicast ([skill](1.0.0/skills/multicast/SKILL.md)) | 3.3.d RPF check, PIM SM, IGMP v2/v3 *(book portion)* | describe | **1.5%** | 🟡 3 |
| 10 | OSPFv3 ([skill](1.0.0/skills/ospfv3/SKILL.md)) | 3.2.b "OSPFv2/**v3**" | configure | **1.0%** | 🟡 3 |
| 12 | Advanced BGP ([skill](1.0.0/skills/bgp-advanced/SKILL.md)) | best-path detail only — multihoming, communities, filtering are ENARSI depth | (part of 3.2.c) | **1.0%** | 🟡 3 |
| 1 | Packet Forwarding ([MAC](1.0.0/skills/mac-addresses/SKILL.md), [L3](1.0.0/skills/layer-3-forwarding/SKILL.md), [VLANs](1.0.0/skills/VLANS/SKILL.md)) | *v1.1 1.6 CEF/CAM/TCAM/FIB/RIB **removed***; foundation only | — | **0.5%** | 🟡 3 |
| 17–21 | Wireless ×5 ([signals](1.0.0/skills/wireless-signals-and-modulation/SKILL.md), [infrastructure](1.0.0/skills/wireless-infrastructure/SKILL.md), [roaming](1.0.0/skills/wireless-roaming-and-location-services/SKILL.md), [client auth](1.0.0/skills/wireless-client-authentication/SKILL.md), [troubleshooting](1.0.0/skills/wireless-troubleshooting/SKILL.md)) | **All removed** (moved to the dedicated wireless track) | — | **0%** | ⚪ skip |
| 30 | Final Preparation | no technical content | — | — | — |
| **gap** | **Multicast SSM, bidir, MSDP** ([gap skill](1.0.0/skills/encor-v1-2-gap/SKILL.md)) | 3.3.d — *not in the book* | describe | **1.0%** | 🟡 3 |
| **gap** | **Catalyst Center AI-powered workflows** ([gap skill](1.0.0/skills/encor-v1-2-gap/SKILL.md)) | 4.5 — *not in the book* | describe | **1.5%** | 🟠 2 |
| | | | | **100%** | |

### Domain roll-up (should match Cisco's numbers)
| Domain | Chapters (weight) | Total |
|---|---|---|
| 1.0 Architecture | 23 (7.0) · 22 (4.0) · 14 (4.0) | **15%** ✓ |
| 2.0 Virtualization | 16 (6.0) · 27 (2.5) · 6-VRF (1.5) | **10%** ✓ |
| 3.0 Infrastructure | 15 (4.5) · 5 (3.5) · 8 (3.5) · 2 (2.5) · 9 (2.5) · 11 (2.5) · 3 (1.5) · 4 (1.5) · 6-PBR (1.5) · 7 (1.5) · 13 (1.5) · 10 (1.0) · 12 (1.0) · 1 (0.5) · gap multicast (1.0) | **30%** ✓ |
| 4.0 Network Assurance | 24 (7.0) · 28-NETCONF/RESTCONF (1.5) · gap AI (1.5) | **10%** ✓ |
| 5.0 Security | 26 (12.5) · 25 (6.0) · 28-REST API security (1.5) | **20%** ✓ |
| 6.0 Automation & AI | 28 (10.0) · 29 (5.0) | **15%** ✓ |

## What the weighting says

- **The back of the book is the front of the exam.** Chapters **22–29 carry ~57%** of the
  weight; chapters 1–13, which most people study first and longest, carry **~26%**.
- **Chapter 28 (programmability) is the single heaviest chapter (13%)** — it feeds three
  domains — and test-takers independently called automation the hardest area. Chapter 26
  (device access / AAA / ACLs / CoPP) is a close second and is almost entirely *configure*.
- **Routing chapters are lab chapters, not volume chapters.** OSPF, BGP, trunks/EtherChannel,
  NAT/FHRP, SPAN, IP SLA and CoPP are low-to-mid weight on paper but are where the labs come
  from — so they stay Tier 1 despite their percentages. Drill them from a blank config.
- **Breadth over depth for 23 (SD-Access/SD-WAN), 25 and the AI workflows:** *explain/describe*
  verbs — know components and roles, not configuration.
- **EIGRP (7), advanced BGP (12) and most of multicast (13) are compare/describe only** —
  their depth belongs to ENARSI.
- **Skip chapters 17–21 entirely** (≈ a sixth of the book).

## Sources
- [Cisco ENCOR v1.2 exam topics (PDF)](https://learningcontent.cisco.com/documents/marketing/exam-topics/350-401-ENCORE-v1.2.pdf) · [v1.1 (PDF)](https://learningcontent.cisco.com/documents/marketing/exam-topics/350-401-ENCORE-v1.1.pdf)
- Test-taker evidence: the "What v1.2 test-takers report" section of
  [`encor-v1-2-gap`](1.0.0/skills/encor-v1-2-gap/SKILL.md) (r/ccnp, r/ccna, posts after 2026-03-19)
