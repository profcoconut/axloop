# axloop

Autonomous AI-driven DevOps team skill for Claude Code. Models a real software team with PM, Designer, 4 Developers, QA, and DevOps — all coordinated via SendMessage.

Built for long-running autonomous improvement cycles on VoxParty, but framework-agnostic.

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
                    Dev1, Dev2, Dev3, Dev4  (4 parallel)
```

- **PM**: owns what to build, prioritizes backlog
- **Designer**: produces UI/UX specs before implementation
- **Manager**: decides how, assigns work, coordinates all agents
- **Dev1-4**: parallel developers, PR-based workflow, TDD
- **QA**: regression tests + design verification gate
- **DevOps**: build gate, deploy pipeline, smoke tests, rollback

## Communication

All agents communicate via `SendMessage`. Files are for state persistence and human visibility only.

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
| rapid | 4 parallel | optional | specific test only | 120s |
| quality | 1 at a time | required | full regression suite | 60s |
| auto | 4 parallel | optional | quality when regressions exist | 60s |

## Requirements

- Claude Code session
- Git repository with `feat/engine` branch or equivalent
- `gh` CLI for PR operations
- Memory path configured in SKILL.md

## Skill Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
- Requirements: `docs/brainstorms/parallel-axloop-requirements.md`
