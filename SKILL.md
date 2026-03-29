---
name: axloop
preamble-tier: 2
version: 1.4.0
description: |
  Autonomous AI-driven DevOps team that runs like a real company.
  A manager coordinates: N developers (unlimited parallel PRs), a designer (specs),
  QA (regression + design verification), and DevOps (build gate + deploy + smoke test).
  Event-driven, no cron, no polling. Uses gstack + CE skills automatically throughout.
  gstack handles decision-making (CEO/eng review) and real browser QA.
  CE handles planning, deep review, and compound knowledge accumulation.
  Takes a doc input (feature spec, brainstorm, plan).
  Built on TDD philosophy. Designed to run autonomously for hours,
  self-correcting via rollback and checkpointing. Project-agnostic — configure per project.
  Use when you want to start a long-running autonomous improvement cycle
  or when you say "run the loop" or "TDD loop".
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
- **gstack**: Decision-layer skills — `/plan-ceo-review` (product), `/plan-eng-review` (architecture), `/qa` (browser QA), `/investigate` (root cause), `/simplify` (quality)
- **CE**: Execution-layer skills — `/ce:plan` (spawns research agents, reads historical learnings), `/ce:review` (6-15 parallel specialized reviewers), `/ce:compound` (builds searchable knowledge base from sessions)
- gstack handles "做不做" (whether to do) and "真实测" (real browser testing)
- CE handles "怎么做" (how to plan), "做得好不好" (deep review), and "记住" (knowledge accumulation)

**TDD Philosophy:**
- TDD is mandatory: write failing tests before implementing
- Never modify tests to make a failing suite pass
- Every failing test must be investigated and resolved

## Manager's 10 Laws — The Constitution

**These are non-negotiable. Violating any of these breaks the loop's self-healing properties.**

1. **Never push to main/feat directly.** All code goes through PR. Always.
2. **Never merge without smoke test passing.** Deploy only after DevOps confirms smoke pass.
3. **Never leave the repo in a broken state.** Every fix must pass tests before commit. If interrupted, revert first.
4. **Never skip rollback on smoke failure.** If deployed code fails smoke, rollback immediately — don't wait, don't ask.
5. **Never write to main/feat branch.** Feature branches only. Manager merges via PR.
6. **Always write loop.log.** Every state change is appended — without it, session resume doesn't work.
7. **Always write cycle-wip before session ends.** If the session dies mid-fix, the next Manager must be able to resume.
8. **Always assign when devs are idle and backlog has items.** Never let workers idle while work exists.
9. **Always spawn in parallel, never sequentially.** If multiple events are ready, spawn all at once. Never wait for one to finish before spawning the next.
10. **Always reconcile on startup.** New session reads all state files before acting. Never assume the repo is clean or empty.

## Gstack Skills as Execution Layer

Each team role has gstack skills baked into their workflow. Agents invoke skills via `Skill` tool. Skills are always loaded — not optional.

### Skill → Role Mapping

| Role | Primary Skills | When Used |
|------|---------------|-----------|
| **Manager** | gstack: `/investigate`, `/simplify`, `/retro`; CE: `/ce:plan`, `/ce:review`, `/ce:compound` | `/ce:compound` after every cycle to build knowledge; `/ce:plan` for feature planning |
| **Developer** | gstack: `/investigate` (debug), `/simplify` (quality), `/review` (before PR) | Before opening PR — every fix goes through review |
| **QA** | gstack: `/review`, `/qa`; CE: `/ce:review` (6-15 specialized reviewers) | Every PR — regression + design + deep code review |
| **DevOps** | gstack: `/browse` (smoke test), `/canary` (post-deploy), `/simplify` (build quality) | Every deploy — smoke test + canary check |
| **Designer** | gstack: `/design-review` (UI check), `/browse` (visual) | Every design spec — before approving |

### Skill Invocation Pattern

Every agent uses this pattern:
```
1. Do the work (read files, investigate)
2. Invoke the appropriate gstack skill for that phase
3. Act on skill output
4. Report to manager via SendMessage
```

**Never skip a skill phase.** Skills are the quality gate, not optional polish.

**CE compound step (Manager only — after every cycle):**
After a fix is merged or a rollback completes, run `/ce:compound` to build the knowledge base:
```
/ce:compound "<session summary: what was fixed, what failed, what worked>"
```
CE's `/ce:compound` spawns 3 parallel agents (Context Analyzer, Solution Extractor, Related Docs Finder) and writes a structured doc to `docs/solutions/` — this compound knowledge is searchable by future `/ce:plan` calls via learnings-researcher.

## Invocation

```
/axloop <path-to-doc> [rapid|quality|auto]
```

**First invocation starts the Manager** — it stays alive for the entire session. Subsequent invocations update the input doc or mode.

**Examples:**
- `/axloop ~/myproject/docs/feature-plan.md`
- `/axloop ~/myfeature.md rapid`
- `/axloop ./brainstorm.md quality`

**Arguments:**
- `path-to-doc` (required): Path to the input document (feature spec, brainstorm, or plan)
- `mode` (optional, default: auto): `rapid` (max velocity), `quality` (strict regression), `auto` (dynamic balance)

## How the Team Works

### Shared Infrastructure (all in worktree/)

Communication is via `SendMessage`. Files are only for state persistence and human visibility. Skills are the execution layer. **Directories are auto-created as needed** — Manager creates worktree/, docs/solutions/, and any state subdirectories without asking.

```
worktree/.backlog.json          — bug backlog (all severities: crash, panic, assertion, compilation, flaky)
worktree/.feature-backlog.json   — feature backlog (PM owns)
worktree/.loop-status.json      — live view of every agent: each agent has a 1-sentence current-status field visible to human at all times
worktree/.loop.log              — append-only event log (human readable, critical for session resume)
worktree/.cycle-wip             — interrupted cycle state: what dev was doing when session died
worktree/.loop-mode             — current mode (rapid/quality/auto)
worktree/.deploy-state.json     — last SHA, health, regressions
worktree/.proposals/<dev_id>/<item_id>.json — developer fix proposal
worktree/.designs/<feature_id>.md — designer specs
worktree/.approved/<item_id>    — QA/DevOps approval marker
worktree/.rejected/<item_id>    — QA/DevOps rejection reason
worktree/.crash-report/<item_id>.json — crash/panic output, backtrace, severity
worktree/.rollback/<item_id>    — rollback reason and from_sha
```

### The Manager Loop — How It Actually Runs Continuously

The Manager is a **persistent background agent**. It does NOT exit after spawning workers. It continuously monitors state files and reacts. **Everything is parallel by default.**

**SendMessage is the only hook.** No polling, no cron — the Manager is woken only when a sub-agent SendMessage completes or a state file is written.

**ALWAYS spawn in parallel. Never wait.**
- When assigning devs: spawn ALL available devs at once, not one-by-one
- When a PR opens: spawn DevOps immediately, don't wait
- When build passes: spawn QA immediately, don't wait
- When QA approves: spawn DevOps merge immediately, don't wait
- When smoke fails: spawn rollback immediately, don't wait
- When `/ce:compound` runs: spawns 3 agents in parallel already
- When `/ce:review` runs: spawns 6-15 agents in parallel already

**If the Manager has no pending work and nothing is active:**
Run the test command to find new failures, parse output, write to backlog, assign to devs. This is the idle sweep — keep everyone busy.

### Session Resume — How a New Session Picks Up

When a new Claude Code session starts and the Manager is invoked, it first reads all state files to understand where things left off. **No work is lost when a session dies** — everything persistent survives.

**Manager resume protocol (runs on every /axloop invocation):**

```
1. Read .loop.log          → understand full history of what happened
2. Read .loop-status.json   → what was each agent doing when session died
3. Read .backlog.json       → what items exist and their statuses
4. Read .deploy-state.json  → last known good SHA and health
5. Read any .cycle-wip      → was a cycle interrupted mid-fix?
6. Read .approved/ and .rejected/ → what QA already decided
7. Check gh pr list          → what PRs are open but not merged
8. Reconcile: for each open PR without .approved or .rejected, continue its pipeline stage
9. For each .cycle-wip: resume from the WIP state instead of restarting
10. For items marked "assigned" but no recent activity: check if the dev agent is still alive — if not, reassign
11. Resume the loop from where it was
```

**Reconciliation rules:**
- PR opened but no build check → spawn DevOps build check
- Build passed but no QA → spawn QA
- QA approved but not merged → spawn DevOps merge+deploy
- Item assigned but dev is dead (no recent status update) → reassign to another dev
- WIP cycle exists → resume from investigation state, don't restart

**The Manager never assumes a session died cleanly.** It always verifies the actual state of everything before continuing.

**`.cycle-wip` schema** — written when a cycle is interrupted mid-fix:
```json
{
  "item_id": "item-042",
  "dev_id": "dev_1",
  "step": "regression-test",
  "investigation_state": "found root cause: off-by-one in grid index calculation",
  "fix_draft": "src/core/isom.rs line 234 — changed > to >=",
  "interrupted_at": "2026-03-29T14:23:00Z",
  "test_results_so_far": "cargo test isom::test_grid_to_screen — FAILED still"
}
```

**`.loop.log` schema** — append-only event log, critical for session recovery:
```json
{"ts": "2026-03-29T14:20:00Z", "event": "dev_1 assigned item-042", "actor": "manager"}
{"ts": "2026-03-29T14:21:00Z", "event": "dev_1 investigation started", "actor": "dev_1"}
{"ts": "2026-03-29T14:22:00Z", "event": "dev_1 PR opened", "actor": "dev_1"}
{"ts": "2026-03-29T14:23:00Z", "event": "session ended", "actor": "system"}
```

**Event → Action mapping (Manager decides what to do next, all async):**

```
EVENT: backlog has unassigned items AND devs are available
ACTION: assign multiple items at once — Dev1 on item_A, Dev2 on item_B, Dev3 on item_C...
(no waiting for Dev1 to finish before assigning Dev2)

EVENT: cargo test finds new failures (parsed from test output)
ACTION: write to .backlog.json with severity, assign to available dev

EVENT: cargo test panics (panic in game runtime detected)
ACTION: create crash backlog item, assign to dev immediately, skip normal queue if severe

EVENT: a dev writes worktree/.proposals/<dev_id>/<item_id>.json (PR opened)
ACTION: immediately spawn DevOps → "check build for <item_id>"

EVENT: DevOps writes worktree/.deploy-state.json with build_pass=true
ACTION: immediately spawn QA → "regression + design check for <item_id>"

EVENT: QA writes worktree/.approved/<item_id>
ACTION: immediately spawn DevOps → "merge + deploy <item_id>"

EVENT: QA writes worktree/.rejected/<item_id>
ACTION: immediately re-assign item to dev with rejection notes

EVENT: DevOps smoke test fails (.rollback/* written)
ACTION: immediately rollback: git revert → re-assign to dev

EVENT: smoke test passes
ACTION: run /ce:compound → close backlog item

EVENT: all devs idle AND backlog empty
ACTION: run cargo test to find new failures (keep loop alive)

EVENT: 10 cycles completed
ACTION: run /ce:simplify — full codebase quality sweep
```

**Pipeline parallelism — multiple items in different stages at the same time:**

```
Dev1: investigating item_A        | Dev2: writing fix item_B      | Dev3: reviewing item_C ...
DevOps: building item_B PR         | QA: running regression on item_A
DevOps: deploying item_C           | QA: running regression on item_B
QA: smoke fail on item_C → rollback | Dev1: already starting item_D ...
```

**The Manager never waits for one stage to empty before feeding the next.** Keep every worker busy at all times. If devs are idle and backlog has items, assign. If QA is idle and any build passed, start regression. If DevOps is idle and any item approved, start deploy.

**Priority order when multiple events compete:**
1. Crash/panic (runtime failure) — game is broken, stop everything
2. Rollback (smoke fail) — deployed code is broken
3. Rejection (dev must retry)
4. Approval (deploy while fresh)
5. Build pass (keep pipeline moving)
6. New PR ready for build
7. Unassigned backlog items (keep devs busy)
8. No work found → run cargo test to find failures
9. Cycle cleanup (compound + retro)

### Skill → Phase Reference

**Root cause analysis** (Developer phase 1):
```bash
/investigate <test-name-or-error> "<investigate the failing test>"
```

**Code quality before PR** (Developer phase 3):
```bash
/simplify focus on code quality and efficiency
```

**Pre-PR review** (Developer phase 4):
```bash
/review "<review the diff for this PR>"
```

**Code quality review** (QA phase 1):
```bash
/review "<review the diff for this PR>"
```

**Functional testing** (QA phase 2):
```bash
/qa "<run functional tests on the affected area>"
```

**Design review** (Designer phase):
```bash
/design-review "<review the design spec for this feature>"
```

**Smoke test** (DevOps phase 2):
```bash
/browse "<navigate to the deployed app and verify it loads correctly>"
```

**Post-deploy monitoring** (DevOps phase 3):
```bash
/canary "<monitor the deployed app for 5 minutes and report any anomalies>"
```

**Cycle retrospective** (Manager — after every cycle):
```bash
/retro "<analyze this cycle: what failed, what succeeded, what to improve>"
```

**Knowledge compound** (Manager — after every cycle):
```bash
/ce:compound "<session summary: what was fixed, what failed, what worked, root cause if debugged>"
```

**Deep planning** (Manager — when starting a new feature):
```bash
/ce:plan "<feature description from input doc>"
```

**Deep review** (Manager or QA — for complex PRs):
```bash
/ce:review "<review the diff for this PR — triggers 6-15 specialized reviewers>"
```

### Manager's Tools & Constraints

**Allowed tools:** Bash, Read, Write, Edit, Grep, Glob, Agent, WebSearch, Skill

- Bash: git operations, cargo commands, gh CLI, file checks, branch creation for backups
- Read: all source files, all state files, all proposals
- Write: .backlog.json, .loop-status.json, .deploy-state.json, memory entries
- Edit: source files only — never modifies tests
- Agent: spawns all sub-agents (developer, QA, DevOps, Designer)
- Skill: invokes gstack skills — `/investigate`, `/simplify`, `/review`, `/retro`, `/canary`, `/browse`, `/qa`, `/design-review` — and CE skills — `/ce:plan`, `/ce:review`, `/ce:compound`
- Never: does not directly approve/reject — QA/DevOps do that
- **Auto-create allowed**: Manager creates state directories (worktree/ subdirs, docs/solutions/, etc.) and repos as needed — no permission needed from human
- **Always running**: Manager continuously checks state files and reacts to events — never idle while work exists

### Sub-Agent Spawning

Manager spawns each sub-agent via `Agent` tool with `run_in_background: true`. Each agent's prompt includes the full gstack skill invocation pattern above.

**Developer:** Investigate → simplify → review → gh pr create → SendMessage → Manager

**Designer:** Write spec → design-review → SendMessage → Manager

**QA:** review → qa → SendMessage → Manager

**DevOps:** cargo build → cargo run (game smoke test — verify binary starts without panic) → browse → canary → SendMessage → Manager

## Modes

| Mode | Developers | Designer | QA | Deploy | Skill Usage |
|------|-----------|---------|-----|--------|-------------|
| rapid | N parallel | optional | specific test only | always | skip /canary, /simplify optional; `/ce:compound` after cycle |
| quality | 1 at a time | required | full regression suite | always after QA | all gstack skills + CE deep review mandatory |
| auto | N parallel | optional | quality if regressions exist | always | /canary + /simplify every 10 cycles; `/ce:compound` after cycle |

## Memory Persistence & Knowledge Accumulation

**CE compound knowledge base (primary — searchable across all future sessions):**
After each fix or rollback, run `/ce:compound` to write a structured doc to `docs/solutions/`. This is the project's compound knowledge — future `/ce:plan` calls find it via learnings-researcher. CE's approach solves **accumulation** (not just continuity): every session contributes to a searchable knowledge base that all future sessions can leverage.

**gstack memory entries (append-only — for human review):**
After each fix or rollback, manager writes to `{memory-dir}/YYYYMMDD-HHMMSS-<slug>.md`:

```markdown
---
name: <slug>
description: <one-line summary>
type: feedback
attempt: 1
outcome: succeeded
---

**What went wrong:** <description>
**Root cause:** <underlying issue>
**How to diagnose:** <what signal indicated the problem>
**How to recognize it again:** <what to look for>
```

Memory entries are append-only. Same bug recurs → new entry with `attempt: N+1`.

**CE vs gstack memory distinction:**
- gstack memory entries (this file): human-readable, append-only, session-scoped
- CE compound docs (`docs/solutions/`): machine-searchable, deduplicated, project-wide knowledge base
- Both compound over time. CE's `docs/solutions/` is the primary knowledge layer; gstack memory is the session continuity layer.

## Visual Verification

```bash
cat worktree/.loop-status.json   # every agent's current state + 1-sentence summary
cat worktree/.loop.log           # event history (append-only, survives session restart)
cat worktree/.backlog.json       # bug backlog
cat worktree/.deploy-state.json  # last SHA + health
cat worktree/.cycle-wip          # interrupted cycle state (if any)
gh pr list                      # all open PRs
cat worktree/.approved/<item_id> # QA approval
cat worktree/.rejected/<item_id> # QA rejection
```

**`.loop-status.json` live status schema** — each agent writes a 1-sentence current status:
```json
{
  "manager": { "status": "spawning Dev1, Dev3 to fix isom tests", "step": "assigning" },
  "dev_1":  { "status": "investigating isom::test_grid_to_screen — reading source", "step": "investigate" },
  "dev_3":  { "status": "writing regression test for grid panic", "step": "regression-test" },
  "qa":     { "status": "idle", "step": "waiting" },
  "devops": { "status": "idle", "step": "waiting" }
}
```

## Resilience & Safe Loop Operations

The loop is designed to run for hours without human intervention. Risk mitigation is built in:

**Backup before risky steps:**
- Before git operations that modify shared state (merge, revert), create a backup branch: `feat/engine-backup-YYYYMMDD-HHMMSS`
- Before major refactors, checkpoint current state to a backup branch
- PRs are always worked on in feature branches — never touch `feat/engine` directly until merge-ready and reviewed

**Never leave repo in broken state:**
- Every fix must pass `cargo test` before commit
- If a fix attempt fails mid-cycle (timeout, crash), stash work and revert any unverified changes
- If a regression is detected post-merge, immediately `git revert` and re-assign to dev

**Checkpointing in-progress work:**
- Write WIP state to `.loop-status.json` at every step so interrupted work can resume
- If cycle exceeds timeout budget, write `.cycle-wip` with investigation state — next cycle resumes from it

**Human intervention is only needed when:**
- A bug is blocked after 3 retry attempts
- A design spec is ambiguous and needs clarification
- The backlog is empty and needs prioritization input
- Human explicitly stops the loop

**Auto-mitigation hierarchy:**
1. First attempt: fix from memory/learnings (fast)
2. Second attempt: deeper investigation with `/investigate`
3. Third attempt: escalate — write to backlog with `blocked` status, request human review
4. Rollback at any smoke/regression failure — never wait for human to notice

## Project Configuration

The axloop skill is project-agnostic. Configure per-project by updating these paths in the skill or passing them at invocation:

- **Worktree path**: Where the codebase lives (e.g. `~/Documents/voxparty/.worktrees/engine/`)
- **Test command**: How to run tests (e.g. `cargo test`, `npm test`, `pytest`)
- **Build command**: How to build (e.g. `cargo build`, `npm run build`)
- **Binary start command**: How to run the app/binary (e.g. `cargo run --`, `npm start`)
- **Smoke test**: Brief run of the binary to verify it starts without panic (e.g. `<start cmd> --headless 2>&1 | head -20`)
- **Memory path**: Where compound knowledge docs are stored
- **Branch**: The target branch for PRs (e.g. `feat/engine`, `main`)

**Crash detection**: Parse test output for panics or crashes (e.g. `thread panicked at`, `Segmentation fault`, `panic:`, `Error:`). Write to `.backlog.json` as `severity: crash` — these escalate immediately above all other priorities.

**axloop skill development note:** This skill develops itself. When axloop is used on a project, it operates on that project's codebase. The skill's own development (SKILL.md, docs) stays in `~/Documents/skills/Axloop/`.

## Exit Conditions

Loop runs until:
1. Human stops it: write `stop` to `worktree/.loop-mode`
2. Session ends
3. All backlogs empty and all agents idle
