---
name: ccnp-bgp-advanced
description: >
  Use this skill when troubleshooting or configuring advanced BGP route
  filtering and manipulation on IOS-XE. Invoke when the user asks about:
  BGP multihoming, transit AS, distribute lists, prefix lists, AS_Path
  ACLs, route maps for BGP, BGP communities, BGP best-path selection.
---

## Purpose
Advanced BGP covers how to avoid an enterprise becoming an unintended
transit AS, how to conditionally match and manipulate routes (ACLs, prefix
lists, regex, route maps), how to tag routes with communities for policy
control across AS boundaries, and the deterministic algorithm BGP uses to
pick a single best path among multiple candidates.

## Key Concepts
- **BGP multihoming:** adding a second circuit/session for redundancy.
  Four common scenarios range from same-SP dual links (protects against
  link failure only) to dual eBGP to different SPs plus an iBGP session
  between the enterprise's own edge routers (protects against link, SP,
  and single-router failure, and optimizes routing).
- **Transit AS risk:** if an enterprise multihomes to two SPs without
  outbound filtering, it can end up advertising routes learned from SP1
  to SP2 (and vice versa), turning the enterprise into free transit
  routing for Internet traffic that has nothing to do with it — this
  saturates the enterprise's own links. Fix: outbound route filtering so
  only locally-originated routes are advertised externally.
- **Branch transit routing:** in multi-site MPLS designs preferring one
  SP (active/passive via local preference), a link failure can make one
  branch router transit traffic for another branch through the WAN,
  causing oversaturated circuits and nondeterministic (asymmetric)
  routing. Fix: branch routers should not advertise WAN-learned routes
  back out to the WAN — only advertise LAN-facing prefixes.
- **Conditional matching tools:** extended ACLs (source field = network,
  destination field = network mask — NOT normal source/destination
  matching), prefix lists (high-order bit pattern + high-order bit count,
  with `ge`/`le` length-range qualifiers), regex (`^` start, `$` end, `_`
  space/boundary, used to match AS_Path), and AS_Path ACLs (regex applied
  to the AS_Path attribute, numbered 1–500).
- **Prefix list matching length rule:** `high-order-bit-count < ge-value
  <= le-value`. A bare prefix (no ge/le) is an exact match.
- **Route map structure (4 parts):** sequence number (processing order),
  conditional match criteria (OR across multiple values on one match line,
  AND across multiple distinct match commands), processing action
  (permit/deny, default permit), optional action (`set` commands that
  modify attributes). A route map with no match statement implicitly
  matches everything. Implicit deny exists at the end, just like ACLs —
  always plan for a final catch-all `permit` sequence with no match
  unless you want a hard deny.
- **`continue` keyword:** lets a route map proceed to a later sequence
  after matching and applying actions in the current one, instead of
  stopping. Rarely used in practice — adds debugging complexity.
- **Four BGP filtering methods (inbound or outbound, per neighbor):**
  distribute list (ACL-based), prefix list, AS_Path ACL/filter-list
  (regex-based), route map (most powerful — can both filter and modify
  attributes). A neighbor cannot use a distribute list and a prefix list
  in the same direction simultaneously.
- **BGP communities:** optional transitive 32-bit path attribute, shown
  as a single decimal number or "new format" `ASN:value` (16-bit:16-bit).
  Communities are NOT sent by default — must enable with `neighbor <ip>
  send-community [standard | extended | both]`. Well-known communities:
  `Internet` (advertise to Internet — not automatic, must be matched by
  policy), `No_Advertise` (never sent to any BGP peer), `Local-AS` (not
  sent to eBGP peers, only confederation peers), `No_Export` (not sent to
  eBGP peers, can go to iBGP peers).
- **Setting communities:** `set community <community> [additive]` — by
  default this overwrites any existing communities; `additive` appends
  instead.
- **BGP best-path algorithm order (memorize this order):** 1) Weight
  (highest, Cisco-only, local, not advertised) → 2) Local preference
  (highest, advertised within the AS only, not to eBGP) → 3) Locally
  originated (network/aggregate beats received) → 4) AIGP (lowest) →
  5) Shortest AS_Path → 6) Lowest origin type (IGP < EGP < Incomplete)
  → 7) Lowest MED (only meaningful comparing paths from the same AS)
  → 8) eBGP over iBGP → 9) Lowest IGP metric to next hop → 10) Oldest
  eBGP session → 11) Lowest router ID → 12) Shortest cluster list →
  13) Lowest neighbor IP address.
- **Longest prefix match still wins over BGP policy.** A router always
  forwards based on the longest matching prefix length first; BGP
  best-path only decides which path wins when multiple paths exist for
  the *same* prefix length. This is exploitable for deterministic traffic
  engineering: advertise a summary from both edge routers and a longer,
  more-specific prefix from only the router that should receive that
  traffic.

## Config Patterns
```ios-xe
! --- Distribute list filtering (extended ACL: source=network, dest=mask) ---
ip access-list extended ACL-ALLOW
 permit ip 192.168.0.0 0.0.255.255 host 255.255.255.255
 permit ip 100.64.0.0 0.0.255.0 host 255.255.255.128
!
router bgp 65100
 neighbor 10.12.1.2 remote-as 65200
 address-family ipv4
  neighbor 10.12.1.2 activate
  neighbor 10.12.1.2 distribute-list ACL-ALLOW in

! --- Prefix list filtering ---
ip prefix-list RFC1918 seq 5 permit 192.168.0.0/16 ge 32
ip prefix-list RFC1918 seq 10 deny 0.0.0.0/0 ge 32
ip prefix-list RFC1918 seq 15 permit 10.0.0.0/8 le 32
ip prefix-list RFC1918 seq 20 permit 172.16.0.0/12 le 32
ip prefix-list RFC1918 seq 25 permit 192.168.0.0/16 le 32
!
router bgp 65100
 address-family ipv4 unicast
  neighbor 10.12.1.2 prefix-list RFC1918 in

! --- AS_Path ACL: restrict outbound advertisements to local-origin only ---
ip as-path access-list 1 permit ^$
!
router bgp 65200
 neighbor 10.12.1.1 remote-as 65100
 neighbor 10.23.1.3 remote-as 65300
 address-family ipv4 unicast
  neighbor 10.12.1.1 activate
  neighbor 10.23.1.3 activate
  neighbor 10.12.1.1 filter-list 1 out
  neighbor 10.23.1.3 filter-list 1 out

! --- Route map: deny RFC1918, raise local-pref for one block, set weight for rest ---
ip prefix-list FIRST-RFC1918 permit 192.168.0.0/16 le 32
ip as-path access-list 1 permit _65200$
ip prefix-list SECOND-CGNAT permit 100.64.0.0/10 le 32
!
route-map AS65200IN deny 10
 match ip address prefix-list FIRST-RFC1918
!
route-map AS65200IN permit 20
 match ip address prefix-list SECOND-CGNAT
 match as-path 1
 set local-preference 222
!
route-map AS65200IN permit 30
 match as-path 1
 set weight 23456
!
route-map AS65200IN permit 40
!
router bgp 65100
 neighbor 10.12.1.2 remote-as 65200
 address-family ipv4 unicast
  neighbor 10.12.1.2 activate
  neighbor 10.12.1.2 route-map AS65200IN in

! --- Conditionally match and discard routes by BGP community ---
ip community-list 100 permit 333:333
!
route-map COMMUNITY-CHECK deny 10
 match community 100
route-map COMMUNITY-CHECK permit 20
 set weight 111
!
router bgp 65100
 address-family ipv4 unicast
  neighbor 10.12.1.2 route-map COMMUNITY-CHECK in

! --- Set private BGP community (overwrite vs. additive) ---
ip prefix-list PREFIX10.23.1.0 seq 5 permit 10.23.1.0/24
ip prefix-list PREFIX10.3.3.0 seq 5 permit 10.3.3.0/24
!
route-map SET-COMMUNITY permit 10
 match ip address prefix-list PREFIX10.23.1.0
 set community 10:23
route-map SET-COMMUNITY permit 20
 match ip address prefix-list PREFIX10.3.3.0
 set community 3:0 3:3 10:10 additive
route-map SET-COMMUNITY permit 30
!
router bgp 65100
 address-family ipv4
  neighbor 10.12.1.2 route-map SET-COMMUNITY in
  neighbor 10.12.1.2 send-community both
```

## Design Baseline
A deviation from this table is a question ("is this intentional here?"), never automatically a finding — real networks deviate from best practice for good and bad reasons.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| No-transit enforcement at every multihomed edge: outbound filters advertise only locally originated routes | Without it, one SP's routes leak to the other and your links become free Internet transit | You actually are a transit AS, by contract | ENCOR 350-401 OCG |
| Route maps built on prefix lists, over raw distribute lists/ACLs | Composable match/set policy with sequenced edits instead of ACL surgery | Quick throwaway lab filters | ENCOR 350-401 OCG |
| Community scheme documented and applied consistently | Communities are the policy language across AS boundaries — undocumented tags are landmines | Simple single-homed edges with no policy to tag | ENCOR 350-401 OCG |
| Policy changes applied via route refresh/soft reconfiguration, not hard session resets | A hard reset drops the session — and traffic — with it | Emergency recovery from a bad policy push | ENCOR 350-401 OCG |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show bgp ipv4 unicast \| begin Network` | Full BGP table — Network, Next Hop, Metric, LocPrf, Weight, Path columns to confirm filtering/manipulation took effect |
| `show bgp ipv4 unicast <network>` | All available paths for one prefix with full attribute detail |
| `show bgp ipv4 unicast <network> best-path-reason` | Explicitly states why each path is/isn't the best (e.g. "Lower weight", "Longer AS path", "Overall best path") |
| `show bgp ipv4 unicast <network> bestpath` | Shows only the selected best path, skipping the others |
| `show bgp ipv4 unicast neighbors <ip> advertised-routes` | Confirms exactly what AS_Path ACL / route map / community filtering allowed outbound |
| `show bgp ipv4 unicast community <community>` | Lists only routes tagged with a specific BGP community |
| `show ip bgp <network>` | Full path attribute detail including Community, Origin, metric, localpref for one prefix |
| `show tcp brief` | Confirms underlying BGP TCP session state before troubleshooting policy issues |
| `clear bgp ipv4 unicast <ip\|*> soft [in\|out]` | Soft-resets a peer (or all peers) to re-apply a changed route map/filter without tearing down the session |

## Intent Questions
- What is the routing policy in words: which exit should each prefix prefer, and what do we advertise to whom?
- Is this AS ever supposed to be transit — and which filters enforce that answer?
- What is each community value supposed to mean, and who sets and reads it?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. Confirm the BGP session is Established (`show bgp ipv4 unicast summary`)
   before chasing a routing-policy problem — policy never applies to a
   down session.
2. If routes that should be filtered are still present: confirm the
   neighbor isn't mixing a distribute list and a prefix list in the same
   direction (not allowed) and that the filter is applied with the
   correct `in`/`out` keyword.
3. If a route map isn't behaving as expected: read it sequence-by-sequence
   — conditional match criteria first, then processing action, then
   optional action; remember a `deny` inside the *matched ACL itself* just
   excludes that prefix from the sequence, it does not deny the route map
   sequence.
4. If all routes are unexpectedly dropped: check for a missing catch-all
   `permit` sequence with no match statement — implicit deny applies to
   anything not explicitly permitted.
5. If AS_Path filtering isn't matching as expected: verify the regex
   pattern against Table 12-5 patterns (`^$` = locally originated,
   `^200_` = from neighbor AS 200, `_200$` = originated in AS 200, `_200_`
   = transits AS 200) — a missing `^` or `$` anchor is the most common
   mistake.
6. If communities aren't showing up on a peer: confirm `neighbor <ip>
   send-community [standard|extended|both]` is configured — IOS XE does
   not send communities by default.
7. If `set community` wiped out communities you expected to keep: check
   whether `additive` was omitted — without it, `set community` overwrites
   the entire community attribute.
8. For best-path selection disputes: run `show bgp afi safi <network>
   best-path-reason` first instead of manually walking the 13-step
   algorithm — it states the deciding factor directly.
9. If two paths from different ASNs aren't comparing the way MED
   "should" suggest: confirm both paths are actually from the same AS —
   MED is only a meaningful differentiator between paths from the same AS.
10. If the enterprise appears to be carrying transit traffic between two
    SPs or two branches: check outbound route filtering at the affected
    edge/branch router — confirm only locally-originated (LAN-facing)
    prefixes are advertised toward the WAN/SP, not routes learned from
    the WAN/SP itself.

## Common Pitfalls
- Multihoming to two SPs without outbound filtering — accidentally
  becomes a transit AS, saturating peering links with traffic that has
  nothing to do with the organization.
- Forgetting `summary-only`-equivalent discipline at branch routers in
  multi-WAN designs — without outbound filtering, a branch router can
  transit traffic between two other branches during a link failure,
  causing nondeterministic, asymmetric routing.
- Confusing extended ACL BGP matching with normal packet-filtering ACL
  matching — for BGP, the source fields match the network and the
  destination fields match the network mask, not a source/destination IP
  pair.
- Forgetting `neighbor <ip> send-community` — communities set in a route
  map silently never leave the local router without this.
- Using `set community` without `additive` when the intent was to add a
  community alongside existing ones — overwrites instead.
- Assuming weight or local preference will influence eBGP peers — weight
  is purely local to the router, and local preference is never advertised
  outside the AS, so neither affects an external peer's incoming decision.
- Comparing MED between paths from different ASNs and expecting it to be
  a deciding factor — it's only meaningful within paths from the same AS.
- Forgetting that a neighbor cannot have both a distribute list and a
  prefix list applied in the same direction at the same time.
