# CCNP ENCOR Skills

A personal CCNP ENCOR (350-401) knowledge base, captured as I study and structured as
installable [Claude Code](https://claude.com/claude-code) skills rather than static notes.

Each topic I study gets turned into a `SKILL.md` file — purpose, key concepts, real
IOS-XE config patterns, verification commands, and an ordered troubleshooting
checklist — generated live via a custom `/ccnp-note <topic>` slash command as I work
through the material. The result is a knowledge base that's both human-readable and
directly usable by an AI coding agent for config review, troubleshooting walkthroughs,
or lab work.

I'm a network engineer (NOC tech → network engineer) moving deeper into network
security and automation (NetDevOps). This repo is part of that work in public —
more written up on [LinkedIn](https://www.linkedin.com/in/stefan-c-reed/) as I go.

## What's next — a troubleshooting agent

This catalog is phase 1: one self-contained skill per topic. Phase 2 is combining
these into a single agent that can diagnose real network problems across multiple
topic areas at once — the way an experienced engineer actually troubleshoots, not
one isolated skill at a time.

That phase 2 work converges with another project of mine,
[netmiko-config-audit](https://github.com/stefcharreed/netmiko-config-audit) — a
config drift tool that already exposes an MCP server. Rather than building a
separate AI layer for each project, phase 2 composes this skill catalog's
knowledge with that tool's MCP server to reason over real config drift, not just
static topics. It's happening in a private repo since it'll likely involve real
device interaction patterns.

If that's interesting to you, [message me on LinkedIn](https://www.linkedin.com/in/stefan-c-reed/) — happy to talk through it.

## How it works

- `/ccnp-note <topic>` (a Claude Code skill) takes raw study notes, textbook excerpts,
  config blocks, or `show` command output and writes a structured skill file to
  `1.0.0/skills/<topic>/SKILL.md`.
- Every skill follows the same template: Purpose, Key Concepts, Config Patterns,
  Verification Commands, Troubleshooting Checklist, Common Pitfalls.
- Every new or updated skill is committed and pushed immediately — see
  [CLAUDE.md](CLAUDE.md) for the repo's working rules.

## Roadmap — ENCOR exam domains

Tracking progress against the six domains in Cisco's ENCOR (350-401) blueprint.

### 1. Architecture (15%)
- [ ] Not yet started

### 2. Virtualization (10%)
- [ ] Not yet started

### 3. Infrastructure (30%)
- [x] [VLANs](1.0.0/skills/VLANS/SKILL.md) — 802.1Q, access/trunk ports, native/allowed VLANs
- [x] [MAC addresses & Layer 2 forwarding](1.0.0/skills/mac-addresses/SKILL.md) — MAC table, CAM, collision domains
- [x] [Layer 3 forwarding](1.0.0/skills/layer-3-forwarding/SKILL.md) — ARP, routing table, SVIs, routed ports
- [x] [Spanning Tree Protocol (802.1D)](1.0.0/skills/spanning-tree-protocol/SKILL.md) — port states/roles, root election, TCNs
- [x] [Rapid Spanning Tree (802.1W)](1.0.0/skills/rapid-spanning-tree-protocol/SKILL.md) — discarding state, proposal/agreement
- [x] [Advanced STP tuning](1.0.0/skills/advanced-stp-tuning/SKILL.md) — root guard, BPDU guard/filter, loop guard, UDLD
- [x] [Multiple Spanning Tree (802.1S)](1.0.0/skills/mst-protocol/SKILL.md) — MSTIs, IST, MST regions, PVST simulation
- [x] [VLAN Trunks & EtherChannel bundles](1.0.0/skills/vlan-trunks-and-etherchannel/SKILL.md) — VTP, DTP, LACP/PAgP, load-balancing hashes
- [x] [IP Routing Essentials](1.0.0/skills/ip-routing-essentials/SKILL.md) — RIB/FIB, AD, static routes, PBR, VRF
- [x] [EIGRP](1.0.0/skills/eigrp/SKILL.md) — DUAL, successor/feasible successor, wide metrics, query/reply convergence
- [x] [OSPF](1.0.0/skills/ospf/SKILL.md) — LSDB, SPF, DR/BDR election, network types, neighbor adjacency
- [x] [Advanced OSPF](1.0.0/skills/advanced-ospf/SKILL.md) — multi-area design, ABRs, LSA types 1-7, discontiguous networks, summarization, route filtering
- [x] [OSPFv3](1.0.0/skills/ospfv3/SKILL.md) — OSPF for IPv6, link-local adjacencies, link/intra-area prefix LSAs, IPv4 support
- [ ] Layer 3 routing protocols (BGP fundamentals)
- [ ] Wireless architecture (autonomous, centralized, cloud-based, embedded)

### 4. Network Assurance (10%)
- [ ] Not yet started

### 5. Security (20%)
- [ ] Not yet started

### 6. Automation (15%)
- [ ] Not yet started

## Repo layout

```
1.0.0/
  package.json          plugin manifest
  skills/
    <topic>/SKILL.md     one structured skill per topic
    ccnp-note/SKILL.md   the skill that generates new topic skills
```

## License

[CC BY-NC-SA 4.0](LICENSE) — share and adapt with attribution, non-commercial
use only, same license for derivatives.

Study notes derived from CCNP ENCOR coursework, restructured in my own words for
reuse as an AI agent skill catalog. Shared for transparency into how I study and
build tooling — not a substitute for the original course material.
