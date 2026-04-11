## hdisplay – Copilot developer instructions

Purpose: This file teaches GitHub Copilot how to work as a contributor in this repository. It captures the tech stack, run/test commands, coding conventions, and the safe ways to add features, fix bugs, and update docs.

Audience: Automated assistants and humans alike. Be concise, deterministic, and verify with tests where possible.

---

## Quick facts

- Runtime: Node.js >= 20 (Docker uses Node 20 alpine)
- Web: Express 4 + Socket.io 4 (server) + vanilla JS client
- CLI: Commander.js, Axios, Chalk
- Tests: Jest + Supertest (API/unit), Playwright (E2E, optional for captures)
- Media/capture: Playwright, Sharp, ffmpeg (optional for MP4)
- State: JSON file at `data/state.json` (no DB)
- Templates: Simple HTML files in `templates/` with `{{placeholder}}` replacement, optional validators

## Commands you should run

- Start server: npm start
- Dev preview (macOS): ./scripts/mac-preview.sh
- CLI from repo: npm link, then hdisplay ...
- Unit/integration tests: npm test
- Playwright E2E: npm run test:e2e (ensure a server is running; default Playwright baseURL is http://localhost:3100)
- Lint: npm run lint
- Format: npm run format
- Docker: docker compose up --build
- Captures: hdisplay capture:all and hdisplay capture:gallery

Notes

- Keep ports consistent: server defaults to 3000; E2E config defaults to 3100 via BASE_URL.
- Uploads and data are persisted under ./uploads and ./data; Docker maps these as volumes.

## Repository structure

- server/ – Express app (CJS entry at server/index.js for tests; ESM entry at server/index.mjs), static client in server/public/
- cli/ – CLI shim at cli/index.js (CJS) delegates to ESM cli/index.mjs; subcommands
- templates/ – HTML templates; validators under templates/\_validators/
- capture/ and capture-profiles/ – Black-box capture system
- test/ – Jest tests; tests-e2e/ – Playwright tests
- uploads/, data/ – runtime persistence

## Coding conventions and constraints

- Language/style
  - Prefer ESM (import/export) for all new code; Node 20+ is the baseline.
  - Use CJS only where compatibility requires it (e.g., Jest require() in tests, npm bin shims). In those cases, create a tiny CJS shim that dynamically imports the ESM module and delegates.
  - No TypeScript unless the repo adds it explicitly.
  - Prefer small, widely used dependencies; pin versions and respect existing package.json patterns.
  - Keep changes minimal and localized; avoid broad refactors in feature PRs.
- APIs and compatibility
  - Don’t break the public HTTP API or CLI flags without updating README and tests.
  - Playlist behavior is intentional: rotation, overrides, delay clamping, and persistence to data/state.json.
  - Keep crossfade behavior client-side intact (no server coupling to transitions).
  - Assume Node 20+ is available; do not add back-compat shims or polyfills for older Node versions (e.g., fetch/polyfill, chalk CJS fallbacks).
  - ESM-first policy: prefer .mjs (or package-level type: module in the future); if CJS consumers need stability, provide a minimal CJS shim that imports and re-exports the ESM surface without duplicating logic.
- Security and safety
  - Service is LAN-only by design. Do not add auth unless asked; do add minimal headers sensibly if working on hardening.
  - Never ship secrets in code; prefer env vars.
- Performance
  - Raspberry Pi friendly: avoid per-frame allocations; keep CPU modest.
- Tests/docs
  - Update or add Jest tests when changing server behavior or CLI parsing.
  - If you add/modify templates, add or update validator tests in test/.
  - If you update screenshots/videos in README, regenerate with capture commands.

Project decisions (maintainer-provided)

- Lint/Prettier: Check in ESLint and Prettier configs and keep them authoritative.
- Playwright: Don’t modify Playwright config or tests right now.
- Dependencies: Ask before adding any new dependencies (runtime or dev). Keep the current set unless approved.
- CI: Don’t add GitHub Actions/CI until requested.
- Captures: It’s fine to commit refreshed captures under `captures/`.

## Template system contract

- Files: templates/<id>.html
- Placeholders: {{ key }} and dotted paths (e.g., {{ a.b }})
- Validators (optional but recommended):
  - Preferred locations: templates/\_validators/<id>.js or templates/<id>.validator.js
  - Return shapes: true | { ok: true } for success; { ok: false, error } for failure
- Reserved parameter names: top-level data keys must not collide with CLI globals: server, timeout, quiet, help, h, data, data-file (and dataFile)
- API to apply: POST /api/template/:id { data }

## Template implementation best practices (learned lessons)

### Title Tag Requirements

- **CRITICAL**: Template `<title>` must match format `hdisplay - {template-id}` exactly
- Template ID must match filename (e.g., `stock-ticker.html` → `hdisplay - stock-ticker`)
- Do NOT dynamically update `document.title` in JavaScript - this causes capture timing issues
- Capture mechanism waits for this exact title format within 4-second timeout

### API Integration Patterns

- Follow weather.js pattern for API modules: provider abstraction, rate limiting, error handling
- Place API modules in `server/` directory (e.g., `server/stocks.js`, `server/weather.js`)
- Use `config.json` for API keys: `config.apiKeys.{provider}`
- Implement graceful fallbacks for API limitations (free tier restrictions)
- Use axios with timeout: `{ timeout: 10000 }` for external API calls

### Data Validation Alignment

- **CRITICAL**: Validator and template must expect same data format
- Common mistake: validator expects array, template uses string (or vice versa)
- Example: `symbols: ['AAPL', 'GOOGL']` (array) vs `symbols: 'AAPL,GOOGL'` (string)
- Always test validator with actual template data to ensure compatibility

### Animation and Scrolling

- Horizontal scrolling: `animation: scroll-left Xs linear infinite`
- Duplicate content for seamless loops: `[...items, ...items]`
- Use `transform: translateX()` for smooth scrolling performance
- Account for card widths + gaps in animation distance calculations

### Capture Profile Configuration

- Set adequate timeouts: 8-15 seconds for API-dependent templates
- Screenshot timing: 3-4 seconds after detection for animations to stabilize
- Video duration: 10-12 seconds for scrolling content to show full cycles
- Sample data should showcase template capabilities (4-6 varied items)

### Provider Abstraction Pattern

```javascript
const providers = {
  primary: {
    id: 'primary',
    needsApiKey: true,
    getApiKey: () => config.apiKeys.primary,
    parseSymbol(symbol) {
      /* format-specific parsing */
    },
    async fetchData(symbol, apiKey) {
      /* API implementation */
    },
  },
};
```

### Timer and Interval Cleanup

- **CRITICAL**: Always clean up setInterval/setTimeout when templates change
- Store interval IDs: `const refreshInterval = setInterval(...)`
- Add cleanup on unload: `window.addEventListener('beforeunload', () => clearInterval(refreshInterval))`
- Prevent multiple intervals: Check and clear existing intervals before starting new ones
- Use global namespace: `window.hdisplay.{templateName}Interval` for cross-template cleanup
- Pattern prevents lingering API requests after template is cleared/replaced
- Example pattern:

```javascript
const refreshInterval = setInterval(load, refreshMs);
window.addEventListener('beforeunload', () => clearInterval(refreshInterval));
if (window.hdisplay) {
  if (window.hdisplay.templateNameInterval) {
    clearInterval(window.hdisplay.templateNameInterval);
  }
  window.hdisplay.templateNameInterval = refreshInterval;
}
```

When adding a template

1. Create templates/<id>.html
2. Add templates/\_validators/<id>.js (enforce required fields, bounds)
3. Add README usage snippet under Templates section if user-facing
4. Add/adjust test in test/templates.validation.api.test.js (or adjacent)
5. Optionally add a capture profile capture-profiles/<id>.yaml and regenerate captures

## CLI data mapping (no-JSON flags)

- The CLI maps flags to template data:
  - Scalars: --text "Hello" → { text: "Hello" }
  - Numbers auto-coerced: --velocity 120 → { velocity: 120 }
  - Arrays: repeated flags build arrays: --items A --items B → { items: ["A","B"] }
  - Nested: dot paths: --theme.bg '#000' → { theme: { bg: "#000" } }
  - Booleans: --wrap = true; --no-wrap = false
- Flags override any JSON provided via --data/--data-file/stdin.
- Reserved/global flags are excluded from data (see cli/flags.js).

## Playlist behavior (server contract)

- State shape persisted under state.json:
  {
  playlist: { delayMs: number, items: [ { id, data? } ] }
  }
- delayMs is clamped to [2000, 300000]
- Rotation rules
  - 0 items: do nothing
  - 1 item: render and do not auto-rotate
  - 2+ items: rotate every delayMs
- Overrides
  - Applying content via set/template/push interrupts rotation for exactly delayMs, then resume at next item
  - Clearing clears playlist and stops rotation
- API: GET/PUT /api/playlist, POST /api/playlist/items, DELETE /api/playlist/items/:index, DELETE /api/playlist/items/by-id/:id, POST /api/playlist/delay

## Quality gates – green before done

Run locally before pushing:

- Build/Start: npm start boots and /healthz returns ok
- Lint/Format: npm run lint and npm run format (format can change files; commit them)
- Unit/Integration tests: npm test is green

PR checklist (what Copilot should produce)

- Code + tests
- Minimal docs updates (README sections and/or TEMPLATES.md if applicable)
- No breaking changes without docs and tests
- Keep diffs focused and avoid unrelated formatting changes

## Common tasks playbook

Add a new template

- Files: templates/<id>.html and templates/\_validators/<id>.js
- Tests: add failure (bad/missing data) and success cases
- Docs: add usage to README Templates section
- Optional: capture profile and regenerated assets

Add a new CLI command

- Implement in ESM (cli/index.mjs or subcommand .mjs) and expose via the existing CJS shim in cli/index.js. Keep global options compatible.
- Update README with usage and examples
- Add a Jest test that exercises the command via the API

Modify playlist logic

- Respect override semantics and delay clamping
- Update test/playlist.api.test.js as needed

Uploads/media changes

- Keep /uploads static serving and deletion behavior intact
- Update tests in upload.test.js and uploads.static.delete.test.js if behavior changes

## Pitfalls and tips

- zsh quoting for JSON: prefer single quotes around JSON, escape inner double-quotes
- E2E baseURL: Playwright defaults to http://localhost:3100; set BASE_URL env var if needed
- ffmpeg optional: WEBM produced; MP4 best-effort
- mDNS: bonjour-service must not crash server when unavailable; errors are caught. Keep mDNS initialization inside direct-run paths for ESM entries so tests importing the app don’t attempt to publish services.

## ESM-first guidance (current exceptions)

- New modules: write ESM. Prefer importing ESM versions of internal code.
- CLI: Primary implementation is ESM in cli/index.mjs; cli/index.js is a minimal CJS shim that dynamic-imports the ESM and forwards argv.
- Server: ESM entry exists at server/index.mjs. Tests currently require('../server/index') (CJS). Until tests move to ESM, keep server/index.js exporting { app, server } and use server/index.mjs for direct runs and tooling.
- Do not duplicate logic between ESM and CJS; shims must be thin wrappers only.

Ergonomics discipline for config/env/CLI

- Do not propose or implement multiple aliases for configuration keys, environment variables, or command-line flags unless explicitly requested.
- Treat these names as unique and unambiguous sources of truth. Avoid offering “helpful” alternatives (e.g., TOMORROWIO_API_KEY vs TOMORROW_API_KEY); pick the single canonical spelling defined in the repo.

## Minimal ask-policy for Copilot

- Prefer doing the work if you can infer missing details from repo patterns
- Ask only when blocked; keep questions specific and few

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ewilderj)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ewilderj)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
