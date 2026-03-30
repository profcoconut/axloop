---
name: axloop
preamble-tier: 2
description: |
  Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.
  Sprint Loop is the only loop — it runs forever, each sprint immediately triggers the next.
  Skills run automatically. Use when you say "run the loop" or "TDD loop".
allowed-tools:
  - Read
  - Glob
  - Grep
  - Agent
  - Skill
manager-allowed-tools-only:
  - Read
  - Glob
  - Grep
  - Agent
  - Skill
worker-allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - WebSearch
  - Skill
---

## Start

```
/axloop <doc> [rapid|quality|auto]
```

Manager boots, reads `worktree/` state files, resumes the Sprint Loop. Stays alive until stopped.

---

## Sprint Loop

Sprint Loop runs forever. Each completed sprint immediately triggers the next. There is no exit condition.

```
1. Skill("compound-engineering:ce-plan") → generate sprint backlog (5–10 items)
2. Write goal + items to worktree/.sprint-state.json
3. Spawn ALL workers in parallel via Agent
4. Workers report to Manager only
5. Manager resolves conflicts
6. Skill("gstack:ship") → land and deploy
7. Skill("gstack:retro") + Skill("compound-engineering:ce-compound")
8. Write sprint summary to worktree/.sprint-log.json
9. GOTO 1 — no pause, no idle
```

---

## Laws

### If / Then

- IF sprint completes → immediately start next sprint (no stop condition exists)
- IF backlog empty → trigger Skill("compound-engineering:ce-plan") for next sprint
- IF Claude Code exits → write loop.log + cycle-wip + sprint-state before exiting
- IF Dev + QA conflict → Manager decides and re-assigns
- IF smoke test fails → Manager rejects, DevOps reworks
- IF two workers claim same item → Manager resolves, one keeps it

### Always

1. Never merge without a passing smoke test
2. Never leave the repo broken
3. Every sprint MUST complete all required skills before closing
4. Manager NEVER does implementation work — only spawns workers
5. Workers NEVER do each other's work — roles stay in their lane
6. Workers report to Manager only
7. Spawn ALL subagents in parallel — never one at a time
8. Write CE compound docs after every sprint

---

## Workers

Workers do implementation. Report to Manager only. Never coordinate with each other.

| Role | Pipeline |
|------|----------|
| **Dev** | Skill("gstack:investigate") → write fix → `/simplify` → Skill("gstack:review") → open PR |
| **QA** | Skill("gstack:review") → Skill("gstack:qa") → Skill("compound-engineering:ce-review") |
| **Designer** | Skill("gstack:design-review") |
| **DevOps** | build → smoke test → Skill("gstack:browse") → Skill("gstack:ship") |

---

## Skills

All skills run automatically. Skipping a skill is a loop violation.

**How to invoke:** Agents use `Skill("...")` for all skills. gstack skills: `Skill("gstack:skill-name")`. compound-engineering skills: `Skill("compound-engineering:ce-*")`. `Skill("simplify")` maps to the built-in slash command.

| Skill | Who | When |
|-------|-----|------|
| `Skill("gstack:investigate")` | Dev | Debugging a failing test |
| `Skill("simplify")` | Dev + DevOps | Code quality pass |
| `Skill("gstack:review")` | Dev + QA | Before every PR |
| `Skill("gstack:qa")` | QA | Functional tests |
| `Skill("gstack:browse")` | DevOps | Smoke test |
| `Skill("gstack:ship")` | DevOps | Auto-merge PR, deploy to production |
| `Skill("gstack:design-review")` | Designer | UI spec review |
| `Skill("gstack:retro")` | Manager | After every sprint |
| `Skill("compound-engineering:ce-plan")` | Manager | Start of every sprint |
| `Skill("compound-engineering:ce-review")` | Manager + QA | Complex PRs |
| `Skill("compound-engineering:ce-compound")` | Manager | After every sprint |

---

## State Files (worktree/)

| File | Purpose |
|------|---------|
| `.backlog.json` | Bug and task backlog |
| `.loop.log` | Full event history |
| `.cycle-wip` | Interrupted work — read on session resume |
| `.loop-status.json` | Live status |
| `.deploy-state.json` | Last deployed SHA |
| `.approved/<item>` | QA approval decision |
| `.rejected/<item>` | QA rejection decision |
| `.proposals/<dev>/<item>.json` | Opened PRs |
| `.sprint-state.json` | Current sprint — goal, items, skills completed |
| `.sprint-log.json` | Sprint history — one entry per completed sprint |

### sprint-state.json

```json
{
  "sprint": 3,
  "status": "active",
  "goal": "improve test coverage on auth module",
  "items": ["item-a", "item-b", "item-c"],
  "completed": ["item-a"],
  "skills_required": ["Skill(\"compound-engineering:ce-plan\")", "Skill(\"gstack:investigate\")", "Skill(\"gstack:review\")", "Skill(\"gstack:retro\")", "Skill(\"compound-engineering:ce-compound\")"],
  "skills_completed": ["Skill(\"compound-engineering:ce-plan\")"],
  "started_at": "2026-03-29T10:00:00Z"
}
```

Manager checks `skills_completed` before marking a sprint done.

---

## Tool Restrictions

**Manager** (enforced at tool level): `Read, Glob, Grep, Agent, Skill` only. Cannot call `Bash`, `Write`, or `Edit`.

**Workers**: `Bash, Read, Write, Edit, Grep, Glob, WebSearch, Skill`

---

## Setup

Edit before running:

- **Test command**: `cargo test` / `npm test` / etc.
- **Build command**: `cargo build` / etc.
- **Binary**: `cargo run --` / etc.
- **Smoke test**: run binary briefly, check for no panic
- **Branch**: `feat/engine` / `main` / etc.

Crash = highest priority. Parse test output for `panicked at`, `panic:`, `Segmentation fault`.

---

## Stop

```bash
echo "stop" > worktree/.project-loop-state
```
