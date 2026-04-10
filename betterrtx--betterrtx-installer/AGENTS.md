
# Project Rules — BetterRTX Installer (Tauri v2)

- Use **Tauri v2** APIs and docs; prefer NSIS installer on Windows.
- Frontend: React + TS; strict mode; Tailwind CSS v4; keep UI edits isolated. Reference Tailwind @docs
- Rust core: small commands with `#[tauri::command]`; pure helpers.
- use context7 to referecne Tauri docs
- Test with `bun run tauri dev` from the `.v3/` directory. Do not use NPM.
- Prefer Tauri plugins over implementing own features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BetterRTX)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/BetterRTX)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
