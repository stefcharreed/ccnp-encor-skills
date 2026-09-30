---
name: enarsi-note
description: Create a new CCNP ENARSI (300-410) topic skill from study notes. Use when the user invokes /enarsi-note or wants to capture an ENARSI networking topic as a reusable skill file.
argument-hint: <topic-slug>
allowed-tools: [Read, Write, Bash]
---

# ENARSI Note — Topic Skill Creator

Creates a structured CCNP ENARSI (300-410) skill file from the user's study notes.

**This skill deliberately does not restate the template.** The ENCOR and ENARSI catalogs use
the *same* skill template, and duplicating ~250 lines of instructions across two files would
guarantee they drift apart. Instead:

> **Read `~/git/ccnp-encor-skills/1.0.0/skills/ccnp-note/SKILL.md` first and
> follow it in full**, with the four substitutions below. If that file and this one ever
> disagree about the template, `ccnp-note` wins — it is the source of truth.

## Substitutions

| `ccnp-note` says | For ENARSI, use instead |
|---|---|
| Target path `1.0.0/skills/<topic>/SKILL.md` | **`1.0.0/skills-enarsi/<topic>/SKILL.md`** |
| Frontmatter `name: ccnp-<topic>` | **`name: enarsi-<topic>`** |
| README section "Roadmap — ENCOR exam domains" | **"Roadmap — ENARSI exam domains"** |
| ENCOR's six domains | **ENARSI's four domains** (below) |

Everything else is identical: the same section order (Purpose, Key Concepts, Procedure,
Reference Tables, Config Patterns, Design Baseline, Verification Commands, Intent Questions,
Troubleshooting Checklist, Common Pitfalls, Exam Preparation Tasks), the same "no source, no
row" rule for Design Baseline, the same honesty rules for the Key Topics coverage map and the
quiz analysis, and the same commit-and-push-immediately workflow.

## ENARSI (300-410) exam domains

Place the new skill under whichever of these four the topic belongs to:

| Domain | Weight | Typical topics |
|---|---|---|
| **1. Layer 3 Technologies** | **35%** | Administrative distance, route maps, loop prevention (filtering, tagging, split horizon, route poisoning), redistribution, summarization, PBR, VRF-Lite, BFD (describe), EIGRP troubleshooting (named/classic, IPv4/IPv6, stubs, load balancing, metrics), OSPFv2/v3 troubleshooting (network/area/router types, virtual links, path preference), BGP troubleshooting (iBGP/eBGP, VRF-Lite, next-hop, multihop, 4-byte/private AS, peer groups, path selection, route reflector, policies) |
| **2. VPN Technologies** | **20%** | MPLS operations (describe: LSR, LDP, label switching, LSP), MPLS L3VPN (describe), DMVPN single hub (GRE/mGRE, NHRP, **IPsec as DMVPN protection**, dynamic neighbor, spoke-to-spoke) |
| **3. Infrastructure Security** | **20%** | IOS AAA (TACACS+, RADIUS, local), IPv4 ACLs (standard, extended, time-based), IPv6 traffic filters, uRPF, CoPP, **IPv6 first-hop security** (describe: RA guard, DHCP guard, binding table, ND inspection/snooping, source guard) |
| **4. Infrastructure Services** | **25%** | Device management (console/VTY, Telnet, HTTP(S), SSH, SCP, (T)FTP), SNMP v2c/v3, logging (local, syslog, debugs, conditional debugs, timestamps), IPv4/IPv6 DHCP (client, IOS server, relay, options), IP SLA (jitter, tracking objects, delay, connectivity), NetFlow (v5, v9, Flexible NetFlow), **DNA Center assurance** |

Source: Cisco's ENARSI **v1.1** blueprint PDF, checked 2026-09-29. **NTP and NETCONF/RESTCONF are not on ENARSI v1.1** (they are ENCOR topics). Re-check the blueprint if Cisco releases a new version.

## Overlap with the ENCOR catalog

**ENARSI revisits several topics ENCOR already covers** — EIGRP, OSPF, BGP, ACLs, CoPP, AAA,
SNMP, NetFlow, IP SLA — usually at greater depth and with a troubleshooting
rather than an implementation slant. Before writing a new ENARSI skill:

1. **Check whether an ENCOR skill already covers it** (`ls 1.0.0/skills/`).
2. If one does, **do not silently duplicate it.** Either:
   - **Extend the ENARSI skill to the delta only** — the deeper troubleshooting material ENARSI
     adds — and cross-reference the ENCOR skill by name for the fundamentals; or
   - **Ask the user** whether they want a standalone ENARSI skill instead, which is the right
     call when the ENARSI treatment is substantially different.
3. **Say which you did** in the confirmation, and add the cross-reference in both directions
   where it helps.

This mirrors the repo's "reconcile, don't blind-merge" rule: two files covering the same
protocol that disagree are worse than one file that is honest about its scope.

## Commit message

Use `Add <topic> ENARSI skill` (or `Update …`) so the two catalogs stay distinguishable in
`git log`. Pushing is pre-authorized for this repo, same as for ENCOR.
