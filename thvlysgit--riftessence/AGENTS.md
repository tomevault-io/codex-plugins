# Copilot Workspace Instructions

Last updated: 2026-04-16

## Project Operating Rules

- Treat Documentation as canonical source of truth.
- Keep architecture boundaries strict:
  - apps/web must not import from apps/api internals
  - apps/api must not import from apps/web internals
- Prefer small, reversible changes over large rewrites.
- After meaningful code changes, update documentation in the same session.

## Required Validation Before Hand-off

- Run relevant build/type checks for touched areas.
- If checks fail, fix or document why unresolved.
- Summarize what was validated and what was not.

## Documentation-Live Rule

For meaningful work, update at least one of:

- Documentation/project/changelog.md
- Documentation/analysis/*.md
- Documentation/architecture/*.md
- Documentation/frontend/*.md
- Documentation/backend/*.md

If docs are intentionally postponed, add a tracked task to Documentation/PENDING_TASKS.md.

## Vibe Coding Mode

When user gives rough intent:

1. Convert intent into concrete scope and acceptance criteria.
2. Implement in phased slices.
3. Validate each slice.
4. Keep docs synchronized after each slice.

Preferred orchestration entrypoint:

- Use /vibe-build or @RiftConductor for one-prompt end-to-end execution.

## No-Assumption Mode

- Do not assume hidden requirements when intent is ambiguous.
- Ask short, high-signal questionnaires to refine constraints.
- After requirements are clear, continue autonomously to minimize user prompt load.

## Reliability Priority

- Optimize delivery for both Vercel frontend and Raspberry Pi dockerized API operations.
- Favor production stability over risky broad rewrites.

## Escalation Rule

If risk is high (auth, data loss, production stability, migrations), stop and ask for explicit confirmation before destructive or irreversible steps.

---
> Source: [thvlysgit/riftessence](https://github.com/thvlysgit/riftessence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
