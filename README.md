# axloop

Autonomous AI-driven DevOps team skill for Claude Code. Models a real software team with PM, Designer, N Developers, QA, and DevOps — all coordinated via SendMessage.

**Dual-skill architecture:** gstack (decisions + browser QA) + CE (planning + deep review + compound knowledge). Framework-agnostic.

Designed to achieve the **forever auto loop** — running indefinitely with self-healing, self-improving, zero human intervention. The 10 Manager's Laws make this possible.

## Usage

```
/axloop <path-to-doc> [rapid|quality|auto]
```

**Example:**
```
/axloop ~/myproject/docs/feature-plan.md rapid
```

## Team Structure

```
PRODUCT MANAGER → MANAGER → Designer, DevOps, QA
                          ↓
                    Dev1, Dev2, ... DevN  (unlimited parallel)
```

- **PM**: owns what to build, prioritizes backlog
- **Designer**: produces UI/UX specs before implementation
- **Manager**: decides how, assigns work, coordinates all agents
- **Dev1-N**: unlimited parallel developers, PR-based workflow, TDD
- **QA**: regression tests + design verification gate
- **DevOps**: build gate, deploy pipeline, smoke tests, rollback

## How It Works

The Manager is a persistent background agent. It never exits after spawning workers — it stays alive and reacts to SendMessage from sub-agents. **Purely event-driven, no cron, no polling.**

All communication via SendMessage. Files are for state persistence and human visibility only.

**The forever auto loop:** Loop survives session deaths (all state in files), self-heals (rollback on smoke fail), compounds knowledge (CE builds searchable knowledge base), never goes idle (devs always working).

## Modes

| Mode | Developers | Designer | QA | Deploy | Skill Usage |
|------|-----------|---------|-----|--------|-------------|
| rapid | N parallel | optional | specific test only | always | skip /canary; `/ce:compound` after cycle |
| quality | 1 at a time | required | full regression suite | always after QA | all skills mandatory |
| auto | N parallel | optional | quality if regressions exist | always | /canary + /simplify every 10 cycles |

## Knowledge Accumulation

After every cycle, `/ce:compound` writes a structured doc to `docs/solutions/` — context, solution, prevention. Future `/ce:plan` calls find these via learnings-researcher. The longer axloop runs, the smarter it gets.

## Session Resume

All state is in `worktree/` files — survives session deaths. New session reads `.loop.log`, `.loop-status.json`, `.cycle-wip`, backlog, and resumes from where it left off. No work is lost.

## Requirements

- Claude Code session
- Git repository with a working branch
- `gh` CLI for PR operations
- Test command and build command configured in SKILL.md

## Skill Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
