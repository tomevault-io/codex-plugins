# PortfolioConstruct UI — Copilot Instructions

## Project Overview
Bloomberg-inspired portfolio construction and statistical analysis platform.
Built with React 19 + TypeScript (strict) + Vite + Zustand.

## Architecture Rules
- **3-Tier Strict**: Presentation → Logic → Data. Never mix layers.
- **Feature-Slicing**: Code organized by `/features/{name}/`, not by technical type.
- **Composite Pattern**: Every widget is a `<Panel>` container — standalone + embeddable.
- **Design Tokens ONLY**: All visual values via CSS Custom Properties. No hardcoded HEX/px.
- **State Separation**: UI state, workspace state, data state, analysis state — never mixed.

## Folder Structure
```
src/
  domain/          # Pure TypeScript types — no React, no side effects
  stores/          # Zustand stores, one per state domain
  components/      # Shared UI primitives (Panel, Button, etc.)
  features/        # Feature slices (data-explorer, analysis, portfolio, reporting)
  shell/           # App shell layout (TopBar, Sidebar, StatusBar, MainContent)
  design-tokens/   # CSS variables, themes
```

## Key Principles
- Buttons live in Panel header (secondary) or footer (primary CTA only)
- No nested vertical scrollbars — one global scroll context
- Panels have min-height, never collapse empty
- Numbers: monospace, right-aligned, tabular-nums, consistent decimals
- Colors are semantic (green=positive, red=negative, yellow=warning, blue=action)
- No decorative animations — only functional transitions
- Every analysis must show generated code (reproducibility)
- Keyboard-first design (shortcuts, Tab navigation)
- Progressive disclosure for advanced parameters

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prattpiss)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/prattpiss)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
