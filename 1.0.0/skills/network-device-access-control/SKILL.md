---
name: ccnp-network-device-access-control
description: >
  Use this skill when securing, troubleshooting, or configuring management access to an
  IOS-XE device and the infrastructure-protection features that sit around it. Invoke when
  the user asks about: network device access control, infrastructure security, device
  hardening, ACL, access control list, standard ACL, extended ACL, named ACL, numbered ACL,
  wildcard mask, implicit deny, ip access-group, access-class, applying an ACL to an
  interface, applying an ACL to a vty line, CLI access methods, console, con 0, cty, aux
  port, auxiliary port, line aux 0, no exec, vty, vty lines, line vty 0 4, terminal lines,
  line password protection, password, login, login local, service password-encryption,
  password types, type 0 password, type 4, type 5, type 6, type 7, type 8, type 9, MD5,
  SHA-256, scrypt, PBKDF2, Vigenere, algorithm-type, secret, username secret, local
  username authentication, privilege levels, privilege level 0, privilege level 1,
  privilege level 15, User EXEC, Privileged EXEC, RBAC, role-based access control,
  privilege exec level, privilege configure level, privilege interface level, show
  privilege, transport input, transport input all, transport input none, transport input
  telnet, transport input ssh, reverse Telnet, show line, tty, vty 578, Telnet, SSH, Secure
  Shell, SSHv1, SSHv2, SSH 1.99, ip ssh version 2, crypto key generate rsa, ip domain-name,
  RSA modulus, 768 bits, 2048 bits, Field Notice 72510, FIPS 140-1, FIPS 140-2, EXEC
  timeout, exec-timeout, exec-timeout 0 0, absolute-timeout, logout-warning, idle timeout,
  AAA, authentication authorization accounting, aaa new-model, method list, default method
  list, custom method list, login authentication, aaa authentication login, aaa
  authentication enable, aaa authorization exec, aaa authorization console, aaa
  authorization commands, aaa authorization config-commands, aaa accounting exec, aaa
  accounting commands, start-stop, stop-only, wait-start, if-authenticated, none method,
  local fallback, lockout, TACACS+, Terminal Access Controller Access-Control System Plus,
  TCP port 49, tacacs server, aaa group server tacacs+, RADIUS, Remote Authentication
  Dial-In User Service, UDP 1645, UDP 1646, UDP 1812, UDP 1813, command authorization,
  command accounting, command authorization failed, Cisco ISE, Cisco Secure ACS, ACS,
  Zone-Based Firewall, ZBFW, security zone, zone security, zone-member security, zone pair,
  zone-pair security, self zone, default zone, class-map type inspect, match-all, match-any,
  policy-map type inspect, service-policy type inspect, drop, drop log, pass, pass log,
  inspect, class-default, stateful inspection, stateful firewall, outside-to-self,
  self-to-outside, show policy-map type inspect zone-pair, show class-map type inspect,
  Control Plane Policing, CoPP, control-plane, service-policy input, police, conform-action,
  exceed-action, violate-action, CIR, bc, be, show policy-map control-plane, EPC, Embedded
  Packet Capture, Catalyst 9000 default CoPP, no cdp enable, no lldp transmit, no lldp
  receive, service tcp-keepalive-in, service tcp-keepalive-out, no ip redirects, ICMP
  redirect, no ip proxy-arp, proxy ARP, no service config, MOP, no mop enabled, PAD, no
  service pad.
---

## Purpose
This is the other half of AAA: Chapter 25 controls who gets *onto* the network, this one
controls who gets *into the box* and what the box will do for them. It covers the CLI access
paths (console, aux, vty), how credentials are stored and how weakly, privilege levels and
RBAC, TACACS+-based AAA for device administration, and the two features that protect the
router itself — ZBFW for the data path and CoPP for the control plane — plus the hardening
commands that turn off what you never use.

## Key Concepts

**Access Control Lists (ACLs)**
- ACLs control access based on **protocol, source IP address, destination IP address, and
  ports**. They are **stateless** and do not inspect a packet's payload, so they cannot tell
  whether an attacker is riding a port that happens to be open.
- Applied to an interface with `ip access-group {access-list-number | name} {in | out}`.
- Applied to a **vty line** with `access-class {access-list-number | access-list-name}
  {in | out}` under line configuration mode — `in` for inbound sessions *into* the device,
  `out` for outbound sessions *from* it.
- Best practice for vty: **allow only internal/trusted source addresses**. Permitting
  external or public addresses to reach the vty lines requires extreme care.

**CLI access methods and line password protection**
- IOS-XE devices are administered over the **console (cty)**, the **auxiliary (aux) port**,
  and the **virtual terminal (vty)** lines via Telnet or SSH.
- Line-level protection options: a **line password** (`password` + `login`), **local
  username-based authentication** (`login local`), or **AAA** (covered later in the chapter).
- Username-based authentication for the **aux and cty lines is only supported in combination
  with AAA on some IOS-XE releases** — a real gotcha when a console `login local` silently
  does not behave as expected.

**Password types**
- A password's *type number* is the algorithm used to store it, and it appears in the running
  config right after the keyword:
  - **Type 0** — plaintext, stored exactly as typed. `username type0 password 0 weak` shows up
    as cleartext.
  - **Type 7** — Cisco's reversible Vigenère encoding, produced by `service
    password-encryption`. Obfuscation, not encryption; trivially reversed.
  - **Type 5** — MD5 hash. `username type5 secret 5 $1$...`
  - **Type 8** — PBKDF2 with SHA-256. `algorithm-type sha256`
  - **Type 9** — scrypt. `algorithm-type scrypt`, shown as `$9$...`
- `username {username} algorithm-type {md5 | sha256 | scrypt} secret {password}` is how you
  choose: **md5 → type 5**, **sha256 → type 8**, **scrypt → type 9**.
- `service password-encryption` only upgrades **type 0 to type 7** — it does nothing for
  passwords already stored as a hash, and type 7 is not a security control.

**Privilege levels and role-based access control (RBAC)**
- The IOS-XE CLI has **three default privilege levels**:
  - **Level 0** — the commands `disable`, `enable`, `exit`, `help`, and `logout`.
  - **Level 1** — **User EXEC mode**. Prompt ends in `>` (`R1>`). **`configure terminal` is
    not available**, so no configuration changes are possible.
  - **Level 15** — **Privileged EXEC mode**. Prompt ends in `#` (`R1#`). All CLI commands.
- **Levels 2 through 14** can be configured for customized access, using
  `privilege {mode} level {level} {command string}` in global configuration mode.
- A privilege level can be attached to a user directly:
  `username noc privilege 5 algorithm-type scrypt secret cisco123`.
- **Setting a multi-keyword command to a level automatically sets the commands starting with
  its first keyword to that level too.** Setting `no shutdown` to level 5 also sets `no` to
  level 5 — necessary, because you cannot run `no shutdown` without access to `no`. The
  running config expands one `privilege interface level 5 no shutdown` into several lines
  (`no shutdown`, `no ip address`, `no ip`, `no`).
- Local authentication plus privilege levels gives adequate security but is **cumbersome to
  manage on every device and very likely to drift into inconsistency** — which is the whole
  argument for AAA.

**Controlling vty access with transport input**
- `transport input {all | none | telnet | ssh}` under line config restricts which protocols
  may reach that line. `telnet ssh` together is equivalent to `all`.
- **vty lines are evaluated from the top (vty 0) onward, and each vty line accepts only one
  user.** That makes per-line `transport input` a real capacity constraint, not just a
  filter: if only vty 0, 2, and 4 allow Telnet, the fourth Telnet session is refused even
  though vty 1 and 3 are idle.
- In `show line`, the configured vty numbers map to **tty numbers** — vty 0 → 578, vty 2 →
  580, vty 4 → 582 in the chapter's example. An **asterisk** to the left of a row means the
  line is in use.
- **The AUX port should be given `transport input none` to block reverse Telnet into it.**

**SSH**
- Telnet is the most popular and **most insecure** management protocol — packets are
  plaintext and trivially sniffed.
- **SSHv1** is an improvement over Telnet but has fundamental implementation flaws and should
  be avoided. **SSHv2** is a complete rework, **not compatible with SSHv1**, closes a security
  hole present in v1, and is **certified under NIST FIPS 140-1 and 140-2**.
- The log message `%SSH-5-ENABLED: SSH 1.99 has been enabled` means **both v1 and v2 are
  enabled**. Force v2-only with `ip ssh version 2` in global configuration mode.
- The RSA modulus must be **at least 768 bits for SSHv2**; the prompt offers a range of
  **360 to 4096**. Longer is stronger but slower to generate.
- **From Cisco IOS-XE 17.11.1 onward, RSA keys smaller than 2048 bits are rejected by
  default.** The enforcement can be disabled by configuration but is not recommended — see
  **Field Notice 72510**.

**Auxiliary port and session timeouts**
- The aux port exists for remote administration over a dialup modem. **In most cases it
  should be disabled with `no exec` under `line aux 0`.**
- The **default EXEC idle timeout is 10 minutes**, changed with
  `exec-timeout {minutes} {seconds}`. **`exec-timeout 0 0` and `no exec-timeout` disable the
  timeout entirely — useful in a lab, not recommended in production.**
- `absolute-timeout {minutes}` terminates an EXEC session when the period expires **even if
  the connection is actively in use**. Pair it with `logout-warning {seconds}` so users get a
  "line termination" warning before being cut off.

**AAA — the framework**
- Three independent security functions:
  - **Authentication** — identify and verify a user before granting access to a device and/or
    network services.
  - **Authorization** — define the access privileges and restrictions enforced for an
    authenticated user.
  - **Accounting** — track and log user access: identities, start and stop times, executed
    CLI commands. A security log of events.
- Two use cases, and the protocol choice differs between them:
  - **Network device access control** — logging into the box. **TACACS+ is the protocol of
    choice.**
  - **Secure network access control** — getting onto the network (Chapter 25). **RADIUS is
    the preferred protocol.**
- Because configuration is centralized on the AAA servers, policy is applied consistently
  network-wide — but **local authentication should still be kept as a fallback** for when the
  AAA servers are unavailable.

**TACACS+**
- Cisco developed TACACS+ and released it as an **open standard in the early 1990s**. Mainly
  used for AAA device access control, though it can do some types of AAA network access.
- Uses **TCP port 49**.
- **The key differentiator: TACACS+ separates authentication, authorization, and accounting
  into independent functions.** That is why it dominates device administration even though
  RADIUS is technically capable of device access control.
- **Encrypts the entire payload.** Does **not** support EAP.
- Can authorize **individual CLI commands** and supports **command accounting**.

**RADIUS**
- An **IETF standard** AAA protocol, client/server, client initiates.
- **The AAA protocol of choice for secure network access, because RADIUS is the AAA transport
  for EAP and TACACS+ is not.**
- **RADIUS must return all authorization parameters in a single reply**; TACACS+ can request
  authorization parameters separately and repeatedly throughout a session. This is the
  mechanical reason TACACS+ wins for device administration: per-command authorization over
  RADIUS would mean stuffing thousands of CLI command combinations into the initial
  authentication response, and **a large authorization result list could trigger memory
  exhaustion on the network device**.
- If all you need is AAA **authentication without authorization**, either protocol works.
- **Cisco Secure ACS** was the AAA server of choice for years; **from ISE 2.0 onward ISE took
  over as Cisco's AAA server for both RADIUS and TACACS+.**

**Method lists**
- `aaa authentication login {default | custom-list-name} method1 [method2 ...]`
- The **`default`** keyword applies the method list to **all lines** (cty, tty, aux, and so
  on). A **custom list name** is applied to specific lines with `login authentication
  {custom-list-name}` under line configuration mode, letting different line types use
  different methods.
- **Methods are tried sequentially, left to right.** In `aaa authentication login default
  group ISE-TACACS+ local enable`: TACACS+ first; if the servers are unavailable, local
  username authentication; if no usernames are defined, the `enable` password as last resort.
  **If there is no enable password configured, the user is effectively locked out.**
- In an `aaa group server tacacs+` group, **the order servers are added dictates the failover
  order, top to bottom** — first added is highest priority.

**Zone-Based Firewall (ZBFW)**
- **ZBFW is the latest integrated stateful firewall technology included in IOS XE**, and it
  reduces the need for a separate firewall at a branch site.
- A **stateful** firewall looks into **Layers 4 through 7** to verify the state of the
  transmission, can detect a port being piggybacked, and can mitigate DDoS intrusions — all
  things a stateless ACL cannot do.
- Router interfaces are assigned to a **security zone**, in a **one-to-one or many-to-one**
  relationship. A zone establishes a security border and defines acceptable traffic between
  zones. **By default, interfaces in the same zone communicate freely; interfaces in
  different zones cannot communicate without passing the configured policy.**
- Two **system-built zones** exist:
  - **The self zone** — a system-level zone containing **all the router's own IP addresses**.
    By default traffic **to and from** it is permitted, so management (SSH, SNMP) and control
    plane (EIGRP, BGP) keep working. **Once a policy is applied between the self zone and
    another zone, interzone communication must be explicitly defined** — which is exactly why
    a locally originated ping breaks after you write only an outside-to-self policy.
  - **The default zone** — a system-level zone into which **any interface not a member of
    another security zone is placed automatically**, once the zone is initialized. Traffic
    from a non-zone interface to a zone interface is **dropped**; most engineers assume no
    policy can permit that flow, but enabling the default zone lets you write a policy map
    between the two zones.
- **Class map** classification: `class-map type inspect [match-all | match-any] {name}`.
  **`match-all` is Boolean AND, `match-any` is Boolean OR, and match-all is the default** if
  neither keyword is given.
- **Policy actions** under `policy-map type inspect`:
  - **`drop [log]`** — the **default action**; silently discards matching packets. `log` adds
    syslog with source and destination IP, port, and protocol.
  - **`pass [log]`** — forwards packets from source zone to destination zone, **in one
    direction only**. A separate policy is required for the return direction. Useful for
    IPsec, ESP, and other inherently secure protocols with predictable behavior.
  - **`inspect`** — state-based control. The router maintains connection/session state and
    **permits return traffic without a second policy**.
- The inspect policy map has an **implicit `class-default` with a `drop` action** — the same
  implicit "deny all" as an ACL. Adding it explicitly may simplify troubleshooting for junior
  engineers. **Do not add `log` to the class-default drop** — it can fill the syslog.
- **Zone pair order is significant**: `zone-pair security {name} source {src} destination
  {dst}` — first zone is the source, second is the destination. **A second zone pair is
  needed for bidirectional traffic patterns when the `pass` action is selected.**
- Even though the ACLs referenced by inspect class maps are **not being used to block
  traffic**, their **counters still increment** as packets match — visible in `show ip access`.

**Control Plane Policing (CoPP)**
- CoPP polices traffic destined for the router's control plane to a given rate, minimizing
  the ability to overload the router. Built from ACLs → `class-map match-all` → `policy-map`
  with `police` → applied under `control-plane`.
- **Finding the correct rate without impacting network stability is not simple.** The
  chapter's method: set **`violate-action transmit` for all the vital classes until a
  baseline for normal traffic flows is established**, then tighten over time. Traffic that
  should have genuinely low packet rates — **ICMP and DHCP** — is set to **`violate-action
  drop`** from the start.
- **`class-default` holds unknown traffic.** Under normal conditions nothing should land
  there, but allowing a minimal amount and monitoring the policy **permits discovery of new or
  unknown traffic that would otherwise have been denied**.
- **Embedded Packet Capture (EPC)** is the tool for tweaking the policies; the access lists
  can be **reversed from `permit` to `deny`** as a filter to gather the unexpected traffic.
- The policy must be **tweaked based on the routing protocols actually in use** in the network.
- **Some Cisco platforms, such as the Catalyst 9000 series, ship with a default CoPP policy
  that typically does not require modification.** If it must be modified, consult the
  platform-specific documentation for restrictions and caveats.

**Device hardening**
- Beyond AAA, CoPP, and ZBFW, disabling unused services **improves the security posture by
  minimizing information exposed externally** and **reduces the router CPU and memory spent
  processing unnecessary packets**.
- All interface-specific hardening commands are applied **only to the interface connected to
  the public network**:
  - **Disable topology discovery** — CDP and LLDP hand information to routers outside your
    control: `no cdp enable`, `no lldp transmit`, `no lldp receive`.
  - **Enable TCP keepalives** — `service tcp-keepalive-in` and `service tcp-keepalive-out`
    confirm the remote end is still reachable and remove half-open or orphaned connections.
  - **Disable IP redirects** — an ICMP redirect informs a device of a better path; IOS-XE
    sends one when it detects traffic hairpinning. `no ip redirects`.
  - **Disable proxy ARP** — proxy ARP lets the router answer ARP requests intended for a
    different router. A MitM intrusion can spoof the router's MAC and pull traffic to the
    attacker. `no ip proxy-arp`.
  - **Disable service configuration** — automatic configuration from remote devices via TFTP
    and other methods: `no service config`.
  - **Disable MOP** — `no mop enabled` globally **and** as an interface parameter.
  - **Disable PAD** — the X.25 packet assembler/disassembler service: `no service pad`.

## Procedure

**Configuring SSH on an IOS-XE device:**
1. Configure a hostname other than `Router` with `hostname {hostname name}`.
2. Configure a domain name with `ip domain-name {domain-name}`.
3. Generate crypto keys with `crypto key generate rsa`. You are prompted for a modulus
   length — **at least 768 bits for SSHv2** (and at least 2048 on IOS-XE 17.11.1 and later,
   where smaller keys are rejected by default). Longer modulus, stronger security, longer
   generation time.
4. Enable username-based login on the lines (`login local`, or an AAA method list), and force
   v2-only with `ip ssh version 2` if the `SSH 1.99` log message shows v1 is still enabled.

**Configuring AAA for network device access control (TACACS+), 11 steps:**
1. **Create a local user with full privilege for fallback**, so enabling AAA cannot lock you
   out: `username {username} privilege 15 algorithm-type {md5 | sha256 | scrypt} secret
   {password}`.
2. Enable AAA: `aaa new-model`.
3. Add the TACACS+ server. **Syntax depends on the IOS-XE version:**
   - Prior to 16.12.2: `tacacs-server host {hostname | host-ip-address} key {key-string}`
   - 16.12.2 and later: `tacacs server {name}` / `address ipv4 {hostname | host-ip-address}`
     / `key {key-string}`
4. Create an AAA group: `aaa group server tacacs+ {group-name}` / `server name {server-name}`.
   **Order of addition dictates failover order, top to bottom.**
5. Enable login authentication: `aaa authentication login {default | custom-list-name}
   method1 [method2 ...]`.
6. Enable EXEC authorization: `aaa authorization exec {default | custom-list-name} method1
   [method2 ...]`. **This applies to all lines except the console.**
7. Enable console authorization: `aaa authorization console`. **Disabled by default,
   specifically to stop inexperienced users locking themselves out.**
8. Enable command authorization: `aaa authorization commands {privilege level} {default |
   custom-list-name} method1 [...]`. **Applied per privilege level**, so a method list is
   needed for every level that requires it. **Commonly configured for levels 0, 1, and 15
   only** — levels 2–14 are useful only for *local* authorization via the `privilege level`
   command.
9. Enable command authorization in global config mode and all its submodes:
   `aaa authorization config-commands`.
10. Enable login accounting: `aaa accounting exec {default | custom-list-name} {start-stop |
    stop-only | wait-start} method1 [...]`. **`start-stop` is the common choice** — accounting
    starts when the session starts and stops when it ends.
11. Enable command accounting: `aaa accounting commands {privilege level} {default |
    custom-list-name} {start-stop | stop-only | wait-start} method1 [...]`. **Also per
    privilege level.**

> The AAA server side also needs configuring: the AAA client information (hostname, IP
> address, key), the users' login credentials, and the commands each user is authorized to
> execute.

**Configuring ZBFW, 5 steps:**
1. **Create the security zones**: `zone security {zone-name}` (with an optional
   `description`). **The self zone is defined automatically** — you only create the others.
2. **Define the inspection class maps**: `class-map type inspect [match-all | match-any]
   {class-name}`, matching on `access-group name {acl}` (or other criteria). Remember
   **match-all is the default** when neither keyword is given.
3. **Define the inspection policy map**: `policy-map type inspect {policy-name}`, then
   `class type inspect {class-name}` and an action of **`drop [log]`**, **`pass [log]`**, or
   **`inspect`**. Optionally add `class class-default` / `drop` explicitly.
4. **Create the zone pair and attach the policy**: `zone-pair security {zone-pair-name}
   source {source-zone} destination {destination-zone}`, then `service-policy type inspect
   {policy-name}`. **Order matters — source first, destination second.**
5. **Apply the zones to the interfaces**: `interface {interface-id}` / `zone-member security
   {zone-name}`.

**Configuring CoPP:**
1. Build extended ACLs that classify the control-plane traffic the device legitimately
   receives — routing, management, IPsec, initialization (DHCP), ICMP.
2. Build `class-map match-all` classes, each matching one of those ACLs with
   `match access-group name {acl}`.
3. Build `policy-map POLICY-CoPP` with a `police {rate} conform-action transmit
   exceed-action transmit violate-action {transmit | drop}` under each class. **Vital classes
   get `violate-action transmit` until a baseline is established; ICMP and DHCP get `drop`.**
   Include `class class-default` with a small rate so unknown traffic is visible.
4. Apply it to the control plane: `control-plane` / `service-policy input POLICY-CoPP`.
5. Verify with `show policy-map control-plane input` and tune, using EPC (with the ACLs
   reversed from `permit` to `deny`) to identify what is landing in class-default.

## Reference Tables

**Table 26-3 — transport input command keywords**

| Keyword | Description |
|---|---|
| `all` | Allows Telnet and SSH |
| `none` | Blocks Telnet and SSH |
| `telnet` | Allows Telnet only |
| `ssh` | Allows SSH only |
| `telnet ssh` | Allows Telnet and SSH |

**Table 26-4 — RADIUS and TACACS+ comparison**

| Component | RADIUS | TACACS+ |
|---|---|---|
| Protocol and port(s) | **Cisco's implementation:** UDP **1645** (auth and authz), UDP **1646** (accounting). **Industry standard:** UDP **1812** (auth and authz), UDP **1813** (accounting) | **TCP port 49** |
| Encryption | Encrypts **only the password field**; supports **EAP** for 802.1x authentication | Encrypts the **entire payload**; **does not support EAP** |
| Authentication and authorization | **Combines** authentication and authorization; **cannot** authorize which CLI commands can be executed individually | **Separates** authentication and authorization; **can** be used for CLI command authorization |
| Accounting | Does **not** support network device CLI command accounting | **Supports** network device CLI command accounting |
| Primary use | **Secure network access** | **Network device access control** |

**Password types on IOS-XE**

| Type | Algorithm | How it is produced | Notes |
|---|---|---|---|
| 0 | None — plaintext | `password 0 {pw}` (or just `password {pw}`) | Stored and displayed as cleartext |
| 5 | MD5 | `algorithm-type md5 secret {pw}`, or `secret 5 {hash}` | Shown as `$1$...` |
| 7 | Vigenère (reversible) | `service password-encryption` applied to a type 0 password | Obfuscation only — trivially reversed |
| 8 | PBKDF2 with SHA-256 | `algorithm-type sha256 secret {pw}` | |
| 9 | scrypt | `algorithm-type scrypt secret {pw}` | Shown as `$9$...` |

> Types 4 and 6 also exist on IOS-XE (SHA-256 without salt-stretching, and reversible AES
> respectively) but were **not part of the captured source pages** — verify against the
> platform's configuration guide before relying on them.

**Privilege levels**

| Level | Name | Prompt | What it includes |
|---|---|---|---|
| **0** | — | — | `disable`, `enable`, `exit`, `help`, `logout` |
| **1** | User EXEC mode | `R1>` | No configuration changes possible — **`configure terminal` is unavailable** |
| **2–14** | Custom | varies | Configured with `privilege {mode} level {level} {command string}` |
| **15** | Privileged EXEC mode | `R1#` | All CLI commands |

**ZBFW policy actions**

| Action | Direction | Return traffic | Typical use |
|---|---|---|---|
| **`drop [log]`** | n/a — discards | n/a | The **default** action; `log` adds src/dst IP, port, protocol to syslog |
| **`pass [log]`** | **One direction only** | **Needs a second policy** | IPsec, ESP, inherently secure protocols with predictable behavior |
| **`inspect`** | Stateful | **Permitted automatically** | General TCP/UDP/ICMP flows you want statefully tracked |

**ZBFW system-built zones**

| Zone | What lands in it | Default behavior |
|---|---|---|
| **self** | **All of the router's own IP addresses** | Traffic **to and from** is permitted by default, so SSH/SNMP management and EIGRP/BGP control plane keep working. **Once a policy is applied between self and another zone, interzone traffic must be explicitly defined** |
| **default** | **Any interface not a member of another security zone**, once the zone is initialized | Traffic from a non-zone interface to a zone interface is **dropped**; enabling the default zone allows a policy map to be written between the two |

**Table 26-6 — command reference**

| Task | Command syntax |
|---|---|
| Apply an ACL to an interface | `ip access-group {access-list-number \| name} {in \| out}` |
| Apply an ACL to a vty line | `access-class {access-list-number \| access-list-name} {in \| out}` |
| Encrypt type 0 passwords in the configuration | `service password-encryption` |
| Create a username with a type 8 and type 9 password option | `username {username} algorithm-type {md5 \| sha256 \| scrypt} secret {password}` |
| Enable username and password authentication on vty lines | `login local` |
| Change command privilege levels | `privilege {mode} level {level} {command string}` |
| Allow only SSH for a vty line without using an ACL | `transport input ssh` |
| Enable SSHv2 on a router | `hostname {hostname name}` / `ip domain-name {domain-name}` / `crypto key generate rsa` |
| Disconnect terminal line users that are idle | `exec-timeout {minutes} {seconds}` |
| Enable AAA | `aaa new-model` |
| Enable AAA authorization for the console line | `aaa authorization console` |
| AAA fallback authorization method that authorizes commands if a user is successfully authenticated | `if-authenticated` |
| Enable AAA authorization for config commands | `aaa authorization config-commands` |
| Apply a ZBFW security zone to an interface | `zone-member security {zone-name}` |
| Apply an inspection policy map to a zone pair | `service-policy type inspect {policy-name}` |
| Apply a CoPP policy map to the control plane (two commands) | `control-plane` / `service-policy {input \| output} {policy-name}` |

## Config Patterns

> **Provenance:** unlike Chapter 25, this chapter is almost entirely IOS-XE configuration.
> Every block below is **transcribed from the chapter's own examples** (26-9 through 26-37),
> with example numbers noted. They are book-accurate but **not gear-validated by this skill**.

**Local username authentication on the lines (Ex. 26-9)**
```ios-xe
username type0 password 0 weak
username type5 secret 5 $1$b1Ju$kZbBS1Pyh4QzwXyZ1kSZ2/
username type9 secret 9 $9$vFpMf8elb4RVV8$seZ/bDAx1uV4yH75Z/nwUuegLJDVCc4UXOAE83JgsOc
!
line con 0
 login local
line aux 0
 login local
line vty 0 4
 login local
```

**Privilege levels / RBAC (Ex. 26-11)**
```ios-xe
username noc privilege 5 algorithm-type scrypt secret cisco123
privilege exec level 5 configure terminal
privilege configure level 5 interface
privilege interface level 5 shutdown
privilege interface level 5 no shutdown
privilege interface level 5 ip address
```
> In the running config this expands — `no shutdown` at level 5 also puts `no`, `no ip`, and
> `no ip address` at level 5, plus `privilege exec level 5 configure`. Verify with
> `show privilege` and by walking `?` at each mode.

**Restricting vty access with an ACL (Ex. 26-13)**
```ios-xe
access-list 1 deny 10.12.1.1
access-list 1 permit any
!
line vty 0 4
 access-class 1 in
```

**Per-line transport input (Ex. 26-14)**
```ios-xe
line vty 0
 login local
 transport input all
line vty 1
 login local
 transport input none
line vty 2
 login local
 transport input telnet
line vty 3
 login local
 transport input ssh
line vty 4
 login local
 transport input telnet ssh
!
line aux 0
 transport input none
 no exec
```

**SSH (Ex. 26-16)**
```ios-xe
hostname R1
username cisco secret cisco
ip domain-name cisco.com
crypto key generate rsa            ! modulus >= 768 for SSHv2; >= 2048 on 17.11.1+
ip ssh version 2                   ! forces v2 only; without it the log shows "SSH 1.99"
!
line vty 0 4
 login local
```

**Session timeouts (Ex. 26-17, 26-18)**
```ios-xe
line con 0
 exec-timeout 5 0
line vty 0 4
 exec-timeout 2 30
!
line vty 4
 exec-timeout 2 0
 absolute-timeout 10
 logout-warning 20
```

**Common AAA configuration for device access control (Ex. 26-19)**
```ios-xe
aaa new-model
!
tacacs server ISE-PRIMARY
 address 10.10.10.1
 key my.S3cR3t.k3y
!
tacacs server ISE-SECONDARY
 address 20.20.20.1
 key my.S3cR3t.k3y
!
aaa group server tacacs+ ISE-TACACS+
 server name ise-primary
 server name ise-secondary
!
aaa authentication login default group ISE-TACACS+ local
aaa authentication login CONSOLE-CUSTOM-AUTHENTICATION-LIST local line enable
aaa authentication enable default group ISE-TACACS+ enable
aaa authorization exec default group ISE-TACACS+ if-authenticated
aaa authorization exec CONSOLE-CUSTOM-EXEC-AUTHORIZATION-LIST none
aaa authorization commands 0 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST none
aaa authorization commands 1 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST none
aaa authorization commands 15 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST none
aaa authorization commands 0 default group ISE-TACACS+ if-authenticated
aaa authorization commands 1 default group ISE-TACACS+ if-authenticated
aaa authorization commands 15 default group ISE-TACACS+ if-authenticated
aaa authorization console
aaa authorization config-commands
aaa accounting exec default start-stop group ISE-TACACS+
aaa accounting commands 0 default start-stop group ISE-TACACS+
aaa accounting commands 1 default start-stop group ISE-TACACS+
aaa accounting commands 15 default start-stop group ISE-TACACS+
!
line con 0
 authorization commands 0 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST
 authorization commands 1 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST
 authorization commands 15 CONSOLE-CUSTOM-COMMAND-AUTHORIZATION-LIST
 authorization exec CONSOLE-CUSTOM-EXEC-AUTHORIZATION-LIST
 privilege level 15
 login authentication CONSOLE-CUSTOM-AUTHENTICATION-LIST
!
line vty 0 4
 ! uses default method-lists for AAA
```
> Note the console gets its own custom lists with `none`/`local line enable` — that is the
> deliberate "don't lock yourself out at the console" pattern, paired with `privilege level 15`.
> **The printed example writes the servers as `ISE-PRIMARY`/`ISE-SECONDARY` but references them
> as `ise-primary`/`ise-secondary` in the group** — names must match on real gear; treat the
> case difference as a typo in the book, not a feature.

**ZBFW — outside-to-self (Ex. 26-21 through 26-27)**
```ios-xe
zone security OUTSIDE
 description OUTSIDE Zone used for Internet Interface
!
ip access-list extended ACL-IPSEC
 permit udp any any eq non500-isakmp
 permit udp any any eq isakmp
ip access-list extended ACL-PING-AND-TRACEROUTE
 permit icmp any any echo
 permit icmp any any echo-reply
 permit icmp any any ttl-exceeded
 permit icmp any any port-unreachable
 permit udp any any range 33434 33463 ttl eq 1
ip access-list extended ACL-ESP
 permit esp any any
ip access-list extended ACL-DHCP-IN
 permit udp any eq bootps any eq bootpc
ip access-list extended ACL-GRE
 permit gre any any
!
class-map type inspect match-any CLASS-OUTSIDE-TO-SELF-INSPECT
 match access-group name ACL-IPSEC
 match access-group name ACL-PING-AND-TRACEROUTE
class-map type inspect match-any CLASS-OUTSIDE-TO-SELF-PASS
 match access-group name ACL-ESP
 match access-group name ACL-DHCP-IN
 match access-group name ACL-GRE
!
policy-map type inspect POLICY-OUTSIDE-TO-SELF
 class type inspect CLASS-OUTSIDE-TO-SELF-INSPECT
  inspect
 class type inspect CLASS-OUTSIDE-TO-SELF-PASS
  pass
 class class-default
  drop
!
zone-pair security OUTSIDE-TO-SELF source OUTSIDE destination self
 service-policy type inspect POLICY-OUTSIDE-TO-SELF
!
interface GigabitEthernet 0/2
 zone-member security OUTSIDE
```

**ZBFW — the self-to-outside policy that makes locally originated traffic work (Ex. 26-31)**
```ios-xe
ip access-list extended ACL-DHCP-OUT
 permit udp any eq bootpc any eq bootps
ip access-list extended ACL-ICMP
 permit icmp any any
!
class-map type inspect match-any CLASS-SELF-TO-OUTSIDE-INSPECT
 match access-group name ACL-IPSEC
 match access-group name ACL-ICMP
class-map type inspect match-any CLASS-SELF-TO-OUTSIDE-PASS
 match access-group name ACL-ESP
 match access-group name ACL-DHCP-OUT
!
policy-map type inspect POLICY-SELF-TO-OUTSIDE
 class type inspect CLASS-SELF-TO-OUTSIDE-INSPECT
  inspect
 class type inspect CLASS-SELF-TO-OUTSIDE-PASS
  pass
 class class-default
  drop log
!
zone-pair security SELF-TO-OUTSIDE source self destination OUTSIDE
 service-policy type inspect POLICY-SELF-TO-OUTSIDE
```
> Without this second zone pair, `R1# ping 8.8.8.8` returns **0 percent (0/5)** — the
> outside-to-self policy alone does not cover packets the router itself originates.

**CoPP (Ex. 26-33 through 26-36)**
```ios-xe
ip access-list extended ACL-CoPP-IPsec
 permit esp any any
 permit gre any any
 permit udp any eq isakmp any eq isakmp
 permit udp any any eq non500-isakmp
 permit udp any eq non500-isakmp any
ip access-list extended ACL-CoPP-Initialize
 permit udp any eq bootps any eq bootpc
ip access-list extended ACL-CoPP-Management
 permit udp any eq ntp any
 permit udp any any eq snmp
 permit tcp any any eq 22
 permit tcp any eq 22 any established
ip access-list extended ACL-CoPP-Routing
 permit tcp any eq bgp any established
 permit eigrp any host 224.0.0.10
 permit ospf any host 224.0.0.5
 permit ospf any host 224.0.0.6
 permit pim any host 224.0.0.13
 permit igmp any any
!
class-map match-all CLASS-CoPP-IPsec
 match access-group name ACL-CoPP-IPsec
class-map match-all CLASS-CoPP-Routing
 match access-group name ACL-CoPP-Routing
class-map match-all CLASS-CoPP-Initialize
 match access-group name ACL-CoPP-Initialize
class-map match-all CLASS-CoPP-Management
 match access-group name ACL-CoPP-Management
class-map match-all CLASS-CoPP-ICMP
 match access-group name ACL-CoPP-ICMP
!
policy-map POLICY-CoPP
 class CLASS-CoPP-ICMP
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-IPsec
  police 64000 conform-action transmit exceed-action transmit violate-action transmit
 class CLASS-CoPP-Initialize
  police 8000 conform-action transmit exceed-action transmit violate-action drop
 class CLASS-CoPP-Management
  police 32000 conform-action transmit exceed-action transmit violate-action transmit
 class CLASS-CoPP-Routing
  police 64000 conform-action transmit exceed-action transmit violate-action transmit
 class class-default
  police 8000 conform-action transmit exceed-action transmit violate-action drop
!
control-plane
 service-policy input POLICY-CoPP
```
> **`ACL-CoPP-Routing` does not classify unicast routing protocol packets** — unicast PIM,
> unicast OSPF, and unicast EIGRP all fall through to class-default.

**Device hardening**
```ios-xe
no service config
no service pad
no mop enabled
service tcp-keepalive-in
service tcp-keepalive-out
!
interface GigabitEthernet0/2          ! public-facing interface only
 no cdp enable
 no lldp transmit
 no lldp receive
 no ip redirects
 no ip proxy-arp
 no mop enabled
```

## Design Baseline

Every row traces to the ENCOR 350-401 OCG Chapter 26 with a page number — this chapter states
its own recommendations explicitly, so no external sourcing was needed and none was invented.
**A deviation is a question for the network's operator, not automatically a finding.**

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| **Only allow internal/trusted IP addresses to reach the vty lines** via `access-class` | The vty lines are the management plane; exposing them to public source addresses is the highest-value target on the box | A jump host outside the trusted range, with compensating controls (bastion, MFA, dedicated management VRF) — but that range should still be explicit, never `permit any` | Ch. 26, p. 796 |
| **Disable the AUX port** with `no exec` under `line aux 0`, and give it `transport input none` | The aux port is a forgotten dialup-era back door; `transport input none` specifically blocks **reverse Telnet** into it | A site genuinely using out-of-band modem access — in which case it needs the same auth and ACL treatment as vty | Ch. 26, pp. 798, 802 |
| **Use SSHv2 only** — `ip ssh version 2` | SSHv1 has fundamental implementation flaws; SSHv2 closes a known security hole and is FIPS 140-1/140-2 certified. `SSH 1.99` in the log means v1 is still accepted | A legacy management tool that cannot speak v2, on a segment where that risk is explicitly accepted and logged | Ch. 26, pp. 801, 802 |
| **Generate RSA keys of at least 2048 bits** | ≥768 is the SSHv2 floor, but IOS-XE **17.11.1 and later reject keys under 2048 by default** as cryptographically weak. The enforcement can be disabled but is not recommended | None recommended — the chapter says so directly; see Field Notice 72510 | Ch. 26, p. 801 (NOTE) |
| **Do not disable the EXEC timeout in production** — avoid `exec-timeout 0 0` and `no exec-timeout` | An abandoned privileged session on an unlocked terminal is an unauthenticated shell | Lab environments only, which is exactly the exception the chapter names | Ch. 26, p. 802 (NOTE) |
| **Pair `absolute-timeout` with `logout-warning`** | `absolute-timeout` kills a session **even while it is in use**; without a warning, users lose work mid-command | None — if you use one, use both | Ch. 26, p. 802 |
| **Create a local privilege-15 user *before* enabling AAA** | `aaa new-model` with no reachable server and no local account is a lockout requiring console/ROMMON recovery | None — this is step 1 of the chapter's own procedure for a reason | Ch. 26, p. 805 (Step 1) |
| **Keep local authentication as an AAA fallback method** | Centralized policy is the point of AAA, but an unreachable server must not mean an unmanageable device | None — but the *order* of fallback methods is the tunable | Ch. 26, p. 796 |
| **Put `if-authenticated` at the end of every authorization command** | When all AAA servers go unreachable, authentication falls back to a local method but **command authorization may still be trying to reach the server**, leaving the user logged in and unable to run anything. `if-authenticated` authorizes commands for anyone who authenticated successfully | None stated. Note `if-authenticated` and `none` are **mutually exclusive** — `none` disables authorization outright | Ch. 26, p. 807 (NOTE) |
| **Leave `aaa authorization console` off unless you have deliberately built a console method list** | It is disabled by default **specifically to prevent inexperienced users from locking themselves out** | Enable it when the console must be under the same policy as vty — and pair it with custom console lists as Example 26-19 does | Ch. 26, p. 807 (Step 7) |
| **Set CoPP `violate-action transmit` on vital classes until a baseline is established**, and `drop` only on genuinely low-rate traffic (ICMP, DHCP) | Guessing a policing rate on a production control plane is how CoPP causes the outage it was meant to prevent | None for the phased approach; the *duration* before tightening is the tunable | Ch. 26, p. 819 |
| **Allow a minimal rate in CoPP `class-default` and monitor it** rather than dropping everything unknown | Unknown control-plane traffic is *discovered* there. A silent drop-all hides the thing you need to classify | None — the chapter treats class-default as an observability tool | Ch. 26, p. 819 |
| **Check whether the platform already has a default CoPP policy before writing one** | Catalyst 9000-series platforms ship with a default CoPP policy that typically does not require modification; modifying it has platform-specific restrictions and caveats | A platform without a default policy, or a documented need to modify it — consult that platform's documentation first | Ch. 26, p. 822 (NOTE) |
| **Disable unused services on public-facing interfaces**: CDP/LLDP, IP redirects, proxy ARP, service config, MOP, PAD; enable TCP keepalives | Reduces information exposed externally **and** reduces CPU/memory spent processing packets you never wanted | CDP/LLDP are often needed on *internal* links for topology and phone discovery — the chapter scopes these to the public-facing interface only | Ch. 26, pp. 822–823 |
| **Do not add `log` to the ZBFW `class-default` drop action** | The catch-all class sees every unmatched packet; logging it can fill the syslog | A short, deliberate troubleshooting window | Ch. 26, p. 812 |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show line` | Line inventory by **tty number** — CTY, AUX, and the VTYs (vty 0 → tty 578 …). **An asterisk on the left means the line is in use.** `Uses` column shows session counts |
| `show running-config \| section line vty` | The actual per-line config: `access-class`, `transport input`, `login`/`login local`, timeouts |
| `show privilege` | `Current privilege level is N` — the fastest way to confirm what a session actually got, vs what the account was supposed to get |
| `show users` | Who is logged in, on which line, from where |
| `show ip access-lists` | ACL contents **and hit counters** — including for ACLs referenced by ZBFW inspect class maps, whose counters increment even though they are not blocking |
| `show ip ssh` | SSH version in effect (**1.99 means v1 is still enabled**), authentication timeout and retries |
| `show crypto key mypubkey rsa` | Whether keys exist and their **modulus size** — the thing that blocks SSH on 17.11.1+ if under 2048 |
| `show aaa servers` | Per-server request/accept/reject/timeout counters — separates "server unreachable" from "server said no" |
| `show tacacs` | TACACS+ server state, socket opens/closes/aborts, and errors |
| `test aaa group {group} {user} {pass} new-code` | End-to-end AAA test independent of any login attempt |
| `show aaa method-lists all` | Every configured method list and its methods, in order — catches a line pointing at a list that does not exist |
| `debug tacacs` / `debug aaa authentication` / `debug aaa authorization` | Last resort; see the network-assurance skill for debug discipline |
| `show class-map type inspect [name]` | The inspect class maps, their **match-all/match-any** mode, and the ACLs they reference |
| `show policy-map type inspect [name]` | The inspect policy map and the action per class (Inspect / Pass / Drop) |
| `show policy-map type inspect zone-pair [name]` | **The operational view** — per-class, per-ACL packet and byte counters, plus session statistics (creations, estab/half-open/terminating, TCP reassembly). This is where you prove traffic is hitting the class you think it is |
| `show zone security` / `show zone-pair security` | Which zones exist, which interfaces are members, and which policy is bound to which pair |
| `show policy-map control-plane input` | **The CoPP operational view** — per-class conformed/exceeded/violated packet and byte counters with the action taken for each, plus the `cir`/`bc`/`be` actually in effect |
| `show policy-map control-plane` | Both directions if an output policy also exists |

## Intent Questions

- **Who is supposed to be able to reach this device's management plane, from where, over
  which protocol?** That answer drives `access-class`, `transport input`, and whether Telnet
  should exist at all. A reachable-but-unauthorized session is a finding; an unreachable
  authorized one is a different bug entirely.
- **Is this device's authentication supposed to be local, AAA, or AAA-with-local-fallback —
  and which is actually in effect right now?** During a TACACS+ outage the *correct* behavior
  is a local login, which looks identical to a misconfiguration if you do not know the intent.
- **What privilege is this account supposed to land at, and is the restriction enforced
  locally (`privilege level`) or centrally (TACACS+ command authorization)?** "Command
  authorization failed" means the server is deciding; a short `?` list means the box is.
- **For ZBFW: which zone is each interface supposed to be in, and does a policy exist for
  every direction the design requires?** `pass` is unidirectional and the self zone stops
  being permissive the moment you write your first policy against it — both produce outages
  that look like routing problems.
- **For CoPP: has a baseline actually been established, or is this still in the
  `violate-action transmit` observation phase?** Violations in the observation phase are data;
  the same violations after tightening are an incident.

## Troubleshooting Checklist

0. **State intent vs. observed.** Answer the Intent Questions above, then write the one-line
   symptom ("the netops account should reach level 1 with read commands, it is getting
   'Command authorization failed' on everything") — before running any show command.
1. **Can you reach the device at all, and on which line type?** If SSH fails but console
   works, the problem is in the vty path, not in authentication. `show line` to see whether a
   session is even landing.
2. **Are you out of vty lines?** `show line` — **each vty accepts exactly one user**, and
   lines are consumed from vty 0 upward. `% Connection refused by remote host` with no
   authentication prompt often means no line was available for that *protocol*, because
   `transport input` differs per line.
3. **Is `transport input` blocking the protocol on the line you landed on?** `show
   running-config | section line vty`. `transport input none` on vty 1 is invisible until
   traffic reaches that specific line.
4. **Is an `access-class` ACL denying the source?** Also `% Connection refused by remote
   host`, and also with no prompt. Check `show ip access-lists` for hits on the deny entry.
5. **SSH specifically:** does the device have a hostname, a domain name, and RSA keys?
   `show ip ssh` and `show crypto key mypubkey rsa`. **On IOS-XE 17.11.1+, a pre-existing
   sub-2048-bit key is rejected** — an upgrade can break SSH that worked the day before.
   `SSH 1.99` means v1 is still permitted, which is a finding in its own right.
6. **Authentication: local or AAA?** If `aaa new-model` is on, the line's method list governs.
   `show aaa method-lists all` — a line pointing at a custom list that was never defined falls
   through in ways that surprise people.
7. **Is the AAA server reachable and answering?** `show aaa servers` and `show tacacs`. Rising
   **timeouts** mean a path, ACL, or source-interface problem; **rejects** mean the server
   answered and said no — a policy problem, not a network one. `test aaa group ... new-code`
   isolates this from the login path entirely.
8. **Shared secret and NAD definition.** A wrong TACACS+ key looks like a timeout, not an
   error. Confirm the device is defined on the AAA server with the matching key.
9. **"Command authorization failed."** The user authenticated but the server declined the
   command. Check the per-level `aaa authorization commands {0|1|15}` lists and what the
   server's policy actually grants. **If the AAA servers are unreachable, this is the classic
   locked-in-but-useless state — the fix is `if-authenticated` at the end of the authorization
   method lists.**
10. **Privilege lower than expected?** `show privilege`, then check whether the level came
    from the AAA server, from `username ... privilege N`, or from `privilege level N` on the
    line. Also remember a custom level needs the *parent* keywords granted — `no shutdown`
    without `no` does not work, which is why IOS expands them automatically.
11. **Locked out after enabling AAA?** This is the step-1 failure: no local privilege-15 user,
    or no `enable` password as the last method. Console recovery.
12. **ZBFW: is the interface actually in a zone?** `show zone security`. An interface in **no**
    zone lands in the default zone, and **traffic from a non-zone interface to a zone
    interface is dropped**.
13. **ZBFW: does a zone pair exist for this direction?** Order is significant, and **`pass` is
    one-way** — bidirectional flows need a second zone pair. The classic symptom is the
    chapter's own: outside-to-self is configured, and `ping 8.8.8.8` from the router still
    fails because **self-to-outside was never written**.
14. **ZBFW: is traffic hitting the class you think?** `show policy-map type inspect zone-pair`
    for per-class counters, and `show ip access-lists` for per-ACE hits. Traffic landing in
    `class-default` is silently dropped — that is the implicit deny.
15. **CoPP: is a class violating?** `show policy-map control-plane input`. Violated counters
    with `actions: drop` on a routing class means CoPP is now the cause of your adjacency
    flaps. Check whether the policy was ever baselined for the routing protocols actually in
    use, and remember **unicast OSPF/EIGRP/PIM are not matched by the chapter's
    `ACL-CoPP-Routing`** and fall to class-default.
16. **Only then, debug.** `debug tacacs`, `debug aaa authorization` — scoped and directed to
    the logging buffer, not the console.

## Common Pitfalls

- **Each vty line accepts one user, and lines fill from vty 0 upward.** A per-line
  `transport input` difference turns into a capacity problem: the chapter's example refuses a
  fourth Telnet session while two vty lines sit idle, because those lines do not allow Telnet.
- **`% Connection refused by remote host` is ambiguous.** It is what you get from an
  `access-class` deny, from `transport input` blocking the protocol, and from no available
  vty line. Three different root causes, one message.
- **`service password-encryption` only converts type 0 to type 7**, and type 7 is reversible
  obfuscation, not encryption. It is not a substitute for `secret` with sha256 or scrypt.
- **`username ... password` and `username ... secret` are not interchangeable.** `password`
  defaults to type 0 (cleartext); `secret` hashes. The chapter's own Example 26-9 puts them
  side by side precisely to make this visible.
- **Username-based auth on the aux and cty lines may require AAA** on some IOS-XE releases.
  A `login local` on the console that "does nothing" is often this, not a typo.
- **Setting a multi-keyword command to a privilege level drags its parent keywords along.**
  `no shutdown` at level 5 also grants `no`, which grants more than people intend — `no ip
  address` becomes available too. Always verify the expanded list in the running config, not
  the commands you typed.
- **Privilege level 1 has no `configure terminal`.** If a user reports "the command doesn't
  exist," check `show privilege` before assuming an IOS version difference.
- **`SSH 1.99` does not mean SSHv1.99.** It means **v1 and v2 are both enabled**. Only
  `ip ssh version 2` removes v1.
- **An IOS-XE upgrade to 17.11.1 or later can break working SSH** by rejecting an existing
  RSA key under 2048 bits. Regenerate before the upgrade window, not during it.
- **`exec-timeout 0 0` is a lab convenience that ships to production constantly.** It disables
  the timeout entirely.
- **`absolute-timeout` cuts the session even while it is actively in use** — without
  `logout-warning`, that is a user losing an in-progress change.
- **TACACS+ is TCP 49; RADIUS is UDP.** And RADIUS has *two* port pairs: Cisco's legacy
  1645/1646 and the industry standard 1812/1813. A mismatch here looks exactly like an
  unreachable server.
- **RADIUS encrypts only the password field; TACACS+ encrypts the entire payload.** "RADIUS is
  encrypted" is a half-truth that matters when someone is capturing management traffic.
- **RADIUS cannot do per-command authorization.** Not "does it badly" — it must return all
  authorization parameters in a single reply, and thousands of CLI command combinations in one
  response risks **memory exhaustion on the device**. That is the actual engineering reason
  TACACS+ owns device administration.
- **TACACS+ does not support EAP**, which is the mirror-image reason RADIUS owns 802.1x.
- **`aaa authorization console` is off by default on purpose.** Turning it on without a
  console-specific method list is a well-trodden path to a lockout that needs physical access.
- **The `if-authenticated` gap is the AAA failure mode people actually hit.** When all servers
  go unreachable, *authentication* falls back locally but *command authorization* may keep
  trying to reach the server — so you log in successfully and then cannot run a single
  command. `if-authenticated` on the end of every authorization list is the fix.
- **`if-authenticated` and `none` are mutually exclusive**, because `none` disables
  authorization entirely.
- **Enabling AAA without a local privilege-15 user is a lockout.** Step 1 of the procedure
  exists because of how often this happens.
- **`match-all` is the default for `class-map type inspect`** when you omit the keyword — a
  class map with several `match access-group` lines and no keyword requires **all** of them,
  which is almost never what was intended.
- **ZBFW `pass` is unidirectional.** A second zone pair is required for the return direction.
  `inspect` is what gives you automatic return traffic.
- **The self zone stops being permissive the moment you write a policy involving it.** Default
  behavior permits management and control plane traffic; after your first self-zone policy,
  everything must be explicit. The chapter's `ping 8.8.8.8` failing at 0/5 is this exact trap.
- **An interface in no zone is dropped toward zoned interfaces** — and most engineers assume
  no policy can fix that. Enabling the **default zone** is what makes a policy map between
  them possible.
- **ZBFW class-default drops silently by default.** Make it explicit if you want the
  troubleshooting clarity, but **do not add `log`** — it can fill the syslog.
- **ACLs referenced by inspect class maps still increment counters** even though they are not
  blocking anything. Counter hits there prove classification, not enforcement.
- **CoPP's `ACL-CoPP-Routing` in the chapter does not match unicast PIM, unicast OSPF, or
  unicast EIGRP.** Those fall into class-default, where the rate is small and the violate
  action may be `drop`.
- **A CoPP policy must be tuned to the routing protocols actually in use.** Copying the
  chapter's policy onto a network running something it does not classify is how CoPP breaks
  adjacencies.
- **Check for a platform default CoPP policy first.** Catalyst 9000-series devices already
  have one, and it typically should not be modified.
- **CDP/LLDP hardening is scoped to public-facing interfaces.** Blanket-disabling CDP
  internally breaks phone discovery and topology tooling — the chapter is explicit that
  interface-specific hardening applies only to the interface connected to the public network.

## Exam Preparation Tasks

### Key topics coverage map

Table 26-5, mapped to where each element lives in this skill. The mapping is not one-to-one:
several key topics collapse into one Reference Table here, and the AAA and ZBFW topics are
each split across Key Concepts, Procedure, Config Patterns, and Common Pitfalls.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Section | Access Control Lists (ACLs) | 781 | Key Concepts → "Access Control Lists (ACLs)" (stateless, no payload inspection, the four match fields). **Partial** — captured at concept level; pages 777–791 were not transcribed example by example |
| List | ACL categories | 781 | Key Concepts → "Access Control Lists (ACLs)". **Gap** — the chapter's explicit category list (standard vs extended, numbered vs named) is referenced but not reproduced verbatim here |
| Paragraph | Applying ACL to an interface | 782 | Key Concepts → "Access Control Lists (ACLs)" (`ip access-group … {in \| out}`); Reference Tables → Table 26-6 |
| List | CLI access methods | 788 | Key Concepts → "CLI access methods and line password protection" (console/cty, aux, vty). **Partial** — concept level |
| List | Line password protection options | 788 | Key Concepts → "CLI access methods and line password protection" (line password, `login local`, AAA). **Partial** — concept level |
| Section | Password Types | 789 | Reference Tables → "Password types on IOS-XE"; Key Concepts → "Password types"; Config Patterns → Example 26-9. **Partial** — types 0/5/7/8/9 are confirmed from the transcribed pages and Table 26-6; **types 4 and 6 are flagged as unverified** |
| List | Local username configuration options | 790 | Key Concepts → "Password types" (`username … algorithm-type … secret`); Config Patterns → Examples 26-9 and 26-11; Reference Tables → Table 26-6. **Partial** — concept level |
| List | Privilege levels | 793 | Key Concepts → "Privilege levels and role-based access control"; Reference Tables → "Privilege levels"; Config Patterns → Example 26-11; two Common Pitfalls bullets |
| List | SSH versions | 801 | Key Concepts → "SSH" (v1 flaws, v2 rework, FIPS 140-1/140-2, SSH 1.99); Procedure → "Configuring SSH"; Design Baseline rows 3–4; two Common Pitfalls bullets |
| List | Authentication, authorization, and accounting (AAA) | 803 | Key Concepts → "AAA — the framework" (the three functions) |
| List | AAA primary use cases | 803 | Key Concepts → "AAA — the framework" (device access control → TACACS+; secure network access → RADIUS, cross-referenced to the `secure-network-access-control` skill) |
| Paragraph | TACACS+ key differentiator | 804 | Key Concepts → "TACACS+" (separates AAA into independent functions); Reference Tables → Table 26-4; Common Pitfalls |
| Paragraph | RADIUS key differentiators | 804 | Key Concepts → "RADIUS" (EAP transport; all authz params in a single reply; the memory-exhaustion argument); Reference Tables → Table 26-4; two Common Pitfalls bullets |
| Paragraph | Zone-Based Firewall (ZBFW) | 810 | Key Concepts → "Zone-Based Firewall (ZBFW)"; Procedure → "Configuring ZBFW, 5 steps"; Config Patterns → Examples 26-21 to 26-27 and 26-31 |
| Paragraph | ZBFW default zones | 810 | Key Concepts → "Zone-Based Firewall" → self zone and default zone bullets; Reference Tables → "ZBFW system-built zones"; Troubleshooting steps 12–13; two Common Pitfalls bullets |
| Section | Control Plane Policing (CoPP) | 817 | Key Concepts → "Control Plane Policing (CoPP)"; Procedure → "Configuring CoPP"; Config Patterns → Examples 26-33 to 26-36; Design Baseline rows 11–13; three Common Pitfalls bullets |

**Coverage note.** Thirteen of the sixteen rows are captured in full from transcribed pages
(792–825). The six rows marked **Partial** and the one marked **Gap** all come from **pages
777–791** — the quiz, the ACL section, and the CLI-access/password-types/username-options
material. Those pages were captured at concept-and-command level rather than transcribed
example by example, so the chapter's explicit ACL category list and its worked ACL and
password examples are **not** reproduced here. Everything from p. 792 onward — privilege
levels, vty control, SSH, timeouts, the full AAA procedure and Example 26-19, all of ZBFW,
all of CoPP, and device hardening — is captured in full, with the config blocks transcribed
from the book's own examples. **Re-run `/ccnp-note network-device-access-control` against
pages 777–791 to close those rows.**

### "Do I Know This Already?" question analysis

**Not captured.** The chapter's opening quiz and its answer key were not part of the
transcribed source pages, so there is nothing here to analyze. Per the template's own rule,
inventing traps that are not there is worse than leaving the subsection empty — fill this in
when pages 777–780 are captured.
