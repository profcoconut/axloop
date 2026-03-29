---
name: axloop
preamble-tier: 2
version: 1.1.0
description: |
  Autonomous AI-driven DevOps team that runs like a real company.
  A manager coordinates: 4 developers (parallel PRs), a designer (specs),
  QA (regression + design verification), and DevOps (build gate + deploy + smoke test).
  Event-driven, no cron. Uses gstack skills automatically throughout the workflow.
  Takes a doc input (feature spec, brainstorm, plan).
  Built on VoxParty's TDD philosophy. Use when you want to start a long-running
  autonomous improvement cycle or when you say "run the loop" or "TDD loop".
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

- **4 Developers** — parallel PRs, TDD, never push to main directly
- **1 Designer** — UI/UX specs before implementation
- **1 QA** — regression tests + design verification gate
- **1 DevOps** — build gate, deploy pipeline, smoke tests, rollback

**Every agent uses gstack skills automatically** as part of their workflow. Skills are not optional — they are the execution layer.

**VoxParty TDD Philosophy (from ~/Documents/voxparty/CLAUDE.md):**
- TDD is mandatory: write failing tests before implementing
- Never modify tests to make a failing suite pass
- Every failing test must be investigated and resolved

## Gstack Skills as Execution Layer

Each team role has gstack skills baked into their workflow. Agents invoke skills via `Skill` tool. Skills are always loaded — not optional.

### Skill → Role Mapping

| Role | Primary Skills | When Used |
|------|---------------|-----------|
| **Manager** | `/investigate` (root cause), `/simplify` (code quality), `/review` (PR review), `/retro` (cycle analysis) | Every cycle — after each fix, after regressions, weekly |
| **Developer** | `/investigate` (debug), `/simplify` (quality), `/review` (before PR) | Before opening PR — every fix goes through review |
| **QA** | `/review` (code review), `/qa` (functional test) | Every PR — regression + design verification |
| **DevOps** | `/browse` (smoke test), `/canary` (post-deploy), `/simplify` (build quality) | Every deploy — smoke test + canary check |
| **Designer** | `/design-review` (UI check), `/browse` (visual) | Every design spec — before approving |

### Skill Invocation Pattern

Every agent uses this pattern:
```
1. Do the work (read files, investigate)
2. Invoke the appropriate gstack skill for that phase
3. Act on skill output
4. Report to manager via SendMessage
```

**Never skip a skill phase.** Skills are the quality gate, not optional polish.

## Invocation

```
/axloop <path-to-doc> [rapid|quality|auto]
```

**Examples:**
- `/axloop ~/Documents/voxparty/docs/plans/2026-03-29-002-feat-auto-cycle-plan.md`
- `/axloop ~/myfeature.md rapid`
- `/axloop ./brainstorm.md quality`

**Arguments:**
- `path-to-doc` (required): Path to the input document
- `mode` (optional, default: auto): `rapid` (max velocity), `quality` (strict regression), `auto` (dynamic balance)

## How the Team Works

### Shared Infrastructure (all in worktree/)

Communication is via `SendMessage`. Files are only for state persistence and human visibility. Skills are the execution layer.

```
worktree/.backlog.json          — bug backlog
worktree/.feature-backlog.json   — feature backlog (PM owns)
worktree/.loop-status.json      — live view of every agent (human readable)
worktree/.loop.log              — append-only event log (human readable)
worktree/.loop-mode             — current mode (rapid/quality/auto)
worktree/.deploy-state.json     — last SHA, health, regressions
worktree/.proposals/<dev_id>/<item_id>.json — developer fix proposal
worktree/.designs/<feature_id>.md — designer specs
worktree/.approved/<item_id>    — QA/DevOps approval marker
worktree/.rejected/<item_id>    — QA/DevOps rejection reason
```

### The Full Team Lifecycle (SendMessage-driven)

```
PM SendMessage → Manager: "here is the prioritized feature backlog"
Manager SendMessage → Designer: "here is your feature item, produce a spec"
Manager SendMessage → Dev1, Dev2, Dev3, Dev4: "here is your item, start"
Dev1-4 (parallel):
  1. /investigate <failing-test> — root cause investigation
  2. Write fix to source file
  3. /simplify — review the fix for quality/efficiency before PR
  4. /review — pre-PR review
  5. gh pr create
  6. SendMessage → Manager: "PR ready: <item_id>"
Designer: writes .designs/<id>.md → /design-review → SendMessage → Manager: "spec ready"
Manager SendMessage → DevOps: "check build for PR <item_id>"
DevOps: cargo build → SendMessage → Manager: "build pass/fail"
Manager SendMessage → QA: "run regression + design check for <item_id>"
QA:
  1. /review — code quality review of the PR
  2. /qa — functional testing
  3. SendMessage → Manager: "approved/rejected: <item_id>"
Manager SendMessage → DevOps: "merge + deploy <item_id>" (if approved)
DevOps:
  1. gh pr merge → cargo build
  2. /browse → smoke test (navigate, verify page loads, check console errors)
  3. /canary — post-deploy monitoring for 5 minutes
  4. SendMessage → Manager: "deployed OK" or "rollback needed"
  5. If rollback needed: git revert → re-assign to dev
Manager: after every cycle → /retro — analyze what worked/what didn't, update memory
Manager: after every 10 cycles → /simplify — full codebase quality sweep
Manager updates .loop-status.json at every step (human visibility)
```

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

### Manager's Tools & Constraints

**Allowed tools:** Bash, Read, Write, Edit, Grep, Glob, Agent, WebSearch, Skill

- Bash: git operations, cargo commands, gh CLI, file checks
- Read: all source files, all state files, all proposals
- Write: .backlog.json, .loop-status.json, .deploy-state.json, memory entries
- Edit: source files only — never modifies tests
- Agent: spawns all sub-agents (developer, QA, DevOps, Designer)
- Skill: invokes gstack skills — `/investigate`, `/simplify`, `/review`, `/retro`, `/canary`, `/browse`, `/qa`, `/design-review`
- Never: does not directly approve/reject — QA/DevOps do that

### Sub-Agent Spawning

Manager spawns each sub-agent via `Agent` tool with `run_in_background: true`. Each agent's prompt includes the full gstack skill invocation pattern above.

**Developer:** Investigate → simplify → review → gh pr create → SendMessage → Manager

**Designer:** Write spec → design-review → SendMessage → Manager

**QA:** review → qa → SendMessage → Manager

**DevOps:** cargo build → browse (smoke) → canary → SendMessage → Manager

## Modes

| Mode | Developers | Designer | QA | Deploy | Skill Usage |
|------|-----------|---------|-----|--------|-------------|
| rapid | 4 parallel | optional | specific test only | always | skip /canary, /simplify optional |
| quality | 1 at a time | required | full regression suite | always after QA | all skills mandatory |
| auto | 4 parallel | optional | quality if regressions exist | always | /canary + /simplify every 10 cycles |

## Memory Persistence

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

## Visual Verification

```bash
cat worktree/.loop-status.json   # every agent's current state
cat worktree/.loop.log           # event history
cat worktree/.backlog.json       # bug backlog
cat worktree/.deploy-state.json  # last SHA + health
gh pr list                      # all open PRs
```

## VoxParty-Specific Paths

- **Engine**: `~/Documents/voxparty/.worktrees/engine/`
- **Memory**: `~/Users/tengpeng/.claude/projects/-Users-tengpeng-Documents-voxparty/memory/`
- **Branch**: `feat/engine`
- **Test command**: `cargo test`
- **Build command**: `cargo build`

## Exit Conditions

Loop runs until:
1. Human stops it: write `stop` to `worktree/.loop-mode`
2. Session ends
3. All backlogs empty and all agents idle
