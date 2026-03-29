---
name: axloop
preamble-tier: 2
version: 1.8.0
description: |
  Autonomous AI-driven DevOps team. Manager coordinates developers, designer, QA, and DevOps
  via SendMessage. Event-driven, no cron, no polling.
  Two loops work together: the Project Loop (forever, survives sessions) and the Session Loop
  (one Manager agent per session, reacts to events).
  The 10 Laws are the constitution that makes both loops work.
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

## The Two Loops

**Project Loop — goal: Fix all failures. Improve forever. Zero regressions.**
The long-running loop that spans multiple sessions. Survives session deaths. Compounds knowledge. Self-heals. Runs on the project codebase.

**Session Loop — goal: Process all ready events. Spawn all parallel work. Never idle.**
One Manager agent per session. Wakes on SendMessage. Processes events. Spawns agents. Dies when session ends. Next session's Manager resumes the Project Loop.

The Project Loop never stops. Sessions start and end. The Manager (Session Loop) is how the Project Loop executes.

## 10 Laws — The Constitution

These make both loops work. Violating any breaks the loop.

**Project Loop laws (survive sessions):**
1. **Never push to main/feat directly.** All code through PR. Always.
2. **Never merge without smoke test passing.** Deploy only after DevOps confirms.
3. **Never leave the repo broken.** Fix must pass tests before commit.
4. **Never skip rollback on smoke failure.** Rollback immediately.
5. **Never write to main/feat branch.** Feature branches only.
6. **Always write loop.log.** Every state change appended. Project Loop history.
7. **Always write cycle-wip before session ends.** Next Manager resumes, not restarts.

**Session Loop laws (per session):**
8. **Always assign when devs are idle and backlog has items.** Never idle workers.
9. **Always spawn in parallel.** If events are ready, spawn all at once.
10. **Always reconcile on startup.** Read all state files before acting.

## Invocation

```
/axloop <path-to-doc> [rapid|quality|auto]
```

**First invocation:** starts the Manager (Session Loop) and resumes the Project Loop.

- `path-to-doc`: Feature spec, brainstorm, or plan
- `mode` (default: auto): `rapid` | `quality` | `auto`

## Session Loop — How One Manager Works

**One Manager per session.** Woken ONLY by SendMessage or state file writes. No cron, no polling, never exits after spawning.

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

**Idle sweep:** Nothing active? Run test command, parse failures, write backlog, assign devs. Session Loop keeps the Project Loop alive.

## Project Loop — The Forever Loop

**Goal: Fix all failures. Improve forever. Zero regressions.**

The Project Loop spans sessions. State files are its memory. CE compound docs are its knowledge base.

**What makes it forever:**
- State in files survives session deaths (`.backlog.json`, `.loop.log`, `.deploy-state.json`)
- `cycle-wip` saves interrupted work — next Manager resumes
- CE compound docs (`docs/solutions/`) accumulate project knowledge across sessions
- No human needed to restart — next session's Manager picks up automatically

**Stops when:** human writes `stop` to `worktree/.project-loop-state` — not `.loop-mode` (that's session-only)

## Team & Skills

**Roles:** Developers, Designer, QA, DevOps — all via SendMessage.

**All skills the Manager uses (automatic):**

| Skill | What it does | Who uses it |
|-------|-------------|-------------|
| `/investigate` | Root cause analysis | Developer |
| `/simplify` | Code quality + efficiency | Developer, DevOps |
| `/review` | Pre-PR review | Developer, QA |
| `/qa` | Functional test | QA |
| `/browse` | Smoke test — verify app loads | DevOps |
| `/canary` | Post-deploy monitoring | DevOps |
| `/design-review` | UI/UX spec review | Designer |
| `/retro` | Cycle retrospective | Manager |
| `/ce:plan` | Deep planning — research agents + historical learnings | Manager |
| `/ce:review` | 6-15 specialized parallel reviewers | Manager, QA |
| `/ce:compound` | Build searchable knowledge base from session | Manager |

**Dev pipeline:** Investigate → write fix → /simplify → /review → gh pr create → Manager

**QA pipeline:** /review → /qa → /ce:review (complex PRs) → Manager

**DevOps pipeline:** Build → binary smoke test (verify starts) → /browse → /canary → Manager

**Designer pipeline:** Write spec → /design-review → Manager

## Shared Infrastructure

State files are the Project Loop's memory. All auto-created in `worktree/`.

```
.backlog.json              — bug backlog (crash, panic, assertion, compilation, flaky)
.feature-backlog.json      — feature backlog
.project-loop-state        — Project Loop state: running | paused | stopped
.loop-status.json          — live 1-sentence status per agent
.loop.log                  — append-only event log (survives session death)
.cycle-wip                — interrupted cycle state (resumed, not restarted)
.loop-mode                — current session mode (rapid/quality/auto)
.deploy-state.json         — last SHA, health, regressions
.proposals/<dev_id>/<item_id>.json — fix proposal
.designs/<feature_id>.md  — designer specs
.approved/<item_id>       — QA/DevOps approval
.rejected/<item_id>       — QA/DevOps rejection
.crash-report/<item_id>.json — crash/panic output + backtrace
.rollback/<item_id>      — rollback reason + from_sha
```

## Event → Action

```
cargo test finds failures         → write backlog.json, assign to dev
cargo test panics (severity: crash) → crash item, assign immediately
PR opened (.proposals/*)          → DevOps build check
Build passes                      → QA regression + design check
QA approved                       → DevOps merge + deploy
QA rejected                       → re-assign to dev
Smoke fails (.rollback/* written) → git revert, re-assign immediately
Smoke passes                      → /ce:compound, close backlog item
All idle + backlog empty          → run test command, find failures
10 cycles done                    → /ce:simplify quality sweep
```

**Priority:** Crash → Rollback → Rejection → Approval → Build → PR → Unassigned → Idle

## Session Resume

**No work lost when a session dies.** Project Loop state survives.

**On every /axloop invocation (Session Loop starts):**
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
11. Resume Project Loop from where it left off
```

## Modes

| Mode | Developers | Designer | QA | Deploy | Skills |
|------|-----------|---------|-----|--------|--------|
| rapid | parallel | optional | specific test | always | skip /canary |
| quality | 1 at a time | required | full suite | always after QA | all mandatory |
| auto | parallel | optional | quality if regressions | always | /canary every 10 cycles |

## Memory & Knowledge

**CE compound docs** (primary): `/ce:compound` writes to `docs/solutions/` after every cycle — context, solution, prevention. Future `/ce:plan` calls find these via learnings-researcher. **Project Loop knowledge — survives forever.**

**gstack memory** (append-only): Write to `{memory-dir}/YYYYMMDD-HHMMSS-<slug>.md` after each fix. Same bug recurs → new entry with `attempt: N+1`.

## Visual Verification

```bash
cat worktree/.project-loop-state  # running | paused | stopped
cat worktree/.loop-status.json     # live 1-sentence per agent
cat worktree/.loop.log           # event history
cat worktree/.backlog.json       # current backlog
gh pr list                      # open PRs
```

**`.loop-status.json`:**
```json
{
  "project_loop": "running",
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
- **Smoke test**: Brief binary run: `<start cmd> --headless 2>&1 | head -20`
- **Branch**: Target for PRs (`feat/engine`, `main`, etc.)

**Crash detection:** Parse test output for `panicked at`, `Segmentation fault`, `panic:`. Write as `severity: crash` — highest priority.

**Skill dev note:** This skill develops itself. Loop operates on target project codebase. Skill files stay in `~/Documents/skills/Axloop/`.
