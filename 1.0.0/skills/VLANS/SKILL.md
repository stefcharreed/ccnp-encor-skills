---
name: ccnp-VLANS
description: >
  Use this skill when troubleshooting or configuring VLANS on IOS-XE.
  Invoke when the user asks about: VLAN, 802.1Q, trunk port, access port,
  native VLAN, allowed VLANs, broadcast domain, switchport mode.
---

## Purpose
VLANs create multiple logical broadcast domains on a single switch, improving switch port utilization and requiring a router to communicate between VLANs.

## Key Concepts
- VLANs are defined by IEEE 802.1Q, which adds a 32-bit tag to frames: TPID (16 bits, 0x8100), PCP (3 bits, CoS/QoS), DEI (1 bit, drop eligibility), VLAN ID (12 bits, up to 4094 VLANs).
- 802.1Q tags are added only on trunk links — never on access ports or for traffic forwarded locally within a switch.
- VLAN ID ranges: VLAN 0 reserved (802.1p, immutable), VLAN 1 default (immutable), 2–1001 normal range, 1002–1005 reserved (cannot be deleted), 1006–4094 extended range.
- Access ports belong to exactly one VLAN and carry untagged traffic; default access VLAN on a Catalyst port is VLAN 1.
- Trunk ports carry multiple VLANs over one link; the remote switch strips 802.1Q headers and forwards based on MAC address per VLAN.
- Native VLAN is the one trunk VLAN sent untagged; default is VLAN 1. It must match on both ends of a trunk or traffic for that VLAN won't pass correctly. Security hardening: move native VLAN off VLAN 1 to an unused VLAN.
- Allowed VLANs on a trunk restrict which VLANs (and their broadcast/flooded traffic) can cross that link — used for traffic engineering and load balancing across multiple trunks.
- All switch control-plane traffic is advertised on VLAN 1 by default.

## Config Patterns
```ios-xe
! Create VLANs
vlan 10
 name PCs
vlan 20
 name Phones
vlan 99
 name Guests

! Configure access ports
interface GigabitEthernet1/0/15
 switchport mode access
 switchport access vlan 99

interface GigabitEthernet1/0/16
 switchport mode access
 switchport access vlan name Guests

! Configure trunk ports
interface GigabitEthernet1/0/2
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 1,10,20,99

! Safely modify allowed VLANs on an existing trunk (use add/remove, not a bare list)
interface GigabitEthernet1/0/2
 switchport trunk allowed vlan add 30
 switchport trunk allowed vlan remove 99
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Keep user and management traffic off VLAN 1 | VLAN 1 carries switch control-plane traffic by default and can't be deleted; separation shrinks the blast radius | Single-switch lab setups | Cisco IOS hardening guide (doc 13608) |
| Native VLAN moved to a dedicated, unused VLAN on all trunks | Mitigates VLAN-hopping/double-tag attacks; keeps untagged traffic out of live VLANs | Provider handoffs that mandate a specific native VLAN | Cisco IOS hardening guide (doc 13608) |
| Explicit allowed-VLAN lists on trunks, never "all" | Bounds the broadcast/failure domain per link and documents which VLANs belong where | Fast-churn environments where list maintenance causes more outages than it prevents | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| Unused ports shut down and parked in an unused VLAN | Removes live drops as an attack surface | Labs and frequently re-patched environments | Cisco IOS hardening guide (doc 13608) |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show vlan brief` | Port-to-VLAN mappings; confirm ports landed in the expected VLAN |
| `show vlan summary` | Total VLAN count, VTP-participating VLANs, extended-range VLANs in use |
| `show vlan id <vlan-id>` / `show vlan name <vlan-name>` | Filtered detail for one VLAN |
| `show interfaces trunk` | Section 1: trunk status + native VLAN per port; Section 2: VLANs allowed; Section 3: VLANs actually forwarding (not blocked) |
| `show running-config \| begin interface <name>` | Confirm `switchport access vlan` resolved to numeric VLAN ID even if configured by name |

## Intent Questions
- Which VLANs are supposed to exist on this switch — and which are transported through it vs. terminated on it?
- Which ports should be access vs. trunk, and is that by design or by DTP accident?
- What is the native VLAN supposed to be on each trunk, and does anything intentionally ride it untagged?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Physical/Layer 1: interface up/up, correct cabling, no err-disable state.
2. Layer 2 port mode mismatch: one side `access`, other side `trunk`/`dynamic` — causes VLAN leakage or no connectivity. Verify with `show interfaces trunk` and `show interfaces switchport`.
3. Native VLAN mismatch between trunk ends — causes CDP/native VLAN mismatch errors and traffic for that VLAN to fail; confirm both sides with `show interfaces trunk`.
4. Allowed VLAN list missing the required VLAN on one or more trunk hops — check `show interfaces trunk` Section 2 against the expected VLAN path end-to-end.
5. VLAN not created on a downstream switch, or pruned by VTP — check `show vlan brief` on every switch in the path.
6. Config error: `switchport trunk allowed vlan <list>` issued without `add`/`remove`, silently overwriting and dropping previously allowed VLANs.
7. Software/platform bug or hardware TCAM limit (rare) — check bug toolkit only after config and topology are confirmed correct.

## Common Pitfalls
- Using `switchport trunk allowed vlan <vlan-ids>` to "add" a VLAN — it overwrites the whole list and silently blackholes the omitted VLANs. Always use `add`/`remove` for incremental changes, especially in scripts.
- Forgetting that two hosts on the same VLAN/subnet can still talk even when one is on an access port and the other on a trunk's native VLAN — not a tag mismatch, but not best practice either (no 802.1Q tag in either case).
- Leaving the native VLAN at the default VLAN 1, which also carries all switch control-plane traffic — a security hardening gap.
- Assuming `switchport access vlan name <vlan-name>` stores the name in config — it always resolves to and stores the numeric VLAN ID.
