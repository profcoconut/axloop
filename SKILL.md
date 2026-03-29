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

The Manager is a **persistent background agent**. It wakes on SendMessage from workers.

```
LOOP FOREVER:
1. Read state files (.backlog, .loop-log, .approved/, .rejected/, gh pr list)
2. For every ready event: spawn DevOps or QA immediately, in parallel
3. Dev finishes → PR opened → Manager sees it → DevOps builds → QA tests → DevOps deploys
4. If smoke fails → rollback + re-assign immediately
5. If session ends → next Manager reads loop.log + cycle-wip, resumes where it left off
6. If all idle → run tests to find new failures
```

## What Workers Do

**Dev**: investigate → write fix → /simplify → /review → open PR → Manager

**QA**: /review → /qa → /ce:review (complex PRs) → Manager

**DevOps**: build → smoke test → /browse → /canary → Manager

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

**Session Loop** (one Manager efficient):
5. Spawn all in parallel
6. Reconcile on startup

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
