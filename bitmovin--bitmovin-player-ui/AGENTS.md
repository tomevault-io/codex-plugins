Follow the canonical repository instructions in [AGENTS.md](../AGENTS.md).

When UI correctness depends on a real browser—layout, computed styles, input/focus, browser APIs,
Shadow DOM, rendered markup, or UI–Player event flows—add or update Playwright coverage under
`test/browser/`; do not rely on Jest/jsdom alone. Use the guarded harness and conventions described
in `AGENTS.md` and `test/browser/README.md`.

---
> Source: [bitmovin/bitmovin-player-ui](https://github.com/bitmovin/bitmovin-player-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-26 -->
