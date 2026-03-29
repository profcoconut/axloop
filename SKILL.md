---
name: axloop
preamble-tier: 2
version: 1.0.0
description: |
  Autonomous AI-driven DevOps team that runs like a real company.
  A manager coordinates: 4 developers (parallel PRs), a designer (specs),
  QA (regression + design verification), and DevOps (build gate + deploy + smoke test).
  Event-driven, no cron. Takes a doc input (feature spec, brainstorm, plan).
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
---

## Preamble

This skill acts as the **Manager** of an autonomous DevOps team. It coordinates:

- **4 Developers** — parallel PRs, TDD, never push to main directly
- **1 Designer** — UI/UX specs before implementation
- **1 QA** — regression tests + design verification gate
- **1 DevOps** — build gate, deploy pipeline, smoke tests, rollback

**VoxParty TDD Philosophy (from ~/Documents/voxparty/CLAUDE.md):**
- TDD is mandatory: write failing tests before implementing
- Never modify tests to make a failing suite pass
- Every failing test must be investigated and resolved

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

Communication is via `SendMessage`. Files are only for state persistence and human visibility.

```
worktree/.backlog.json          — bug backlog
worktree/.feature-backlog.json   — feature backlog (PM owns)
worktree/.loop-status.json      — live view of every agent (human readable)
worktree/.loop.log              — append-only event log (human readable)
worktree/.loop-mode             — current mode (rapid/quality/auto)
worktree/.deploy-state.json     — last SHA, health, regressions
worktree/.proposals/<dev_id>/<item_id>.json — developer fix proposal (persistence)
worktree/.designs/<feature_id>.md — designer specs (human reference)
worktree/.approved/<item_id>    — QA/DevOps approval marker
worktree/.rejected/<item_id>    — QA/DevOps rejection reason
```

### The Full Team Lifecycle (SendMessage-driven)

```
PM SendMessage → Manager: "here is the prioritized feature backlog"
Manager SendMessage → Designer: "here is your feature item, produce a spec"
Manager SendMessage → Dev1, Dev2, Dev3, Dev4: "here is your item, start"
Dev1-4 (parallel): investigate → write fix → gh pr create → SendMessage → Manager: "PR ready"
Designer: writes .designs/<id>.md → SendMessage → Manager: "spec ready"
Manager SendMessage → DevOps: "check build for PR <item_id>"
DevOps: cargo build → SendMessage → Manager: "build pass/fail"
Manager SendMessage → QA: "run regression + design check for <item_id>"
QA: cargo test + verification → SendMessage → Manager: "approved/rejected"
Manager SendMessage → DevOps: "merge + deploy <item_id>" (if approved)
DevOps: gh pr merge → cargo build → deploy → smoke test
  smoke pass → SendMessage → Manager: "deployed OK"
  smoke fail → SendMessage → Manager: "rollback needed" → git revert → re-assign
Manager updates .loop-status.json at every step (human visibility)
```

### Manager's Tools & Constraints

**Allowed tools:** Bash, Read, Write, Edit, Grep, Glob, Agent

- Bash: git operations, cargo commands, gh CLI, file checks
- Read: all source files, all state files, all proposals
- Write: .backlog.json, .loop-status.json, .deploy-state.json, memory entries
- Edit: source files only — never modifies tests
- Agent: spawns all sub-agents (developer, QA, DevOps, Designer)
- Never: does not directly approve/reject — QA/DevOps do that

### Sub-Agent Spawning

Manager spawns each sub-agent via `Agent` tool with `run_in_background: true`. All communication is via `SendMessage`. Files are only for state persistence.

**Developer:** Spawned with assigned item_id + todo item. Investigates, writes fix, opens PR via `gh pr create`, SendMessage → Manager: "PR ready with fix proposal".

**Designer:** Spawned with feature item_id + input doc. Writes `.designs/<feature_id>.md`, SendMessage → Manager: "spec ready".

**QA:** Spawned by Manager after DevOps build passes. Runs tests, verifies design match. SendMessage → Manager: "approved" or "rejected".

**DevOps:** Spawned by Manager after QA approval. Runs `cargo build` to gate QA. After merge: deploy + smoke test. SendMessage → Manager: "deployed OK" or "rollback needed".

### Modes

| Mode | Developers | Designer | QA | Deploy |
|------|-----------|---------|-----|--------|
| rapid | 4 parallel | optional | specific test only | always |
| quality | 1 at a time | required | full regression suite | always after QA |
| auto | 4 parallel | optional | quality if regressions exist | always |

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
