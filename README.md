# axloop

Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.

Three loops: **Project Loop** (forever) · **Sprint Loop** (finite, repeatable, triggers next immediately) · **Session Loop** (one Manager per session)

## Start

```
/axloop <doc> [rapid|quality|auto]
```

## Roles

**Manager** — persistent background agent. Never does work. Spawns workers, enforces skills, resolves conflicts. Tool-restricted: Read, Glob, Grep, Agent, Skill only.

**Workers** — do implementation work. Report to Manager only. Full tool access.

## Skills

`Skill("gstack:investigate")` `/simplify` `Skill("gstack:review")` `Skill("gstack:qa")` `Skill("gstack:browse")` `Skill("gstack:ship")` `Skill("gstack:design-review")` `Skill("gstack:retro")` `Skill("compound-engineering:ce-plan")` `Skill("compound-engineering:ce-review")` `Skill("compound-engineering:ce-compound")`

## Laws

**If/Then:** Sprint completes → start next immediately · Backlog empty → Skill("compound-engineering:ce-plan") · Session ends → write loop.log + cycle-wip + sprint-state

**Always:** Never merge without smoke test · Never leave repo broken · Complete all skills before closing sprint · Manager never does implementation work · Workers never do each other's work · Workers report to Manager only · Spawn ALL in parallel · Write CE docs after every sprint

## Tool Restrictions

Manager: Read, Glob, Grep, Agent, Skill only
Workers: Bash, Read, Write, Edit, Grep, Glob, WebSearch, Skill

## Session Resume

All state in worktree/. New Manager reads loop.log + cycle-wip + sprint-state, resumes mid-sprint.

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
