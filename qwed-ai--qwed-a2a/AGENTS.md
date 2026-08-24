# Copilot Instructions for QWED A2A

Repository-specific rules:

- Do not suggest fallback execution paths that bypass verification.
- Do not suggest graceful degradation that continues past failed verification.
- Do not suggest retries that weaken enforcement.
- Do not trust agent output as proof of correctness.
- Prefer fail-closed behavior over convenience or availability.
- Treat unresolved vulnerabilities as still critical — new layers are additive.

If a suggestion conflicts with fail-closed enforcement rules, the suggestion must be rejected.

---
> Source: [QWED-AI/qwed-a2a](https://github.com/QWED-AI/qwed-a2a) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-22 -->
