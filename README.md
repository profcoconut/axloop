# axloop

Autonomous AI-driven DevOps team skill for Claude Code. Models a real software team with PM, Designer, 4 Developers, QA, and DevOps — all coordinated via SendMessage.

Framework-agnostic. Configure test command, build command, and smoke test per project.

Designed to achieve the **forever auto loop** — running indefinitely with self-healing, self-improving, zero human intervention. The 10 Manager's Laws make this possible. Project-agnostic — configure test/build commands per project.

**Dual-skill architecture:** gstack (decision + browser QA) + CE (planning + deep review + compound knowledge). Framework-agnostic.

## Usage

```
/axloop <path-to-doc> [rapid|quality|auto]
```

**Arguments:**
- `path-to-doc` (required): Feature spec, brainstorm, or plan document
- `mode` (optional, default: `auto`): `rapid` (max velocity), `quality` (strict regression), `auto` (dynamic balance)

**Example:**
```
/axloop ~/Documents/myproject/docs/feature-plan.md rapid
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

## Skill Architecture

**gstack** handles decision-making and real browser testing:
- `/plan-ceo-review`, `/plan-eng-review` — product and architecture review
- `/qa` — browser-based functional testing
- `/investigate`, `/simplify`, `/retro` — root cause, quality, cycle analysis

**CE** handles planning depth and compound knowledge:
- `/ce:plan` — spawns research agents, reads historical learnings
- `/ce:review` — 6-15 specialized parallel reviewers
- `/ce:compound` — builds searchable knowledge base after each cycle

## Communication

All agents communicate via `SendMessage`. Files are for state persistence and human visibility only.

## Knowledge Accumulation

After every cycle, `/ce:compound` writes a structured doc to `docs/solutions/` capturing:
- Context: what was fixed, what failed
- Solution: root cause and fix
- Prevention: how to avoid it in the future

This knowledge is searchable by future `/ce:plan` calls — the longer axloop runs, the smarter it gets.

## Session Resume

The loop survives session deaths. All state is in `worktree/` files:
- `.loop.log` — full event history
- `.loop-status.json` — what each agent was doing
- `.cycle-wip` — interrupted cycle state (resume from WIP, not restart)
- `.backlog.json`, `.deploy-state.json`, open PRs

New session reads these files and resumes the pipeline from where it left off. No work is lost.

## Status Files

```bash
cat worktree/.loop-status.json   # live agent states
cat worktree/.loop.log           # event history
cat worktree/.backlog.json       # current backlog
gh pr list                       # all open PRs
```

## Modes

| Mode | Developers | Designer | QA | Worker Timeout |
|------|-----------|---------|-----|----------------|
| rapid | N parallel | optional | specific test only | 120s |
| quality | 1 at a time | required | full regression suite | 60s |
| auto | N parallel | optional | quality when regressions exist | 60s |

## Requirements

- Claude Code session
- Git repository with a working branch (e.g. `feat/engine`, `main`)
- `gh` CLI for PR operations
- Test command and build command configured in SKILL.md

## Skill Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
