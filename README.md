# @turystack/proof-mode

Proof-mode skill — the delivery harness for one task: it resolves the task's context, routes to the pattern skills, runs the gate ladder per slice and closes with an evidence-backed report attached to the board task it came from. `turystack-harness` is what decides the session and hands the task over. Installed into .claude/skills and/or .codex/skills via the turystack CLI.

## Installation

```bash
pnpm add -D @turystack/proof-mode
```

## Contents

- [Proof mode — overview](00-overview.md)
- [Context — what the task is built on](01-context.md)
- [Plan — specs before code](02-plan.md)
- [Build — slices and the gate loop](03-build.md)
- [Gates — the ladder](04-gates.md)
- [Evidence — what has to be shown](05-evidence.md)
- [Delivery — the report and the handoff](06-delivery.md)
- [Skill manifest](SKILL.md)

## Where it sits

```text
turystack-harness                                 ← before a task exists: which mode, which task
└── turystack-proof-mode                          ← this skill: when, and what proves it
    ├── turystack-architecture-pattern            the law
    ├── turystack-backend-pattern                 backend mechanics
    ├── turystack-frontend-pattern                frontend mechanics
    └── turystack-frontend-primitives-pattern     how one UI primitive is written
```

`@turystack/proof-mode-gates` runs the checks the ladder names.

## Documentation

**https://tury.dev/libs/proof-mode**
