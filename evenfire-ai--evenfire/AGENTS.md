# Copilot Instructions

When you edit files under `control-ui/**`, `desktop-app/ui/**`, or `profile-ui/**`, apply the shared UI rules in [`docs/agents/frontend-style-rules.md`](../docs/agents/frontend-style-rules.md).

For `control-ui/**` and `profile-ui/**`, sidebar destinations and shareable tab-like sections must use canonical Next App Router paths and child routes, not `?tab=...`, `profileTab`, `localStorage`, or component-only state as routing.

Those instructions are scoped to those three frontend projects only. Do not assume they apply to the rest of the repository.

---
> Source: [evenfire-ai/evenfire](https://github.com/evenfire-ai/evenfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-21 -->
