# Copilot Code Review Instructions

When performing a code review:
- Respond in Japanese.
- Prioritize correctness, security, and maintainability over style.
- Require tests for behavioral changes; flag missing unit/integration tests.
- Verify error handling, logging, and edge cases.
- Check for secrets, credential leakage, and unsafe deserialization.
- For Python: enforce type hints, avoid broad except, and keep functions cohesive.
- For frontend: check accessibility (ARIA, focus management) and performance regressions.

Output format:
- Summary (1-3 bullets)
- High-risk issues (blocking)
- Medium/low issues
- Suggested fixes (with code snippets if possible)

---
> Source: [auto-res2/20260213-airas-m-test](https://github.com/auto-res2/20260213-airas-m-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-03 -->
