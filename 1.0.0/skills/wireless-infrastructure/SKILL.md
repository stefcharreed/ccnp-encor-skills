---
name: ccnp-wireless-infrastructure
description: >
  Use this skill when troubleshooting or configuring wireless infrastructure on IOS-XE —
  AP/WLC deployment topologies, AP-to-controller pairing, controller config segmentation,
  and antenna selection for coverage. Invoke when the user asks about: wireless deployment
  model, autonomous AP, lightweight AP, split-MAC architecture, wireless LAN controller,
  WLC, CAPWAP, CAPWAP tunnel, DTLS, centralized wireless deployment, cloud-based WLC,
  distributed wireless deployment, controller-less deployment, embedded wireless controller,
  EWC, AP modes, local mode, FlexConnect, monitor mode, sniffer mode, rogue detector,
  bridge mode, Flex+Bridge, SE-Connect, AP state machine, WLC discovery, WLC join,
  CAPWAP Discovery Request, DHCP option 43, CISCO-CAPWAP-CONTROLLER, primed controller,
  primary secondary tertiary controller, master controller, least-loaded controller,
  AP priority, AP keepalive, heartbeat, HA SSO, stateful switchover, AP groups,
  profiles and tags, site tag, RF tag, policy tag, AP profile, flex profile, WLAN profile,
  policy profile, default-site-tag, default-rf-tag, default-policy-tag, AireOS vs IOS XE,
  antenna, radiation pattern, polar plot, E plane, H plane, azimuth, elevation, antenna gain,
  dBi, beamwidth, polarization, omnidirectional antenna, dipole, integrated antenna,
  directional antenna, patch antenna, Yagi antenna, parabolic dish antenna, client density.
---

## Purpose
Defines how many APs and one or more WLCs are assembled into a working wireless
topology — where the controller sits, how each AP finds and binds to it, how that
AP's configuration is scoped, and which antenna shapes the resulting RF cell.

## Key Concepts

**Autonomous vs lightweight**
- An **autonomous AP** is self-contained: it offers one or more fully functional
  standalone BSSs and bridges SSIDs directly to wired VLANs at the access layer.
  It needs a **trunk** to the access switch (data VLANs + a dedicated management
  VLAN) and its own management IP.
- Scaling autonomous APs is painful: every data VLAN and the management VLAN must
  be trunked to every AP, STP becomes load-bearing, and client roaming is limited
  to the Layer 2 domain — the extent of a single VLAN.
- Autonomous APs do have one advantage: the **shortest data path**. Two clients on
  the same autonomous AP reach each other through the AP without going up into the
  wired network.
- A Cisco **lightweight AP** loses that self-sufficiency and must join a WLC. The
  cooperation is a **split-MAC architecture**: the AP handles the real-time 802.11
  processes, the WLC handles the management functions.
- AP and WLC are joined by a logical **pair of CAPWAP tunnels** (control + data)
  that extends through the wired infrastructure. Each AP has its own tunnel pair;
  the network scales by adding WLCs once a controller hits its max AP count.
- Because all VLANs/WLANs ride the same CAPWAP tunnel, a lightweight AP connects to
  an **access port** and needs only a **single IP address** to terminate the tunnel.
  The Layer 3 boundary for data VLANs lives at or near the WLC.
- Consequence: in a centralized deployment, two clients on the *same* AP still send
  traffic up the CAPWAP tunnel to the WLC and back down again — through **the AP and
  its controller**, not AP-only.

**Two WLC platforms** — the newer platform runs **IOS XE**; its predecessor ran
**AireOS**. From the AP's perspective both connect via CAPWAP tunnels; IOS XE offers
more scalability, performance, availability, and maintainability. The ENCOR blueprint
stays platform-agnostic, so exam scenarios may come from either.

**Deployment models** (see Reference Tables)
- **Centralized** — WLC in a data center or near the core; maximizes APs per
  controller; convenient single point to enforce security policy for all wireless users.
- **Cloud-based** — a centralized controller located in a public cloud or a private
  cloud in the enterprise DC. **Public cloud forces FlexConnect mode** (control tunnel
  only; data must be locally switched). Private cloud allows local or FlexConnect.
- **Distributed** — smaller WLCs placed at each site, lower in the hierarchy.
- **Controller-less** — an **EWC (embedded wireless controller)** is a regular AP that
  also runs WLC software; no discrete WLC exists. The hosting AP forms a CAPWAP tunnel
  with its own embedded WLC, as do the other APs at that location.
- Centralized, cloud, and distributed all use standalone WLCs → all are
  **controller-based** deployments.
- **RTT between an AP and its controller should be under 100 ms.** Beyond that, APs
  may decide the controller isn't responding fast enough, disconnect, and go find a
  more responsive one.

**Controller availability**
- After joining, an AP sends **keepalive/heartbeat** messages to the WLC over the wired
  network. Default: every **30 s**. An unanswered keepalive escalates to **four more at
  3-second intervals** — so a failure is detected in as little as **35 s** by default.
- Tunable: regular keepalive **1–30 s**, fast heartbeat **1–10 s** → minimum detection
  about **6 s**.
- Falling back to "next least-loaded controller" is *not* deterministic — 1000 orphaned
  APs all rediscover and stampede at once. Priming **primary/secondary/tertiary** is the
  deterministic approach.
- **HA SSO** pairs controllers into active + hot standby. APs only need to know the
  primary (active) controller. The active unit syncs CAPWAP tunnels, AP states, client
  states, configs, and image files to the standby, including the state of each client in
  RUN state — so failover is transparent to end users.
- Every controller has a max AP count by platform or license; a full controller **rejects**
  additional APs. **AP priority** (default **low**; settable low/medium/high/critical) lets
  an oversubscribed controller reject a lower-priority AP to make room for a higher one.

**Image handling** — you cannot choose the image a lightweight AP runs; **the WLC it joins
determines the release**. Download scenarios: version mismatch on join, a WLC code upgrade,
or a WLC failure that pushes APs elsewhere. If an AP might rehome between controllers, keep
both on the same release, or move it in a maintenance window. **Predownload** stages a new
image on APs while they keep running the old one, so a controller reboot doesn't stall on
image downloads.

**Segmenting configuration** — AP parameters fall into three categories: (1) AP-controller
CAPWAP relationship and FlexConnect behavior, per site; (2) RF operation per band;
(3) WLAN definitions and security policies.
- **AireOS**: mostly global config plus **AP groups**. Each AP belongs to exactly one group,
  granular control means duplicating changes across many groups, and group changes often
  force radio resets or AP reboots.
- **IOS XE**: object-oriented **profiles and tags**. Define site, RF, and policy profiles,
  then tag each AP to select which it uses. The three tags map as:
  - **Site tag** → **AP profile** (a.k.a. AP join profile, used in local client-serving mode)
    + **Flex profile** (used for FlexConnect). Covers CAPWAP timers, AP fallback, TCP MSS,
    rogue detection, ICap, QoS; and native VLAN, local auth, policy ACL, VLAN, DNS security.
  - **RF tag** → **RF profiles per band** (2.4 / 5 / 6 GHz), each tunable independently:
    data rates, MCS, RRM, coverage hole detection, TPC, DCA.
  - **Policy tag** → **WLAN profile** (SSID, band, Layer 2/Layer 3 security, AAA) +
    **policy profile** (VLAN, multicast, ACL, URL filters, QoS ingress/egress policies).
- Defaults: **default-site-tag** → default-ap-profile + default-flex-profile;
  **default-rf-tag** → the controller's global RF config; **default-policy-tag** → maps to
  nothing, because there's no default WLAN/SSID config for any network.
- Tags aren't limited to groups — you can map profiles and tags to a single AP.

**Antennas**
- **Client density** is devices per AP. More active clients on a channel = less airtime each.
  A good design covers where coverage is needed *and* distributes users across enough APs.
  A more constrained antenna pattern is one way to limit the clients an AP serves.
- A **radiation pattern** plots relative signal strength around an antenna. Slice the 3D
  pattern with two orthogonal planes: the **H plane** (XY, horizontal/azimuth, top-down view)
  and the **E plane** (XZ, elevation, side view). Each outline is drawn on a **polar plot**.
- Polar plot rings are **relative to the maximum at the outer ring**, not absolute dB values.
  **Gain is not shown on E/H plots** — only the manufacturer's spec sheet has it.
- **Gain** measures how effectively an antenna focuses RF energy in a direction. Antennas are
  **passive** — they add gain by *shaping* energy, not amplifying it. Isotropic compared to
  itself = 10log10(1) = **0 dBi**.
- **Beamwidth** is the angle between the two points where the pattern falls **3 dB** below its
  strongest point, listed in degrees for both H and E planes. Low gain ↔ large beamwidth;
  high gain ↔ small beamwidth.
- **Polarization** is the orientation of the electrical field wave relative to the horizon.
  Cisco antennas are vertically polarized when mounted per recommendation. Polarization alone
  doesn't matter — but TX and RX polarization **must match**, or the received signal is
  severely degraded. Knocking an antenna sideways changes both the pattern and the polarization.
- **Omnidirectional** antennas are thin cylinders radiating equally away from the cylinder but
  not along its length → donut pattern, wider in the H plane than the E plane, relatively low
  gain, good for broad coverage of a room/floor with the antenna centered.
- **Directional** antennas focus energy in one general direction, giving higher gain — long
  hallways, warehouse aisles, outdoor areas, building-to-building, or ceiling-mounted pointing
  down to shrink an AP's cell.

## Procedure

**AP state machine (lightweight AP, boot to serving clients):**
1. **AP boots** — powers up, boots a small IOS image, and gets an IP address via DHCP or
   static config so it can communicate.
2. **WLC discovery** — works through the discovery steps to build a list of candidate WLCs.
3. **CAPWAP tunnel** — attempts to build a CAPWAP tunnel providing a secure **DTLS** channel
   for control messages; AP and WLC authenticate each other by exchanging digital certificates.
4. **WLC join** — selects a WLC from the candidates, sends a **CAPWAP Join Request**; the WLC
   replies with a **CAPWAP Join Response**.
5. **Download image** — the WLC states its software release. On mismatch the AP downloads the
   matching image, reboots, and **returns to step 1**. Matching releases skip the download.
6. **Download config** — the AP pulls RF, SSID, security, and QoS parameters from the WLC.
7. **Run state** — the WLC places the fully initialized AP in "run"; AP and WLC begin providing
   a BSS and accepting wireless clients.
8. **Reset** — if the WLC resets the AP, it tears down client associations and CAPWAP tunnels,
   reboots, and starts the whole state machine again.

**WLC discovery sequence** (goal: build a list of *live candidate* controllers; an AP sends a
unicast CAPWAP Discovery Request to a controller IP on **UDP 5246**, or a broadcast to the local
subnet, and a working controller returns a CAPWAP Discovery Response):
1. The AP **broadcasts a CAPWAP Discovery Request on its local wired subnet**; any WLCs on that
   subnet answer with a Discovery Response.
2. **Primed entries** — an AP can be primed with up to three controllers (primary, secondary,
   tertiary) stored in nonvolatile memory so they survive reboot/power failure. Otherwise, if it
   previously joined a controller, it stored up to **8 of a list of 32** WLC addresses received
   from that controller. It tries to contact as many as possible.
3. **DHCP option 43** from the DHCP server that gave the AP its IP suggests a list of WLC addresses.
4. **DNS** — the AP resolves `CISCO-CAPWAP-CONTROLLER.localdomain` (localdomain learned from DHCP)
   and tries to contact a WLC at that address.
5. If none of the steps succeed, the AP **resets itself and starts discovery over again**.

Other listed discovery inputs: prior knowledge of WLCs, and plug-and-play with Cisco DNA Center.
**An over-the-air neighbor message from another AP is not a discovery method.**

**WLC selection (after discovery builds the candidate list):**
1. If the AP previously joined a controller and is **primed** with primary/secondary/tertiary,
   it tries those in succession.
2. If the AP knows no candidate, it tries to discover one — a controller configured as a
   **master controller** responds to the AP's request.
3. The AP attempts to join the **least-loaded WLC** — during discovery each controller reports
   its load as the ratio of currently joined APs to total AP capacity; lowest ratio wins.

Joining sends a CAPWAP Join Request and waits for a Join Response; from there AP and WLC build
the DTLS tunnel securing CAPWAP control messages.

**IOS XE profile/tag customization:**
1. Configure AP and Flex profiles and map them to **site tags**.
2. Configure RF profiles and map them to **RF tags**.
3. Configure WLAN and policy profiles and map them to **policy tags**.
4. Assign the appropriate site, RF, and policy tags to the APs — manually, by CSV import,
   by location, or by regular expression against AP names.

## Reference Tables

**Wireless deployment models**

| Model | WLC location | Typical scale | AP mode | Notes |
|---|---|---|---|---|
| Autonomous | none (AP is standalone) | — | n/a | Trunk to each AP; roaming limited to one VLAN; shortest data path |
| Centralized (controller-based) | data center / near core | up to 6000 APs, 64,000 clients | Local | Single policy enforcement point; client-to-client traffic hairpins via WLC |
| Cloud-based, public | public cloud, over Internet | up to 6000 APs, 64,000 clients | **FlexConnect only** | CAPWAP **control only**; data locally switched |
| Cloud-based, private | private cloud in enterprise DC | up to 6000 APs, 64,000 clients | Local or FlexConnect | Controller inside the enterprise |
| Distributed (controller-based) | one WLC per site, at access layer | up to 250 APs, 5,000 clients | Local (or Flex) | Smaller standalone WLCs per site |
| Controller-less (EWC) | embedded in an AP | up to 100 APs, 2,000 clients | Local | Small/midsize/multisite branch; no discrete WLC |

**Cisco AP modes (set from the WLC)**

| Mode | Radios serve clients? | What it does |
|---|---|---|
| **Local** (default) | Yes | One or more BSSs on a specific channel; scans other channels between transmits for noise, interference, rogues, IDS events |
| **FlexConnect** | Yes | Control CAPWAP tunnel to a central WLC, data forwarded locally without a CAPWAP tunnel; keeps switching SSID↔VLAN locally if the WAN/control tunnel drops |
| **Monitor** | No (RX only) | Dedicated sensor: IDS events, rogue detection, location-based services |
| **Sniffer** | No | Dedicates radios to capturing 802.11 traffic, forwarded to a PC running Wireshark |
| **Rogue detector** | No | Correlates MACs heard on the wire with those heard over the air; devices on both are rogues |
| **Bridge** | No | Dedicated point-to-point or point-to-multipoint bridge; multiple bridge APs form an indoor/outdoor mesh |
| **Flex+Bridge** | Yes | FlexConnect operation enabled on a mesh AP |
| **SE-Connect** | No | Dedicates radios to spectrum analysis on all channels (MetaGeek Chanalyzer, Cisco Spectrum Expert) |

A lightweight AP is normally in **local mode** when providing BSSs. Configuring any other mode
**disables local mode and the BSSs**.

**Key timers, ports, and limits**

| Item | Value |
|---|---|
| CAPWAP discovery / control port | UDP **5246** |
| AP↔WLC round-trip time budget | **< 100 ms** |
| Default AP keepalive interval | **30 s** |
| Escalated keepalives after no answer | **4 more at 3 s** intervals |
| Default failure detection | as little as **35 s** |
| Tunable keepalive / fast heartbeat range | **1–30 s** / **1–10 s** → ~**6 s** detection |
| Primed controllers stored | **3** (primary, secondary, tertiary) |
| WLC addresses cached from last controller | up to **8** of a list of **32** |
| AP priority values | low (default), medium, high, critical |

**IOS XE tags → profiles → parameters**

| Tag | Profiles it maps to | Parameters covered |
|---|---|---|
| **Site** | AP profile (AP join) + Flex profile | CAPWAP timers, AP fallback, TCP MSS, rogue detection, ICap, QoS; native VLAN, local auth, policy ACL, VLAN, DNS security |
| **RF** | RF profile 2.4 GHz + 5 GHz + 6 GHz | Data rates, MCS, RRM, coverage hole detection, TPC, DCA |
| **Policy** | WLAN profile + policy profile | SSID, band, L2 security, L3 security, AAA; VLAN, multicast, ACL, URL filters, QoS ingress/egress |

**Antenna types and typical gain**

| Antenna | Type | Typical gain | Pattern / use |
|---|---|---|---|
| Isotropic | theoretical reference | **0 dBi** | Perfect sphere; doesn't physically exist |
| Omnidirectional (generic) | omni | ~**+4 dBi** | Donut; broad coverage of a room or floor, antenna centered |
| Dipole | omni | **+2 to +5 dBi** | Two radiating wires; articulated or rigid |
| Integrated (inside AP case) | omni | **2 dBi @ 2.4 GHz, 5 dBi @ 5 GHz** | Merged pattern still roughly spherical |
| USB adapter / smartphone | omni | **0 dBi or negative** | Tiny antennas, lower performance — still radiate |
| Patch | directional | **6–8 dBi @ 2.4 GHz, 7–10 dBi @ 5 GHz** | Flat rectangle, wall/ceiling mount; broad egg-shaped pattern |
| Yagi (Yagi-Uda) | directional | **10–14 dBi** | Parallel elements of increasing length; focused egg along its length |
| Parabolic dish | highly directional | **20–30 dBi** — highest of all WLAN antennas | Long narrow elliptical beam for line-of-sight links |

Omnidirectional ⇒ **low gain + large beamwidth**. Directional ⇒ higher gain, narrower beamwidth.

## Config Patterns

```ios-xe
! ---- Access switch: port for a LIGHTWEIGHT AP (access port, not a trunk) ----
interface GigabitEthernet1/0/10
 description AP-FLOOR2-01
 switchport mode access
 switchport access vlan 10
 power inline auto
 spanning-tree portfast
!
! ---- Access switch: port for an AUTONOMOUS AP (trunk: data VLANs + mgmt VLAN) ----
interface GigabitEthernet1/0/11
 description AUTONOMOUS-AP-01
 switchport mode trunk
 switchport trunk allowed vlan 10,100,200
!
! ---- Router/L3 switch: relay AP subnet broadcasts to WLCs on another subnet ----
ip forward-protocol udp 5246
!
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 ip helper-address 10.10.10.10
 ip helper-address 10.10.10.11
!
! ---- DHCP scope for APs with option 43 pointing at the WLC management IPs ----
ip dhcp pool AP-MGMT
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 option 43 hex f104.0a0a.0a0a
!
! ---- IOS XE WLC (C9800): profiles ----
ap profile AP-PROF-SITE1
 description "Site 1 AP join profile"
!
wireless profile flex FLEX-PROF-SITE1
 native-vlan-id 10
!
wireless profile policy POL-CORP
 vlan 100
 no shutdown
!
wlan WLAN-CORP 1 corp-ssid
 security wpa psk set-key ascii 0 <key-from-secrets-not-repo>
 no shutdown
!
! ---- IOS XE WLC: tags that bind those profiles ----
wireless tag site ST-SITE1
 ap-profile AP-PROF-SITE1
 flex-profile FLEX-PROF-SITE1
!
wireless tag rf RF-TAG-X
 24ghz-rf-policy RF-24G-LOWPWR
 5ghz-rf-policy  RF-5G-HIGHDENSITY
!
wireless tag policy PT-CORP
 wlan WLAN-CORP policy POL-CORP
!
! ---- IOS XE WLC: assign the three tags to an AP ----
ap F4DB.E63A.1234
 policy-tag PT-CORP
 rf-tag RF-TAG-X
 site-tag ST-SITE1
!
! ---- IOS XE WLC (exec): prime primary/secondary/tertiary for deterministic failover ----
ap name AP-FLOOR2-01 controller primary WLC-1 10.10.10.10
ap name AP-FLOOR2-01 controller secondary WLC-2 10.10.10.11
ap name AP-FLOOR2-01 controller tertiary WLC-3 10.10.10.12
!
! ---- IOS XE WLC (exec): AP mode and priority ----
ap name AP-BRANCH-01 mode flexconnect
ap name AP-FLOOR2-01 priority 4
```

> Config note: the switch, router-relay, and DHCP option 43 blocks are standard IOS-XE.
> The C9800 profile/tag block reproduces the chapter's model in real controller syntax
> and is **book-derived, not gear-validated** — verify against your controller's release
> before pasting. Never commit a real PSK; pull it from env/secrets at runtime.

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Keep AP↔WLC round-trip time under 100 ms | Above that, APs judge the controller unresponsive, disconnect, and go hunt for another — flapping the whole cell | None benign; if a remote site can't meet it, the fix is FlexConnect or a local/distributed WLC, not accepting the latency | ENCOR 350-401 OCG, Ch. 18, p. 550 (NOTE) |
| Use local mode when AP and WLC share the local infrastructure; FlexConnect when they're remote from each other | Local mode tunnels both control and user data to the WLC — fine on a LAN, wasteful and fragile over a WAN | A remote site with a local/distributed WLC can legitimately stay in local mode | ENCOR 350-401 OCG, Ch. 18, p. 551 (TIP) |
| Prime primary/secondary/tertiary controllers on every AP | "Least-loaded" failover is non-deterministic — a 1000-AP controller failure becomes a stampede while clients sit stranded | With HA SSO pairs, APs need only the active primary; secondary/tertiary become an extra redundancy layer, not a requirement | ENCOR 350-401 OCG, Ch. 18, pp. 556–557 |
| Use HA SSO pairs for controller redundancy | Active syncs CAPWAP tunnels, AP/client state, config, and images to hot standby, so failover is transparent to users in RUN state | Small sites (EWC, single distributed WLC) where a second controller isn't justified | ENCOR 350-401 OCG, Ch. 18, pp. 556–557 |
| Run the same code release on any controllers an AP might rehome between | A version mismatch on join forces a full image download and reboot — live clients hang with no AP while it downloads | A deliberate staged upgrade, done in a maintenance window with predownload staged to the APs first | ENCOR 350-401 OCG, Ch. 18, p. 554 |
| Predownload a new release to APs before rebooting the controller onto it | APs reboot onto an already-staged image instead of serializing downloads from one WLC | Small AP counts where the download window is acceptable | ENCOR 350-401 OCG, Ch. 18, p. 554 |
| Build custom site/RF/policy profiles and tags instead of editing the default ones | Changes to default profiles hit **all** APs globally, and it forecloses future granular policy | A genuinely single-policy site where global defaults are the intent — worth confirming, not assuming | ENCOR 350-401 OCG, Ch. 18, p. 558 |
| Configure the AP's switch port with the correct access VLAN, access mode, and inline power before the AP powers up | Lightweight APs are "touch free" — the switch port is the one thing you must get right for the AP to reach a WLC at all | Static AP addressing or a non-PoE AP with a local power injector | ENCOR 350-401 OCG, Ch. 18, p. 552 |
| Match antenna polarization between transmitter and receiver | A polarization mismatch severely degrades the received signal even with adequate signal strength | None intentional; deliberate cross-polarization belongs to specialized designs outside this scope | ENCOR 350-401 OCG, Ch. 18, pp. 563–564 |
| Size APs for client density, not coverage alone; constrain antenna pattern to limit clients per AP | Coverage-only designs put too many active clients on one channel — airtime contention, poor user experience | Low-density spaces (warehouse aisles, outdoor coverage) where coverage genuinely is the constraint | ENCOR 350-401 OCG, Ch. 18, pp. 559–560 |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show ap summary` | AP name, model, MAC, IP, state — is the AP joined at all, and to this WLC? |
| `show ap status` | Registered vs not; APs stuck out of Run state |
| `show ap uptime` | AP uptime vs association uptime — a short association uptime on a long-lived AP means it's been rehoming/flapping |
| `show ap name <ap> config general` | AP mode (local/FlexConnect/monitor/…), IP, and the **primary/secondary/tertiary controller** names and addresses actually primed |
| `show ap tag summary` | Site/RF/policy tag actually applied per AP — compare against intent |
| `show wireless tag site detail <tag>` | Which AP profile and Flex profile the site tag really maps to |
| `show wireless tag policy detail <tag>` | Which WLAN and policy profiles the policy tag really carries |
| `show wireless tag rf detail <tag>` | The 2.4/5/6 GHz RF policies bound to the RF tag |
| `show wireless stats ap join summary` | Per-AP join attempts and the **last failure reason** — the fastest read on a failing join |
| `show ap join stats detailed <ap-mac>` | Where in discovery/DTLS/join the AP is dying |
| `show ap cdp neighbors` | Which switch and port each AP is actually on |
| `show wireless summary` | Joined AP count vs platform/license max — is the controller full? |
| `show redundancy` / `show chassis` | HA SSO pair state: active vs hot standby, sync status |
| `show ap image` | Per-AP image version vs controller version; predownload status |
| `show ap name <ap> config dot11 24ghz` / `dot11 5ghz` | Channel, TX power, and configured **antenna gain** for the radio |
| `show capwap client rcb` (on the AP CLI) | AP's view: controller name/IP, AP mode, CAPWAP state |
| `show power inline gi1/0/10` (switch) | AP powering up at all — Layer 1 before anything else |
| `show run interface gi1/0/10` (switch) | Access mode + correct access VLAN for lightweight; trunk for autonomous |
| `show run interface vlan 10` (router/L3) | `ip helper-address` present when APs and WLCs are on different subnets |

## Intent Questions
- Which deployment model is this site *supposed* to be — autonomous, centralized, cloud (public or private), distributed, or controller-less EWC — and does each AP's mode (local vs FlexConnect) match where its WLC actually sits?
- Which controller is each AP supposed to join, and by which mechanism — primed primary/secondary/tertiary, DHCP option 43, DNS, subnet broadcast, or master controller? Is that mechanism deterministic on purpose, or is the AP just landing on the least-loaded WLC?
- Which site, RF, and policy tags is this AP supposed to carry, and do the profiles behind those tags match the intended VLAN, SSID list, security, and RF behavior for this location?
- What coverage area and client density is this AP/antenna combination meant to serve, and does the antenna type, gain, and polarization match that intent?

## Troubleshooting Checklist
0. **State intent vs. observed:** answer the Intent Questions above for this network, then write
   the one-line symptom ("should ___, isn't ___") — before running any show command.
1. **Layer 1 / power:** is the AP powered? `show power inline` on the switch port; PoE budget
   exhausted, wrong injector, or a bad cable stops everything downstream.
2. **Antenna physical:** external antennas actually attached, oriented per the design, and not
   knocked sideways — that changes both radiation pattern and **polarization**.
3. **Layer 2 / switch port:** lightweight AP on an **access** port with the right access VLAN
   (a trunk is for autonomous APs); autonomous AP trunk carrying data VLANs **plus** the
   management VLAN.
4. **Layer 3 / addressing:** did the AP get an IP (DHCP or static)? Can it reach the WLC
   management address? Is UDP **5246** blocked by an ACL or firewall in the path?
5. **Cross-subnet discovery:** APs and WLCs on different subnets need
   `ip forward-protocol udp 5246` plus `ip helper-address` pointing at each WLC — a local
   broadcast alone will never leave the subnet.
6. **Discovery inputs:** check DHCP option 43 contents, the `CISCO-CAPWAP-CONTROLLER.localdomain`
   DNS record, and the primed entries stored on the AP. An AP that exhausts every method
   **resets and starts discovery over** — a rebooting AP with no WLC is this loop.
7. **Latency:** measure AP→WLC RTT. Over **100 ms** and APs will disconnect to hunt for a more
   responsive controller — the symptom looks like random flapping, not a hard failure.
8. **Join rejection:** is the controller at its platform/license max AP count? Check
   `show wireless summary` and AP **priority** — a low-priority AP is the one evicted first.
9. **Image mismatch:** `show ap image`. A version mismatch means download → reboot → re-run the
   whole state machine; a slow or repeated cycle looks like an AP that never comes up.
10. **DTLS / certificates:** the CAPWAP tunnel authenticates AP and WLC by certificate exchange —
    an expired or untrusted certificate (or badly wrong device clock/NTP) fails the tunnel before
    the join.
11. **Config / tags:** AP joined and in Run state but not serving the right WLAN? Check
    `show ap tag summary` — wrong policy tag, or an AP still on **default-policy-tag**, which maps
    to nothing and therefore advertises no SSID.
12. **Mode mismatch:** an AP in monitor/sniffer/rogue-detector/SE-Connect mode has **local mode and
    its BSSs disabled** — "AP is up but no SSID" is often just the configured mode.
13. **RF / coverage:** wrong antenna type for the space, gain misconfigured on the radio, or too
    many clients per AP (client density) rather than a coverage hole.
14. **Redundancy behavior:** if APs moved unexpectedly, check HA SSO pair state and whether the
    primed primary/secondary/tertiary are set — undeterministic least-loaded selection explains
    "why did half my APs land over there?"
15. **Software bugs:** only after the above — check release notes for the controller and AP images.

## Common Pitfalls
- **Assuming client-to-client traffic stays local on a lightweight AP.** In default local mode it
  goes up the CAPWAP tunnel to the WLC and back down — through **the AP and its controller**. Only
  an autonomous AP gives the short path.
- **Trunking the port to a lightweight AP.** All VLANs/WLANs ride one CAPWAP tunnel, so the AP needs
  an **access** port and a **single** IP. Trunks belong to autonomous APs.
- **Thinking you pick the AP's software image.** The **WLC** determines the release; the AP downloads
  to match and reboots.
- **Forgetting that broadcast discovery doesn't cross subnets.** Without
  `ip forward-protocol udp 5246` + `ip helper-address`, step 1 of discovery silently gets nowhere.
- **Treating "over-the-air neighbor message from another AP" as a discovery method.** It isn't one.
- **Relying on least-loaded selection for failover.** It's the *least* deterministic option; priming
  primary/secondary/tertiary is the deterministic strategy.
- **Editing the default profiles.** Changes to default-ap-profile / default-flex-profile / the global
  RF config hit **every** AP that hasn't been given custom tags.
- **Expecting default-policy-tag to do something.** It maps to nothing by default — there is no default
  WLAN/SSID config, so an AP left on it serves no SSID.
- **Confusing AireOS AP groups with IOS XE tags.** An AP belongs to exactly one AP group (AireOS); under
  IOS XE it carries three independent tags (site, RF, policy) that can be scoped down to a single AP.
- **Putting a public-cloud WLC in front of local-mode APs.** Public cloud is CAPWAP **control only** —
  the APs must run FlexConnect and switch data locally.
- **Reading gain off a polar plot.** Gain isn't on E/H plots; only the manufacturer's spec sheet has it.
  Plot rings are relative to the outer-ring maximum, not absolute dB.
- **Chasing high gain for coverage.** Higher gain means a *narrower* beamwidth — an omni's virtue is its
  large beamwidth and low gain. A dish gets 20–30 dBi by covering almost nothing off-axis.
- **Ignoring polarization on both ends.** Adequate signal strength with mismatched polarization still
  gives a badly degraded link.
- **Rehoming APs between controllers on different code releases** outside a maintenance window — live
  clients hang while every AP serially downloads an image.
