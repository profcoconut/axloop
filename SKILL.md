---
name: axloop
preamble-tier: 2
version: 1.6.0
description: |
  Autonomous AI-driven DevOps team that runs like a real company.
  A manager coordinates: N developers (unlimited parallel PRs), a designer (specs),
  QA (regression + design verification), and DevOps (build gate + deploy + smoke test).
  Event-driven, no cron, no polling. Uses gstack + CE skills automatically throughout.
  gstack handles decision-making and real browser QA.
  CE handles planning, deep review, and compound knowledge accumulation.
  Built on TDD philosophy. The 10 Manager's Laws are the constitution that makes
  the forever auto loop possible — a loop that runs indefinitely with self-healing
  and zero human intervention.
  Use when you want to start the loop or when you say "run the loop" or "TDD loop".
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

## Preamble

This skill acts as the **Manager** of an autonomous DevOps team. It coordinates:

- **N Developers** — unlimited parallel PRs, TDD, never push to main directly
- **1 Designer** — UI/UX specs before implementation
- **1 QA** — regression tests + design verification gate
- **1 DevOps** — build gate, deploy pipeline, smoke tests, rollback

**Every agent uses skills automatically** as part of their workflow. Skills are not optional — they are the execution layer.

**gstack + CE dual-skill architecture:**
- **gstack**: `/plan-ceo-review`, `/plan-eng-review` (decisions), `/qa` (browser QA), `/investigate` (root cause), `/simplify` (quality)
- **CE**: `/ce:plan` (research agents + historical learnings), `/ce:review` (6-15 parallel reviewers), `/ce:compound` (knowledge base)

## Manager's 10 Laws — The Constitution

These laws enable the forever auto loop. Violating any breaks the self-healing loop.

1. **Never push to main/feat directly.** All code goes through PR. Always.
2. **Never merge without smoke test passing.** Deploy only after DevOps confirms.
3. **Never leave the repo in a broken state.** Every fix must pass tests before commit.
4. **Never skip rollback on smoke failure.** Rollback immediately — don't wait.
5. **Never write to main/feat branch.** Feature branches only.
6. **Always write loop.log.** Every state change is appended.
7. **Always write cycle-wip before session ends.** Next Manager must resume, not restart.
8. **Always assign when devs are idle and backlog has items.** Never let workers idle.
9. **Always spawn in parallel, never sequentially.** If events are ready, spawn all at once.
10. **Always reconcile on startup.** Read all state files before acting.

## The Forever Auto Loop

A loop that runs **forever** without human intervention.

**What "forever" means:**
- Survives session deaths (files persist all state)
- Survives crashes (cycle-wip + reconciliation recovers)
- Self-heals (rollback on smoke fail, reassign on dev death)
- Compounds knowledge (CE builds searchable docs/solutions/ over time)
- Never goes idle (devs always working or being assigned)
- Never needs to be told what to do next (Manager watches and reacts)

**The compound effect:** Each cycle fixes AND learns. Each session builds on all previous sessions. Bugs that took hours to debug in week 1 take minutes in week 4.

**When does it stop?**
- Human writes `stop` to `worktree/.loop-mode`
- Backlog empty, all devs idle, tests green
- Session killed externally

## Invocation

```
/axloop <path-to-doc> [rapid|quality|auto]
```

First invocation starts the Manager — it stays alive for the entire session.

**Arguments:**
- `path-to-doc` (required): Feature spec, brainstorm, or plan document
- `mode` (optional, default: auto): `rapid` (max velocity), `quality` (strict regression), `auto` (dynamic balance)

## Shared Infrastructure

Communication is via `SendMessage`. Files are only for state persistence and human visibility. **Directories are auto-created as needed.**

```
worktree/.backlog.json              — bug backlog (crash, panic, assertion, compilation, flaky)
worktree/.feature-backlog.json      — feature backlog (PM owns)
worktree/.loop-status.json          — live status: each agent's 1-sentence current state
worktree/.loop.log                  — append-only event log (critical for session resume)
worktree/.cycle-wip                — interrupted cycle state
worktree/.loop-mode                — current mode (rapid/quality/auto)
worktree/.deploy-state.json         — last SHA, health, regressions
worktree/.proposals/<dev_id>/<item_id>.json  — developer fix proposal
worktree/.designs/<feature_id>.md  — designer specs
worktree/.approved/<item_id>       — QA/DevOps approval marker
worktree/.rejected/<item_id>       — QA/DevOps rejection reason
worktree/.crash-report/<item_id>.json — crash/panic output + backtrace
worktree/.rollback/<item_id>      — rollback reason + from_sha
```

## How the Manager Loop Works

**The Manager is a persistent background agent.** It stays alive across the session. It is woken ONLY by SendMessage from sub-agents or state files being written. No cron, no polling.

**Every wake-up, the Manager checks and acts in this order:**
1. Crash/panic events → highest priority
2. Rollback events (smoke fail)
3. QA rejections
4. QA approvals → deploy immediately
5. Build passes → QA immediately
6. New PRs opened → DevOps build check immediately
7. Unassigned backlog items → assign to devs immediately
8. All idle + backlog empty → run test command to find new failures

**Always spawn in parallel.** When assigning: spawn ALL available devs at once. When events are ready: spawn all at once. Never wait for one to finish before spawning the next.

**Idle sweep:** If Manager has no pending work and all workers are idle, it runs the test command, parses failures, writes to backlog, assigns to devs. Loop stays alive.

## Session Resume

**No work is lost when a session dies.** Everything persists in files.

**Manager resume protocol (runs on every /axloop invocation):**
```
1. Read .loop.log          → understand full history
2. Read .loop-status.json   → what each agent was doing
3. Read .backlog.json       → items and their statuses
4. Read .deploy-state.json  → last known good SHA
5. Read .cycle-wip          → was a cycle interrupted?
6. Read .approved/ and .rejected/
7. Check gh pr list          → open PRs not yet merged
8. Reconcile: continue each open PR from its correct pipeline stage
9. For items "assigned" but dev dead → reassign
10. For WIP cycles → resume from WIP state, don't restart
```

**Reconciliation rules:**
- PR opened, no build check → DevOps build check
- Build passed, no QA → QA regression
- QA approved, not merged → DevOps merge+deploy
- Dev dead (no recent status) → reassign
- WIP exists → resume from WIP state

## Event → Action Mapping

```
EVENT: cargo test finds failures
ACTION: write to backlog.json with severity, assign to available dev

EVENT: cargo test panics (severity: crash)
ACTION: create crash backlog item, assign immediately (escalates above all)

EVENT: dev writes .proposals/<dev_id>/<item_id>.json (PR opened)
ACTION: spawn DevOps → build check

EVENT: DevOps writes .deploy-state.json with build_pass=true
ACTION: spawn QA → regression + design check

EVENT: QA writes .approved/<item_id>
ACTION: spawn DevOps → merge + deploy

EVENT: QA writes .rejected/<item_id>
ACTION: re-assign to dev with rejection notes

EVENT: smoke test fails (.rollback/* written)
ACTION: git revert → re-assign to dev immediately

EVENT: smoke test passes
ACTION: /ce:compound → close backlog item

EVENT: all idle + backlog empty
ACTION: run test command → find new failures → assign

EVENT: 10 cycles completed
ACTION: /ce:simplify — full codebase quality sweep
```

**Priority order:** Crash → Rollback → Rejection → Approval → Build pass → New PR → Unassigned → Idle sweep → Cleanup

## Team Roles & Skill → Phase Reference

**Developer:** Investigate → write fix → /simplify → /review → gh pr create → SendMessage → Manager

**QA:** /review → /qa (functional) → /ce:review (deep, for complex PRs) → SendMessage → Manager

**DevOps:** cargo build → binary smoke test (verify starts without panic) → /browse → /canary → SendMessage → Manager

**Designer:** Write spec → /design-review → SendMessage → Manager

**Manager:** /ce:plan (feature planning) → /retro + /ce:compound (after every cycle)

```
/investigate <test-name-or-error> "<investigate>"
/simplify "focus on code quality and efficiency"
/review "<review the diff for this PR>"
/qa "<run functional tests>"
/design-review "<review the design spec>"
/browse "<navigate to deployed app, verify it loads>"
/canary "<monitor deployed app for 5 minutes, report anomalies>"
/retro "<analyze this cycle: what failed, what succeeded>"
/ce:compound "<session summary: what was fixed, what failed>"
/ce:plan "<feature description from input doc>"
/ce:review "<review diff — triggers 6-15 specialized reviewers>"
```

## Modes

| Mode | Developers | Designer | QA | Deploy | Skill Usage |
|------|-----------|---------|-----|--------|-------------|
| rapid | N parallel | optional | specific test only | always | skip /canary; `/ce:compound` after cycle |
| quality | 1 at a time | required | full regression suite | always after QA | all skills mandatory |
| auto | N parallel | optional | quality if regressions exist | always | /canary + /simplify every 10 cycles |

## Memory & Knowledge Accumulation

**CE compound docs** (primary, searchable): After every cycle, `/ce:compound` writes to `docs/solutions/` — context, solution, prevention. Future `/ce:plan` calls find these via learnings-researcher.

**gstack memory entries** (append-only): After each fix, write to `{memory-dir}/YYYYMMDD-HHMMSS-<slug>.md`. Same bug recurs → new entry with `attempt: N+1`.

## Visual Verification

```bash
cat worktree/.loop-status.json   # live 1-sentence status per agent
cat worktree/.loop.log           # event history (survives session death)
cat worktree/.backlog.json       # current backlog
cat worktree/.deploy-state.json  # last SHA + health
cat worktree/.cycle-wip          # interrupted cycle (if any)
gh pr list                       # all open PRs
```

**`.loop-status.json` schema:**
```json
{
  "manager": { "status": "spawning Dev1, Dev3 to fix isom tests", "step": "assigning" },
  "dev_1":  { "status": "investigating isom::test_grid_to_screen — reading source", "step": "investigate" },
  "dev_3":  { "status": "writing regression test for grid panic", "step": "regression-test" },
  "qa":     { "status": "idle", "step": "waiting" },
  "devops": { "status": "idle", "step": "waiting" }
}
```

**`.cycle-wip` schema:**
```json
{
  "item_id": "item-042",
  "dev_id": "dev_1",
  "step": "regression-test",
  "investigation_state": "found root cause: off-by-one in grid index",
  "fix_draft": "src/core/isom.rs line 234 — changed > to >=",
  "interrupted_at": "2026-03-29T14:23:00Z"
}
```

**`.loop.log` schema:**
```json
{"ts": "2026-03-29T14:20:00Z", "event": "dev_1 assigned item-042", "actor": "manager"}
{"ts": "2026-03-29T14:22:00Z", "event": "dev_1 PR opened", "actor": "dev_1"}
{"ts": "2026-03-29T14:23:00Z", "event": "session ended", "actor": "system"}
```

## Project Configuration

axloop is project-agnostic. Configure per-project:

- **Worktree path**: Where the codebase lives
- **Test command**: `cargo test`, `npm test`, `pytest`, etc.
- **Build command**: `cargo build`, `npm run build`, etc.
- **Binary start command**: How to run the app (`cargo run --`, `npm start`, etc.)
- **Smoke test**: Brief binary run to verify starts without panic: `<start cmd> --headless 2>&1 | head -20`
- **Memory path**: Where compound docs are stored
- **Branch**: Target branch for PRs (`feat/engine`, `main`, etc.)

**Crash detection:** Parse test output for panics/crashes (`thread panicked at`, `Segmentation fault`, `panic:`). Write to `.backlog.json` as `severity: crash` — highest priority.

**axloop skill development:** This skill develops itself. When used on a project, the loop operates on that project's codebase. The skill's own files stay in `~/Documents/skills/Axloop/`.
