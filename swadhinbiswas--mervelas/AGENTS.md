# Mervelas Developer Guidelines

Welcome to the Mervelas codebase! Mervelas is a fast, local-first AI coding CLI written in React (Ink) and compiled with Bun. Follow these specific strategies to stay productive and aligned with project architecture:

## 1. Project Architecture & Tooling

- **Custom UI System:** Mervelas uses a highly custom, optimized implementation of Ink for rendering the CLI interface.
  - **CRITICAL:** **Never import `ink` directly from npm.** Always import layout components (`Box`, `Text`) and hooks from `src/ink.ts`, `src/ink.tsx`, or `src/ink/index.ts`.
- **AI Integrations:** The codebase speaks to LLM APIs (OpenAI, OpenRouter) and defines agent capabilities using `@anthropic-ai/sdk` and Model Context Protocol (`@modelcontextprotocol/sdk`).
- **Bridge Sync/Daemon:** The `src/bridge/` folder abstracts away remote daemons, environments (Chromium, bash tasks), and external environment APIs.
- **Build Workflows:** We use `bun` as the package manager and bundler. The build script (`scripts/build.ts`) uses extreme size optimization strategies like module stubbing (`namespace: text-stub`, `namespace: native-stub` or internal feature stripping). Keep dependencies lightweight. 

## 2. Developer Constraints & Conventions

- **CLI Startup Speed & Lazy Loading:**
  - Avoiding top-level side effects is paramount. Do not execute heavy logic, file reads, or instantiate state singletons on load.
  - See `src/main.tsx` for usage of `profileCheckpoint(...)` to measure startup bounds.
  - **Commands (defined in `src/commands/...`) should be lazy-loaded:** The `load: () => import('./my-command.js')` pattern keeps startup overhead strictly on a "need-to-execute" basis.
- **Agent Tools (`src/tools/...`):** Tools are structured as React components that implement both the AI execution logic (often matching an MCP schema) and an interactive CLI UI/result view (e.g., `BashTool.tsx`, `BashToolResultMessage.tsx`).
- **Formatting:** Keep in mind the codebase uses Biome rules in certain places (`// biome-ignore-all ...`). Follow strictly the formatting present in the file you edit.

## 3. Recommended Workflows

- **Building the Extension:** `bun run build`. (Also used via `npm run build` or `npm start` over `dist/cli.mjs`).
- **Testing Logic:** Test critical logic via `bun test src/...` and catch type errors using `npm run typecheck` (`tsc --noEmit`).
- **Performance Tooling:** Since `.perf-artifacts/` are relied upon, any changes to slow workflows like startup execution or initial queries MUST be profiled using:
  - `bun run perf:startup --` (Check Idle CPU, RSS memory)
  - `bun run perf:query --` 
  - (Check `PERFORMANCE.md` for our specific baselining methodology before raising performance-impacting UI elements).

---
> Source: [swadhinbiswas/Mervelas](https://github.com/swadhinbiswas/Mervelas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-23 -->
