# axloop

Autonomous AI-driven DevOps team. Manager coordinates developers, designer, QA, DevOps — all via SendMessage. Event-driven, no cron, no polling.

Two loops work together:

- **Project Loop** — Fix all failures. Improve forever. Zero regressions. Survives sessions.
- **Session Loop** — Process all ready events. Spawn all parallel work. Never idle. One Manager per session.

## Usage

```
/axloop <path-to-doc> [rapid|quality|auto]
```

## 10 Laws

**Project Loop** (survive sessions):
1. Never push to main/feat directly — all PR
2. Never merge without smoke test
3. Never leave repo broken
4. Never skip rollback on smoke failure
5. Never write to main/feat branch
6. Always write loop.log
7. Always write cycle-wip before session ends

**Session Loop** (per session):
8. Always assign when devs idle and backlog has items
9. Always spawn in parallel
10. Always reconcile on startup

## Skills (automatic)

| Skill | Who | What |
|-------|-----|------|
| `/investigate` | Dev | Root cause |
| `/simplify` | Dev + DevOps | Quality |
| `/review` | Dev + QA | Pre-PR review |
| `/qa` | QA | Functional test |
| `/browse` | DevOps | Smoke test |
| `/canary` | DevOps | Post-deploy monitor |
| `/design-review` | Designer | UI/UX spec |
| `/retro` | Manager | Cycle retro |
| `/ce:plan` | Manager | Deep planning |
| `/ce:review` | Manager + QA | 6-15 parallel reviewers |
| `/ce:compound` | Manager | Build knowledge base |

## Session Resume

All Project Loop state in `worktree/` files. New Manager reads `.loop.log`, `.backlog.json`, `.cycle-wip`, open PRs — resumes from where left off. No work lost.

## Modes

| Mode | Developers | QA | Skills |
|------|-----------|-----|--------|
| rapid | parallel | specific test | skip /canary |
| quality | 1 at a time | full suite | all mandatory |
| auto | parallel | quality if regressions | /canary every 10 cycles |

## Requirements

- Claude Code session
- Git repository + `gh` CLI
- Test + build commands in SKILL.md

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
