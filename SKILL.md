---
name: axloop
preamble-tier: 2
description: |
  Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.
  Two loops: Project Loop (forever) + Session Loop (one Manager per session).
  Skills run automatically. Use when you say "run the loop" or "TDD loop".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Agent
  - WebSearch
  - Skill
---

## Start

```
/axloop <doc> [rapid|quality|auto]
```

Manager boots, reads state files, resumes the Project Loop. Stays alive for the session.

## Two Loops

**Project Loop** — runs forever, survives sessions. Fix all failures. Improve forever. State in worktree/ files.

**Session Loop** — one Manager per session. Reacts to events. Spawns all work in parallel. Never idles.

## What the Manager Does

The Manager is a **persistent background agent**. It does NOT do work itself — only spawns workers and resolves conflicts.

```
LOOP FOREVER:
1. ALWAYS spawn subagents in parallel — never one at a time
2. Workers validate and report to Manager (not to each other)
3. Manager reads state files, decides what to do next
4. If workers disagree — Manager resolves the conflict
5. If session ends → next Manager reads loop.log + cycle-wip, resumes where it left off
```

**Workers validate and report to Manager. Manager handles conflicts.**

**Conflict examples:**
- Dev says fix works, QA says it broke something else → Manager decides who is right and what to do
- DevOps says build passes, smoke test says binary panics → Manager picks the smoke test result, re-assigns
- Two devs assign to same item → Manager resolves, one keeps it

**Key rule: ALWAYS use subagents in parallel.** Manager never does work itself — only spawns workers.

## What Workers Do

Workers validate and report to Manager. Manager never asks a worker to report to another worker.

**Dev**: investigate → write fix → /simplify → /review → open PR → **report to Manager**

**QA**: /review → /qa → /ce:review → **report approved/rejected to Manager**

**DevOps**: build → smoke test → /browse → /canary → **report pass/fail to Manager**

## Skills

All run automatically. No skipping.

| Skill | When |
|-------|------|
| `/investigate` | Dev debugging a failing test |
| `/simplify` | Dev + DevOps on code quality |
| `/review` | Dev + QA before PR |
| `/qa` | QA functional tests |
| `/browse` | DevOps smoke test |
| `/canary` | DevOps post-deploy monitoring |
| `/design-review` | Designer on UI specs |
| `/retro` | Manager after every cycle |
| `/ce:plan` | Manager on new features |
| `/ce:review` | Manager + QA on complex PRs (6-15 reviewers) |
| `/ce:compound` | Manager after every cycle — writes to docs/solutions/ |

## State Files (worktree/)

- `.backlog.json` — bug backlog
- `.loop.log` — event history
- `.cycle-wip` — interrupted work
- `.loop-status.json` — live status
- `.deploy-state.json` — last SHA
- `.approved/<item>` / `.rejected/<item>` — QA decisions
- `.proposals/<dev>/<item>.json` — PR opened

## Laws

**Project Loop** (survive sessions):
1. Never merge without smoke test
2. Never leave repo broken
3. Write loop.log + cycle-wip before session ends
4. Write CE compound docs after every cycle

**Session Loop**:
5. Manager never does work — always spawn workers
6. Workers never do each other's work — each role stays in their lane
7. Workers validate and report to Manager only — Manager handles conflicts
8. Spawn ALL subagents in parallel — never wait for one to finish

## Setup (per project)

Edit in SKILL.md:
- Test command: `cargo test`, `npm test`, etc.
- Build command: `cargo build`, etc.
- Binary: `cargo run --`, etc.
- Smoke test: run binary briefly, check no panic
- Branch: `feat/engine`, `main`, etc.

Crash = highest priority. Parse test output for `panicked at`, `panic:`, `Segmentation fault`.

## Stop

Write `stop` to `worktree/.project-loop-state`
