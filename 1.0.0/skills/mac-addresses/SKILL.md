---
name: ccnp-mac-addresses
description: >
  Use this skill when troubleshooting or configuring mac-addresses on IOS-XE.
  Invoke when the user asks about: MAC address table, CAM, collision domain,
  unknown unicast flooding, switchport status, static MAC entry, CSMA/CD,
  OUI.
---

## Purpose
The MAC address table lets a switch forward frames only out the port where the destination MAC actually lives, instead of flooding every port, which shrinks collision domains and enables full duplex.

## Key Concepts
- A switch builds its MAC address table dynamically by reading the **source** MAC address of every frame it receives and associating it to the ingress port and VLAN.
- MAC address is 48 bits / 6 octets in hex. First 3 octets = OUI (vendor-assigned), last 3 octets = vendor-assigned unique device ID.
- Broadcast `FF:FF:FF:FF:FF:FF` is processed by every device on the segment and is not forwarded past a Layer 3 boundary.
- Collision domain = a segment where only one device can transmit at a time (CSMA/CD); hubs put every connected device in one shared collision domain (half-duplex), switches give each port its own collision domain (full duplex).
- Unknown unicast flooding happens when a destination MAC isn't yet in the table — the switch floods the frame out all ports in that VLAN until it learns the real port.
- A port showing multiple learned MACs means a hub, switch, or VM host (virtual switch) is connected downstream of that port, not a single end host.
- The MAC table lives in CAM (content-addressable memory) — purpose-built fast-lookup memory, not general RAM, so large tables can be searched at line rate.
- Static MAC entries pin a MAC to a port (or instruct the switch to drop traffic to it) — used for legacy load-balancing setups to stop unknown unicast flooding for a known address.

## Config Patterns
```ios-xe
! Add a static MAC address table entry tied to a port
mac address-table static 0011.2233.4455 vlan 10 interface GigabitEthernet1/0/5

! Add a static entry that drops traffic to that MAC (e.g. known bad actor)
mac address-table static 0011.2233.4455 vlan 10 drop

! Flush the entire MAC address table
clear mac address-table dynamic

! Flush only a specific interface, VLAN, or MAC address
clear mac address-table dynamic interface GigabitEthernet1/0/5
clear mac address-table dynamic vlan 10
clear mac address-table dynamic address 0011.2233.4455
```

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show mac address-table` | Full table: VLAN, MAC, type (static/dynamic), port — confirm device-to-port mapping |
| `show mac address-table dynamic` | Only learned (not static) entries |
| `show mac address-table address <mac>` | Confirm exactly which port/VLAN one known MAC is learned on |
| `show mac address-table vlan <vlan-id>` | All MACs learned within one VLAN |
| `show interfaces <id> switchport` | Operational mode (access/trunk/down), access VLAN, native VLAN for one port |
| `show interfaces status` | Condensed per-port view: Status (connected/notconnect/err-disabled), VLAN, Duplex, Speed, Type — fast first-look for down/misconfigured ports |

## Troubleshooting Checklist
1. Layer 1: `show interfaces status` — confirm `connected`, not `notconnect` or `err-disabled`.
2. Layer 2 learning: `show mac address-table address <mac>` — confirm the expected MAC is learned on the expected port/VLAN; if missing, traffic from that host never reached this switch.
3. Unexpected multiple MACs on one port — port is a switch/hub/hypervisor uplink, not an end host; trace downstream to find the real device.
4. Port mode/VLAN mismatch — check `show interfaces <id> switchport` for Administrative Mode vs Operational Mode and Access Mode VLAN.
5. Static MAC entry conflicting with a dynamically learned one, or a stale entry after a device moved ports — `clear mac address-table dynamic` for that MAC/interface and relearn.
6. CAM table exhaustion on very large switches (rare, high MAC count) — check platform-specific TCAM/CAM utilization only after the above are ruled out.

## Common Pitfalls
- Confusing `show interfaces switchport` (per-port detail, requires interface ID) with `show interfaces status` (condensed, all ports) — use the latter for a fast sweep, the former for deep-dive on one port.
- Assuming a quiet/idle MAC table entry means the host is gone — dynamic entries simply age out after inactivity, which can look like a flush but isn't a fault.
- Forgetting static entries don't age out and can mask a device move (host plugged into a new port) until the static entry is manually cleared/updated.
- Treating broadcast flooding and unknown unicast flooding as the same thing — broadcast is always flooded by design; unknown unicast flooding is supposed to be temporary, until the switch learns the destination port.
