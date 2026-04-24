
# Refcounting Rules

Read `docs/MEMORY_POLICY.md` before changing memory ownership logic.

`RETAIN`, `RELEASE`, `AUTORELEASE`, and `ASSIGN` are null-safe macros.
Do not add explicit null checks before these macros.

Use:
- `RELEASE(x)`

Not:
- `if (x) RELEASE(x)`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g3ntleman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
