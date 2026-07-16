---
name: ccnp-mst-protocol
description: >
  Use this skill when troubleshooting or configuring mst-protocol on IOS-XE.
  Invoke when the user asks about: MST, 802.1S, MSTI, IST, MST region,
  MST region boundary, PVST simulation, CST, VLAN-to-instance mapping.
---

## Purpose
MST maps many VLANs onto a small number of STP instances (MSTIs) instead of running one STP topology per VLAN like PVST+, drastically cutting BPDU processing and convergence overhead in large-VLAN environments while still allowing per-instance traffic engineering.

## Key Concepts
- Evolution: CST (802.1D) — one shared STP topology for every VLAN, no per-VLAN load balancing. PVST/PVST+ — one full STP instance per VLAN, maximum tuning flexibility but a processing/convergence burden as VLAN count grows (e.g. 14 VLANs = 14 STP instances to maintain). MST (802.1S) — many VLANs mapped onto a handful of MST instances (MSTIs), blending PVST+'s flexibility with CST's lighter processing load.
- An **MST region** is a group of switches sharing identical region name, revision number, and VLAN-to-instance mapping table — switches that don't match exactly are in different regions.
- A region presents itself as a single virtual switch to anything outside it — external (non-MST or different-region) switches calculate their STP topology against the region as one hop, not against each internal switch individually.
- The **IST (Internal Spanning Tree)** is always MST instance 0, runs on every port in the region regardless of which VLANs that port carries, and is mandatory — it can't be removed. Information about every other MSTI is nested inside IST BPDUs, so the region only ever floods one set of BPDUs no matter how many MSTIs exist.
- Instances 1–15 (platform-dependent, but commonly at least 16 total instances supported) are plain MSTIs — no special name, just numbered. All VLANs default to instance 0 (the IST) until explicitly mapped elsewhere.
- `show spanning-tree` for MST shows MST instance numbers, not VLANs — for a VLAN-aware view use `show spanning-tree mst configuration` or `show spanning-tree mst [instance-number]`.
- Displayed bridge/root priority for an MSTI is the configured base priority plus the MST instance number (analogous to PVST+'s priority + VLAN/sys-id-ext).
- **MST region boundary**: any port connecting to a switch in a different MST region, or to a switch running plain 802.1D/802.1W (PVST+/RPVST). MSTIs never interact outside the region — only the IST (as CST) is visible externally.
- **PVST simulation mechanism**: at a region boundary, the MST switch translates the IST into PVST+/RPVST-style BPDUs (one per VLAN) so non-MST neighbors can understand it. It only engages when a PVST+ BPDU is actually received on that port, and it always maps incoming PVST+ traffic from VLAN 1 only back to the IST — never a full per-VLAN remap.
- Boundary design has two valid states: the MST region **is** the root bridge (region floods superior BPDUs for every VLAN, so the PVST+ neighbor does all the blocking — region can still load-balance VLANs across uplinks via per-VLAN PVST+ cost from the neighbor's side), or the MST region **is not** root for any VLAN (boundary ports can only forward or block as a single unit for all VLANs — no per-VLAN load balancing is possible in this mode).
- **PVST simulation check**: if an MST boundary port receives a superior BPDU for a specific VLAN it shouldn't (i.e., the region is supposed to be non-root but something claims to be better for one VLAN), the switch blocks that port and places it in a root-inconsistent state — a loop-prevention safety net, similar in spirit to root guard.

## Procedure
Configuring an MST region:
1. Set the spanning tree mode to MST: `spanning-tree mode mst`.
2. (Optional) Set MST instance priority, either with `spanning-tree mst <instance> priority <priority>` (0–61440, increments of 4096) or `spanning-tree mst <instance> root {primary | secondary} [diameter <n>]` (primary = 24576, secondary = 28672).
3. Enter MST configuration submode (`spanning-tree mst configuration`) and map VLANs to instances with `instance <instance-number> vlan <vlan-id>` — any VLAN not explicitly mapped stays on instance 0 (the IST).
4. Set the MST revision number with `revision <version>` — this must match exactly across every switch intended to be in the same region.
5. (Optional) Set the region name with `name <mst-region-name>` — this must also match exactly across the region; default is an empty string.

## Config Patterns
```ios-xe
! Full MST region setup: mode, region identity, instance priorities, VLAN mapping
spanning-tree mode mst
spanning-tree mst 0 root primary
spanning-tree mst 1 root primary
spanning-tree mst 2 root primary
spanning-tree mst configuration
 name ENTERPRISE_CORE
 revision 2
 instance 1 vlan 10,20
 instance 2 vlan 99

! Tune port cost / port priority on a per-instance basis
interface GigabitEthernet1/0/1
 spanning-tree mst 0 cost 1
interface GigabitEthernet1/0/5
 spanning-tree mst 1 port-priority 64
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Region name, revision, and VLAN-to-instance map identical on every switch meant to share a region | Any mismatch silently splits the region and moves the boundary | Deliberate multi-region designs — then the boundary IS the design | ENCOR 350-401 OCG |
| VLAN ranges pre-mapped to MSTIs before the VLANs exist | Remapping later changes the region config and forces re-convergence | Small, stable networks where remaps are rare and scheduled | ENCOR 350-401 OCG |
| User VLANs kept off instance 0 (IST), mapped to numbered MSTIs | The IST carries region-wide duties; separating user traffic keeps tuning clean | Tiny deployments effectively running a single instance anyway | ENCOR 350-401 OCG |
| Boundary strategy chosen deliberately: region is root for all VLANs, or for none | Accidental mixed boundary roles trigger PVST simulation blocks (root-inconsistent) | — | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show spanning-tree mst configuration` | Region name, revision, and the full instance-to-VLAN mapping table — confirm it matches exactly on every switch meant to share the region |
| `show spanning-tree mst [instance-number]` | Per-instance topology including the VLANs mapped to it, root ID, and per-interface role/state — easiest view for troubleshooting since it shows VLANs directly |
| `show spanning-tree mst interface <id>` | Per-interface MST detail across all instances on that port: edge/boundary status, link type, BPDU filter/guard state, and role/cost/priority per instance |
| `show spanning-tree` | MST instance numbers and per-interface role/state — useful but does not show VLAN mappings directly, prefer `show spanning-tree mst` for VLAN-aware troubleshooting |

## Intent Questions
- Which switches are supposed to be in which MST region — and is every name/revision/mapping identical on purpose?
- Which VLANs map to which instance by design, and why those groupings?
- Where are the region boundaries supposed to be, and is the region supposed to be root beyond them?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1/2: confirm physical link state (`show interfaces status`) before assuming an MST-specific issue — MST inherits all the same Layer 1/2 dependencies as RSTP.
2. Region mismatch: `show spanning-tree mst configuration` on every switch expected to share a region — name, revision, and VLAN-to-instance table must match *exactly*, or switches end up in separate regions and a boundary forms where none was intended.
3. Unintended blocking from IST/VLAN misalignment: if traffic between two access ports on the same VLAN takes an unexpected path, check whether that VLAN is sitting on the IST (instance 0) instead of a dedicated MSTI — the IST topology spans every port in the region regardless of VLAN, so it can introduce a blocking port that doesn't match the access-layer VLAN's actual traffic pattern.
4. Trunk pruning causing isolation: if hosts lose connectivity to a server VLAN after a trunk pruning change, verify all VLANs belonging to the *same MSTI* were pruned together on the same links — pruning one VLAN from an MSTI's link while leaving another VLAN in the same MSTI on that link breaks the assumption that the instance's topology still matches reality.
5. Boundary/PVST simulation issues: `show spanning-tree mst interface <id>` — check the Boundary field; if a region-boundary port unexpectedly drops to root-inconsistent, that's the PVST simulation check rejecting a superior per-VLAN BPDU it wasn't expecting (review whether the MST region is supposed to be root for that VLAN or not).
6. Config error: VLAN left unintentionally on the IST instead of being explicitly mapped to its intended MSTI — this is one of the two textbook common MST misconfigurations.
7. Config error: trunk pruning applied inconsistently within a single MSTI — the second textbook common MST misconfiguration.
8. Software/platform bug (rare) — only after region identity, instance mapping, and boundary configuration are all confirmed consistent across every switch involved.

## Common Pitfalls
- Assuming a VLAN's traffic follows its access-port assignment when it's actually still riding the IST (instance 0) — every port in the region participates in the IST regardless of VLAN, so an access VLAN can get blocked by IST topology decisions that have nothing to do with that VLAN specifically.
- Pruning VLANs on a trunk link for load-balancing purposes without checking which MSTI they belong to — pruning must be applied consistently across all VLANs in the same MSTI on a given link, or the live topology stops matching what MST calculated.
- Forgetting that region membership requires an *exact* match of name, revision, and VLAN mapping table — being "close" is the same as being in a completely different region as far as MST is concerned.
- Expecting per-VLAN load balancing at an MST region boundary when the region is not the root bridge for any VLAN — in that mode, boundary ports can only block or forward as a single unit for all VLANs; per-VLAN tuning is only possible when the MST region itself is the root.
- Confusing `show spanning-tree` (shows MST instance numbers) with `show spanning-tree mst` (shows VLANs mapped to each instance) — use the latter when troubleshooting by VLAN.
