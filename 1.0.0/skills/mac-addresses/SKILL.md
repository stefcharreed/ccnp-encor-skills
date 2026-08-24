---
name: ccnp-mac-addresses
description: >
  Use this skill when troubleshooting or configuring mac-addresses on IOS-XE.
  Invoke when the user asks about: MAC address table, CAM, collision domain,
  unknown unicast flooding, switchport status, static MAC entry, CSMA/CD,
  OUI, MAC aging time, ARP vs CAM timers, gratuitous ARP, GARP, MAC
  flapping, HSRP/VRRP virtual MAC, unicast flooding from silent hosts.
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
- **The CAM table and a router's ARP cache are separate state about the same host, at different layers, with different timers.** CAM maps MAC→port (L2); ARP maps IP→MAC (L3). Nothing synchronizes them, and their defaults don't match — which is the root of most "unexplained flooding" cases.
- **Default aging times differ by platform** — Catalyst IOS/IOS-XE MAC aging is **300s**, but a router's ARP cache is **4 hours (14400s)**. Nexus raised its MAC default to **1800s** on the 5600/6000/7000 specifically to reduce this mismatch (the L2-only N5500 stayed at 300s, since it has no ARP cache of its own to drift against).
- **The classic flooding mechanism:** a host that only *responds* and never initiates goes quiet → its CAM entry ages out at 5 minutes → the router's ARP entry stays valid for another ~3h55m → the router keeps unicasting to that MAC → the switch no longer knows the port → **every one of those unicast frames is flooded to the whole VLAN.** Asymmetric routing makes it worse, because return traffic never traverses the switch to refresh the CAM entry.
- **Gratuitous ARP (GARP) updates the CAM table as a side effect, not by design.** A GARP is an L2 **broadcast** (`FF:FF:FF:FF:FF:FF`) where sender IP == target IP. The switch never parses the ARP payload — it updates CAM purely from the **Ethernet source address**, via ordinary source-MAC learning. Other *hosts* update their ARP caches from the **payload**. One frame, two mechanisms, two layers.
- GARP is broadcast precisely so it floods the entire VLAN — an announcement only works if every device in the broadcast domain hears it. It does not cross an L3 boundary, so a GARP in VLAN 10 never reaches VLAN 20.
- **HSRP/VRRP virtual MACs** appear in the CAM table owned by whichever router is Active: HSRP `0000.0c07.acXX` (XX = group in hex), HSRPv2 `0000.0c9f.fXXX`, VRRP `0000.5e00.01XX`. On failover the new Active sends a GARP so switches relearn that MAC on the new port — **if that GARP is lost or arrives while the port is still in STP listening/learning, traffic black-holes until the CAM entry ages out.**

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

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Dynamic learning only; static MAC entries reserved for documented needs | Static entries never age out and mask device moves | Legacy setups that otherwise flood constantly; deliberate `drop` entries for known-bad MACs | ENCOR 350-401 OCG |
| MAC flapping gets investigated, not cleared | Flapping usually means a loop, a mis-cabled redundant path, or spoofing | Expected during VM live migration or host failover | ENCOR 350-401 OCG |
| One MAC (or phone+PC pair) expected per access port | Multiple MACs on an access port usually means an undocumented switch/hub/AP downstream | Hypervisor uplinks and documented daisy-chained gear | Cisco IOS hardening guide (doc 13608) |
| ARP timeout aligned to MAC aging time on routed VLANs (e.g. `arp timeout 300` on the SVI) | Default 4hr ARP vs 300s CAM causes sustained unicast flooding for silent hosts | Nexus platforms already default MAC aging to 1800s; environments with no silent hosts or no asymmetric paths | ENCOR 350-401 OCG |
| PortFast on access ports carrying servers/VMs | A GARP sent during STP listening/learning is lost, so post-failover traffic black-holes until CAM ages out | Ports where BPDU Guard/loop risk outweighs the convergence gain | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show mac address-table` | Full table: VLAN, MAC, type (static/dynamic), port — confirm device-to-port mapping |
| `show mac address-table dynamic` | Only learned (not static) entries |
| `show mac address-table address <mac>` | Confirm exactly which port/VLAN one known MAC is learned on |
| `show mac address-table vlan <vlan-id>` | All MACs learned within one VLAN |
| `show interfaces <id> switchport` | Operational mode (access/trunk/down), access VLAN, native VLAN for one port |
| `show interfaces status` | Condensed per-port view: Status (connected/notconnect/err-disabled), VLAN, Duplex, Speed, Type — fast first-look for down/misconfigured ports |
| `show mac address-table aging-time` | Current MAC aging (default 300s on Catalyst) — compare against the router's `show ip arp` timers when diagnosing flooding |
| `show mac address-table count` | Entry counts per VLAN and total — spot CAM exhaustion or an unexpectedly empty VLAN |
| `show mac address-table address 0000.0c07.acXX` | Which port an **HSRP virtual MAC** is learned on — run before and after a failover to confirm the GARP actually moved it |

## Intent Questions
- Which hosts (MACs) are supposed to be learned on which ports and VLANs?
- Is flooding here expected (silent host, freshly booted switch) or a symptom?
- Are any static MAC entries supposed to exist — and is the reason still documented?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
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
- **Diagnosing sustained unicast flooding as an L2 problem when the cause is the router's ARP timer.** If flooding persists for one specific destination, compare MAC aging (300s) against ARP timeout (4hr) before hunting for loops — the switch is behaving correctly; the router is the one holding stale state.
- Assuming a GARP is "sent to the CAM table" — it's broadcast onto the wire, and the switch learns from the **Ethernet source address** like any other frame. The ARP payload is only read by hosts.
- Expecting every host to honor a GARP — **many stacks only *update* an existing ARP entry and won't *create* one**, so behavior varies across a mixed OS environment and failover can look inconsistent for reasons that aren't the network's fault.
- Seeing an HSRP/VRRP virtual MAC in the CAM table and hunting for the "missing" device — no physical interface owns it; it moves between routers on failover by design.
