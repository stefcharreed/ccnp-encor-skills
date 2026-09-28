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

> **Read `~/.claude/plugins/cache/local/ccnp-encor/1.0.0/skills/ccnp-note/SKILL.md` first and
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
| **1. Layer 3 Technologies** | **35%** | EIGRP and OSPFv2/v3 troubleshooting, BGP (eBGP/iBGP, attributes, communities), route redistribution, route maps and prefix lists, administrative distance, VRF-lite, policy-based routing, routing loop prevention |
| **2. VPN Technologies** | **20%** | MPLS (LDP, L3VPN), DMVPN (phases 1–3, NHRP, mGRE), IPsec (site-to-site, tunnel vs transport) |
| **3. Infrastructure Security** | **20%** | ACLs, uRPF, CoPP, device access control and AAA, control plane protection |
| **4. Infrastructure Services** | **25%** | Device management (SSH, console, logging), SNMP, NetFlow/Flexible NetFlow, IP SLA and object tracking, NETCONF/RESTCONF, DHCP, NTP |

## Overlap with the ENCOR catalog

**ENARSI revisits several topics ENCOR already covers** — EIGRP, OSPF, BGP, ACLs, CoPP, AAA,
SNMP, NetFlow, IP SLA, NETCONF/RESTCONF — usually at greater depth and with a troubleshooting
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
