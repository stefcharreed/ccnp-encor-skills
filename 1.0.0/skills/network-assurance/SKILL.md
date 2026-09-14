---
name: ccnp-network-assurance
description: >
  Use this skill when troubleshooting, instrumenting, or monitoring an IOS-XE network —
  the tools that tell you what the network is actually doing. Invoke when the user asks
  about: network assurance, ping, extended ping, ping sweep, traceroute, debug,
  conditional debug, debug condition, undebug all, logging buffer, logging console,
  logging monitor, terminal monitor, SNMP, SNMPv1, SNMPv2c, SNMPv3, MIB, OID, SMI,
  enterprises subtree, sysDescr, sysObjectID, sysUpTime, sysContact, sysName,
  sysLocation, community string, read-only community, read-write community, snmp-server
  community, snmp-server host, snmp-server enable traps, SNMP trap, SNMP inform, get,
  getnext, getbulk, set, NMS, syslog, syslog severity, logging trap, logging host,
  logging buffered, emergencies alerts critical errors warnings notifications
  informational debugging, LOG_EMERG, UDP 514, NetFlow, Flexible NetFlow, FNF, NetFlow
  Data Capture, NetFlow Data Export, flow record, flow exporter, flow monitor, flow
  sampler, key field, non-key field, match, collect, ip flow ingress, ip flow egress,
  ip flow-export, ip flow-top-talkers, top talkers, NetFlow v9, flow cache, cache
  timeout active, SPAN, local SPAN, RSPAN, ERSPAN, monitor session, encapsulation
  replicate, filter vlan, remote-span, RSPAN VLAN, erspan-source, erspan-id, origin ip
  address, erspan ttl, port mirroring, traffic analyzer, splitter, tap, IP SLA,
  icmp-echo, http get, ip sla schedule, ip sla responder, CISCO-RTTMON-MIB, jitter,
  packet loss, one-way delay, Cisco DNA Center Assurance, Catalyst Center Assurance,
  Network Time Travel, Client 360, Device 360, Path Trace, guided remediation, SWIM,
  streaming telemetry.
---

## Purpose
Network assurance is the instrumentation layer — ping, traceroute, debug, SNMP, syslog,
NetFlow, SPAN, IP SLA, and Cisco DNA Center Assurance — that turns "the network feels
slow" into a specific device, interface, flow, or client you can point at. Each tool
answers a different question, and picking the wrong one is why troubleshooting stalls.

## Key Concepts

**Which tool answers which question**
- **ping** — is the destination reachable *right now*, from this source, at this size?
- **traceroute** — *where* along the path does it stop, and what is the per-hop latency?
- **debug** — what is the control plane doing, packet by packet, *as it happens*.
- **SNMP** — poll device state on a schedule, and receive event notifications (traps).
- **syslog** — what did the device *say happened*, with a timestamp and a severity.
- **NetFlow** — who is talking to whom, how much, and over what — flow accounting.
- **SPAN / RSPAN / ERSPAN** — what is actually on the wire, bit for bit.
- **IP SLA** — synthetic probes measuring the path continuously, before users complain.
- **DNA Center Assurance** — all of the above correlated, scored, and played back in time.

**ping and traceroute**
- `ping` uses ICMP echo request / echo reply. Extended ping (`ping` with no arguments in
  privileged EXEC) lets you set the **source interface, datagram size, DF bit, repeat
  count, and timeout** — which is how you test MTU and asymmetric-routing problems that a
  plain `ping <ip>` cannot see.
- A **ping sweep** walks a range of addresses to find what answers; useful for discovery
  and for proving a subnet is silent rather than one host.
- `traceroute` works by sending probes with an **incrementing TTL** — TTL 1 elicits a
  *time exceeded* from the first hop, TTL 2 from the second, and so on — until the
  destination replies. Each hop's round-trip time is shown.
- A hop showing `* * *` is not proof of a break: many devices rate-limit or suppress ICMP
  unreachables. Read the hop *after* it before concluding anything.

**Debugging**
- `debug` output is process-switched and CPU-expensive. On a busy production device an
  unqualified debug can wedge the box — this is the single most dangerous command in the
  assurance toolkit.
- **Conditional debugging** (`debug condition interface ...`, `debug condition ip ...`,
  or a protocol's own `debug ... <acl>` form) narrows output to one interface, peer, or
  address, and is what makes debugging survivable in production.
- By default **all syslog messages, including debug output, go to the console.** Sending
  debugs to the logging buffer instead (`no logging console` + `logging buffered
  debugging`) keeps the console usable while you work. Over SSH/Telnet, debug output
  requires `terminal monitor` on the VTY line.
- **Always `undebug all` when finished.** A forgotten debug is a latent outage.

**SNMP**
- Three components: the **managed device** (runs the SNMP agent), the **agent**, and the
  **NMS / manager** that polls it. IOS-XE ships the agent; the NMS is separate software.
- **Versions:** v1 and v2c authenticate with a cleartext **community string** and carry
  no encryption — v2c adds `getbulk` and `inform`. **v3** adds real authentication and
  privacy (`noAuthNoPriv` / `authNoPriv` / `authPriv`) and is the only version that
  belongs on an untrusted path.
- **Community strings** come in **read-only (ro)** and **read-write (rw)** flavors.
  Binding each to an access list is what limits which hosts may use it.
- **Operations:** `get`, `getnext`, `getbulk` (v2c+), `set`, `response`, plus the
  device-initiated **trap** (fire and forget) and **inform** (acknowledged).
- **MIB** — the structured database of manageable objects; each object has an **OID**,
  a SYNTAX, a MAX-ACCESS (`read-only` / `read-write`), a STATUS, and a DESCRIPTION.
  MIB files are plain text and human-readable.
- Vendor-specific objects live under the **SMI enterprises subtree, `1.3.6.1.4.1`** —
  the OCG's own example: if vendor "Flintstones, Inc." were assigned `1.3.6.1.4.1.424242`
  it could assign `1.3.6.1.4.1.424242.1.1` to its "Fred Router".
- The `system` group objects that map directly to device config:

| MIB object | OID suffix | SYNTAX | MAX-ACCESS | What it is |
|---|---|---|---|---|
| `sysDescr` | `system 1` | DisplayString | read-only | Textual description of the entity |
| `sysObjectID` | `system 2` | OBJECT IDENTIFIER | read-only | Vendor's authoritative "what kind of box" ID, under `1.3.6.1.4.1` |
| `sysUpTime` | `system 3` | TimeTicks | read-only | Hundredths of a second since the management portion was last re-initialized |
| `sysContact` | `system 4` | DisplayString | **read-write** | Contact person + how to reach them; zero-length string if unknown |
| `sysName` | `system 5` | DisplayString | **read-write** | Administratively assigned name; **by convention the node's FQDN** |
| `sysLocation` | `system 6` | DisplayString | **read-write** | Physical location, e.g. "telephone closet, 3rd floor" |

**syslog**
- Messages can go to three places at once — the **console**, the **logging buffer**, and
  an **off-box collector** — and each destination can be set to a *different* severity.
- **By default all syslog messages are sent to the console.** That is why `debug` output
  appears there.
- **The clock must be right before logging is useful.** Timestamps that don't reflect real
  time make correlation impossible. NTP is what fixes this — and the OCG explicitly notes
  NTP itself is not covered in this chapter.
- Off-box logging defaults to **UDP port 514**, changeable if needed.
- The default logging buffer is **4096 bytes** and gets overwritten quickly; expanding it
  is standard practice.
- Severity is set *by number or by keyword* — `logging buffered 7` and `logging buffered
  debugging` are the same thing. **A configured level includes every level below it**
  (numerically lower = more severe).
- Having syslog configured does not find the issue for you. It guides you toward it;
  reading it still takes skill.

**NetFlow**
- Two components that must both be configured: **NetFlow Data Capture** (captures the
  traffic statistics into the cache) and **NetFlow Data Export** (ships them to a
  collector such as **Cisco DNA Center** or **Cisco Prime Infrastructure**). If you only
  want local visibility, the export step can be skipped.
- **Design warning: NetFlow consumes memory.** Statistics live in a memory cache whose
  default size is **platform specific** — investigate it before enabling, especially on
  older platforms with less memory.
- NetFlow captures on **ingress and egress**.
- A **flow** is a **unidirectional** traffic stream identified by a combination of key
  fields: source IP, destination IP, source port, destination port, Layer 3 protocol
  type, ToS, and input logical interface.
- **Top talkers** (`ip flow-top-talkers`) gives a fast snapshot of what is loudest on a
  device, sorted by bytes or packets, for 1–200 talkers.

**Flexible NetFlow (FNF)**
- Built for more complex analysis than traditional NetFlow, through **reuse of
  configuration components**. Multiple **flow monitors can watch the same traffic at the
  same time**, so two departments can analyze the same packets with different parameters.
- Four components: **flow records**, **flow monitors**, **flow exporters**, **flow
  samplers**.
- **Sampling trade-off:** sampled data means less memory and CPU load, but lower accuracy
  — something can be missed. Whether that is acceptable is a business decision.
- **`match` selects key fields; `collect` selects non-key fields.** Key fields define what
  makes a flow distinct; non-key fields are the data gathered about it.
- FNF is a genuine **security** tool: it tracks all parts of the IP header, can build
  individual caches per flow type, and can filter ingress traffic destined to a single
  destination — useful for identifying **DoS attacks and worm propagation**.
- **Each flow monitor has its own cache**, and the flow record assigned to it defines how
  that cache is carved up.

**SPAN technologies**
- Three ways to see the wire at Layer 2: insert an **optical splitter** (splits light
  through a prism; the original source stays intact), have the **network device mirror
  packets at the data plane** to another port, or **insert a switch** between the two
  devices and mirror from it. SPAN covers the second and third.
- **Local SPAN** — source and destination on the same switch.
- **RSPAN** — sources on one switch, destination on another, carried in a dedicated
  **RSPAN VLAN** across Layer 2.
- **ERSPAN** — sources on a remote device, destination reached through **Layer 3 routing**.
  The powerful case: packet captures from anywhere with IP connectivity across a WAN.
- **Local SPAN source can only be:** one or more switch ports, a port channel
  (EtherChannel), or a VLAN (traffic received by the switch for all hosts in that VLAN —
  **this does not include the SVI**).
- Practical limits: most switches support **at least two** SPAN sessions (newer hardware
  more); a **source port can be reused** between sessions but a **destination port cannot**;
  sources can be switched or routed ports; and the destination can be **saturated** if the
  sources out-run it (10G sources into a 1G destination will drop).
- Direction is `rx`, `tx`, or `both` — **`both` is the default**.
- A SPAN session **normally strips 802.1Q tags and Layer 2 protocols** (STP BPDUs, CDP,
  VTP, DTP, PAgP, LACP). **`encapsulation replicate`** keeps them.
- A SPAN destination port **normally only transmits and drops ingress traffic**. The
  `ingress {dot1q vlan | untagged vlan | vlan}` option re-enables receive, which is what
  you need when the analyzer is a Windows PC you reach over RDP.
- **RSPAN VLAN behaves differently from a normal VLAN:** MAC addresses are **not learned**
  on its ports (so the switch never tries to use an RSPAN port to reach a real host), and
  traffic is **flooded out all ports associated with it**.
- RSPAN duplicates traffic onto a **trunk link**, which can starve normal traffic. **STP
  runs on the RSPAN VLAN and its BPDUs cannot be filtered**, because filtering them could
  create a forwarding loop.

**IP SLA**
- A tool built into Cisco IOS for **continuous** monitoring. Probe types cover: delay
  (round-trip and one-way), jitter (directional), packet loss (directional), packet
  sequencing, path (per hop), connectivity (directional), server/website download time,
  and voice quality scores.
- **Why it beats a provider SLA:** a service provider's SLA only monitors traffic *while
  it crosses the provider's network*. That is not end-to-end visibility. IP SLA measures
  customer-site to customer-site, across the provider and both edges.
- An operation is **configured** and then must be **scheduled** — an unscheduled IP SLA
  operation does nothing at all.
- Results can be read via the **CISCO-RTTMON-MIB** with SNMP, and traps sent to an NMS.
- IP SLA can also track reachability, monitor interface states, and **manipulate routing
  based on the operation's result** (the object-tracking pattern).

**Cisco DNA Center Assurance**
- Capabilities span far beyond monitoring: SD-Access fabric configuration, **software
  image management (SWIM)**, simplified device provisioning, wireless network management,
  simplified security policies, configuration templates, third-party integration, network
  assurance, and **Plug and Play**.
- Assurance encodes **30+ years of Cisco TAC experience** and uses machine learning to
  diagnose issues — then supplies **guided remediation steps**, not just an alert.
- **Network Time Travel** is a DVR for the network: it records the environment with
  **streaming telemetry**, so "last Tuesday at 3 p.m. I couldn't get on wireless" becomes
  an answerable question instead of a dead end.
- **Client 360** pulls everything about a user or endpoint into one view — device type, OS
  version, MAC, IPv4, VLAN ID, connectivity status, when last seen, what it is connected
  to, wireless SSID, and last known location — from a single search of the *user's name*,
  because DNAC integrates with **Active Directory and ISE**. Other integrations include
  **ServiceNow and Infoblox**, via DNAC's open APIs and SDKs.
- **Path Trace** is a visual traceroute and diagnostic run periodically or continuously at
  a set refresh interval. It detects an **ACL blocking the traffic** and, on hover, shows
  the **ACL name, the interface it is applied to, the direction (ingress/egress), and the
  result (permit/deny)**.
- An issue view shows **Impact of Last Occurrence** (which building, how many clients), a
  detailed description, and **Suggested Actions**. With ServiceNow integration all of it —
  issues, impacted locations, path trace, remediation steps — lands in the helpdesk ticket
  before a human opens it.

## Procedure

**Setting up SNMP on a device to be polled and to send traps:**
1. Define the SNMP host — the NMS to send traps to.
2. Create an access list to restrict access via SNMP.
3. Define the read-only community string.
4. Define the read/write community string.
5. Define the SNMP location.
6. Define the SNMP contact.

*These do not have to be done in any particular order.* **But configure the access list
first, then the RO and RW strings bound to it** — that way the moment the device is
reachable by SNMP it is already locked to the permitted hosts.

**Enabling SNMP traps toward the NMS:**
1. Enable the traps themselves — `snmp-server enable traps` turns on *all* of them, which
   is usually more than the operations team wants. Use `snmp-server enable traps ?` to see
   what this platform actually supports (the list is platform specific) and enable only
   the ones that matter.
2. Point the traps at the NMS host with the chosen community string —
   `snmp-server host <nms-ip> traps <community>`.

**Enabling logging to the buffer:**
1. Enable logging to the buffer.
2. Set the severity level of syslog messages to send to the buffer.
3. Set the logging buffer to a larger size (default is only 4096 bytes).

**Sending syslog to an off-box collector:**
1. Enable logging to host `<collector-ip>`.
2. Set the severity level of syslog messages to send to the host.

**Using the logging buffer for debugging instead of the console:**
1. `no logging console` — stop debug output from flooding the console.
2. Confirm the buffer is at debugging level and large enough (above).
3. Run the `debug` command.
4. `show logging` to read the captured output.
5. `undebug all` when finished.

**Configuring traditional NetFlow with export:**
1. Set the export version — `ip flow-export version 9`.
2. Set the export destination and UDP port — `ip flow-export destination <collector> <port>`.
3. On each interface to be watched, enable capture — `ip flow ingress` and/or `ip flow egress`.
4. Verify with `show ip flow interface`, `show ip flow export`, and `show ip cache flow`.

*(If the data is not being exported to a collector, step 1 and 2 can be skipped.)*

**Configuring NetFlow top talkers:**
1. Enter top-talkers config — `ip flow-top-talkers`.
2. Set the number of talkers to track — `top <1-200>`.
3. Set the sort order — `sort-by bytes` or `sort-by packets`, depending on the use case.
4. Verify with `show ip flow top-talkers`.

**Configuring a custom Flexible NetFlow flow record:**
1. Define the flow record name.
2. Set a useful description of the flow record.
3. Set match criteria for key fields.
4. Define non-key fields to be collected.

**Configuring a Flexible NetFlow flow exporter:**
1. Define the flow exporter name.
2. Set a useful description of the flow exporter.
3. Specify the destination of the flow exporter to be used.
4. Specify the NetFlow version to export.
5. Specify the UDP port.

**Configuring a Flexible NetFlow flow monitor:**
1. Define the flow monitor name.
2. Set a useful description of the flow monitor.
3. Specify the flow record to be used.
4. Specify a cache timeout of 60 for active connections (exports the cache to the
   collector every 60 seconds).
5. Assign the exporter to the monitor.

**Turning Flexible NetFlow on (the final step):**
1. Apply the flow monitor to the desired interfaces — `ip flow monitor <name> input`
   (and/or `output`). **Until this is done, nothing is collected.**
2. Verify with `show flow monitor <name> cache`.

*Order matters across these four FNF procedures: record → exporter → monitor (which
references both) → interface application.*

**Configuring a local SPAN session:**
1. Specify the source — `monitor session <id> source {interface <intf> | vlan <vlan-id>}
   [rx | tx | both]`. Multiple interfaces or VLANs take a comma (list) or hyphen (range);
   repeating the command with a different value also updates the source range.
2. If the source is a trunk port, restrict what is captured —
   `monitor session <id> filter vlan <vlan-range>`.
3. Specify the destination — `monitor session <id> destination interface <intf>`, adding
   `encapsulation replicate` to keep 802.1Q tags and Layer 2 protocols, and/or
   `ingress {dot1q vlan <id> | untagged vlan <id> | vlan <id>}` if the analyzer needs to
   send as well as receive.
4. Verify with `show monitor session {<session-id> [detail] | local [detail]}`.

**Configuring an RSPAN session:**
1. Create the RSPAN VLAN **on every switch in the path** — `vlan <id>`, `name <name>`,
   `remote-span`. The VLAN ID must be the same on all of them.
2. On the **source** switch, select the source ports exactly as for local SPAN.
3. On the **source** switch, set the destination to the RSPAN VLAN —
   `monitor session <id> destination remote vlan <rspan-vlan-id>`.
4. On the **destination** switch, set the source to the RSPAN VLAN —
   `monitor session <id> source remote vlan <rspan-vlan-id>`.
5. On the **destination** switch, select the destination port as for local SPAN.
6. Verify — `show monitor session <id>` on the destination switch shows
   *Remote Destination Session*; `show monitor session remote` on the source switch shows
   *Remote Source Session*.

*The session-id is only locally significant, but keeping it the same on both switches
prevents confusion.*

**Configuring an ERSPAN source session:**
1. Create the session and declare its type — `monitor session <n> type erspan-source`.
2. Set a useful description documenting the session's purpose — `description <text>`.
3. Define the source — `source {interface <type number> | vlan <vlan-ID>} [, | - | both | rx | tx]`.
4. If the source is a trunk port, filter it —
   `filter {ip {<std-acl> | <ext-acl> | <acl-name>} | ipv6 {access-group <acl-name>} | vlan <vlan-ID>}`.
5. **Enable the session — `no shutdown`.** An ERSPAN source session is administratively
   down until this is issued.
6. Enter destination subconfiguration mode — `destination`.
7. Set the analyzer's IP address — `ip address <ip-address>`.
8. Set the unique session identifier — `erspan-id <erspan-ID>`.
9. Set the source/origin of the ERSPAN traffic — `origin ip address <ip-address>`.
10. From global config, assign ToS or TTL to the ERSPAN traffic —
    `erspan {tos <tos-value> | ttl <ttl-value>}`.
11. Verify with `show monitor session erspan-source session`.

**Configuring an IP SLA ICMP echo operation:**
1. Enter IP SLA configuration mode for a specific operation number — `ip sla <operation-number>`.
   (Multiple IP SLA instances can run on one device, each doing a different job — the
   number is what separates them.)
2. Configure the probe — `icmp-echo {<dest-ip> | <dest-hostname>} [source-ip {<ip> | <hostname>} | source-interface <intf>]`.
3. Set how often it runs — `frequency <seconds>`.
4. **Schedule and activate it** — `ip sla schedule <operation-number> [life {forever | <seconds>}]
   [start-time {[hh:mm:ss] [month day | day month] | pending | now | after hh:mm:ss}]
   [ageout <seconds>] [recurring]`.
5. Verify with `show ip sla configuration <operation-number>` — check *Type of operation*,
   *Target address/Source interface*, *Operation frequency*, and that
   *Next Scheduled Start Time* reads "Start Time already passed".

**Configuring an IP SLA HTTP GET operation:**
1. Enter IP SLA configuration mode — `ip sla <operation-number>`.
2. Configure the probe — `http {get | raw} <url> [name-server <ip>] [version <version-number>]
   [source-ip {<ip> | <hostname>}] [source-port <port>] [cache {enable | disable}] [proxy <proxy-url>]`.
3. Set how often it runs — `frequency <seconds>`.
4. Schedule it — `ip sla schedule <operation-number> [life {forever | <seconds>}] [start-time ...]`.
5. Verify with `show ip sla configuration <operation-number>`.

**Triaging a user-reported issue in DNA Center Assurance (the workflow that replaces
device-by-device `show`):**
1. Search the **user's name** in the Assurance search box — AD/ISE integration resolves it
   to every device that user owns.
2. Open **Client 360** for the affected device; read the timeline (Network Time Travel) at
   the time the user reported the problem, not just now.
3. Read the **Issues** list correlated to that point on the timeline.
4. Run a **Path Trace** from source to destination to see the topology path and whether an
   ACL is dropping the traffic (hover for ACL name, interface, direction, result).
5. Open the issue for **Impact of Last Occurrence** (locations and client count), the root
   cause description, and the **Suggested Actions**.

## Reference Tables

**Table 24-7 — Syslog message severity levels**

| Level keyword | Level | Description | syslog definition |
|---|---|---|---|
| emergencies | 0 | System unstable | LOG_EMERG |
| alerts | 1 | Immediate action needed | LOG_ALERT |
| critical | 2 | Critical conditions | LOG_CRIT |
| errors | 3 | Error conditions | LOG_ERR |
| warnings | 4 | Warning conditions | LOG_WARNING |
| notifications | 5 | Normal but significant conditions | LOG_NOTICE |
| informational | 6 | Informational messages only | LOG_INFO |
| debugging | 7 | Debugging messages | LOG_DEBUG |

**Table 24-8 — NetFlow ingress and egress collected traffic types (NetFlow v9 on IOS-XE)**

| Ingress | Egress |
|---|---|
| IP to IP packets | NetFlow accounting for all IP traffic packets |
| IP to MPLS packets | MPLS to IP packets |
| Frame Relay terminated packets | — |
| ATM terminated packets | — |

**NetFlow flow key fields — what makes a flow distinct**

| Key field |
|---|
| Source IP address |
| Destination IP address |
| Source port number |
| Destination port number |
| Layer 3 protocol type |
| Type of service (ToS) |
| Input logical interface |

**Table 24-9 — Flexible NetFlow components**

| Component name | Description |
|---|---|
| Flow Records | Combination of key and non-key fields. There are predefined and user-defined records. |
| Flow Monitors | Applied to the interface to perform network traffic monitoring. |
| Flow Exporters | Exports NetFlow Version 9 data from the Flow Monitor cache to a remote host or NetFlow collector. |
| Flow Samplers | Samples partial NetFlow data rather than analyzing all NetFlow data. |

**Table 24-10 — Flow record key and non-key fields**

| Field | Key or non-key | Definition |
|---|---|---|
| IP ToS | Key | Value in the type of service (ToS) field |
| IP protocol | Key | Value in the IP protocol field |
| IP source address | Key | IP source address |
| IP destination address | Key | IP destination address |
| Transport source port | Key | Value of the transport layer source port field |
| Transport destination port | Key | Value of the transport layer destination port field |
| Interface input | Key | Interface on which the traffic is received |
| Flow sampler ID | Key | ID number of the flow sampler (if flow sampling is enabled) |
| IP source AS | Non-key | Source autonomous system number |
| IP destination AS | Non-key | Destination autonomous system number |
| IP next-hop address | Non-key | IP address of the next hop |
| IP source mask | Non-key | Mask for the IP source address |
| IP destination mask | Non-key | Mask for the IP destination address |
| TCP flags | Non-key | Value in the TCP flag field |
| Interface output | Non-key | Interface on which the traffic is transmitted |
| Counter bytes | Non-key | Number of bytes seen in the flow |
| Counter packets | Non-key | Number of packets seen in the flow |
| Time stamp system uptime first | Non-key | System uptime (ms since boot) when the first packet was switched |
| Time stamp system uptime last | Non-key | System uptime (ms since boot) when the last packet was switched |

**SPAN / RSPAN / ERSPAN compared**

| | Local SPAN | RSPAN | ERSPAN |
|---|---|---|---|
| Source and destination | Same switch | Different switches | Different devices, possibly different sites |
| Transport between them | None — internal | **Layer 2**, across a dedicated **RSPAN VLAN** on trunks | **Layer 3** — routed to the analyzer's IP |
| Extra prerequisite | None | RSPAN VLAN created with `remote-span` on **every** switch in the path | Session `type erspan-source`, `erspan-id`, `origin ip address`, and `no shutdown` |
| Key risk | Destination port saturation | STP runs on the RSPAN VLAN and its BPDUs **cannot** be filtered; duplicated traffic can starve the trunk | Routed copy of production traffic crossing the WAN |
| Session command | `monitor session <id> source/destination interface` | `... destination remote vlan` / `... source remote vlan` | `monitor session <n> type erspan-source` |

**SNMP versions at a glance**

| Version | Authentication | Encryption | Notable additions |
|---|---|---|---|
| v1 | Community string (cleartext) | None | Original get/getnext/set/trap |
| v2c | Community string (cleartext) | None | `getbulk`, `inform` (acknowledged notification) |
| v3 | User-based, with authentication | **Yes** (privacy) | Security levels: noAuthNoPriv, authNoPriv, authPriv |

**Table 24-4 — OSPF network types and hello/dead intervals**
*(Referenced by the chapter's Key Topics table as part of the troubleshooting material —
mismatched timers are a classic "adjacency won't form" root cause.)*

| OSPF network type | Hello interval | Dead interval |
|---|---|---|
| Broadcast | 10 sec | 40 sec |
| Non-broadcast | 30 sec | 120 sec |
| Point-to-point | 10 sec | 40 sec |
| Point-to-multipoint | 30 sec | 120 sec |
| Point-to-multipoint non-broadcast | 30 sec | 120 sec |

## Config Patterns
```ios-xe
! ============================================================================
! SNMP - ACL FIRST, then the community strings bound to it
! ============================================================================
access-list 99 permit 192.168.14.100 0.0.0.0
snmp-server community <RO-STRING> ro 99    ! substitute a non-guessable string
snmp-server community <RW-STRING> rw 99    ! RW is a CONFIG channel - guard the ACL
!
snmp-server location Building3-IDF2-Rack4
snmp-server contact netops@example.com
!
! See what this platform actually supports before enabling everything:
!   snmp-server enable traps ?
snmp-server enable traps config
snmp-server host 192.168.14.100 traps <RO-STRING>

! ============================================================================
! syslog - buffer, then off-box collector
! ============================================================================
logging buffer 100000                 ! default is only 4096 bytes
logging buffer debugging              ! severity 7; "logging buffered 7" is identical
no logging console                    ! keep the console usable while debugging
!
logging host 192.168.14.100
logging trap 7                        ! severity sent to the collector (UDP 514 default)
! do show logging

! ============================================================================
! Debugging into the buffer, then clean up
! ============================================================================
! R1# debug ip ospf hello
! R1# show logging
! R1# undebug all                     ! NEVER leave a debug running

! ============================================================================
! Traditional NetFlow + export
! ============================================================================
ip flow-export version 9
ip flow-export destination 192.168.14.100 9999
!
interface Ethernet0/1
 ip flow ingress
 ip flow egress
!
ip flow-top-talkers
 top 10
 sort-by bytes

! ============================================================================
! Flexible NetFlow - record -> exporter -> monitor -> interface
! ============================================================================
flow record CUSTOM1
 description Custom Flow Record for IPv4 Traffic
 match ipv4 destination address        ! match = KEY fields
 collect counter bytes                 ! collect = NON-KEY fields
 collect counter packets
!
flow exporter CUSTOM1
 description EXPORT-TO-NETFLOW-COLLECTOR
 destination 192.168.14.100
 export-protocol netflow-v9
 transport udp 9999
!
flow monitor CUSTOM1
 description Uses Custom Flow Record CUSTOM1 for IPv4 Traffic
 record CUSTOM1
 exporter CUSTOM1
 cache timeout active 60
!
interface Ethernet0/1
 ip flow monitor CUSTOM1 input         ! nothing is collected until this line exists
interface Ethernet0/2
 ip flow monitor CUSTOM1 input

! ============================================================================
! Local SPAN - basic, trunk-source-with-filter, and ingress-enabled variants
! ============================================================================
monitor session 1 source interface gi1/0/1 - 2
monitor session 1 destination interface gi1/0/9
!
! Trunk source: keep tags/L2 protocols, and restrict to one VLAN
monitor session 1 source interface gi1/0/10
monitor session 1 destination interface Gi1/0/9 encapsulation replicate
monitor session 1 filter vlan 123
!
! Analyzer reached over RDP - destination port must also receive
monitor session 1 source interface gi1/0/1
monitor session 1 destination interface gi1/0/2 ingress untagged vlan 123

! ============================================================================
! RSPAN - the RSPAN VLAN must exist on EVERY switch in the path
! ============================================================================
! --- on SW1 and SW2 both ---
vlan 99
 name RSPAN_VLAN
 remote-span
!
! --- source switch (SW2) ---
monitor session 1 source interface gi1/0/3
monitor session 1 destination remote vlan 99
!
! --- destination switch (SW1) ---
monitor session 1 source remote vlan 99
monitor session 1 destination interface gi1/0/9

! ============================================================================
! ERSPAN source session - note the "no shutdown"
! ============================================================================
monitor session 1 type erspan-source
 description SOURCE-PC-D-TRAFFIC
 source interface GigabitEthernet 1/0/4 rx
 filter vlan 34
 no shutdown                          ! session is admin-down without this
 destination
  ip address 10.123.1.100             ! the traffic analyzer
  erspan-id 2
  origin ip address 10.34.1.4
 exit
!
erspan ttl 32

! ============================================================================
! IP SLA - configure, then SCHEDULE (unscheduled = does nothing)
! ============================================================================
ip sla 1
 icmp-echo 192.168.14.100 source-interface Loopback0
 frequency 300
!
ip sla schedule 1 life forever start-time now
!
ip sla 2
 http get http://192.168.14.100
 frequency 90
!
ip sla schedule 2 start-time now life forever
```

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Configure the SNMP **access list first**, then bind the RO and RW community strings to it | The moment the device becomes reachable by SNMP it is already locked down to the allowed hosts — the reverse order leaves a window where any host can poll or write | Bootstrap/staging on an isolated build VLAN where the ACL is applied before the device ever reaches production | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("SNMP", p. 699) |
| Permit **only the specific NMS host addresses**, not the whole subnet | Permitting the subnet is explicitly called out as "more of a security risk"; SNMP RW is a configuration channel | A large NMS cluster with churning addresses where a tightly-scoped subnet is a documented, accepted risk | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 699) |
| Use SNMP community strings that are **not easy to guess** | v1/v2c community strings cross the wire in cleartext and are the entire authentication mechanism | None worth having; the real fix is moving to SNMPv3 with authPriv | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 699) |
| **Be selective about which traps you enable** rather than issuing a bare `snmp-server enable traps` | The bare command enables traps with no significance to the operations team, burying the ones that matter; available traps are platform specific, so check with `?` or the platform docs | A greenfield NMS onboarding exercise where you deliberately collect everything for a bounded period to decide what matters | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 700) |
| **Configure the clock (NTP) correctly before configuring any logging** | Timestamps that don't reflect real time make it impossible to correlate issues with logs — which is the entire point of having them | None. An air-gapped lab may run without NTP, but then the logs are not evidence | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("syslog", p. 701) |
| **Expand the logging buffer** beyond the 4096-byte default | "This size can get overwritten quite quickly" — a small buffer silently loses the messages you went looking for | Memory-constrained platforms where the buffer competes with forwarding resources | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 702) |
| Send debug output to the **logging buffer rather than the console** | Debugging doesn't interfere with console output, making the device workable while you troubleshoot — as long as the debugging level is not also set on the console | Console-only access during a recovery scenario where the buffer isn't reachable | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 702) |
| **Investigate the platform's default NetFlow cache size before enabling NetFlow** | NetFlow consumes memory resources; the default cache size is platform specific, "especially the case with older platforms that potentially have lower memory resources" | None — this is a pre-flight check, not a policy | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("NetFlow and Flexible NetFlow", p. 706) |
| Give every FNF **flow record, exporter, and monitor a real description** | The description appears in `show` output and in context-sensitive help, so the config self-documents the intent of the policy — the same reason descriptions matter in QoS | None | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (p. 713) |
| Do **not reuse a SPAN destination port** across sessions, and size it against the sources | The destination cannot be reused between two different SPAN sessions; 10G sources into a 1G destination will drop packets on the destination port | None for reuse (it is a hard limit); deliberate sampling-by-oversubscription is an accepted trade-off only if you know you are doing it | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("Local SPAN", p. 717) |
| Treat the **SPAN destination port as a loop risk** | STP is disabled on the destination port to keep extra BPDUs out of the analysis — "Great care should be taken to prevent a forwarding loop on this port" | None | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 (NOTE, p. 719) |
| **Do not associate the RSPAN VLAN with any port that is not a trunk** between the source and destination switches | Traffic is flooded out all ports associated with the RSPAN VLAN; MAC addresses are not learned on them | None | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("Remote SPAN", pp. 720–721) |
| **Filter a trunk source port to specific VLANs** when using it as a SPAN source | A trunk source captures every VLAN traversing the port, which "might provide too much data and add noise to the traffic analysis tool" | An intentional wide capture when you do not yet know which VLAN the problem is in | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("Specifying the Source Ports", p. 718) |
| Use **IP SLA rather than relying on the provider's SLA** for end-to-end path health | A provider SLA only monitors traffic as it flows across the provider's network — that is not end-to-end visibility | None; the provider SLA is still the contractual instrument, IP SLA is the evidence | CCNP/CCIE Enterprise Core ENCOR 350-401 OCG, Ch. 24 ("IP SLA", p. 724) |

*A deviation from this table is a question for the network's operator — "is this
intentional here?" — never automatically a finding.*

## Verification Commands

| Command / GUI path | What to look for |
|---|---|
| `ping` (extended, no arguments) | Set source, size, DF bit, repeat — the only way to test MTU and source-dependent reachability |
| `traceroute <dest>` | The **last hop that answers**; `* * *` alone is not proof of a break (ICMP may be rate-limited) |
| `show logging` | Every destination at once: console/monitor/buffer/trap levels, buffer size, collector IP and UDP port, and the buffered messages themselves |
| `debug condition interface <intf>` / `debug condition ip <addr>` | Scope debugging before you enable it — the difference between troubleshooting and an outage |
| `undebug all` | "All possible debugging has been turned off" — run it every time |
| `terminal monitor` | Required on a VTY session (SSH/Telnet) to see debug/log output at all |
| `show snmp` | Agent status and packet counters — proves whether the NMS is even reaching the device |
| `show snmp community` / `show snmp host` | Which strings exist, which ACL each is bound to, and where traps are being sent |
| `show ip flow interface` | Which interfaces have `ip flow ingress` / `ip flow egress` actually applied |
| `show ip flow export` | Export version, destination and port, flows exported, and the **drop counters** (no fib, adjacency, fragmentation, encapsulation fixup) |
| `show ip cache flow` | The live flows NetFlow is capturing, plus packet size distribution and cache utilization |
| `show ip flow top-talkers` | The loudest N flows sorted by bytes/packets — fastest "what is eating this link" answer |
| `show flow record [<name>]` | Custom and predefined records; bare form lists all |
| `show running-config flow record` / `flow exporter` / `flow monitor` | What was actually committed, useful when `show flow ...` output looks right but traffic isn't collected |
| `show flow exporter <name>` | Export protocol, destination/source IP, transport, destination port, DSCP, TTL |
| `show flow monitor <name>` | Which record and exporter are bound, and the cache timeouts (inactive 15 / active 60 / update 1800 / synchronized 600). **`Flow Exporter: <name> (inactive)` means the monitor is not yet applied to an interface** |
| `show flow monitor <name> cache` | Cache type/size, current entries, high watermark, flows added/aged — and the per-flow data itself. Empty cache with a healthy config = the monitor was never applied to an interface |
| `show monitor session <id>` | Type (Local / Remote Source / Remote Destination), source ports and direction, destination, encapsulation (Native vs Replicate), ingress state, filter VLANs |
| `show monitor session local [detail]` | Restrict output to local SPAN sessions only |
| `show monitor session remote` | On an RSPAN source switch: *Remote Source Session* and the Dest RSPAN VLAN |
| `show monitor session erspan-source session` | ERSPAN: **Status: Admin Enabled** (proves `no shutdown` was issued), RX/TX source ports, destination IP, ERSPAN ID, origin IP |
| `show ip sla configuration <n>` | Type of operation, target/source, frequency, and **Next Scheduled Start Time** — "Start Time already passed" is what proves it was scheduled |
| `show ip sla statistics [<n>]` | Latest operation return code and RTT — the actual result, as opposed to the configuration |
| **DNAC → Assurance → Client 360** | Per-user/endpoint health, onboarding, Issues correlated to a point in time (Network Time Travel) |
| **DNAC → Assurance → Device 360** | Device resource usage, loss, latency |
| **DNAC → Client 360 → Run New Path Trace** | Visual traceroute; hover an ACL entry for name, interface, direction, and permit/deny result |

## Intent Questions
- **What is this instrumentation supposed to be telling someone, and who reads it?** An
  enabled trap or a syslog collector nobody watches is not assurance, it is noise with
  storage costs.
- **Which tool is the right one for the symptom?** "Is it up" is ping; "where does it
  stop" is traceroute; "who is using the bandwidth" is NetFlow; "what is literally on the
  wire" is SPAN; "was it bad at 3 p.m. last Tuesday" is IP SLA or DNAC Assurance. Reaching
  for `debug` first is the classic mistake.
- **Is the device's clock correct, and is NTP healthy?** Every timestamp in every log and
  flow record depends on it. Answer this before reading any output.
- **Is SNMP here read-only or read-write, and which hosts are permitted?** RW is a
  configuration channel; if the ACL is loose, the "monitoring" system is a change system.
- **For NetFlow/SPAN: what is this device's resource headroom?** NetFlow eats memory, SPAN
  can saturate a destination port, and debug eats CPU. Assurance tooling that takes the
  device down has failed at its job.
- **For an IP SLA operation: was it scheduled?** Configured-but-unscheduled is the single
  most common reason an IP SLA "isn't working."

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this network, then
   write the one-line symptom ("should ___, isn't ___") — before running any show command
   or enabling any instrumentation.
1. **Check the clock first.** `show clock` and NTP status. Wrong time makes every log,
   flow record, and Assurance timeline unusable for correlation, and you will chase
   ghosts.
2. **Reachability before instrumentation.** `ping` from the *correct source interface*
   (extended ping), then `traceroute`. If the collector/NMS is unreachable from the
   device's actual source address, nothing downstream will work.
3. **Confirm the tool is applied, not just configured.** The most common failure in this
   whole chapter: `ip flow ingress`/`ip flow monitor ... input` missing from the
   interface, an IP SLA never scheduled, an ERSPAN session never `no shutdown`. Config
   that parses is not config that runs.
4. **Check the destination side.** Is the collector listening on the port you exported to
   (NetFlow UDP 9999 in the examples, syslog UDP 514 by default)? `show ip flow export`
   drop counters and `show logging` trap counters tell you whether the device thinks it
   sent anything.
5. **Check ACLs and firewalls in the path** between device and NMS/collector/analyzer —
   SNMP, syslog, NetFlow export, and ERSPAN are all just UDP/IP and all get dropped
   silently.
6. **SNMP specifics:** community string exact match (case sensitive), the right one for
   the operation (RO vs RW), and the bound ACL permitting the NMS's *actual* source
   address. Then whether the required traps were enabled at all.
7. **syslog specifics:** is the severity level at the destination high enough to include
   the messages you want? A level includes everything below it — `logging trap 3` will
   never show an informational message. Is `logging console` disabled while you're
   expecting console output, or `terminal monitor` missing on a VTY session?
8. **NetFlow specifics:** interfaces applied (`show ip flow interface`), export destination
   and version (`show ip flow export`), and cache actually populating (`show ip cache
   flow` / `show flow monitor <name> cache`). For FNF, walk the chain: record → exporter →
   monitor → interface, and check for `Flow Exporter: <name> (inactive)`.
9. **SPAN specifics:** destination port reused across sessions (not allowed), destination
   oversubscribed by the sources, trunk source not filtered to a VLAN so the analyzer is
   drowning, or missing `encapsulation replicate` when you needed the tags and BPDUs.
   For RSPAN, confirm the RSPAN VLAN exists with `remote-span` on **every** switch in the
   path and is allowed on the trunks.
10. **IP SLA specifics:** scheduled (`show ip sla configuration` → "Start Time already
    passed"), correct source interface, and — for operations that need one — an
    `ip sla responder` on the far end.
11. **Only now, debug.** Scope it with `debug condition` or an ACL, send it to the buffer
    rather than the console, capture what you need, and `undebug all`. On a production
    device assume unqualified debug will hurt.
12. **Escalate to the correlated view.** If the device-level tools disagree with the user's
    report, DNAC Assurance Client 360 + Network Time Travel can show what was happening at
    the moment they complained, which no live `show` command can reconstruct.

## Common Pitfalls
- **`debug` is the last resort, not the first.** It is process-switched and CPU-expensive;
  an unqualified debug on a busy production device can take it down. Condition it, buffer
  it, and `undebug all` afterwards — every time.
- **By default all syslog messages go to the console**, which is why debug output shows up
  there and why the console becomes unusable during a debug.
- **Over SSH/Telnet you see nothing without `terminal monitor`** — the debug is running,
  you just aren't being shown it. People conclude the debug "doesn't work."
- **A severity level includes every level below it.** Setting `logging trap 7` sends
  everything; setting `logging trap 3` silently discards warnings, notifications, and
  informational messages. Numerically lower = more severe.
- **Severity can be set by number or keyword** — `logging buffered 7` and
  `logging buffered debugging` are identical. Do not treat them as different features.
- **The default logging buffer is 4096 bytes** and is overwritten quickly. The evidence you
  went looking for may simply have scrolled out.
- **Timestamps are worthless if the clock is wrong.** Configure NTP *before* logging. The
  OCG raises this and then explicitly notes NTP is not covered in this chapter — don't
  read that as "it doesn't matter."
- **NetFlow has two components and people configure only one.** Data Capture without Data
  Export gives local visibility with nothing at the collector; export configured without
  `ip flow ingress`/`ip flow egress` on any interface gives a collector with nothing in it.
- **A flow is unidirectional.** Client-to-server and server-to-client are two flows, not
  one — this is why byte counts look "half" of what people expect.
- **`match` = key fields, `collect` = non-key fields.** Swapping them is the most common
  FNF config error, and the record will still be accepted.
- **A Flexible NetFlow monitor does nothing until it is applied to an interface.**
  `show flow monitor <name>` showing `Flow Exporter: <name> (inactive)` and a cache with
  `Status: not allocated` is exactly this — the config chain is complete but never turned on.
- **NetFlow consumes memory, and the default cache size is platform specific.** Check it
  before enabling, especially on older hardware.
- **Sampled NetFlow trades accuracy for load.** Less memory and CPU, but things can be
  missed — that is a business decision, not a free optimization.
- **A local SPAN VLAN source does not include the SVI.** It is the traffic received by the
  switch for the hosts in that VLAN.
- **A SPAN destination port cannot be reused between sessions**, even though a *source*
  port can be.
- **SPAN strips 802.1Q tags and Layer 2 protocols by default** (STP BPDUs, CDP, VTP, DTP,
  PAgP, LACP). If you are troubleshooting a Layer 2 problem and the captures look
  suspiciously clean, you needed `encapsulation replicate`.
- **A SPAN destination port normally drops everything it receives.** If the analyzer is a
  Windows PC you reach over RDP, you must add the `ingress` option or you will lose access
  to the analyzer the moment the session comes up.
- **STP is disabled on the SPAN destination port** — which is exactly why a mis-patched
  destination port can create a forwarding loop.
- **The RSPAN VLAN must exist with `remote-span` on every switch in the path**, with the
  same VLAN ID, and must not touch any non-trunk port. MAC addresses are not learned on it
  and traffic floods out every associated port.
- **STP BPDUs cannot be filtered on the RSPAN VLAN** — filtering them could introduce a
  forwarding loop. RSPAN also duplicates traffic onto the trunk and can starve production
  traffic.
- **An ERSPAN source session is administratively down until `no shutdown`.** The config
  will look complete and capture nothing.
- **An IP SLA operation that is configured but never scheduled does nothing.** `ip sla
  schedule` is a separate command, and `show ip sla configuration` showing
  "Next Scheduled Start Time: Start Time already passed" is the proof it ran.
- **SNMPv1 and v2c carry the community string in cleartext and offer no encryption.** "We
  use SNMP for read-only so it's safe" ignores that the string itself is sniffable and the
  same string is often reused on devices that do have RW.
- **SNMP RW is a configuration channel, not a monitoring one.** Treat its ACL with the same
  seriousness as VTY access.
- **`snmp-server enable traps` with no keyword enables everything**, including traps with
  no significance to the operations team. The available set is platform specific — check
  with `?` rather than assuming.
- **`sysName`, `sysLocation`, and `sysContact` are `read-write` MIB objects** — an NMS with
  the RW string can change them. `sysDescr`, `sysObjectID`, and `sysUpTime` are read-only.
- **DNA Center Assurance's value is correlation and time travel, not prettier graphs.**
  The thing no CLI can do is answer "what was happening at 3 p.m. last Tuesday" — that is
  streaming telemetry recorded over time, and it is the reason to reach for it.
- **Path Trace shows ACL drops explicitly** (name, interface, direction, permit/deny),
  which is often the answer people spend an hour hunting for with `show access-lists`
  across multiple hops.

## Exam Preparation Tasks

### Key topics coverage map

Table 24-11, mapped to where each element actually lives in this skill. The mapping is
deliberately not one-to-one: several key topics collapse into a single Procedure or
Reference Table here, and a few are split between Key Concepts and Common Pitfalls.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Section | ping | 675 | Key Concepts → "Which tool answers which question" and "ping and traceroute" (extended ping, source/size/DF, ping sweep); Verification Commands. **Partial** — captured at concept level; the chapter's worked `ping` examples were not transcribed |
| Section | traceroute | 680 | Key Concepts → "ping and traceroute" (incrementing-TTL mechanism, per-hop RTT); Troubleshooting step 2; Common Pitfalls (`* * *` is not proof of a break). **Partial** — concept level, worked examples not transcribed |
| Section | Debugging | 685 | Key Concepts → "Debugging"; Procedure → "Using the logging buffer for debugging instead of the console"; Troubleshooting step 11; four Common Pitfalls bullets. **Partial** — conditional-debug syntax captured generically, the chapter's specific examples were not transcribed |
| Table 24-4 | OSPF Network Types and Hello/Dead Intervals | 689 | Reference Tables → "Table 24-4 — OSPF network types and hello/dead intervals" (cross-referenced; the OSPF topic itself lives in the `ospf` skill) |
| Section | Simple Network Management Protocol (SNMP) | 695 | Key Concepts → "SNMP" incl. the `system` group MIB object table; Procedure → two SNMP procedures; Config Patterns; Design Baseline rows 1–4; five Common Pitfalls bullets |
| Section | NetFlow and Flexible NetFlow | 706 | Key Concepts → "NetFlow" and "Flexible NetFlow"; Procedure → five NetFlow/FNF procedures; Reference Tables 24-8, key fields, 24-9, 24-10; Design Baseline rows 8–9 |
| Section | Specifying the Source Ports | 717 | Procedure → "Configuring a local SPAN session" step 1–2; Key Concepts → "SPAN technologies" (allowed source types, rx/tx/both default, trunk filtering); Design Baseline row 12 |
| Section | Encapsulated Remote SPAN (ERSPAN) | 722 | Key Concepts → "SPAN technologies"; Procedure → "Configuring an ERSPAN source session" (11 steps); Reference Tables → "SPAN / RSPAN / ERSPAN compared"; Common Pitfalls (`no shutdown`) |
| Section | IP SLA | 724 | Key Concepts → "IP SLA"; Procedure → the ICMP echo and HTTP GET procedures; Design Baseline row 14; Common Pitfalls (unscheduled operation) |
| Section | Cisco DNA Center Assurance | 728 | Key Concepts → "Cisco DNA Center Assurance"; Procedure → "Triaging a user-reported issue in DNA Center Assurance"; Verification Commands (GUI paths) |

**Coverage honesty note.** The four rows marked **Partial** (ping, traceroute, Debugging,
and by extension the SNMP material before p. 695) come from the front half of the chapter,
pages 672–697. Those pages were captured at concept-and-mechanism level rather than
transcribed example by example, so the worked `ping`/`traceroute`/`debug` output walkthroughs
and any conditional-debug syntax specific to the OCG's examples are **not** reproduced here.
Everything from p. 698 onward — the MIB excerpt, all SNMP/syslog/NetFlow/FNF/SPAN/RSPAN/
ERSPAN/IP SLA/DNAC material, and every ordered step list — is captured in full. Re-run
`/ccnp-note network-assurance` against pages 672–697 to close those four rows.

### "Do I Know This Already?" question analysis

**Not captured.** The chapter's opening quiz (and its answer key) was not part of the
source material provided, so there is nothing here to analyze. Per the template's own
rule, inventing traps that aren't there is worse than leaving the subsection empty —
this subsection should be filled in when pages 671–673 are captured.
