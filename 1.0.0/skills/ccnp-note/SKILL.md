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

## Verification Commands
| Command | What to look for |
|---------|-----------------|
| `show ...` | <expected output indicator> |
| `show ...` | <expected output indicator> |

## Troubleshooting Checklist
1. <most likely failure — check this first>
2. <next>
3. ...

## Common Pitfalls
- <thing that trips people up>
- <another>
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
- Troubleshooting checklist should be ordered: Layer 1 → Layer 2 → Layer 3 → config errors → software bugs
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
