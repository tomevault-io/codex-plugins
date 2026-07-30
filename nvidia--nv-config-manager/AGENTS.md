
# Python Tooling

Always use `uv run` to execute Python tools and utilities (ruff, pytest, mypy, etc.) inside this repository. Do NOT invoke them directly or via `python -m`.

```bash
# ✅ Correct
uv run ruff check src/
uv run ruff format src/
uv run pytest tests/
uv run mypy src/

# ❌ Wrong
ruff check src/
python -m ruff check src/
python -m pytest tests/
```

---
> Source: [NVIDIA/nv-config-manager](https://github.com/NVIDIA/nv-config-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-29 -->
