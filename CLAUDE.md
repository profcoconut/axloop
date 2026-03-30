# Axloop Skill Writing Guide

This guide governs how SKILL.md is written and maintained.

---

## SKILL.md Structure

```
---
(name, description, allowed-tools, worker-allowed-tools)
---

## Start
## Sprint Loop
## Laws
## Workers
## Skills
## State Files
## Stop
```

**No Setup section.** Project-specific config belongs in the project's own `.axloop` or `SKILL.md`, not here.

---

## Skill Invocation Format

**All skills use `Skill("...")` uniformly.** No slash commands.

| Type | Format | Examples |
|------|--------|----------|
| gstack skills | `Skill("skill-name")` | `Skill("investigate")`, `Skill("qa")`, `Skill("ship")` |
| compound-engineering skills | `Skill("compound-engineering:ce-*")` | `Skill("compound-engineering:ce-plan")` |
| built-in slash commands | `Skill("skill-name")` | `Skill("simplify")` (maps to `/simplify`) |

**Never use `/skill-name` format in SKILL.md.** That format is for human CLI input, not agent invocation.

---

## Laws Format

Laws are split into **If / Then** and **Always**.

**If / Then** — conditional triggers:
- Event → Consequence
- Written as: `IF event → consequence`

**Always** — invariant rules:
- Numbered list, starting at 1
- Written as imperatives: "Never...", "Always...", "MUST..."

Tool restrictions belong in Always as numbered items (e.g., "9. Manager: Read, Glob, Grep, Agent, Skill only").

---

## Sprint Loop Format

- Numbered steps 0–9
- Step 0: read state files if not in memory (resumption)
- Steps use `Skill("...")` format
- Ends with `GOTO 0` (loop back, no pause)

---

## Skills Table Format

```
| Skill | Who | When |
|-------|-----|------|
| `Skill("...")` | Role | Description |
```

- All skills in `Skill("...")` format
- No `/slash` in table cells
- Who = role(s), When = trigger

---

## State Files Section

- Table of files and purposes
- JSON schema example for `.sprint-state.json`
- Schema must use `Skill("...")` format for `skills_required` and `skills_completed`

---

## Formatting Rules

- **One loop only.** Sprint Loop is the only loop. No "Project Loop" or "Session Loop" sections.
- **No session boundary.** Session end is a Claude Code event, not an axloop concept.
- **No Setup section.** Project-specific config lives elsewhere.
- **Consistent Skill() format everywhere.** Check for `gstack:` prefix, slash commands, mixed formats.
- **Laws after Loop.** Sprint Loop → Laws → Workers → Skills → State Files → Stop.
- Use backticks for tool names, skill names, file paths.
- Use `Skill("...")` for all skill invocations in tables, steps, and laws.
- Keep it lean — delete redundant sections when consolidating.

---

## Core Principles

These principles from README.md govern all SKILL.md design decisions:

1. **Loops, not prompts** — `GOTO 0` is always the final step. No exit condition.
2. **Skills are law** — Skipping a skill is a loop violation. `skills_completed` must match `skills_required`.
3. **Strict role separation** — Dev codes, QA tests, DevOps ships. Manager coordinates only (Read, Glob, Grep, Agent, Skill).
4. **Parallel by default** — All workers spawn simultaneously. No serialization.
5. **Hard safety gate** — Nothing merges without a passing smoke test.
6. **Crash-proof state** — All state in `worktree/` files, not memory.
