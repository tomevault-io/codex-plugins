# Copilot Code Review Instructions — Ironclad Workflow

## Project context

Structured 4-phase development workflow (PLAN, EXECUTE, VERIFY, SHIP) with AI code review via Google Gemini, file integrity tracking, and zero dependencies.
Stack: JavaScript, Node.js, zero external dependencies.

## Review priorities

1. **Security first** — flag hardcoded API keys. Gemini API key must use env vars.
2. **Zero dependencies** — this project has no npm dependencies by design. Flag any require/import of external packages.
3. **File integrity** — hash calculations and file tracking must be deterministic and correct.
4. **Error handling** — CLI must surface clear error messages, never crash silently.

## Code style

- Conventional commits: feat:, fix:, docs:, refactor:, test:, chore:.
- Comments explain *why*, never *what*.
- Pure Node.js stdlib only — no external packages.

## Patterns to flag

- Any import/require of external npm packages.
- Non-deterministic behavior in hash or integrity calculations.
- Missing error handling in file I/O or API calls.

---
> Source: [As-The-Geek-Learns/ironclad-workflow](https://github.com/As-The-Geek-Learns/ironclad-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-09 -->
