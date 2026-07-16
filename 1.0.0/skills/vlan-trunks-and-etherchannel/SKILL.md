---
name: ccnp-vlan-trunks-and-etherchannel
description: >
  Use this skill when troubleshooting or configuring vlan-trunks-and-etherchannel on IOS-XE.
  Invoke when the user asks about: VTP, VLAN Trunking Protocol, DTP, Dynamic
  Trunking Protocol, EtherChannel, port channel, LACP, PAgP, channel-group,
  load-balancing hash.
---

## Purpose
VTP centrally propagates VLAN definitions across a domain instead of configuring each switch manually, DTP negotiates whether a link becomes a trunk, and EtherChannel bundles multiple physical links into one logical interface so STP and routing see one link instead of several redundant ones.

## Key Concepts
- VTP roles: **Server** (creates/modifies/deletes VLANs, advertises them domain-wide), **Client** (receives and applies VTP advertisements, cannot configure VLANs locally), **Transparent** (forwards VTP advertisements but keeps its own independently-configured local VLAN database), **Off** (does not participate in or forward VTP advertisements at all; local VLANs only).
- VTP Versions 1/2 only propagate VLANs 1–1005; Version 3 supports the full 1–4094 range and requires designating one server as the **primary** server (`vtp primary`) — only the primary can add/remove VLANs in v3, unlike v1/v2 where any server could.
- VTP advertisement types: **Summary** (every 300s or on any VLAN change — version, domain, revision number, timestamp), **Subset** (sent after a VLAN change — full detail needed to apply it), **Client request** (a switch asks for a Subset advertisement after seeing a Summary with a higher revision than its own).
- The VTP configuration revision number is the trust mechanism — a switch with a stale but *higher* revision number than the real domain can overwrite/delete VLANs domain-wide the moment it joins. Always reset revision to 0 (by changing the VTP domain name, then changing it back, or via newer reset methods) before connecting any switch with prior VTP history to a production domain.
- DTP negotiates trunk formation dynamically and requires the VTP domain to match between the two switches; it advertises every 30 seconds. Modes: **Trunk** (statically trunk, still sends DTP to try to bring the far end up dynamically), **Dynamic desirable** (actively tries to negotiate a trunk), **Dynamic auto** (only responds, never initiates — the Catalyst default). Auto-auto on both ends never forms a trunk. See Reference Tables for the full negotiation matrix.
- `switchport nonegotiate` stops a statically-set trunk port from sending DTP at all — best practice is to fix both ends as explicit `access` or `trunk` and use `nonegotiate` to remove ambiguity entirely.
- EtherChannel (port channel, IEEE 802.3AD) bundles physical "member interfaces" into one logical link; STP and routing protocols see only the logical interface, so a member link flapping doesn't trigger a topology recalculation as long as at least one member stays up. Works for Layer 2 (access/trunk) or Layer 3 (routed) forwarding.
- Static EtherChannel (`mode on`) has no health-check mechanism — if a member's physical medium degrades while line protocol stays up (e.g., a failure on one leg through intermediary optical/DWDM equipment), the switch keeps forwarding traffic onto a broken path. Dynamic protocols (LACP/PAgP) detect this because the adjacency itself fails, not just line protocol.
- PAgP (Cisco proprietary) modes: **Auto** (responds only, no adjacency if both ends are auto), **Desirable** (initiates, forms adjacency with auto or desirable). LACP (open standard) modes: **Passive** (responds only, no adjacency if both ends passive), **Active** (initiates, forms adjacency with active or passive).
- Member interface settings that must match across every interface in an EtherChannel:
  - **Port type** — every port must be consistently Layer 2 (switchport) or Layer 3 (routed, `no switchport`).
  - **Port mode** — Layer 2 port channels must be all-access or all-trunk; the two cannot be mixed within one channel.
  - **Native VLAN** — Layer 2 trunk members must share the same `switchport trunk native vlan <vlan-id>`.
  - **Allowed VLAN** — Layer 2 trunk members must share the same `switchport trunk allowed vlan-ids`.
  - **Speed** — all members must run the same speed.
  - **Duplex** — all members must run the same duplex.
  - **MTU** — all Layer 3 members must share the same MTU; a mismatched MTU blocks the interface from being added to the channel at all.
  - **Load interval** — must be configured identically across members.
  - **Storm control** — settings must be configured identically across members.
- LACP fast (`lacp rate fast`, 1s advertisement) reduces failure detection from ~90s (default 30s interval × 3 missed) to ~3s — but both ends must use the same rate (fast or slow) or the EtherChannel won't come up.
- `port-channel min-links` sets a floor on active member interfaces before the port channel comes up at all; `lacp max-bundle` caps how many members are actively forwarding (extras become hot-standby) — useful for keeping active member counts at powers of two for clean load-balancing hashes.
- LACP system priority picks which switch is "primary" and therefore controls which members are active vs hot-standby when there are more members than the max-bundle allows; lower priority value wins. LACP port priority does the same job at the interface level on the primary switch; lower value preferred, tiebreak by lower interface number.
- Load balancing across member links is hash-based (consistent per-flow), not round-robin per-packet — configured globally with `port-channel load-balance <hash>`. Choosing a hash that includes a field that never varies (e.g., a router's MAC address as the only factor) defeats load balancing; prefer IP- or L4-port-based hashes for router-facing channels. Member link counts should be powers of two (2, 4, 8) since the hash is a binary function.

## Procedure
Configuring VTP:
1. Set the VTP version: `vtp version {1 | 2 | 3}`.
2. Set the VTP domain name: `vtp domain <domain-name>` — note this resets the local switch's configuration revision number to 0.
3. Set the VTP mode: `vtp mode {server | client | transparent | off}`.
4. (Optional, recommended) Secure the domain with a password: `vtp password <password>`.
5. (VTP Version 3 only) Designate the primary server with the privileged exec command `vtp primary` — only needed once per domain, run on the intended primary server.

## Reference Tables
DTP dynamic trunk negotiation matrix — successful trunk formation by mode pairing:

| Switch 1 \ Switch 2 | Trunk | Dynamic Desirable | Dynamic Auto |
|---|---|---|---|
| Trunk | Trunk forms | Trunk forms | Trunk forms |
| Dynamic Desirable | Trunk forms | Trunk forms | Trunk forms |
| Dynamic Auto | Trunk forms | Trunk forms | No trunk (both passive) |

Logical EtherChannel (port-channel) interface status fields (`show etherchannel summary`):

| Field | Description |
|---|---|
| U | The EtherChannel interface is working properly |
| D | The EtherChannel interface is down |
| M | At least one LACP adjacency exists, but active members are below the configured `port-channel min-links` minimum — traffic will not forward |
| S | The EtherChannel is configured for Layer 2 switching |
| R | The EtherChannel is configured for Layer 3 routing |

EtherChannel member interface status fields (`show etherchannel summary`):

| Field | Description |
|---|---|
| P | The interface is actively participating and forwarding traffic |
| H | Max active interfaces reached — this member is a hot-standby (LACP adjacency up, not forwarding); from `lacp max-bundle` |
| I | No LACP activity detected on this interface — treated as an individual link, not bundled |
| w | Waiting on a packet from this neighbor to confirm it's still alive |
| s | The member interface is in a suspended state |
| r | The module associated with this interface has been removed from the chassis |

## Config Patterns
```ios-xe
! VTP server configuration (v3)
vtp version 3
vtp domain CISCO
vtp mode server
vtp password PASSWORD
end
vtp primary

! VTP client configuration
vtp version 3
vtp domain CISCO
vtp mode client
vtp password PASSWORD

! VTP transparent configuration
vtp domain CISCO
vtp mode transparent
vtp password PASSWORD

! DTP — dynamic auto on one side, dynamic desirable on the other
interface GigabitEthernet1/0/2
 switchport mode dynamic auto
interface GigabitEthernet1/0/1
 switchport mode dynamic desirable

! Best practice: fixed trunk with negotiation disabled
interface GigabitEthernet1/0/2
 switchport mode trunk
 switchport nonegotiate

! LACP EtherChannel — active on one side, passive on the other
interface range GigabitEthernet1/0/1-2
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk

! PAgP EtherChannel with non-silent mode
interface range GigabitEthernet1/0/3-4
 channel-group 2 mode desirable non-silent

! Static EtherChannel (no health check — avoid unless dynamic protocol unsupported)
interface range GigabitEthernet1/0/5-6
 channel-group 3 mode on

! LACP tuning: fast rate, min/max links, system and port priority
interface range GigabitEthernet1/0/1-2
 lacp rate fast
interface Port-channel1
 port-channel min-links 2
 lacp max-bundle 1
lacp system-priority 1
interface GigabitEthernet1/0/2
 lacp port-priority 1

! EtherChannel load-balancing hash (global)
port-channel load-balance src-dst-mixed-ip-port
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| DTP disabled: ports hardcoded `access` or `trunk`, plus `switchport nonegotiate` | Prevents negotiation-based VLAN hopping and accidental trunks | Lab convenience | Cisco IOS hardening guide (doc 13608) |
| VTP transparent/off unless VTP is deliberately operated (v3 with a designated primary) | A stale switch with a higher revision number can wipe the domain's VLAN database | Disciplined VTPv3 deployments with a designated primary server | Cisco IOS hardening guide (doc 13608) |
| LACP (active) over PAgP or static `on` | Open standard, and adjacency failure catches degraded members that `on` mode keeps forwarding into | NIC teams or appliances that only support static bundling | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| Load-balance hash includes fields that actually vary; member counts at powers of two | A non-varying hash field concentrates all traffic on one member | Two-member channels where nearly any hash spreads acceptably | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show vtp status` | VTP version, domain name, operating mode, VLAN count, and **Configuration Revision** — confirm revision matches across the domain and the mode is what's expected |
| `show interface <id> trunk` | Trunk Mode column (`on`/`desirable`/`auto`), native VLAN, and allowed VLANs per trunk port |
| `show interfaces <id> switchport \| i Trunk` | Administrative/Operational Trunking Encapsulation and Negotiation of Trunking (On/Off) for one port |
| `show etherchannel summary` | Port-channel status flags (expect `SU`) and per-member flags (`P` for active, `H` for hot-standby, `I` for individual/not bundled) |
| `show interfaces port-channel <id>` | Combined bandwidth of all active members, member interface list, up/down state |
| `show etherchannel port` | Detailed local config plus neighbor LACP/PAgP packet info per member — verbose, use targeted commands below for faster triage |
| `show lacp neighbor [detail]` | Neighbor system ID, system priority, fast/slow LACPDU interval |
| `show lacp sys-id` | Local LACP system priority and MAC — confirm both switches expect the same primary |
| `show lacp counters` | LACP packets sent/received per member — a non-incrementing Received column with incrementing Sent means the remote end isn't transmitting |
| `show pagp neighbor` / `show pagp counters` | PAgP equivalents of the LACP neighbor/counter commands |
| `show etherchannel load-balance` | The active load-balancing hash algorithm per traffic type (non-IP, IPv4, IPv6) |

## Intent Questions
- Which links are supposed to be trunks — and are they trunks by explicit config or by DTP negotiation?
- Which bundles should exist, with which members, using which protocol (LACP/PAgP/static) — and why that choice?
- Is VTP supposed to be managing VLANs here at all, and if so, which switch is the authority?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1: confirm all member interfaces are physically up and matched on speed/duplex (`show interfaces status`) — mismatched speed/duplex prevents clean bundling.
2. Layer 2 trunk/DTP: `show interface <id> trunk` and `show interfaces <id> switchport` — confirm the negotiated mode and that both ends agree (or that `nonegotiate` + matching static modes are used); also confirm VTP domain matches, since DTP requires it.
3. EtherChannel member state: `show etherchannel summary` — flags other than `P` (e.g. `I` individual, `s` suspended) on a member mean it never joined the bundle; cross-check `show lacp counters` / `show pagp counters` for one-sided packet loss.
4. VTP propagation failure: `show vtp status` on server and client — mismatched domain name, password, or version blocks propagation; a client showing a lower revision than expected may not have received updates, or a rogue switch with a stale higher revision may have just overwritten VLANs domain-wide.
5. Config error: mismatched member interface settings (port mode, native VLAN, allowed VLANs, MTU, storm control) — these must be identical across every member or the bundle won't form correctly or will behave inconsistently.
6. Config error: LACP rate mismatch (`lacp rate fast` on one end, default slow on the other) — prevents the EtherChannel from coming up at all.
7. Config error: forgetting to reset VTP revision to 0 before connecting a switch with prior VTP history — risk of accidental domain-wide VLAN deletion if its stale revision is higher than the real domain's.
8. Bundle formation basics: confirm the member link is strictly point-to-point between only two devices (no intervening hub/switch), that every member port is administratively and operationally active, and that both ends use a compatible mode — either both statically `on`, or one side LACP active with the other active/passive, or one side PAgP desirable with the other auto/desirable.
9. Software/platform bug (rare) — only after trunk negotiation, VTP state, and member interface consistency are all confirmed correct.

## Common Pitfalls
- Connecting a switch with leftover VTP configuration (and a non-zero revision number) to a production VTP domain without resetting its revision first — if that stale revision is higher than the domain's current one, it gets treated as authoritative and can delete VLANs domain-wide.
- Using a static EtherChannel (`mode on`) across intermediary optical/transport equipment (DWDM, taps, L2 firewalls) where a one-way failure doesn't propagate to line protocol — static channels have no health-check, so traffic keeps flowing onto a broken leg; use LACP or PAgP instead so adjacency loss (not just line protocol) drives member removal.
- Choosing a load-balancing hash that includes a field that never varies for a given path (e.g., relying on a router's MAC address as the differentiator) — effectively disables load balancing across that channel even though the hash "looks" configured correctly.
- Building EtherChannels with a member count that isn't a power of two (e.g., 3 active members) — the hash is binary, so traffic distribution degrades compared to 2 or 4 active members.
- Assuming `dynamic auto` on both ends of a link will form a trunk — it never will, since neither side initiates; this is the one DTP combination guaranteed to fail.
- Forgetting that `port-channel min-links` and `lacp max-bundle` only need to be configured on one switch (the primary) to function, but leaving them unset on the peer makes troubleshooting confusing later — configure on both sides as a practice even though it's not strictly required.
