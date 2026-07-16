---
name: ccnp-rapid-spanning-tree-protocol
description: >
  Use this skill when troubleshooting or configuring rapid-spanning-tree-protocol on IOS-XE.
  Invoke when the user asks about: RSTP, 802.1W, discarding state, alternate
  port, backup port, edge port, point-to-point port, proposal agreement,
  RSTP synchronization, PVST+.
---

## Purpose
RSTP (802.1W) replaces 802.1D's slow timer-based convergence with an active proposal/agreement handshake between switches, cutting Layer 2 failover from ~30-50s down to sub-second in most topologies.

## Key Concepts
- RSTP collapses the five 802.1D port states into three: **Discarding** (merges disabled, blocking, and listening — port enabled but not forwarding, no MAC learning), **Learning** (MAC learning active, BPDUs only), **Forwarding** (full forwarding + learning).
- RSTP adds two new port roles beyond RP/DP: **Alternate port** — a backup path to the root via a *different* switch (replaces 802.1D's blocking role for redundant root paths); **Backup port** — redundancy onto the *same* shared segment, typically only seen with hubs/shared media.
- RSTP port types: **Edge port** (host-facing, single device, cannot loop — equivalent to portfast-enabled), **Non-Edge port** (has received a BPDU, so it's switch-facing), **Point-to-point port** (full duplex link to another RSTP switch — full duplex is the fast heuristic for "only two devices on this segment").
- Half-duplex (multi-access, e.g. hub) ports cannot use RSTP's fast mechanisms and must fall back to legacy 802.1D forwarding state behavior.
- If a switch's RSTP handshake doesn't get a response from the far end, it assumes the neighbor is non-RSTP-capable and that port reverts to plain 802.1D timing — host devices on such a port still see ~30s of delay before forwarding.
- RSTP ages out stale port info after missing 3 consecutive hellos — 6 seconds with default 2-second hello timer, versus the 20-second Max Age timer in 802.1D.
- If a downstream switch fails to acknowledge (agree to) a proposal, RSTP falls back to 802.1D-style behavior on that link to avoid a forwarding loop.

## Procedure
Synchronization handshake when two switches first connect:
1. Both switches verify the link is point-to-point by checking full-duplex status.
2. Both switches propose themselves as the Designated Port (DP) for the segment, advertised in configuration BPDUs.
3. Each switch compares bridge IDs using the same tiebreak as 802.1D (lowest priority, then lowest MAC) to determine which is superior and which is inferior.
4. The inferior switch sets its local port to Root Port (RP) and immediately moves *all* its non-edge ports to discarding state, stopping local switching on those ports.
5. The inferior switch sends an agreement BPDU back to the superior switch, signaling synchronization is underway.
6. The inferior switch's RP and the superior switch's DP both move straight to forwarding.
7. The inferior switch repeats this same proposal/agreement process toward any of its own downstream switches.

## Config Patterns
```ios-xe
! Enable Rapid PVST+ (RSTP per VLAN) globally — standard Catalyst RSTP mode
spanning-tree mode rapid-pvst

! Mark a host-facing port as an edge port (portfast) — only on ports with a single end device
interface GigabitEthernet1/0/14
 spanning-tree portfast

! Explicitly force point-to-point link type if auto-detection via duplex is unreliable
interface GigabitEthernet1/0/2
 spanning-tree link-type point-to-point

! Force a half-duplex/shared-media port to be treated as shared (no RSTP fast transitions)
interface GigabitEthernet1/0/5
 spanning-tree link-type shared
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| `spanning-tree mode rapid-pvst` (or MST) as the domain-wide mode | Sub-second convergence; alternate/edge roles behave as designed | Legacy neighbors that force 802.1D fallback anyway | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| Edge (portfast) only on true single-host ports, paired with BPDU guard | Until a BPDU arrives, an edge port forwards instantly — a switch plugged in there is a loop window | Documented non-bridging devices on the port | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| Switch-to-switch links run full duplex so RSTP treats them as point-to-point | Half duplex forces the slow 802.1D fallback on that segment | Legacy hubs/shared media (rare) | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show spanning-tree vlan <id>` | Per-interface Role column — confirm Root/Desg/Altn/Back roles match the expected topology, and `Sts` shows FWD not stuck in discarding/learning |
| `show spanning-tree summary` | Confirms RSTP/Rapid-PVST+ is the active protocol mode and edge-port count |
| `show spanning-tree interface <id> detail` | Link type (point-to-point vs shared) and edge/non-edge designation for one port |
| `show spanning-tree root` | Root ID and cost per VLAN — same as 802.1D, root path cost should still be 0 on the root bridge |
| `show spanning-tree vlan <id> detail` | Topology change count — still relevant under RSTP for detecting flapping links |

## Intent Questions
- Which ports are supposed to be edge (host-facing) vs. switch-facing?
- Which links should be point-to-point (full duplex) so proposal/agreement fast convergence applies?
- Where should the alternate port sit when the topology is healthy?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1: confirm full duplex on the link (`show interfaces status` / `show interfaces <id>`) — RSTP's fast point-to-point path requires full duplex; half duplex forces the slow 802.1D fallback.
2. Layer 2 role/state: `show spanning-tree vlan <id>` — verify the port has an Alternate or Backup role where redundancy is expected, not stuck as a plain blocking/discarding port with no role context.
3. Edge port misconfiguration: a switch-to-switch link mistakenly configured with `spanning-tree portfast` skips the proposal/agreement safety check and can create a transient loop — confirm edge ports are genuinely host-only.
4. Link type mismatch: `show spanning-tree interface <id> detail` — if RSTP fails to negotiate point-to-point on what should be a direct switch link, force it with `spanning-tree link-type point-to-point`.
5. Non-RSTP neighbor: if a connected device never completes the handshake, that port silently reverts to 802.1D timing (~30s) — verify the neighbor's STP mode/capability if convergence on that link feels slow.
6. Software/platform bug (rare) — only after duplex, link type, and role assignments are confirmed correct on both ends of the link.

## Common Pitfalls
- Enabling `spanning-tree portfast` on a port that actually connects to another switch — bypasses the proposal/agreement loop-prevention check and risks a forwarding loop, exactly the scenario edge ports are supposed to exclude.
- Assuming RSTP always converges in sub-second time — any link that can't establish full duplex / point-to-point still falls back to legacy 802.1D timers (~30s), so duplex mismatches silently erase RSTP's speed benefit.
- Confusing 802.1D's single "blocking" role with RSTP's two distinct roles (Alternate vs Backup) — Alternate means a different switch offers a redundant root path; Backup means redundancy onto the same shared segment, which is rare outside hub-connected scenarios.
- Forgetting that PVST+ (Cisco's per-VLAN RSTP) is what `spanning-tree mode rapid-pvst` enables — RSTP itself (802.1W) is a single-instance concept, but Catalyst switches run one RSTP instance per VLAN under PVST+.
