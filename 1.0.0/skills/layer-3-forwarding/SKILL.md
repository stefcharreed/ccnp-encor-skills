---
name: ccnp-layer-3-forwarding
description: >
  Use this skill when troubleshooting or configuring layer-3-forwarding on IOS-XE.
  Invoke when the user asks about: ARP table, routing table, RIB, next-hop,
  routed subinterface, SVI, routed switch port, no switchport, ip address
  secondary, administrative distance.
---

## Purpose
Layer 3 forwarding decides whether a packet is delivered directly on the local subnet via ARP-resolved MACs, or routed hop-by-hop toward a remote subnet by rewriting Layer 2 headers at each router.

## Key Concepts
- Forwarding is decided top-down: Layer 3 routing logic runs before Layer 2 framing on the sending device.
- Two paths only: same-subnet (local, resolved via ARP) or different-subnet (routed via the routing table / RIB).
- ARP maps IP→MAC for hosts on the same Layer 2 segment; the requesting host broadcasts an ARP request, only the owner unicasts back an ARP reply, and the table entry is cached until it goes stale and ages out.
- The ARP table never holds entries for remote-network hosts directly — it holds the entry for the *next-hop* IP needed to reach that remote network.
- Next-hop IP can come from a static route, a default-gateway (catch-all static default route), or a routing protocol.
- At each router hop: source MAC is rewritten to the outbound interface's MAC, destination MAC is rewritten to the next-hop's (or final destination's) MAC — the IP source/destination addresses never change in transit.
- Directly connected, "up" interfaces with an IP address are injected into the RIB with administrative distance 0 — no static route or routing protocol can ever override a connected route.
- An interface can carry multiple IPv4 networks via `secondary` addresses; IPv6 supports multiple addresses on one interface natively, just by repeating the `ipv6 address` command.
- Three ways to put an SVI/routed interface where VLANs need routing: a router-on-a-stick subinterface (`encapsulation dot1Q <vlan>`) on a trunk, a switch SVI (`interface vlan <id>`, requires an up port in that VLAN), or a routed switch port (`no switchport`) for point-to-point links — the last avoids burning a "transit VLAN" that could leak elsewhere in the L2 domain or get hit by spanning tree.

## Procedure
ARP resolution for same-subnet (local) delivery:
1. The sending host needs the destination's MAC address but only knows the destination IP — it checks the local ARP table first.
2. If no entry exists, the host broadcasts an ARP request to the entire Layer 2 segment asking who owns that IP.
3. Every host on the segment receives the broadcast, but only the host owning that IP address replies.
4. The owning host sends a unicast ARP reply containing its IP and MAC address.
5. The requesting host updates its local ARP table with the new IP→MAC mapping.
6. The requesting host adds the resolved MAC as the destination MAC in the Layer 2 header and forwards the original packet.

Per-hop rewrite for routed (remote-subnet) delivery:
1. The source device determines the destination is on a different network and looks up the next-hop IP in its routing table (static route, default gateway, or routing protocol).
2. The source resolves the next-hop IP to a MAC via its ARP table, then adds Layer 2 headers using its own MAC as source and the next-hop's MAC as destination.
3. The next router receives the frame based on the destination MAC, strips the Layer 2 header, and inspects the destination IP.
4. The router looks up the destination network in its own routing table and identifies the outbound interface.
5. The router resolves the MAC for the destination device (or the next next-hop router) via ARP.
6. The router rewrites the source MAC to its own outbound interface's MAC and the destination MAC to the resolved MAC, then forwards.
7. This repeats at every hop until the frame arrives at the segment where the destination host lives.

## Config Patterns
```ios-xe
! Routed physical interface with primary + secondary IPv4, and IPv6
interface GigabitEthernet0/0/0
 ip address 10.10.10.254 255.255.255.0
 ip address 172.16.10.254 255.255.255.0 secondary
 ipv6 address 2001:db8:10::254/64

! Router-on-a-stick: routed subinterfaces over a trunk
interface GigabitEthernet0/0/1.10
 encapsulation dot1Q 10
 ip address 10.10.10.2 255.255.255.0
 ipv6 address 2001:db8:10::2/64
interface GigabitEthernet0/0/1.99
 encapsulation dot1Q 99
 ip address 10.20.20.2 255.255.255.0
 ipv6 address 2001:db8:20::2/64

! Switched Virtual Interface (SVI) for inter-VLAN routing on a multilayer switch
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 ipv6 address 2001:db8:10::1/64
 no shutdown

! Routed switch port (Layer 3 point-to-point link, no transit VLAN needed)
interface GigabitEthernet1/0/14
 no switchport
 ip address 10.20.20.1 255.255.255.0
 ipv6 address 2001:db8:20::1/64
 no shutdown
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Routed switch ports (`no switchport`) for point-to-point L3 links, not transit VLAN + SVI | Removes STP exposure and VLAN leakage from links that are purely Layer 3 | Link must carry more than one VLAN, or the platform lacks routed-port support | Campus LAN & WLAN Design Guide (Cisco Design Zone) |
| One primary IPv4 subnet per interface; `secondary` addresses are migration state, not steady state | Secondaries complicate routing protocols, DHCP relay, and troubleshooting | Mid-renumbering migrations; documented legacy multi-net segments | ENCOR 350-401 OCG |
| L2/L3 boundary placed deliberately; VLANs not spanned between closets by default | Smaller failure domains; no cross-closet STP dependence | Applications requiring L2 adjacency (clusters, live migration, legacy protocols) | Campus LAN & WLAN Design Guide (Cisco Design Zone) |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ip arp [mac-address \| ip-address \| vlan vlan-id \| interface-id]` | Confirm IP→MAC mapping exists for a local host or for the next-hop toward a remote network |
| `show ip interface brief` | Up/up status and assigned IPv4 per interface; pipe `\| exclude unassigned` to cut noise on high-port-count switches |
| `show ip interface brief \| exclude unassigned` | Streamlined list of only interfaces with a configured IPv4 address |
| `show ipv6 interface brief` | Up/up status and assigned IPv6 (link-local + global) per interface |
| `show ipv6 interface brief \| exclude unassigned\|GigabitEthernet` | Streamlined IPv6 view limited to configured, non-Gig-default interfaces |
| `show ip route` | Confirm a route exists to the destination network, and which source (connected/static/protocol, AD) is installed |

## Intent Questions
- Which device is supposed to be the gateway for each VLAN/subnet — and via what construct (SVI, subinterface, routed port)?
- Where is the L2/L3 boundary supposed to sit for this segment?
- Which next-hop should traffic toward a given remote subnet actually be using?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Layer 1/2: confirm the interface or SVI's underlying link is up (`show ip interface brief` — Status/Protocol both `up`); an SVI needs at least one port in that VLAN up.
2. ARP failure: `show ip arp` for the destination (same subnet) or next-hop (remote subnet) — missing entry means address resolution never completed, check for ARP broadcasts being filtered or the target host being down.
3. Routing table: `show ip route` — confirm a route to the destination network exists and points to the expected next-hop/interface; missing route means it's neither connected, statically configured, nor learned dynamically.
4. Config error: missing `secondary` keyword when adding a second IPv4 to the same interface (overwrites instead of adding), or `encapsulation dot1Q <vlan>` mismatched with the actual VLAN on the trunk.
5. Config error: SVI created but the associated VLAN has no up access/trunk port anywhere on the switch, so the SVI itself stays down.
6. Software/platform bug (e.g. CEF/hardware forwarding inconsistency) — check only after route table, ARP, and interface state are all confirmed correct.

## Common Pitfalls
- Forgetting `secondary` on a second `ip address` command — it silently replaces the first IP instead of adding a second one.
- Assuming the ARP table holds an entry for every remote host you can reach — it only ever holds the next-hop's entry for remote networks, never the far-end host directly.
- Building a "transit VLAN" + SVI for a switch-to-router point-to-point link instead of using a routed switch port (`no switchport`) — adds unnecessary L2/STP exposure for a link that's purely L3.
- Believing administrative distance can let a static route or routing protocol override a connected interface route — connected routes (AD 0) are always preferred, full stop.
- Reading `show ip interface brief` raw on a high-port switch and missing the configured interfaces in the noise — always pipe `| exclude unassigned` on dense platforms.
