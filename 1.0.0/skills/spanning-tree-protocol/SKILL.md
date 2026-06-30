---
name: ccnp-spanning-tree-protocol
description: >
  Use this skill when troubleshooting or configuring spanning-tree-protocol on IOS-XE.
  Invoke when the user asks about: STP, 802.1D, RSTP, MST, PVST+, BPDU,
  root bridge, root port, designated port, blocking port, TCN, topology
  change, root path cost.
---

## Purpose
STP builds a loop-free Layer 2 topology by electing a root bridge and selectively blocking redundant ports, preventing broadcast storms and MAC table instability from physical loops.

## Key Concepts
- STP iterations on Catalyst switches today: PVST+, RSTP (802.1W), and MST (802.1S) — all backward compatible with the original 802.1D.
- 802.1D port states in order: Disabled → Blocking (BPDU rx only, no MAC learning) → Listening (BPDU tx/rx, no learning) → Learning (MAC learning, no forwarding) → Forwarding (full forwarding + learning); Broken/discarding is a fault state. Full blocking-to-forwarding convergence takes ~30s with default timers (15s listening + 15s learning, on top of detection).
- Port roles: Root Port (RP) — one per switch per VLAN, the best path toward the root bridge; Designated Port (DP) — forwards onto a segment, one active DP per link; Blocking port — loses the DP election on that segment, stays non-forwarding.
- Root bridge election: every switch starts assuming it's root, then defers to any neighbor BPDU with a lower bridge ID (priority + sys-id-ext, tiebreak by lowest system MAC). Lower MAC wins ties, which historically favored older switches — override deliberately by lowering priority on the intended root.
- Bridge priority shown in `show spanning-tree` already includes the VLAN (sys-id-ext) added to the base priority — e.g. base 32768 + VLAN 10 = displayed priority 32778.
- Root path cost accumulates the *receiving* interface's STP cost at each hop; it's always 0 on the root bridge itself and the BPDU never includes the cost of the port it was sent out of.
- Root port selection tiebreak order: lowest root path cost → lowest neighbor system priority → lowest neighbor system MAC → lowest neighbor port priority → lowest neighbor port number.
- Designated port blocking tiebreak between two non-root switches sharing a segment: lower path cost to root wins → lower local system priority wins → lower local system MAC wins.
- Short-mode STP cost (default on most platforms) vs long-mode (`spanning-tree pathcost method long`) — must be set identically on every switch in the L2 domain or the topology becomes inconsistent.
- TCN (topology change notification) flows from the detecting switch out its RP toward the root; the root then floods a configuration BPDU with the Topology Change flag, which makes every switch shrink its MAC aging timer to the forward delay (default 15s) to flush stale entries — this temporarily increases unknown unicast flooding.
- Convergence time depends on failure type: direct link failure with an already-blocking alternate is fast (no recalculation needed); direct failure requiring a new RP costs ~30s (listening+learning); indirect failures (link up but BPDUs not getting through) cost the full Max Age timer (20s) plus listening+learning (~50s total).
- *TYPE_Inc* in `show spanning-tree` Type field signals a port type mismatch with the connected switch — usually trunk/access mode mismatch.

## Config Patterns
```ios-xe
! Tune STP timers per VLAN (only change from defaults with a full topology audit)
spanning-tree vlan 10 hello-time 2
spanning-tree vlan 10 max-age 20
spanning-tree vlan 10 forward-time 15

! Bias root bridge election by lowering priority (must be a multiple of 4096)
spanning-tree vlan 10 priority 4096

! Switch to long-mode path cost (must match across the entire L2 domain)
spanning-tree pathcost method long
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show spanning-tree root` | Root ID, root path cost, and root port per VLAN — cost 0 with no Root Port field means this switch is the root |
| `show spanning-tree vlan <id>` | Full root bridge + local bridge identity, plus per-interface Role/Sts/Cost/Type for one VLAN |
| `show spanning-tree vlan <id> detail` | Topology change count and time since last change — spikes indicate flapping ports or instability |
| `show spanning-tree interface <id> [detail]` | STP state for one interface across all VLANs it carries — fastest way to audit a trunk without scrolling full `show spanning-tree` output |
| `show spanning-tree` | Full per-VLAN, per-interface table — use only when you need the complete picture, otherwise prefer the filtered commands above |

## Troubleshooting Checklist
1. Layer 1: confirm the physical link is actually up (`show interfaces status`) before assuming an STP problem — STP only reacts to what Layer 1/2 reports.
2. Layer 2 port role/state: `show spanning-tree vlan <id>` — confirm RP/DP placement matches the expected topology and no port is unexpectedly stuck in Blocking/Listening/Learning.
3. Port type mismatch: look for `*TYPE_Inc` in the Type column — check trunk vs access mode agreement on both ends of the link.
4. Missing VLAN on a trunk: `show spanning-tree interface <id> detail` — if a VLAN is absent, check `switchport trunk allowed vlan` on that interface.
5. Topology instability: `show spanning-tree vlan <id> detail` — a high or rapidly increasing topology change count points to a flapping port or an unstable downstream switch; trace toward the reported originating interface.
6. Root bridge placement: confirm the intended switch is root (`show spanning-tree root`) — an unintended root (e.g., an old switch with a low MAC) usually means priority was never set deliberately.
7. Indirect failure suspicion (link up, but BPDUs not arriving): expect slower convergence (~50s) bound by the Max Age timer — check for a filtering device, bad SFP, or duplex mismatch in the path.

## Common Pitfalls
- Forgetting the displayed bridge/root priority already includes the VLAN ID (sys-id-ext) added to the configured base priority — don't mistake 32778 for a misconfigured priority when it's just base 32768 + VLAN 10.
- Assuming all convergence events take the same ~30 seconds — direct failures with a pre-existing blocking alternate are near-instant, while indirect failures take the full Max Age timer plus listening/learning (~50s).
- Treating an old switch becoming root bridge as STP malfunctioning — it's STP working as designed (lowest MAC wins ties); fix by deliberately setting root priority, not by assuming a bug.
- Confusing `show spanning-tree` (every VLAN, every interface) with `show spanning-tree interface <id>` (one interface, every VLAN it carries) — using the wrong one on a busy trunk buries the answer in noise.
- Mixing short-mode and long-mode path cost across switches in the same L2 domain — produces an inconsistent topology; this must be uniform everywhere and verified by audit before changing.
