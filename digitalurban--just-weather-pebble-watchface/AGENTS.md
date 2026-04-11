## Project snapshot (what this is)

- Pebble SDK v3 watch app (native C) with a companion phone JS app.
- Watch code: `src/c/myfirstproject.c` — UI, clock, and AppMessage handlers.
- Phone JS: `src/pkjs/app.js` (entry via `src/pkjs/index.js`) — fetches Open-Meteo, reverse-geocodes, and sends messages to the watch.
- Build glue: `wscript` (uses `ctx.load('pebble_sdk')`) and the Pebble bundle pipeline. JS bundling produces `build/pebble-js-app.js` (see generated `build/webpack/pkjs/webpack.config.js`).

## Quick dev workflow (how to build/run/debug)

- Build the app (requires Pebble SDK / waf wrapper): use the Pebble CLI in your environment, e.g. `pebble build` from repo root. This runs the `wscript` which calls `pbl_build`/`pbl_bundle`.
- Install to a connected device or emulator: `pebble install --phone <IP>` or via your Pebble development tools.
- View device logs / APP_LOG output (C) and JS console logs: `pebble logs` (the SDK routes APP_LOG and Pebble JS console messages).
- JS bundling entry: `src/pkjs/index.js` requires `./app` (so modify `src/pkjs/app.js` for phone behavior). The bundler writes `build/pebble-js-app.js` used at runtime.

## Important code patterns and conventions (project-specific)

- Message keys: numeric mapping is in `js/message_keys.json` and generated artifacts live under `include/message_keys.auto.h` and `src/message_keys.auto.c`. JS prefers `require('message_keys')` but falls back to numeric keys if missing (see `src/pkjs/app.js` top).
- AppMessage sizes: the native app opens AppMessage with `app_message_open(2048, 512)` — expect larger inbox buffer (2KB) and smaller outbox (512B). Keep messages small but expect the code accepts ~2KB payloads.
- JS networking: uses XHR with explicit timeouts (no `fetch`) and Open-Meteo endpoints. Tests/emulator runs may need network access; the code gracefully falls back to hardcoded coords if geolocation fails.
- UI: watch uses text-only conditions (no icons). Layout is platform-conditional via preprocessor directives like PBL_PLATFORM_BASALT in `src/c/myfirstproject.c` — adjust carefully for platform variants.
- Multi-JS builds: package.json sets `pebble.enableMultiJS = true`. The `wscript` pbl_bundle includes `src/pkjs/**/*.js` and `js_entry_file='src/pkjs/index.js'` — maintain that layout when adding JS modules.

## Where to change common settings

- App metadata and target platforms: `package.json` → `pebble` section (displayName, uuid, sdkVersion, targetPlatforms).
- Add native C source: `src/c/**/*.c`; `wscript` already includes `src/c/**/*.c` for building.
- Add companion JS modules: put under `src/pkjs/` and ensure `index.js` requires the top-level entry module.

## Integration points and gotchas for agents

- When editing message keys, update `js/message_keys.json` and regenerate (build will create `include/message_keys.auto.h` and `src/message_keys.auto.c`). JS code tolerates missing generated module by falling back to numbers — but prefer editing the JSON and letting the build produce headers.
- The JS app uses reverse geocoding (Open-Meteo). Avoid changing network behavior without adding timeouts — code expects XHR with 8–10s timeouts and handles failures by sending fallback data.
- The C app expects specific numeric keys (see the define list at top of `src/c/myfirstproject.c`). If you rename keys, ensure both JSON and native defines align (or rely on generated headers).
- Logs: use `APP_LOG(...)` in C and `console.log(...)` in JS to help trace; `pebble logs` captures both.

## Quick examples to cite in edits

- Message handling (native): see `inbox_received_callback` in `src/c/myfirstproject.c` — uses `dict_find(iterator, MESSAGE_KEY_PRESSURE)` and updates `s_pressure_layer`.
- JS key fallback: in `src/pkjs/app.js` the code does `MessageKeys = require('message_keys');` in a try/catch and falls back to literal indices (0..6).
- Bundler entry: `wscript`'s `ctx.pbl_bundle(..., js_entry_file='src/pkjs/index.js')` — don't change the entry filename unless you update the wscript.

## If you're an AI agent editing this repo — checklist

1. Read `package.json` and `wscript` first to learn build targets and js entry file.
2. When changing message keys, update `js/message_keys.json` and run a build to regenerate headers. Do not hardcode mismatched keys between JS and C.
3. Preserve AppMessage buffer sizes unless you audit memory: large inbox (2048) is intentional.
4. Prefer small, idempotent changes — this repo is a simple watch+phone app; major refactors of the JS network flow or C UI require manual device testing.

---
If any part of the workflow is unclear (CI hooks, tests, or preferred emulator commands), tell me which area you'd like expanded and I will iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digitalurban)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/digitalurban)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
