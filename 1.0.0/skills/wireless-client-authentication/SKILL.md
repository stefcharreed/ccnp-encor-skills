---
name: ccnp-wireless-client-authentication
description: >
  Use this skill when troubleshooting or configuring wireless client authentication on IOS-XE.
  Invoke when the user asks about: wireless client authentication, Open Authentication, 802.11
  authentication request, WEP, pre-shared key, PSK, personal mode, enterprise mode, Wi-Fi Protected
  Access, WPA, WPA1, WPA2, WPA3, SAE, Simultaneous Authentication of Equals, forward secrecy,
  dictionary attack on the four-way handshake, 802.1x, EAP, Extensible Authentication Protocol,
  EAPOL, EAP over LAN, four-way EAPOL handshake, ANonce, SNonce, MIC, PMK, GMK, PTK, GTK,
  pairwise master key, groupwise master key, pairwise transient key, groupwise transient key,
  supplicant, authenticator, authentication server, RADIUS server, ISE, PEAP, EAP-TLS, EAP-FAST,
  LEAP, EAP-TTLS, EAP-SIM, AAA method list, dot1x method list, AES CCMP128, Auth Key Mgmt,
  WebAuth, Web Authentication, LWA, Local Web Authentication, CWA, Central Web Authentication,
  acceptable use policy, AUP, captive portal, WebAuth parameter map, guest WLAN, WLAN Layer 2
  security mode, WLAN Layer 3 security, C9800 WLAN security tab, Security column [open]
  [WPA2][PSK][AES] [WPA2][802.1x][AES] [Web_Auth].
---

## Purpose
Wireless client authentication controls which devices and users are allowed to move from "associated to a BSS" to "actually passing traffic on the WLAN," and it is configured per-WLAN on the wireless LAN controller — so getting it wrong either locks out legitimate users or silently hands your corporate SSID to anyone in range.

## Key Concepts

**Why authenticate at all (p.592–593)**
- Clients must first discover a BSS and request association; they should be authenticated *before* becoming functioning members of the WLAN.
- Trusted devices get corporate resources; guests get a separate guest WLAN with nonconfidential/public resources; rogue clients should not associate at all.
- Some methods use only a static text string common to all trusted clients and APs — stored on the device, so a stolen/lost device still authenticates. Stronger methods require a corporate user database and a username/password the thief would not know.
- Wireless security is configured **per WLAN**. Every configuration task in this chapter lives under **WLAN > Edit > Security** tab.

**The four methods covered (p.589, 593)**
- Open Authentication, Pre-Shared Key (PSK), EAP/802.1x, and WebAuth.
- For each, you start by creating a WLAN on the WLC, assigning a controller interface, and enabling the WLAN.

**Open Authentication (p.593)**
- The original 802.11 standard offered only two choices: Open Authentication and WEP.
- Open Authentication offers open access; the only requirement is that the client send an 802.11 authentication request before it attempts to associate. **No other credentials are needed.**
- Its actual purpose: validate that the client is a valid 802.11 device — it authenticates the wireless hardware and protocol, **not** the user's identity. User identity is handled as a true security process by other means.
- Typical of public locations; if any client screening is used, it comes as WebAuth layered on top.
- GUI Security column shows `[open]` (p.594).

**WPA versions and the two modes (p.595)**
- WPA (a.k.a. WPA1), WPA2, and WPA3 are certified by the **Wi-Fi Alliance** so clients and APs using the same version are known to be compatible. The WPA versions also specify encryption and data integrity methods.
- **All three WPA versions support two client authentication modes**: pre-shared key (PSK) = **personal mode**, and **802.1x** = **enterprise mode**. Which one you pick depends on the scale of the deployment.
- Personal mode: a key string must be shared/configured on **every client and AP**. The pre-shared key is kept confidential and **is never sent over the air**. Instead, clients and APs run a four-way handshake that uses the PSK string to construct and exchange encryption key material that *can* be openly exchanged. On success the AP authenticates the client and the two can secure data frames.
- **WPA-Personal and WPA2-Personal weakness**: a malicious user can eavesdrop and capture the four-way handshake between a client and an AP, then run a **dictionary attack** to guess the pre-shared key. Success lets them decrypt wireless data or join the network posing as a legitimate user.
- **WPA3-Personal** strengthens the key exchange with **Simultaneous Authentication of Equals (SAE)** — rather than a client authenticating against a server or AP, the client and AP initiate the authentication process **equally and even simultaneously**.
- WPA3-Personal also offers **forward secrecy**: even if a password or key is compromised, an attacker cannot use it to decrypt data already transmitted over the air.
- GUI Security column shows `[WPA2][PSK][AES]` (p.597).

**EAP and 802.1x (p.597–599)**
- Client authentication generally involves a challenge, a response, and a decision to grant access — plus, behind the scenes, an exchange of session/encryption keys.
- **EAP** is a flexible, scalable authentication *framework*, not a single method. It defines a set of common functions that actual authentication methods use to authenticate users.
- EAP integrates with the **IEEE 802.1x** port-based access control standard. When 802.1x is enabled it limits access to the network medium until the client authenticates: a wireless client can **associate** with an AP but cannot pass data to the rest of the network until it successfully authenticates.
- With Open Auth and PSK, clients are authenticated **locally at the AP**. With 802.1x, the client uses **Open Authentication to associate**, and then the actual client authentication occurs at a dedicated authentication server.
- The three-party 802.1x arrangement (Key Topic, p.598):
  - **Supplicant** — the client device that is requesting access.
  - **Authenticator** — the network device that provides access to the network (usually a wireless LAN controller [WLC]).
  - **Authentication server (AS)** — the device that takes user or client credentials and permits or denies network access based on a user database and policies (usually a **RADIUS server**).
- The controller is a **middleman**: it controls user access with 802.1x and talks to the AS using the EAP framework. For wired and wireless networks this process uses **EAP over LAN (EAPOL)**.
- **Key hierarchy (p.598)**: after successful authentication, client and AP build encryption keys hierarchically, beginning with a **Pairwise Master Key (PMK)** and a **Groupwise Master Key (GMK)** that are generated and distributed during EAP authentication. **Pairwise keys protect unicast** traffic across the air; **Groupwise keys protect broadcast and multicast** traffic.
- NOTE (p.598): with PSK there is no EAP at all — the PSK is already known to both the client and the AP, so **the PMK is derived from it**.
- The PMK is used to derive a **Pairwise Transient Key (PTK)** that secures unicast traffic. The PTK is also used **with the GMK** to derive a **Groupwise Transient Key (GTK)** that secures broadcast and multicast traffic.
- Enterprise mode supports many EAP methods — **LEAP, EAP-FAST, PEAP, EAP-TLS, EAP-TTLS, EAP-SIM** — but **you do not configure any specific method on a WLC**. Specific EAP methods are configured on the authentication server and supported on the client devices; the WLC is only the EAP middleman.
- Cisco WLCs can use either **external RADIUS servers** on the wired network **or a local EAP server on the WLC**.
- GUI Security column shows `[WPA2][802.1x][AES]`; the chapter text says the column "should display [802.1X]" for EAP-based auth (p.602).

**WebAuth (p.603)**
- None of the other three methods involve direct interaction with the end user: Open Auth requires nothing from user or device; PSK is exchanged between device and AP; EAP can prompt for credentials **only if the EAP method supports it**, and even then the user sees nothing about the network or its provider.
- **Web Authentication (WebAuth)** presents the end user with content to read and interact with before granting access — e.g., an **acceptable use policy (AUP)** the user must accept. It can also prompt for user credentials, display enterprise information, and so on. The user must open a web browser to see the WebAuth content.
- **WebAuth can be used as an additional layer in concert with Open Authentication, PSK-based authentication, and EAP-based authentication.**
- **Local Web Authentication (LWA)** handles WebAuth locally on the WLC for smaller environments. The five LWA modes (Key Topic, p.603):
  - LWA with an internal database on the WLC
  - LWA with an external database on a RADIUS or LDAP server
  - LWA with an external redirect after authentication
  - LWA with an external splash page redirect, using an internal database on the WLC
  - LWA with passthrough, requiring user acknowledgment
- When many controllers provide Web Authentication, use LWA with an external database on a RADIUS server such as **ISE** to keep the user database centralized. Moving the Web Authentication *page* onto the central server too is **Central Web Authentication (CWA)**.
- GUI Security column shows `[Web_Auth]` per the text on p.605; Figure 20-17 shows the combined value `[WPA2][802.1x][AES],[Web Auth]` for the Guest_webauth WLAN (p.606).

**GUI navigation paths recorded verbatim from the chapter**
- Create a WLAN: **Configuration > Wireless Setup > WLAN Wizard**, or **Configuration > Tags & Profiles > WLANs > Add** (p.594).
- WLAN security: **Add/Edit WLAN > Security > Layer2 / Layer3 / AAA** tabs (p.594, 602, 605).
- RADIUS servers: **Configuration > Security > AAA > Servers/Groups > Add** (p.600).
- Method lists: **Configuration > Security > AAA > AAA Method List** (p.600, 604).
- WebAuth parameter maps: **Configuration > Security > Web Auth > Add** (p.603).
- Verification list: **Configuration > Tags & Profiles > WLANs** (p.594, 602, 605). *(On p.597 and p.600 the book writes this same path as "Configuration > Tags & Policies > WLANs" — see Ambiguities.)*

**Field values the chapter prints (worth memorizing)**
- Layer 2 Security Mode drop-down choices: **None, WPA + WPA2, WPA2 + WPA3, WPA3** (Figs 20-2, 20-4).
- WPA2 Encryption choices: **AES(CCMP128), CCMP256, GCMP128, GCMP256**. AES(CCMP128) is "the most robust encryption" per p.600.
- Auth Key Mgmt choices: **802.1x, PSK, CCKM, FT + 802.1x, FT + PSK, 802.1x-SHA256, PSK-SHA256**.
- Other Layer 2 fields: MAC Filtering, OWE Transition Mode, Transition Mode WLAN ID (1–16), Protected Management Frame / **PMF (Disabled)**, GTK Randomize, OSEN Policy, PSK Format (**ASCII**), PSK Type (**Unencrypted**), Fast Transition (**Disabled** / **Adaptive Enabled**), Over the DS, **Reassociation Timeout 20**, MPSK Configuration / MPSK.
- RADIUS server fields (Fig 20-8): Name `radius1`, Server Address `192.168.10.9`, PAC Key, Key Type `Clear Text`, Key / Confirm Key, **Auth Port 1812**, **Acct Port 1813**, **Server Timeout (seconds) 1–1000**, **Retry Count 0–100**, Support for CoA `ENABLED`, CoA Server Key Type, CoA Server Key / Confirm, Automate Tester.
- AAA method list fields (Figs 20-9, 20-15): Method List Name, **Type** (`dot1x` for 802.1x, `login` for WebAuth), Group Type `group`, Fallback to local, Available Server Groups (`ldap`, `tacacs+`) → Assigned Server Groups (`radius`).
- WebAuth parameter map fields (Figs 20-13, 20-14): Parameter-map Name, **Maximum HTTP connections 1–200** (example `100`), **Init-State Timeout(secs) 60–3932100** (example `120`), **Type: webauth / authbypass / consent / webconsent**, Banner Title, Banner Type (None / Banner Text / File Name), Banner Text, Captive Bypass Portal, Disable Success Window, Disable Logout Window, Disable Cisco Logo, Sleeping Client Status, **Sleeping Client Timeout (minutes) 720**. The **Advanced** tab defines WebAuth redirects and customized login success/failure pages.
- Layer3 tab fields (Fig 20-16): Web Policy, Web Auth Parameter Map, Authentication List, and the on-screen note: *"For Local Login Method List to work, please make sure the configuration 'aaa authorization network default local' exists on the device."*
- The controller in Figure 20-17 is a **Cisco Embedded Wireless Controller on Catalyst Access Points, version 17.6.4**.

## Procedure

### A. Configure Open Authentication on a WLAN (p.594)
1. Navigate to **Configuration > Wireless Setup > WLAN Wizard**, or **Configuration > Tags & Profiles > WLANs** and select **Add**.
2. Under the **General** tab, enter the SSID string. (Chapter example: WLAN named `guest`, SSID `Guest`.)
3. Select the **Security** tab, then **Layer 2**.
4. In the **Layer 2 Security Mode** drop-down, select **None** for Open Authentication.
5. Click **Apply to Device**.
6. Configure a **Policy profile** to identify the VLAN number the WLAN maps to.
7. Apply the WLAN and Policy profiles to the APs.
8. Verify under **Configuration > Tags & Profiles > WLANs**: Security shows `[open]`, and the WLAN status is enabled and active.

### B. Configure WPA2/WPA3 Personal (PSK) on a WLAN (p.595–597)
1. Navigate to **Configure > Tags & Profiles > WLANs**, select **Add** (or select an existing WLAN to edit).
2. Set the parameters on the **General** tab appropriately.
3. Select **Security > Layer 2**. In **Layer 2 Security Mode**, select the appropriate WPA version. (Chapter example: `WPA + WPA2` on the WLAN named `devices`.)
4. Under **WPA+WPA2 Parameters**, narrow the version: **uncheck WPA Policy**, **check WPA2 Policy**, and check **WPA2 Encryption AES**.
5. Under **Auth Key Mgmt**, check **only** the box next to **PSK**.
6. Enter the pre-shared key string in the **Pre-Shared Key** box. (PSK Format `ASCII` in the example.)
7. Click **Apply to Device**.
8. Verify under the WLAN list: Security column shows `[WPA2][PSK][AES]`, status enabled and active.

### C. Configure EAP-based authentication with an external RADIUS server (p.600–602)
1. **Configuration > Security > AAA**, select the **Servers/Groups** tab, click **Add** (or edit an existing server definition).
2. Enter the server's **Name** and **IP address**, plus the **shared secret key** the controller uses to talk to the server. Confirm the RADIUS **port numbers** are correct (defaults shown: Auth 1812, Acct 1813); enter different ports if needed. Apply.
3. **Configuration > Security > AAA**, select the **AAA Method List** tab. Select the "default" list or click **Add** to define a new one.
4. In the **Type** drop-down select **dot1x** to use external RADIUS servers.
5. Define the order of server groups: under **Available Server Groups** select **radius**, then the **>** button to move it into **Assigned Server Groups**. (Chapter example: method list `myRadius`.) Click **Apply to Device**.
6. **Configuration > Tags & Policies > WLANs**, click **Add** to add a new WLAN. (Chapter example: `staff_eap`.)
7. Under the **Layer 2** tab select **WPA+WPA2**; make sure **WPA2 Policy is checked and WPA Policy is not**.
8. Beside **WPA2 Encryption**, check **AES (CCMP128)** for the most robust encryption.
9. Under **Auth Key Mgmt**, select **802.1x** to enable enterprise mode. Make sure **PSK is NOT checked** so personal mode stays disabled.
10. Select the **Security > AAA** tab and choose the desired authentication group list in the **Authentication List** drop-down (example: `myRadius`).
11. Click **Apply to Device**.
12. Verify under **Configuration > Tags & Profiles > WLANs**: Security column displays `[802.1X]` (shown as `[WPA2][802.1x][AES]`), status enabled and active.

### D. Configure WebAuth on a WLAN (p.603–605)
1. **Configuration > Security > Web Auth**, select **Add** to create a **parameter map** holding the global and custom parameters WebAuth will use. (Chapter example: `MyWebAuth`, Type `webauth`.) Click **Apply to Device**.
2. Select the newly created parameter map from the list and edit its parameters — e.g., configure a **WebAuth banner** (Banner Title, Banner Type, Banner Text), Maximum HTTP connections, Init-State Timeout.
3. Optionally select the **Advanced** tab to define WebAuth redirects and customized login success and failure pages.
4. **Configuration > Security > AAA > AAA Method List**: define the authentication method list WebAuth will use. The method list **Type must be set to "Login"** to interact with the end user attempting to authenticate. (Chapter example: list named `webauth`, Type `login`, only `radius` in Assigned Server Groups.)
5. **Configuration > Tags & Profiles > WLANs**: name the WLAN and SSID in the **General** tab.
6. Select the **Security** tab and define the **Layer 2** parameters.
7. Select the **Layer3** tab to configure the WebAuth operation: check **Web Policy**, then select the appropriate **Web Auth Parameter Map** and **Authentication List**.
8. Click **Apply to Device**.
9. Complete the WLAN creation process by linking the WLAN profile to a **policy profile via a policy tag**, then apply the tag to some APs.
10. Verify under **Configuration > Tags & Profiles > WLANs**: the WLAN shows `[Web_Auth]` in the Security column (Fig 20-17 shows `[WPA2][802.1x][AES],[Web Auth]`).

### E. Four-way EAPOL handshake message exchange (Figure 20-7, Key Topic, p.599)
Preconditions: `802.1x Access Blocked`; EAP authentication completes between supplicant and AS. Both supplicant and authenticator then hold **PMK Is Known / GMK Is Known**. The exchange **begins with the AP and ends with the client**.

1. **Message 1: EAPOL-Key (ANonce)** — AP → client.
2. Client: **Derive PTK**.
3. **Message 2: EAPOL-Key (SNonce, MIC)** — client → AP.
4. AP: **Derive PTK**, **Generate GTK**.
5. **Message 3: EAPOL-Key (GTK, MIC)** — AP → client.
6. Client: **Install PTK**, **Install GTK**.
7. **Message 4: EAPOL-Key (MIC)** — client → AP.
8. Result: `802.1x Access Unblocked`.

Summary from the text: the **first two messages** involve the AP and client exchanging enough information to **derive a PTK**; the **last two messages** involve **exchanging the GTK** that the AP generates and the client acknowledges. If all four messages are successful, the client's association can be protected and the 802.1x process unblocks wireless access for the client to use.

## Reference Tables

### The four client authentication methods (Ch. 20, p.593–605)
| Method | What is presented | Where the decision is made | Interacts with the end user? | GUI Security column |
|---|---|---|---|---|
| Open Authentication | 802.11 authentication request only; no credentials | At the AP | No | `[open]` |
| Pre-Shared Key (PSK) | Static key string, never sent over the air | Locally at the AP | No | `[WPA2][PSK][AES]` |
| EAP / 802.1x | User or client credentials via EAP over LAN | Dedicated authentication server (usually RADIUS) | Only if the EAP method supports it | `[WPA2][802.1x][AES]` (text: `[802.1X]`) |
| WebAuth | Web page: AUP, credentials, enterprise info | WLC (LWA) or central server (CWA) | Yes — user must open a browser | `[Web_Auth]` |

### WPA versions: personal vs. enterprise (Ch. 20, p.595, 599)
| Version | Personal mode | Enterprise mode | Notes from the chapter |
|---|---|---|---|
| WPA (WPA1) | PSK | 802.1x | Certified by the Wi-Fi Alliance; four-way handshake capture + dictionary attack is possible |
| WPA2 | PSK | 802.1x | Same handshake-capture weakness as WPA-Personal; use with AES/CCMP |
| WPA3 | PSK using **SAE** | 802.1x | SAE lets client and AP authenticate equally/simultaneously; adds **forward secrecy** |

### 802.1x roles (Key Topic, Ch. 20, p.598)
| Role | Definition | Typically |
|---|---|---|
| Supplicant | The client device that is requesting access | Wireless client |
| Authenticator | The network device that provides access to the network | Wireless LAN controller (WLC) |
| Authentication server (AS) | Takes user or client credentials and permits or denies access based on a user database and policies | RADIUS server |

### Key hierarchy (Ch. 20, p.598–599)
| Key | Full name | Derived from | Protects |
|---|---|---|---|
| PMK | Pairwise Master Key | Generated/distributed during EAP auth; **derived from the PSK** when PSK is used | Root of the pairwise hierarchy |
| GMK | Groupwise Master Key | Generated/distributed during EAP auth | Root of the groupwise hierarchy |
| PTK | Pairwise Transient Key | PMK | Unicast traffic |
| GTK | Groupwise Transient Key | PTK **with** the GMK | Broadcast and multicast traffic |

### LWA modes (Key Topic, Ch. 20, p.603)
| Mode | User database / page location |
|---|---|
| LWA with an internal database on the WLC | Both on the WLC |
| LWA with an external database on a RADIUS or LDAP server | Database external, page on the WLC |
| LWA with an external redirect after authentication | Redirect to an external destination post-auth |
| LWA with an external splash page redirect, using an internal database on the WLC | Splash page external, database on the WLC |
| LWA with passthrough, requiring user acknowledgment | No credentials — acknowledgment only |
| (CWA — Central Web Authentication) | Both database **and** the WebAuth page on the central server |

### Key topics for the chapter (Table 20-2, Ch. 20, p.606)
| Key topic element | Description | Page |
|---|---|---|
| Paragraph | WPA personal mode for PSK | 595 |
| List | 802.1x roles | 598 |
| Figure 20-7 | Message Exchange During the Four-Way EAPOL Handshake | 599 |
| List | WebAuth modes | 603 |

## Config Patterns

```ios-xe
! ---- RADIUS server + dot1x method list (mirrors Figs 20-8 / 20-9) ----
radius server radius1
 address ipv4 192.168.10.9 auth-port 1812 acct-port 1813
 key <from-secrets-not-repo>
!
aaa group server radius radius
 server name radius1
!
aaa new-model
aaa authentication dot1x myRadius group radius
!
! ---- Open Authentication WLAN (Fig 20-2: Layer 2 Security Mode = None) ----
wlan guest 1 Guest
 no security wpa
 no security wpa akm dot1x
 no security wpa wpa2
 no shutdown
!
! ---- WPA2 Personal / PSK WLAN (Fig 20-4) ----
wlan devices 2 devices
 security wpa wpa2
 no security wpa wpa1
 security wpa wpa2 ciphers aes
 security wpa akm psk
 security wpa psk set-key ascii 0 <from-secrets-not-repo>
 no shutdown
!
! ---- WPA2 Enterprise / 802.1x WLAN (Fig 20-10 / 20-11) ----
wlan staff_eap 3 staff_eap
 security wpa wpa2
 no security wpa wpa1
 security wpa wpa2 ciphers aes
 security dot1x authentication-list myRadius
 no shutdown
!
! ---- WebAuth (Figs 20-13 / 20-15 / 20-16) ----
parameter-map type webauth MyWebAuth
 type webauth
 timeout init-state sec 120
 max-http-conns 100
 banner title Test
!
aaa authentication login webauth group radius
!
wlan Guest_webauth 4 Guest_webauth
 security web-auth
 security web-auth authentication-list webauth
 security web-auth parameter-map MyWebAuth
 no shutdown
```

**Honesty note on the block above.** Chapter 20 is **entirely GUI-driven** — it prints *no CLI at all*, only C9800 / EWC web UI navigation paths and screenshots (Figures 20-2 through 20-17). The faithful record of what the chapter actually teaches is the GUI navigation in the **Procedure** and **Key Concepts** sections above. The CLI block is **reconstructed from the chapter's GUI fields, not printed in the book, and NOT gear-validated** — treat it as a starting point to diff against a real `show run | section wlan`, not as copy-paste truth. Exact keyword forms vary by IOS-XE release. **No real pre-shared key, RADIUS shared secret, or password appears here**: `<from-secrets-not-repo>` is a placeholder and must stay one.

## Design Baseline
"No source, no row." Only one source was available for this file — ENCOR Chapter 20 as read. A deviation from any row below is a **question for the operator**, not automatically a finding.

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Use the highest WPA version available on your WLCs, APs, and client devices | Maximizes security; WPA3-Personal adds SAE and forward secrecy that WPA/WPA2-Personal lack | Client or AP hardware/software does not support the higher version | ENCOR 350-401 OCG, Ch. 20, p. 595 |
| Keep the pre-shared key a well-kept secret; never divulge it to an unauthorized person | The key string authenticates every device on the WLAN | None stated | ENCOR 350-401 OCG, Ch. 20, p. 595 |
| Expect to touch **every** device to change a PSK, unless PSK with ISE is used | Every device using the WLAN must be configured with an identical pre-shared key | PSK with Identity Services Engine (ISE) removes the identical-key requirement | ENCOR 350-401 OCG, Ch. 20, p. 595 |
| Enable both the WPA and WPA2 check boxes **only** if legacy clients requiring WPA are mixed in with newer WPA2 clients | The WLAN will only be as secure as the weakest security suite configured on it | Documented legacy client population that cannot be upgraded | ENCOR 350-401 OCG, Ch. 20, p. 597 |
| Ideally use WPA2 or WPA3 with AES/CCMP and avoid any other hybrid mode | Hybrid modes such as WPA with AES and WPA2 with TKIP cause compatibility issues and have been deprecated | Same legacy-client exception, accepted knowingly | ENCOR 350-401 OCG, Ch. 20, p. 597 |
| Select AES (CCMP128) as the WPA2 encryption cipher | Called out as "the most robust encryption" in the enterprise walkthrough | Client base requires a different supported cipher | ENCOR 350-401 OCG, Ch. 20, p. 600 |
| For enterprise mode, enable 802.1x under Auth Key Mgmt and make sure PSK is **not** checked | Leaving PSK checked leaves personal mode enabled alongside enterprise mode | None stated | ENCOR 350-401 OCG, Ch. 20, p. 600 |
| Do not try to pick PEAP/EAP-TLS on the WLC; configure the EAP method on the RADIUS server and match it on the client supplicant | The controller only has to know that 802.1x will be in use; it is the EAP middleman | None stated | ENCOR 350-401 OCG, Ch. 20, p. 602 |
| With many controllers offering WebAuth, use LWA with an external database on a RADIUS server such as ISE | Keeps the user database centralized instead of duplicated per WLC | A single small controller with a local user population | ENCOR 350-401 OCG, Ch. 20, p. 603 |
| Set the WebAuth AAA method list Type to "Login" | The method list must interact with the end user attempting to authenticate | None stated | ENCOR 350-401 OCG, Ch. 20, p. 604 |
| Put guests on a separate guest WLAN with nonconfidential/public resources | Corporate WLANs carry confidential resources and should admit only trusted, expected devices | None stated | ENCOR 350-401 OCG, Ch. 20, p. 592 |
| After creating the WLAN, configure a policy profile for the VLAN and bind it via a policy tag applied to APs | Without the policy profile/tag the WLAN is not actually delivered by any AP | None stated | ENCOR 350-401 OCG, Ch. 20, p. 594, 605 |

## Verification Commands

**GUI verification (what the chapter actually shows).** Navigate to **Configuration > Tags & Profiles > WLANs** and read the **Status** and **Security** columns:

| GUI value in the Security column | What it means |
|---|---|
| `[open]` | Open Authentication — Layer 2 Security Mode = None (p.594) |
| `[WPA2][PSK][AES]` | WPA2 personal mode, PSK auth key mgmt, AES/CCMP128 (p.597) |
| `[WPA2][802.1x][AES]` | WPA2 enterprise mode, 802.1x auth key mgmt, AES/CCMP128 (p.602) |
| `[Web_Auth]` / `[WPA2][802.1x][AES],[Web Auth]` | WebAuth enabled as a Layer 3 policy, alone or layered on a Layer 2 suite (p.605–606) |
| Status column green/enabled | Chapter repeatedly says to confirm the WLAN status is **enabled and active** |

**CLI verification.** *These `show` commands are standard C9800 IOS-XE commands, not printed anywhere in Chapter 20 — they are supplied for lab use and are NOT gear-validated here.*

| Command | What to look for |
|---------|-----------------|
| `show wlan summary` | The WLAN list equivalent of the GUI table: ID, profile name, SSID, status, and security suite per WLAN |
| `show wlan name staff_eap` | Per-WLAN detail: WPA2 enabled, AKM (PSK vs 802.1x), cipher AES/CCMP, auth method list name, web-auth state |
| `show run \| section wlan` | The committed WLAN security config to diff against intent (AKM, ciphers, PSK presence, auth list) |
| `show wireless client summary` | Which clients are associated, on which WLAN/AP, and their state — associated vs run/authenticated |
| `show wireless client mac-address <H.H.H> detail` | Per-client policy manager state, auth method, key management, VLAN — where in the join/auth sequence a client is stuck |
| `show aaa servers` | RADIUS server IP, auth port 1812 / acct port 1813, and request/timeout/dead counters — proves the WLC can reach the AS |
| `show run aaa` | Method lists: `dot1x` list for enterprise mode, `login` list for WebAuth, and the group each points at |
| `show parameter-map type webauth all` | WebAuth parameter map type (webauth/consent/authbypass/webconsent), banner, timeouts |
| `debug wireless mac <H.H.H>` | Live per-client trace of association, EAP/EAPOL exchange, and key installation for one MAC |

## Intent Questions
1. **Who is this WLAN for** — trusted corporate devices, corporate users, or guests? That answer decides between PSK (device trust), 802.1x/EAP (user trust via a directory), and WebAuth (guest acknowledgment/AUP).
2. **Is there a user database in play, and where does it live?** External RADIUS/ISE, an LDAP server, a local EAP server on the WLC, or an internal WebAuth database — this decides LWA vs CWA and whether a `dot1x` method list is even required.
3. **What WPA version do the WLCs, APs, and the actual client population all support?** The baseline is "highest available" — the deviation is a client-support fact you must confirm, not assume.
4. **Is WebAuth meant to be the whole security posture, or a layer on top?** WebAuth can ride on Open, PSK, or EAP — knowing which was intended tells you whether `[open]` alone on a corporate SSID is a misconfiguration or the design.

## Troubleshooting Checklist
0. **State intent vs. observed**: answer the Intent Questions above for this network, then write the one-line symptom ("should ___, isn't ___") — before running any show command. Example: "guest SSID should let visitors on after accepting the AUP, isn't presenting the WebAuth page at all."
1. **Layer 1 / RF**: is the client actually within range of an AP advertising this SSID, and is the AP up? A client that never sees the BSS never gets to authenticate — nothing in this chapter applies until the client can discover the BSS and request association.
2. **Layer 2 — association**: can the client complete the 802.11 authentication request and association request? Remember 802.1x lets a client **associate but not pass data** — "associated but no traffic" points at 802.1x/EAP, not at RF.
3. **Layer 2 — key exchange**: for PSK and enterprise alike, walk the four-way EAPOL handshake in order. Failure at Message 1/2 means the PTK cannot be derived (wrong PSK → wrong PMK, or EAP never completed). Failure at Message 3/4 means the GTK exchange or key install did not complete. Access stays blocked until all four succeed.
4. **Layer 2 — suite mismatch**: does the client support the WPA version, cipher (AES/CCMP128 vs CCMP256/GCMP), and AKM (PSK vs 802.1x) the WLAN is configured for? Hybrid modes (WPA with AES, WPA2 with TKIP) are called out as a compatibility problem and are deprecated.
5. **Layer 3 / reachability to the AS**: can the WLC reach the RADIUS server IP on the configured auth port (1812) and acct port (1813)? Is the shared secret identical on both ends? Is the server timeout / retry count leaving clients hanging?
6. **Layer 3 / WebAuth path**: for WebAuth, the client needs an IP address, DNS, and a browser session before the portal can appear — a client stuck without DHCP will look like "WebAuth broken." Check the parameter map's Init-State Timeout and Maximum HTTP connections.
7. **Config errors on the WLC**: PSK checked when you meant enterprise (or 802.1x unchecked); WPA Policy left checked alongside WPA2; the AAA Authentication List left unset on the Security > AAA tab; the WebAuth method list Type set to something other than "login"; Web Policy unchecked on the Layer3 tab; parameter map or authentication list not selected.
8. **Config errors off the WLC**: EAP method mismatch — the RADIUS server offers PEAP but the supplicant is configured for EAP-TLS (or vice versa). The WLC will not show you this; it only knows 802.1x is in use.
9. **Provisioning gaps**: the WLAN exists but no Policy profile maps it to a VLAN, or the WLAN/policy profiles were never bound via a policy tag and pushed to the APs. Symptom: a WLAN that looks correct in the GUI but no AP is broadcasting it.
10. **Software bugs / version behavior**: only after the above. Note the controller platform and version (Fig 20-17 shows EWC on Catalyst APs 17.6.4) — GUI field names and available ciphers/AKMs shift between releases.

## Common Pitfalls
- **"Open Authentication must need *something*."** It requires **none** of 802.1x, RADIUS, HTTP/HTTPS, or a pre-shared key (quiz Q1 answer **E**, p.594). The only requirement is an 802.11 authentication request.
- **Assuming Open Authentication is a dead end.** It is routinely combined with **WebAuth** (quiz Q2 answer **B**) — that is how public/guest WLANs screen users. It is not combined with PSK, EAP, or 802.1x as a second Layer 2 method.
- **Thinking the PSK is configured only on clients, or that each client gets a unique key.** Without ISE, **all wireless clients AND all APs/WLCs** must carry the same key string (quiz Q3 answers **B, C**). No RADIUS server is involved.
- **Confusing personal with enterprise.** WPA2 **personal** mode uses **Pre-Shared Key** (quiz Q4 answer **B**), not 802.1x, and not "Open Authentication."
- **Treating all personal modes as equivalent.** They are not — **WPA3** personal is the most secure (quiz Q5 answer **D**) because SAE resists the offline dictionary attack that works against captured WPA/WPA2 four-way handshakes, and it adds forward secrecy.
- **Forgetting that PSK exists in all three WPA versions' personal modes** — WPA personal, WPA2 personal, and WPA3 personal (quiz Q6 answers **A, C, E**). Enterprise modes never use PSK.
- **Believing the four-way EAPOL handshake authenticates the user.** It does not — it **exchanges encryption keys** (quiz Q7 answer **C**). User authentication already happened via EAP with the AS before the handshake begins.
- **Mixing up the 802.1x roles.** With an external RADIUS server, the **WLC is the authenticator** (quiz Q8 answer **C**), the RADIUS server is the authentication server, and the **supplicant lives on the wireless client** (quiz Q9 answer **A**) — not on the AP or WLC.
- **Reaching for 802.1x when the requirement is "read and accept an AUP."** Only **WebAuth** inherently handles presenting a document for the user to read and accept (quiz Q10 answer **D**).
- **Looking for a PEAP or EAP-TLS drop-down on the WLC.** There isn't one, by design. Configure the EAP method on the RADIUS server and match it in the client supplicant (p.602).
- **Enabling both WPA and WPA2 "for compatibility" by default.** The WLAN is only as secure as the weakest suite enabled on it; hybrid modes are deprecated and cause compatibility problems (p.597).
- **Assuming PSK WLANs have no PMK because EAP never runs.** The PMK still exists — it is derived from the pre-shared key itself (NOTE, p.598).
- **Stopping after "Apply to Device."** The WLAN still needs a policy profile for its VLAN and a policy tag applied to APs before any client can join (p.594, p.605).

---

### Ambiguities in the source chapter (flagged, not resolved)
- **Navigation path inconsistency.** The verification path is written as **Configuration > Tags & Profiles > WLANs** on p.594, p.602 and p.605, but as **Configuration > Tags & Policies > WLANs** on p.597 and p.600. The `Tags & Profiles` form matches every screenshot in the chapter; `Tags & Policies` appears to be a book typo. Not resolved against real gear here.
- **WebAuth Security column value.** The p.605 text says the WLAN is shown with `[Web_Auth]`, but Figure 20-17 on p.606 shows `[WPA2][802.1x][AES],[Web Auth]` for the same `Guest_webauth` WLAN — with a space, not an underscore, and layered on a Layer 2 suite. Both forms are recorded above.
- **Chapter says "three," lists four.** The chapter opener says it "explores three different methods to authenticate wireless clients," while p.593 says "the sections that follow explain four types of client authentication" and the chapter covers four (Open, PSK, EAP, WebAuth).
- **PSK GUI walkthrough vs. figure.** The p.595 text says WPA2 or WPA3 personal mode and the PSK are configured "in one step," but Figure 20-4 shows the Auth Key Mgmt list rendered twice (a collapsed and an expanded view) in the same screenshot, which makes the exact click order ambiguous.
- **No CLI anywhere.** Chapter 20 contains zero command-line syntax, so nothing in the Config Patterns block is quotable from the book.
