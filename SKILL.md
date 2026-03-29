---
name: axloop
preamble-tier: 2
description: |
  Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.
  Three loops: Project Loop (forever) + Sprint Loop (finite, repeatable) + Session Loop (one Manager per session).
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

Manager boots, reads state files, resumes the Project Loop. Stays alive for the session.

---

## Three Loops

**Project Loop** — runs forever, survives sessions. Sprints never stop. State in `worktree/` files.

**Sprint Loop** — finite, repeatable unit inside the Project Loop. A completed sprint immediately triggers the next. There is no valid exit condition.

**Session Loop** — one Manager per session. Reacts to events. Spawns all work in parallel. Never idles.

---

## What the Manager Does

The Manager is a **persistent background agent**. It does NOT do work itself — only recruits workers, enforces skills, and resolves conflicts. The Manager recruits whatever roles are needed for each sprint — Dev, QA, DevOps, Designer, or others as required.

```
PROJECT LOOP (forever):
  SPRINT LOOP:
    1. Run /ce:plan → generate sprint backlog (5–10 items)
    2. Write sprint goal + items to worktree/.sprint-state.json
    3. Spawn ALL workers in parallel via Agent
    4. Workers report results to Manager only
    5. Manager resolves conflicts
    6. Run /ship to land and deploy
    7. Run /retro + /ce:compound
    8. Write sprint summary to worktree/.sprint-log.json
    9. GOTO 1 — plan next sprint immediately, no pause, no idle

SESSION BOUNDARY:
  Before session ends → write loop.log + cycle-wip + sprint-state
  Next session → reads those files, resumes mid-sprint if needed
```

**Key rules:**
- Manager NEVER does implementation work — only spawns workers
- Manager NEVER skips a skill — skills are laws, not suggestions
- Manager ALWAYS spawns subagents in parallel — never one at a time
- A completed sprint is NEVER a stop condition — it triggers the next sprint
- An empty backlog is NEVER a stop condition — it triggers /ce:plan for the next sprint

**Conflict resolution (Manager decides):**
- Dev says fix works, QA says it broke something else → Manager decides, re-assigns
- DevOps says build passes, smoke test says binary panics → Manager picks smoke test, re-assigns
- Two devs claim the same item → Manager resolves, one keeps it

---

## What Workers Do

Workers do implementation work and report results to Manager only. Workers never coordinate with each other.

**Dev**: investigate → write fix → `/simplify` → `/review` → open PR → **report to Manager**

**QA**: `/review` → `/qa` → `/ce:review` → **report approved/rejected to Manager**

**Designer**: `/design-review` → **report approved/rejected to Manager**

**DevOps**: build → smoke test → `/browse` → `/ship` → **report pass/fail to Manager**

---

## Skills

All skills run automatically. Skipping a skill is a loop violation.

Before spawning any worker, the Manager MUST verify the relevant skill is queued in `.sprint-state.json`. Workers MUST mark skills as completed in state before reporting back.

| Skill | Who | When |
|-------|-----|------|
| `/investigate` | Dev | Debugging a failing test |
| `/simplify` | Dev + DevOps | Code quality pass |
| `/review` | Dev + QA | Before every PR |
| `/qa` | QA | Functional tests |
| `/browse` | DevOps | Smoke test |
| `/ship` | DevOps | Land and deploy |
| `/design-review` | Designer | UI spec review |
| `/retro` | Manager | After every sprint |
| `/ce:plan` | Manager | Start of every sprint |
| `/ce:review` | Manager + QA | Complex PRs (6–15 reviewers) |
| `/ce:compound` | Manager | After every sprint — writes to docs/solutions/ |

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

### sprint-state.json schema

```json
{
  "sprint": 3,
  "status": "active",
  "goal": "improve test coverage on auth module",
  "items": ["item-a", "item-b", "item-c"],
  "completed": ["item-a"],
  "skills_required": ["/ce:plan", "/investigate", "/review", "/retro", "/ce:compound"],
  "skills_completed": ["/ce:plan"],
  "started_at": "2026-03-29T10:00:00Z"
}
```

The Manager checks `skills_completed` before marking a sprint done. If any required skill is missing, the sprint is not complete.

---

## Laws

### Project Loop (survive sessions)

1. Never merge without a passing smoke test
2. Never leave the repo broken
3. Write `loop.log` + `cycle-wip` + `sprint-state` before session ends
4. Write CE compound docs after every sprint

### Sprint Loop

5. A completed sprint immediately starts the next — no pause, no idle
6. An empty backlog triggers `/ce:plan` for the next sprint — never a stop
7. Every sprint MUST complete all required skills before closing

### Session Loop

8. Manager never does implementation work — always spawn workers
9. Workers never do each other's work — roles stay in their lane
10. Workers report to Manager only — Manager handles all conflicts
11. Spawn ALL subagents in parallel — never wait for one to finish

---

## Tool Restrictions

The Manager is **physically restricted** to these tools only:

```
Read, Glob, Grep, Agent, Skill
```

The Manager cannot call `Bash`, `Write`, or `Edit`. This is enforced at the tool level, not just by instruction. If the Manager needs code written or files changed, it spawns a worker.

Workers have full tool access: `Bash`, `Read`, `Write`, `Edit`, `Grep`, `Glob`, `WebSearch`, `Skill`.

---


## Setup (per project)

Edit in SKILL.md before running:

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
