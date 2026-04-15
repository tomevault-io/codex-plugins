
# Cloudflare Pages + Static SvelteKit

## Static build expectations
- Prefer prerendered routes.
- Avoid Node-only APIs at runtime.
- Keep environment usage compatible with Cloudflare Pages build/runtime.

## When adding routes
- Ensure new routes can be prerendered or clearly document why not.
- For dynamic routes, provide a strategy (e.g., known entries list or CMS-driven build-time generation).

## Build behavior
- Keep adapter/static conventions intact.
- Avoid introducing server endpoints unless the project explicitly supports them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/copa-airlines-digital-marketing)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/copa-airlines-digital-marketing)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
