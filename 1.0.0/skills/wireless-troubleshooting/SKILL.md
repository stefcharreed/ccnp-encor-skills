---
name: ccnp-wireless-troubleshooting
description: >
  Use this skill when troubleshooting wireless client and AP connectivity from a Cisco WLC.
  Invoke when the user asks about: troubleshooting wireless connectivity, wireless client
  cannot connect, "the Wi-Fi is down", scope of a wireless problem, gather information from
  the end user, conditions for a successful wireless association, client is within RF range,
  client authenticates, client gets an IP address, wireless client MAC address, searching a
  client MAC on the WLC, Client 360 View, WLC GUI search bar, Search APs and Clients,
  Monitoring > Wireless > Clients, client association and signal status, client signal
  strength dBm, signal quality SNR, channel width, client device type, client capabilities,
  spatial streams, Client Properties tab, AP Properties tab, Security Information tab,
  policy profile, WLAN profile, BSSID, session timeout, current TxRateSet, MCS, Radioactive
  Trace, conditional debug, Conditional Debug Global State, debug trace file, Enable internal
  logs, Wireless Debug Analyzer, troubleshooting connectivity problems at the AP, split-MAC
  architecture, AP joined the controller, AP Statistics, Join Statistics, last reboot reason,
  last disconnect reason, tag modified, AP 360 View, AP radio details, channel utilization,
  transmit utilization, receive utilization, AP transmit power level, AP operational
  configuration, hierarchy icon, policy tag site tag RF tag, misconfigured APs, FlexConnect
  AP without a controller, defective AP radio.
---

## Purpose
This chapter is the method for turning a vague wireless complaint into a scoped, evidence-backed
diagnosis using the wireless LAN controller as the primary troubleshooting instrument — first
for a single client, then broadening outward to the AP that serves many of them.

## Key Concepts

**Start by scoping, not by acting**
- When users report problems, the first course of action is to **gather more information**.
  Begin with a broad perspective, then ask pointed questions to narrow the scope of possible
  causes. Do not panic or waste time chasing irrelevant things — look for patterns and
  similarities in the answers you get (p.610).
- The report pattern points at the layer to work on (p.610):
  - **Many people in the same area** → an AP is probably misconfigured or malfunctioning.
  - **Many areas, or one single SSID** → likely a controller configuration problem.
  - **Only one user** → don't spend time troubleshooting a controller that is serving many
    other users successfully; focus on that one client device and its interaction with an AP.
- The chapter's sections deliberately move in that order: start on a single client device, then
  broaden outward to where multiple clients might be affected (p.611).

**What a successful association actually requires (Figure 21-1, p.611)**
- The client is **within RF range of an AP and asks to associate**.
- The client **authenticates** (802.1X toward the AAA server, in the figure).
- The client **requests and receives an IP address** (DHCP).
- Therefore "I cannot connect" or "the Wi-Fi is down" is not a diagnosis. It might mean the
  device cannot associate, cannot authenticate, or cannot get an IP address — three different
  problems (p.611).

**What to collect from the user**
- At a minimum you need the **wireless adapter MAC address** of the client device and its
  **physical location** (p.611).
- The user may point at a specific AP in the room or within view. Record it — but the
  **client device selects which AP it wants to use, not the human user**. The device may well
  be using a completely different AP (p.611).

**The WLC is the troubleshooting instrument**
- Most time managing and monitoring a wireless network is spent in the **WLC GUI**. As a
  wireless client probes and attempts to associate with an AP, it is essentially communicating
  with the controller, so a wealth of troubleshooting information is available from the
  controller — **as long as you know the client's MAC address** (p.611).
- IOS-XE-based WLCs (for example the Catalyst 9800) present monitoring, configuration,
  administration, licensing, and troubleshooting functions in the GUI. The default screen shows
  a network summary dashboard on the right and a menu of functions on the left (p.611–612).
- Entering a known client MAC in the **Search APs and Clients** bar at top right returns a
  **Client MAC Result** match; selecting it opens the **Client 360 View** (p.611–612, Figures
  21-3 through 21-5).

**What each WLC screen tells you**
- **Client 360 View** (p.613): username if known, wireless MAC, connection uptime, WLAN name
  (SSID), the AP name where the client is associated and its channel, signal strength at which
  the AP received the client, signal quality (SNR), channel width, device type, wireless
  capabilities (802.11n, 5 GHz, spatial streams), plus Top Applications on the right.
- **General > Client Properties** (p.614): client MAC, IPv4/IPv6 address, policy profile in use
  on the AP, WLAN/SSID, the AP's **BSSID**, connection uptime, session timeout, and toward the
  bottom the current transmit rate set or **MCS** in use, plus QoS and mobility/roaming activity.
- **General > AP Properties** (p.614): the client's AP wired MAC address and name, the client's
  current status (associated or not), the 802.11 protocol in use, and the current channel number.
- **General > Security Information** (p.615): security parameters for the client — in the
  example, WPA2, CCMP (AES), pre-shared key, no EAP, and an 1800-second session timeout.
- **Radioactive Trace** (p.615–616): collects detailed WLC-perspective event logs triggered by
  specific MAC addresses over a period of time. Use it when a client is having problems but the
  **root cause isn't obvious**.
- **Monitoring > Wireless > AP Statistics** (p.617): searching an AP name shows whether the AP is
  found and joined, its Admin status (green checkmark = up), AP model, EWC capability, image type,
  IP address, AP radio MAC, and Ethernet MAC.
- **Join Statistics tab** (p.618): basic information about the last known events when the AP tried
  to **join or leave** the controller — including **Last Reboot Reason (Reported by AP)** and
  **Last Disconnect Reason**.
- **AP 360 View** (p.618): AP location, IP address, model, serial number, PoE status, country
  code, WPA3 capability, VLAN tag number, software version, and connection uptime.
- **AP radio details in the 360 View** (p.618–619): per-slot radio type, admin status, number of
  clients on each radio, current channel, transmit power level, and **channel / transmit / receive
  utilization** indicators.
- **AP Operational Configuration** (p.619): the small **hierarchy icon** next to the AP's name
  opens a hierarchical view of the entire profile and tag structure applied to that AP.

**Split-MAC means several places to troubleshoot (p.617)**
- If multiple users in the same general area report problems, focus on an AP. The problem could
  be as simple as a **defective radio** where no clients are receiving a signal — in that case you
  may have to go onsite to confirm the transmitter is not working.
- Otherwise, split-MAC creates several distinct points to troubleshoot. Successfully operating a
  lightweight AP and providing a working BSS requires:
  - The AP must have **connectivity to its access layer switch**.
  - The AP must have **connectivity to its WLC**, *unless* it is operating in **FlexConnect mode**.
- Verifying AP-to-controller connectivity is normally done when a **new AP is installed**, to make
  sure it can discover and join a controller **before clients arrive** — and can be repeated any
  time as a quick health check of the AP.

**Tags and profiles are a troubleshooting surface (p.619)**
- Each AP must be mapped to **three tags**, each mapping to one or more types of profiles. With
  several tags configured for unique requirements around a campus, the associations are hard to
  remember — and a problem may be caused by **unexpected or misconfigured profiles or tags**.
- In the chapter's example, AP `ap-gndfloor-1` uses policy tag `my-policy-tag`, site tag
  `default-site-tag`, and RF tag `my-rf-tag` (Figure 21-17, p.619). The viewer also shows the
  individual profiles referenced by each tag and some basic operational information used by each
  profile (p.620).

**Chapter housekeeping** — Chapter 21 has **no memory tables and no key terms** (p.620). Its key
topics are Figure 21-1 (p.611), Figure 21-5 Client 360 View (p.613), Figure 21-9 Troubleshooting a
Wireless Client (p.615), Figure 21-14 AP 360 View (p.618), and Figure 21-16 AP Operational
Configuration (p.619).

## Procedure

### A. Scope the problem from user reports (p.610–611)
1. Gather more information before touching anything — start broad, then ask pointed questions to
   narrow the scope.
2. Look for patterns across the reports: same area? many areas or one SSID? a single user?
3. Route on the pattern: same area → suspect an AP; many areas or one SSID → suspect controller
   configuration; single user → focus on that client device and its interaction with an AP.
4. Ask the user what the device is actually experiencing — associate, authenticate, or IP address —
   rather than accepting "I cannot connect."
5. Collect, at minimum, the client's **wireless adapter MAC address** and its **physical location**.
6. Record any AP the user names, but treat it as unconfirmed: the client device, not the user,
   chooses the AP.

### B. Inspect a single client from the WLC GUI (p.611–615)
1. Open a browser to the WLC management address; the default dashboard screen appears.
2. Enter the client's MAC address in the **Search APs and Clients** bar at top right.
3. Select the returned **Client MAC Result** match to open the **Client 360 View**.
4. On **360 View**, read username, MAC, uptime, WLAN/SSID, AP name and channel, and the
   **Client Performance** line — signal strength, signal quality (SNR), and channel width.
5. Open **General > Client Properties** for IP addressing, policy profile, WLAN/SSID, BSSID,
   session timeout, and the current TxRateSet / MCS.
6. Open **General > AP Properties** for the AP's wired MAC and name, associated status, 802.11
   protocol, and current channel.
7. Open **General > Security Information** for policy type, encryption cipher, AKM, EAP type, and
   session timeout.

### C. Run a Radioactive Trace end to end (p.615–616)
1. Navigate to the trace controls one of two ways: select **Troubleshooting > Radioactive Trace**
   from the main default WLC screen, **or** select the small **wrench icon** underneath the client's
   MAC address on the **Monitoring > Wireless > Clients** page ("Click here to troubleshoot this
   client").
2. On the Radioactive Trace page, confirm the client MAC is populated in the list. Select **Add**
   to add more clients so troubleshooting information is collected for them too.
3. Select **Start** to begin collecting WLC logs involving the MAC addresses in the list. The
   **Conditional Debug Global State** changes from *Stopped* to *Started*.
4. Let the trace run until the client has had a chance to try to join the network again, or has
   experienced the event that needs further investigation.
5. Select **Stop** to end data collection.
6. Select the green **Generate** button next to the client's MAC address to create a readable debug
   trace file.
7. In the pop-up window, choose the time interval of logs to collect: **the last 10 minutes, the
   last 30 minutes, the last one hour, or since the last WLC reboot**. Check the **Enable internal
   logs** box to use the logs collected by the WLC.
8. After the file is generated, two small blue icons appear next to the client's MAC address:
   select the **downward-arrow icon** to download the file (e.g. `debugTrace_38c9.86ed.476e.txt`)
   to your local machine, or the **document icon** ("View Logs") to display the trace file contents
   in the bottom portion of the Radioactive Trace page.
9. Check the **Last Run Result** panel (State, MAC/IP address, start time, end time, trace file)
   and the **Wireless Debug Analyzer** result on the right side of the page.

### D. Verify an AP's connectivity and health from the WLC (p.617–619)
1. Enter the AP's name in the search bar to look for it in the list of live APs that have joined
   the controller.
2. If the search reveals a live AP, select it to display basic **AP Statistics** — verify the AP is
   **found**, that Admin status is a green checkmark (up), that it is joined, and that it has a
   **valid IP address**, along with model, EWC capability, image type, radio MAC, and Ethernet MAC.
3. Select the **Join Statistics** tab for the last known join/leave events, including
   **Last Reboot Reason (Reported by AP)** and **Last Disconnect Reason**.
4. Select the AP's name from the **AP Name** column to open its **360 View**: location, IP address,
   model, serial number, PoE status, country code, WPA3 capability, VLAN tag, software version,
   join date/time, and uptime.
5. Scroll down in the 360 View to the per-radio operational summary: confirm both radios are
   enabled, note the number of clients on each, the current channel, the transmit power level, and
   the channel / transmit / receive utilization indicators.
6. If a misconfigured profile or tag is suspected, select the small **hierarchy icon** next to the
   AP's name ("AP Operational Configuration") to display the AP's full policy tag / site tag /
   RF tag hierarchy and the profiles each tag references.

## Reference Tables

### Observed client signal values and what the chapter says about them (p.613)
| Observed value | Chapter's own interpretation |
|---|---|
| Signal strength −43 dBm | "sufficiently strong" |
| Signal quality (SNR) 51 dB | "very good" |
| Channel width 40 MHz | Stated as the client's current channel width (no judgement given) |
| Signal strength −75 dBm | "rather low"; client most likely using a low data rate |
| SNR 18 dB | "rather low"; client most likely using a low data rate |

The chapter's read on the −75 dBm / 18 dB case: it is safe to assume the client has moved too far
from the AP where it is associated, so the signal strength is too low to support faster performance.
That may mean a new AP is needed in that area to boost RF coverage, or the client device is not
roaming soon enough to a new AP with a stronger signal (p.613). Note the chapter states no numeric
good/bad threshold — only that **SNR can be a low value for lower data rates to be used
successfully, but must be greater to leverage higher data rates** (p.613).

### Observed AP radio values (Figure 21-15, p.619)
| Item | Slot 0 (2.4 GHz) | Slot 1 (5 GHz) |
|---|---|---|
| Radio type | 802.11ax – 2.4 GHz | 802.11ax – 5 GHz |
| Radio role | Remote | Remote |
| Admin status | Enabled | Enabled |
| Number of clients | 0 | 2 |
| Current channel | 11 | 64 |
| Power level | *1/8 (20 dBm) | *1/8 (20 dBm) |
| Channel utilization | 3% | 0% |
| Transmit utilization | 0% | 0% |
| Receive utilization | 0% | 0% |

The chapter's only stated rule for these: **if channel utilization values are high, you can assume
the channel is heavily used, probably slowing communication and making it difficult for wireless
stations to have an opportunity to transmit** (p.618). The one worked example of "high" is the quiz
scenario of **85% channel utilization**, whose correct reading is that utilization is too high and
is keeping clients from using the channel (p.610, answer 7 D on p.612). No numeric threshold is
given anywhere in the chapter.

### Quiz-derived signal readings (p.608–610, answers p.612)
| Scenario as printed | Correct reading per the answer key |
|---|---|
| SNR of 5 on the 2.4 GHz band | SNR is at a very low level, which is **bad** for wireless performance (6 C) |
| Channel 60, 5 dBm transmit power, 6 clients, "45 dBm SNR", 85% channel utilization | The **channel utilization is too high**, keeping clients from using the channel (7 D) |

## Config Patterns

```ios-xe
! NOT FROM THIS CHAPTER — see the note below. Book-derived equivalents only,
! NOT gear-validated by me. Chapter 21 contains zero CLI.
!
! Conditional debug / Radioactive Trace equivalent on a Catalyst 9800 (IOS XE):
debug wireless mac 38c9.86ed.476e internal
! ... let the client reattempt the join ...
no debug wireless mac 38c9.86ed.476e internal
! the resulting trace file is written to bootflash: and is read with
!   dir bootflash: | include ra_trace
!   more bootflash:ra_trace_MAC_38c986ed476e_<timestamp>.txt

! Wireless security settings the GUI shows read-only under General > Security Information
! are configured in a WLAN profile; never put the real key in a repo:
wlan devices 2 devices
 security wpa wpa2
 security wpa wpa2 ciphers aes
 security wpa akm psk set-key ascii 0 <from-secrets-not-repo>
 no shutdown
```

**Note:** Chapter 21 is almost entirely **GUI-driven** — every technique in it is a WLC web UI
navigation path (Search APs and Clients bar, Client 360 View, General > Client Properties /
AP Properties / Security Information, Troubleshooting > Radioactive Trace, Monitoring > Wireless >
Clients, Monitoring > Wireless > AP Statistics, the Join Statistics tab, and the AP Operational
Configuration hierarchy icon). Those paths are recorded faithfully in the **Procedure** section
above; that is the chapter's actual method. The CLI block above is **book-derived / general-knowledge
equivalent, not printed in this chapter and not validated on gear by me** — treat it as a lead to
verify on a real 9800, not as a citation.

## Design Baseline

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| Gather information and find the scope of the problem before taking any action on the WLC | Prevents panicking and chasing irrelevant causes; report patterns point directly at AP vs controller vs single client | An outage with an obvious, already-known trigger (a change just made) may be diagnosed from the change record first | ENCOR 350-401 OCG, Ch. 21, p.610 (and quiz Q1, answer B, p.612) |
| Collect at minimum the client's wireless adapter MAC address and its physical location | The MAC is the key that unlocks every WLC screen; location is what makes AP/RF conclusions possible | Device is unreachable or the user cannot retrieve the MAC — then search by AP or SSID instead | ENCOR 350-401 OCG, Ch. 21, p.611 |
| Record but do not trust the AP the user names | The client device selects which AP it uses, not the human user; the device may be on a completely different AP | None stated in the chapter — the AP Properties tab confirms the real AP anyway | ENCOR 350-401 OCG, Ch. 21, p.611 |
| Search the client's MAC address on the controller rather than checking every AP or every AP status | Most efficient way to find a client when the user gave no AP/controller details | A very small single-controller network where the AP list is trivially short | ENCOR 350-401 OCG, Ch. 21, quiz Q4, answer C, p.612 |
| For a failure that already happened and has no obvious root cause, run a Radioactive Trace on the user's MAC and analyze the output | Collects the WLC-side event logs for that MAC over a time window instead of relying on the user's retelling | The symptom is already explained by 360 View data (e.g. −75 dBm signal), so no trace is needed | ENCOR 350-401 OCG, Ch. 21, p.615 and quiz Q3, answer D, p.612 |
| Check the "Enable internal logs" box when generating the trace file | Uses the logs collected by the WLC itself in the readable trace output | None stated | ENCOR 350-401 OCG, Ch. 21, p.616 |
| Verify a newly installed AP can discover and join its controller before clients arrive | Catches AP-to-switch or AP-to-WLC problems before they become user-reported outages | AP is deliberately running FlexConnect, which does not require WLC connectivity to keep serving | ENCOR 350-401 OCG, Ch. 21, p.617 |
| When searching a new AP by name, verify it is found, that Admin status is up/joined, and that it has a valid IP address | These are the concrete pass conditions for "the AP joined" | None stated | ENCOR 350-401 OCG, Ch. 21, p.617 and quiz Q5, answers A/B/C, p.612 |
| When a problem might be caused by unexpected or misconfigured profiles or tags, open the AP's Operational Configuration hierarchy | Each AP maps to three tags and many profiles; the hierarchy view is the fast way to see what is actually applied | Single-site network using only the default tags | ENCOR 350-401 OCG, Ch. 21, p.619 |
| Treat "the AP disconnected because a tag was modified" as expected, not as a fault | A tag change causes the AP to refresh its controller connection or cycle radios/WLANs to commit the change | Repeated tag-modified disconnects with no corresponding change activity are worth questioning | ENCOR 350-401 OCG, Ch. 21, p.618 |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| **GUI:** WLC search bar ("Search APs and Clients") with a client MAC | A "Client MAC Result" match; if none, the controller does not know this client (p.611–612) |
| **GUI:** Monitoring > Wireless > Clients | Client MAC, IPv4/IPv6 address, AP name, SSID, WLAN ID, client type, State (e.g. Run), protocol (e.g. 11n(5)) (p.615) |
| **GUI:** Client > 360 View | Username, uptime, WLAN name, AP name + channel, Client Performance: signal strength (dBm), signal quality (SNR dB), channel BW, device type, capabilities/spatial streams (p.613) |
| **GUI:** Client > General > Client Properties | IPv4/IPv6, policy profile, flex profile, WLAN profile/SSID, BSSID, uptime, session timeout + remaining, Current TxRateSet / MCS, supported rates (p.614) |
| **GUI:** Client > General > AP Properties | AP MAC, AP name, AP slot, Status = Associated, protocol, channel, association ID, authentication algorithm (p.614) |
| **GUI:** Client > General > Security Information | Policy type (WPA2), encryption cipher (CCMP AES), AKM (PSK), EAP type, session timeout, Authorized = TRUE (p.615) |
| **GUI:** Troubleshooting > Radioactive Trace (or the wrench icon under the client MAC on Monitoring > Wireless > Clients) | Conditional Debug Global State Started/Stopped, MAC list, generated trace file, Last Run Result, Wireless Debug Analyzer (p.615–616) |
| **GUI:** Monitoring > Wireless > AP Statistics (search AP name) | AP found; Admin status green checkmark; joined; AP model; EWC capable; image type; valid IP address; radio MAC; Ethernet MAC (p.617) |
| **GUI:** AP Statistics > Join Statistics tab | Last Reboot Reason (Reported by AP) and Last Disconnect Reason — e.g. "Tag modified" (p.618) |
| **GUI:** AP name > 360 View | Location, IP, model, serial, PoE status, country code, WPA3 capability, AP VLAN tag, software version, join date/time, uptime (p.618) |
| **GUI:** AP name > 360 View, scrolled to radio slots | Per-slot admin status, number of clients, current channel, power level, channel/transmit/receive utilization (p.618–619) |
| **GUI:** AP Operational Configuration (hierarchy icon next to the AP name) | Policy tag / site tag / RF tag applied to the AP and the profiles each references (p.619) |
| `show wireless client mac-address <H.H.H> detail` | CLI counterpart to the Client 360 View / Client Properties data — **not in this chapter; unverified on gear** |
| `show wireless client summary` | CLI counterpart to Monitoring > Wireless > Clients — **not in this chapter; unverified on gear** |
| `show ap summary` | CLI counterpart to AP Statistics: AP name, model, IP, joined state — **not in this chapter; unverified on gear** |
| `show ap join stats summary` | CLI counterpart to the Join Statistics tab — **not in this chapter; unverified on gear** |
| `show ap dot11 24ghz summary` / `show ap dot11 5ghz summary` | CLI counterpart to the per-radio channel/power/utilization view — **not in this chapter; unverified on gear** |
| `show ap tag summary` | CLI counterpart to AP Operational Configuration tags — **not in this chapter; unverified on gear** |

## Intent Questions
1. What is this SSID/WLAN supposed to provide on this network — which user population, which
   security policy (WPA2/PSK vs 802.1X/EAP), and which VLAN or policy profile should the client
   land on?
2. Which AP is *supposed* to cover the location the user is standing in, and on which band and
   channel — and is the client actually associated to that AP, or to a different one it chose?
3. What tags and profiles should this AP be carrying (policy tag, site tag, RF tag), and is the
   AP expected to depend on the controller at all, or is it a FlexConnect AP that keeps serving
   without one?
4. Is this a new install that has never worked (has the AP ever joined?) or a working system that
   changed — and if it changed, was a tag or profile modified recently?

## Troubleshooting Checklist
0. **State intent vs. observed.** Answer the Intent Questions above for this network, then write
   the one-line symptom ("should ___, isn't ___") — before running any show command or opening any
   WLC screen.
1. **Scope it from the reports first (p.610).** Many users in one area → work the AP. Many areas or
   one SSID → work the controller configuration. A single user → work that client device and its
   interaction with an AP. Ask what "cannot connect" really means: associate, authenticate, or IP.
2. **Layer 1 / RF.** Get the client's wireless MAC and physical location. In the Client 360 View,
   read signal strength and SNR. A low pair (the chapter's example: −75 dBm with 18 dB SNR) means
   the client is too far from its AP for faster rates — consider added AP coverage or a client that
   is not roaming soon enough. On the AP side, check channel utilization on the serving radio; high
   utilization means the channel is heavily used and stations struggle to get airtime (p.613, p.618).
   If no client at all is receiving a signal from one AP, suspect a **defective radio** and go onsite
   to confirm the transmitter (p.617).
3. **Layer 1/2 for the AP itself (split-MAC, p.617).** Confirm the AP has connectivity to its access
   layer switch, and connectivity to its WLC — unless it is a FlexConnect AP. On the WLC, search the
   AP name: is it found, is Admin status up, is it joined, does it have a valid IP address?
4. **Layer 2 association and authentication.** In AP Properties confirm Status = Associated, the
   802.11 protocol, and the channel. In Security Information confirm the policy type, cipher, AKM,
   EAP type, and that Authorized is TRUE. A client that associates but never authenticates is a
   different problem from one that never associates (p.611, p.614–615).
5. **Layer 3.** In Client Properties confirm the client actually received an IPv4 address (and IPv6
   if expected) — the third condition for a successful association is requesting and receiving an
   IP address (p.611, p.614).
6. **Configuration errors.** Check the policy profile, WLAN profile/SSID, BSSID, and VLAN the client
   landed on against intent. Then open the AP's **Operational Configuration** hierarchy and verify
   the policy tag, site tag, and RF tag and the profiles they reference — unexpected or misconfigured
   profiles/tags are an explicitly called-out cause (p.614, p.619). Also check the Misconfigured APs
   counters (Tag, Country Code, LSC Fallback) on the AP Statistics page (p.617, p.619).
7. **History and stability.** Check Join Statistics for the AP's last reboot reason and last
   disconnect reason. "Tag modified" is normally expected behavior — a tag change makes the AP
   refresh its controller connection or cycle its radios/WLANs to commit the change (p.618). Check
   AP uptime and join date/time in the AP 360 View against when users started complaining (p.618).
8. **Non-obvious root causes / software behavior.** When the cause still isn't obvious, run a
   **Radioactive Trace** on the client's MAC (Procedure C), let the client reattempt while it runs,
   generate over an interval that covers the failure (10 min / 30 min / 1 hour / since last WLC
   reboot) with **Enable internal logs** checked, then download or view the trace and use the
   Wireless Debug Analyzer result (p.615–616).

## Common Pitfalls
- **Rebooting the controller, or closing the ticket because no alarms exist, instead of scoping the
  problem.** The correct first step is to gather more information to find the scope (quiz Q1, answer
  B — p.607, p.612). "No alarms found" is not a diagnosis.
- **Trusting the AP the user points at.** The **client device**, not the human, selects which AP it
  uses; the device may well be using a completely different AP. Record the user's claim, then verify
  in AP Properties (p.611).
- **Asking for the wrong identifier.** The **wireless** MAC address of the client adapter is what
  finds the client on the WLC — not the Ethernet MAC, not the username, not the application name
  (quiz Q2, answer C — p.612).
- **Chasing a past failure by asking the user to retry or by bouncing the WLAN.** For a join that
  failed several minutes ago with the correct SSID, run a Radioactive Trace on the user's MAC and
  analyze the output (quiz Q3, answer D — p.612). Disabling and re-enabling the SSID disrupts every
  other user on it and destroys the evidence.
- **Brute-forcing the search.** Walking to the site with your own laptop, checking every AP on every
  WLC, or searching for the client MAC on each AP are all less efficient than searching the client's
  MAC address on each controller (quiz Q4, answer C — p.612).
- **Misreading SNR direction.** SNR is how many decibels the signal is above the noise floor, so
  **low SNR is bad**, not good — an SNR of 5 is a very low level and is bad for wireless performance
  (quiz Q6, answer C — p.612). Note also that "45 dBm SNR" as printed in quiz Q7 is a unit error;
  SNR is expressed in dB (p.613).
- **Misreading channel utilization direction.** High channel utilization is the problem — it means
  the channel is heavily used and stations have difficulty getting an opportunity to transmit. Low
  utilization does not keep clients off the channel (quiz Q7, answer D — p.612; p.618).
- **Expecting the wrong thing from a Radioactive Trace.** It collects WLC event logs triggered by a
  **specific MAC address for a period of time** — it is not a roaming-path trace, not a CPU/memory
  trace, and not a per-AP radio activity trace (quiz Q8, answer B — p.612).
- **Forgetting to Stop before Generate.** The sequence is Add → Start → let it run → Stop →
  Generate → choose interval → download/view. Generating without having collected across the failure
  window produces nothing useful (p.616).
- **Treating every AP disconnect as a fault.** A "Tag modified" disconnect reason is usually expected
  behavior (p.618).
- **Assuming an AP must reach the WLC to serve clients.** A **FlexConnect** AP is the stated
  exception to the WLC-connectivity requirement (p.617).
- **Assuming there is a threshold table to memorize.** This chapter gives interpreted example values
  (−43 dBm / 51 dB "very good"; −75 dBm / 18 dB "rather low"; 85% utilization "too high") but never
  states numeric good/bad boundaries. Do not invent them.
