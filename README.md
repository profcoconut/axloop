# axloop

Autonomous AI-driven DevOps team. Manager coordinates developers, designer, QA, DevOps — all via SendMessage. Event-driven, no cron, no polling.

**The Manager is a persistent background agent.** It stays alive across the session, reacts to events, never waits for one step to finish before spawning the next.

**gstack + CE skills run automatically throughout.** The 10 Laws are the constitution that makes the forever auto loop possible.

## Usage

```
/axloop <path-to-doc> [rapid|quality|auto]
```

## Team

```
PRODUCT MANAGER → MANAGER → Designer, DevOps, QA
                          ↓
                    Dev1, Dev2, ... DevN  (parallel)
```

- **Manager**: persistent background agent, watches state, reacts to events, spawns all roles
- **Developers**: parallel, TDD, PR workflow
- **QA**: regression + design verification gate
- **DevOps**: build gate, deploy, smoke test, rollback
- **Designer**: UI/UX specs before implementation

## The Forever Auto Loop

Runs forever without human intervention. Survives session deaths. Self-heals on failure. Compounds knowledge over time. Never goes idle.

## 10 Laws

1. Never push to main/feat directly — all PR
2. Never merge without smoke test
3. Never leave repo broken
4. Never skip rollback on smoke failure
5. Never write to main/feat branch
6. Always write loop.log
7. Always write cycle-wip before session ends
8. Always assign when devs idle and backlog has items
9. Always spawn in parallel
10. Always reconcile on startup

## Skills (automatic)

| Skill | Role |
|-------|------|
| `/investigate` | Dev: root cause |
| `/simplify` | Dev + DevOps: quality |
| `/review` | Dev + QA: pre-PR review |
| `/qa` | QA: functional test |
| `/browse` | DevOps: smoke test |
| `/canary` | DevOps: post-deploy monitor |
| `/retro` | Manager: cycle retrospective |
| `/ce:plan` | Manager: deep planning |
| `/ce:review` | Manager + QA: 6-15 parallel reviewers |
| `/ce:compound` | Manager: build knowledge base |

## Session Resume

All state in `worktree/` files. New session reads loop.log, backlog, cycle-wip, deploy-state, open PRs — resumes from where left off. No work lost.

## Modes

| Mode | Developers | QA | Skills |
|------|-----------|-----|--------|
| rapid | parallel | specific test | skip /canary |
| quality | 1 at a time | full suite | all mandatory |
| auto | parallel | quality if regressions | /canary every 10 cycles |

## Requirements

- Claude Code session
- Git repository + `gh` CLI
- Test + build commands configured in SKILL.md

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
