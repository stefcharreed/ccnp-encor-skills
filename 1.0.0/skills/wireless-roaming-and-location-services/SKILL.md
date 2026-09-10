---
name: ccnp-wireless-roaming-and-location-services
description: >
  Use this skill when troubleshooting or configuring wireless roaming and location services on IOS-XE.
  Invoke when the user asks about: wireless roaming, client mobility, roaming between autonomous APs,
  reassociation request, association request, intracontroller roaming, intercontroller roaming,
  Layer 2 roam, local-to-local roam, Layer 3 roam, local-to-foreign roam, anchor controller,
  foreign controller, guest anchor, static anchor WLAN, mobility group, mobility group name,
  mobility list, mobility domain, 24 controllers per mobility group, 72 mobility list entries,
  CCKM, Cisco Centralized Key Management, key caching, 802.11r, fast BSS transition, fast roaming,
  CCX, split-MAC roaming, CAPWAP tunnel between controllers, client database, client association table,
  roam time 10 ms, roam time 20 ms, DHCP renew on roam, 802.1x reauthentication on roam,
  sticky client, locating devices in a wireless network, received signal strength, RSS, RSSI location,
  trilateration with three APs, RF fingerprinting, RF calibration model, real-time location services,
  RTLS, Cisco Spaces, Cisco CMX, Connected Mobile Experiences, Mobility Services Engine, MSE,
  Cisco DNA Center location, Cisco Prime Infrastructure, floor map location dots, probe request tracking,
  RFID tag tracking, rogue device location, CleanAir interference location.
---

## Purpose
This topic controls how a wireless client keeps its connection (and its IP address) while moving between APs and controllers, and how the same AP/controller infrastructure is reused to compute where that client physically is — the two things that decide whether voice/time-critical traffic survives movement and whether asset tracking is possible at all.

## Key Concepts
- Roaming is a client-side decision. The client continuously evaluates its connection quality, and when the signal degrades it scans channels, sends Probe Requests to find candidate APs, picks one, and reassociates. The AP, the candidate AP, and the WLC do not make the roam decision (ENCOR OCG, Ch. 19, pp. 574–575; quiz Q2 answer A, p. 576).
- What the client leaves and joins is a **BSS**, not the SSID/ESS/DS (quiz Q1 answer B, p. 576). A client must associate and authenticate with an AP before it can use that AP's basic service set.
- **Association Request vs. Reassociation Request:** Association Requests form a *new* association; Reassociation Requests are used to roam from one AP to another, preserving the client's original association status (p. 575, NOTE).
- **Autonomous AP roaming:** each autonomous AP keeps its own association table. After the roam, both APs update their client lists; if the old AP still has buffered frames for the client, it forwards them to the new AP over the *wired* infrastructure, simply because that's where the client's MAC address now lives (pp. 575–576, Figures 19-1 and 19-2).
- Cells are deliberately overlapped so a moving client always has a better candidate AP; the exact roam point depends on the client's own roaming algorithm (p. 576, Figure 19-3).
- **Intracontroller roaming:** both APs are joined to the same WLC over CAPWAP; because of split-MAC, the *controller* (not the APs) handles the roam. It just updates its client database so it knows which CAPWAP tunnel reaches the client. Takes **less than 10 ms**. The client cannot tell the difference — it has no knowledge of the CAPWAP tunnels at all (pp. 577–578).
- The controller's client database holds more than the figure shows: AP, associated clients, WLAN, plus client MAC and IP addresses, QoS parameters, and other information (p. 577).
- Two other processes can be triggered alongside reassociation and must be streamlined for roaming to feel seamless: **DHCP** (client may renew or request a new address — while waiting it is essentially cut off from the network) and **client authentication** (802.1X to a RADIUS server plus key generation/exchange, the single biggest time cost) (pp. 578–579).
- Three Cisco techniques minimize key-exchange time during roams (p. 579):
  - **CCKM (Cisco Centralized Key Management)** — one controller maintains the client/key database on behalf of its APs and hands keys to other controllers and their APs during roams. Requires **CCX** (Cisco Compatible Extensions) support on the client.
  - **Key caching** — the *client* keeps a list of keys from prior AP associations and presents them as it roams. The destination AP must be in that list, which is limited to **eight AP/key entries**.
  - **802.11r** — the amendment for fast roaming / fast BSS transition: the client caches a portion of the authentication server's key and presents it to future APs, and can also maintain its QoS parameters across the roam.
  - All three require client-side help: a supplicant/driver compatible with fast roaming that can cache the credential pieces.
- **Intercontroller roaming** happens when the two APs are joined to different WLCs; the two controllers must coordinate the move.
- **Layer 2 roam (local-to-local):** the WLAN interfaces on both controllers map to the **same VLAN ID/subnet**, so nothing special is needed — the client keeps its IP address and the roam is fast, **usually less than 20 ms** (pp. 580–581). No tunnel is used to carry client data between the controllers for a Layer 2 roam (quiz Q6 answer D, p. 576).
- **Layer 3 roam (local-to-foreign):** the controllers compare the VLAN IDs assigned to their WLAN interfaces. Same VLAN ID → Layer 2 roam. Different VLAN IDs → they build an extra **CAPWAP tunnel between the controllers** so traffic is carried as if the client were still on its original controller and subnet, letting the client keep its IP address (pp. 581–583).
- **Anchor controller** = the client's original controller (where it first associated). **Foreign controller** = the controller the client roamed to. The client stays anchored to its original controller and subnet no matter where it roams or how many controllers it passes through (pp. 582–583).
- Anchor/foreign roles are normally determined automatically, but you can configure a **static anchor** for a WLAN — the classic use case is forcing guest users onto a specific controller behind a firewall or in a protected DMZ rather than letting them anchor to whichever controller they hit first (p. 583).
- Clients usually cannot detect that they changed subnets — they only see the AP roam. Only clients that aggressively contact DHCP after every roam would keep working without the anchor tunnel, and DHCP is exactly the time-consuming process you want to avoid (p. 581).
- **Mobility groups** organize controllers so intercontroller roaming scales. Same mobility group → clients roam quickly, both Layer 2 and Layer 3 roaming supported, along with CCKM, key caching, and 802.11r credential caching. **A mobility group can contain up to 24 controllers** (p. 584).
- Different mobility groups → clients can **still roam**, but the roam is inefficient: credentials are not cached and shared, so the client must do a **full authentication** during the roam (p. 584; quiz Q8 answer C, p. 576).
- **Mobility list / mobility domain:** each controller keeps a mobility list containing its own MAC address and the MAC addresses of other controllers, each with a mobility group name. The mobility list defines the **mobility domain** — a controller knows and trusts only the controllers in its list. If two controllers are not in each other's lists they are unknown to each other and clients **cannot roam** between them at all; they must associate and authenticate from scratch. **A mobility list can contain up to 72 controller entries** (p. 584, Figure 19-10).
- **Locating devices:** the crudest resolution is "which AP is the client joined to," which is often not granular enough (one AP can cover a large area) and is distorted by sticky clients that stay associated to a distant AP even when a closer, stronger AP exists (p. 585).
- **RSS (received signal strength)** is the location input (quiz Q9 answer C, p. 576). Free space path loss attenuates an RF signal exponentially as a function of frequency and distance, so distance can be computed from RSS. One AP with an omnidirectional antenna only narrows the client to a circle of fixed radius; **three or more APs** correlated to find where the circles intersect gives an actual position (p. 585, Figure 19-11 — example values -60 dBm from one AP; -58, -40, -69 dBm from three APs).
- **RTLS** is not inherent to the infrastructure. Split-MAC means APs touch the clients at the real-time layer and WLCs forward data; the WLCs must keep a management platform (Cisco DNA Center, Cisco Prime Infrastructure) informed as clients probe, join, and leave, and pass along wireless statistics such as each client's RSS. The actual location is computed on a **separate location server platform** — Cisco Spaces, Cisco MSE (Mobility Services Engine), or Cisco CMX (Connected Mobile Experiences) (pp. 585–586).
- **RF fingerprinting** is the Cisco fix for the fact that real buildings are not free space: walls, doors, windows, furniture, cubicles, and shelving attenuate signals. Each mapped area gets an RF calibration template that resembles the real attenuation. Calibration is either measured manually by walking the area with a device, or applied from models — sample models include **cubes and walled offices, drywalled offices, indoor high ceilings, and outdoor open spaces** (p. 586).
- **Map display:** square icons are manually placed AP locations; small colored dots are device locations placed dynamically at regular intervals. **Green dots = devices successfully associated with APs; red dots = devices not associated but actively sending Probe Requests.** In the sample figure, selecting one device drew lines to the **seven APs** that had recorded a current RSS measurement for it (pp. 586–587, Figure 19-12).
- Location works on devices that never associate, because clients send **Probe Requests on every channel and band they support**, all sourced from the same client MAC, and neighboring APs hear them on their own channels. That covers passing smartphones with Wi-Fi enabled and **RFID tags** — some tags actively join the network, others just "wake up" periodically to send Probe Requests or multicast frames to announce their presence (p. 587).
- Location also covers **rogue devices** (they probe, so they can be discovered and located) and **non-802.11 interference sources** (cordless phones, wireless video cameras, other transmitters). Cisco APs detect these with dedicated spectrum analysis and **CleanAir**, report received signal strength on a channel, and the location server computes a probable location and plots it on the map (p. 587).

## Procedure

**A. Roam between two autonomous APs (pp. 574–576)**
1. Client is associated/authenticated to AP-1 and appears in AP-1's association table.
2. Client continuously evaluates its connection quality; signal from AP-1 degrades.
3. Client actively scans channels and sends Probe Requests to discover candidate APs.
4. Client selects a candidate and sends a **Reassociation Request** to AP-2 (preserving original association status).
5. Both APs update their association tables — client leaves AP-1's list, appears in AP-2's.
6. Any frames AP-1 had buffered for the client are forwarded to AP-2 across the wired infrastructure.

**B. Intracontroller roam — both APs on the same WLC (pp. 577–578)**
1. Client associated to AP-1; WLC-1's client database maps Client-1 → AP-1 → WLAN.
2. Client decides to roam and reassociates to AP-2 (same controller).
3. The controller — not the APs — handles the roam, because of split-MAC.
4. WLC updates its client association table so it knows which CAPWAP tunnel now reaches the client.
5. Completes in **less than 10 ms**; the client sees an ordinary roam.
6. Optionally, DHCP renew/request and 802.1X client authentication follow — both must be streamlined or the roam stops being seamless.

**C. Layer 2 intercontroller roam — different WLCs, same VLAN (pp. 579–581)**
1. Client on AP-1/WLC-1 holds an IP address from the VLAN/subnet bound to that WLAN interface (e.g., VLAN 100, 192.168.100.0/24).
2. Client reassociates to AP-2, which is joined to WLC-2.
3. The two controllers compare the VLAN IDs assigned to their respective WLAN interfaces.
4. VLAN IDs match → nothing special happens; the client keeps its existing IP address.
5. Client entry moves to WLC-2's database; roam completes **usually in less than 20 ms**. No inter-controller data tunnel is required.

**D. Layer 3 intercontroller roam — different WLCs, different VLANs (pp. 581–583)**
1. Client on AP-1/WLC-1 with an address from VLAN 100 / 192.168.100.0/24 (e.g., 192.168.100.199).
2. Client reassociates to AP-2 on WLC-2, whose WLAN interface is VLAN 200 / 192.168.200.0/24.
3. Controllers compare VLAN IDs — they differ, so they arrange a Layer 3 (local-to-foreign) roam.
4. An extra **CAPWAP tunnel is built between the two controllers**.
5. WLC-1 becomes the **anchor controller** (its database shows the client as reachable via WLC-2, marked Mobile, still on WLAN Staff / VLAN 100); WLC-2 becomes the **foreign controller** and shows Client-1 on AP-2.
6. Client traffic is tunneled to/from the anchor as if the client were still on its original controller and subnet, so the client keeps its IP address — no DHCP round trip.
7. The tether persists regardless of how many further controllers the client roams through.

**E. Locating a wireless device (pp. 585–587)**
1. Client (or unassociated device / RFID tag) transmits — Probe Requests are sent on every channel and band the device supports, all from the same MAC.
2. Multiple neighboring APs hear the transmission on their own channels and record a received signal strength (RSS) value.
3. APs report client activity and RSS to their WLCs.
4. WLCs keep the management platform (Cisco DNA Center / Prime Infrastructure) informed as clients probe, join, and leave, passing along the RSS statistics.
5. A separate location server (Cisco Spaces, MSE, or CMX) correlates RSS from **three or more APs**, converting each to a distance and finding where the circles intersect.
6. The result is adjusted by the map's **RF fingerprinting** calibration (walked measurements or a construction model) to account for real-world attenuation.
7. The computed position is plotted on a floor map at regular intervals — green dot = associated device, red dot = probing-but-unassociated device.

## Reference Tables

### Roam types and cost (pp. 577–584)

| Roam type | Scope | Client IP address | Time / efficiency | Source |
|---|---|---|---|---|
| Autonomous AP roam | Two standalone APs, no controller | Unchanged (same subnet in the example) | Not stated; old AP forwards buffered frames over the wire | ENCOR 350-401 OCG, Ch. 19, pp. 575–576 |
| Intracontroller roam | Two APs on the same WLC | Unchanged | **< 10 ms** — fastest of the three | ENCOR 350-401 OCG, Ch. 19, p. 578 |
| Layer 2 intercontroller roam (local-to-local) | Two WLCs, same VLAN ID on the WLAN interfaces | Kept | **Usually < 20 ms**; no inter-WLC data tunnel | ENCOR 350-401 OCG, Ch. 19, pp. 580–581 |
| Layer 3 intercontroller roam (local-to-foreign) | Two WLCs, different VLAN IDs | Kept, via anchor tunnel | Extra CAPWAP tunnel anchor↔foreign; slower than Layer 2 | ENCOR 350-401 OCG, Ch. 19, pp. 581–583 |

### Fast-roaming key techniques (p. 579)

| Technique | Who holds the keys | Requirement / limit |
|---|---|---|
| CCKM (Cisco Centralized Key Management) | One controller keeps a client/key database on behalf of its APs and provides keys to other controllers and their APs | Requires Cisco Compatible Extensions (CCX) support on the client |
| Key caching | The client keeps a list of keys from prior AP associations and presents them when roaming | Destination AP must be in the list; list limited to **eight AP/key entries** |
| 802.11r (fast BSS transition) | Client caches a portion of the authentication server's key and presents it to future APs | Client supplicant/driver must support it; also preserves the client's QoS parameters |

### Mobility group / mobility list limits and behavior (p. 584)

| Item | Value / behavior |
|---|---|
| Max controllers in one mobility group | **24** |
| Max entries in a controller's mobility list | **72** |
| Same mobility group | Fast roaming; Layer 2 and Layer 3 roams supported; CCKM, key caching, and 802.11r credential caching all work |
| Different mobility groups, but present in each other's mobility lists | Roaming still possible but inefficient — credentials are not cached/shared, so the client does a full authentication |
| Not in each other's mobility list | Controllers are unknown to each other; **no roaming** — client associates and authenticates from scratch |
| Mobility list contents | Controller's own MAC plus other controllers' MACs, each tagged with a mobility group name; collectively defines the **mobility domain** |

### Chapter Key Topics (Table 19-2, pp. 587–588)

| Key Topic Element | Description | Page |
|---|---|---|
| Figure 19-2 | After Roaming Between Autonomous APs | 576 |
| Figure 19-5 | Cisco Wireless Network After an Intracontroller Roam | 578 |
| Figure 19-7 | After an Intercontroller Roam | 581 |
| Figure 19-9 | After a Layer 3 Intercontroller Roam | 583 |
| Figure 19-10 | Mobility Group Hierarchy | 584 |
| Figure 19-11 | Locating a Wireless Device with One AP (left) and Three APs (right) | 585 |
| Figure 19-12 | A Sample Map Showing Real-Time Location Data for Tracked Devices | 586 |

## Config Patterns

```ios-xe
! ---- Mobility group / mobility list (C9800, global config) ----
wireless mobility group name CAMPUS-MG
wireless mobility group member mac-address 00a1.b2c3.d4e5 ip 10.10.10.11 group CAMPUS-MG
wireless mobility group member mac-address 00a1.b2c3.d4f6 ip 10.10.20.11 group CAMPUS-MG

! ---- Static (guest) anchor on a WLAN policy profile ----
wireless profile policy GUEST-POLICY
 mobility anchor 10.99.99.10 priority 1

! ---- Fast-roaming key techniques on the WLAN ----
wlan STAFF 1 Staff
 security wpa akm ft dot1x            ! 802.11r / fast BSS transition
 security ft
 security dot1x authentication-list AAA-DOT1X

! ---- Verify ----
show wireless mobility summary
show wireless client mac-address aaaa.bbbb.cccc detail
```

**Honesty note:** Chapter 19 contains **no CLI and no GUI navigation at all** — it is a purely conceptual chapter (its only figures are topology diagrams, a mobility-group hierarchy, an RSS trilateration illustration, and a Cisco Spaces floor map). Everything in the block above is **book-derived context, not printed in this chapter, and NOT gear-validated** by me. Treat the exact command syntax as unverified and confirm it against the C9800 configuration guide for your IOS-XE release before use. Any shared secret (RADIUS key, mobility group tunnel key) must come from `<from-secrets-not-repo>` — never inline it in a repo or a runbook.

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Overlap AP cells across the coverage area | A roaming client only moves when a better candidate AP exists; without overlap there is nothing to roam to | Intentional coverage islands (warehouse aisles, outdoor spot coverage) where no roaming is expected | ENCOR 350-401 OCG, Ch. 19, p. 576 |
| Keep both APs of a roam on one controller where you can | Intracontroller roams are the simplest and fastest — **< 10 ms**, just a client-database update | Network has outgrown one controller; AP counts, HA design, or site separation force multiple WLCs | ENCOR 350-401 OCG, Ch. 19, p. 578 |
| Bind the same WLAN to the same VLAN ID/subnet on controllers clients roam between | Matching VLAN IDs produce a Layer 2 roam: client keeps its IP, roam is **usually < 20 ms**, no inter-WLC tunnel | Scalability — a large WLAN deliberately broken into per-controller subnets; then accept Layer 3 roaming | ENCOR 350-401 OCG, Ch. 19, pp. 580–581 |
| Put controllers clients roam between in the **same mobility group** | Same group gets fast roaming plus CCKM, key caching, and 802.11r credential caching | Administrative/security separation between buildings or tenants; accept full re-auth on the cross-group roam | ENCOR 350-401 OCG, Ch. 19, p. 584 |
| Make sure every controller appears in the others' **mobility lists** | The mobility list defines the mobility domain; controllers not in each other's list are unknown and roaming is impossible | Deliberate isolation of a controller (e.g., a standalone DMZ/guest WLC that should never accept roams) | ENCOR 350-401 OCG, Ch. 19, p. 584 |
| Stay within **24 controllers per mobility group** and **72 entries per mobility list** | These are the printed hard limits on group and list size | None stated in the chapter — these are ceilings, not preferences | ENCOR 350-401 OCG, Ch. 19, p. 584 |
| Enable a fast-roaming method (CCKM, key caching, or 802.11r) for time-critical WLANs | Authentication (RADIUS dialog + key generation) is the biggest single cost in a roam; voice clients cannot absorb it | Client fleet lacks a compatible supplicant/driver — every method needs client-side support; CCKM additionally needs CCX | ENCOR 350-401 OCG, Ch. 19, p. 579 |
| Avoid designs that force DHCP renew/request on every roam | A client renewing its address is effectively cut off from the network until the DHCP server answers | A client policy you do not control aggressively renews after each roam; then Layer 2 design matters even more | ENCOR 350-401 OCG, Ch. 19, pp. 578–579, 581 |
| Configure a **static anchor** for guest WLANs onto a controller behind a firewall / in a protected environment | Otherwise the guest's first controller becomes its anchor — guests should not be allowed to anchor to just any controller | No guest WLAN, or the guest path is segregated by other means | ENCOR 350-401 OCG, Ch. 19, p. 583 |
| Require RSS from **three or more APs** before trusting a computed location | A single AP with an omnidirectional antenna only places the client on a circle of fixed radius | Coarse "which AP is it on" resolution is genuinely good enough for the use case | ENCOR 350-401 OCG, Ch. 19, p. 585 |
| Apply **RF fingerprinting** calibration (walked measurements or a construction model) to every location map | Free-space assumptions break indoors — walls, doors, windows, furniture, cubicles and shelving all attenuate | Outdoor open space, where the free-space assumption is closer to true (still one of the offered models) | ENCOR 350-401 OCG, Ch. 19, p. 586 |
| Deploy a dedicated location server (Cisco Spaces / MSE / CMX) alongside the management platform | RTLS is not inherent to the infrastructure; WLCs feed statistics to DNA Center/Prime, but location is computed on a separate platform | No location use case at all — then don't build the location tier | ENCOR 350-401 OCG, Ch. 19, pp. 585–586 |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show wireless mobility summary` | The local controller's mobility group name and MAC, plus every peer in the mobility list with its IP, MAC, group name, and control/data link status — the fastest way to confirm two WLCs actually know each other and whether they share a group |
| `show wireless mobility peer ip <peer-ip>` | Per-peer tunnel state and statistics for a specific mobility peer when the summary shows a peer down |
| `show wireless mobility statistics` | Counts of mobility control messages, handoff requests, and failures — rising failures point at mobility-list/group misconfiguration rather than RF |
| `show wireless client summary` | Which AP and WLAN each client is currently on, and its state; run repeatedly to see whether a client is actually roaming or is sticky on a distant AP |
| `show wireless client mac-address <mac> detail` | Current AP, WLAN/policy profile, VLAN, IP address, RSSI/SNR, security/AKM in use (shows whether FT/802.11r or CCKM is negotiated), and mobility role (local / anchor / foreign / export) |
| `show wireless client mac-address <mac> mobility history` | Per-client roam history — which APs/controllers it moved through and when; the first place to look for a roaming complaint |
| `show wireless client mac-address <mac> stats` | Per-client counters including retries and data rates, to separate "roamed badly" from "bad RF on the current AP" |
| `show wireless profile policy detailed <policy-name>` | Whether a mobility anchor is configured on that policy profile and which anchor IP/priority — confirms guest anchoring intent |
| `show ap summary` / `show ap dot11 5ghz summary` | AP-to-controller joins, channel and tx power — needed to reason about cell overlap and whether candidate APs exist where the client is losing signal |
| `show ap name <ap> neighbor summary` | Which APs hear which — sanity check that three or more APs can plausibly hear a client in a given area for location |
| `show wireless stats client detail` | Aggregate client association/roaming/auth failure counters across the controller, for spotting a fleet-wide roaming regression |

**Honesty note:** these are real C9800 IOS-XE show commands but they are **not printed in Chapter 19** (the chapter shows no CLI); they are book-derived and not gear-validated by me. Confirm exact forms against your release's command reference.

## Intent Questions
1. What kind of roam is this network *supposed* to be doing between these two APs — intracontroller, Layer 2 intercontroller, or Layer 3 (anchor/foreign)? Which VLAN ID is the WLAN bound to on each controller, and was that deliberate?
2. Which controllers are meant to be in the same mobility group, and which are deliberately separate? Is every controller a client can roam to actually present in the others' mobility lists?
3. What is riding on this WLAN — voice/time-critical traffic that needs fast roaming (CCKM / key caching / 802.11r) with a client fleet that supports it, or best-effort data where a full re-auth per roam is tolerable?
4. Is location a requirement here, and at what resolution — "which AP" is enough, or does the use case (asset tracking, wayfinding, rogue/interference hunting) need three-AP RSS plus an RF-fingerprinted map and a location server?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command.
1. **Layer 1 / RF:** is there a candidate AP to roam to where the client is failing? Check cell overlap, AP channel/power, and the client's RSSI/SNR on its current AP. A client that never roams may simply have nothing better to hear — or be sticky, staying on a far AP while a closer one is available (p. 585).
2. **Layer 1 / interference:** if the drop is not roam-shaped, look for non-802.11 interference — spectrum analysis / CleanAir can detect and locate transmitters that the 802.11 side never sees (p. 587).
3. **Layer 2 / association:** is the client sending Reassociation Requests (roam) or Association Requests (starting over)? Starting over means the roaming path is broken, not slow (p. 575). Confirm the client actually appears under the new AP in the controller's client database.
4. **Layer 2 / VLAN:** compare the VLAN ID bound to the WLAN interface on each controller. Same ID → expect a Layer 2 roam with the IP preserved; different IDs → expect anchor/foreign tunneling. A VLAN mismatch you did not intend is the classic cause of a surprise Layer 3 roam (p. 581).
5. **Layer 3 / mobility plane:** verify the anchor↔foreign CAPWAP tunnel is up between the controllers, and that the client shows the roles you expect (original controller = anchor, roamed-to = foreign). A dead inter-controller tunnel breaks a Layer 3 roam even though both controllers are individually healthy (pp. 582–583).
6. **Layer 3 / client addressing:** did the client keep its IP or go to DHCP? A client that renews on every roam is cut off until the DHCP server answers — that alone can explain "voice drops when I walk down the hall" (pp. 578–579).
7. **Config — mobility list:** are both controllers in each other's mobility list with the correct MAC, IP, and group name? Missing entries mean the controllers are unknown to each other and roaming is impossible, not merely slow (p. 584).
8. **Config — mobility group:** are they in the *same* group? Cross-group roams work but skip credential caching, forcing a full authentication — that looks like a slow roam, not a failed one (p. 584).
9. **Config — scale limits:** has the group exceeded 24 controllers or the mobility list exceeded 72 entries? (p. 584).
10. **Config — fast roaming:** is CCKM / key caching / 802.11r actually negotiated for this client, or only configured? Check the AKM in the client detail. Remember key caching only holds **eight** AP/key entries, so a client crossing many APs can fall out of cache (p. 579).
11. **Config — anchor policy:** for guest WLANs, is the static anchor pointed at the intended protected controller, or is the client anchoring to whichever controller it hit first? (p. 583).
12. **Client software:** every fast-roaming method needs a compatible supplicant/driver, and CCKM additionally needs CCX. A single non-compliant client model behaving badly while others roam fine points here, not at the infrastructure (p. 579).
13. **Location path (if the symptom is location, not connectivity):** confirm the WLC is feeding DNA Center / Prime, that a separate location server (Spaces / MSE / CMX) is receiving data, that three or more APs report RSS for the device, and that the floor map's AP positions were entered correctly and RF-fingerprint calibration was applied (pp. 585–587).
14. **Software bugs:** only after the above — if intent, RF, VLANs, mobility list/group, and client software all check out and the roam still fails or the mobility tunnel flaps, treat it as a controller/AP software defect and get the release notes and a TAC case involved.

## Common Pitfalls
- **Thinking the AP or the controller decides the roam.** It's the client (quiz Q2 answer A). You can influence roaming with RF design and fast-roaming features, but you cannot order a client to move.
- **Thinking the client leaves the SSID/ESS.** It leaves and joins a **BSS** (quiz Q1 answer B). One SSID spans many BSSs.
- **Assuming a Layer 2 roam needs a tunnel between controllers.** It doesn't — quiz Q6's correct answer is "None of these answers are correct" (answer D). The extra CAPWAP tunnel is a **Layer 3** roam artifact only.
- **Getting anchor and foreign backwards.** The *original* controller is the anchor; the controller the client roamed *to* is the foreign controller (quiz Q7 answer D). "Anchor" sounds like a destination but it's the origin.
- **Assuming multiple APs on one controller means intercontroller roaming.** Ten APs on one WLC roam *intra*controller (quiz Q3 answer C) — the controller count, not the AP count, is what matters.
- **Assuming all roam types cost the same.** They don't: intracontroller (< 10 ms) is fastest, then Layer 2 intercontroller (< 20 ms), then Layer 3 (quiz Q4 answer C).
- **Assuming different mobility groups means no roaming.** Roaming still happens if the controllers are in each other's mobility lists — it's just inefficient because credentials aren't shared and the client fully re-authenticates (quiz Q8 answer C). *No* roaming is the symptom of a missing **mobility list** entry, which is a different failure.
- **Confusing mobility group with mobility domain.** The group is a named set (max 24 controllers); the mobility list (max 72 entries) is what defines the domain and determines who a controller trusts.
- **Confusing CCKM with something else that sounds close.** CCKM is Cisco Centralized Key Management (quiz Q5 answer C) — not CCX (the client-side compatible-extensions program CCKM depends on), not PGP, not EoIP.
- **Forgetting fast roaming is a two-sided contract.** Configure CCKM/key caching/802.11r all you like; without a compatible client supplicant/driver nothing is cached.
- **Forgetting the eight-entry key cache limit.** A client roaming across a large floor can revisit an AP that has aged out of its own cache and pay a full authentication anyway.
- **Assuming location is built into the WLC.** RTLS is *not* inherent to the infrastructure — the position is computed on a separate location server (Spaces/MSE/CMX) fed via DNA Center/Prime.
- **Trusting a single-AP location.** One AP puts the device somewhere on a circle. Correlating three or more APs is what produces a point.
- **Applying free-space math indoors.** Walls, doors, windows, furniture, cubicles and shelving attenuate; without RF fingerprinting calibration the computed location will be wrong in ways that look arbitrary.
- **Assuming only associated clients can be located.** Devices that merely send Probe Requests — passing smartphones, passive RFID tags, rogues — are locatable too; on the map they're the **red** dots, while green dots are associated devices.
