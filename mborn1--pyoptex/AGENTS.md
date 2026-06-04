
# Building

- **Editable install**: `venv/bin/pip install -e ".[dev]"` (compiles Cython)
- **Cython only**: `venv/bin/python setup.py build_ext --inplace`
- **Wheel**: `venv/bin/python -m build --wheel`
- **Docs**: `make -C docs html` (output in `docs/_build/html/`)

Makefile uses `venv/` by default; no need to activate.

---
> Source: [mborn1/pyoptex](https://github.com/mborn1/pyoptex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-04 -->
