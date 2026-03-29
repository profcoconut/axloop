# axloop

Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.

**Project Loop** — runs forever, survives sessions
**Session Loop** — one Manager per session, spawns all work in parallel

## Start

```
/axloop <doc> [rapid|quality|auto]
```

## Skills

`/investigate` `/simplify` `/review` `/qa` `/browse` `/canary` `/design-review` `/retro` `/ce:plan` `/ce:review` `/ce:compound`

All run automatically.

## Laws

**Project Loop:** Never merge without smoke test · Never leave repo broken · Write loop.log + cycle-wip · Write CE docs

**Session Loop:** ALWAYS use subagents in parallel — Manager never does work itself · Reconcile on startup

## Session Resume

New Manager reads worktree/ state files, resumes where it left off.

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
