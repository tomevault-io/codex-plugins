# CLAUDE.md — presidio-hardened-fastapi

See `PRESIDIO-REQ.md` for requirements and `SECURITY.md` for security policy.

## Verification

```bash
.venv/bin/python -m ruff check . && .venv/bin/python -m ruff format --check . && .venv/bin/python -m pytest tests/ -x -q --tb=short
```

---
> Source: [presidio-v/presidio-hardened-fastapi](https://github.com/presidio-v/presidio-hardened-fastapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-24 -->
