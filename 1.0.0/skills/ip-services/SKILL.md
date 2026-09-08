---
name: ccnp-ip-services
description: >
  Use this skill when troubleshooting or configuring IP services on IOS-XE —
  time synchronization, first-hop gateway redundancy, and address translation.
  Invoke when the user asks about: NTP, stratum, ntp master, ntp peer, ntp
  server prefer, show ntp associations, clock offset, root dispersion, PTP,
  IEEE 1588, PTPv2, boundary clock, transparent clock, grand master, FHRP,
  first-hop redundancy, HSRP, standby, VRRP, VRRPv3, GLBP, AVG, AVF, virtual
  IP, VIP gateway, virtual MAC, preemption, object tracking, track decrement,
  NAT, PAT, NAT overload, inside local, inside global, outside local, outside
  global, static NAT, pooled NAT, ip nat inside source, show ip nat
  translations, NAT pool exhaustion, RFC 1918.
---

## Purpose
IP services are the router functions layered on top of plain routing and
switching: keeping clocks consistent so logs and certificates can be trusted
(NTP, PTP), keeping a default gateway alive when the router owning it dies
(HSRP, VRRP, GLBP), and rewriting addresses so one address realm can reach
another (NAT, PAT).

## Key Concepts

### Time synchronization — NTP
- RFC 958 introduced NTP. UDP port **123** on the server; the client source
  port is dynamic. Distributed client/server architecture.
- **Stratum** measures distance from the authoritative clock, not accuracy of
  the network path. A device directly attached to an authoritative source
  (atomic clock, GPS) is **stratum 1**; a client of a stratum 1 server is
  stratum 2. Counting continues to **stratum 15**. Higher stratum = more
  accumulated drift, because each hop adds its own error.
- Stratum is about *hops from truth*, not topology hops. A router querying a
  stratum 1 server across five routed hops is still stratum 2.
- Synchronization is slow. A large offset closes to within a couple of seconds
  after a few polling cycles, but tens-of-milliseconds accuracy takes **hours
  or days** of comparison. Client time drifts *toward* server time.
- A Cisco device can act as a server once it has itself synchronized. It can
  also be forced to serve authoritatively with `ntp master stratum-number`,
  which reports the local reference clock as **127.127.1.1**.
- **Stratum preference:** a client configured with several servers uses only
  the server with the **lowest stratum** — it does not blend all of them. If
  the path to that server breaks, it falls back to the next-best server and its
  own stratum rises accordingly; when the path returns, it reverts.
- **NTP peers** (`ntp peer`) act as client *and* server to each other and blend
  their time toward one another, rather than one side dictating. Used when two
  devices each hold a different external reference and should back each other
  up. Peers correct at a maximum of **two minutes per query**, so a large
  discrepancy takes a long time to close.
- Contrast: an NTP *client* changes its clock to match the server; the server
  never moves toward the client. A *peer* treats the other as an equal.

### Time synchronization — PTP
- **IEEE 1588-2002** defined PTP; **IEEE 1588-2008 is PTPv2 and is NOT backward
  compatible with the original PTP.** (Common exam trap.)
- Built for networked measurement and control — industrial networks, energy
  providers doing peak/off-peak usage billing — where drift tolerance is far
  tighter than NTP provides. Low bandwidth and overhead.
- PTP compensates for *variable* delay: congestion, interface buffering, and
  memory delay (a switch doing MAC lookups and CRC verification) all make
  latency inconsistent, which is exactly what breaks naive time transfer.
- **Transparent clock:** the mechanism that measures and accounts for the delay
  a device itself adds, connecting PTP Server and PTP Client clocks. Other
  devices point at these to reduce the latency of getting time from the Grand
  Server.
- **Boundary clock:** a device sitting between areas of the network that
  exchanges PTP messages with the devices nearest it — a geographic hierarchy.
  Per-port PTP requires the switch to be in Boundary mode.
- Two PTPv2 message classes: **General** (not timestamped; builds the
  client/server topology) and **Event** (carries the timestamps).
- Redundancy works like NTP: if the primary server is down, clients redirect to
  the secondary.
- PTP options vary heavily by platform and product family — always check the
  device's own configuration guide.

### First-hop redundancy (FHRP)
- The problem: a host has one default gateway. If that gateway dies, the host
  is isolated even though a second router sits on the same segment, because
  many operating systems cannot use multiple gateways.
- FHRPs solve this by creating a **virtual IP (VIP) gateway instance** shared
  between the Layer 3 devices. Hosts point at the VIP and never learn that the
  router behind it changed.
- Failover is transparent because the **virtual MAC moves with the virtual IP**
  — the hosts' ARP entries stay valid.
- Layer 2 resiliency (multiple switches) and Layer 3 resiliency (multiple
  routers/multilayer switches) are separate problems; FHRP solves only the
  Layer 3 half. STP is still blocking somewhere in the Layer 2 topology.
- **HSRP** and **GLBP** are Cisco proprietary; **VRRP** is the industry standard.
- Only **GLBP** load balances within a single group. HSRP and VRRP load balance
  only by running multiple groups and splitting the hosts across them.

### Object tracking
- Object tracking links FHRP (and other features, like conditional static
  routes) to the real state of something else, so a router can step aside when
  the path it is fronting for is gone.
- Two things commonly tracked:
  - Route reachability: `track N ip route route/prefix-length reachability`
  - Interface line protocol: `track N interface interface-id line-protocol`
- The classic failure it prevents: a distribution switch stays HSRP active
  after losing its uplink to the core, and black-holes everything the hosts
  send it.
- State changes are logged as `%TRACK-6-STATE`.

### HSRP
- Minimum two devices: one **active** (forwards packets sent to the virtual
  MAC), one **standby** ready to take over.
- Election: **highest priority wins** (default **100**); tie broken by the
  **highest IP address** on the segment.
- **HSRP does not preempt by default.** A router that boots later with a
  higher priority stays standby until the current active fails. This is the
  single most common "why isn't my preferred router active?" answer.
- Hellos are multicast UDP. Missing hellos promote the standby.
- Default hello **3 sec**, hold **10 sec**.
- Multiple HSRP instances can run on the same interface; pointing half the
  hosts at one VIP and half at another, with priorities reversed, is how HSRP
  load balances.
- Tracking lowers priority on failure:
  `standby N track object-id decrement decrement-value`. **The decrement must
  be large enough that the resulting priority drops below the peer's** — a
  decrement of 5 against a 10-point lead changes nothing.
- State changes log as `%HSRP-5-STATECHANGE`.

### VRRP
- Behaves almost identically to HSRP. The differences are what get tested:
  - The preferred router is the **master**; the others are **backup** routers.
  - **Preemption is enabled by default** (the opposite of HSRP).
  - Virtual MAC is `0000.5e00.01xx`, where `xx` is the group ID in hex.
  - Multicast address **224.0.0.18**.
- **VRRPv2** supports IPv4 only. **VRRPv3** supports IPv4 and IPv6.
- **VRRPv2 and VRRPv3 are not compatible** — VRRPv3 offers a `vrrpv2`
  compatibility mode for migration.
- VRRPv3 configuration on IOS-XE is *hierarchical*: `fhrp version vrrp v3`
  globally, then an address-family sub-mode under the interface where the
  address, priority, and tracking are nested.
- Default advertisement interval 1000 msec; master down interval ~3.609 sec.
- State changes log as `%VRRP-6-STATE`.

### GLBP
- Provides gateway redundancy **and** load balancing inside one group.
- Two roles:
  - **AVG (Active Virtual Gateway):** one elected per group. It answers the
    initial ARP requests for the VIP, handing back the virtual MAC of one of
    the AVFs. This is the load-balancing lever — the AVG decides which router
    each host will use by which MAC it hands out.
  - **AVF (Active Virtual Forwarder):** actually routes the traffic from the
    hosts assigned to it. The AVG creates and assigns each AVF a unique virtual
    MAC. ARP replies are **unicast**, so other hosts on the segment do not see
    them. AVFs appear as **Fwd** instances in the CLI.
- A group supports **four active AVFs and one AVG**. One router can be both AVG
  and an AVF at the same time.
- AVG failure causes **no traffic disruption** — the role moves to a standby
  AVG. AVF failure causes another router to take over that AVF's forwarding
  duties, including its virtual MAC.
- Three load-balancing methods (default **round-robin**):
  - **Round robin:** hand out each AVF's virtual MAC in sequence.
  - **Weighted:** distribute in proportion to configured weights — give bigger
    routers more traffic.
  - **Host dependent:** derive the AVF from the host's own MAC, so a given host
    keeps the same gateway as long as the number of forwarders is unchanged.
- Defaults seen in `show glbp`: hello 3 sec, hold 10 sec, redirect time 600
  sec, forwarder timeout 14400 sec, weighting 100 (thresholds lower 1, upper
  100).
- Logs as `%GLBP-6-STATECHANGE` (AVG) and `%GLBP-6-FWDSTATECHANGE` (AVF).

### NAT
- **RFC 1918** defined the private blocks that must never appear on the
  Internet: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- NAT lets an internal network appear as a publicly routed one. A NAT device
  (router or firewall) rewrites source or destination addresses in the IP
  header as the packet arrives on the inside or outside interface.
- NAT is not only an Internet-access tool — it is also how you connect two
  merged companies whose address space overlaps.
- **Most routers and switches translate only the IP header, not the payload.**
  An address embedded in a DNS response or an application payload is untouched.
  Some firewalls can do payload NAT for certain traffic types.
- The four address terms are defined by *whose network the host is on* (inside
  vs outside) and *which realm you are looking from* (local vs global).
- **The inside global address does not have to live on the outside network.**
  It must not be the outside interface's own address, and it may belong to a
  network that does not exist on the NAT router at all — as long as every
  outside router has a route pointing that prefix back toward the NAT router.
- Pooled NAT's weakness is **exhaustion**: when every global address is in use,
  new translations simply fail and packets are dropped until one ages out.
- Default translation timeout is **24 hours**.
- **Direction is named for the traffic that creates the translation, not for
  which interface a packet happens to enter.** `ip nat inside source` rewrites
  the source of inside-originated traffic; `ip nat outside source` rewrites the
  source of outside-originated traffic.

## Procedure

**NTP stratum re-selection when a path fails:**
1. The client is configured with multiple NTP servers and synchronizes only
   with the one advertising the **lowest stratum**.
2. That path breaks (an intermediate router crashes, or the server stops
   answering).
3. The client falls back to the next-lowest-stratum server it can still reach.
4. The client's own stratum rises to that server's stratum + 1 — so a client
   that was stratum 2 off a stratum 1 server becomes stratum 4 behind a
   stratum 3 server.
5. When the preferred path recovers, the client re-selects the lowest-stratum
   server and its own stratum drops back.

**HSRP election and failover:**
1. Each HSRP-enabled interface in the group is configured with the same
   virtual IP.
2. HSRP elects the active router: **highest priority** (default 100) wins;
   on a tie, the **highest interface IP address** wins.
3. The active router assumes the group's virtual IP *and* virtual MAC and
   forwards all traffic sent to that virtual MAC.
4. The active router multicasts hello messages. The standby watches them.
5. If the standby stops receiving hellos (or the active stops sending), the
   standby with the next-highest priority becomes active.
6. The transition is transparent to hosts because the virtual MAC moves with
   the virtual IP — no host ARP entry has to change.
7. Without `preempt`, the recovered router does **not** take the role back,
   even if its priority is higher.

**HSRP failover driven by object tracking:**
1. Create a tracked object against the thing that actually matters — usually
   the WAN/core uplink's line protocol or the existence of a route learned
   through it.
2. Raise the preferred router's HSRP priority above its peer's (e.g. 110 vs
   the default 100) so it wins the election normally.
3. Bind the tracking to HSRP with a decrement large enough to push the
   priority *below* the peer's when the object goes down (e.g. `decrement 20`
   turns 110 into 90, losing to 100).
4. The uplink fails. The tracked object transitions to down and logs
   `%TRACK-6-STATE`.
5. HSRP applies the decrement, the router loses the election, and it moves
   Active → Speak → Standby (`%HSRP-5-STATECHANGE`).
6. The peer becomes active and traffic stops being drawn toward the broken
   uplink. Recovery reverses the sequence — but only if preemption is enabled.

**NAT translation processing (inside static NAT, packet in both directions):**
1. Traffic enters the router's **inside** interface. The router performs a
   route lookup on the destination, which points out the **outside**
   interface. Knowing one side is `ip nat inside` and the other `ip nat
   outside`, it consults the NAT table.
2. Only the static entry exists, so the router creates a **dynamic** entry
   recording the packet's destination as the outside local and outside global
   address.
3. The router translates the packet's **source** address from the inside local
   to the inside global address, and forwards it.
4. The far-end host sees the session as coming from the inside global address
   and sends its reply there. Intermediate routers forward that reply back
   toward the NAT router using their normal routes.
5. The return packet arrives on the router's **outside** interface. Recognizing
   it as an outside NAT interface, the router checks the NAT table.
6. It matches the packet's source and destination ports against the existing
   entry and rewrites the **destination** address from the inside global back
   to the inside local address.
7. The router routes the packet out the inside interface to the real host.

**Pooled NAT allocation and exhaustion:**
1. An inside host sends traffic matching the ACL that defines NAT-eligible
   sources.
2. The router allocates the next free global address from the pool and creates
   a dynamic one-to-one binding.
3. Subsequent flows from that same inside host reuse the same global address
   while the binding lives.
4. The binding survives until traffic stops **and** the timeout (24 hours by
   default) expires; the global address then returns to the pool.
5. If every pool address is already bound when a new host needs one,
   allocation **fails and the packet is dropped** — `debug ip nat detailed`
   shows `NAT: failed to allocate address` followed by
   `NAT: translation failed (A), dropping packet`. The host sees
   "Destination unreachable."
6. `clear ip nat translation *` frees the pool immediately — at the cost of
   breaking every active session, since surviving flows may be re-mapped to
   different global addresses.

**GLBP host assignment:**
1. The group elects one **AVG**; each participating router becomes an **AVF**
   and is assigned a unique virtual MAC by the AVG.
2. A host ARPs for the VIP.
3. The AVG — and only the AVG — answers, returning the virtual MAC of one AVF
   chosen by the configured load-balancing method.
4. The reply is **unicast**, so other hosts on the segment do not learn or
   cache that mapping.
5. The host sends its traffic to the VIP, which resolves to that AVF's virtual
   MAC, and that router forwards it.
6. If an AVF fails, another router assumes its virtual MAC so the hosts already
   pointed at it keep working without re-ARPing.

## Reference Tables

**HSRPv1 vs HSRPv2**

| | HSRPv1 | HSRPv2 |
|---|---|---|
| Timers | Does not support millisecond timer values | Supports millisecond timer values |
| Group range | 0 to 255 | 0 to 4095 |
| Multicast address | 224.0.0.2 | 224.0.0.102 |
| MAC address range | `0000.0C07.ACxy`, where *xy* is a hex value representing the HSRP group number | `0000.0C9F.F000` to `0000.0C9F.FFFF` |

**FHRP comparison**

| | HSRP | VRRP | GLBP |
|---|---|---|---|
| Standard | Cisco proprietary | Industry standard | Cisco proprietary |
| Preferred router called | Active | Master | AVG (gateway) / AVF (forwarder) |
| Preemption default | **Disabled** | **Enabled** | Disabled (`preempt` to enable) |
| Virtual MAC | v1 `0000.0C07.ACxy` / v2 `0000.0C9F.Fxxx` | `0000.5e00.01xx` | `0007.b400.xxyy` per AVF |
| Multicast address | v1 224.0.0.2 / v2 224.0.0.102 | 224.0.0.18 | 224.0.0.102 |
| Priority range | 0–255 (default 100) | 1–255 (default 100) | 1–255 (default 100) |
| Load balancing | Only via multiple groups | Only via multiple groups | **Yes, within one group** |
| IPv6 | HSRPv2 | VRRPv3 | Yes |

**NAT address terminology**

| Term | Meaning |
|---|---|
| **Inside local** | The actual private IP assigned to a device on the inside network(s) |
| **Inside global** | The public IP that represents one or more inside local IPs to the outside |
| **Outside local** | The IP of an outside host *as it appears to the inside network*; need not be reachable by the outside, but must be reachable by the inside |
| **Outside global** | The public IP actually assigned to a host on the outside network; must be reachable by the outside network |

**NAT types**

| Type | Mapping | Behavior |
|---|---|---|
| **Static NAT** | One-to-one, fixed | Manually configured mapping of a local to a global address; entry is permanent |
| **Pooled NAT** | One-to-one, dynamic | Global address temporarily assigned from a pool; returned after the idle timeout (24 h default) expires |
| **PAT (NAT overload)** | Many-to-one, dynamic | Many local addresses share one global address; the NAT device rewrites source **ports** to keep return traffic unambiguous |

**PTP message types**

| Class | Message | Purpose |
|---|---|---|
| General | Announce | Determines which Grand Master is selected Best Master |
| General | Follow_Up | Conveys a captured timestamp of a transmitted SYNC message |
| General | Delay_Response | Measures delay between IEEE 1588 devices |
| General | Pdelay_Response_Follow_Up | Measures the delay on an incoming link |
| General | Management | Used between management devices and clocks |
| General | Signaling | Used by clocks to deliver how messages are sent |
| Event | Sync | Conveys time |
| Event | Delay_Request | Measures delay from downstream devices |
| Event | Pdelay_Request | Initiates and measures delay |
| Event | Pdelay_Response | Responds and measures delay |

General messages are **not** timestamped and exist to build the client/server
topology. Sync and Delay_Request synchronize ordinary and boundary clocks;
the Pdelay_* messages measure link delay between devices.

**Default PTP parameters (Cisco IE2000 switch)**

| Feature | Default setting |
|---|---|
| PTP Boundary Mode | Disabled |
| PTP Forward Mode | Disabled |
| PTP Transparent Mode | Enabled |
| PTP Priority1 and Priority2 | 128 |
| PTP Announce Interval | 2 seconds |
| PTP Announce Timeout | 8 seconds |
| PTP Delay Request Interval | 32 seconds |
| PTP Sync Interval | 1 second |
| PTP Sync Limit | 50,000 nanoseconds |

**`show ntp associations` flags**

| Flag | Meaning |
|---|---|
| `*` | sys.peer — the server currently being synchronized to |
| `#` | selected |
| `+` | candidate |
| `-` | outlier |
| `x` | falseticker — the server's time is rejected as wrong |
| `~` | configured |

## Config Patterns

```ios-xe
! ===== NTP =====
! Authoritative server (no upstream reference available)
ntp master 1

! Client pointing at a server, sourcing queries from a stable interface
ntp server 192.168.1.1 source Loopback0
ntp server 192.168.2.2 prefer

! NTP peering — two devices with different upstream references backing
! each other up; each treats the other as an equal
ntp peer 192.168.2.2

! NTP authentication (Cisco hardening baseline)
ntp authenticate
ntp authentication-key 5 md5 <key-string>
ntp trusted-key 5
ntp server 172.16.1.5 key 5
clock timezone EST -5

! ===== PTP (platform-dependent; IE2000 shown) =====
ptp mode boundary

! ===== Object tracking =====
track 1 interface Vlan1 line-protocol
track 2 ip route 192.168.3.3/32 reachability

! ===== HSRP with tracking =====
interface Vlan10
 ip address 172.16.10.2 255.255.255.0
 standby version 2
 standby 10 ip 172.16.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track 1 decrement 20
 standby 10 authentication md5 key-string <key-string>

! Peer switch — same VIP, default priority, preempt enabled
interface Vlan10
 ip address 172.16.10.3 255.255.255.0
 standby version 2
 standby 10 ip 172.16.10.1
 standby 10 preempt

! ===== VRRPv2 (legacy, flat syntax, IPv4 only) =====
interface GigabitEthernet0/0
 ip address 172.16.20.2 255.255.255.0
 vrrp 20 ip 172.16.20.1
 vrrp 20 priority 110
 vrrp 20 track 1 decrement 20

! ===== VRRPv3 (hierarchical; note the global version command first) =====
fhrp version vrrp v3
!
interface Vlan22
 ip address 172.16.22.2 255.255.255.0
 vrrp 22 address-family ipv4
  address 172.16.22.1
  priority 110
  track 1 decrement 20

! ===== GLBP with weighted load balancing =====
interface Vlan30
 ip address 172.16.30.2 255.255.255.0
 glbp 30 ip 172.16.30.1
 glbp 30 preempt
 glbp 30 load-balancing weighted
 glbp 30 weighting 20

! ===== NAT: interface roles come first, always =====
interface GigabitEthernet0/0
 ip nat outside
interface GigabitEthernet0/1
 ip nat inside

! Inside static NAT — one-to-one, permanent
ip nat inside source static 10.78.9.7 10.45.1.7

! Outside static NAT — hides real external addresses from inside hosts;
! add-route installs the required route toward the outside-local address
ip nat outside source static 10.123.4.2 10.123.4.222 add-route

! Pooled NAT — dynamic one-to-one from a pool
ip access-list standard ACL-NAT-CAPABLE
 permit 10.78.9.0 0.0.0.255
!
ip nat pool R5-OUTSIDE-POOL 10.45.1.10 10.45.1.11 prefix-length 24
ip nat inside source list ACL-NAT-CAPABLE pool R5-OUTSIDE-POOL

! PAT (NAT overload) onto the outside interface address
ip nat inside source list ACL-NAT-CAPABLE interface GigabitEthernet0/0 overload

! Tuning and clearing
ip nat translation timeout 3600
! clear ip nat translation *          ! breaks every active session
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Explicitly configure a trusted time source and enable NTP authentication (`ntp authenticate` / `ntp authentication-key` / `ntp trusted-key` / `ntp server … key N`) | Unauthenticated NTP lets an attacker feed false time, which invalidates log correlation, certificate validity checks, and time-based keys; it can also be used to crash or overload the router | Fully isolated lab or OOB management network with no untrusted reachability; platform or upstream server that cannot do MD5 keying | [Cisco Guide to Harden Cisco IOS Devices](https://www.cisco.com/c/en/us/support/docs/ip/access-lists/13608-21.html) |
| Configure the device time zone (`clock timezone`) rather than leaving it default | Timestamps must be accurately correlated across devices during an incident; mixed or unset zones make cross-device log correlation unreliable | Standardizing every device on UTC instead — a deliberate, equally valid choice as long as it is uniform | [Cisco Guide to Harden Cisco IOS Devices](https://www.cisco.com/c/en/us/support/docs/ip/access-lists/13608-21.html) |
| Align the active FHRP gateway with the STP root bridge for each VLAN | If the active gateway and the root bridge sit on different distribution switches, every off-VLAN packet climbs to the wrong switch and tromboned across the inter-distribution trunk | Routed-access designs with no Layer 2 between distribution switches; a stack or VSS/StackWise-Virtual pair that presents one logical device | [Campus Network for High Availability Design Guide (CVD)](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Campus/HA_campus_DG/hacampusdg.html) |
| Set FHRP preempt delay to roughly the measured system boot time **+50%** (e.g. `standby 1 preempt delay minimum 180`) | A rebooting distribution switch can win preemption before its uplinks to the core converge, becoming an active gateway that black-holes traffic | Deliberately leaving preemption off so failback is a manual, scheduled action; environments where the peer is known to be the long-term preferred router | [Campus Network for High Availability Design Guide (CVD)](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Campus/HA_campus_DG/hacampusdg.html) |
| Use millisecond FHRP timers where sub-second gateway failover is required | Default 3/10-second hello/hold timers leave a multi-second outage; millisecond timers reliably achieve sub-second (~800 ms) HSRP/GLBP failover | CPU-constrained platforms, or shared/unstable links where aggressive timers cause false failovers and role flapping — a stable 3-second failover beats a flapping 200 ms one | [Campus Network for High Availability Design Guide (CVD)](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Campus/HA_campus_DG/hacampusdg.html) |
| Address inside networks from RFC 1918 space and translate at the edge | The private blocks are defined as non-globally-routable; using non-RFC-1918 space internally means squatting on address space someone else owns and can break reachability to the real owner | Organizations holding genuine public allocations that route them internally without NAT; overlap remediation after a merger where one side must be re-addressed anyway | [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) |

*A deviation from this table is a question for the network's operator — "is
this intentional here?" — never automatically a finding.*

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show ntp status` | "Clock is synchronized" plus the local stratum and reference clock. `.LOCL.` means the device is its own reference (`ntp master`). Unsynchronized, or a stratum higher than expected, means it fell back to a worse source |
| `show ntp associations` | The `*` flag marks the server actually in use. `x` (falseticker) means the server's time was rejected; a `reach` value other than 377 means polls are being lost |
| `show ptp clock` | PTP Device Type (Boundary / End-to-End transparent), Clock Identity, Priority1/Priority2, Offset From Master, Mean Path Delay, Steps Removed |
| `show ptp port <interface>` | **`Port state FAULTY: FALSE`** is healthy. `FAULTY: TRUE` means this port cannot communicate PTP with its neighbor |
| `show track [object-number]` | The object's state (Up/Down), what it tracks, the change counter, and the first-hop interface. A high change count means something is flapping |
| `show standby brief` | Per-interface group, priority, `P` for preempt configured, State (Active/Standby/Speak/Init), the active and standby addresses, and the virtual IP |
| `show standby` | Adds state-change count and time since last change, virtual MAC ("MAC In Use" only on the active), timers, and `Track object N state … decrement …` — this is where you confirm tracking is actually wired to the group |
| `show vrrp brief` | Group, priority, `Pre` column for preemption, State (Master/Backup), master address, and group (virtual) address |
| `show vrrp` | Virtual MAC (`0000.5e00.01xx`), advertisement interval, master down interval, and tracked-object decrement |
| `show glbp brief` | The AVG row shows `-` in the Fwd column; the numbered rows are the AVFs with their virtual MACs, showing which router is Active for each forwarder |
| `show glbp` | Load-balancing method, weighting and thresholds, group members by MAC, and per-forwarder state, owner ID, and redirect/timeout counters |
| `show ip nat translations` | Pro / Inside global / Inside local / Outside local / Outside global. A static entry has `---` in the protocol and outside fields; dynamic session entries carry ports |
| `show tcp brief` | On the *end hosts* — confirms which address the far end actually sees, proving whether translation happened and to what |
| `debug ip nat detailed` | `NAT: failed to allocate address for <ip>, list/map <ACL>` followed by `translation failed (A), dropping packet` = **pool exhaustion** |
| `show ip route` | On the outside routers — a route must exist pointing the inside global prefix back toward the NAT router, or return traffic never arrives |

## Intent Questions
- **Time:** what is the authoritative source of time for this network, how many
  stratum levels sit between it and this device, and is the synchronization
  path authenticated? Is the device supposed to be a client, a peer, or an
  authoritative server (`ntp master`)?
- **Gateway redundancy:** which device is *supposed* to be the active gateway
  for each VLAN, and why that one? Is that choice aligned with the STP root,
  or is traffic intended to trombone?
- **Failover behavior:** what is the FHRP supposed to be watching — just the
  local interface, or an upstream path? Should the preferred router take the
  role back automatically after recovery, or is manual failback intended?
- **NAT:** which direction is traffic supposed to be initiated from, and which
  addresses are meant to be visible to whom? Is any inbound reachability to an
  inside host expected (which requires a static entry, not PAT)?

## Troubleshooting Checklist
0. **State intent vs. observed:** answer the Intent Questions above for this
   network, then write the one-line symptom ("VLAN 10 hosts should exit via
   SW2, but SW3 is HSRP active"; "inside hosts should reach the Internet, but
   only the first two get through"). Do this before running any show command.
1. **Layer 1/2 — is the segment actually up?** FHRP peers must share a Layer 2
   broadcast domain. Check that the SVI/interface is up, the VLAN exists and
   is allowed on the trunks between the peers, and STP is not blocking the path
   between them. Two routers that cannot hear each other both become active,
   producing a duplicate-IP and MAC-flapping symptom.
2. **Layer 3 — is the addressing right?** Confirm both peers are in the same
   subnet, share the same group number, and use the same virtual IP. A VIP
   configured differently on each peer produces two independent "working"
   groups. For VRRPv3, confirm `fhrp version vrrp v3` is set on both.
3. **Verify FHRP role and why:** `show standby brief` / `show vrrp brief` /
   `show glbp brief`. If the wrong device is active, check priority first, then
   whether **preemption is enabled** — HSRP does not preempt by default, so a
   lower-priority router that booted first legitimately keeps the role.
4. **Verify object tracking is actually linked:** `show track` shows the object
   state; `show standby` shows whether the group references it and by how much.
   A tracked object that is down but whose decrement is too small changes the
   priority without changing the outcome — check the arithmetic against the
   peer's priority, not just that tracking exists.
5. **NTP: is it synchronized at all?** `show ntp status` — "Clock is
   unsynchronized" points at reachability to the server, an ACL or firewall
   blocking UDP 123, or a mismatched authentication key. Remember that
   synchronization is genuinely slow; a freshly configured client legitimately
   takes several polling cycles.
6. **NTP: is it synchronized to the *right* source?** `show ntp associations` —
   confirm the `*` is on the intended server and check `reach`. An unexpectedly
   high local stratum means the device fell back to an inferior server, which
   usually means the preferred path is broken upstream, not here.
7. **NAT: are the interface roles configured?** Missing `ip nat inside` or
   `ip nat outside` is the most common NAT failure — the translation statement
   is accepted and looks correct, but nothing is ever translated. Verify on
   both interfaces.
8. **NAT: is traffic matching the ACL?** Confirm the ACL permits the actual
   source subnet. Check `show ip nat translations` for entries appearing at
   all; no entries means matching or interface roles, not translation logic.
9. **NAT: is the pool exhausted?** If some hosts work and others do not, run
   `debug ip nat detailed` and look for `failed to allocate address`. Pooled
   NAT with fewer global addresses than concurrent hosts fails exactly this
   way; PAT is the fix, not a bigger pool.
10. **NAT: does return traffic have a path?** The outside routers must have a
    route for the inside global prefix pointing at the NAT router. For outside
    static NAT, a route for the outside-local address must exist *before* NAT
    occurs — this is what `add-route` installs.
11. **Config errors and version mismatches:** HSRPv1 vs v2 (different multicast
    addresses and group ranges — mismatched versions never form), VRRPv2 vs
    VRRPv3 (not compatible), PTPv2 vs PTP (not backward compatible),
    authentication key mismatches between peers.
12. **Software behavior:** PTP feature support and defaults vary substantially
    by platform and product family — validate against that device's own
    configuration guide before concluding the configuration is wrong.

## Common Pitfalls
- **Assuming an NTP client blends time from all configured servers.** It does
  not — it uses only the lowest-stratum reachable server. Extra servers are
  redundancy, not averaging.
- **Assuming HSRP preempts.** It does not by default. The device that came up
  first keeps the active role regardless of priority. VRRP is the opposite —
  it preempts by default — so migrating a design between the two silently
  changes failback behavior.
- **A tracking decrement too small to matter.** Tracking that fires correctly
  but only drops the priority from 110 to 105 against a peer at 100 changes
  nothing. The decrement must push the priority *below* the peer's.
- **Tracking the wrong object.** Tracking the local interface the hosts sit on
  tells you nothing about whether the uplink to the core is alive — which is
  the failure that actually black-holes traffic.
- **Forgetting `ip nat inside` / `ip nat outside`.** The translation command is
  accepted without complaint and the config reads correctly, but no translation
  ever happens. This is the number-one NAT mistake.
- **Assuming the inside global address must live on the outside network.** It
  must not be the outside interface's own address, and it can even belong to a
  network that does not exist on the NAT router — provided the outside routers
  route that prefix back toward it.
- **Expecting NAT to fix embedded addresses.** Routers and switches translate
  the IP header only. An address inside a DNS response or an application
  payload passes through untouched.
- **Confusing `ip nat inside source` with `ip nat outside source`.** The
  keyword names the traffic that creates the translation, not the interface a
  given packet enters on.
- **Running `clear ip nat translation *` on a production router.** It removes
  every translation and interrupts active sessions, which may be remapped to
  different global addresses on reconnect.
- **Sizing a NAT pool by host count instead of concurrent flows,** then being
  surprised when some users work and others get "Destination unreachable."
  Pool exhaustion is silent unless you are debugging.
- **Believing PTPv2 will interoperate with original PTP.** IEEE 1588-2008 is
  explicitly not backward compatible with IEEE 1588-2002.
- **Ignoring `Port state FAULTY: TRUE` on a PTP port** — it is the direct
  indicator that the port cannot communicate PTP with its neighbor.
- **Treating GLBP as a drop-in HSRP replacement for load balancing across
  distribution switches.** GLBP splits hosts across AVFs by handing out
  different virtual MACs in ARP replies; if the upstream Layer 2 or STP
  topology is not symmetric, half the hosts get a path that tromboned.
