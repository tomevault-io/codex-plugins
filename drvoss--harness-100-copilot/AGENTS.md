# Copilot Instructions — harness-100-copilot

This file provides team standards that GitHub Copilot should reference when generating code in this repository.

## Repository Purpose

This is a Copilot CLI harness library — a collection of multi-agent workflows (harnesses) that orchestrate specialist AI agents to solve development lifecycle tasks. Code in this repository consists primarily of Markdown files (SKILL.md, agent .md files) that define agent behaviors and orchestration logic.

## Testing

- Write failing tests before implementing (TDD — Red → Green → Refactor)
- For bug fixes: always write a reproduction test first (Prove-It Pattern)
- General TDD target ratios: Unit 80% / Integration 15% / E2E 5%
  - (test-automation harness uses 70/20/10 to reflect its Testcontainers + Playwright scope)
- DAMP over DRY in tests — each test should be self-contained and readable
- Prefer real implementations over mocks; use mocks only at boundaries

## Code Quality

- ~100 lines changed per PR = good; ~300 lines = acceptable; ~1000 lines = split it
- Separate refactoring commits from feature commits
- Label review findings: Critical / Nit / Optional / FYI
- Dead code after refactoring: list it explicitly, ask before deleting
- No "LGTM" without evidence of actual review

## Security

- Never commit secrets; always use `.env.example` with placeholder values
- Parameterized queries only — no string concatenation into SQL
- All external input validated at system boundaries
- Rate limiting: auth endpoints ≤10 attempts per 15 minutes, general API ≤100 per 15 minutes
- Never: `eval()`, `innerHTML` with user input, `localStorage` for auth tokens

## Performance

- Measure before optimizing — always establish a baseline first
- Core Web Vitals targets: LCP ≤2.5s, INP ≤200ms, CLS ≤0.1
- JS bundle budget: < 200KB gzipped (initial load)
- API response time SLA: P95 < 200ms
- Optimization workflow: Measure → Identify → Fix → Verify → Guard

## Harness Development

- Each SKILL.md should have: agent composition, phases, scale modes, error handling, test scenarios
- Preferred new sections: Common Rationalizations, Red Flags, Verification
- Agents communicate via file-bus (`_workspace/messages/`) not direct calls
- SKILL.md body target: 200-400 lines
- Reference files > 300 lines must have a table of contents

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drvoss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
