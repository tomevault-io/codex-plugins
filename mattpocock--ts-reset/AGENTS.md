Use pnpm

Each entrypoint must be in `src/entrypoints`, and have an entry in `package.json#exports`. It may optionally be added to `src/recommended.d.ts`.

When typechecking, always run `pnpm typecheck`. This runs against the entire repo, which is good for testing regressions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattpocock)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mattpocock)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
