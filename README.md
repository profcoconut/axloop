# axloop

> **Your codebase ships itself. You just watch.**

An autonomous DevOps team running forever inside Claude Code. One command. Infinite sprints. No waiting.

---

## What it does

- **Plans** its own sprint backlog every cycle
- **Writes** code, opens PRs, resolves conflicts
- **Tests** everything before it touches main
- **Ships** to production automatically
- **Reflects** with a retro — then immediately starts the next sprint

No idle state. No manual nudging. The loop runs until you stop it.

---

## Why axloop?

Most AI coding tools are reactive — you prompt, they respond, you review, you prompt again. **You're still the loop.**

axloop is different. It encodes the full rhythm of software development as an autonomous, self-driving system. Finish a sprint at 3am and it starts the next one. Come back in the morning to completed work, written retros, and new sprints already running.

---

## Core principles

### Loops, not prompts
- No exit condition — `GOTO 0` is always the final step
- Runs until *you* stop it, not until it decides it's done
- Always making forward progress, even while you sleep

Software development doesn't have a finish line, so axloop doesn't either.

### Skills are law
- Every role has required skills that must run at specific moments
- Skipping a skill is a **loop violation** — the Manager enforces it hard
- Sprint close is blocked until `skills_completed` matches `skills_required`

Autonomous systems cut corners under pressure. Mandatory, auditable skills close that gap.

### Strict role separation
- Dev codes. QA tests. DevOps ships. Nobody crosses lanes.
- Workers never talk to each other — everything routes through the Manager
- Manager is **tool-restricted**: no `Bash`, no `Write`, no `Edit` — coordination only

When the Manager can implement, it stops coordinating. The tool restriction is what makes the architecture real, not just a convention.

### Parallel by default
- All workers spawn simultaneously at sprint start — never one at a time
- Dev, QA, and DevOps work concurrently across separate backlogs
- Conflicts get resolved by the Manager, not avoided by serializing

Sequential spawning creates false dependencies. Real teams work in parallel.

### Hard safety gate
- Nothing merges without a passing smoke test. Period.
- Failed smoke test → Manager rejects → DevOps reworks
- The repo never ships broken, no matter how fast the loop moves

### Crash-proof state
- Every event written to `worktree/` — goals, items, skills, WIP
- Session dies? Manager reads state files, restarts dead workers, continues
- The loop's continuity lives in files, not in memory

---

## Get started

```
/axloop <doc> [rapid|quality|auto]
```

| Mode | What it does |
|------|-------------|
| `rapid` | Tighter sprints, max throughput |
| `quality` | Deeper reviews, full CE documentation |
| `auto` | Manager reads backlog and decides |

**Stop anytime:**
```bash
echo "stop" > worktree/.project-loop-state
```

---

## Architecture

```
Manager  (Read, Glob, Grep, Agent, Skill only)
├── reads worktree/ state on boot
├── generates sprint backlog via ce-plan
├── spawns Dev + QA + DevOps in parallel
├── resolves conflicts, enforces skills
└── retro + ce-compound after every sprint

Dev      → investigate → implement → simplify → review → PR
QA       → review → qa → ce-review
DevOps   → build → smoke test → browse → ship
```

---

## Skills

| Skill | Who | When |
|-------|-----|------|
| `ce-plan` | Manager | Start of every sprint |
| `investigate` | Dev | Debugging |
| `simplify` | Dev, DevOps | Code quality pass |
| `review` | Dev, QA | Before every PR |
| `qa` | QA | Functional tests |
| `browse` | DevOps | Smoke test |
| `ship` | DevOps | Auto-merge and deploy |
| `retro` | Manager | After every sprint |
| `ce-review` | Manager, QA | Complex PRs |
| `ce-compound` | Manager | After every sprint |

---

## State files

| File | Purpose |
|------|---------|
| `.sprint-state.json` | Active sprint — goal, items, skills |
| `.sprint-log.json` | Full sprint history |
| `.backlog.json` | Bug and task backlog |
| `.loop.log` | Full event log |
| `.cycle-wip` | Interrupted work, read on resume |
| `.approved/` · `.rejected/` | QA decisions |
| `.proposals/<dev>/` | Open PRs |
