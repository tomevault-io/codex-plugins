# Copilot / AI Agent Instructions for Pluely

This file contains concise, actionable guidance for AI coding agents working in this repo.

1) Big picture
- Frontend: React + TypeScript (Vite). Entry: [src/main.tsx](src/main.tsx#L1).
- Backend: Tauri (Rust) in `src-tauri/` with Rust entry [src-tauri/src/main.rs](src-tauri/src/main.rs#L1).
- IPC: frontend uses `@tauri-apps/api` `invoke` and `listen` to call Rust commands (see [src/contexts/app.context.tsx](src/contexts/app.context.tsx#L1) for examples).
- Storage & DB: uses `safeLocalStorage` helpers and Tauri SQL plugin (preload `sqlite:pluely.db` in `src-tauri/tauri.conf.json`).

2) Key workflows / commands
- Dev (frontend): `npm run dev` (Vite) — Vite is configured to run on port 1420 for Tauri dev. See `vite.config.ts`.
- Build (frontend): `npm run build` then `npm run tauri` (Tauri CLI). `tauri.conf.json` defines `beforeDevCommand` and `beforeBuildCommand`.
- Run packaged: use the Tauri build workflow (`npm run tauri` / `tauri build`).

3) Project conventions & patterns
- Path alias: `@` → `src/` (see `vite.config.ts`). Use `@/components`, `@/hooks`, `@/contexts`, `@/lib`.
- Centralized contexts: app-wide state lives under `src/contexts` (notably `AppProvider` in [src/contexts/app.context.tsx](src/contexts/app.context.tsx#L1)). Prefer adding shared logic to contexts or `src/lib` helpers.
- Hooks: domain logic lives in `src/hooks` (e.g., `useCompletion` implements AI request streaming and abort logic). When adding behavior prefer hooks + small presentational components.
- Streaming AI responses: `fetchAIResponse` is used as an async iterator in `useCompletion` to stream chunks — preserve the request-id / abort pattern when modifying.

4) Integration details & IPC examples
- Frontend invokes Rust commands with `invoke('command_name', { ... })`. Example: `invoke('set_app_icon_visibility', { visible })` (see [src/contexts/app.context.tsx](src/contexts/app.context.tsx#L1)).
- Listen to Tauri events with `listen('event-name', handler)`; the app listens for `handle-app-icon-on-hide` / `handle-app-icon-on-show`.
- When changing an IPC name, update both Rust handlers and all `invoke`/`listen` usages.

5) Storage & provider patterns
- Custom AI/STT providers are stored as curl-style entries saved in localStorage keys (see `STORAGE_KEYS` usage in `app.context`). The code validates curl strings with `@bany/curl-to-json`.
- Keys live in `src/config` and storage helpers in `src/lib/storage`. Prefer using those helpers for read/write.

6) Build & dev gotchas
- Vite server MUST run on port 1420 with `strictPort: true` for Tauri dev; do not change port without updating `tauri.conf.json`.
- `vite.config.ts` ignores `src-tauri` from watcher to avoid accidental rebuilds.
- To debug native side, run `cargo run` / `tauri dev` from `src-tauri` context; ensure frontend is reachable at the `devUrl`.

7) Where to make common changes
- UI: `src/components/*` and `src/components/*/index.tsx`.
- App-wide logic: `src/contexts/*` (settings, theme, app state).
- Business logic + utils: `src/lib/*` (API wrappers, db actions, helpers).
- Persistent data: `src-tauri` (Rust) and SQL plugin preload (`pluely.db`).

8) Tests & linting
- This repo does not include a test suite in the workspace root. Follow existing patterns (hooks → unit logic in `src/lib`) if adding tests.

9) Quick examples for agents
- To add a new IPC command: add Rust handler in `src-tauri/src/*`, expose via Tauri, then call from frontend via `invoke('your_command', payload)`.
- To stream AI responses: model `useCompletion` — create a streaming generator in `src/lib/fetchAIResponse` and consume with the same abort/request-id approach.

If anything above is unclear or you'd like more examples (e.g., specific `invoke` usages, or a mapping of storage keys), tell me which area to expand. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vcode2407)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Vcode2407)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
