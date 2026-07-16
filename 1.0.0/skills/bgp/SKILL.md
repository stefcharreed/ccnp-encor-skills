---
name: ccnp-bgp
description: >
  Use this skill when troubleshooting or configuring bgp on IOS-XE.
  Invoke when the user asks about: BGP neighbor sessions, eBGP, iBGP, AS_Path,
  BGP path attributes, BGP route summarization/aggregate-address, multiprotocol
  BGP for IPv6, BGP neighbor states, Loc-RIB/Adj-RIB-In/Adj-RIB-Out.
---

## Purpose
BGP is the path-vector EGP used to exchange routes between autonomous systems
(and within one, via iBGP) — it controls how an organization connects to the
Internet or stitches together multiple internal routing domains, using path
attributes instead of a simple metric to pick the best route.

## Key Concepts
- **Autonomous System (AS):** a collection of routers under one organization's
  control. Public ASNs come from IANA; private ASNs are 64,512–65,534 (16-bit)
  or 4,200,000,000–4,294,967,294 (32-bit) and must never be exposed on the
  public Internet.
- **Path attributes (PAs):** classified as well-known mandatory, well-known
  discretionary, optional transitive, or optional non-transitive. Well-known
  mandatory PAs (like AS_Path and Next Hop) must be on every prefix
  advertisement and every implementation must recognize them.
- **AS_Path loop prevention:** a well-known mandatory attribute listing every
  AS the prefix has traversed. A router discards any advertisement where its
  own AS already appears in AS_Path — this is BGP's loop prevention, since BGP
  (a path-vector protocol) has no full topology like a link-state IGP.
- **Address families (AFI/SAFI):** BGP was extended (RFC 2858, MP-BGP) so one
  session can carry multiple protocols — IPv4 unicast (AFI 1/SAFI 1), IPv6
  unicast (AFI 2/SAFI 1), etc. Each AFI/SAFI maintains its own database and
  routing policy even over the same TCP session.
- **TCP port 179, multi-hop capable:** unlike IGP hellos (single-hop only),
  BGP rides TCP and can peer multiple hops away — but the underlying path to
  the peer's IP must already be resolvable in the RIB (static route or IGP);
  a default route is NOT sufficient for multi-hop BGP sessions.
- **iBGP vs eBGP:**
  - iBGP: same AS, AD 200, TTL 255 (multi-hop by default), full mesh required
    within the AS so all routers learn the same routing information (BGP
    routes are NOT redistributed into IGP as a substitute — IGPs can't scale
    to Internet-size tables and can't carry path attributes).
  - eBGP: different AS, AD 20, TTL 1 by default (single-hop unless
    ebgp-multihop configured). Advertising router rewrites next-hop to itself
    and prepends its own ASN to AS_Path on the way out.
- **BGP messages:** OPEN (establishes session, negotiates hold time/RID/
  capabilities), KEEPALIVE (sent every 1/3 of hold timer, default 60s with a
  180s default hold timer), UPDATE (advertises/withdraws routes, also resets
  hold timer), NOTIFICATION (error → session torn down).
- **BGP neighbor FSM states (in order):** Idle → Connect → Active → OpenSent →
  OpenConfirm → Established. Established is where UPDATE messages actually
  exchange routes.
- **Three route tables per peer relationship:**
  - Adj-RIB-In — routes as received, before inbound policy (purged after
    processing to save memory).
  - Loc-RIB — the actual BGP table; everything that passed validity checks,
    after best-path selection. This is what's shown by `show bgp afi safi`.
  - Adj-RIB-Out — what's actually advertised to a specific peer, after
    outbound policy. BGP only advertises the best path per peer by default.
- **network statement ≠ enabling an interface.** It just tells BGP to search
  the RIB for an exact prefix/mask match and install it into Loc-RIB if found
  — works for connected, static, or any IGP-learned route.
- **Route summarization (aggregate-address):** reduces table size and hides
  downstream link flaps from upstream routers. Two methods: static (route to
  Null0 + network statement — always advertised even if components are gone)
  or dynamic (`aggregate-address`, only appears while a contributing component
  route exists). `summary-only` suppresses the more-specific component routes;
  without it, both summary and components are advertised. The originating
  router installs a discard route to Null0 for the summary, for loop
  prevention.
- **Atomic aggregate / AS_SET:** summarizing a route drops the original
  AS_Path info by default and sets the `atomic-aggregate` attribute to signal
  that loss. The `as-set` keyword preserves the original ASNs (wrapped in
  braces, counted as a single AS_Path hop) — but watch for self-loop discards
  when the AS_SET ends up containing the receiving router's own ASN.

## Config Patterns
```ios-xe
! --- Basic eBGP session (R1, default IPv4 AFI enabled) ---
router bgp 65100
 bgp router-id 192.168.1.1
 neighbor 10.12.1.2 remote-as 65200
 network 10.12.1.0 mask 255.255.255.0
 network 192.168.1.1 mask 255.255.255.255

! --- Same, with default IPv4 AFI explicitly disabled (R2) ---
router bgp 65200
 bgp router-id 192.168.2.2
 no bgp default ipv4-unicast
 neighbor 10.12.1.1 remote-as 65100
 !
 address-family ipv4
  network 10.12.1.0 mask 255.255.255.0
  network 192.168.2.2 mask 255.255.255.255
  neighbor 10.12.1.1 activate

! --- Redistributing an IGP into BGP (safe direction) ---
router bgp 65100
 redistribute ospf 1
 neighbor 10.12.1.2 remote-as 65200

! --- IPv4 route summarization, suppressing components ---
router bgp 65100
 aggregate-address 172.16.0.0 255.255.240.0 summary-only
 redistribute connected
 neighbor 10.12.1.2 remote-as 65200

! --- Summarization preserving AS_Path history ---
router bgp 65200
 address-family ipv4
  aggregate-address 192.168.0.0 255.255.0.0 as-set summary-only
  aggregate-address 172.16.0.0 255.255.240.0 as-set summary-only

! --- IPv6 BGP (MP-BGP), RID must be set statically if IPv6-only ---
router bgp 65100
 bgp router-id 192.168.1.1
 no bgp default ipv4-unicast
 neighbor 2001:DB8:0:12::2 remote-as 65200
 !
 address-family ipv6
  neighbor 2001:DB8:0:12::2 activate
  redistribute connected

! --- IPv6 route summarization ---
router bgp 65200
 address-family ipv6
  network 2001:DB8::2/128
  aggregate-address 2001:DB8::/58 summary-only
  neighbor 2001:DB8:0:12::1 activate
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Explicit inbound and outbound filtering on every eBGP session | Never accept or advertise "everything" — protects against route leaks and accidental transit | Internal test ASes in a lab | Cisco IOS hardening guide (doc 13608) |
| iBGP peering on loopbacks with `update-source` | The session survives any single link failure | Directly connected single-link designs where the session should die with the link | ENCOR 350-401 OCG |
| iBGP full mesh or a deliberate route-reflector/confederation design | iBGP-learned routes aren't re-advertised to other iBGP peers; a missing session silently blackholes | Two-router ASes where "full mesh" is one session | ENCOR 350-401 OCG |
| Session authentication on all eBGP peerings | Protects the TCP session from spoofed resets and hijack | Peers that can't support it | Cisco IOS hardening guide (doc 13608) |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show bgp ipv4 unicast summary` | Neighbor state/PfxRcd column — a number means Established and prefixes received; otherwise shows FSM state (Idle/Active/etc.) |
| `show bgp ipv4 unicast neighbors <ip>` | `BGP state = Established`, hold/keepalive timers, negotiated AFI/SAFI capabilities |
| `show bgp ipv4 unicast` | `*` = valid path, `>` = best path; check Next Hop, Weight, Path columns |
| `show bgp ipv4 unicast <network>` | All paths for a specific prefix and why one was chosen `best` |
| `show bgp ipv4 unicast neighbors <ip> advertised-routes` | Confirms exactly what's in the Adj-RIB-Out for that peer |
| `show ip route bgp` | Routes actually installed in the RIB — AD 20 (eBGP) or 200 (iBGP) |
| `show tcp brief` | Confirms the underlying TCP session is ESTAB; local port 179 = this router accepted, remote port 179 = this router initiated |
| `show bgp ipv6 unicast summary` | Same as IPv4 summary but for the IPv6 AFI — separate Up/Down and PfxRcd per neighbor |

## Intent Questions
- Which eBGP/iBGP sessions are supposed to exist, to which peer addresses, in which ASes?
- Which prefixes are we supposed to advertise, and which are we supposed to accept — per peer?
- Is this router supposed to be transit for anyone, or is this an edge/stub AS?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Confirm IP reachability to the peer (ping/traceroute) — for multi-hop
   sessions, confirm a route (static or IGP) actually resolves the peer's IP;
   a default route alone will not work.
2. `show tcp brief` — is the TCP session itself up on port 179? If not, BGP
   never gets past Connect/Active.
3. Check `remote-as` on both sides matches what the peer expects, and that
   eBGP TTL (default 1) isn't being dropped by an intermediate hop — use
   `ebgp-multihop` if eBGP peers aren't directly connected.
4. Check BGP router IDs are unique between peers — a duplicate RID blocks
   OpenConfirm.
5. Check hold-time/keepalive mismatches and any configured authentication
   (password) mismatch.
6. If session is Established but no routes: check `address-family` is
   initialized and `neighbor <ip> activate` was issued — IOS XE enables IPv4
   unicast by default but every other AFI needs explicit activation.
7. If routes exist in Loc-RIB but aren't advertised: check outbound route
   policies/route-maps and whether the route passed the next-hop validity
   check.
8. If a summary route isn't suppressing components: confirm `summary-only`
   is present on the `aggregate-address` command.
9. If a summarized route is being silently dropped by a downstream router:
   check whether `as-set` caused the receiving router's own ASN to appear in
   the merged AS_SET, triggering loop discard.

## Common Pitfalls
- Assuming BGP discovers neighbors dynamically like OSPF — it doesn't;
  neighbors must be explicitly configured by IP address.
- Forgetting `neighbor <ip> activate` after enabling a non-default address
  family (very common with IPv6, since `no bgp default ipv4-unicast` is
  often copy-pasted without realizing IPv6 always needs explicit activation).
- Redistributing BGP routes into an IGP "to fix reachability" — BGP can
  carry the entire Internet table; most IGPs become unstable well before
  that scale. Redistributing an IGP into BGP is safe; the reverse direction
  needs real caution.
- Forgetting that `aggregate-address` without `summary-only` advertises
  *both* the summary and every component route — looks like summarization
  worked but the table didn't actually shrink.
- Expecting iBGP to behave like an IGP for forwarding within the AS — iBGP
  requires a full mesh (or route reflectors/confederations, covered later)
  because iBGP-learned routes are not re-advertised to other iBGP peers by
  default.
- Mixing up which AD applies — eBGP routes get AD 20, iBGP routes get AD
  200, so a static route or even some IGP routes can outrank an iBGP path
  for the same prefix.
