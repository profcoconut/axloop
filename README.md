# axloop

Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.

Three loops:
- **Project Loop** — runs forever, survives sessions
- **Sprint Loop** — finite, repeatable, immediately triggers next sprint
- **Session Loop** — one Manager per session, spawns all work in parallel

## Start

```
/axloop <doc> [rapid|quality|auto]
```

## Three Roles

**Manager** — persistent background agent. Never does work. Spawns workers, enforces skills, resolves conflicts. Tool-restricted: Read, Glob, Grep, Agent, Skill only.

**Workers** — do implementation work. Report to Manager only. Full tool access.

## Skills (automatic)

`Skill("gstack:investigate")` `/simplify` `Skill("gstack:review")` `Skill("gstack:qa")` `Skill("gstack:browse")` `Skill("gstack:canary")` `Skill("gstack:design-review")` `Skill("gstack:retro")` `Skill("compound-engineering:ce-plan")` `Skill("compound-engineering:ce-review")` `Skill("compound-engineering:ce-compound")`

## Laws

**Project Loop:** Never merge without smoke test · Never leave repo broken · Write loop.log + cycle-wip + sprint-state · Write CE docs

**Sprint Loop:** Sprint immediately triggers next sprint · Empty backlog triggers Skill("compound-engineering:ce-plan") · All skills must complete before closing sprint

**Session Loop:** Manager never does work · Workers never do each other's work · Workers report to Manager only · Spawn ALL in parallel

## Tool Restrictions

Manager: Read, Glob, Grep, Agent, Skill only
Workers: Bash, Read, Write, Edit, Grep, Glob, WebSearch, Skill

## Session Resume

All state in worktree/. New Manager reads loop.log + cycle-wip + sprint-state, resumes mid-sprint.

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
