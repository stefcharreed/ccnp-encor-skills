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

### Step 3 — Commit and push

This skill catalog is a git repo (see `~/.claude/plugins/cache/local/ccnp-encor/CLAUDE.md`).
Every new or updated skill must be pushed immediately — do not leave it
uncommitted. Run:

```bash
cd ~/.claude/plugins/cache/local/ccnp-encor
git add -A
git commit -m "Add $ARGUMENTS skill"
git push
```

This is pre-authorized for this repo — no need to ask before pushing.

### Step 4 — Confirm

After writing and pushing, tell the user:
- The full path where the skill was saved
- That it was committed and pushed to the `ccnp-encor-skills` repo
- That it will be available after starting a new Claude Code session
- Suggest the next topic to capture based on what they just covered

## Notes
- Use real IOS-XE syntax in config blocks — no pseudocode
- Troubleshooting checklist should be ordered: Layer 1 → Layer 2 → Layer 3 → config errors → software bugs
- Keep descriptions keyword-rich so the skill auto-triggers correctly
- Always commit and push after writing a skill file — see Step 3
