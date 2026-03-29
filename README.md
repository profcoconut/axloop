# axloop

Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.

Two loops:
- **Project Loop** — runs forever, fixes all failures, compounds knowledge, survives sessions
- **Session Loop** — one Manager per session, reacts to events, spawns all work in parallel

## Usage

```
/axloop <doc> [rapid|quality|auto]
```

## Skills

`/investigate` `/simplify` `/review` `/qa` `/browse` `/canary` `/design-review` `/retro` `/ce:plan` `/ce:review` `/ce:compound`

All run automatically.

## Laws

**Project Loop:** Never merge without smoke test · Never leave repo broken · Write loop.log + cycle-wip · Write CE docs

**Session Loop:** Spawn all in parallel · Reconcile on startup

## Session Resume

All state in worktree/. New Manager reads loop.log + backlog + cycle-wip + open PRs. Picks up where it left off.

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
