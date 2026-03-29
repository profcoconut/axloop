---
name: axloop
preamble-tier: 2
description: |
  Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, and DevOps
  via SendMessage. Event-driven, no cron.
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

## Two Loops

**Project Loop — runs forever, spans sessions**

Fix all failures. Improve forever. Zero regressions.
Survives session deaths. State lives in worktree/ files. CE compound docs accumulate knowledge.

**Session Loop — one Manager per session**

React to all ready events. Spawn all work in parallel. Never idle.
Manager wakes on SendMessage. Dies when session ends. Next Manager resumes the Project Loop.

## How It Works

1. Start: `/axloop <doc>` — Manager boots, reads state files, resumes Project Loop
2. Manager watches for events: PR opened, build passed, QA approved, smoke failed, etc.
3. On event: spawn DevOps or QA immediately, in parallel
4. Dev writes fix → opens PR → Manager sees it → DevOps builds → QA tests → DevOps deploys → smoke test
5. If smoke fails: rollback + re-assign immediately
6. If session dies: next Manager reads loop.log + cycle-wip, resumes from where it left off
7. If all idle: run tests to find new failures

## Skills (automatic)

| Skill | When |
|-------|------|
| `/investigate` | Dev investigating a failing test |
| `/simplify` | Dev + DevOps on code quality |
| `/review` | Dev + QA before PR |
| `/qa` | QA running functional tests |
| `/browse` | DevOps doing smoke test |
| `/canary` | DevOps post-deploy monitoring |
| `/design-review` | Designer reviewing UI specs |
| `/retro` | Manager after every cycle |
| `/ce:plan` | Manager starting a new feature |
| `/ce:review` | Manager or QA on complex PRs (6-15 parallel reviewers) |
| `/ce:compound` | Manager after every cycle — writes to docs/solutions/ |

## State Files (all in worktree/)

- `.backlog.json` — bug backlog
- `.loop-status.json` — live 1-sentence status per agent
- `.loop.log` — event history (survives session death)
- `.cycle-wip` — interrupted work (resume, don't restart)
- `.deploy-state.json` — last SHA + health
- `.approved/<item>` / `.rejected/<item>` — QA decisions
- `.proposals/<dev>/<item>.json` — dev fix proposals (PR opened)
- `.crash-report/<item>.json` — crash/panic output
- `.rollback/<item>` — rollback reason

## Laws

**Project Loop:**
1. Never merge without smoke test passing
2. Never leave repo broken
3. Write loop.log + cycle-wip before session ends
4. Write CE compound docs to docs/solutions/ after every cycle

**Session Loop:**
5. Spawn all work in parallel — never wait
6. Reconcile all state files on startup

## Session Resume

Session died? No problem. New Manager reads loop.log, backlog, cycle-wip, open PRs — picks up where it left off.

## Modes

`rapid` — parallel devs, skip /canary
`quality` — one dev at a time, all skills mandatory
`auto` — parallel devs, /canary every 10 cycles

## Project Setup

Configure in SKILL.md:
- Test command: `cargo test`, `npm test`, etc.
- Build command: `cargo build`, etc.
- Binary start: `cargo run --`, etc.
- Smoke test: run binary briefly, check no panic
- Branch: `feat/engine`, `main`, etc.
- Memory path: where CE docs go

Crash = highest priority. Parse test output for `panicked at`, `panic:`, `Segmentation fault`.

Stop loop: write `stop` to `worktree/.project-loop-state`
