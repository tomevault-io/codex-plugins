# Copilot Instructions for QWED

Read and follow [QWED_RULES.md](../QWED_RULES.md) for every suggestion.

Additional repository-specific rules:

- Do not suggest fallback execution paths.
- Do not suggest graceful degradation that continues past failed verification.
- Do not suggest retries that weaken enforcement.
- Do not trust model output as proof of correctness.
- Prefer fail-closed behavior over convenience or availability.

If a suggestion conflicts with QWED enforcement rules, the suggestion must be
rejected.

---
> Source: [QWED-AI/qwed-verification](https://github.com/QWED-AI/qwed-verification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-23 -->
