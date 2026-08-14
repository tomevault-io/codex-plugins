
# Packages (`packages/`)

- **Shared types/utilities**: `@titanclip/shared`. **Schema/migrations**: `@titanclip/db`. **Adapter glue**: `@titanclip/adapter-utils` and `packages/adapters/*`.
- **Plugins**: follow `docs/plugin-authoring-guide.md` and `packages/plugins/sdk`. Example plugins under `packages/plugins/examples/*` are **reference only**, not the production app.
- When changing a package API consumed by `server` or `ui`, run **`pnpm run typecheck`** from the repo root to catch cross-package breaks.

---
> Source: [CES-Ltd/TitanClip](https://github.com/CES-Ltd/TitanClip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-14 -->
