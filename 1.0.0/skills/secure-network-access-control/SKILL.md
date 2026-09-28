---
name: ccnp-secure-network-access-control
description: >
  Use this skill when designing, troubleshooting, or configuring secure network access
  control on IOS-XE — the Cisco security framework, the endpoint/network security
  portfolio, and the NAC technologies that decide who gets on the network and what they
  can reach. Invoke when the user asks about: secure network access control, network
  security design, threat defense, Cisco SAFE, Secure Architectural Framework, places in
  the network, PIN, secure domains, security intelligence, segmentation, secure services,
  attack continuum, before during after, next-generation endpoint security, Cisco Talos,
  threat intelligence, IronPort SecApps, Sourcefire VRT, TRAC team, honeypots, Cisco
  Secure Malware Analytics, Threat Grid, sandbox, static file analysis, dynamic file
  analysis, behavioral analysis, Glovebox, AMP, Cisco Advanced Malware Protection,
  Malware Defense, AMP Cloud, AMP connectors, file disposition, file reputation,
  retrospection, IoC, indicators of compromise, AMP for Networks, AMP for Meraki MX,
  Cisco Secure Endpoint, FireAMP, Cisco Security Connector, CSC, Cisco Secure Client,
  AnyConnect, Secure Mobility Client, VPN posture, HostScan, ISE Posture module, Cisco
  Umbrella, OpenDNS, DNS-layer security, Anycast DNS, roaming security module, Umbrella
  roaming client, Cisco Secure Web Appliance, WSA, web gateway, web reputation filters,
  reputation score, web filtering, Dynamic Content Analysis, DCA engine, AVC,
  Application Visibility and Control, cloud access security, CASB, CloudLock, parallel
  AV scanning, Layer 4 traffic monitoring, DLP, data loss prevention, ICAP, Global
  Threat Analytics, GTA, Cognitive Threat Analytics, CTA, Cisco Secure Email, ESA,
  Email Security Appliance, CASE, Context Adaptive Scanning Engine, forged email
  detection, BEC, Cisco Advanced Phishing Protection, CAPP, Cisco Domain Protection,
  CDP, graymail, Safe Unsubscribe, outbreak filters, web interaction tracking, Cisco
  Secure IPS, FirePOWER NGIPS, IDS, IPS, next-generation IPS, Snort, Cisco Secure
  Firewall, NGFW, next-generation firewall, stateful inspection, ASA, ASA 5500-X, FTD,
  Firepower Threat Defense, FTD software image, FMC, Firepower Management Center,
  Secure Firewall Management Center, FDM, Firepower Device Manager, Cisco Defense
  Orchestrator, CSM, ASDM, SecureX, Cisco Secure Network Analytics, Stealthwatch
  Enterprise, SMC, Stealthwatch Management Console, Flow Collector, Flow Sensor, UDP
  Director, Cisco Telemetry Broker, Data Store, Data Node, Flow Rate License, Threat
  Feed License, Encrypted Traffic Analytics, ETA, Cisco Secure Cloud Analytics,
  Stealthwatch Cloud, Public Cloud Monitoring, Network Analytics SaaS, VPC flow logs,
  Cisco ISE, Identity Services Engine, NAC, network access control, device profiling,
  endpoint posture service, posture assessment, guest lifecycle management, device
  onboarding, supplicant provisioning, internal certificate authority, pxGrid, Cisco
  Platform Exchange Grid, pxGrid 1.0, pxGrid 2.0, XMPP, STOMP, pxGrid node, ANC,
  quarantine, unquarantine, 802.1x, dot1x, port-based network access control, PNAC,
  EAP, Extensible Authentication Protocol, EAPoL, EAP over LAN, RADIUS, supplicant,
  authenticator, authentication server, NAD, network access device, EAP methods,
  EAP-MD5, EAP-TLS, PEAP, PEAPv0, PEAPv1, EAP-FAST, EAP-TTLS, EAP-GTC, EAP-MSCHAPv2,
  inner method, outer method, tunneled TLS, PAC, protected access credentials, EAP
  chaining, machine and user authentication, MAB, MAC Authentication Bypass, dACL,
  downloadable ACL, dVLAN, dynamic VLAN assignment, WebAuth, Web Authentication, LWA,
  Local Web Authentication, CWA, Central Web Authentication, CoA, change of
  authorization, guest VLAN, AUP, acceptable use policy, FlexAuth, Enhanced Flexible
  Authentication, Access Session Manager, IBNS, Identity-Based Networking Services,
  IBNS 2.0, C3PL, Common Classification Policy Language, Cisco TrustSec, SGT, Security
  Group Tag, Scalable Group Tag, ingress classification, propagation, egress
  enforcement, dynamic SGT assignment, static SGT assignment, IP to SGT, subnet to SGT,
  VLAN to SGT, port to SGT, inline tagging, native tagging, CMD, Cisco Metadata,
  0x8909, SXP, SGT Exchange Protocol, speaker, listener, single-hop SXP, multi-hop SXP,
  IP-SGT binding table, SGACL, Security Group ACL, SGFW, Security Group Firewall,
  production matrix, MACsec, 802.1AE, hop-by-hop encryption, MACsec Security Tag, ICV,
  Integrity Check Value, GMAC, AES-GCM, 0x88e5, TCI/AN, SCI, Secure Channel Identifier,
  SAP, Security Association Protocol, MKA, MACsec Key Agreement, downlink MACsec,
  uplink MACsec.
---

## Purpose
Secure network access control is the decision layer in front of the network: a framework
(Cisco SAFE) for *where* to put security, a product portfolio for *detecting* threats
before/during/after an attack, and a set of NAC technologies (802.1x, MAB, WebAuth,
TrustSec, MACsec) that authenticate every endpoint and then constrain what it is allowed
to reach. The through-line is identity — nothing gets an authorization result until
something has proven who or what it is.

## Key Concepts

**Cisco SAFE — the framework**
- Cisco created the **Secure Architectural Framework (SAFE)** because no single product
  secures an organization against phishing, smishing, malware, ransomware, and web-based
  exploits. SAFE is **modular** — PINs that do not exist in a network are simply removed.
- SAFE organizes security around **places in the network (PINs)** — the six locations
  security gets designed for:
  - **Branch** — typically the least secure PIN, because applying full campus/DC controls
    to a large number of branches is cost-prohibitive. Prime breach targets. Top threats:
    endpoint/POS malware, rogue-AP MitM and DoS, unauthorized client activity, exploitation
    of trust.
  - **Campus** — many users (employees, contractors, guests, partners). Easy targets for
    phishing, web exploits, unauthorized access, malware propagation, botnets.
  - **Data center** — the crown jewels, so the primary goal of targeted threats. Hundreds
    or thousands of servers make rule management hard. Threats: data extraction, malware
    propagation, application compromise, botnet infestation (scrumping), data loss,
    privilege escalation, reconnaissance.
  - **Edge** — primary ingress/egress for Internet traffic, the **highest-risk PIN** and
    most important for e-commerce. Threats: web server vulnerabilities, DDoS, data loss,
    MitM.
  - **Cloud** — security dictated by SLAs with the provider; requires independent
    certification audits and risk assessments. Threats: web server vulns, loss of access,
    data loss, malware, MitM.
  - **WAN** — connects the PINs together; managing security across hundreds of branches is
    the hard part. Threats: malware propagation, unauthorized access, WAN sniffing, MitM.
- SAFE also defines **secure domains** — the operational areas used to evaluate each PIN:
  **Management**, **Security intelligence**, **Compliance** (PCI DSS 3.0, HIPAA),
  **Segmentation**, **Threat defense**, **Secure services**.
- The **SAFE key** (Figure 25-1) is the visual: the ring of PINs is the key's bow, the
  secure domains are the teeth.
- SAFE covers the **full attack continuum** for all PINs:
  - **Before** — know every asset and the threats against it; establish policy and
    prevention. Tools: Secure Firewalls, NAC, identity services.
  - **During** — abilities and actions when an attack gets through: threat analysis and
    incident response. Tools: NGIPS, NGFW, malware protection, email/web security.
  - **After** — detect, contain, remediate; feed lessons learned back into the design.
    Tools: AMP, NGFW, Secure Network Analytics for scope/contain/remediate.
- **SAFE is about integrating security services within each PIN.** For the underlying
  networking design and implementation guidance, the companion is the **Cisco Validated
  Design (CVD)** guides at `www.cisco.com/go/cvd`.

**Cisco Talos — the intelligence source**
- **Cisco Talos** is the Cisco threat intelligence organization. It is *not* a product —
  it is the feed that every other product in the portfolio consumes.
- Formed from three teams: **IronPort Security Applications (SecApps)**, the **Sourcefire
  Vulnerability Research Team (VRT)**, and the **Cisco Threat Research, Analysis, and
  Communications (TRAC)** team.
- Daily scale: ~**16 billion web requests**, **600 billion emails**, **1.5 million unique
  malware samples**.
- Intelligence feeds: advanced Microsoft and industry disclosures; the AMP community;
  ClamAV, Snort, Immunet, SpamCop, SenderBase, Secure Malware Analytics and Talos user
  communities; honeypots; the Sourcefire AEGIS program; private and public threat feeds;
  dynamic analysis.

**Cisco Secure Malware Analytics (formerly Threat Grid) — the sandbox**
- Performs **static file analysis** (filenames, MD5 checksums, file types) *and* **dynamic
  file analysis** — a.k.a. **behavioral analysis** — by detonating files in a controlled,
  monitored sandbox and comparing against millions of samples and billions of artifacts.
- Behavioral analysis is combined with Talos intelligence and existing security tech.
- Malware commonly contains code to detect that it is running in a virtual sandbox and
  refuses to run. **Malware Analytics evades that detection by not having the typical
  instrumentation** a sandbox normally exposes.
- **Glovebox** is the sandbox environment where an analyst can interact with a suspicious
  file directly and watch its behavior.
- Available as an appliance and in the cloud; integrated into other Cisco Secure products
  and third-party solutions. Automatic sample submission exists for integrated products;
  otherwise files are uploaded manually.

**Cisco AMP (Advanced Malware Protection / Malware Defense) — beyond point-in-time**
- Point-in-time detection is **completely blind to the scope and depth of a breach after
  it happens**. AMP exists to cover that gap.
- Across the attack continuum:
  - **Before** — global threat intelligence from Talos and Secure Malware Analytics.
  - **During** — file reputation (clean/malicious verdict) plus sandboxing.
  - **After** — **retrospection**, **indicators of compromise (IoCs)**, breach detection,
    tracking, analysis, and surgical remediation.
- Architecture: **AMP Cloud** (private or public) + **AMP connectors** + threat
  intelligence from Talos and Secure Malware Analytics.
- AMP connectors: **Cisco Secure Endpoint** (formerly FireAMP / AMP for Endpoints —
  Windows, macOS X, Android, iOS, Linux), **Cisco Secure Email** (formerly AMP for Email /
  ESA), **Cisco Secure Web Appliance** (formerly AMP for Web / WSA), **AMP for Networks**
  (on Secure Firewall appliances and dedicated AMP appliances), **AMP for Meraki MX**.
- **AMP Cloud is the most important component** — it holds the database of files and their
  reputations (malware, clean, unknown, custom), called **file dispositions**. A
  disposition can *change* based on new data from Talos or Malware Analytics.
- Flow: connector uploads a sample → if malicious, stored in the cloud and reported to
  every connector that sees the same file → if **unknown**, sent to Malware Analytics for
  sandbox behavioral analysis.
- Connectors stay lightweight by **sending a hash, not the file**, and letting the cloud
  return the verdict (clean / malicious / unknown). This is what separates AMP from
  signature-database antivirus.
- On Apple iOS, Secure Endpoint is packaged as the **Cisco Security Connector (CSC)**,
  which incorporates Secure Endpoint **and** Umbrella.

**Cisco Secure Client (formerly AnyConnect Secure Mobility Client)**
- Modular endpoint software — **not just a VPN client**. VPN access over TLS/SSL and IPsec
  IKEv2, plus built-in modules.
- **VPN Posture (HostScan)** module and **ISE Posture** module assess endpoint compliance
  (antivirus, antispyware, host firewall). Noncompliant endpoints can have network access
  restricted until remediated.
- Also provides web security via Cisco Cloud Web Security, network visibility into endpoint
  flows for Secure Network Analytics, and roaming protection with Umbrella **even when the
  VPN is off**.
- Platforms: Windows, macOS, iOS, Linux, Android, Windows Phone/Mobile, BlackBerry, ChromeOS.
- Terminology note: SSL is deprecated by the IETF in favor of TLS, so "TLS/SSL" should be
  read as TLS.

**Cisco Umbrella (formerly OpenDNS) — DNS-layer first line of defense**
- Blocks requests to malicious destinations (domains, IPs, URLs) **using DNS, before an IP
  connection is established or a file is downloaded**. 100% cloud delivered — no hardware,
  no software to maintain.
- 40+ data centers using **Anycast DNS** to guarantee 100% uptime; traffic routes to the
  closest location regardless of site.
- Intelligence from ~**500 billion daily DNS requests** from 90+ million users, fed in real
  time into a graph database with statistical and ML models, supplemented by Talos.
- Corporate deployment is as simple as **changing DHCP on all Internet gateways** (routers,
  APs) so every device — including guests — forwards DNS to Umbrella.
- Off-network laptops: either enable the **roaming security module** in Cisco Secure Client
  (no additional agent required), or deploy the standalone **Umbrella roaming client**,
  which tags, encrypts, and forwards Internet-bound DNS queries.

**Cisco Secure Web Appliance (WSA) — the web gateway**
- All-in-one web gateway; leverages real-time intelligence from the AMP Threat Intelligence
  Cloud. Provides multiple layers of malware defense plus DLP.
- **Before an attack:**
  - **Web reputation filters** — Talos refreshes reputation data **every three to five
    minutes**; analyzes **200+** web-traffic and network parameters (domain owner, hosting
    server, site age, site type) and assigns a **reputation score from -10 to +10** rather
    than a binary good/bad. The score plus policy decides block / allow / warn.
  - **Web filtering** — traditional URL filtering against a Cisco database of **50+ million
    blocked sites**, combined with the **Dynamic Content Analysis (DCA) engine**, which
    identifies inappropriate content in real time for **90% of unknown URLs** by scanning
    text, scoring relevancy, calculating model document proximity, and returning the closest
    category match. Talos updates the URL filtering DB every 3–5 minutes from firewalls,
    IPS, web, email, and VPNs.
  - **Cisco AVC (Application Visibility and Control)** — classifies the most widely used web
    and mobile applications plus **150,000+ micro-applications** (e.g. Facebook Messenger).
    Granular enough to permit Facebook/YouTube while blocking "Like" clicks or specific
    videos/channels.
- **During an attack:** cloud access security via **CASB** partners (e.g. Cisco CloudLock);
  **parallel AV scanning** (multiple engines simultaneously on one appliance); **Layer 4
  traffic monitoring** scanning all traffic/ports/protocols to catch spyware "phone-home"
  and identify infected clients; **file reputation and analysis with AMP** (fingerprints
  each file traversing the gateway, sends to AMP Cloud for a zero-day verdict); **DLP** via
  **ICAP** integration with third-party DLP appliances.
- **After an attack:** continuous inspection using **AMP retrospection** — keeps scanning
  files over an extended period and **alerts when a file disposition changes** (unknown →
  malware). **Global Threat Analytics (GTA)**, formerly Cognitive Threat Analytics (CTA),
  analyzes web traffic + Secure Endpoint data + Secure Network Analytics data and uses ML to
  spot malicious activity before exfiltration.
- Deployable in the cloud, as a virtual appliance, on-premises, or hybrid — **all features
  available across any deployment option**.

**Cisco Secure Email (ESA)**
- Email is the top attack vector for breaches. Multilayered protection across the continuum:
  - **Global threat intelligence** from Talos and Secure Malware Analytics.
  - **Reputation filtering** — blocks unwanted email based on Talos reputation.
  - **Spam protection** — the **Context Adaptive Scanning Engine (CASE)**; **>99% catch
    rate**, **<1 in 1,000,000 false positives**.
  - **Forged email detection** — protects high-value targets (executives) against **BEC**.
  - **Cisco Advanced Phishing Protection (CAPP)** — Talos intelligence + local email
    intelligence + ML to model trusted email behavior and stop identity-deception attacks.
  - **Cisco Domain Protection (CDP)** — prevents phishing emails being sent *using the
    customer's own domain*.
  - **Malware defense**, **graymail detection and Safe Unsubscribe** (graymail = marketing /
    social / bulk mail whose unsubscribe link may itself be phishing).
  - **URL-related protection** — filtering and scanning of URLs in attachments and shortened
    URLs.
  - **Outbreak filters** — **rewrite URLs** in suspicious messages so a click redirects the
    recipient to the Secure Web Appliance, which scans the content live and shows a block
    screen if malicious.
  - **Web interaction tracking** — reports on top users who clicked malicious URLs, top
    malicious URLs clicked, and date/time/rewrite reason/action taken.
  - **Data security for outgoing mail** — 100+ expert policies trigger encryption, footers,
    disclaimers, BCCs, notifications, quarantining.
- Available as a hardware appliance or the cloud offering **Cisco Secure Email Threat
  Defense**.

**Cisco Secure IPS (FirePOWER NGIPS)**
- **IDS** = passively monitors and analyzes traffic and logs intrusion data. **IPS** = IDS
  functions **plus** automatic blocking.
- Gartner's **NGIPS** definition requires: real-time contextual awareness, advanced threat
  protection, intelligent security automation, unparalleled performance and scalability,
  AVC and URL filtering.
- Cisco acquired **Sourcefire in 2013**; FirePOWER NGIPS is now Cisco Secure IPS, and it
  **exceeds** the Gartner NGIPS bar:
  - **Real-time contextual awareness** — discovers applications, users, endpoints, OSs,
    vulnerabilities, services, processes, network behaviors, files, threats.
  - **Advanced threat protection and remediation** — integrated AMP for Networks and Secure
    Malware Analytics sandboxing.
  - **Intelligent security automation** — correlates threat events, contextual info, and
    vulnerability data to automate policy updates, identify users hit by client-side
    attacks, alert on config-policy violations, baseline normal traffic to detect malware
    spread, and tag potentially compromised hosts with an **IoC**.
  - **Performance** — low-latency single-pass design on Secure Firewall and ASA appliances.
  - **AVC** — detection of **4000+** commercial applications, custom app support.
  - **URL filtering** — **80+ categories**, **280+ million** individual URLs.
- Beyond the NGIPS definition: **centralized management via FMC**; global threat intelligence
  from Talos for signature and URL updates; the **Snort** detection engine; an open API
  third-party ecosystem; and **ISE integration** so FMC can remediate compromised hosts —
  **Quarantine** (limit/block endpoint access), **Unquarantine**, **Shutdown** (shut the port
  the endpoint is on).

**Cisco Secure Firewall (NGFW)**
- A **firewall** monitors traffic and allows/blocks by simple packet filtering and stateful
  inspection on ports and protocols, establishing a barrier between trusted internal and
  untrusted external networks.
- Gartner's **NGFW** definition requires: standard firewall capabilities such as stateful
  inspection, an integrated IPS, application-level inspection, and the ability to leverage
  external security intelligence.
- Cisco integrated ASA software with Secure IPS services software, producing the Firepower
  NGFW (now Cisco Secure Firewall) — the industry's first fully integrated, threat-focused
  NGFW with unified management.
- Form factors: Secure Firewall Appliances; Secure Industrial Security Appliance (ISA);
  Secure Firewall Threat Defense Virtual; Secure Firewall Cloud Native; Secure Web
  Application Firewall (WAF) and bot protection; all ASA 5500-X appliances **except the
  5585-X**.
- Software images:
  - **ASA software image** — standard legacy firewall, no Secure IPS services. Supported on
    all Secure Firewall and ASA appliances.
  - **ASA image + Secure IPS image (FirePOWER NGIPS)** — two images in one appliance, each
    needing a different management application. Supported **only on 5500-X (except 5585-X)**.
  - **Firepower Threat Defense (FTD) software image** — merges the ASA image and the Secure
    IPS image into a **single unified image**. Supported on all Secure Firewall and ASA
    5500-X appliances **except the 5585-X**.
- FTD also runs on **ISR modules** and on Threat Defense Virtual / Cloud Native in VMware,
  KVM, AWS, GCP, HyperFlex, Nutanix, OpenStack, Alibaba Cloud, and Azure.
- Management — for FTD or Secure IPS Services software: **Cisco SecureX**, **FMC**, **FDM**
  (small appliances). For ASA software: **CLI**, **CSM**, **ASDM**, **Cisco Defense
  Orchestrator**.
- **Capitalization matters in Cisco docs:** **FirePOWER** (uppercase) = the Secure IPS
  (NGIPS) services software or the NGIPS services ASA module. **Firepower** (lowercase) =
  Cisco Secure Firewall or the FTD unified image.
- **FTD / Secure IPS Services software CLI configuration is not supported** — the CLI exists
  only for initial setup and troubleshooting.

**Cisco Secure Firewall Management Center (FMC)**
- Centralized platform aggregating and correlating threat events, contextual information,
  and network device performance data; single pane of glass for event collection and policy.
- Performs event and policy management for: Cisco Secure Firewall (physical) and Firewall
  Threat Defense (virtual), Cisco FTD for ISR, and Cisco ASA with FirePOWER Services.

**Cisco Secure Network Analytics (formerly Stealthwatch Enterprise)**
- A **collector and aggregator of network telemetry** performing security analysis and
  monitoring. Detects threats that infiltrate the network *and* threats that originate
  inside it: C&C attacks, ransomware, DDoS, illicit cryptomining, unknown malware, insider
  threats.
- **Agentless.** Scales into the cloud (with Secure Cloud Analytics), across the network,
  to branches, into the data center, and down to endpoints. **Detects malware in encrypted
  traffic and ensures policy compliance without decryption.**
- Core components:
  - **Network Analytics Manager** (formerly Stealthwatch Management Console, SMC) — the
    control center; aggregates and presents analysis from up to **25 Flow Collectors**, ISE,
    and other sources. Hardware appliance or VM.
  - **Flow Collectors** — collect and analyze NetFlow, **IPFIX**, and other flow data from
    routers, switches, firewalls, endpoints, and proxy data sources (which GTA can analyze).
    Pinpoint malicious patterns in encrypted traffic with **Encrypted Traffic Analytics
    (ETA)** without decrypting. Appliance or VM.
  - **Flow Rate License** — required for collection, management, and analysis of flow
    telemetry; aggregates flows at the Manager and defines the volume of flows collectible.
- Optional but recommended:
  - **Flow Sensors** — produce telemetry for segments that **can't generate NetFlow**, and
    add application-layer visibility.
  - **UDP Director** — receives UDP data streams from multiple locations and forwards them
    as a single stream to one or more destinations. Instead of every router exporting
    NetFlow to Flow Collectors *and* LiveAction *and* Arbor, each router exports once to the
    UDP Director, which replicates.
  - **Cisco Telemetry Broker** — like the UDP Director, but can also **ingest** telemetry
    (e.g. AWS VPC Flow Logs), **filter** unneeded data, and **transform** it into a format
    (e.g. IPFIX) the consumer understands — Secure Network Analytics, Splunk, etc.
  - **Data Store** — centralized flow/telemetry storage instead of distributed across Flow
    Collectors. Greater capacity, greater flow rate, more resiliency. **Minimum three Data
    Node appliances.**
  - **Threat Feed License** (Talos threat intelligence feed), **Endpoint License** (extends
    visibility into endpoints), **Secure Cloud Analytics** (extends visibility into AWS,
    GCP, Azure).
- Benefits: real-time threat detection; incident response and forensics; network
  segmentation; network performance and capacity planning; satisfying regulatory
  requirements.

**Cisco Secure Cloud Analytics (formerly Stealthwatch Cloud)**
- Cloud-based **SaaS** solution providing visibility and continuous threat detection for
  on-premises, hybrid, and multicloud. Detects malware, ransomware, data exfiltration,
  network vulnerabilities, and role changes that indicate compromise.
- Two deployment models:
  - **Public Cloud Monitoring** (formerly Stealthwatch Cloud Public Cloud Monitoring) —
    visibility and threat detection in AWS, GCP, Azure. **Agentless**, relying on native
    telemetry such as **VPC flow logs**. Models all IP traffic inside VPCs, between VPCs, and
    to external IPs. Integrates with CloudTrail, CloudWatch, AWS Config, Inspector, IAM,
    Lambda, and more.
  - **Cisco Secure Network Analytics SaaS** (formerly Stealthwatch Cloud Private Network
    Monitoring) — visibility and threat detection for the **on-premises** network delivered
    from the cloud. A lightweight virtual appliance consumes native telemetry or extracts
    metadata from packet flow; metadata is encrypted and sent to the platform.
- **Consumes metadata only — actual packet payloads are never retained or transferred
  outside the network.**

**Cisco Identity Services Engine (ISE) — the policy brain**
- Security **policy management platform** providing NAC to users and devices across
  **wired, wireless, and VPN**. Gives visibility into who is connected, which applications
  are installed and running (for posture), and more.
- Key capabilities:
  - **Streamlined network visibility** — stores a detailed attribute history of all devices,
    endpoints, and users (guests, employees, contractors).
  - **Cisco DNA Center integration** — applies TrustSec software-defined segmentation
    through SGT tags and SGACLs.
  - **Centralized secure network access control** — **RADIUS**, required to enable
    802.1x/EAP, MAB, and local/centralized WebAuth consistently across wired, wireless, VPN.
  - **Centralized device access control** — **TACACS+** for AAA device administration
    (covered in Chapter 26).
  - **Cisco TrustSec** — implements TrustSec policy via SGTs, SGACLs, and SXP.
  - **Guest lifecycle management** — customizable, brandable guest web portals for WebAuth.
  - **Streamlined device onboarding** — automates 802.1x supplicant provisioning and
    certificate enrollment; integrates with MDM/EMM vendors.
  - **Internal certificate authority** — ISE can act as its own CA.
  - **Device profiling** — automatically detects, classifies, and associates endpoints to
    endpoint-specific authorization policies based on device type.
  - **Endpoint posture service** — audits for latest OS patch, endpoint firewall enabled,
    anti-malware definitions current, disk encryption, mobile PIN lock, rooted/jailbroken
    status. Noncompliant devices are remediated (blocked from network/apps/services) until
    compliant. Also provides hardware inventory of every connected device.
  - **Active Directory support** — AD 2012, 2012R2, 2016, 2019.
  - **Cisco Platform Exchange Grid (pxGrid)** — shares contextual information over a single
    API between Cisco platforms and 50+ technology partners. An **IETF framework** for
    automatically identifying, containing, mitigating, and remediating threats.
    **ISE is the central pxGrid controller (pxGrid server)**; all Cisco and third-party
    platforms are **pxGrid nodes** that publish, subscribe, and query.
    - **pxGrid 1.0** — released with ISE 1.3, based on **XMPP**. From **ISE 3.1 all pxGrid
      connections must be pxGrid 2.0**.
    - **pxGrid 2.0** — **WebSocket** and REST API over **STOMP 1.2**.
- The ISE **session directory** context shared over pxGrid includes: session IP, Audit
  Session Id, UserName, AD domain/NetBIOS/resolved identities and DNs, MAC addresses,
  State, **ANCstatus** (e.g. `ANC_Quarantine`), **SecurityGroup** (e.g.
  `Quarantined_Systems`), EndpointProfile, NAS IP, NAS Port, RADIUS AV pairs, Posture
  Status/Timestamp, LastUpdateTime, Authorization_Profiles.

**802.1x — port-based network access control**
- IEEE **802.1x** (Dot1x) is the standard for **port-based network access control (PNAC)**,
  providing an authentication mechanism for LANs and WLANs.
- Components:
  - **EAP (Extensible Authentication Protocol)** — the message format and framework defined
    by **RFC 4187** providing encapsulated transport for authentication parameters.
  - **EAP method** (a.k.a. EAP type) — the actual authentication method carried in EAP.
  - **EAPoL (EAP over LAN)** — the Layer 2 encapsulation defined by 802.1x for transporting
    EAP over IEEE 802 wired and wireless networks.
  - **RADIUS** — the AAA protocol used by EAP.
- Roles:
  - **Supplicant** — software on the endpoint that provides identity credentials via EAPoL
    to the authenticator. Windows and macOS native supplicants and Cisco Secure Client all
    support **machine and user** authentication.
  - **Authenticator** — the network access device (**NAD**), a switch or WLC, controlling
    access based on authentication status. It takes Layer 2 EAP-encapsulated packets from
    the supplicant and encapsulates them into RADIUS for the authentication server.
  - **Authentication server** — a RADIUS server that validates the endpoint's identity and
    returns an authorization result (accept/deny).
- **The EAP identity exchange and authentication happen between the supplicant and the
  authentication server.** The authenticator has **no idea what EAP type is in use** — it
  just relays EAPoL↔RADIUS and opens the port when told. EAP authentication is completely
  transparent to the authenticator.

**EAP methods**
- Most are based on TLS. Categories:
  - **Challenge-based:** EAP-MD5.
  - **TLS:** EAP-TLS.
  - **Tunneled TLS (outer methods):** EAP-FAST, EAP-TTLS, PEAP.
  - **Inner methods:** EAP-GTC, EAP-MSCHAPv2, EAP-TLS.
- Inner methods are tunneled *within* PEAP, EAP-FAST, and EAP-TTLS — the **outer** or
  **tunneled TLS** methods. The outer method establishes a TLS tunnel between supplicant and
  authentication server; credentials are then negotiated inside it. Analogous to HTTPS: the
  browser validates the site certificate (one-way trust), the tunnel forms, then the user
  types credentials through it.
- Per method:
  - **EAP-MD5** — MD5-hashes the credentials; hash compared against a local hash on the
    server. **No mutual authentication** — the server validates the supplicant, but the
    supplicant never validates the server. That makes it a poor choice.
  - **EAP-TLS** — TLS **PKI certificate** authentication, **mutual** in both directions.
    Both supplicant and authentication server need a certificate signed by a mutually
    trusted CA. **Most secure**, but **most difficult to deploy** because of the
    administrative burden of a certificate on every supplicant.
  - **PEAP** — **only the authentication server requires a certificate**, which cuts the
    administrative burden. Forms an encrypted TLS tunnel, then uses an inner method:
    - **EAP-MSCHAPv2 (PEAPv0)** — credentials sent encrypted inside an MSCHAPv2 session.
      **The most common inner method**, because it allows simple transmission of
      username/password (or computer name/password) to RADIUS for AD authentication.
    - **EAP-GTC (PEAPv1)** — created by Cisco as an alternative to MSCHAPv2, allowing
      generic authentication to virtually any identity store: OTP token servers, LDAP,
      NetIQ eDirectory, and more.
    - **EAP-TLS** — most secure EAP authentication (a TLS tunnel inside another TLS tunnel),
      rarely used due to the certificate-on-every-supplicant deployment complexity.
  - **EAP-FAST** — Cisco-developed, similar to PEAP, for **faster re-authentication** and
    faster wireless roaming. Forms a TLS outer tunnel and sends credentials within it. The
    major difference from PEAP is re-authenticating faster using **protected access
    credentials (PACs)** — a PAC is like a secure cookie stored locally on the host as proof
    of a successful authentication. **EAP-FAST also supports EAP chaining.**
  - **EAP-TTLS** — similar in function to PEAP but **not as widely supported**. The major
    difference: PEAP supports **only EAP inner methods**, while EAP-TTLS also supports
    non-EAP legacy inner methods — **PAP, CHAP, MS-CHAP**.

**EAP chaining**
- **EAP-FAST includes the option of EAP chaining**, which supports **machine and user
  authentication inside a single outer TLS tunnel**, combining them into a **single overall
  authentication result**.
- The point: it lets you grant greater privileges (or apply different posture assessments)
  to users who connect **from corporate-managed devices** — because you can prove both the
  machine *and* the user in one result.

**MAC Authentication Bypass (MAB)**
- Port-based access control using the **MAC address** of the endpoint, typically as a
  **fallback mechanism to 802.1x** for devices with no supplicant (printers, cameras, badge
  readers). A MAB-enabled port is dynamically enabled or disabled based on the MAC that
  connects.
- **MAB is not more secure than 802.1x — it is less.** MAC addresses are trivially spoofed.
  MAB-authenticated endpoints **should be given very restricted access**, limited to only
  the networks and services they actually need.
- Authorization options a Cisco switch can apply from the RADIUS result: **downloadable ACLs
  (dACLs)**, **dynamic VLAN assignment (dVLAN)**, and **SGT tags**.

**Web Authentication (WebAuth)**
- For endpoints with no 802.1x supplicant and no known MAC for MAB: employees/contractors
  with misconfigured 802.1x, or visitors and guests needing Internet access.
- Like MAB, WebAuth is a **fallback** for 802.1x. If both MAB and WebAuth are configured as
  fallbacks, on 802.1x timeout the switch tries **MAB first**, then **WebAuth**.
- Endpoints get a web portal requesting username/password; the switch (or WLC/firewall) sends
  those to RADIUS in a standard access-request **on behalf of the endpoint** — the endpoint
  is not authenticating directly to the switch.
- **Unlike MAB, WebAuth is only for users, not devices** — it requires a browser and manual
  credential entry.
- Two types:
  - **Local Web Authentication (LWA)** — the first form created. The switch/WLC redirects
    HTTP/HTTPS to a **locally hosted portal running on the switch**. When credentials are
    submitted the switch sends the RADIUS access-request. **The switch sending credentials
    on behalf of the user is what makes it LWA.**
    - On Cisco switches, **LWA portals are not customizable** — a blocker for organizations
      that require corporate branding.
    - No native support for advanced services: AUP acceptance pages, password changing,
      device registration, self-registration.
    - **LWA does not support VLAN assignment — only ACL assignment.** It also **does not
      support CoA**, so access policy cannot change based on posture or profiling state, and
      an admin cannot quarantine an endpoint after a malware event.
  - **Central Web Authentication (CWA) with Cisco ISE** — created to overcome LWA's
    deficiencies. **Supports CoA** for posture profiling, plus **dACL and VLAN**
    authorization options, and all advanced services: client provisioning, posture
    assessments, AUPs, password changing, self-registration, device registration.
    - Like LWA, CWA requires a browser and manual credentials. With CWA, **WebAuth and guest
      VLAN remain mutually exclusive**.
- **Guest VLAN and LWA are mutually exclusive.** Cisco and many third-party 802.1x switches
  can assign a guest VLAN to endpoints without a supplicant; many production deployments
  still use this legacy option for wired guest Internet access.

**Enhanced FlexAuth and IBNS 2.0**
- By default a Cisco switch configured with 802.1x, MAB, and WebAuth always tries **802.1x
  first, then MAB, then WebAuth** — so a non-802.1x endpoint waits a considerable time
  before WebAuth is even offered.
- **Enhanced FlexAuth** (a.k.a. **Access Session Manager**) fixes this by allowing **multiple
  authentication methods concurrently** (e.g. 802.1x and MAB), bringing endpoints online
  faster.
- **Cisco IBNS 2.0 (Identity-Based Networking Services)** is the integrated solution
  offering authentication, access control, and user policy enforcement with a common
  end-to-end policy for **wired and wireless**. It is the combination of: **Enhanced
  FlexAuth (Access Session Manager)**, **Cisco Common Classification Policy Language
  (C3PL)**, and **Cisco ISE**.

**Cisco TrustSec**
- Next-generation access control enforcement addressing the operational cost of traditional
  VLAN-based segmentation and hand-maintained firewall rules/ACLs, using **Security Group
  Tags (SGTs)**.
- TrustSec uses SGTs to perform **ingress tagging** and **egress filtering**. ISE assigns
  SGTs to users/devices successfully authenticated and authorized via **802.1x, MAB, or
  WebAuth**; the SGT is delivered to the authenticator as an authorization option, exactly
  like a dACL. Once assigned, allow/drop policy can be applied at **any egress point**.
- SGTs are called **scalable group tags** in Cisco SD-Access.
- SGTs represent the **context** of the user, device, use case, or function, so they are
  named after roles or business use cases (e.g. `Mac_Corporate` for a compliant corporate
  Mac authenticated via 802.1x with EAP chaining; `Mac_Guest` if not posture-compliant).
- **Endpoints are not aware of the SGT tag.** The tag is only known and applied within the
  network infrastructure.
- The SGT **name** is used on ISE and network devices to build policy; what is actually
  inserted into a Layer 2 frame is a **numeric value** (shown in decimal/hex on ISE).
- TrustSec configuration has **three phases**: **ingress classification → propagation →
  egress enforcement**.

**Phase 1 — Ingress classification**
- Assigning SGTs to users, endpoints, or resources as they ingress the TrustSec network.
- **Dynamic assignment** — SGT downloaded as an authorization option from ISE when
  authenticating via 802.1x, MAB, or WebAuth.
- **Static assignment** — for environments like a data center that don't do 802.1x/MAB/
  WebAuth, so dynamic assignment isn't possible. SGTs are statically mapped on SGT-capable
  devices as: **IP to SGT**, **subnet to SGT**, **VLAN to SGT**, **Layer 2 interface to
  SGT**, **Layer 3 logical interface to SGT**, **port to SGT**, **port profile to SGT**.
- As an alternative to per-port assignment, ISE can centrally hold a database of IP-to-SGT
  mappings that SGT-capable devices **download from ISE**.

**Phase 2 — Propagation**
- Communicating the mappings to the TrustSec devices that will enforce policy. Two methods:
- **Inline tagging (native tagging)** — the switch inserts the SGT **inside the frame** so
  upstream devices can read and apply policy. It is **completely independent of any Layer 3
  protocol (IPv4 or IPv6)**, so the tag survives across routers, switches, and firewalls to
  the egress point.
  - Frame layout: `DMAC | SMAC | 802.1Q | CMD | ETYPE | PAYLOAD | CRC`, where **CMD (Cisco
    Metadata)** carries `CMD EtherType 0x8909 | Ver | Len | SGT Option + Len | **SGT Value**
    | Other Options`. The SGT value is **16-bit — a 64K name space**.
  - **Downside:** supported only by Cisco devices with **ASIC support for TrustSec**. If a
    tagged frame reaches a device that does not support native tagging in hardware, **the
    frame is dropped**.
- **SXP (SGT Exchange Protocol) propagation** — a **TCP-based peer-to-peer** protocol for
  devices that **don't support inline tagging in hardware**. Communicates **IP-to-SGT
  mappings** from non-inline-tagging switches to other network devices, which keep an SGT
  mapping database to check packets against and enforce policy.
  - The peer that **sends** IP-to-SGT bindings is the **speaker**; the peer that **receives**
    them is the **listener**. SXP connections can be **single-hop or multi-hop** (a device
    can be listener on one side and speaker on the other).
  - **ISE itself can be an SXP speaker** — if a user authenticates via 802.1x to a switch
    that supports neither inline tagging nor SXP, ISE sends the mapping over SXP to an
    upstream TrustSec-capable device. ISE can also push SGT mapping information upstream via
    **pxGrid**.

**Phase 3 — Egress enforcement**
- Once SGTs are assigned (classification) and transmitted (propagation), policy is enforced
  at the **egress point** of the TrustSec network. Two enforcement types:
  - **SGACL (Security Group ACL)** — enforcement on **routers and switches**; filters based
    on **source and destination SGT**.
  - **SGFW (Security Group Firewall)** — enforcement on **Cisco Secure Firewalls**; requires
    **tag-based rules defined locally on the firewall**.
- The ISE **production matrix** visualizes SGACL enforcement: left column = source SGTs, top
  row = destination SGTs, the cell where they meet is the ACL enforced. Direction is always
  **source SGT → destination SGT**. `Permit IP` = permit all; `Deny IP` = deny all.
- More granular SGACLs are supported, not just permit-all/deny-all — e.g. a `Permit_FTP`
  SGACL containing `permit tcp eq 21` / `deny ip`, applied employee→employee.
- SGACL policies deliver TrustSec software-defined segmentation to **wired, wireless, and
  VPN**, all centrally managed through ISE, as an alternative to traditional VLAN-based
  segmentation. **Traffic is blocked on egress, not ingress** — and for endpoints on the
  same switch, that switch is both the ingress and the egress point (so intra-VLAN
  enforcement works).

**MACsec**
- **MACsec** is the IEEE **802.1AE** standards-based **Layer 2 hop-by-hop encryption**
  method. Traffic is encrypted **only on the wire between two MACsec peers** and is
  **unencrypted as it is processed internally within the switch**.
- That "decrypt at each hop" property is the *point*: it lets the switch look into the inner
  packets for things like **SGT tags** to perform enforcement or QoS prioritization —
  something end-to-end encryption would prevent.
- MACsec uses **onboard ASICs** for encryption/decryption rather than offloading to a crypto
  engine as IPsec does.
- Frame format: Ethernet plus an additional **16-byte MACsec Security Tag (802.1AE header)**
  and a **16-byte Integrity Check Value (ICV)**. **All devices in the flow of MACsec
  communications must support MACsec** for these fields to be used.
- Provides authentication using **GMAC (Galois Message Authentication Code)** or
  authenticated encryption using **AES-GCM (Galois/Counter Mode AES)**.
- In the full frame, the 802.1AE header is **authenticated** along with DMAC/SMAC, while
  802.1Q, CMD (with the SGT), ETYPE, and PAYLOAD are **encrypted**; ICV and CRC trail.
- MACsec Security Tag fields:
  - **MACsec EtherType** (octets 1–2) — **0x88e5**, designating the frame as MACsec.
  - **TCI/AN** (octet 3) — Tag Control Information / Association Number, designating the
    version number if confidentiality or integrity is used on its own.
  - **SL** (octet 4) — Short Length, the length of the encrypted data.
  - **Packet Number** (octets 5–8) — for replay protection and building the initialization
    vector.
  - **SCI** (octets 9–16) — Secure Channel Identifier, classifying the connection to the
    virtual port.
- Two keying mechanisms:
  - **SAP (Security Association Protocol)** — **proprietary Cisco** keying protocol used
    **between Cisco switches**.
  - **MKA (MACsec Key Agreement protocol)** — provides session keys and manages encryption
    keys. 802.1AE with MKA is supported **between endpoints and the switch** as well as
    **between switches**.
- **Downlink MACsec** — the encrypted link **between an endpoint and a switch**, keyed by
  **MKA**. Requires a MACsec-capable switch and a MACsec-capable supplicant (e.g. Cisco
  Secure Client). Endpoint-side encryption may be hardware (if the endpoint has it) or
  software using the main CPU.
  - The switch can **force encryption, make it optional, or force non-encryption**. Set
    manually per port (uncommon) or **dynamically as an authorization option from ISE**
    (much more common). **If ISE returns an encryption policy with the authorization result,
    the ISE policy overrides anything set on the switch CLI.**
- **Uplink MACsec** — encrypting a link **between switches** with 802.1AE. **By default
  uplink MACsec uses Cisco proprietary SAP encryption.** The encryption itself is the same
  **AES-GCM-128** used by both uplink and downlink MACsec. Uplink MACsec may be achieved
  manually or dynamically; **dynamic MACsec requires 802.1x authentication between the
  switches**.

## Procedure

**Successful 802.1x authentication (Figures 25-5, 25-6):**
1. **Initiation.** When the authenticator notices a port coming up it starts authentication
   by sending periodic **EAP-request/identity** frames. The supplicant can also initiate by
   sending an **EAPoL-start** to the authenticator.
2. **Authentication.** The authenticator relays EAP messages between supplicant and
   authentication server, copying the EAP message in the EAPoL frame into an **AV-pair
   inside a RADIUS packet** and vice versa, until an EAP method is selected. Multiple
   Access-Challenge / Access-Request exchanges are possible. Authentication then takes place
   using the selected EAP method.
3. **Authorization.** On success the authentication server returns a **RADIUS Access-Accept**
   with an encapsulated **EAP-Success** plus an authorization option such as a **dACL**
   (and/or VLAN, SGT). The authenticator then **opens the port**.

**Successful MAB authentication (Figure 25-7):**
1. **802.1x timeout.** The switch initiates authentication by sending an EAPoL identity
   request to the endpoint **every 30 seconds by default**. After **three timeouts (90
   seconds by default)** the switch determines the endpoint has no supplicant and proceeds
   to MAB.
2. **MAC authentication.** The switch opens the port to accept a **single packet** from
   which it learns the source MAC. Packets sent **before** the port fell back to MAB (i.e.
   during the 802.1x timeout phase) are **discarded immediately and cannot be used** to
   learn the MAC. After learning the source MAC the switch **discards the packet** and
   crafts a RADIUS access-request using the endpoint's MAC as the identity.
3. **Authorization.** The RADIUS server determines whether the device is granted access and
   at what level, and sends an access-accept to the authenticator. It can include
   authorization options such as **dACLs, dVLANs, and SGT tags**.
- **If 802.1x is not enabled**, the sequence is the same except MAB starts **immediately
  after linkup** instead of waiting for 802.1x to time out.

**Central Web Authentication (CWA) with ISE:**
1. The endpoint entering the network has **no configured supplicant, or a misconfigured one**.
2. The switch performs **MAB**, sending the RADIUS access-request to ISE.
3. ISE sends the RADIUS result **including a URL redirection** to the centralized portal on
   the ISE server itself.
4. The endpoint is assigned an IP address, DNS server, and default gateway via **DHCP**.
5. The end user opens a browser and enters credentials into the centralized portal. **Unlike
   LWA, the credentials are stored in ISE and are tied together with the MAB coming from the
   switch.**
6. ISE sends a **re-authentication change of authorization (CoA-reauth)** to the switch.
7. The switch sends a **new MAB request with the same session ID** to ISE. ISE sends the
   final authorization result for the end user, including an authorization option such as a
   **dACL**.

**TrustSec deployment (the three phases):**
1. **Ingress classification** — assign SGTs at the network edge, dynamically from ISE via
   802.1x/MAB/WebAuth, or statically (IP/subnet/VLAN/interface/port/port-profile to SGT) in
   environments like the data center where no endpoint authentication occurs.
2. **Propagation** — carry the tag to the enforcement point: **inline tagging** where every
   device in the path has TrustSec ASIC support, or **SXP** (speaker → listener,
   single- or multi-hop) where it does not. ISE itself can speak SXP or push mappings over
   pxGrid when even the access switch is incapable.
3. **Egress enforcement** — apply **SGACLs** on routers/switches or **SGFW** rules on Secure
   Firewalls, source SGT → destination SGT, at the egress point.

**Attack-continuum design method (Cisco SAFE):**
1. **Before** — inventory every asset that must be protected and identify the threats that
   could target them; establish policy and implement prevention to reduce risk (Secure
   Firewalls, NAC, identity services).
2. **During** — define the abilities and actions required when an attack gets through:
   threat analysis and incident response (NGIPS, NGFW, malware protection, email/web
   security).
3. **After** — detect, contain, and remediate; then **incorporate lessons learned back into
   the existing security solution** (AMP, NGFW, Secure Network Analytics).

## Reference Tables

**Cisco SAFE — PINs and secure domains**

| PINs (places in the network) | Secure domains |
|---|---|
| Branch | Management |
| Campus | Security intelligence |
| Data center | Compliance |
| Edge | Segmentation |
| Cloud | Threat defense |
| WAN | Secure services |

> Note: in Figure 25-1 the **Internet** sits at the *center* of the key — it is the thing
> being connected to, **not one of the six PINs**. This is exactly the trap in quiz Q2.

**Cisco product name changes (old → new)**

| Formerly | Now |
|---|---|
| Threat Grid | Cisco Secure Malware Analytics |
| FireAMP / AMP for Endpoints | Cisco Secure Endpoint |
| AMP for Email / Email Security Appliance (ESA) | Cisco Secure Email |
| AMP for Web / Web Security Appliance (WSA) | Cisco Secure Web Appliance |
| Cisco AnyConnect Secure Mobility Client | Cisco Secure Client |
| OpenDNS | Cisco Umbrella |
| FirePOWER NGIPS | Cisco Secure IPS |
| Firepower NGFW | Cisco Secure Firewall |
| Firepower Management Center | Cisco Secure Firewall Management Center (FMC) |
| Firepower Device Manager | Cisco Secure Firewall Device Manager (FDM) |
| Stealthwatch Enterprise | Cisco Secure Network Analytics |
| Stealthwatch Management Console (SMC) | Cisco Secure Network Analytics Manager |
| Stealthwatch Cloud | Cisco Secure Cloud Analytics |
| Stealthwatch Cloud Private Network Monitoring | Cisco Secure Network Analytics SaaS |
| Cognitive Threat Analytics (CTA) | Global Threat Analytics (GTA) |
| AMP (product family) | Malware Defense |

**EAP methods compared**

| Method | Category | Certificates required | Mutual auth | Notes |
|---|---|---|---|---|
| EAP-MD5 | Challenge-based | None | **No** | Server validates supplicant only — poor choice |
| EAP-TLS | TLS | **Both** supplicant and server | Yes | Most secure, hardest to deploy |
| PEAP | Tunneled TLS (outer) | **Server only** | Yes (server validated) | Inner: EAP-MSCHAPv2 (PEAPv0), EAP-GTC (PEAPv1), EAP-TLS. **EAP inner methods only** |
| EAP-FAST | Tunneled TLS (outer) | Server (PAC-based) | Yes | Cisco. Faster re-auth via **PAC**; **supports EAP chaining** |
| EAP-TTLS | Tunneled TLS (outer) | Server only | Yes | Less widely supported than PEAP; also supports **non-EAP** inner methods (PAP, CHAP, MS-CHAP) |
| EAP-GTC | Inner | — | — | Cisco alternative to MSCHAPv2; any identity store (OTP, LDAP, eDirectory) |
| EAP-MSCHAPv2 | Inner | — | — | Most common inner method; username/password to AD via RADIUS |

**NAC methods compared**

| | 802.1x | MAB | LWA | CWA |
|---|---|---|---|---|
| Identity basis | Supplicant credentials (EAP) | MAC address | Username/password in portal | Username/password in portal |
| Requires supplicant | Yes | No | No | No |
| Users or devices | Both (machine + user with EAP chaining) | Devices | **Users only** | **Users only** |
| Portal location | n/a | n/a | **On the switch/WLC** | **On ISE** |
| Portal customizable | n/a | n/a | **No** (Cisco switches) | Yes (branded) |
| dACL assignment | Yes | Yes | Yes | Yes |
| VLAN assignment | Yes | Yes | **No** | Yes |
| CoA support | Yes | Yes | **No** | **Yes** |
| Posture / profiling-driven policy | Yes | Yes | **No** | Yes |
| Guest VLAN | — | — | **Mutually exclusive** | **Mutually exclusive** |
| Fallback order (default) | 1st | 2nd | 3rd | 3rd |

**TrustSec phases and mechanisms**

| Phase | What happens | Mechanisms |
|---|---|---|
| Ingress classification | SGT assigned to user/endpoint/resource entering the TrustSec domain | **Dynamic** (ISE authz via 802.1x/MAB/WebAuth); **static** (IP, subnet, VLAN, L2 interface, L3 logical interface, port, port profile → SGT); central IP-SGT DB downloaded from ISE |
| Propagation | Mappings communicated to enforcing devices | **Inline (native) tagging** — SGT in the CMD field, EtherType **0x8909**, 16-bit value; **SXP** — TCP peer-to-peer, **speaker → listener**, single- or multi-hop; **pxGrid** from ISE |
| Egress enforcement | Allow/drop applied at the egress point | **SGACL** (routers/switches, source SGT × destination SGT); **SGFW** (Secure Firewalls, tag-based rules defined locally) |

**MACsec Security Tag fields**

| Field | Octets | Meaning |
|---|---|---|
| MACsec EtherType | 1–2 | **0x88e5** — designates the frame as MACsec |
| TCI/AN | 3 | Tag Control Information / Association Number — version if confidentiality or integrity used alone |
| SL | 4 | Short Length — length of the encrypted data |
| Packet Number | 5–8 | Replay protection and IV construction |
| SCI | 9–16 | Secure Channel Identifier — classifies the connection to the virtual port (optional) |

**Downlink vs uplink MACsec**

| | Downlink MACsec | Uplink MACsec |
|---|---|---|
| Link | **Endpoint ↔ switch** | **Switch ↔ switch** |
| Default keying | **MKA** | **SAP** (Cisco proprietary) |
| Cipher | AES-GCM-128 | AES-GCM-128 (same) |
| Requirements | MACsec-capable switch **and** MACsec-capable supplicant (e.g. Cisco Secure Client) | 802.1AE support on both switches |
| Encryption mode control | Force encrypt / optional / force non-encrypt — per port manually (uncommon) or **dynamically from ISE** (common); **ISE overrides the CLI** | Manual or dynamic; **dynamic requires 802.1x between the switches** |

**Secure Network Analytics components**

| Component | Required? | Role |
|---|---|---|
| Network Analytics Manager (SMC) | Core | Control center; aggregates from up to **25** Flow Collectors, ISE, other sources |
| Flow Collectors | Core | Collect/analyze NetFlow, IPFIX, proxy telemetry; ETA for encrypted-traffic detection without decryption |
| Flow Rate License | Core | Required for collection/management/analysis; defines collectible flow volume |
| Flow Sensors | Optional | Telemetry for segments that **can't generate NetFlow**; application-layer visibility |
| UDP Director | Optional | Receives UDP streams from many sources, replicates to many destinations |
| Telemetry Broker | Optional | UDP Director plus **ingest** (e.g. AWS VPC Flow Logs), **filter**, and **transform** (e.g. to IPFIX) |
| Data Store | Optional | Centralized flow storage instead of distributed; **minimum three Data Nodes** |
| Threat Feed License | Optional | Talos threat intelligence feed |
| Endpoint License | Optional | Extends visibility into endpoints |
| Secure Cloud Analytics | Optional | Extends visibility into AWS, GCP, Azure |

**Cisco Secure Firewall software images**

| Image | What it gives you | Supported on |
|---|---|---|
| ASA software image | Standard legacy firewall, **no** Secure IPS services | All Secure Firewall and ASA appliances |
| ASA image **+** Secure IPS image (FirePOWER NGIPS) | Two images in one appliance, **different management app for each**; makes the ASA an NGFW | **Only 5500-X (except 5585-X)** |
| **FTD** software image | ASA image and Secure IPS image merged into a **single unified image** | All Secure Firewall and ASA 5500-X **except 5585-X** |

## Config Patterns

> **Provenance:** Chapter 25 is a design-and-portfolio chapter — it contains **no IOS-XE
> configuration examples** (its only "config" artifacts are ISE GUI screenshots and a pxGrid
> session-directory dump). The blocks below are canonical IBNS 2.0 / TrustSec / MACsec
> syntax **added by this skill, not transcribed from the chapter**, and are **not
> gear-validated** — verify against the platform's configuration guide and software release
> before applying. Where the chapter *does* specify a value (30 s tx-period, 3 retries,
> AES-GCM-128), that value is reflected here.

**RADIUS / AAA foundation for ISE**
```ios-xe
aaa new-model
!
radius server ISE-1
 address ipv4 10.10.10.10 auth-port 1812 acct-port 1813
 key <radius-shared-secret>
!
aaa group server radius RAD_ISE
 server name ISE-1
!
aaa authentication dot1x default group RAD_ISE
aaa authorization network default group RAD_ISE
aaa accounting dot1x default start-stop group RAD_ISE
!
! Change of Authorization — required for CWA, posture, and ISE-driven quarantine
aaa server radius dynamic-author
 client 10.10.10.10 server-key <radius-shared-secret>
!
! dACLs and SGT-to-IP mapping both depend on device tracking
device-tracking policy IPDT_POLICY
 tracking enable
!
radius-server attribute 6 on-for-login-auth
radius-server attribute 8 include-in-access-req
radius-server attribute 25 access-request include
!
dot1x system-auth-control
```

**IBNS 2.0 / C3PL — 802.1x with MAB fallback (concurrent via Access Session Manager)**
```ios-xe
class-map type control subscriber match-all DOT1X_NO_RESP
 match method dot1x
 match result-type method dot1x agent-not-found
!
class-map type control subscriber match-all MAB_FAILED
 match method mab
 match result-type method mab authoritative
!
policy-map type control subscriber DOT1X_MAB_POLICY
 event session-started match-all
  10 class always do-until-failure
   10 authenticate using dot1x priority 10
 event authentication-failure match-first
  10 class DOT1X_NO_RESP do-until-failure
   10 terminate dot1x
   20 authenticate using mab priority 20
  20 class MAB_FAILED do-until-failure
   10 terminate mab
   20 authentication-restart 60
!
interface GigabitEthernet1/0/10
 switchport mode access
 switchport access vlan 10
 device-tracking attach-policy IPDT_POLICY
 access-session host-mode multi-auth
 access-session closed
 access-session port-control auto
 mab
 dot1x pae authenticator
 dot1x timeout tx-period 7
 service-policy type control subscriber DOT1X_MAB_POLICY
```
> `access-session closed` is **closed mode** (nothing passes before authentication). Drop it
> for **monitor mode** during a phased rollout — see Design Baseline.

**TrustSec — classification, propagation, enforcement**
```ios-xe
! Device gets its TrustSec credentials and environment data from ISE
cts credentials id SW1 password <cts-password>     ! (exec mode, not running-config)
cts authorization list RAD_ISE
!
! --- Static ingress classification (data center style, no 802.1x) ---
cts role-based sgt-map 10.1.100.0/24 sgt 12
cts role-based sgt-map vlan-list 100 sgt 12
!
! --- Propagation: inline tagging on a TrustSec-capable uplink ---
interface TenGigabitEthernet1/0/1
 cts manual
  policy static sgt 2 trusted
!
! --- Propagation: SXP where the peer can't inline-tag ---
cts sxp enable
cts sxp default password <sxp-password>
cts sxp default source-ip 10.0.0.1
cts sxp connection peer 10.0.0.2 password default mode local speaker
!
! --- Egress enforcement ---
cts role-based enforcement
cts role-based enforcement vlan-list 100-110
```

**MACsec — uplink (switch-to-switch, MKA with pre-shared key)**
```ios-xe
key chain MKA_KC macsec
 key 01
  cryptographic-algorithm aes-128-cmac
  key-string <32-hex-char-key>
!
mka policy MKA_POL
 macsec-cipher-suite gcm-aes-128
!
interface TenGigabitEthernet1/0/2
 macsec network-link
 mka policy MKA_POL
 mka pre-shared-key key-chain MKA_KC
```

**MACsec — downlink (switch-to-endpoint, keyed by MKA, mode from ISE)**
```ios-xe
interface GigabitEthernet1/0/10
 macsec
 access-session port-control auto
 dot1x pae authenticator
```
> The encryption policy (must-secure / should-secure / no-encrypt) is normally returned by
> ISE in the authorization result, and **an ISE-returned policy overrides the switch CLI**.

## Design Baseline

Rows sourced from the ENCOR 350-401 OCG Chapter 25 itself (page cited) and from named Cisco
guides. Per the "no source, no row" rule, practices I could not trace to a named document
were left out rather than written from memory. **A deviation is a question for the network's
operator, not automatically a finding.**

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Give MAB-authenticated endpoints **very restricted access** — only the networks and services they actually need | MAC addresses are trivially spoofed; MAB is an identity claim anyone can make | A closed lab or an isolated OT segment where the blast radius is already bounded; a transitional phase while profiling data is still being gathered | ENCOR OCG Ch. 25, p. 763 |
| **Do not use EAP-MD5** where the supplicant needs to trust the network | No mutual authentication — the supplicant never validates the server, so a rogue authentication server is undetectable | Legacy gear that supports nothing else, on a segment where that risk is explicitly accepted and logged | ENCOR OCG Ch. 25, p. 761 |
| Prefer **CWA over LWA** wherever posture, profiling, or quarantine matter | LWA supports **no CoA** and **no VLAN assignment**, so policy can never change after the initial decision — including to quarantine a compromised host | No ISE in the deployment; a small site where a static ACL-only guest policy is genuinely sufficient | ENCOR OCG Ch. 25, pp. 764–765 |
| Use **EAP chaining (EAP-FAST)** when privilege should depend on *corporate-managed device + user*, not user alone | Machine and user auth combine into a **single** result, so "right person on the wrong laptop" is distinguishable from "right person on a corporate laptop" | Supplicants that don't support EAP-FAST; environments that are BYOD by design and don't differentiate on device ownership | ENCOR OCG Ch. 25, p. 762 |
| Verify **every device in a MACsec path** supports MACsec before enabling it | The 802.1AE header and ICV require support end to end along the MACsec flow; a non-capable device in the path breaks it | None on the data path itself — this is a hard requirement, not a preference | ENCOR OCG Ch. 25, p. 773 |
| Confirm the software release supports **SGT encapsulation inside MACsec** before relying on both together | Support is release-dependent; the chapter explicitly says to check the documentation | None — verify, don't assume | ENCOR OCG Ch. 25, p. 773 (NOTE) |
| Expect a **non-TrustSec-capable device in the path to drop inline-tagged frames** — plan SXP instead | Native tagging needs TrustSec ASIC support; the frame is dropped, not stripped, by a device without it | None — choose the propagation method to match the hardware in the path | ENCOR OCG Ch. 25, p. 768 |
| Deploy **SAFE per PIN, removing PINs that don't exist**, and pair it with the CVD guides for the underlying network design | SAFE covers security service integration only; the networking design and implementation guidance lives in the CVDs | None — this is how the framework is defined | ENCOR OCG Ch. 25, pp. 738–739; Cisco Validated Design guides, `www.cisco.com/go/cvd` |
| Roll 802.1x out in **monitor mode first**, then move to low-impact or closed mode | Authenticating in log-only mode surfaces every unprofiled printer, camera, and badge reader **before** an enforcement change locks them off the network | None for the phased approach itself; the *duration* of monitor mode is the tunable | Cisco ISE Secure Wired Access Prescriptive Deployment Guide (cisco.com) — **verify the current revision; cited from familiarity, not read for this skill** |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show access-session` | IBNS 2.0 session summary — interface, MAC, method (dot1x/mab/webauth), domain, status (Authz Success / Authz Failed / Running) |
| `show access-session interface <int> details` | Per-session detail: user, status, oper host mode, oper control dir, authorized-by, VLAN, **ACS ACL / dACL name**, SGT, method status list |
| `show authentication sessions interface <int> details` | Same view on IBNS 1.0 / older releases |
| `show dot1x all` / `show dot1x interface <int> details` | Global and per-port 802.1x state, PAE role, tx-period, max-reauth-req; confirms the authenticator is even trying |
| `show mab all` / `show mab interface <int> details` | MAB enabled and its session state — confirms fallback actually engaged |
| `show aaa servers` | Per-server request/accept/reject/timeout counters — proves whether RADIUS is reachable and answering |
| `show radius statistics` | Access-request vs Access-accept/reject/timeout totals; rising timeouts = path or shared-secret problem |
| `test aaa group RAD_ISE <user> <pass> new-code` | End-to-end RADIUS reachability and credential test, independent of any endpoint |
| `show ip access-lists interface <int>` | The **dACL actually applied** to the session — not the static config, the downloaded one |
| `show device-tracking database` | IP-to-MAC bindings; empty here means dACLs and IP-to-SGT mapping have nothing to bind to |
| `show cts environment-data` | Environment data downloaded from ISE, SGT name table, refresh state — the first thing to break when TrustSec auth fails |
| `show cts pacs` | The PAC obtained from ISE for TrustSec authorization |
| `show cts role-based sgt-map all` | Every IP-to-SGT binding and its **source** (LOCAL / SXP / CLI / INTERNAL) — confirms classification and propagation |
| `show cts role-based permissions` | The SGACL matrix as the device actually sees it, source SGT → destination SGT |
| `show cts role-based counters` | Per-cell hit counters — proves enforcement is happening and which cell is matching |
| `show cts sxp connections` | SXP peer state (**On** / PendingOn), speaker/listener mode, source and peer IP |
| `show cts interface <int>` | TrustSec mode on the link (manual / dot1x), SAP status, propagate SGT, static SGT + trusted |
| `show macsec summary` | Which interfaces have MACsec and in what state |
| `show macsec interface <int>` | Cipher suite, encryption/decryption packet and error counters, SCI |
| `show mka sessions` / `show mka sessions details` | MKA session status, key server, CKN/CAK state — where downlink and MKA-keyed uplink MACsec actually fail |
| `show mka policy` | Configured MKA policy and cipher suite |
| `debug dot1x all` / `debug mab all` / `debug radius authentication` | Last resort — CPU-expensive and process-switched; see the network-assurance skill for debug discipline |

## Intent Questions

- **What is this port *supposed* to authenticate, and with which method?** Is it 802.1x with
  a supplicant, MAB for a known device class, WebAuth for guests, or deliberately open? An
  unexpected `Method: mab` on a port that should be doing dot1x is the finding, not the port
  being down.
- **What mode is the deployment in — monitor, low-impact, or closed?** A failed
  authentication in monitor mode is *expected behavior*; the same failure in closed mode is
  an outage. Knowing which one you're in changes whether the symptom is a bug.
- **Where is the enforcement point meant to be?** If segmentation is TrustSec-based,
  enforcement is at **egress**, possibly several hops from the user and possibly on the same
  switch. Looking for a deny at the ingress port will find nothing.
- **What is the authorization result supposed to contain** — a dACL, a dynamic VLAN, an SGT,
  a MACsec encryption policy, or some combination? "Authenticated" is not the goal;
  authenticated *with the right authorization payload* is.
- **Is ISE the intended source of truth for this decision?** ISE-returned policy overrides
  local switch CLI for MACsec encryption mode, and ISE holds the SGT and dACL definitions —
  a local config that disagrees is not necessarily what's in effect.

## Troubleshooting Checklist

0. **State intent vs. observed.** Answer the Intent Questions above for this network, then
   write the one-line symptom ("this printer should land in VLAN 30 via MAB with a
   restricted dACL, it is landing in the data VLAN with no ACL") — before running any show
   command.
1. **Is the port physically up and is the endpoint actually sending?** A supplicant that
   never sends EAPoL-start and never answers EAP-request/identity looks identical to a dead
   NIC for the first 90 seconds. `show access-session interface <int> details`.
2. **Which method ran, and did it run at all?** `show access-session` — if the method is
   `mab` on a port that should do dot1x, the supplicant is missing or misconfigured; if
   there is **no session**, `dot1x system-auth-control` or the interface commands are
   missing. If MAB engaged, confirm the 90-second 802.1x timeout is the reason and not a
   config that skipped dot1x entirely.
3. **Is RADIUS reachable and answering?** `show aaa servers` and `show radius statistics` —
   rising **timeouts** mean a path, ACL, or source-interface problem; rising **rejects**
   mean the request arrives and ISE says no, which is a policy problem, not a network one.
   `test aaa group ... new-code` isolates this from the endpoint entirely.
4. **Shared secret and NAD definition.** A wrong key looks like a timeout, not an error. Is
   this switch defined as a network device in ISE with the matching secret and the right
   device group (the group often drives which policy set matches)?
5. **Read the ISE side.** ISE Live Logs name the failed policy set, authorization rule, and
   failure reason far faster than the switch will. This is the step people skip.
6. **Did the authorization payload actually apply?** Authenticated ≠ authorized correctly.
   `show ip access-lists interface <int>` for the dACL, the session details for the VLAN and
   SGT. A dACL that didn't apply is frequently **device tracking** — check
   `show device-tracking database` for an IP binding.
7. **CoA path, if the flow needs one.** CWA, posture, and quarantine all depend on
   **CoA-reauth** reaching the switch. Confirm `aaa server radius dynamic-author` exists with
   the correct client and key, and that nothing filters UDP 1700/3799 between ISE and the
   NAD. **LWA does not support CoA at all** — if the design assumed it could re-authorize,
   the design is the bug.
8. **WebAuth specifics.** Redirect ACL present and correct? DNS reachable *before*
   authentication (step 4 of CWA is DHCP/DNS)? Remember **guest VLAN and WebAuth are
   mutually exclusive** — configuring both is a design conflict, not a transient failure.
9. **TrustSec classification.** `show cts role-based sgt-map all` — is there a binding at
   all, and what is its **source**? No binding means classification failed; a `CLI` source
   where you expected `LOCAL` means the dynamic assignment from ISE never arrived.
10. **TrustSec environment data.** `show cts environment-data` and `show cts pacs` — if the
    device has no environment data it has no SGT name table and enforcement silently does
    nothing useful. This usually traces back to `cts credentials` or the authorization list.
11. **TrustSec propagation.** If the path is inline-tagged, confirm **every** device in it
    has TrustSec ASIC support — a device without it **drops** tagged frames. If SXP,
    `show cts sxp connections` should read **On**; check speaker/listener roles aren't both
    the same on a pair, and that the password and source IP match.
12. **TrustSec enforcement.** `show cts role-based permissions` for the matrix the device
    actually holds, then `show cts role-based counters` to prove traffic is hitting the cell
    you think it is. Remember enforcement is at **egress** — and for two hosts on the same
    switch, that switch is both ingress and egress.
13. **MACsec.** `show macsec summary` and `show mka sessions` — for downlink, confirm the
    supplicant is MACsec-capable and MKA is the keying protocol; for uplink, remember the
    default is **SAP**, and **dynamic** uplink MACsec requires 802.1x between the switches.
    Check whether an **ISE-returned encryption policy is overriding** the CLI setting.
14. **Only then, debug.** `debug dot1x all` / `debug mab all` / `debug radius
    authentication`, scoped with a conditional debug and directed to the logging buffer, not
    the console.

## Common Pitfalls

- **The authenticator does not know or care what EAP method is in use.** It relays EAPoL to
  RADIUS and back. Troubleshooting an EAP-method problem by looking at the switch is looking
  in the wrong place — the conversation is supplicant ↔ authentication server.
- **The Internet is not one of the six SAFE PINs.** The PINs are Branch, Campus, Data center,
  Edge, Cloud, WAN. Figure 25-1 puts Internet at the *center* of the key. (Quiz Q2's trap.)
- **"Segregation" is not a SAFE secure domain.** The six are Management, Security
  intelligence, Compliance, Segmentation, Threat defense, Secure services. Segregation sounds
  plausible next to Segmentation and is an invented distractor. (Quiz Q3's trap.)
- **Talos is the intelligence organization; Secure Malware Analytics is the sandbox; Secure
  Network Analytics is the telemetry aggregator.** These three get swapped constantly because
  all three "analyze threats." The discriminators: Talos = the *feed*, Malware Analytics =
  *files in a sandbox*, Network Analytics = *NetFlow/IPFIX telemetry*. (Quiz Q4, Q5, Q6.)
- **pxGrid requires ISE.** ISE is the pxGrid controller/server; everything else is a pxGrid
  *node*. Without ISE there is no controller, so there is no pxGrid. (Quiz Q7.)
- **SGTs do not reach the endpoint.** Endpoints are entirely unaware of the tag — it is
  assigned, propagated, and enforced **within the network infrastructure only**. (Quiz Q9.)
- **TrustSec has three phases, not four.** Classification → propagation → enforcement.
  "Distribution" and "aggregation" sound like they belong but do not. (Quiz Q10.)
- **EAP-FAST is the one with EAP chaining**, via PACs. PEAP, EAP-TTLS, and EAP-GTC do not
  chain. (Quiz Q8.)
- **EAP-TLS appears twice** — as a standalone method *and* as an inner method inside PEAP.
  The standalone case is "most secure, hardest to deploy"; the inner case is "a TLS tunnel
  inside another TLS tunnel," which is rarer still. Don't treat them as the same row.
- **PEAP supports EAP inner methods only.** EAP-TTLS is the one that also carries legacy
  non-EAP inner methods (PAP, CHAP, MS-CHAP). This is the single most useful discriminator
  between the two.
- **LWA has no CoA and no VLAN assignment.** Any design that assumes it can re-authorize a
  guest after posture, or drop them into a different VLAN, has picked the wrong WebAuth type.
  That is precisely why CWA exists.
- **Guest VLAN and WebAuth (LWA or CWA) are mutually exclusive.** Configuring both is a
  design conflict, not a failover.
- **MAB is a fallback, not a security control.** MAC addresses are spoofable by anyone;
  restrict what a MAB session can reach.
- **Packets sent during the 802.1x timeout window are discarded and cannot seed MAB.** The
  switch only learns the MAC from a packet arriving *after* the port has fallen back. A
  chatty-then-silent device can therefore sit unauthenticated longer than expected.
- **MAB starts immediately after linkup if 802.1x is not enabled** — the 90-second wait only
  exists because dot1x is timing out first.
- **Inline-tagged frames are dropped, not stripped, by devices without TrustSec ASIC
  support.** This fails as a black hole, not as an error message.
- **TrustSec enforcement is at egress.** Hunting for the deny at the user's ingress port
  finds nothing — and when both hosts are on the same switch, that switch is simultaneously
  the ingress and egress point.
- **MACsec is hop-by-hop, and traffic is plaintext inside the switch.** That is deliberate —
  it is what lets the switch read the SGT for enforcement and QoS. Treating MACsec as
  end-to-end encryption misrepresents what it protects.
- **Uplink MACsec defaults to SAP, not MKA.** The cipher is the same AES-GCM-128 either way,
  but the keying protocol differs, and dynamic uplink MACsec additionally requires 802.1x
  *between the switches*.
- **An ISE-returned MACsec encryption policy overrides the switch CLI.** A port configured
  one way at the CLI can be behaving another way entirely.
- **FirePOWER (uppercase) ≠ Firepower (lowercase).** Uppercase = Secure IPS (NGIPS) services
  software or the ASA NGIPS module. Lowercase = Cisco Secure Firewall or the FTD unified
  image. Cisco's own documentation relies on this distinction.
- **The 5585-X is the exception everywhere.** FTD and the ASA+IPS dual-image configuration
  are supported on all ASA 5500-X appliances **except** the 5585-X.
- **FTD/Secure IPS CLI configuration is not supported** — CLI is for initial setup and
  troubleshooting only. Reaching for the CLI to make a policy change is not just discouraged,
  it is unsupported.
- **AMP connectors send a hash, not the file.** The cloud makes the decision. This is why
  connectors stay lightweight and why a disposition can change retroactively for files
  already seen.
- **A file disposition can change after the fact** — that is the whole point of retrospection.
  "It was clean when it came through" is not a durable statement.

## Exam Preparation Tasks

### Key topics coverage map

Table 25-2, mapped to where each element actually lives in this skill. The mapping is
deliberately not one-to-one — several product sections collapse into a single Key Concepts
block here, and the NAC key topics are split across Key Concepts, Procedure, Reference
Tables, and Common Pitfalls.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Paragraph | Cisco SAFE places in the network (PINs) | 738 | Key Concepts → "Cisco SAFE — the framework" (all six PINs with their top threats); Reference Tables → "Cisco SAFE — PINs and secure domains"; Common Pitfalls (Internet is not a PIN) |
| List | Cisco SAFE Full attack continuum | 740 | Key Concepts → "Cisco SAFE" (before/during/after with the tools for each); Procedure → "Attack-continuum design method" |
| Section | Cisco Talos | 741 | Key Concepts → "Cisco Talos — the intelligence source" (three founding teams, daily scale, intelligence feeds); Common Pitfalls (Talos vs Malware Analytics vs Network Analytics) |
| Section | Cisco Secure Malware Analytics (Threat Grid) | 742 | Key Concepts → "Cisco Secure Malware Analytics" (static vs dynamic analysis, sandbox-evasion-evasion, Glovebox); Reference Tables → product name changes |
| Section | Cisco Advanced Malware Protection (AMP) | 742 | Key Concepts → "Cisco AMP" (attack continuum, retrospection, IoCs, file dispositions, hash-not-file model) |
| List | Cisco AMP components | 743 | Key Concepts → "Cisco AMP" (AMP Cloud, the five connectors, Talos/Malware Analytics intelligence, CSC on iOS) |
| Section | Cisco Secure Client (AnyConnect) | 744 | Key Concepts → "Cisco Secure Client" (modules, VPN Posture/HostScan, ISE Posture, roaming, platforms) |
| Section | Cisco Umbrella | 744 | Key Concepts → "Cisco Umbrella" (DNS-layer, Anycast, scale, DHCP deployment, roaming module vs roaming client) |
| Section | Cisco Secure Web Appliance (WSA) | 746 | Key Concepts → "Cisco Secure Web Appliance" (before/during/after, reputation −10..+10, DCA, AVC, CASB, L4 monitor, DLP via ICAP, GTA) |
| Section | Cisco Secure Email (ESA) | 748 | Key Concepts → "Cisco Secure Email" (CASE, forged email/BEC, CAPP, CDP, graymail, outbreak filters, web interaction tracking) |
| Section | Cisco Secure IPS (FirePOWER NGIPS) | 749 | Key Concepts → "Cisco Secure IPS" (IDS vs IPS, Sourcefire 2013, Snort, ISE quarantine/unquarantine/shutdown) |
| List | Next-generation IPS (NGIPS) capabilities | 749 | Key Concepts → "Cisco Secure IPS" (the five Gartner capabilities, then the Cisco-exceeds list) |
| Section | Cisco Secure Firewall (NGFW) | 751 | Key Concepts → "Cisco Secure Firewall"; Reference Tables → "Cisco Secure Firewall software images"; Common Pitfalls (FirePOWER vs Firepower, 5585-X, CLI unsupported) |
| List | NGFW firewall capabilities | 751 | Key Concepts → "Cisco Secure Firewall" (the four Gartner NGFW requirements) |
| Section | Cisco Secure Network Analytics (Stealthwatch Enterprise) | 753 | Key Concepts → "Cisco Secure Network Analytics" (agentless, ETA without decryption, insider threats) |
| List | Cisco Secure Network Analytics components | 753 | Reference Tables → "Secure Network Analytics components" (core vs optional, 25 Flow Collectors, 3 Data Nodes) |
| Section | Cisco Secure Cloud Analytics (Stealthwatch Cloud) | 755 | Key Concepts → "Cisco Secure Cloud Analytics" (SaaS, metadata only, VPC flow logs) |
| List | Cisco Secure Network Analytics offerings | 755 | Key Concepts → "Cisco Secure Cloud Analytics" (the two deployment models); Reference Tables → product name changes |
| Section | Cisco Identity Services Engine (ISE) | 756 | Key Concepts → "Cisco ISE — the policy brain" (all features incl. pxGrid 1.0/2.0 and the session-directory context) |
| Section | 802.1x | 758 | Key Concepts → "802.1x — port-based network access control"; Procedure → "Successful 802.1x authentication" |
| List | 802.1x components | 758 | Key Concepts → "802.1x" (EAP/RFC 4187, EAP method, EAPoL, RADIUS) |
| List | 802.1x roles | 758 | Key Concepts → "802.1x" (supplicant, authenticator/NAD, authentication server); Common Pitfalls (authenticator is method-blind) |
| List | EAP methods | 760 | Key Concepts → "EAP methods" (all seven, by category); Reference Tables → "EAP methods compared"; three Common Pitfalls bullets |
| Section | EAP Chaining | 762 | Key Concepts → "EAP chaining"; Design Baseline row 4; Common Pitfalls (EAP-FAST is the one) |
| Section | MAC Authentication Bypass (MAB) | 762 | Key Concepts → "MAC Authentication Bypass"; Procedure → "Successful MAB authentication"; Reference Tables → "NAC methods compared"; Design Baseline row 1; three Common Pitfalls bullets |
| Section | Web Authentication (WebAuth) | 764 | Key Concepts → "Web Authentication"; Procedure → "Central Web Authentication with ISE"; Reference Tables → "NAC methods compared"; Design Baseline row 3 |
| List | WebAuth types | 764 | Key Concepts → "Web Authentication" (LWA vs CWA, incl. LWA's no-CoA/no-VLAN limits); Common Pitfalls (LWA limits, guest VLAN exclusivity) |
| Section | Cisco TrustSec | 766 | Key Concepts → "Cisco TrustSec"; Config Patterns → TrustSec block; Common Pitfalls (endpoints unaware of SGT) |
| List | Cisco TrustSec phases | 767 | Key Concepts → phases 1–3; Procedure → "TrustSec deployment (the three phases)"; Reference Tables → "TrustSec phases and mechanisms"; Common Pitfalls (three phases, not four) |
| Paragraph | Cisco TrustSec SGT propagation methods | 768 | Key Concepts → "Phase 2 — Propagation" (inline/native tagging with CMD 0x8909 and the 16-bit value; SXP speaker/listener, single/multi-hop; ISE as speaker; pxGrid); Reference Tables; Design Baseline row 7 |
| List | Cisco TrustSec SGT types of enforcement | 770 | Key Concepts → "Phase 3 — Egress enforcement" (SGACL vs SGFW, production matrix, granular `Permit_FTP`); Common Pitfalls (enforcement is at egress) |
| Section | MACsec | 772 | Key Concepts → "MACsec" (802.1AE, hop-by-hop, why plaintext internally, GMAC/AES-GCM, tag fields); Reference Tables → "MACsec Security Tag fields" |
| List | MACsec keying mechanisms | 773 | Key Concepts → "MACsec" (SAP vs MKA); Reference Tables → "Downlink vs uplink MACsec"; Common Pitfalls (uplink defaults to SAP) |
| Section | Downlink MACsec | 774 | Key Concepts → "MACsec" → Downlink bullet; Reference Tables → "Downlink vs uplink MACsec"; Config Patterns → downlink block; Common Pitfalls (ISE overrides CLI) |
| Section | Uplink MACsec | 774 | Key Concepts → "MACsec" → Uplink bullet; Reference Tables → "Downlink vs uplink MACsec"; Config Patterns → uplink MKA block |

**Coverage note.** All 35 rows of Table 25-2 map to content in this skill — no gaps. The one
honest weakness is **Config Patterns**: the chapter itself contains no IOS-XE configuration,
so the config blocks here were added by this skill from standard IBNS 2.0 / TrustSec / MACsec
syntax and are **not gear-validated**. Chapter 26 ("Network Device Access Control and
Infrastructure Security") is where the OCG's actual AAA/device-access configuration lives.

### "Do I Know This Already?" question analysis

Answer key as printed on p. 740: **1** C · **2** B through G · **3** A, B, D · **4** C ·
**5** B · **6** B · **7** A · **8** B · **9** B · **10** A, B, E.

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| 1 | Whether you know the framework's actual acronym expansion — **S**ecure **A**rchitectural **F**ramework → SAFE | **C** — Cisco SAFE | "Cisco SEAF" is an invented term built from the same initials in the wrong order. "Cisco Validated Designs" is a **real, adjacent** thing — CVDs cover the *networking* design that SAFE explicitly defers to (p. 739 NOTE), so this is the high-value distractor: two real Cisco frameworks, different scopes |
| 2 | Whether you can enumerate the six PINs **and** notice what isn't one | **B through G** — Data center, Branch, Edge, Campus, Cloud, WAN | "Internet" (a) is the trap, and a good one: Figure 25-1 literally draws the Internet at the **center** of the SAFE key. It is what the PINs connect to, not a place security is designed for. Promoted to Common Pitfalls |
| 3 | Whether you know the six secure domains, which are easy to half-remember | **A, B, D** — Threat defense, Segmentation, Compliance | "Segregation" (c) is an invented term that sounds plausible **because Segmentation is real** — a one-word swap. The full six (Management, Security intelligence, Compliance, Segmentation, Threat defense, Secure services) are the defence. Promoted to Common Pitfalls |
| 4 | Whether you can separate the *intelligence organization* from the products that consume its feed | **C** — Cisco Talos | "Cisco TRAC team" (d) is the sharpest distractor: TRAC is **real and was one of the three teams Talos was formed from**, so it's a right-answer-one-level-too-deep. Secure Network Analytics and Secure Malware Analytics are real products, not intelligence orgs |
| 5 | Product identity: which one is the sandbox | **B** — the Cisco sandbox malware analysis solution | Each distractor is the correct description of a **different** product in the same chapter: (a) = Talos, (c) = SAFE, (d) = Secure Network Analytics. This is a four-way product-confusion test, not recall. Promoted to Common Pitfalls |
| 6 | Which product is telemetry/flow-driven | **B** — Cisco Secure Network Analytics | Reverse of Q5 — same product set, asked from the capability side. The discriminator is "NetFlow/IPFIX telemetry aggregation" vs Talos (intelligence feed), Malware Analytics (file sandbox), WSA (web gateway) |
| 7 | Whether you understand pxGrid's **architecture**, not just that it exists | **A** — True | Tempting to answer False because pxGrid is an **IETF framework** with 50+ third-party nodes, which sounds vendor-neutral. But ISE is the pxGrid **controller/server** and everything else is a node — no ISE, no controller, no pxGrid. Promoted to Common Pitfalls |
| 8 | Which EAP method carries EAP chaining | **B** — EAP-FAST | All four options are real EAP methods, so there is no giveaway. The link to remember: EAP-FAST → PACs → fast re-auth → **and** EAP chaining (machine + user in one outer tunnel, one result). EAP-GTC (c) is a distractor that is a real **inner** method, not an outer one — a category error as well as a wrong answer |
| 9 | Whether you know the **scope** of an SGT | **B** — False | Sounds true because the tag *represents* the endpoint's identity and is assigned at the endpoint's point of entry. But the chapter is explicit: endpoints are **not aware** of the SGT; it exists only inside the network infrastructure. Promoted to Common Pitfalls |
| 10 | The three TrustSec phases | **A, B, E** — Classification, Enforcement, Propagation | "Distribution" (c) and "Aggregation" (d) are invented terms, and "distribution" is especially plausible because **propagation** genuinely means distributing the mappings — the concept is right, the term is wrong. Promoted to Common Pitfalls |

**Pattern worth noting.** Seven of the ten questions are product- or terminology-discrimination
tests (Q1, Q3, Q4, Q5, Q6, Q8, Q10) rather than mechanism questions, and in five of those the
distractor is a **real Cisco thing from a neighbouring slot** (CVD, TRAC, Talos, Secure
Network Analytics, EAP-GTC) rather than a fabrication. The defence for this chapter is a clean
mental table of *which product does what* and *which term belongs to which list* — the two
mechanism questions (Q7, Q9) are the only ones where reasoning from first principles beats
memorization.
