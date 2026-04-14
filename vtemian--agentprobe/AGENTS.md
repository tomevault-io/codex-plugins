
# Provider Pipeline

- Follow provider contract shape exactly: `discover -> read -> normalize`.
- `discover` must return `{ inputs, watchPaths, warnings }`.
- `read` must return raw records + health; do not canonicalize here.
- `normalize` must output canonical `{ agents, health }`.
- Parse external data defensively (`safeParse`), accumulate warnings, continue on bad records.
- Keep status names consistent: `running | idle | completed | error`.
- Put shared tunables in `constants.ts`; keep file-local constants at module top.
- Prefer options interfaces with defaults via `??`; use explicit `false` to disable optional features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vtemian)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vtemian)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
