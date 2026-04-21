# Copilot instructions for `bilibili-tui`

## Build, test, and lint commands

Use Cargo commands directly from the repository root:

- Build (dev): `cargo build`
- Build (release): `cargo build --release`
- Run app: `cargo run`
- Fast compile check: `cargo check`
- Lint (warning-free policy): `cargo clippy -- -D warnings`
- Format: `cargo fmt`
- Full test suite: `cargo test`
- Run a single test: `cargo test api::wbi::tests::test_encode_wbi`

Pre-commit is configured and mirrors project checks:

- Run all hooks: `pre-commit run --all-files`

Runtime tools expected by the app (from project docs): `mpv` and `yt-dlp`.

## High-level architecture

- Entry point is `src/main.rs`, which initializes Ratatui + mouse capture and runs `App::run()` from `src/app/mod.rs`.
- `App` is the central state machine:
  - Owns global state (`current_page`, `api_client`, credentials, sidebar/theme/keybindings, cached home page).
  - Runs the event loop (keyboard + mouse + periodic `tick()`).
  - Routes page intents via `AppAction` (`src/app/action.rs`) in `handle_action()`.
  - Performs async page initialization in `init_current_page()` whenever switching pages.
- UI uses a component model in `src/ui/mod.rs`:
  - Every page implements `Component` (`draw`, input handlers, optional mouse handling).
  - `Page` enum holds all concrete pages (`Login`, `Home`, `Search`, `Dynamic`, `History`, `Live`, detail pages, `Settings`).
- Networking is centralized in `src/api/client.rs`:
  - Shared `reqwest::Client` with fixed Bilibili headers (`User-Agent`, `Referer`).
  - Cookie and WBI key state are kept in `RwLock`s.
  - Feature APIs (search/recommend/video/comments/live/etc.) are methods on `ApiClient`.
- Live streaming has a separate stack:
  - `ApiClient::get_danmu_info()` fetches signed WS connection metadata.
  - `src/api/live_client.rs` manages WS lifecycle + heartbeat.
  - `src/api/live_ws.rs` handles packet encode/decode, zlib/brotli decompression, and message parsing.
- Media playback is in `src/player/mod.rs`:
  - Launches `mpv` asynchronously.
  - Exports cookies for authenticated playback.
  - Sends watch heartbeat/start/end events through heartbeat APIs.
- Persistence is in `src/storage/mod.rs`:
  - Reads/writes `~/.config/bilibili-tui/{credentials.json,config.json}`.
  - Stores theme + keybindings in `AppConfig`.

## Key conventions in this repository

- Page-to-app communication is action-based: pages should return `Option<AppAction>` instead of directly mutating global app state.
- New page-level async loading should follow the existing lifecycle split:
  - initial load in `App::init_current_page()`
  - background/non-blocking updates in `App::tick()`.
- Input handling should use `Keybindings` matcher helpers (`matches_*`) instead of hardcoded `KeyCode` checks to preserve user-configurable bindings.
- Home page state is intentionally cached when navigating to detail pages and restored on return (`App.cached_home`); avoid changes that force unnecessary home reloads.
- Image-heavy pages use non-blocking cover loading with `tokio::spawn` + mpsc channels (`start_cover_downloads` / `poll_cover_results`) to keep the TUI responsive.
- API wrappers consistently deserialize `ApiResponse<T>` and either:
  - return explicit errors for required payloads, or
  - provide typed empty defaults for list-style responses.
- WBI signing is mandatory for WBI endpoints (search/recommend and live danmu info); preserve signing logic and the retry-on-`-352` refresh flow in `get_danmu_info()`.
- UI copy and labels are predominantly Chinese; keep new user-facing text consistent with that localization style.

---
> Source: [MareDevi/bilibili-tui](https://github.com/MareDevi/bilibili-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-21 -->
