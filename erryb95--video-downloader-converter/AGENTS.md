<!-- Workspace instructions for Copilot. See
https://code.visualstudio.com/docs/copilot/copilot-customization -->

## Project: Video Downloader + Converter

A Windows desktop app that downloads video and audio from the sites yt-dlp
supports, and converts files the user already has.

- **Frontend**: React + TypeScript. Plain CSS with custom properties for
  theming — there is no CSS framework. Tailwind was configured once but never
  imported, so every utility class in the markup was inert; it has been removed
  and should not come back without the stylesheet to go with it.
- **Backend**: Rust via Tauri 2, driving bundled `yt-dlp` and `ffmpeg`
  sidecars.

### Layout

- `src-tauri/src/commands.rs` — every `#[tauri::command]` the UI calls
- `src-tauri/src/utils.rs` — sidecar paths, parsing, error explanations
- `src/components/` — download, batch, converter, history, updates
- `scripts/fetch-binaries.mjs` — downloads the sidecars; they are not in the
  repository

### House rules, learned the hard way

- **There is exactly one place that finds ffmpeg**, in `utils.rs`, and it
  returns an error rather than the bare name `"ffmpeg"`. A second copy of that
  function once shadowed it and every merged download came out silent while the
  test suite stayed green. A test fails the build if a second one appears.
- **Never pin yt-dlp's `player_client`.** YouTube breaks individual clients on
  its own schedule; a hardcoded list rots silently and has broken downloads
  outright before.
- **A single video format is never downloaded on its own.** YouTube's format
  table is almost all DASH, and DASH video carries no audio, so a lone format id
  is resolved to `<id>+bestaudio` first.
- **Do not report success from an exit code.** Probe the file that was
  produced. yt-dlp exits 0 having merged nothing.
- **Verify changes by running the built binary**, not by driving yt-dlp and
  ffmpeg from the repository — the app resolves its tools relative to the
  running executable, so those are not the same thing:
  `video-downloader.exe --selftest <url> <dir>`, or `npm run verify`.
- Keep a `MutexGuard` in a scope. An unscoped one deadlocked the whole subtitle
  stage for months.

---
> Source: [Erryb95/video-downloader-converter](https://github.com/Erryb95/video-downloader-converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-25 -->
