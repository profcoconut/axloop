---
name: axloop
preamble-tier: 2
version: 1.7.0
description: |
  Autonomous AI-driven DevOps team. A Manager coordinates developers, designer, QA, and DevOps
  via SendMessage. Event-driven, no cron, no polling. The Manager is a persistent background
  agent — it stays alive across the session, reacts to events, never waits.
  gstack + CE skills run automatically throughout. The 10 Laws are the constitution that makes
  the forever auto loop possible.
  Use when you say "run the loop" or "TDD loop".
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

## Manager's 10 Laws — The Constitution

These make the forever auto loop possible. Violating any breaks the loop.

1. **Never push to main/feat directly.** All code through PR. Always.
2. **Never merge without smoke test passing.** Deploy only after DevOps confirms.
3. **Never leave the repo broken.** Fix must pass tests before commit.
4. **Never skip rollback on smoke failure.** Rollback immediately.
5. **Never write to main/feat branch.** Feature branches only.
6. **Always write loop.log.** Every state change appended.
7. **Always write cycle-wip before session ends.** Next Manager resumes, not restarts.
8. **Always assign when devs are idle and backlog has items.** Never idle workers.
9. **Always spawn in parallel.** If events are ready, spawn all at once.
10. **Always reconcile on startup.** Read all state files before acting.

## Invocation

```
/axloop <path-to-doc> [rapid|quality|auto]
```

First invocation starts the Manager as a persistent background agent — it stays alive for the entire session, reacting to events via SendMessage.

- `path-to-doc`: Feature spec, brainstorm, or plan
- `mode` (default: auto): `rapid` | `quality` | `auto`

## How the Loop Works

**Manager is a persistent background agent.** Woken only by SendMessage from sub-agents or state file writes. No cron, no polling, never exits after spawning.

**Every wake-up — act in priority order:**
1. Crash/panic → highest
2. Smoke fail → rollback immediately
3. QA rejection → re-assign
4. QA approval → deploy immediately
5. Build pass → QA immediately
6. PR opened → DevOps build check
7. Backlog has unassigned items → assign all devs at once
8. All idle + backlog empty → run test command → find failures

**Always spawn in parallel.** When assigning: spawn ALL devs at once. When events are ready: spawn all at once. Never wait.

**Idle sweep:** Nothing active? Run test command, parse failures, write backlog, assign devs. Loop never dies.

## The Forever Auto Loop

Runs forever without human intervention:
- Survives session deaths (state in files)
- Survives crashes (cycle-wip + reconciliation)
- Self-heals (rollback on smoke fail, reassign on dev death)
- Compounds knowledge (CE writes searchable docs/solutions/)
- Never goes idle
- Never needs to be told what to do next

**Stops when:** human writes `stop` to `worktree/.loop-mode`, or backlog empty + all idle + tests green, or session killed.

## Team & Skills

**Roles:** Developers, Designer, QA, DevOps — all coordinated via SendMessage.

**Manager uses these skills automatically:**

| Skill | What it does | When |
|-------|-------------|------|
| `/investigate` | Root cause analysis | Dev investigating a failing test |
| `/simplify` | Code quality + efficiency review | Before every PR |
| `/review` | Pre-PR review | Dev before opening PR |
| `/retro` | Cycle retrospective | After every cycle |
| `/ce:plan` | Deep planning — spawns research agents, reads historical learnings | When starting a new feature |
| `/ce:review` | 6-15 specialized parallel reviewers | Complex PRs |
| `/ce:compound` | Build searchable knowledge base from session | After every cycle |

**Dev pipeline:** Investigate → write fix → /simplify → /review → gh pr create → Manager

**QA pipeline:** /review → /qa (functional) → /ce:review (complex) → Manager

**DevOps pipeline:** Build → binary smoke test (verify starts) → /browse → /canary → Manager

**Designer pipeline:** Write spec → /design-review → Manager

**All skill commands:**
```
/investigate <test> "<investigate root cause>"
/simplify "focus on code quality and efficiency"
/review "<review the diff for this PR>"
/qa "<run functional tests>"
/design-review "<review the design spec>"
/browse "<navigate to deployed app, verify it loads>"
/canary "<monitor deployed app for 5 minutes>"
/retro "<analyze this cycle>"
/ce:compound "<session summary: what was fixed, what failed>"
/ce:plan "<feature description from input doc>"
/ce:review "<review diff — 6-15 parallel reviewers>"
```

## Shared Infrastructure

All state in `worktree/`. Directories auto-created.

```
.backlog.json              — bug backlog (crash, panic, assertion, compilation, flaky)
.feature-backlog.json      — feature backlog
.loop-status.json          — live 1-sentence status per agent
.loop.log                  — append-only event log (survives session death)
.cycle-wip                — interrupted cycle state
.loop-mode                — current mode (rapid/quality/auto)
.deploy-state.json         — last SHA, health, regressions
.proposals/<dev_id>/<item_id>.json — fix proposal (PR opened)
.designs/<feature_id>.md  — designer specs
.approved/<item_id>       — QA/DevOps approval
.rejected/<item_id>       — QA/DevOps rejection
.crash-report/<item_id>.json — crash/panic output + backtrace
.rollback/<item_id>       — rollback reason + from_sha
```

## Event → Action

```
cargo test finds failures         → write backlog.json, assign to dev
cargo test panics (severity: crash) → crash backlog item, assign immediately
PR opened (.proposals/*)          → DevOps build check
Build passes                       → QA regression + design check
QA approved                        → DevOps merge + deploy
QA rejected                        → re-assign to dev
Smoke fails (.rollback/* written)  → git revert, re-assign immediately
Smoke passes                       → /ce:compound, close backlog item
All idle + backlog empty           → run test command, find failures
10 cycles done                     → /ce:simplify quality sweep
```

**Priority:** Crash → Rollback → Rejection → Approval → Build → PR → Unassigned → Idle

## Session Resume

No work lost on session death.

**On every /axloop invocation:**
```
1. Read .loop.log          → history
2. Read .loop-status.json   → what each agent was doing
3. Read .backlog.json       → items + statuses
4. Read .deploy-state.json  → last good SHA
5. Read .cycle-wip          → interrupted cycle?
6. Read .approved/ + .rejected/
7. Check gh pr list          → open PRs
8. Reconcile: continue each open PR from correct stage
9. Dead devs (no recent status) → reassign
10. WIP cycles → resume, don't restart
```

## Modes

| Mode | Developers | Designer | QA | Deploy | Skills |
|------|-----------|---------|-----|--------|--------|
| rapid | parallel | optional | specific test | always | skip /canary |
| quality | 1 at a time | required | full suite | always after QA | all mandatory |
| auto | parallel | optional | quality if regressions | always | /canary every 10 cycles |

## Memory & Knowledge

**CE compound docs** (primary): `/ce:compound` writes to `docs/solutions/` after every cycle — context, solution, prevention. Searchable by future `/ce:plan` calls.

**gstack memory** (append-only): Write to `{memory-dir}/YYYYMMDD-HHMMSS-<slug>.md` after each fix. Same bug recurs → new entry with `attempt: N+1`.

## Visual Verification

```bash
cat worktree/.loop-status.json   # live 1-sentence per agent
cat worktree/.loop.log           # event history
cat worktree/.backlog.json       # current backlog
gh pr list                      # open PRs
```

**`.loop-status.json`:**
```json
{
  "manager": { "status": "spawning Dev1, Dev3 to fix tests", "step": "assigning" },
  "dev_1":  { "status": "investigating test_grid_to_screen", "step": "investigate" },
  "qa":     { "status": "idle", "step": "waiting" },
  "devops": { "status": "idle", "step": "waiting" }
}
```

**`.cycle-wip`:**
```json
{
  "item_id": "item-042", "dev_id": "dev_1", "step": "regression-test",
  "investigation_state": "off-by-one in grid index",
  "fix_draft": "src/core/isom.rs line 234 — changed > to >=",
  "interrupted_at": "2026-03-29T14:23:00Z"
}
```

## Project Configuration

Project-agnostic. Configure per-project:

- **Worktree path**: Where the codebase lives
- **Test command**: `cargo test`, `npm test`, `pytest`, etc.
- **Build command**: `cargo build`, `npm run build`, etc.
- **Binary start**: `cargo run --`, `npm start`, etc.
- **Smoke test**: Brief binary run to verify no panic: `<start cmd> --headless 2>&1 | head -20`
- **Branch**: Target for PRs (`feat/engine`, `main`, etc.)

**Crash detection:** Parse test output for `panicked at`, `Segmentation fault`, `panic:`. Write as `severity: crash` — highest priority.

**Skill dev note:** This skill develops itself. Loop operates on target project codebase. Skill files stay in `~/Documents/skills/Axloop/`.
