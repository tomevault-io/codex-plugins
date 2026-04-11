# Scope
- Do NOT modify UI, CSS, images, or assets.
- Only touch:
  - src/core/event-bus.js (VITE_API_BASE wiring & SSE URL)
  - vite.config.js (dev proxy only)
  - .env.local.example
  - scripts/test/vercel-health-check.js
  - .github/workflows/*
- Use Node 20. No TypeScript in scripts/test/.
- All API calls & SSE must respect VITE_API_BASE if defined; fallback to window.location.origin.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bomussa)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Bomussa)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
