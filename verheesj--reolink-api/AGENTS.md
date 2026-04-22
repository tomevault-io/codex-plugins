## Copilot / Agent guidance (concise)

This repo is a small TypeScript SDK + CLI for Reolink devices. Key facts an automated agent must know to be productive:

- Core contract: `ReolinkClient` (src/reolink.ts) is the single HTTP client. Feature modules (e.g. `src/stream.ts`, `src/snapshot.ts`, `src/ptz.ts`, `src/events.ts`) accept the client as the first argument or call `client.api(...)`.
- Auth modes: two modes — `mode: "long"` (token session with auto-refresh; see `login()` and token expiry logic) and `mode: "short"` (per-request user/pass). Token errors return rspCode -1 or HTTP 401; client retries by re-login.
- API shape: requests are POSTs to `/cgi-bin/api.cgi?cmd=...` with body `[{cmd, action, param}]`. Responses: `[{code:0, value:...}]` or `[{code:N, error:{rspCode,detail}}]`. Snapshot (`Snap`) is a GET returning JPEG bytes (see `src/snapshot.ts`).
- Channel numbering: API calls use zero-based channels. RTSP URLs pad channels as `01` (see `src/stream.ts`). Be careful when generating stream identifiers.
- CLI dispatcher: `src/cli.ts` parses args, falls back to env vars (`REOLINK_NVR_HOST`, `REOLINK_NVR_USER`, `REOLINK_NVR_PASS`), creates client, optionally logs in, executes command, then closes client. Binary commands (snap) write bytes to stdout and log to stderr.
- Exports: `package.json` contains granular exports — if you add a new public module, update the `exports` map and the `bin` if the CLI needs it.
- Tests: Vitest is used. Tests live next to sources (`*.test.ts`). Common patterns: mock `global.fetch`, use `vi` fake timers and `vi.advanceTimersByTimeAsync()`, use `Buffer.from([0xff,0xd8])` for JPEGs. Coverage target ~80%.
- Platform & build: ESM-only, Node >= 18 (native fetch), build via `npm run build` (tsc). Helpful scripts: `npm test`, `npm run test:coverage`, `npm run lint`, `npm run typecheck`.

Quick examples an agent may need to generate or modify code:

- Call API from module:
  - `await client.api("GetEnc", { channel: 0 });`
- Snapshot handling: check JPEG magic bytes `0xFF 0xD8` and convert ArrayBuffer → Buffer before writing to disk or stdout.

When adding features:
- Update `src/*` module, add/adjust types in `src/types.ts`, add unit tests beside the module, and update `package.json` `exports` for public modules. If CLI user-facing, add a handler in `src/cli.ts`.

Debugging and logs:
- Use `debug: true` in client options or `--debug` CLI flag for request/response dumps to stderr. Use `DEBUG=reolink:module` for module-scoped debug.

Files to read first when changing behavior: `src/reolink.ts`, `src/cli.ts`, `src/stream.ts`, `src/snapshot.ts`, `src/events.ts`, and `package.json`.

If this summary misses any specific workflows you'd like covered (examples, common PR checks, or CI), tell me which area to expand and I will iterate.
```

---
> Source: [verheesj/reolink-api](https://github.com/verheesj/reolink-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
