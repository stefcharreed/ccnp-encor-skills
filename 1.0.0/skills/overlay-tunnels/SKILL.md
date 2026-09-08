---
name: ccnp-overlay-tunnels
description: >
  Use this skill when troubleshooting or configuring overlay tunnels on
  IOS-XE — GRE, IPsec VPNs, LISP, and VXLAN. Invoke when the user asks about:
  overlay network, underlay network, GRE tunnel, tunnel source, tunnel
  destination, recursive routing, tunnel MTU, IPsec, AH, ESP, transport mode,
  tunnel mode, transform set, crypto map, IPsec profile, tunnel protection,
  ISAKMP, IKE, IKEv1, IKEv2, main mode, aggressive mode, quick mode, MM1,
  QM_IDLE, SA_INIT, IKE_AUTH, CREATE_CHILD_SA, Diffie-Hellman group, PFS,
  perfect forward secrecy, pre-shared key, VTI, virtual tunnel interface,
  DMVPN, GET VPN, FlexVPN, show crypto isakmp sa, show crypto ipsec sa, LISP,
  EID, RLOC, ITR, ETR, xTR, PITR, PETR, map server, map resolver, map cache,
  map request, negative map reply, VXLAN, VNI, VTEP, MAC-in-IP, SD-Access.
---

## Purpose
Overlay tunnels carry one network's traffic across another network that knows
nothing about it — encapsulating packets so private sites, Layer 2 segments,
or mobile endpoints can span an untrusted or differently-addressed underlay.
GRE provides the encapsulation with no security, IPsec provides the security,
and LISP and VXLAN replace static point-to-point tunnels with a mapping system
that scales.

## Key Concepts

### Overlay vs underlay
- The **underlay** is the physical/routed infrastructure that actually moves
  packets. The **overlay** is the virtual network built on top of it out of
  tunnels. The underlay must be working and routable before any overlay can
  form — the tunnel endpoints have to reach each other natively.
- This is the single most useful troubleshooting frame for this whole topic:
  **an overlay problem is an underlay problem until proven otherwise.**

### GRE
- **Generic Routing Encapsulation**, IP **protocol 47**. Developed by Cisco,
  standardized in RFC 2784. Encapsulates a wide variety of protocols inside
  IP, creating a virtual point-to-point link between two endpoints.
- GRE adds **24 bytes** of overhead (a 20-byte outer IP header plus a 4-byte
  GRE header), which is why the default tunnel IP MTU is 1476 on a 1500-byte
  underlay.
- **GRE offers no encryption, no authentication, and no security services at
  all — it is highly susceptible to attack.** Traffic that needs protecting
  must be paired with IPsec. This is the reason GRE over IPsec exists.
- GRE's value is what IPsec alone cannot do: it can carry multicast and
  routing protocols, so an IGP can form an adjacency across the tunnel.
- **Recursive routing** is GRE's classic self-inflicted failure: if the router
  learns the route to the tunnel *destination* through the tunnel itself, the
  tunnel tears itself down and flaps (`%TUN-5-RECURDOWN`). The fix is to make
  sure the tunnel destination is reachable via the underlay only — never
  advertise the tunnel endpoints into the overlay routing protocol.

### IPsec fundamentals
- IPsec is a framework, not one protocol. It provides four security services:
  **confidentiality** (encryption), **data integrity** (hashing),
  **peer authentication**, and **anti-replay** (sequence numbers).
- Two protocols do the work:
  - **AH (Authentication Header)** — IP **protocol 51**. Integrity and
    authentication **but no encryption**. Because it authenticates the outer
    IP header, **AH breaks through NAT.**
  - **ESP (Encapsulating Security Payload)** — IP **protocol 50**. Encryption
    plus integrity and authentication. This is what is actually deployed.
- Two encapsulation modes:
  - **Transport mode** keeps the original IP header and protects only the
    payload. Used between endpoints, and — importantly — for GRE over IPsec,
    because GRE has already added its own routable header.
  - **Tunnel mode** encrypts the entire original packet and adds a new IP
    header. Used for plain site-to-site VPNs and for VTI.
- A **transform set** is the agreed combination of encryption and hashing
  algorithms, plus the mode. Both peers must agree on a transform set during
  IPsec SA negotiation for a given data flow.

### IKE — how the SAs get built
- **IKEv1** works in two phases:
  - **Phase 1** builds a single **bidirectional** ISAKMP/IKE SA, using either
    **main mode** (6 messages) or **aggressive mode** (3 messages).
  - **Phase 2** uses **quick mode** (3 messages) over the existing IKE SA and
    produces **two unidirectional IPsec SAs**, one on each peer.
  - Totals worth memorizing: **main mode 6 + quick 3 = 9 messages**;
    **aggressive 3 + quick 3 = 6 messages**.
- Main mode **hides the identities** of the two peers and can offer more
  security proposals — more secure and more flexible, but slower. Aggressive
  mode is faster but **exposes the peers' identities to eavesdropping** and is
  less flexible.
- The **lifetime** (default 24 hours) is **the only phase 1 parameter that does
  not have to match exactly** — if the peers disagree, they use the smaller of
  the two values. Everything else in the proposal must match.
- **Perfect Forward Secrecy (PFS)** is an optional but recommended phase 2
  function. It requires additional DH exchanges (extra CPU) and derives session
  keys independently of any previous key, so **a compromised key does not
  compromise future keys.**
- **IKEv2** restructures all of this into request/response pairs called
  *exchanges*:
  - **IKE_SA_INIT** negotiates cryptographic algorithms, exchanges nonces, and
    performs the DH exchange — the equivalent of IKEv1's MM1–MM4 in one pair.
  - **IKE_AUTH** authenticates the previous messages, exchanges identities and
    certificates, and establishes the IKE SA plus a child SA (the IPsec SA) —
    the equivalent of MM5–MM6 plus QM1–QM2 in one pair.
  - **Four messages total**, versus six with aggressive mode or nine with main
    mode. Additional IPsec SAs need only two messages via **CREATE_CHILD_SA**,
    versus three with quick mode.
- **IKEv1 and IKEv2 are incompatible** — the SA exchanges are completely
  different. See RFC 7296.
- IKEv2 additions worth knowing: **EAP** support (which makes IKEv2 the natural
  fit for remote-access VPNs), **ECDSA-SIG**, **NGE** algorithms,
  **asymmetric authentication** (each peer may choose its own authentication
  method, specified during IKE_AUTH — IKEv1 requires both peers to use the
  same method), and **anti-DoS** protection.

### GRE over IPsec vs VTI
- Two ways to encrypt traffic over a GRE tunnel: **crypto maps** or **tunnel
  IPsec profiles**.
- **Crypto maps should not be used for tunnel protection.** The book is
  explicit about why: they cannot natively support MPLS, configuration becomes
  overly complex, **crypto ACLs are commonly misconfigured**, and crypto ACL
  entries can consume excessive TCAM. They remain widely deployed, so they
  still have to be understood — but IPsec profiles are the modern answer.
- A crypto map is applied to the **outside physical interface, not the tunnel
  interface**. An IPsec profile is applied to the **tunnel interface** with
  `tunnel protection ipsec profile`.
- **GRE over IPsec in transport mode:** the GRE IP header stays in the clear
  and is used for routing; ESP encrypts only the GRE payload.
- **GRE over IPsec in tunnel mode** (also called *IPsec over GRE*): the entire
  GRE packet is encrypted and a new IPsec IP header is added; routing uses the
  IPsec IP header. This is **double encapsulation** — which is exactly why
  transport mode is preferred for GRE over IPsec.
- In **both** GRE over IPsec modes the outer IP protocol number is **50 (ESP)**,
  not 47 — the next header is ESP.
- **VTI (Virtual Tunnel Interface) over IPsec** encapsulates IPv4 or IPv6
  **without an additional GRE header**. Configuration is the same as GRE over
  IPsec with IPsec profiles plus `tunnel mode ipsec {ipv4 | ipv6}` on the
  tunnel interface and `mode tunnel` in the transform set. Revert with
  `tunnel mode gre {ip | ipv6}`.

### Cisco IPsec VPN solutions
- **Site-to-site (LAN-to-LAN)** IPsec VPNs are the most versatile because they
  are the only **multivendor** option — but they are very difficult to manage
  at scale, do not support routing or multicast, and scale poorly.
- **DMVPN** simplifies hub-and-spoke and spoke-to-spoke by combining
  **multipoint GRE (mGRE), IPsec, and NHRP**. Spoke-to-spoke tunnels come up
  on demand and are torn down automatically when no traffic is present.
- **GET VPN** is **tunnel-less** — it preserves the original IP header — for
  any-to-any encryption across an MPLS or private WAN without breaking the
  provider's multicast and QoS services. It needs GRE or DMVPN to carry private
  addresses across the Internet. Often deployed to satisfy HIPAA, SOX, PCI DSS,
  and GLBA requirements.
- **FlexVPN** is Cisco's implementation of the IKEv2 standard — **IKEv2 only** —
  unifying site-to-site, remote access, hub-and-spoke, and spoke-to-spoke in
  one modular framework built on virtual access interfaces.
- **Remote-access VPN** on IOS-XE is FlexVPN (IKEv2 only); ASA 5500-X and Cisco
  Secure Firewalls also provide it via TLS/DTLS.

### LISP
- **Locator/ID Separation Protocol** — a routing architecture plus a data plane
  and control plane protocol, created to address the routing scalability
  problems of the **default-free zone** (the Internet routing table):
  **aggregation issues** (many non-aggregable provider-independent routes),
  **traffic engineering** (injecting more-specifics makes the table worse),
  **multihoming** (proper multihoming needs the full table — 900,000+ IPv4
  routes — requiring routers too expensive for small sites), and **routing
  instability / route churn** (heavy CPU and memory consumption).
- The core idea: traditional routing makes an endpoint's IP represent both its
  **identity and its location**, so moving changes the IP. LISP splits these
  into **EIDs** (identity) and **RLOCs** (location), so an endpoint can roam
  and only its RLOC changes — **the EID stays the same.**
- The **control plane works like DNS**: DNS resolves a name to an IP, LISP
  resolves an EID to an RLOC by sending map requests to the map resolver. It is
  a **pull model** — only the mapping actually needed is requested — as opposed
  to the **push model** of BGP and OSPF, which flood every route including
  unnecessary ones.
- The **data plane** is **IP-in-IP/UDP**: the ITR wraps the original packet
  (preserved as the *inner header*) in an outer IP/UDP header addressed with
  RLOCs, with a **LISP shim header** in between carrying forwarding-plane
  information such as network virtualization.
- The outer **UDP source port is tactically selected by the ITR** so that
  traffic between the same two LISP sites does not always hash to the same ECMP
  path — it exists to **prevent polarization** and improve load sharing.
- The **Instance ID** is a **24-bit** field providing device- and path-level
  network virtualization — it enables VRFs and VPNs much as VPN IDs do for
  MPLS, preventing address duplication within a site and acting as a secure
  boundary between organizations.
- Beyond the DFZ problem, LISP is deployed in data centers, campus networks,
  branches, next-gen WANs, and SP cores, and underpins use cases such as
  mobility, network virtualization, IoT, IPv4-to-IPv6 transition, traffic
  engineering, and **Cisco SD-Access**.
- EID prefixes must be explicitly configured on the ETR. **Subnets attached to
  an ETR that are not configured as EID prefixes are forwarded natively** using
  traditional routing — a silent and very common "why isn't this site working"
  cause.

### VXLAN
- Server virtualization broke traditional Layer 2: the **12-bit VLAN ID yields
  only 4000 VLANs**, MAC tables must hold hundreds of thousands of VM and
  container addresses, **STP blocks links** (unacceptable at scale), **ECMP is
  not supported**, and **host mobility is difficult**.
- VXLAN is an **overlay data plane encapsulation scheme** extending Layer 2 and
  Layer 3 overlays over a Layer 3 underlay using **MAC-in-IP/UDP** tunneling.
  Each overlay is a **VXLAN segment**.
- The **VNI (VXLAN network identifier)** is **24-bit**, allowing up to **16
  million** segments to coexist in one infrastructure. It sits in the VXLAN
  shim header wrapping the original inner MAC frame and provides segmentation
  for both Layer 2 and Layer 3 traffic.
- **VTEPs (virtual tunnel endpoints)** originate and terminate VXLAN tunnels
  and map Layer 2/3 packets to the VNI. Each has two interfaces: **local LAN
  interfaces** (bridging between local hosts) and an **IP interface**
  (core-facing; its address identifies the VTEP and is used for encapsulation
  and de-encapsulation).
- A **VXLAN gateway** is a VTEP that joins a VXLAN segment and a classic VLAN
  segment into one Layer 2 domain, for devices that cannot do VXLAN.
- **VXLAN defines a data plane but deliberately no control plane.** Cisco
  devices support four combinations: **multicast underlay**, **static unicast
  tunnels**, **MP-BGP EVPN**, and **LISP**. MP-BGP EVPN and multicast dominate
  data center and private cloud; **for campus, VXLAN with a LISP control plane
  is preferred** — that combination is what **Cisco SD-Access** is.
- **The key distinction from LISP:** LISP encapsulation is IP-in-IP/UDP only,
  so it supports **Layer 3 overlays only**. VXLAN encapsulates the original
  Ethernet header (MAC-in-IP), so it supports **Layer 2 and Layer 3 overlays**.
- Historical note that explains the family resemblance: the VXLAN spec
  originated from a **Layer 2 LISP** specification (`draft-smith-lisp-layer2-00`)
  and did not port over some of its fields, which were left reserved.

## Procedure

**IKEv1 phase 1 — main mode (MM1–MM6):**
1. **MM1:** the initiator sends its SA proposals — hash, encryption,
   authentication method, DH group, and lifetime. Every parameter must match
   the peer exactly except the lifetime.
2. **MM2:** the responder returns the SA proposal that it matched.
3. **MM3:** the initiator starts the Diffie-Hellman key exchange, based on the
   DH group the responder matched in the proposal.
4. **MM4:** the responder sends its own key. Encryption keys are now shared and
   encryption is established for the ISAKMP SA.
5. **MM5:** the initiator starts authentication by sending the peer router its
   IP address.
6. **MM6:** the responder sends back a similar packet and authenticates the
   session. The ISAKMP SA is established.

**IKEv1 phase 1 — aggressive mode (AM1–AM3):**
1. **AM1:** the initiator sends everything contained in MM1 through MM3, plus
   MM5, in a single message.
2. **AM2:** the responder sends everything contained in MM2, MM4, and MM6.
3. **AM3:** the initiator sends the authentication contained in MM5.

**IKEv1 phase 2 — quick mode (QM1–QM3):**
1. **QM1:** the initiator (which can be either peer) can start multiple IPsec
   SAs in a single exchange message, carrying the encryption and integrity
   algorithms agreed during phase 1 plus a definition of what traffic is to be
   encrypted or secured.
2. **QM2:** the responder replies with matching IPsec parameters.
3. **QM3:** after this message, two **unidirectional** IPsec SAs exist between
   the peers — one on each.

**IKEv2 exchange sequence:**
1. **IKE_SA_INIT** (one request/response pair) negotiates cryptographic
   algorithms, exchanges nonces, and performs the Diffie-Hellman exchange.
2. **IKE_AUTH** (one request/response pair) authenticates the previous
   messages, exchanges identities and certificates, then establishes the IKE SA
   and a child SA (the IPsec SA).
3. Four messages total bring up the bidirectional IKE SA and the unidirectional
   IPsec SAs.
4. Any additional IPsec SAs use a **CREATE_CHILD_SA** exchange — two messages.

**GRE over IPsec using crypto maps (legacy, but still widely deployed):**
1. Configure a **crypto ACL** to classify VPN traffic:
   `ip access-list extended acl_name`, then
   `permit gre host <tunnel-source-IP> host <tunnel-destination-IP>`. This
   matches all traffic passing through the GRE tunnel.
2. Configure an **ISAKMP policy** for the IKE SA with
   `crypto isakmp policy <priority>` (priority 1 is highest), setting
   `encryption`, `hash`, `authentication`, and `group`.
3. Configure the **pre-shared key** with
   `crypto isakmp key <keystring> address <peer-address> [mask]`. The keystring
   must match on both peers; `0.0.0.0 0.0.0.0` matches any peer.
4. Create a **transform set** with `crypto ipsec transform-set <name>
   <transform1> [transform2 [transform3]]`, then set `mode [tunnel|transport]`.
5. Create the **crypto map** with `crypto map <map-name> <seq-num>
   ipsec-isakmp`, then inside it `match address <acl-name>`,
   `set peer <ip>`, and `set transform-set <name>` (list multiple transform
   sets in priority order, highest first; `set peer` may be repeated).
6. Apply the crypto map to the **outside physical interface — not the tunnel
   interface** — with `crypto map <map-name>`.

**GRE over IPsec using IPsec profiles (preferred):**
1. Configure an **ISAKMP policy** with `crypto isakmp policy <priority>` and
   its `encryption` / `hash` / `authentication` / `group` settings.
2. Configure the **pre-shared key** with
   `crypto isakmp key <keystring> address <peer-address> [mask]`.
3. Create a **transform set** and set `mode [tunnel|transport]`. **Choose
   transport mode to avoid double encapsulation from GRE and IPsec.**
4. Create an **IPsec profile** with `crypto ipsec profile <name>` and specify
   `set transform-set <name>` (priority order, highest first).
5. Apply the profile to the **tunnel interface** with
   `tunnel protection ipsec profile <name>`.

*To convert this to VTI over IPsec:* remove the crypto map from the physical
interface, change the transform set to `mode tunnel`, and add
`tunnel mode ipsec ipv4` to the tunnel interface.

**LISP map registration and notification:**
1. The **ETR** sends a **map register** message to the **MS** to register its
   associated EID prefix, including the RLOC IP the MS should use when
   forwarding map requests (reformatted as encapsulated map requests) received
   through the mapping database system. An ETR responds to map requests itself
   by default, but in the map register it may ask the MS to answer on its
   behalf by setting the **proxy map reply flag (P-bit)**.
2. The **MS** sends a **map notify** message back to the ETR confirming the map
   register was received and processed. Map notify uses **UDP port 4342 for
   both source and destination**.

**LISP map request and reply:**
1. The endpoint in LISP Site 1 sends a **DNS request** for the remote host; DNS
   replies with the destination **EID**. The host sends packets to its default
   gateway — here, the ITR. (If the host were not directly connected to the
   ITR, the packets would traverse the LISP site as normal IP packets using
   traditional routing until they reached the ITR.)
2. The **ITR** performs an **FIB lookup** and evaluates two forwarding rules:
   - Did the packet match a **default route** because no route was found for
     the destination? If no → forward natively using the matched route.
   - Is the **source IP a registered EID prefix** in the local map cache? If
     no → forward natively.
3. The ITR sends an **encapsulated map request** to the **MR** for the
   destination EID. Map requests use **UDP destination port 4342**; the source
   port is chosen by the ITR.
4. Because MR and MS functionality is on the same device, the **MS mapping
   database system forwards the map request to the authoritative ETR.** If MR
   and MS were separate devices, the MR would forward the encapsulated map
   request to the MS as received from the ITR, and the MS would forward it on
   to the ETR.
5. The **ETR** sends the ITR a **map reply** containing the EID-to-RLOC
   mapping. The map reply uses **UDP source port 4342** and, as its
   destination, the port the ITR chose in the map request.
6. The ITR installs the EID-to-RLOC mapping in its **local map cache** and
   programs the **FIB**. It is now ready to forward LISP traffic.

**LISP data path:**
1. The ITR receives a packet from an EID host destined to a remote EID host.
2. The ITR performs an FIB lookup, finds a match, encapsulates the EID packet
   and adds an outer header with **the ITR's RLOC as source and the ETR's RLOC
   as destination**. It forwards using **UDP destination port 4341** with a
   tactically selected source port in case ECMP load balancing is needed.
3. The ETR receives the encapsulated packet, de-encapsulates it, and forwards
   it to the destination host.

**LISP proxy ETR (PETR) — reaching a non-LISP destination:**
1. A LISP-site host does a DNS lookup for an external destination and starts
   forwarding packets to the ITR with that destination IP.
2. The ITR sends a **map request** to the MR for that address.
3. Because the destination is not a registered EID, the mapping database system
   responds with a **negative map reply**, which includes a **calculated
   non-LISP prefix** — the shortest prefix matching the requested destination
   that does *not* match any LISP EID — for the ITR to add to its map cache
   and FIB.
4. The ITR starts sending **LISP-encapsulated packets to the PETR**. For this
   to work the ITR must be configured to send traffic to the PETR's RLOC for
   destinations that receive a negative map reply.
5. The **PETR de-encapsulates** the traffic and sends it to the destination.
   A PETR registers no EID addresses with the mapping database system.

**LISP proxy ITR (PITR) — reaching a LISP EID from a non-LISP site:**
1. Traffic from a non-LISP site is received by the **PITR** with a destination
   IP inside a LISP site.
2. The PITR sends a **map request** to the MR for that EID.
3. The mapping database system forwards the map request to the ETR.
4. The ETR sends a **map reply** to the PITR with the EID-to-RLOC mapping.
5. The PITR **LISP-encapsulates** the packets and forwards them to the ETR.
6. The ETR receives the encapsulated packets, de-encapsulates them, and sends
   them to the destination host.

*Key difference:* a **PITR sends map requests even when the source is not an
EID**, because the traffic originates outside LISP. An **ITR first checks
whether the source is a registered EID** in its local map cache — if it is
not, the traffic is not eligible for LISP encapsulation and traditional
forwarding rules apply.

## Reference Tables

**IPsec security services**

| Service | Provided by | Examples |
|---|---|---|
| Confidentiality | Encryption algorithms | DES, 3DES, AES |
| Data integrity | Hashing algorithms | MD5, SHA |
| Peer authentication | PSK or digital signatures | Pre-shared key, RSA signatures/digital certificates |
| Anti-replay | Sequence numbers | Sequence numbering in the SA |

**AH vs ESP**

| | AH | ESP |
|---|---|---|
| IP protocol number | 51 | 50 |
| Encryption | **No** | Yes |
| Integrity / authentication | Yes | Yes |
| Works through NAT | **No** — it authenticates the outer IP header | Yes (with NAT-T) |
| Practical use | Rare | The one actually deployed |

**Encapsulation modes compared**

| Mode | Outer IP protocol | What is encrypted | What is used for routing |
|---|---|---|---|
| GRE (no IPsec) | 47 (GRE) | Nothing | GRE IP header |
| GRE over IPsec, transport mode | 50 (ESP) | GRE payload only — **not** the GRE IP header | GRE IP header |
| GRE over IPsec, tunnel mode (*IPsec over GRE*) | 50 (ESP) | The entire GRE packet; a new IPsec IP header is added | IPsec IP header |
| IPsec tunnel mode with VTI | 50 (ESP) | The original IP packet; no GRE header at all | IPsec IP header |

**IKEv1 vs IKEv2**

| | IKEv1 | IKEv2 |
|---|---|---|
| Exchange modes | Main mode, aggressive mode, quick mode | IKE_SA_INIT, IKE_AUTH, CREATE_CHILD_SA |
| Minimum messages to establish IPsec SAs | Nine (main mode); six (aggressive mode) | **Four** |
| Authentication methods | PSK, RSA-SIG, public key. **Both peers must use the same method** | PSK, RSA-SIG, ECDSA-SIG, EAP. **Asymmetric authentication supported** — method specified during IKE_AUTH |
| Next-Generation Encryption (NGE) | Not supported | AES-GCM, SHA-256/384/512, HMAC-SHA-256, ECDH-384, ECDSA-384 |
| Attack protection | MitM, eavesdropping | MitM, eavesdropping, **anti-DoS** |

**Cisco IPsec VPN solutions**

| Feature | Site-to-Site IPsec | DMVPN | GET VPN | FlexVPN | Remote-Access VPN |
|---|---|---|---|---|---|
| Product interoperability | **Multivendor** | Cisco only | Cisco only | Cisco only | Cisco only |
| Key exchange | IKEv1 and IKEv2 | IKEv1 and IKEv2 (both optional) | IKEv1 and IKEv2 | **IKEv2 only** | TLS/DTLS and IKEv2 |
| Scale | Low | Thousands hub-and-spoke; hundreds partially meshed spoke-to-spoke | Thousands | Thousands | Thousands |
| Topology | Hub-and-spoke; small-scale meshing | Hub-and-spoke; on-demand spoke-to-spoke partial mesh, torn down when idle | Hub-and-spoke; any-to-any | Hub-and-spoke; any-to-any; remote access | Remote access |
| Routing | Not supported | Supported | Supported | Supported | Not supported |
| QoS | Supported | Supported | Supported | Native support | Supported |
| Multicast | Not supported | Tunneled | Natively supported across MPLS and private IP | Tunneled | Not supported |
| Non-IP protocols | Not supported | Not supported | Not supported | Not supported | Not supported |
| Private IP addressing | Supported | Supported | Requires GRE or DMVPN to carry private addresses across the Internet | Supported | Supported |
| High availability | Stateless failover | Routing | Routing | Routing; IKEv2-based dynamic route distribution and server clustering | Not supported |
| Encapsulation | Tunneled IPsec | Tunneled IPsec | **Tunnel-less IPsec** | Tunneled IPsec | Tunneled IPsec/TLS |
| Transport network | Any | Any | **Private WAN / MPLS** | Any | Any |

**Diffie-Hellman groups**

| Group | Size / type | Status per the ENCOR OCG |
|---|---|---|
| 1 | 768-bit DH | No longer recommended (**this is the IOS default**) |
| 2 | 1024-bit DH | No longer recommended |
| 5 | 1536-bit DH | No longer recommended |
| 14 | 2048-bit DH | Recommended floor |
| 15 | 3072-bit DH | Recommended |
| 16 | 4096-bit DH | Recommended |
| 19 | 256-bit ECDH | Recommended |
| 20 | 384-bit ECDH | Recommended |
| 24 | 2048-bit DH/DSA | Recommended |

**LISP terminology**

| Term | Meaning |
|---|---|
| **EID** (endpoint identifier) | The IP address of an endpoint within a LISP site — ordinary IPv4/IPv6 addresses that operate the same way they do today |
| **LISP site** | The name of a site where LISP routers and EIDs reside |
| **ITR** (ingress tunnel router) | LISP-encapsulates packets from EIDs destined outside the LISP site |
| **ETR** (egress tunnel router) | De-encapsulates LISP packets from outside the site destined to EIDs within it |
| **xTR** (tunnel router) | Performs both ITR and ETR functions — most routers |
| **PITR** (proxy ITR) | Like an ITR, but for non-LISP sites sending traffic to EID destinations |
| **PETR** (proxy ETR) | Like an ETR, but for EIDs sending traffic to destinations at non-LISP sites |
| **PxTR** (proxy xTR) | Performs both PITR and PETR functions |
| **RLOC** (routing locator) | The IPv4/IPv6 address of an ETR that is Internet facing or network core facing |
| **MS** (map server) | Learns EID-to-prefix mappings from an ETR and stores them in a local EID-to-RLOC mapping database |
| **MR** (map resolver) | Receives encapsulated map requests from an ITR and finds the ETR to answer them by consulting the map server |
| **MS/MR** | One device performing both functions |

**Overlay port and protocol numbers**

| Protocol | Port / number | Used for |
|---|---|---|
| GRE | IP protocol **47** | GRE encapsulation |
| ESP | IP protocol **50** | IPsec encryption (all GRE-over-IPsec and VTI modes) |
| AH | IP protocol **51** | IPsec integrity without encryption |
| LISP data plane | UDP **4341** | LISP-encapsulated data traffic |
| LISP control plane | UDP **4342** | Map request, map reply, map register, map notify |
| VXLAN | UDP **4789** (IANA); **8472** on some prestandard/Linux implementations | VXLAN-encapsulated traffic |

**LISP vs VXLAN**

| | LISP | VXLAN |
|---|---|---|
| Encapsulation | IP-in-IP/UDP | **MAC-in-IP/UDP** |
| Overlays supported | **Layer 3 only** | **Layer 2 and Layer 3** |
| Identifier field | Instance ID (24-bit) | VNI (24-bit) — up to 16 million segments |
| Control plane | Its own (map server / map resolver) | **Not defined by the standard** — multicast, static unicast, MP-BGP EVPN, or LISP |
| Preserves original Ethernet header | No | Yes |

**Command reference**

| Task | Command |
|---|---|
| Create a GRE tunnel interface | `interface tunnel <tunnel-number>` |
| Enable keepalives on a GRE tunnel interface | `keepalive [seconds [retries]]` |
| Create an ISAKMP policy | `crypto isakmp policy <priority>` |
| Create an IPsec transform set | `crypto ipsec transform-set <name> <transform1> [transform2 [transform3]]` |
| Create a crypto map for IPsec | `crypto map <map-name> <seq-num> [ipsec-isakmp]` |
| Apply a crypto map to an outside interface | `crypto map <map-name>` |
| Create an IPsec profile for tunnel interfaces | `crypto ipsec profile <ipsec-profile-name>` |
| Apply an IPsec profile to a tunnel interface | `tunnel protection ipsec profile <profile-name>` |
| Turn a GRE tunnel into a VTI tunnel | `tunnel mode ipsec {ipv4 \| ipv6}` |
| Turn a VTI tunnel into a GRE tunnel | `tunnel mode gre {ip \| ipv6}` |
| Display information about ISAKMP SAs | `show crypto isakmp sa` |
| Display detailed information about IPsec SAs | `show crypto ipsec sa` |

## Config Patterns

```ios-xe
! ===== Plain GRE tunnel (no security — pair with IPsec for real traffic) =====
interface Tunnel100
 bandwidth 4000
 ip address 192.168.100.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source GigabitEthernet0/1
 tunnel destination 100.64.2.2
 keepalive 5 3

! ===== GRE over IPsec — CRYPTO MAP method (legacy) =====
crypto isakmp policy 10
 authentication pre-share
 hash sha256
 encryption aes
 group 14
!
crypto isakmp key <keystring> address 100.64.2.2
!
crypto ipsec transform-set AES_SHA esp-aes esp-sha-hmac
 mode transport
!
! Crypto ACL matches the GRE traffic between the tunnel endpoints
ip access-list extended GRE_IPSEC_VPN
 10 permit gre host 100.64.1.1 host 100.64.2.2
!
crypto map VPN 10 ipsec-isakmp
 match address GRE_IPSEC_VPN
 set transform-set AES_SHA
 set peer 100.64.2.2
!
! The crypto map goes on the PHYSICAL interface, not the tunnel
interface GigabitEthernet0/1
 ip address 100.64.1.1 255.255.255.252
 crypto map VPN
!
interface Tunnel100
 bandwidth 4000
 ip address 192.168.100.1 255.255.255.0
 ip mtu 1400
 tunnel source GigabitEthernet0/1
 tunnel destination 100.64.2.2
!
router ospf 1
 router-id 1.1.1.1
 network 10.1.1.1 0.0.0.0 area 1
 network 192.168.100.1 0.0.0.0 area 0

! ===== GRE over IPsec — IPSEC PROFILE method (preferred) =====
crypto isakmp policy 10
 authentication pre-share
 hash sha256
 encryption aes
 group 14
!
crypto isakmp key <keystring> address 100.64.1.1
!
crypto ipsec transform-set AES_SHA esp-aes esp-sha-hmac
 mode transport
!
crypto ipsec profile IPSEC_PROFILE
 set transform-set AES_SHA
!
interface GigabitEthernet0/1
 ip address 100.64.2.2 255.255.255.252
!
! The profile goes on the TUNNEL interface
interface Tunnel100
 bandwidth 4000
 ip address 192.168.100.2 255.255.255.0
 ip mtu 1400
 tunnel source GigabitEthernet0/1
 tunnel destination 100.64.1.1
 tunnel protection ipsec profile IPSEC_PROFILE

! ===== Converting GRE over IPsec to VTI over IPsec =====
! 1. VTI uses IPsec profiles, so remove any crypto map from the interface
interface GigabitEthernet0/1
 no crypto map VPN
!
! 2. Change transport mode to tunnel
crypto ipsec transform-set AES_SHA esp-aes esp-sha-hmac
 mode tunnel
!
crypto ipsec profile IPSEC_PROFILE
 set transform-set AES_SHA
!
! 3. Enable VTI on the tunnel interface and apply the profile
interface Tunnel100
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC_PROFILE
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Never use DES or RC4; phase out 3DES; use AES — preferring AES-GCM | DES and RC4 are classified "Avoid" and 3DES "Legacy"; **DES is the IOS default encryption**, so an unspecified `crypto isakmp policy` silently lands on the weakest option | Interoperating with a legacy peer or vendor that genuinely cannot do AES — a documented, time-boxed exception, not a standing state | [Cisco Next Generation Encryption](https://sec.cloudapps.cisco.com/security/center/resources/next_generation_cryptography); ENCOR OCG Ch.16 |
| Never use MD5 for integrity; phase out SHA-1; use SHA-256 or better | MD5 is classified "Avoid" and SHA-1 "Legacy"; the ENCOR OCG states plainly that "the MD5 hash is no longer recommended" | Legacy peer interoperability under the same time-boxed exception | [Cisco Next Generation Encryption](https://sec.cloudapps.cisco.com/security/center/resources/next_generation_cryptography); ENCOR OCG Ch.16 |
| Avoid DH groups 1, 2, and 5. Group 14 (DH-2048) is the floor; groups 19/20 (ECDH) or 15 (DH-3072) are the target | **Group 1 is the IOS default**, so an unset `group` command lands on 768-bit DH. Note the two sources differ in strictness: the OCG says "14 and higher"; Cisco NGE rates DH-2048 only *acceptable* and recommends ECDH 19/20 | Peer or platform without ECDH support — group 14 remains acceptable; hardware without crypto offload where ECDH cost is prohibitive | [Cisco Next Generation Encryption](https://sec.cloudapps.cisco.com/security/center/resources/next_generation_cryptography); ENCOR OCG Ch.16 |
| Use IPsec profiles with `tunnel protection`, not crypto maps, for tunnel protection | Crypto maps cannot natively support MPLS, become overly complex, **crypto ACLs are commonly misconfigured**, and crypto ACL entries can consume excessive TCAM | An existing brownfield deployment already built on crypto maps where migration is scheduled separately; a platform or IOS version that does not support profiles | ENCOR OCG Ch.16, "Site-to-Site GRE over IPsec" |
| Use IPsec **transport** mode for GRE over IPsec | GRE has already added a routable IP header; tunnel mode adds a second one, causing double encapsulation and wasted overhead | Where a device or intermediate network requires the original GRE header to be hidden, tunnel mode is the deliberate choice; VTI configurations legitimately use tunnel mode | ENCOR OCG Ch.16, GRE over IPsec with IPsec profiles, Step 3 |
| Enable Perfect Forward Secrecy on phase 2 | PFS derives session keys independently of any previous key, so **a compromised key does not compromise future keys** | Additional DH exchanges cost CPU cycles — a constrained platform or very high tunnel count may justify omitting it, as a logged risk acceptance | ENCOR OCG Ch.16, "Perfect Forward Secrecy" |
| Set tunnel `ip mtu` to at least 1400 and `ip tcp adjust-mss` 40 bytes below it | GRE adds 24 bytes, forcing fragmentation once packets exceed the outbound MTU. Fragmentation degrades performance, loads the router, and costs a full retransmit if any fragment is lost | A path with a verified larger MTU (jumbo-frame-capable underlay) where the values can be raised accordingly | [Resolve IPv4 Fragmentation, MTU, MSS, and PMTUD Issues with GRE and IPsec](https://www.cisco.com/c/en/us/support/docs/ip/generic-routing-encapsulation-gre/25885-pmtud-ipfrag.html) |
| Never carry sensitive traffic over bare GRE | GRE provides no encryption, no authentication, and no security services — it is "highly susceptible to attacks" | A fully trusted underlay (a private lab, or a link already encrypted at another layer such as MACsec) — worth confirming as intentional | ENCOR OCG Ch.16, "Site-to-Site IPsec Configuration" |

*A deviation from this table is a question for the network's operator — "is
this intentional here?" — never automatically a finding.*

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show interface tunnel100 \| include Tunnel protocol` | `Tunnel protocol/transport GRE/IP` for a GRE tunnel; `IPSEC/IP` for a VTI. This is the fastest way to confirm which kind of tunnel you actually have |
| `show interface tunnel100` | Line protocol state, tunnel source/destination, and the tunnel's `ip mtu`. A tunnel that is up/down usually means the destination is unreachable via the underlay |
| `show ip ospf neighbor` | A FULL adjacency across the tunnel interface proves the overlay is passing traffic, not just that the tunnel is up |
| `show ip route ospf` | Remote prefixes learned `via <tunnel-peer>, Tunnel100` — confirms routing is actually using the overlay |
| `show crypto isakmp sa` | State **`QM_IDLE`** with status **`ACTIVE`** is the healthy phase 1 result — the SA remains authenticated with its peer and can be used for subsequent quick mode exchanges. Anything else (MM_KEY_EXCH, MM_NO_STATE) means phase 1 is failing |
| `show crypto ipsec sa` | `#pkts encaps/encrypt/digest` (outgoing) and `#pkts decaps/decrypt/verify` (incoming). **Encaps incrementing but decaps flat means return traffic is not arriving** — look at the far end or the path back |
| `show crypto ipsec sa` (continued) | `local crypto endpt.` / `remote crypto endpt.` addresses, the inbound and outbound `esp sas` with their SPI and `transform:` line, `in use settings ={Transport, }` or `{Tunnel, }`, and `Status: ACTIVE` for both directions |
| `show crypto session` | Condensed per-peer session state — quicker triage than reading both SA outputs |
| `show crypto map` | Which ACL, peer, and transform set a crypto map actually references — the fastest way to catch a misconfigured crypto ACL |
| `debug crypto isakmp` / `debug crypto ipsec` | Phase 1 vs phase 2 failure isolation. Mismatched proposals appear here as no-matching-policy messages |
| `show ip cef <destination>` | Whether traffic is being resolved out the tunnel or natively — key for LISP, where a lookup miss means native forwarding |

## Intent Questions
- **Overlay purpose:** what is this tunnel for — reaching private address space
  across an untrusted transit, carrying a routing protocol or multicast that
  IPsec alone cannot, or segmenting one physical fabric into many logical ones?
  The answer determines whether GRE, GRE over IPsec, VTI, or a fabric overlay
  is the right tool.
- **Security expectation:** is this traffic supposed to be encrypted? Bare GRE
  looks identical to GRE over IPsec from a routing perspective — a tunnel that
  is "working" says nothing about whether anything is protected.
- **Underlay reachability:** how are the tunnel endpoints supposed to reach
  each other, and is that path independent of the tunnel itself? Any dependency
  of the underlay on the overlay is a recursive routing failure waiting to fire.
- **Scale and topology intent:** is this meant to stay point-to-point, or grow
  into hub-and-spoke or any-to-any? Static site-to-site tunnels that outgrow
  their design are what DMVPN, FlexVPN, and fabric overlays exist to replace.
- **For LISP/VXLAN:** which prefixes are supposed to be EIDs, and which are
  meant to be forwarded natively? A subnet not registered as an EID prefix is
  silently routed traditionally rather than encapsulated.

## Troubleshooting Checklist
0. **State intent vs. observed:** answer the Intent Questions above, then write
   the one-line symptom ("the tunnel should be encrypting branch traffic, but
   `show crypto ipsec sa` shows zero encaps"; "OSPF should be adjacent over
   Tunnel100, but it is stuck in INIT"). Do this before any show command.
1. **Underlay first — can the endpoints reach each other natively?** Ping the
   tunnel destination from the tunnel source address using the underlay. If the
   endpoints cannot reach each other outside the tunnel, nothing above matters.
   **An overlay problem is an underlay problem until proven otherwise.**
2. **Is the tunnel interface up/up?** `show interface tunnel<N>`. Up/down
   almost always means the tunnel destination has no route or is unreachable.
   Verify `tunnel source` and `tunnel destination` are correct and that the
   source interface itself is up.
3. **Check for recursive routing.** `%TUN-5-RECURDOWN` in the log, or a tunnel
   that flaps up and down on a cycle, means the route to the tunnel destination
   is being learned *through the tunnel*. Confirm the tunnel endpoints are not
   advertised into the overlay routing protocol.
4. **MTU and fragmentation.** If small pings succeed and large transfers or
   TLS handshakes hang, this is MTU. GRE costs 24 bytes and IPsec more on top.
   Confirm `ip mtu` and `ip tcp adjust-mss` are set on the tunnel interface.
5. **Phase 1 — is the ISAKMP SA up?** `show crypto isakmp sa`. `QM_IDLE` /
   `ACTIVE` is healthy. If it is absent or stuck, the proposals do not match:
   compare `encryption`, `hash`, `authentication`, and `group` on both peers —
   **every parameter must match exactly except the lifetime.**
6. **Phase 1 — is the pre-shared key right?** The keystring must match on both
   peers, and `crypto isakmp key … address <peer>` must name the correct peer
   address. Authentication failures show in `debug crypto isakmp`.
7. **Phase 2 — are the IPsec SAs built?** `show crypto ipsec sa`. Confirm both
   an inbound and an outbound ESP SA exist, both `ACTIVE`, and that the
   `in use settings` mode (Transport vs Tunnel) matches on both ends.
8. **Is traffic actually being protected?** Check `#pkts encaps` and
   `#pkts decaps`. Encaps incrementing with decaps flat means the return path
   is broken. Both flat means traffic is not being classified into the tunnel
   at all — go to the crypto ACL or the tunnel routing.
9. **Crypto map specifics (if used):** confirm the crypto map is on the
   **outside physical interface**, not the tunnel, and that the crypto ACL
   matches `gre host <source> host <destination>` with the correct addresses.
   Misconfigured crypto ACLs are the named, most common failure of this method.
10. **IPsec profile specifics (if used):** confirm `tunnel protection ipsec
    profile` is on the **tunnel interface**, and that the transform set the
    profile references exists and is in the intended mode.
11. **Version and mode incompatibilities:** IKEv1 and IKEv2 cannot interoperate.
    Transport and tunnel mode must match. A VTI on one side and plain GRE on
    the other will not form — check
    `show interface tunnel<N> | include Tunnel protocol` on both ends.
12. **For LISP:** verify the EID prefix is actually registered on the ETR — an
    attached subnet not configured as an EID prefix is forwarded natively and
    never encapsulated. Then confirm the ITR's map cache holds the EID-to-RLOC
    mapping, and that RLOCs are reachable in the underlay.
13. **For VXLAN:** confirm both VTEPs agree on the UDP port (**4789** per IANA,
    but **8472** on some prestandard and Linux implementations) — a port
    mismatch in a multivendor deployment silently blackholes the overlay. Then
    confirm the VNI matches on both ends.

## Common Pitfalls
- **Treating a tunnel that is "up" as a tunnel that is secure.** A GRE tunnel
  with a broken or absent IPsec configuration passes traffic perfectly and
  looks healthy in the routing table — in the clear. Only
  `show crypto ipsec sa` with incrementing encaps/decaps proves encryption.
- **Advertising the tunnel endpoints into the routing protocol running over
  the tunnel.** This is recursive routing and it makes the tunnel flap forever.
- **Leaving `crypto isakmp policy` parameters unset.** The defaults are the
  worst available options: **DES encryption and DH group 1 (768-bit)**. An
  unspecified policy is not a neutral policy.
- **Putting the crypto map on the tunnel interface.** It belongs on the outside
  physical interface. IPsec profiles are the opposite — those go on the tunnel.
- **Using tunnel mode for GRE over IPsec**, producing double encapsulation and
  needless overhead when transport mode is what the design calls for.
- **Assuming AH will work through NAT.** It authenticates the outer IP header,
  so any NAT in the path breaks it. Use ESP.
- **Assuming IKEv1 and IKEv2 will negotiate down to each other.** Their SA
  exchanges are completely different and they are incompatible — one side on
  FlexVPN (IKEv2 only) and the other on a legacy IKEv1 config will never form.
- **Forgetting that only the lifetime may differ in a phase 1 proposal.**
  Engineers often assume the peers will negotiate to a common denominator;
  every other parameter must match exactly.
- **Ignoring MTU until it manifests as an application problem.** Ping works,
  SSH works, but file transfers and TLS handshakes hang — that is fragmentation,
  not a "flaky tunnel."
- **Expecting site-to-site IPsec to carry a routing protocol or multicast.** It
  supports neither; that is precisely why GRE (or DMVPN/FlexVPN) is paired
  with it.
- **Assuming an ETR encapsulates everything attached to it.** Subnets not
  explicitly configured as EID prefixes are forwarded natively with traditional
  routing, silently.
- **Confusing ITR and PITR behavior.** An ITR checks whether the *source* is a
  registered EID before sending a map request; a PITR does not, because its
  traffic originates outside LISP.
- **Assuming LISP can carry Layer 2.** LISP is IP-in-IP/UDP — Layer 3 overlays
  only. Extending a Layer 2 segment requires VXLAN's MAC-in-IP encapsulation.
- **Mismatched VXLAN UDP ports in a multivendor deployment.** IANA assigned
  4789, but prestandard and some Linux implementations default to 8472.
