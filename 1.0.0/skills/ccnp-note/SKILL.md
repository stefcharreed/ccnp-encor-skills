---
name: ccnp-note
description: Create a new CCNP ENCOR topic skill from study notes. Use when the user invokes /ccnp-note or wants to capture a networking topic as a reusable skill file.
argument-hint: <topic-slug>
allowed-tools: [Read, Write, Bash]
---

# CCNP Note — Topic Skill Creator

Creates a structured CCNP ENCOR skill file from the user's study notes.

## What to do

The user invoked `/ccnp-note $ARGUMENTS`.

**Topic slug:** `$ARGUMENTS` (e.g. `ospf-neighbor-states`, `bgp-path-selection`). If empty, ask for one.

### Step 1 — Gather content

Ask the user to provide the topic content in any of these forms (accept all that they give):
- Paste raw study notes
- Paste IOS-XE config blocks
- Paste `show` command output with explanation
- Describe the topic in their own words

Also offer to read any currently open file if they have notes there. If they say yes, ask for the path and use Read.

### Step 2 — Write the skill file

Target path: `~/.claude/plugins/cache/local/ccnp-encor/1.0.0/skills/$ARGUMENTS/SKILL.md`

Use this exact template — fill every section from the user's content:

```
---
name: ccnp-$ARGUMENTS
description: >
  Use this skill when troubleshooting or configuring $ARGUMENTS on IOS-XE.
  Invoke when the user asks about: <list 5-8 relevant keywords from the topic>.
---

## Purpose
<One sentence: what this topic controls and why it matters.>

## Key Concepts
- <concept 1>
- <concept 2>
- ...

## Procedure
<Only include this section if the source notes describe one or more explicit
sequential processes — a handshake, a convergence sequence, an election
algorithm, an ordered set of phases/steps. Omit the section entirely if the
topic is purely conceptual with no real sequence to preserve. If the source
material has more than one distinct procedure (e.g. election logic AND a
separate convergence/teardown flow), give each its own short label and
numbered list under this same heading — don't collapse multiple sequences
into one, and don't cap it at a single procedure.>
<Procedure name 1>:
1. <step 1>
2. <step 2>
3. ...

<Procedure name 2, if the source has another distinct sequence>:
1. <step 1>
2. ...

## Reference Tables
<Only include this section if the source notes contain tabular reference
data — e.g. a cost/value table, a port state/role matrix, a timer defaults
table, a comparison table. Reproduce it as a real markdown table, not a
bullet list, since the row/column structure is the point. Omit the section
entirely if the source has no tabular data. If there's more than one table,
give each its own short heading.>
<Table name 1>

| <column 1> | <column 2> | ... |
|---|---|---|
| <value> | <value> | ... |

## Config Patterns
```ios-xe
<canonical minimal config block — real IOS-XE syntax>
```

## Design Baseline
<Best-practice defaults for this topic, drawn from Cisco validated designs
(Design Zone), Cisco hardening/config guides, or the ENCOR OCG. Every row
cites a real, named source — no source, no row. A deviation from this table
is a QUESTION ("is this intentional here?"), never automatically a finding:
real networks deviate from best practice for good and bad reasons.>

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| <practice> | <what it protects> | <known-good deviation cases> | <named doc> |

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ...` | <expected output indicator> |
| `show ...` | <expected output indicator> |

## Intent Questions
<The 2-4 questions that define "what is this topic supposed to be doing
here?" on a given network. Answering them is step 0 of troubleshooting —
intent vs. observed comes before any show command.>
- <question 1>
- <question 2>

## Troubleshooting Checklist
0. State intent vs. observed: answer the Intent Questions above for this
   network, then write the one-line symptom ("should ___, isn't ___") —
   before running any show command.
1. <most likely failure — check this first>
2. <next>
3. ...

## Common Pitfalls
- <thing that trips people up>
- <another>

## Exam Preparation Tasks
<Only include this section if the source material carries the OCG's end-of-chapter
"Exam Preparation Tasks" apparatus — a Key Topics table and/or a "Do I Know This
Already?" quiz. Omit whichever of the two subsections the source lacks.>

### Key topics coverage map
<Reproduce the chapter's "Key Topics for Chapter N" table verbatim (element type,
description, page) and add a fourth column pointing at where in THIS skill each key
topic is actually covered — by section name and, where useful, the specific bullet
or table.

The mapping is deliberately not 1:1. One bullet or table row in this skill can
satisfy several key topics at once, and one key topic can be split across Key
Concepts, a Reference Table, and Common Pitfalls. Say which it is instead of
forcing a clean pairing. If a key topic is thin or missing in this skill, write
"gap" in that column rather than papering over it — the map doubles as a coverage
audit.>

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| <List / Section / Figure / Table> | <description from the OCG table> | <page> | <section name in this skill, or "gap"> |

### "Do I Know This Already?" question analysis
<For every quiz question in the source, three things: what the question is *actually*
testing underneath its wording, the correct answer, and which pitfall the distractors
are built to expose.

Be specific about distractor design. The useful cases are:
- a distractor naming a real technology that belongs somewhere else (a genuine
  real-world confusion worth its own Common Pitfalls bullet);
- a distractor that is true of a *related* protocol but not this one;
- a distractor that is half-right — correct mechanism, wrong set of devices;
- an invented term that sounds plausible next to real ones.

A question that is pure recall with no trap should say so plainly — don't
manufacture a pitfall that isn't there. Where a question's trap is genuinely
important, it should also appear as a bullet in Common Pitfalls; note that
cross-reference here.>

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| <n> | <underlying concept, not the surface wording> | <letter(s) + short text> | <the trap, or "none — straight recall"> |
```

### Step 3 — Update the README roadmap

Read `~/.claude/plugins/cache/local/ccnp-encor/README.md` and update the
"Roadmap — ENCOR exam domains" section so it stays in sync with the skill
catalog:
- Find the ENCOR domain the new topic belongs to (Architecture,
  Virtualization, Infrastructure, Network Assurance, Security, Automation).
- Add a new checked item `- [x] [<Topic Name>](1.0.0/skills/$ARGUMENTS/SKILL.md) — <short description>`
  under that domain's heading, replacing a `- [ ] Not yet started` placeholder
  if that's the first entry for that domain.
- Do not remove or reorder existing entries — only add the new one.

This step is required for every skill, not optional — a skill file without a
matching roadmap entry is an incomplete run.

### Step 4 — Commit and push

This skill catalog is a git repo (see `~/.claude/plugins/cache/local/ccnp-encor/CLAUDE.md`).
Every new or updated skill (and the README update from Step 3) must be pushed
immediately — do not leave it uncommitted. Run:

```bash
cd ~/.claude/plugins/cache/local/ccnp-encor
git add -A
git commit -m "Add $ARGUMENTS skill"
git push
```

This is pre-authorized for this repo — no need to ask before pushing.

### Step 5 — Confirm

After writing and pushing, tell the user:
- The full path where the skill was saved
- That it was committed and pushed to the `ccnp-encor-skills` repo
- That the README roadmap was updated to reflect the new topic
- That it will be available after starting a new Claude Code session
- Suggest the next topic to capture based on what they just covered

## Notes
- Use real IOS-XE syntax in config blocks — no pseudocode
- Troubleshooting checklist should be ordered: step 0 (intent vs. observed) → Layer 1 → Layer 2 → Layer 3 → config errors → software bugs
- Design Baseline rows follow "no source, no row": every practice must trace
  to a named, real document (a Design Zone validated design, the Cisco IOS
  hardening guide, the ENCOR OCG) — never written from memory. If no source
  is at hand, leave the row out rather than inventing one.
- A Design Baseline deviation is a question for the network's operator, never
  automatically a finding — real networks deviate from best practice for good
  and bad reasons (credit: Stephan M. and Phil Lafontaine, see README)
- Keep descriptions keyword-rich so the skill auto-triggers correctly
- Always commit and push after writing a skill file — see Step 4
- Always update the README roadmap before committing — see Step 3
- If the source notes contain an explicit step-by-step procedure (a
  handshake, an election process, an ordered convergence sequence), keep it
  as a numbered list in the Procedure section instead of flattening it into
  a Key Concepts bullet — the ordering is often the part worth remembering
- If the source notes contain a reference table (cost tables, timer
  defaults, port state/role matrices, comparison tables), reproduce it as a
  real markdown table in the Reference Tables section instead of flattening
  it into prose bullets — the row/column structure carries information that
  prose loses
- If the source material includes the OCG's end-of-chapter apparatus, the Exam
  Preparation Tasks section is required, not optional. The Key Topics table is
  Cisco telling you exactly what it considers testable — reproduce it and map
  every row to where this skill covers it. A row you cannot map is a real gap
  in the skill; mark it "gap" and say so in the confirmation rather than
  quietly dropping the row
- The quiz analysis is not an answer key. The value is in naming what each
  question is testing underneath its wording and which misconception the
  distractors were designed to catch. Distractors that name a real technology
  belonging to a different solution (e.g. an EVPN/MP-BGP option offered as an
  SD-Access control plane) are the highest-value ones — promote those to a
  Common Pitfalls bullet as well
- Never invent a trap that isn't there. A pure-recall question should be
  labeled as such
