---
name: ccnp-advanced-stp-tuning
description: >
  Use this skill when troubleshooting or configuring advanced-stp-tuning on IOS-XE.
  Invoke when the user asks about: root guard, BPDU guard, BPDU filter,
  loop guard, UDLD, portfast, errdisable recovery, root bridge placement,
  port cost tuning, port priority tuning, MAC flapping.
---

## Purpose
Advanced STP tuning deliberately places the root bridge, shapes the forwarding topology with cost/priority, and layers protection mechanisms (root guard, BPDU guard, loop guard, UDLD) so STP behaves predictably and resists misconfiguration or rogue devices instead of just preventing loops by accident.

## Key Concepts
- Deliberate root placement beats letting MAC-address tiebreaks decide it: set the intended root's priority to the lowest value (commonly 0) and the secondary root next-lowest (commonly 4096), and ideally raise priority on all other switches — pairs with root guard so this placement can't be hijacked.
- `spanning-tree vlan <id> root {primary | secondary} [diameter <n>]` is a convenience script: primary sets priority to 24576, secondary to 28672; if a competing switch already has a lower priority, the script lowers the local priority further to still win. `diameter` tunes convergence timers based on max Layer 2 hop count to the root — only needs to be set on the root, since timers propagate via root BPDUs.
- Root path cost is calculated by the *receiving* switch adding its local port's STP cost to the cost already in the BPDU — the advertising switch never includes its own egress port cost.
- You can manually reshape the topology two ways: lower `spanning-tree cost` on a port to pull it toward Designated/Root role (or raise it to push toward blocking/alternate); or lower `spanning-tree port-priority` (default 128) on the **upstream** switch's port to make it preferred when a downstream switch has two equal-cost links to choose between — port priority/number tiebreaks are controlled by the switch closer to the root, not the downstream switch.
- A flapping MAC address (`%SW_MATM-4-MACFLAP_NOTIF` syslog) between two ports is the classic symptom of an active Layer 2 forwarding loop — common causes: STP disabled somewhere, a misconfigured load balancer, a hypervisor's virtual switch bridging two NICs (vSwitches typically don't run STP), or a dumb hub/switch introduced by a user.
- **Root guard** (`spanning-tree guard root`, per-port): placed on designated ports facing switches that must never become root. If a superior BPDU arrives, the port goes root-inconsistent (acts like listening, no forwarding) until the superior BPDUs stop — prevents a rogue/misconfigured downstream switch from taking over as root.
- **STP Portfast**: skips listening/learning and suppresses TCN generation on access ports — correct only for host-only ports (DHCP/PXE benefit from instant forwarding). Receiving a BPDU on a portfast port immediately removes portfast behavior and the port reverts to normal STP progression. Enabling portfast also makes the port an RSTP Edge port.
- **BPDU guard**: safety net for portfast ports — any BPDU received puts the port into err-disabled immediately (no behavior reversion like plain portfast). Default-recommended on every host-facing portfast port; ports don't self-recover unless `errdisable recovery cause bpduguard` is configured.
- **BPDU filter**: stops BPDU transmission/processing on a port — high-risk, rarely needed. Behavior differs by enable mode (interface vs global) and by whether the local switch is "preferred" vs "non-preferred" relative to a neighbor that starts sending BPDUs; misuse can defeat loop protection entirely. Treat as a feature to avoid unless a specific use case demands it.
- **Loop guard**: protects against unidirectional-link-style loops by preventing a root/alternate port from becoming a designated port just because it stopped receiving BPDUs — puts the port in loop-inconsistent state instead, until BPDUs resume. Must never be combined with portfast (the role logic conflicts).
- **UDLD**: detects unidirectional fiber faults (one strand broken, link still shows up/up) by having both ends echo back system/port ID info. Normal mode leaves an unacknowledged link up as "undetermined"; aggressive mode retries 8 times at 1s intervals and then error-disables the port. Must be enabled on both ends of a link to function.

## Config Patterns
```ios-xe
! Deliberate root bridge placement
spanning-tree vlan 1 root primary
spanning-tree vlan 1 priority 0

! Secondary (backup) root bridge
spanning-tree vlan 1 root secondary
spanning-tree vlan 1 priority 4096

! Tune port cost / port priority to reshape the topology
interface GigabitEthernet1/0/1
 spanning-tree cost 1
 spanning-tree port-priority 64

! Root guard — on designated ports facing switches that must never become root
interface GigabitEthernet1/0/4
 spanning-tree guard root

! Portfast — host-facing access ports only
interface GigabitEthernet1/0/13
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
spanning-tree portfast default
spanning-tree portfast bpdufilter default

! BPDU guard — globally on all portfast ports, with per-port override
spanning-tree portfast bpduguard default
interface GigabitEthernet1/0/8
 spanning-tree bpduguard disable

! Errdisable auto-recovery for BPDU guard violations
errdisable recovery cause bpduguard
errdisable recovery interval 120

! Loop guard — never combine with portfast on the same port
spanning-tree loopguard default
interface GigabitEthernet1/0/1
 spanning-tree guard loop

! UDLD — global enable plus per-port aggressive mode and recovery
udld enable
udld recovery interval 60
interface TenGigabitEthernet1/1/3
 udld port aggressive
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show spanning-tree vlan <id>` | Root ID priority reflects intended placement (e.g. 24577 after `root primary`); interface Cost/Prio.Nbr/Role match the intended reshaped topology |
| `show spanning-tree interface <id> detail` | Confirms portfast, BPDU guard, BPDU filter, and link type status per port — also shows BPDU sent/received counters |
| `show spanning-tree inconsistentports` | Lists every VLAN/interface currently in root-inconsistent or loop-inconsistent state — should normally be empty |
| `show interfaces status` | `err-disabled` status confirms a BPDU guard (or other errdisable cause) trip; cross-reference with syslog `%PM-4-ERR_DISABLE` |
| `show errdisable recovery` | Confirms which errdisable causes have auto-recovery enabled and the recovery interval |
| `show udld neighbors` | Confirms `Bidirectional` state per UDLD-enabled neighbor link |
| `show udld <interface-id>` | Detailed UDLD state, device IDs, and originating/return port IDs for one link |

## Troubleshooting Checklist
1. Layer 1: for suspected unidirectional link issues, confirm physical fiber/cable integrity and `show udld neighbors` state — `Bidirectional` expected, anything else signals one strand is broken even though line protocol shows up.
2. Layer 2 inconsistent state: `show spanning-tree inconsistentports` — a populated list with `Root Inconsistent` means root guard tripped (unexpected superior BPDU from a downstream device); `Loop Inconsistent` means loop guard tripped (BPDUs stopped arriving on a root/alternate port).
3. Err-disabled ports: `show interfaces status` for `err-disabled`, then check syslog for `%SPANTREE-2-BLOCK_BPDUGUARD` (BPDU guard trip — usually a switch/hub plugged into a portfast access port) or other errdisable causes.
4. MAC flapping syslog (`%SW_MATM-4-MACFLAP_NOTIF`): treat as an active forwarding loop — check every switch hosting that VLAN for STP being enabled and functioning, and look for hypervisor vSwitches or load balancers bridging traffic outside STP's visibility.
5. Config error: portfast enabled on a port that's actually switch-facing (not host-only) — defeats the proposal/agreement safety check RSTP relies on; or loop guard combined with portfast on the same port, which directly conflicts with root/alternate port logic.
6. Config error: root/secondary priority set inconsistently, or root guard omitted on designated ports facing untrusted/downstream switches — leaves root bridge placement vulnerable to a rogue device with a lower priority.
7. Software/platform bug (rare) — only after root placement, port cost/priority, guard features, and UDLD state are all confirmed correctly configured and consistent with the intended topology.

## Common Pitfalls
- Enabling BPDU filter without fully understanding its asymmetric preferred/non-preferred behavior — it can silently stop the local port from generating or reacting to BPDUs in ways that disable loop protection precisely where it's needed most; avoid unless a specific design genuinely requires it.
- Combining loop guard and portfast on the same port — these features have directly conflicting assumptions about root/alternate port behavior and must never be enabled together.
- Assuming err-disabled ports recover on their own — by default they stay down until manually re-enabled or `errdisable recovery cause bpduguard` (or the relevant cause) is configured with a recovery interval.
- Forgetting port-priority/port-number tiebreaks are decided by the **upstream** switch (closer to root), not the downstream switch making the choice — tuning the wrong end's `spanning-tree port-priority` has no effect on which link is preferred.
- Deploying UDLD on only one end of a link — both sides must have UDLD enabled for bidirectional monitoring to function at all.
- Forgetting that `spanning-tree vlan <id> root primary/secondary` is a one-time priority calculation at the moment it's run — if a third switch is later configured with an even lower priority, the "primary" root doesn't automatically re-lower itself; the command must be re-run or priorities re-audited.
