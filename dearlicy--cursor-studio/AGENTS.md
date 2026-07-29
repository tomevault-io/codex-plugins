# Cursor Studio Repository Instructions

- Cursor Studio is a Windows desktop application built with React, TypeScript,
  Vite, and Electron.
- Follow the established components, design tokens, and interaction patterns.
  Keep user-facing copy clear, concise, and suitable for a desktop product.
- Keep changes focused. Do not introduce generated output, local configuration,
  dependencies, installer files, or diagnostic artifacts into source commits.
- Prefer the existing data stores and IPC boundaries over new cross-layer
  shortcuts. Preserve error handling and loading states in user-visible flows.
- Before requesting review, run `npm run typecheck` and `npm run build` from
  the repository root. Run a focused smoke test when the affected area has one.
- Changes that affect packaging or updates must retain the Windows MSI release
  path and keep version metadata consistent across the application and package.

---
> Source: [DearLicy/cursor-studio](https://github.com/DearLicy/cursor-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-25 -->
