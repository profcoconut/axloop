# axloop

Autonomous DevOps team in Claude Code. Manager coordinates developers, QA, DevOps via SendMessage.

**One loop:** Sprint Loop runs forever — each completed sprint immediately triggers the next.

## Start

```
/axloop <doc> [rapid|quality|auto]
```

## Roles

**Manager** — persistent background agent. Never does work. Spawns workers, enforces skills, resolves conflicts. Tool-restricted: Read, Glob, Grep, Agent, Skill only.

**Workers** — do implementation work. Report to Manager only. Full tool access.

## Skills

`Skill("investigate")` `Skill("simplify")` `Skill("review")` `Skill("qa")` `Skill("browse")` `Skill("ship")` `Skill("design-review")` `Skill("retro")` `Skill("compound-engineering:ce-plan")` `Skill("compound-engineering:ce-review")` `Skill("compound-engineering:ce-compound")`

## Laws

**If/Then:** Sprint completes → start next immediately · Backlog empty → Skill("compound-engineering:ce-plan")

**Always:** Never merge without smoke test · Never leave repo broken · Complete all skills before closing sprint · Manager never does implementation work · Workers never do each other's work · Workers report to Manager only · Spawn ALL in parallel · Write CE docs after every sprint

## Tool Restrictions

Manager: Read, Glob, Grep, Agent, Skill only
Workers: Bash, Read, Write, Edit, Grep, Glob, WebSearch, Skill

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
