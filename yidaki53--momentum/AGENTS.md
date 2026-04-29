# Momentum agent instructions

## Hard rule
- Never commit or push changes.

## Command execution
- Use Poetry-first Python commands (`poetry run ...`) for linting, typing, tests, scripts, and app entry points.

## Cross-surface settings parity
- Desktop UI settings and mobile app settings must stay behaviorally aligned.
- Any setting added, removed, renamed, or changed on one surface must be reviewed and updated on the other surface in the same change unless the user explicitly asks for a temporary exception.
- Treat settings parity as applying to both persistence and visible UI behavior: labels, defaults, effects, and runtime application should match across desktop and mobile.

## Mobile UI learnings
- On Android/Kivy accordion-style sections, collapsed children must also shrink to zero height and be disabled; hiding only the parent container is not sufficient to keep touch behavior reliable.
- Keep passive encouragement or status copy in a stable always-visible area near the bottom of the home screen rather than burying it inside a collapsible section.
- Show ACT-specific controls only when the assessment-derived support threshold says they are needed; otherwise remove the affordance entirely instead of showing a disabled tease.

## Instruction assets index
- Core rules: `.github/instructions/01-core-rules.md`
- Architecture and Cython learnings: `.github/instructions/02-architecture-and-performance.md`
- Release and versioning policy: `.github/instructions/03-release-and-versioning.md`
- Android release safety: `.github/instructions/04-android-release-safety.md`
- Validation gates: `.github/instructions/05-validation-gates.md`
- CI and runtime operations: `.github/instructions/06-ci-and-runtime.md`
- Runtime diagnostics playbook: `.github/instructions/07-runtime-diagnostics.md`

## Prompt assets index
- Feature workflow: `.github/prompts/feature-workflow.prompt.md`
- Cross-surface parity: `.github/prompts/cross-surface-parity.prompt.md`

## GitHub workflow notes
- For this repository, use the `yidaki53` GitHub account for `gh` and git-authenticated GitHub operations.
- If a push updates `.github/workflows/*`, ensure the active token has `workflow` scope before pushing.
- If push is rejected with OAuth workflow-scope errors, refresh token scope before retrying.

---
> Source: [yidaki53/momentum](https://github.com/yidaki53/momentum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
