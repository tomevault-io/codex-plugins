
# Arktype Usage Guidelines

This rulecard is intentionally short. For examples and edge cases, see `docs/reference/arktype-usage.md`.

## Rules of thumb

- Use optional keys via `'key?'` (not `string | undefined`).
- Use `type({ '[string]': Schema })` for records.
- Use `Schema.array()` for arrays (avoid tuple mistakes).

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
