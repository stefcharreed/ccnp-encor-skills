---
name: ccnp-fabric-technologies
description: >
  Use this skill when designing, reviewing, or troubleshooting Cisco fabric overlay
  technologies — SD-Access for campus and branch, SD-WAN (Viptela) for the WAN.
  Invoke when the user asks about: fabric network, overlay network, virtual network, VN,
  underlay network, campus fabric, SD-Access, SDA, Software-Defined Access,
  Cisco DNA, Cisco DNA Center, DNAC, intent-based networking, LAN Automation,
  manual underlay, automated underlay, IS-IS underlay, extension node,
  SD-Access architecture layers, physical layer, network layer, controller layer,
  management layer, NCP, Network Control Platform, NDP, Network Data Platform,
  Cisco ISE, Identity Services Engine, LISP control plane, EID, RLOC, map server,
  map resolver, MS/MR, xTR, PxTR, proxy tunnel router, VXLAN data plane, VXLAN-GPO,
  VXLAN Group Policy Option, Group Policy ID, G bit, D bit, A bit, Don't Learn Bit,
  Policy Applied Bit, VTEP, MAC-in-IP, Cisco TrustSec, SGT, Scalable Group Tag,
  Security Group Tag, SGACL, SXP, SGT Exchange Protocol, fabric roles,
  fabric edge node, fabric control plane node, fabric border node, internal border,
  default border, anywhere border, fabric WLAN controller, fabric AP, fabric wireless,
  intermediate node, host pool, scalable group, anycast gateway, instance ID, VNID,
  802.1x, MAB, MAC Authentication Bypass, WebAuth, design policy provision assurance,
  SD-WAN, Cisco SD-WAN, Viptela, Meraki SD-WAN, vManage, vSmart, vBond, vAnalytics,
  vEdge, cEdge, SD-WAN edge device, OMP, Overlay Management Protocol, DTLS tunnel,
  STUN, NAT detection, transport independent, single pane of glass,
  Cloud OnRamp, CoR, CoR for SaaS, CoR for IaaS, DIA site, gateway site, client site,
  vQoE score, application-aware routing, AAR, BFD probes, brownout, soft failure,
  centralized policy, local policy, topology policy, VPN membership policy,
  traffic data policy, service chaining, Direct Connect, ExpressRoute.
---

## Purpose
Covers Cisco's two overlay fabric solutions — SD-Access for the campus and branch, and
SD-WAN for the WAN — by separating *who runs each plane* (control, data, policy,
management) from *what device role plays it*, so that a fabric problem can be localized
to a plane and a role instead of guessed at.

## Key Concepts

**What a fabric is**
- A **fabric network** is an **overlay network** (a *virtual network*, **VN**) built over
  an **underlay network** (the physical network) using overlay tunneling technologies
  such as **VXLAN**.
- Fabrics exist to overcome shortcomings of traditional physical networks by enabling
  **host mobility, network automation, network virtualization, and segmentation**. They
  are more manageable, flexible, secure (by means of encryption), and scalable than
  traditional networks.
- Two next-generation overlay fabrics: **SD-Access** for campus networks, **SD-WAN** for
  WAN networks.
- **SD-Access was designed for enterprise campus and branch environments only** — not for
  data center, service provider, or WAN environments.

**Why SD-Access exists**
- Traditional campus networks are configured by hand: changes are slow, misconfigurations
  cause service disruptions, and the problem compounds as users, endpoints, and
  applications keep being added. Consistent policy is hard to maintain; separate wired and
  wireless policies leave the network vulnerable; and locating users and troubleshooting
  gets harder as people move around.
- SD-Access answers that with: **network automation** (one point of automation,
  orchestration, and management via DNA Center, with best-practice configurations);
  **network assurance and analytics** (telemetry-driven proactive prediction of network
  and security risks, including encrypted traffic); **host mobility** for wired and
  wireless; **identity services** (ISE identifies users and devices and supplies the
  contextual information policy needs); **policy enforcement** based on group identity via
  **SGACLs** instead of IP addresses and subnets; **secure segmentation** for guest,
  corporate, facilities, and IoT; and **network virtualization** — one physical
  infrastructure supporting multiple VRF instances (**virtual networks, VNs**), each with
  a distinct set of access policies.

**SD-Access = Campus Fabric + Cisco DNA Center**
- The **campus fabric** is a Cisco-validated fabric overlay solution containing all the
  features and protocols (control, data, management, and policy planes) needed to operate
  the network infrastructure.
- Managed via the **CLI or an API using NETCONF/YANG**, that same fabric is a **campus
  fabric solution**. Managed via **Cisco DNA Center**, it is **SD-Access**. Same
  hardware, same protocols — the management method is what changes the name.

**Four architecture layers**
- **Physical layer** — everything runs on real devices: Cisco **switches** (wired LAN
  access; Catalyst and Nexus), **routers** (WAN and branch access), **wireless** (WLCs and
  APs), and **controller appliances** — **Cisco DNA Center and Cisco ISE are the two
  required controller appliances**. Devices actively participating in the fabric must
  support the required hardware ASICs and FPGAs. Access switches that do *not* actively
  participate in the fabric but are part of it because of automation are **SD-Access
  extension nodes**.
- **Network layer** — the underlay plus the overlay; both deliver packets to and from the
  devices participating in SD-Access, and all of this information is made available to the
  controller layer.
- **Controller layer** — all the management subsystems for the management layer, provided
  by DNA Center and ISE.
- **Management layer** — the DNA Center UI/UX, the intent-based networking aspect of
  Cisco DNA.

**Underlay network**
- Its **sole purpose** is to transport data packets between network devices for the
  fabric overlay. It must be built for performance, scalability, and high availability,
  because any problem in the underlay affects the operation of the overlay.
- A **Layer 2 underlay running STP is possible but not recommended.** The recommended
  design is a **Layer 3 routed access campus design using IS-IS as the IGP**. IS-IS is
  chosen for concrete operational reasons: neighbor establishment **without IP
  dependencies**, peering using **loopback addresses**, and **agnostic treatment of IPv4,
  IPv6, and non-IP traffic**.
- **Manual underlay** — configured and managed by hand (CLI or API) rather than through
  DNA Center. Its advantage is **customization**: changing the IGP to OSPF for a special
  design requirement, or running SD-Access on top of a **legacy or third-party IP-based
  network**.
- **Automated underlay** — every aspect configured and managed by the DNA Center **LAN
  Automation** feature. LAN Automation creates an **IS-IS routed access** campus design
  and uses **Cisco Network Plug and Play** to deploy both **unicast and multicast**
  routing configuration in the underlay, improving traffic delivery efficiency. It
  eliminates misconfigurations, reduces complexity, and greatly speeds up building the
  underlay. Its downside: **it does not allow manual customization** for special design
  requirements.

**Overlay network (the SD-Access fabric)**
- A virtual (tunneled) network that virtually interconnects all the devices forming a
  fabric of interconnected nodes, **abstracting the inherent complexities and limitations
  of the underlay**.
- Provides policy-based network segmentation, host mobility for wired and wireless hosts,
  and enhanced security beyond normal switching and routing.
- **The fabric overlay is fully automated regardless of which underlay model is used**
  (manual or automated). It includes all necessary overlay control plane protocols and
  addressing, plus all global configuration associated with operating the fabric.
- The overlay *can* be configured manually without DNA Center — but managed via CLI or
  NETCONF/YANG it is a campus fabric solution, **not** SD-Access.

**Three planes of operation in the SD-Access fabric**
- **Control plane — LISP**
- **Data plane — VXLAN**
- **Policy plane — Cisco TrustSec**

**SD-Access control plane (LISP)**
- **Locator/ID Separation Protocol**, an IETF standard defined in **RFC 6830**, based on a
  simple **endpoint identifier (EID)** to **routing locator (RLOC)** mapping system that
  separates the *identity* (the endpoint's IP address) from its current *location* (the
  network edge or border router's IP address).
- It eliminates the need for every router to process every possible IP destination and
  route, by moving remote destination information into a centralized mapping database —
  the LISP **map server (MS)**, which in SD-Access is a **control plane node**. Each
  router then manages only its local routes and queries the map system to locate
  destination EIDs.
- Advantages for SD-Access: smaller routing tables; dynamic host mobility for wired and
  wireless endpoints; **address-agnostic mapping (IPv4, IPv6, and/or MAC)**; and built-in
  network segmentation through VRF instances.
- Cisco added several enhancements to the original LISP specifications for SD-Access:
  **distributed Anycast Gateway, VN Extranet, and Fabric Wireless**.

**SD-Access fabric data plane (VXLAN)**
- **VXLAN encapsulation is IP/UDP based**, so it can be forwarded by any IP-based network
  (legacy or third party), and it creates the overlay network for the fabric.
- **Although LISP is the control plane, SD-Access does not use LISP data encapsulation.**
  It uses VXLAN because VXLAN **can encapsulate the original Ethernet header to perform
  MAC-in-IP encapsulation, while LISP does not**. That single capability is what gives
  SD-Access **Layer 2 *and* Layer 3 virtual topologies**, the ability to operate over any
  IP-based network, built-in network segmentation (VRF instance / VN), and built-in
  group-based policy.
- **VXLAN-GPO (VXLAN Group Policy Option)** — the original VXLAN specification was
  enhanced for SD-Access to support Cisco TrustSec **Scalable Group Tags (SGTs)** by
  adding new fields to the **first 4 bytes of the VXLAN header**, in order to transport up
  to **64,000 SGT tags**. Defined in IETF draft `draft-smith-vxlan-group-policy-05`.
- Cisco TrustSec **Security** Group Tags are referred to as **Scalable** Group Tags in
  Cisco SD-Access — the same tag under two names.

**SD-Access fabric policy plane (Cisco TrustSec)**
- TrustSec SGT tags are assigned to **authenticated groups** of users or end devices.
  Network policy (ACLs, QoS) is then applied throughout the fabric **based on the SGT tag
  instead of a network address** (MAC, IPv4, or IPv6).
- That allows security, QoS, policy-based routing, and network segmentation policies based
  **only** on the SGT tag of the user or endpoint.
- Advantages: supports both **network-based segmentation using VNs (VRF instances)** and
  **group-based segmentation (policies)**; network address-independent policies reduce
  complexity; dynamic enforcement of group-based policies **regardless of location**, for
  both wired and wireless traffic; policy constructs over a legacy or third-party network
  using VXLAN; and **extended policy enforcement to external networks** (cloud or data
  center) by transporting the tags to TrustSec-aware devices using **SGT Exchange Protocol
  (SXP)**.

**Fabric edge node**
- Provides onboarding and mobility services for wired users and devices — including
  fabric-enabled WLCs and APs — connected to the fabric. It is a LISP **tunnel router
  (xTR)**.
- Provides the **anycast gateway**, endpoint authentication, assignment to overlay **host
  pools** (static or DHCP), and group-based policy enforcement for traffic to fabric
  endpoints.
- It first identifies and authenticates wired endpoints **through 802.1x**, to place them
  in a host pool (SVI and VRF instance) and a scalable group (SGT assignment). It then
  registers the specific EID host address — **MAC, /32 IPv4, or /128 IPv6** — with the
  control plane node.
- Provides a single Layer 3 anycast gateway (the same SVI with the same IP address on all
  fabric edge nodes) for its connected endpoints, and performs the encapsulation and
  de-encapsulation of host traffic.
- **An edge node must be either a Cisco switch or router operating in the fabric overlay.**

**Fabric control plane node**
- A LISP **map server/resolver (MS/MR)** with enhanced functions for SD-Access such as
  fabric wireless and SGT mapping. It maintains a simple **host tracking database** mapping
  EIDs to RLOCs.
- The control plane (host database) maps all EID locations to the current fabric edge or
  border node, and is capable of multiple EID lookup types (**IPv4, IPv6, or MAC**).
- It **receives registrations** from fabric edge or border nodes for known EID prefixes
  from wired endpoints, and from **fabric mode WLCs** for wireless clients. It **resolves
  lookup requests** from fabric edge or border nodes to locate destination EIDs, and
  **updates** fabric edge and border nodes with wired and wireless client mobility and
  RLOC information.
- **Control plane devices must maintain all endpoint (host) mappings in a fabric** — so a
  device with sufficient hardware and software scale for the fabric must be selected for
  this function.
- A control plane node must be either a Cisco switch or router, operating **either inside
  or outside** the SD-Access fabric.

**Fabric border nodes**
- LISP **proxy tunnel routers (PxTRs)** that connect external Layer 3 networks to the
  fabric and **translate reachability and policy information — such as VRF and SGT
  information — from one domain to another**.

**Fabric WLC and SD-Access wireless**
- A fabric-enabled WLC connects APs and wireless endpoints to the fabric. **The WLC is
  external to the fabric and connects to it through an internal border node.**
- The fabric WLC provides onboarding and mobility services for wireless users and
  endpoints, performs **PxTR registrations to the fabric control plane on behalf of the
  fabric edges**, and can be thought of as *a fabric edge for wireless clients*. The
  control plane node maps the host EID to the current fabric AP and the fabric edge node
  that AP is attached to.
- **Traditional wireless:** the WLC is centralized and **all** control plane *and* data
  plane (client data) traffic is tunneled to the WLC through **CAPWAP**.
- **SD-Access wireless:** the wireless **control plane remains centralized** (still
  CAPWAP to the WLC), but the **data plane is distributed using VXLAN directly from the
  fabric-enabled APs**. Fabric APs establish a **VXLAN tunnel to the fabric edge** to
  transport wireless client data traffic instead of the CAPWAP tunnel.
- **For this to work the AP must be directly connected to the fabric edge or to a fabric
  extended node.** Using VXLAN increases performance and scalability because client data
  no longer has to be tunneled to the WLC, and the routing decision is taken directly by
  the fabric edge.
- SGT- and VRF-based policies for wireless users on fabric SSIDs are applied **at the
  fabric edge, exactly as for wired users**. Wireless clients use regular host pools for
  traffic and policy enforcement, and the fabric WLC registers client EIDs with the control
  plane node (as located on the edge).

**Fabric concepts**
- **Virtual network (VN)** — virtualization at the device level, using VRF instances to
  create multiple Layer 3 routing tables. VRFs provide segmentation across IP addresses,
  allowing **overlapped address space** and traffic segmentation. In the **control plane**,
  **LISP instance IDs** maintain separate VRF instances; in the **data plane**, edge nodes
  add a **VXLAN VNID** to the fabric encapsulation.
- **Host pool** — a group of endpoints assigned to an IP pool subnet in the fabric. Fabric
  edge nodes have an **SVI for each host pool**, used by endpoints as their default
  gateway. The fabric uses EID mappings to advertise each host pool (per instance ID),
  which allows **host-specific (/32, /128, or MAC) advertisement and mobility**. Host
  pools can be assigned **dynamically** (host authentication such as 802.1x) and/or
  **statically** (per port).
- **Scalable group** — a group of endpoints with similar policies. The policy plane
  assigns every endpoint to a scalable group using TrustSec SGT tags, either **statically
  per fabric edge port** or **dynamically through AAA/RADIUS using ISE**. The same scalable
  group is configured on **all fabric edge and border nodes**, and groups are defined in
  DNA Center and/or ISE and advertised through TrustSec. There is a **direct one-to-one
  relationship between host pools and scalable groups**, so **scalable groups operate
  within a VN by default**. Fabric edge and border nodes include the SGT tag ID in **each
  VXLAN header**, carried across the fabric data plane, which keeps each scalable group
  separate and allows SGACL policy and enforcement.
- **Anycast gateway** — a pervasive Layer 3 default gateway where the **same SVI is
  provisioned on every edge node with the same SVI IP address and MAC address**. This
  allows an IP subnet to be stretched across the fabric: a subnet provisioned on the fabric
  is deployed across all edge nodes, and an endpoint in it can move to any edge node
  **without a change to its IP address or default gateway**. It simplifies IP address
  assignment and allows **fewer but larger** IP subnets. In essence the fabric behaves like
  a logical switch spanning multiple buildings.

**Controller layer subsystems**
- **Cisco Network Control Platform (NCP)** — integrated directly into DNA Center; provides
  all the **underlay and fabric automation and orchestration** services for the physical and
  network layers. Configures and manages devices using **NETCONF/YANG, SNMP, SSH/Telnet**,
  and so on, then reports automation status to the management layer.
- **Cisco Network Data Platform (NDP)** — integrated directly into DNA Center; the **data
  collection, analytics, and assurance** subsystem. Analyzes and correlates network events
  from multiple sources (**NetFlow and SPAN**), identifies historical trends, provides
  contextual information to NCP and ISE, and reports network operational status to the
  management layer.
- **Cisco ISE** — provides all the **identity and policy services** for the physical and
  network layers. Delivers NAC and identity services for dynamic endpoint-to-group mapping
  and policy definition using **802.1x, MAC Authentication Bypass (MAB), and Web
  Authentication (WebAuth)**. Collects contextual information shared from NDP and NCP (and
  other systems such as Active Directory and AWS), places profiled endpoints into the
  correct scalable group and host pool, and is **responsible for programming group-based
  policies on the network devices**.
- All three integrate with each other through **APIs**, and that contextual information is
  surfaced to the management layer.

**Management layer**
- The DNA Center management layer is the **UI/UX** layer where information from all other
  layers is presented in a centralized dashboard. It is the **intent-based networking
  aspect of Cisco DNA**.
- **A full understanding of the network layer (LISP, VXLAN, TrustSec) or the controller
  layer (NCP, NDP, ISE) is not required to deploy the fabric in SD-Access**, nor is there a
  requirement to know how to configure each individual device and feature to create the
  consistent end-to-end behavior SD-Access offers.
- Four workflows: **design, policy, provision, assurance**.

**Why SD-WAN exists**
- Digital transformation drivers: customers embracing multicloud, applications moving to
  the cloud, mobile and IoT devices growing exponentially, and the Internet edge moving to
  the branch.
- Customers adopting SD-WAN want to: centralize device configuration and network
  management; lower costs and reduce risk with simple WAN automation and orchestration;
  extend enterprise networks seamlessly into the public cloud; provide optimal user
  experience for SaaS applications; leverage a **transport-independent WAN** for lower cost
  and higher diversity — **the underlay can be any type of IP-based network: the Internet,
  MPLS, 3G/4G LTE, satellite, or dedicated circuits**; enhance application visibility and
  use it with **intelligent path control** to meet SLAs for business-critical and real-time
  applications; and provide end-to-end WAN traffic **segmentation and encryption**.
- Cisco offers two SD-WAN solutions: **Cisco SD-WAN (based on Viptela)** — preferred for
  organizations needing cloud-based initiatives with granular segmentation, advanced
  routing, advanced security, and complex topologies while connecting to cloud instances;
  and **Meraki SD-WAN** — recommended for organizations requiring **UTM** (unified threat
  management: firewall, VPN, intrusion prevention, antivirus, antispam, and web content
  filtering in a single appliance) with SD-WAN functionality, or existing Meraki customers
  expanding into SD-WAN. This topic covers Cisco SD-WAN based on Viptela.

**SD-WAN architecture — four main components plus one optional service**
- **SD-WAN edge devices** — physical or virtual devices that forward traffic across
  transports (WAN circuits/media) between locations.
- **vManage NMS** — the controller persona providing a **single pane of glass** (GUI) for
  managing and monitoring the solution.
- **vSmart controller** — the controller persona responsible for **advertising routes and
  data policies to edge devices**.
- **vBond orchestrator** — the controller persona that **authenticates and orchestrates
  connectivity between edge devices, vManage, and vSmart controllers**.
- **vAnalytics** — an **optional** analytics and assurance service.
- The vManage, vSmart, and vBond personas are **independent and operate as separate devices
  or virtual machines**. They can be hosted by Cisco, by select Cisco partners, or under the
  control of customers in their own environments. **Ensuring that edge devices can
  communicate with the three controller personas across all of the underlay circuits is an
  important design topic.**

**vBond orchestrator**
- A **virtualized vEdge running a dedicated function of the vBond persona**. Devices can
  locate the vBond through specific IP addresses or **FQDNs** — **FQDN is preferred**
  because it allows horizontal scaling of vBond devices and flexibility if a vBond ever
  needs to change its IP address.
- **Authentication** — the vBond authenticates **every device in the fabric**. As a device
  comes online it must authenticate to the vBond, which determines eligibility to join the
  fabric. Basic authentication of an SD-WAN router is done using **certificates and RSA
  cryptography**.
- **NAT detection** — detects when devices are placed behind NAT devices using **STUN
  (Session Traversal Utilities for NAT, RFC 5389)**. **Placing a vBond behind a NAT device
  is not recommended, but requires a 1:1 static NAT if it is.**
- **Load balancing** — provides load balancing of sessions to fabrics that have multiple
  vSmart or vManage controllers.
- **Every vBond has a permanent control plane connection over a DTLS tunnel with every
  vSmart controller.** As edge devices authenticate with the vBond they are directed to the
  appropriate vSmart and vManage device; NAT is detected; and then **the edge device session
  with the vBond is torn down**. A session with vBond is formed across **all** edge device
  transports so that NAT detection can take place for **every circuit**.

**vManage NMS**
- The **single pane of glass** NMS GUI used to configure and manage the full SD-WAN
  solution. It contains all the edge device configurations, controls software updates, and
  handles control and data plane policy creation. It also provides a method of configuring
  the SD-WAN fabric **via APIs**.

**vSmart controller**
- Uses **DTLS tunnels with edge devices** to establish **Overlay Management Protocol (OMP)**
  neighborships. **OMP is a proprietary routing protocol similar to BGP** that can advertise
  routes, next hops, keys, and policy information needed to establish and maintain the
  fabric.
- Processes OMP routes learned from SD-WAN edge devices (or other vSmart controllers) and
  advertises reachability information learned from those routes to the edge devices.
- Implements all the **control plane policies** created on vManage: logical tunnel
  topologies (hub and spoke, regional, partial mesh), service chaining, traffic engineering,
  and segmentation per VPN topology. Example: a policy created on vManage for an application
  (such as YouTube) requiring no more than 1% loss and 150 ms latency is downloaded to the
  vSmart controller; vSmart converts it into a format all edge devices understand and sends
  the data plane policy to the applicable edge devices — **without any need to log in to
  edge devices to configure the policy via CLI**.

**SD-WAN edge devices**
- Routers delivering the essential WAN, security, and multicloud capabilities of the
  solution. Available as physical hardware or as software virtualized routers sitting at the
  perimeter of a site — remote office, branch office, campus, data center, or cloud provider.
- They support **standard router features** (OSPF, EIGRP, BGP, ACLs, QoS, and routing
  policies) in addition to the SD-WAN overlay control and data plane functions.
- Each SD-WAN router **automatically establishes a secure DTLS connection with the vSmart
  controller** and forms an **OMP neighborship over that tunnel** to exchange routing
  information. It also establishes **standard IPsec sessions with other SD-WAN routers** in
  the fabric.
- SD-WAN routers have **local intelligence** to make site-local decisions regarding routing,
  high availability, interfaces, ARP management, and ACLs. The vSmart controller provides
  the remote site routes and reachability information necessary to build the fabric.
- **vEdge vs cEdge:** original Viptela hardware platforms running a dedicated Viptela OS are
  **vEdge** routers — **no longer available for purchase and considered legacy**, because
  they do not provide some of the security features enabled on IOS XE platforms. Cisco IOS XE
  platform devices are **cEdge** routers, using a unified image with autonomous features
  **starting with 17.2**. **vManage provisions, configures, and troubleshoots cEdge and vEdge
  routers exactly the same way.**
- **URL filtering and IPS are not supported on vEdge platforms** and may not be present on
  some cEdge platforms because they operate outside the operating system and process in IOS
  XE containers — some platforms such as the **ASR1K do not provide that capability**.

**vAnalytics**
- Optional analytics and assurance service: visibility into applications and infrastructure
  across the WAN, forecasting and what-if analysis, and intelligent recommendations.
- Example value: when a branch office experiences latency or loss on its MPLS link,
  vAnalytics detects it and **compares against other organizations it monitors in the same
  area** to see whether they see the same loss and latency — so the issue can be reported to
  the service provider with confidence. It can also **predict how much bandwidth is truly
  required** for a location, useful when deciding whether a circuit can be downgraded to
  reduce costs.

**Cloud OnRamp (CoR)**
- A set of functionalities addressing optimal cloud **SaaS** application access and **IaaS**
  connectivity. CoR delivers the best application quality of experience for SaaS
  applications by **continuously monitoring SaaS performance across diverse paths and
  selecting the best-performing path based on jitter, loss, and delay**. It also simplifies
  hybrid cloud and multicloud IaaS connectivity by extending the SD-WAN fabric to the public
  cloud while increasing high availability and scale.

**SD-WAN policy**
- The **most powerful component** of Cisco SD-WAN is the ability to push a **unified policy
  across the fabric**. Policy can modify the topology, influence traffic forwarding
  decisions, or filter traffic. Configuring policy on vManage allows changes to be pushed to
  **thousands of devices in a matter of minutes**.
- **Local policy** — part of the configuration pushed to the edge device by vManage:
  ACLs, QoS policies, and routing policies. (The OCG adds that centralized policies "would
  also include configuration of the on-device security stack.")
- **Centralized policy** — configuration changes to the **vSmarts**, where control plane
  functions are processed **before OMP routes are advertised** to edge devices. Data plane
  functionality is transmitted to the edge devices' **volatile memory** for enforcement.

**Application-Aware Routing (AAR)**
- Uses **BFD probes** inside the SD-WAN tunnels to track a tunnel's **packet loss, latency,
  and jitter**. BFD was originally designed for fast forwarding-path failure detection
  between two adjacent routers; in SD-WAN it is leveraged both to detect **path liveliness
  (up/down)** and to **measure quality** — loss, latency, jitter, and **IPsec tunnel MTU**.
- AAR considers factors in path selection **outside** those used by standard routing
  protocols (interface bandwidth, interface delay, hop count). An AAR policy ensures edge
  devices forward an application's traffic across a path meeting that application's defined
  needs. AAR can prefer one transport over another, but **if the preferred transport exceeds
  the defined thresholds for that application, AAR forwards the traffic across a different
  transport that meets the requirements**.
- This is what keeps business-critical applications on the best available path when network
  **brownouts or soft failures** occur.

**Cloud OnRamp for SaaS**
- SaaS applications reside mainly on the Internet, so the **best-performing Internet exit
  point** has to be selected. **In CoR SaaS, BFD is not used**, because there is no SD-WAN
  edge device on the SaaS side to form a BFD session with. Instead, the edge device at the
  remote site sends **small HTTP probes** to the SaaS application through both Internet
  circuits to measure **latency and loss**.
- A site that plans on forwarding SaaS traffic **directly to the Internet** is classified as
  a **CoR DIA site**.
- CoR for SaaS **does support load balancing** across Internet circuits providing similar
  scores; the variations for packet loss and latency are configurable to increase or decrease
  the ability to load balance traffic at a site.
- Connection quality is quantified as a **Viptela Quality of Experience (vQoE) score on a
  scale of 0 to 10** — 0 worst, 10 best — observable in the vManage GUI. The remote site edge
  uses the circuit with the **highest QoE score for that application**, and the forwarding
  decision is made on an **application-by-application** basis. Probing continues, and if the
  performance characteristics change the edge device changes the circuit used.
- **Gateway scenario:** with a single Internet circuit plus an MPLS circuit to a regional
  hub, CoR for SaaS is also enabled on the **regional hub edge device, designated as a
  gateway node**. HTTP quality probing starts on both the remote site and the hub. The hub's
  edge reports its HTTP connection loss and latency characteristics to the remote site edge in
  an **OMP message exchange through the vSmart controllers**. The remote site edge then
  compares its local Internet circuit's performance against the hub-reported performance,
  **also accounting for the loss and latency of traversing the SD-WAN fabric between the
  remote site and the hub — which is calculated using BFD** — and makes the forwarding
  decision.
- **Client site scenario:** customers using solely private transports (such as two different
  MPLS VPN providers) have their remote site classified as a **client site**. Client sites
  identify which path between two different gateways is best for a specific application based
  on the QoE score, and traffic is forwarded to the appropriate gateway site.

**Cloud OnRamp for IaaS**
- Multicloud is the new norm: certain workloads remain in private data centers while others
  are hosted in public cloud environments such as AWS and Microsoft Azure.
- With SD-WAN, **ubiquitous connectivity, zero-trust security, end-to-end segmentation, and
  application-aware QoS policies** can be extended into IaaS environments using **SD-WAN
  cloud routers**. The solution's transport-independent capability allows a variety of
  connectivity methods to securely extend the fabric to the public cloud across any underlay
  transport: the Internet, MPLS, 3G/4G LTE, satellite, and dedicated circuits such as
  **AWS's DX (Direct Connect)** and **Microsoft Azure's ER (ExpressRoute)**.

## Procedure

**SD-Access wired endpoint onboarding (at the fabric edge node):**
1. The endpoint connects to a fabric edge port.
2. The fabric edge identifies and authenticates the wired endpoint **through 802.1x**
   (or MAB / WebAuth via ISE).
3. The endpoint is placed into a **host pool** — an SVI plus a VRF instance — which becomes
   its default gateway (the anycast gateway).
4. The endpoint is assigned to a **scalable group** (SGT assignment), statically per port or
   dynamically through AAA/RADIUS using ISE.
5. The fabric edge **registers the specific EID host address** — MAC, /32 IPv4, or /128
   IPv6 — **with the control plane node**.
6. The control plane node records the EID-to-RLOC mapping in its host tracking database and
   resolves subsequent lookup requests from other fabric edge and border nodes, updating
   them with client mobility and RLOC information as the endpoint moves.

**SD-WAN edge device joining the fabric:**
1. The device comes online and locates the **vBond** by IP address or, preferably, FQDN.
2. It **authenticates to the vBond**, which determines eligibility to join the fabric —
   basic authentication of an SD-WAN router uses **certificates and RSA cryptography**.
3. A session with the vBond is formed **across all of the edge device's transports**, so
   **NAT detection using STUN** can take place for every circuit.
4. The vBond **directs the edge device to the appropriate vSmart and vManage** devices.
5. NAT is detected, and then **the edge device's session with the vBond is torn down**.
6. The edge device **automatically establishes a secure DTLS connection with the vSmart
   controller** and forms an **OMP neighborship** over that tunnel to exchange routing
   information.
7. The edge device establishes **standard IPsec sessions with other SD-WAN routers** in the
   fabric for data plane traffic.

**Cloud OnRamp for SaaS — DIA site path selection:**
1. CoR for SaaS is configured for a SaaS application on vManage.
2. The remote site edge device starts sending **small HTTP probes** to the SaaS application
   **through both Internet circuits** — BFD is not used, because there is no SD-WAN device
   on the SaaS side.
3. Latency and loss are measured per circuit and expressed as a **vQoE score from 0 to 10**,
   visible in the vManage GUI.
4. The edge device forwards that application's traffic out the circuit with the **highest
   QoE score**, on an application-by-application basis.
5. Probing continues. If the performance characteristics of the chosen circuit change, the
   edge device **moves the application to a different circuit**.

**Cloud OnRamp for SaaS — gateway site path selection:**
1. CoR for SaaS is configured on vManage and becomes active on the remote site edge device,
   and is also enabled on the **regional hub edge device designated as a gateway node**.
2. HTTP quality probing toward the SaaS application starts on **both** the remote site
   SD-WAN device and the regional hub SD-WAN device.
3. The regional hub's edge device reports its **HTTP connection loss and latency
   characteristics** to the remote site edge device in an **OMP message exchange through the
   vSmart controllers**.
4. The remote site edge evaluates the performance of its **local Internet circuit** against
   the performance reported by the hub.
5. It also factors in the **loss and latency incurred by traversing the SD-WAN fabric between
   the remote site and the hub site, calculated using BFD**.
6. It makes the forwarding decision, sending the application traffic down the best-performing
   path toward the SaaS application.

## Reference Tables

**SD-Access planes of operation**

| Plane | Technology | What it does |
|---|---|---|
| Control plane | **LISP** (RFC 6830) | EID-to-RLOC mapping — separates endpoint identity from location; centralized mapping database on the control plane node (map server) |
| Data plane | **VXLAN** (VXLAN-GPO in SD-Access) | IP/UDP, MAC-in-IP encapsulation; carries the VNID for segmentation and the SGT in the Group Policy ID field |
| Policy plane | **Cisco TrustSec** | SGT-based policy (ACLs, QoS, PBR, segmentation) applied on group identity rather than network address |

**SD-Access architecture layers**

| Layer | What it contains |
|---|---|
| Physical | Cisco switches (wired access), routers (WAN/branch access), WLCs and APs (wireless), and the two required controller appliances — DNA Center and ISE |
| Network | The underlay (transport) plus the overlay (the fabric); information from both is made available to the controller layer |
| Controller | NCP (automation/orchestration), NDP (analytics/assurance), ISE (identity/policy) — DNA Center and ISE |
| Management | The DNA Center GUI: base automation, design, policy, provision, assurance — the intent-based networking aspect of Cisco DNA |

**Underlay models**

| Model | Built by | IGP | Advantage | Downside |
|---|---|---|---|---|
| Manual underlay | CLI or API, not DNA Center | Whatever you choose (e.g. OSPF) | Customization for special design requirements; lets SD-Access run over a **legacy or third-party IP network** | Manual effort, and the misconfiguration risk that comes with it |
| Automated underlay | DNA Center **LAN Automation** + Network Plug and Play | **IS-IS**, routed access | Eliminates misconfigurations, reduces complexity, greatly speeds up the build; deploys both unicast and multicast routing config | **Does not allow manual customization** for special design requirements |

**SD-Access fabric roles**

| Role | LISP equivalent | Responsibility | Typical placement |
|---|---|---|---|
| Control plane node | **Map server / resolver (MS/MR)** | Holds the EID-to-RLOC mapping system and host tracking database for the fabric overlay; must scale to **all** endpoint mappings | A Cisco switch or router, **inside or outside** the fabric |
| Fabric border node | **Proxy tunnel router (PxTR)** | Connects external Layer 3 networks to the fabric; **translates VRF and SGT information** between domains | Core layer device |
| Fabric edge node | **Tunnel router (xTR)** | Connects wired endpoints; anycast gateway, 802.1x authentication, host pool and SGT assignment, EID registration, encap/decap | Access or distribution layer device; **must be a Cisco switch or router** |
| Fabric WLAN controller | Performs **PxTR registrations** on behalf of fabric edges | Connects APs and wireless endpoints; onboarding and mobility for wireless | **External to the fabric**, reaching it through an **internal border node** |
| Intermediate node | — | **No fabric role other than underlay services** | Intermediate routers or extended switches |

**Fabric border node types**

| Type | Also called | Connects to | Distinguishing configuration |
|---|---|---|---|
| Internal border | "rest of company" | **Only known areas** of the organization — WLC, firewall, data center | Knows the organization's internal prefixes |
| Default border | "outside" | **Only unknown areas** outside the organization | Configured with a **default route** to reach external networks (Internet, public cloud) not known to the control plane nodes |
| Internal + default border | "anywhere" | **Transit areas as well as known areas** | Combines internal and default border functionality into a single node |

**LISP vs VXLAN encapsulation — why SD-Access uses VXLAN for the data plane**

| | LISP encapsulation | VXLAN encapsulation |
|---|---|---|
| Encapsulation type | **IP-in-IP/UDP** | **MAC-in-IP/UDP** — encapsulates the original Ethernet header |
| Overlay support | **Layer 3 overlay only** | **Layer 2 *and* Layer 3 overlay** |
| Header stack | Outer Ethernet / Outer LISP IP / Outer LISP UDP / LISP header / Original IP / Data | Outer Ethernet / Outer VXLAN IP / Outer VXLAN UDP / VXLAN header / **Original Ethernet** / Original IP / Data |
| Segmentation field | Instance ID | **VNID (VXLAN Network Identifier)** |
| Role in SD-Access | **Control plane** | **Data plane** |

**VXLAN-GPO — fields added to the first 4 bytes of the VXLAN header**

| Field | Size | Meaning |
|---|---|---|
| **Group Policy ID** | 16-bit | Carries the SGT tag — up to **64,000** SGT tags |
| **Group Based Policy Extension Bit (G bit)** | 1-bit | **1** = an SGT tag is being carried in the Group Policy ID field; **0** = it is not |
| **Don't Learn Bit (D bit)** | 1-bit | **1** = the egress **VTEP must not learn** the source address of the encapsulated frame |
| **Policy Applied Bit (A bit)** | 1-bit | Defined as the A bit **only when the G bit is set to 1**. **1** = group policy already applied, further policies must **not** be applied by network devices. **0** = group policies must be applied, and the device must set A to 1 afterward |

**Traditional wireless vs SD-Access wireless**

| | Traditional wireless | SD-Access wireless |
|---|---|---|
| Wireless control plane | Centralized on the WLC, over **CAPWAP** | **Still centralized on the WLC, over CAPWAP** |
| Wireless data plane | Tunneled to the WLC over **CAPWAP** | **Distributed — VXLAN tunnel from the fabric AP to the fabric edge** |
| Routing decision | Taken at the WLC after the CAPWAP tunnel | Taken **directly by the fabric edge** |
| AP placement requirement | Any reachable switch | AP must be **directly connected to a fabric edge or fabric extended node** |
| Policy for wireless users | Applied centrally | **SGT- and VRF-based policy applied at the fabric edge, same as wired** |

**DNA Center workflows**

| Workflow | Tools |
|---|---|
| **Design** | Network Hierarchy (geolocation, building, floorplan, site ID); Network Settings (DNS, DHCP, AAA, device credentials, IP management, wireless settings); Image Repository (images, version compliance, deploy); Network Profiles (LAN/WAN/WLAN profiles such as SSID, applied to sites) |
| **Policy** | Dashboard (VNs, scalable groups, policies, recent changes); **Group-Based Access Control** (= SGACLs, integrating with ISE); IP-Based Access Control (ACL-equivalent); Application (QoS via application policies); Traffic Copy (**ERSPAN**); Virtual Network (set up VNs and associate scalable groups) |
| **Provision** | Devices (assign to site ID, confirm/update software, provision underlay config); Fabrics (set up fabric domains); **Fabric Devices** (add devices and specify roles — control plane, border, edge, WLC); **Host Onboarding** (host authentication type, assign wired/wireless host pools to VNs) |
| **Assurance** | Dashboard (global health of fabric and non-fabric devices and clients, scored by site); **Client 360**; **Devices 360**; Issues (reactive open issues and proactive developing trends) |

**SD-WAN components**

| Component | Plane | Mandatory? | Function |
|---|---|---|---|
| **vBond orchestrator** | Orchestration | **Yes** | Authenticates every device (certificates + RSA), NAT detection via **STUN**, load balances sessions to multiple vSmart/vManage; permanent **DTLS** connection with **every vSmart** |
| **vManage NMS** | Management | **Yes** | **Single pane of glass** GUI; holds all edge configs, software updates, control and data plane policy creation; also configurable via **APIs** |
| **vSmart controller** | Control | **Yes** | **DTLS** tunnels to edge devices carrying **OMP** (proprietary, BGP-like); advertises routes and data policies; implements centralized control plane policy |
| **SD-WAN edge devices** | Data | **Yes** | Forward traffic across transports; DTLS + OMP to vSmart, **IPsec to other SD-WAN routers**; standard routing features plus local site intelligence |
| **vAnalytics** | Analytics | **No — optional** | Visibility across the WAN, forecasting/what-if, intelligent recommendations, cross-organization comparison for SP escalation |

**SD-WAN policy types**

| Policy | Pushed to | Plane | Contains |
|---|---|---|---|
| **Local policy** | Edge devices, by vManage | Local config | ACLs, QoS policies, routing policies |
| **Centralized — Topology** | vSmarts | Control | Drop or modify routing behavior by changing path metrics or even the next hop |
| **Centralized — VPN Membership** | vSmarts | Control | Control advertisement of specific VPN prefixes to a specific site |
| **Centralized — Application-Aware Routing (AAR)** | vSmarts → edges | **Data** | Enhances forwarding on an application-by-application basis based on tunnel characteristics |
| **Centralized — Traffic Data** | vSmarts → edges | **Data** | Filter traffic per application, modify flows (next-hop change, service chaining, NAT), QoS functions, packet loss protection |

**Cloud OnRamp for SaaS — site types and probing**

| Site type | Transports | How the path is chosen |
|---|---|---|
| **DIA site** | Forwards SaaS traffic directly to the Internet | **HTTP probes** out each Internet circuit → latency/loss → **vQoE 0–10** → highest score wins, per application |
| **Gateway site** | Local Internet plus MPLS to a regional hub | Both remote site and hub probe via HTTP; the hub reports loss/latency via **OMP through the vSmarts**; the remote site compares local vs hub-reported, **plus the BFD-measured cost of traversing the fabric to the hub** |
| **Client site** | Solely private transports (e.g. two MPLS VPN providers) | Identifies which path between two different **gateways** is best for a specific application based on QoE score; traffic forwarded to the appropriate gateway site |

## Config Patterns
```ios-xe
! ============================================================================
! HONESTY NOTE: ENCOR Ch. 23 contains NO CLI. SD-Access is provisioned through
! Cisco DNA Center (Design > Policy > Provision > Assurance) and SD-WAN through
! vManage - by design, the management layer exists so you do NOT configure each
! device and feature by hand.
!
! The commands below are the real IOS-XE surfaces underneath those controllers.
! They are NOT from the source material and are NOT gear-validated here. Treat
! them as orientation for what the controller is writing, not as a deploy recipe.
! ============================================================================

! ---- Manual campus fabric solution: the LISP/VXLAN a fabric edge (xTR) runs ----
! Managed this way - CLI or NETCONF/YANG - the result is a CAMPUS FABRIC SOLUTION,
! not SD-Access. DNA Center management is what makes it SD-Access.
router lisp
 locator-set rloc_site1
  IPv4-interface Loopback0 priority 10 weight 10
 exit-locator-set
 !
 instance-id 4099                       ! LISP instance ID == the VN (VRF)
  remote-rloc-probe on-route-change
  dynamic-eid EMPLOYEE_10_1_10_0
   database-mapping 10.1.10.0/24 locator-set rloc_site1
  exit-dynamic-eid
  !
  service ipv4
   eid-table vrf EMPLOYEE
   map-cache 0.0.0.0/0 map-request
   itr map-resolver 10.255.0.1          ! the fabric control plane node (MS/MR)
   etr map-server 10.255.0.1 key <key>
   etr
   itr
  exit-service-ipv4
 exit-instance-id

! ---- Fabric edge: anycast gateway SVI (same IP AND MAC on every edge node) ----
interface Vlan1021
 description --- EMPLOYEE host pool - anycast gateway ---
 vrf forwarding EMPLOYEE
 ip address 10.1.10.1 255.255.255.0
 mac-address 0000.0c9f.f45e
 lisp mobility EMPLOYEE_10_1_10_0

! ---- Fabric edge access port: 802.1x onboarding into host pool + scalable group ----
interface GigabitEthernet1/0/10
 switchport access vlan 1021
 switchport mode access
 access-session host-mode multi-auth
 access-session port-control auto
 dot1x pae authenticator
 mab
 service-policy type control subscriber FABRIC_POLICY

! ---- TrustSec / policy plane ----
cts authorization list ISE_LIST
cts role-based enforcement
cts role-based sgt-map 10.1.10.50 sgt 15

! ---- SD-WAN cEdge: the identity that lets vBond authenticate this router ----
system
 system-ip             10.255.1.1
 site-id               101
 organization-name     "EXAMPLE-ORG"
 vbond vbond.example.com            ! FQDN preferred over an IP for horizontal scaling
!
! Everything else - OMP, IPsec to other edges, AAR thresholds, traffic data policy -
! is pushed from vManage. The device does not get configured feature-by-feature.
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Build the SD-Access underlay as a **Layer 3 routed access campus design using IS-IS** | IS-IS establishes neighbors without IP dependencies, peers on loopbacks, and treats IPv4, IPv6, and non-IP traffic agnostically — which is exactly what an overlay transport needs | A **manual underlay** deliberately chosen to meet a special design requirement, such as changing the IGP to OSPF, or running SD-Access on top of a legacy or third-party IP network | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("Underlay Network") |
| Do **not** build the underlay as a Layer 2 network running STP | It is possible but not recommended; any problem in the underlay directly affects the operation of the overlay, and STP reintroduces exactly the convergence and loop exposure the fabric is meant to remove | A brownfield site where re-architecting the underlay is not yet funded and the risk is a knowingly accepted, dated decision | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("Underlay Network") |
| Use DNA Center **LAN Automation** for the underlay unless a specific customization is required | It eliminates misconfigurations, reduces complexity, speeds the build, and deploys both unicast and multicast routing config for efficient traffic delivery | A special design requirement that the automated underlay cannot express — it **does not allow manual customization** — or an existing legacy/third-party underlay | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("Manual underlay" / "Automated underlay") |
| Select a control plane node with **sufficient hardware and software scale for the entire fabric** | Control plane devices must maintain **all** endpoint (host) mappings in the fabric — under-sizing this role fails the whole overlay, not one device | Small fabrics where any supported platform has ample headroom; designs distributing the role across multiple control plane nodes | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 (NOTE under "Fabric Control Plane Node") |
| Connect fabric APs **directly to a fabric edge node or a fabric extended node** | The distributed VXLAN data plane from AP to fabric edge only works with that direct attachment; otherwise wireless data stays on CAPWAP to the WLC and the performance and scalability benefit is lost | SSIDs deliberately kept on traditional central-switched CAPWAP alongside the fabric, where the AP is intentionally not fabric-enabled | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("Fabric Wireless Controller (WLC)") |
| Have SD-WAN edge devices locate the **vBond by FQDN rather than IP address** | FQDN allows **horizontal scaling** of vBond devices and gives flexibility if a vBond ever needs to change its IP address | Environments where DNS resolution is not dependably available to edge devices during onboarding | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("vBond Orchestrator") |
| Do **not** place a vBond behind a NAT device; if it must be, use a **1:1 static NAT** | The vBond is the device every other device authenticates against and the point where NAT detection happens for every circuit — putting it behind dynamic NAT undermines that role | Hosted or cloud deployments whose architecture requires it, with the 1:1 static NAT explicitly configured | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("vBond Orchestrator — NAT detection") |
| Ensure edge devices can communicate with **all three controller personas across all underlay circuits** | The OCG calls this out as an important design topic: a session with vBond is formed across every transport precisely so NAT detection happens per circuit, and control plane reachability on only one transport is a hidden single point of failure | A transport deliberately restricted from controller reachability (e.g. an MPLS VRF with no controller path) where single-transport control plane is an accepted, documented decision | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 23 ("Cisco SD-WAN Architecture") |

*A deviation from this table is a question for the network's operator — "is
this intentional here?" — never automatically a finding.*

*The OCG also points at the [Cisco Validated Design guides](https://www.cisco.com/go/cvd)
for SD-Access design and deployment — go there before committing a fabric design.*

## Verification Commands

*Ch. 23 contains no show commands — it is a controller-driven chapter. The CLI below is
the real IOS-XE surface underneath DNA Center and vManage, offered for orientation. It is
not from the source material and is not gear-validated here; the GUI paths are the
chapter's own answer.*

| Command / GUI path | What to look for |
|---|---|
| `show lisp instance-id <id> ipv4 database` | Local EIDs this fabric edge has registered — if an onboarded endpoint is missing here, the problem is at onboarding, not forwarding |
| `show lisp instance-id <id> ipv4 map-cache` | Resolved EID-to-RLOC entries; a missing or negative entry points at the control plane node |
| `show lisp session` | Session state to the map server/resolver (the control plane node) — down here explains every downstream symptom |
| `show access-session interface <intf> details` | 802.1x/MAB authentication result on a fabric edge port, and the authorized VLAN/SGT applied |
| `show cts role-based sgt-map all` | Which SGT each endpoint actually got — compare against the scalable group the design intends |
| `show cts environment-data` | TrustSec environment data downloaded from ISE; empty means the policy plane never armed |
| `show device-tracking database` | Endpoint tracking entries on the fabric edge feeding EID registration |
| `show sdwan control connections` | Which of vBond, vSmart, vManage are **up**, and **on which transport (color)** — the first command on any SD-WAN onboarding problem |
| `show sdwan control local-properties` | System IP, site ID, organization name, and certificate status — mismatches here are why a device never joins |
| `show sdwan bfd sessions` | Tunnel liveliness plus the loss/latency/jitter that AAR decisions are built on |
| `show sdwan omp peers` / `show sdwan omp routes` | OMP neighborship to the vSmarts and the routes it is advertising/receiving |
| `show sdwan app-route stats` | Per-tunnel loss, latency, and jitter against AAR thresholds — why traffic moved transports |
| **DNAC → Provision → Fabric Devices** | The role each device is actually assigned (control plane, border, edge, WLC) versus the design |
| **DNAC → Provision → Host Onboarding** | Host authentication type and which host pools are mapped to which VNs |
| **DNAC → Assurance → Client 360 / Devices 360** | Client onboarding and app experience; device resource usage, loss, and latency |
| **vManage → Monitor → Network** | Per-device control connections, BFD/tunnel health, and application-aware routing behavior |

## Intent Questions
- Is this deployment actually **SD-Access, or a campus fabric solution**? If DNA Center is
  not managing the overlay — if it is driven by CLI or NETCONF/YANG — it is the latter, and
  every automation-based troubleshooting path is off the table.
- Which **fabric role(s)** is this device supposed to hold — control plane, border (internal,
  default, or anywhere), edge, WLC, or intermediate — and does its configuration match? Nodes
  legitimately hold more than one role.
- Is the underlay **manual or LAN-Automated**, and what IGP is it supposed to be running?
  An automated underlay is IS-IS by definition; anything else means someone chose manual.
- Is segmentation here supposed to be **network-based (VN / VRF instance)**, **group-based
  (SGT / SGACL)**, or both? They are different mechanisms with different failure modes.
- For SD-WAN: which **transports** is this site supposed to have, and is controller
  reachability (vBond, vSmart, vManage) expected on **all** of them or only some?

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then write
   the one-line symptom ("should ___, isn't ___") — before running any show command or
   opening any dashboard.
1. **Underlay first.** The overlay can never be healthier than the underlay. Check IGP
   adjacencies (IS-IS in an automated underlay), loopback reachability between RLOCs, and
   link errors. A fabric symptom is very often an underlay fault wearing a costume.
2. **MTU.** VXLAN adds encapsulation on top of the original Ethernet header; the underlay has
   to carry it. Intermittent large-packet failure with working pings is the classic signature.
3. **Fabric role assignment.** In DNAC → Provision → Fabric Devices, confirm the device holds
   the role the design says. A device that was never added to the fabric domain will look
   healthy and do nothing.
4. **Control plane node health.** Is it reachable, is the LISP session up, and does its host
   tracking database contain the EID in question? Confirm it has the scale to hold **all**
   fabric mappings.
5. **Edge onboarding.** Did 802.1x/MAB/WebAuth actually succeed on the port? Was the endpoint
   placed in the intended host pool (SVI + VRF) and the intended scalable group?
6. **EID registration.** Did the fabric edge register the specific host address (MAC, /32, or
   /128) with the control plane node? Onboarding can succeed while registration fails.
7. **Wireless specifics.** Is the AP **directly connected** to a fabric edge or extended node?
   Is the WLC fabric-enabled and reaching the fabric through an **internal border node**?
   Remember the control plane is still CAPWAP — only the data plane moved to VXLAN.
8. **Border type.** Is the right border in play for the destination? No **default border**
   means no path to destinations unknown to the control plane nodes — including the Internet.
9. **Policy plane.** Was an SGT assigned, and is the SGACL deployed on the enforcement point?
   Check whether the **A bit** is already set, which tells downstream devices not to apply
   policy again.
10. **SD-WAN control plane.** `show sdwan control connections` — which controllers are up and
    on which transports? Then certificate validity, organization-name and system-IP matches,
    and whether a NAT in the path was detected correctly.
11. **SD-WAN data plane.** BFD sessions up? OMP peers and routes present? Are AAR thresholds
    (loss, latency, jitter) being exceeded and moving traffic between transports — which looks
    like instability but is the policy working as designed?
12. **Platform limits and software.** vEdge platforms do not support URL filtering or IPS;
    some cEdge platforms (the ASR1K among them) do not provide it either. cEdge unified image
    features start at **17.2**. Confirm the feature exists on the platform before debugging
    why it isn't working.

## Common Pitfalls
- **SD-Access = campus fabric + Cisco DNA Center.** Manage the exact same fabric via CLI or
  NETCONF/YANG and it is a **campus fabric solution**, not SD-Access. Same hardware, same
  protocols, different answer.
- **LISP is the control plane; VXLAN is the data plane.** SD-Access does **not** use LISP
  data encapsulation.
- The reason is **MAC-in-IP**: VXLAN can encapsulate the original Ethernet header, so it
  supports **Layer 2 and Layer 3** overlays. LISP encapsulation is IP-in-IP/UDP and supports
  Layer 3 only. It is not about a smaller header (VXLAN's is larger) and not about IPv6.
- **EVPN / MP-BGP is a real VXLAN control plane — for data center fabrics, not SD-Access.**
  This is the highest-value distractor in the chapter and a genuine real-world confusion.
- **The SD-Access VXLAN header is not the original VXLAN header.** It is **VXLAN-GPO**, with
  fields added to the **first 4 bytes** to carry up to **64,000 SGT tags**.
- **The SGT-carrying field is the Group Policy ID** (16-bit). "Scalable Group ID," "Group
  Based Tag," and "Group Based Policy" are plausible-sounding inventions.
- **TrustSec Security Group Tag and SD-Access Scalable Group Tag are the same tag** under two
  names.
- The **A bit exists only when the G bit is 1**. A=1 means policy has already been applied and
  downstream devices must **not** apply it again — a silent cause of "my SGACL isn't hitting."
- **SD-Access is for enterprise campus and branch.** Not data center, not service provider,
  not WAN.
- **Firewalls and IPS are not SD-Access fabric components.** Switches, routers, WLCs, APs,
  ISE, and DNA Center are. **ISE and DNA Center are both required** controller appliances.
- **The WLC is external to the fabric** and connects to it through an **internal border node**.
- In SD-Access wireless the **control plane stays centralized on CAPWAP** — only the data
  plane moves to VXLAN from the AP. "CAPWAP is gone" is wrong.
- **The fabric AP must be directly connected to a fabric edge or extended node** for the VXLAN
  data plane to work.
- **Automated underlay means IS-IS and no manual customization** — that constraint is the
  entire reason a manual underlay exists as an option.
- A **Layer 2 STP underlay is possible but not recommended**.
- **Extension nodes are part of SD-Access through automation but do not actively participate
  in the fabric.**
- **Host pools and scalable groups are one-to-one**, which is why scalable groups operate
  within a VN by default.
- **Anycast gateway means the same SVI IP *and* the same MAC** on every edge node — not just
  the same subnet.
- **SD-WAN has exactly four mandatory components**: vManage, vSmart, vBond, and the SD-WAN
  routers. **vAnalytics is optional**, and **ISE and DNA Center are not SD-WAN components at
  all**.
- **vSmart uses DTLS to edge devices for OMP — not IPsec.** IPsec runs **between SD-WAN
  routers** for data plane traffic. The permanent DTLS relationship is **vBond to every
  vSmart**.
- **SD-WAN is transport independent** — Internet, MPLS, 3G/4G LTE, satellite, and dedicated
  circuits. "Internet or MPLS only" is wrong.
- **vManage is SD-WAN's single pane of glass.** DNA Center is SD-Access's. Do not cross them.
- **The vBond authenticates the vSmart controllers and the SD-WAN routers** and orchestrates
  connectivity between them. Distractors swap vManage into that sentence — read the pairing
  carefully.
- The **vBond is itself a virtualized vEdge** running a dedicated persona, and the edge
  device's session with it is **torn down** once NAT is detected and it has been handed off.
- **CoR for SaaS uses HTTP probes, not BFD** — there is no SD-WAN device on the SaaS side to
  form a BFD session with. BFD is still used *inside* the fabric, including to measure the
  remote-site-to-hub cost in the gateway scenario.
- **BFD in SD-WAN does more than liveliness**: it measures loss, latency, jitter, **and IPsec
  tunnel MTU**.
- **vEdge is legacy Viptela OS; cEdge is IOS XE** (unified image from 17.2). vManage manages
  both identically, but **URL filtering and IPS are unsupported on vEdge** and absent on some
  cEdge platforms such as the **ASR1K**.
- **Centralized policy's data plane functionality lands in the edge devices' volatile
  memory** — it is not written to the startup configuration.

## Exam Preparation Tasks

### Key topics coverage map

Table 23-2, mapped to where each element actually lives in this skill. The mapping is
deliberately not one-to-one: several key topics collapse into a single Reference Table here,
and a few are split between Key Concepts and Common Pitfalls.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| List | SD-Access capabilities, features, and functionalities | 645 | Key Concepts → "Why SD-Access exists" (the seven capabilities as one bullet block) |
| Figure 23-2 | Cisco SD-Access Architecture | 647 | Reference Tables → "SD-Access architecture layers"; expanded in Key Concepts → "Four architecture layers" |
| Section | Underlay Network | 648 | Key Concepts → "Underlay network"; Design Baseline rows 1–3; Troubleshooting step 1 |
| List | Types of underlay networks supported by SD-Access | 648 | Reference Tables → "Underlay models" (manual vs automated, including the customization tradeoff) |
| Section | Overlay Network (SD-Access Fabric) | 649 | Key Concepts → "Overlay network (the SD-Access fabric)" — including the campus-fabric-vs-SD-Access distinction, which also appears in Common Pitfalls |
| List | SD-Access basic planes of operation | 649 | Reference Tables → "SD-Access planes of operation" |
| Section | SD-Access Control Plane | 649 | Key Concepts → "SD-Access control plane (LISP)"; verification via `show lisp …` |
| Section | SD-Access Fabric Data Plane | 650 | Key Concepts → "SD-Access fabric data plane (VXLAN)"; Reference Tables → "LISP vs VXLAN encapsulation" (this one row satisfies both the section and quiz Q1/Q2) |
| Section | SD-Access Fabric Policy Plane | 651 | Key Concepts → "SD-Access fabric policy plane (Cisco TrustSec)"; Reference Tables → "VXLAN-GPO fields" |
| List | SD-Access fabric roles | 652 | Reference Tables → "SD-Access fabric roles" (single table covering all five roles + their LISP equivalents) |
| Section | Fabric Edge Nodes | 652 | Key Concepts → "Fabric edge node"; Procedure → "SD-Access wired endpoint onboarding" |
| Section | Fabric Control Plane Node | 653 | Key Concepts → "Fabric control plane node"; Design Baseline row 4 (scale selection) |
| Section | Fabric Border Nodes | 654 | Key Concepts → "Fabric border nodes" |
| List | Types of border nodes | 654 | Reference Tables → "Fabric border node types" |
| Section | Fabric Wireless Controller (WLC) | 654 | Key Concepts → "Fabric WLC and SD-Access wireless"; Reference Tables → "Traditional wireless vs SD-Access wireless"; Design Baseline row 5 |
| List | SD-Access fabric concepts | 655 | Key Concepts → "Fabric concepts" (VN, host pool, scalable group, anycast gateway) |
| Section | Controller Layer | 656 | Key Concepts → "Controller layer subsystems" |
| List | SD-Access three main controller subsystems | 656 | Key Concepts → "Controller layer subsystems" (NCP, NDP, ISE — same block satisfies both this list and the section above) |
| Section | Management Layer | 657 | Key Concepts → "Management layer"; Reference Tables → "DNA Center workflows" |
| List | SD-WAN main components | 662 | Reference Tables → "SD-WAN components" (with the mandatory/optional column that quiz Q7 turns on) |
| Section | vBond Orchestrator | 662 | Key Concepts → "vBond orchestrator"; Procedure → "SD-WAN edge device joining the fabric"; Design Baseline rows 6–7 |
| Section | vManage NMS | 663 | Key Concepts → "vManage NMS" |
| Section | vSmart Controller | 663 | Key Concepts → "vSmart controller" |
| Section | Cisco SD-WAN Edge Devices | 663 | Key Concepts → "SD-WAN edge devices" (including the vEdge/cEdge distinction and platform feature limits) |
| Section | SD-WAN Cloud OnRamp | 664 | Key Concepts → "Cloud OnRamp (CoR)", "Cloud OnRamp for SaaS", "Cloud OnRamp for IaaS"; Reference Tables → "CoR for SaaS site types"; Procedure → two CoR path-selection sequences |

No gaps — every row in Table 23-2 maps to content in this skill.

Two key topics not in Table 23-2 but worth the same weight, because the quiz tests them
directly: **VXLAN-GPO's added fields** (page 651, tested by Q4) and **SD-Access's intended
environment** (page 643, tested by Q5).

### "Do I Know This Already?" question analysis

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| 1 | Whether you know *why* the data plane and control plane use different protocols — that VXLAN was chosen for **MAC-in-IP**, giving Layer 2 overlay support LISP encapsulation cannot provide | **B** — VXLAN supports Layer 2 networks | Every distractor is a plausible-sounding property that is either **also true of LISP** ("supports IPv6") or **factually backwards** ("much smaller header" — VXLAN's header is larger). Tests whether you memorized a fact or understood the encapsulation difference |
| 2 | Whether you know SD-Access modified the VXLAN header rather than adopting it as-is | **B — False** | The trap is assuming "standards-based VXLAN" means "unmodified VXLAN." SD-Access uses **VXLAN-GPO**, with new fields in the first 4 bytes. Anyone who learned VXLAN from the data center side will get this wrong |
| 3 | Clean separation of control plane from data plane in SD-Access | **A** — LISP control plane | **The highest-value distractor in the chapter: "EPVN MP-BGP."** EVPN/MP-BGP is a *real* VXLAN control plane — in **data center** fabrics. Offering it here catches anyone who learned VXLAN in a DC context and assumed the control plane came with it. Also promoted to Common Pitfalls |
| 4 | Precise recall of the VXLAN-GPO field that carries the SGT | **A** — Group Policy ID | Three invented terms ("Scalable Group ID," "Group Based Tag," "Group Based Policy") sitting next to real SD-Access vocabulary. Every one sounds like something you've read, because SD-Access really does use "scalable group" and "group-based policy" elsewhere. Tests exact field naming, not concept |
| 5 | Scope boundary — what SD-Access was designed for | **C** — Enterprise campus and branch | Data center, service provider, and WAN are all offered because they are all places Cisco sells fabric. The trap is generalizing "fabric" into "fabric everywhere." SD-WAN, not SD-Access, is the WAN answer |
| 6 | Which device classes are actual SD-Access fabric architecture components | **A, B, D, E, F, G** — WLCs, routers, switches, APs, ISE, DNA Center | The two exclusions are **firewalls** and **intrusion prevention systems** — real security devices that live *adjacent* to the fabric (an internal border connects to a firewall) but are not fabric components. Tests whether you can separate "attached to" from "part of" |
| 7 | Mandatory vs optional SD-WAN components | **A, B, C, D** — vManage, vSmart, SD-WAN routers, vBond | **vAnalytics is the trap**: it is a genuine part of the solution but explicitly **optional**. ISE and DNA Center are offered to catch anyone blending SD-Access and SD-WAN component lists |
| 8 | Which tunnel type each SD-WAN relationship uses, and between which devices | **B — False** | Half-right by design: **permanent** is true (of vBond↔vSmart), and **IPsec** is true (between SD-WAN routers). The statement pairs the right mechanisms with the wrong devices. vSmart↔edge is **DTLS carrying OMP**. Correct mechanism, wrong endpoints — the most common way SD-WAN questions are built |
| 9 | Whether "transport independent" actually registered | **B — False** | Internet and MPLS are named first in most SD-WAN material, so they stick. LTE, satellite, and dedicated circuits are equally valid underlays. Tests whether you read the definition or pattern-matched the common examples |
| 10 | Single pane of glass for SD-WAN | **C** — vManage | Straight recall, with one meaningful trap: **DNA Center** is offered, and it *is* a single pane of glass — for **SD-Access**. The distractor works only on people who merged the two solutions' vocabulary |
| 11 | The vBond's exact role and, more importantly, exactly **which devices** it authenticates | **B** — authenticate the vSmart controllers and the SD-WAN routers, and orchestrate connectivity between them | All three options are the *same sentence* with the device pair swapped. There is no conceptual shortcut — you have to know the pairing. Note the OCG's own body text is broader ("edge devices, vManage, and vSmart controllers"), so this question rewards the answer key's narrower pairing; read the options against each other, not against memory |

**Pattern across the quiz:** nine of eleven questions are built on **cross-solution
confusion** (SD-Access vocabulary offered for SD-WAN questions and vice versa) or on
**right-mechanism-wrong-devices** pairings. Neither is tested by recall. Both are caught by
being able to say which *plane* and which *device role* a given technology belongs to — which
is why this skill's Reference Tables are organized that way.
