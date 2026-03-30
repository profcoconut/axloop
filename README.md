# axloop

Autonomous DevOps team that runs forever.

## Philosophy

**The loop runs until the goal is reached — not until a session dies, a skill asks a question, or a timeout hits.** Session boundaries are irrelevant. Workers die and get restarted. Manager session ends and state files carry the work forward. The only exit is a manual signal from outside.

Every sprint enforces the same skills in the same order. No skipping. No shortcuts. The discipline is built into the loop itself.

## Start

```
/axloop <doc> [rapid|quality|auto]
```

## How it works

**Manager** — spawns workers, enforces skills, resolves conflicts. Never touches code.

**Workers** — do the work. Report to Manager only.

Every sprint runs: plan → implement → review → test → ship → document. Each skill completes before the next starts. Sprint ends and the next one begins immediately.

## Skills enforced

investigate · simplify · review · qa · browse · ship · design-review · retro · ce-plan · ce-review · ce-compound

## Stop

```bash
echo "stop" > worktree/.project-loop-state
```

## Location

- Project-level: `.claude/skills/axloop/SKILL.md`
- Source: `~/Documents/skills/Axloop/`
