# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JAF (Just Another Flow) is a streaming data processing system for JSON/JSONL with lazy evaluation, composability, and a fluent API. It processes data using S-expression queries, supporting three interchangeable syntaxes (S-expr, JSON arrays, infix DSL), and emphasizes memory-efficient streaming operations.

## Development Commands

```bash
# Setup
make install-dev          # Full dev environment (creates venv, installs deps)

# Testing
make test                 # Run all tests
make test-cov             # Tests with coverage report
venv/bin/pytest tests/test_jaf_core.py -v                          # Single file
venv/bin/pytest tests/test_jaf_core.py::TestJAFCore::test_simple_eq_query -v  # Single test
venv/bin/pytest -k "test_path"                                     # Pattern match
venv/bin/pytest tests/ -x                                          # Stop on first failure

# Quality
make lint                 # flake8
make format               # black (optional)
make check                # tests + lint

# Release
make bump-patch           # 0.8.0 → 0.8.1 (also bump-minor, bump-major)
make release              # Full: test → lint → build → tag → upload
```

**Note:** API tests (`test_api*.py`) use `pytest.importorskip` and need `pip install jaf[api]`. MCP tests (`test_mcp_server*.py`) are auto-skipped unless run directly and need `pip install jaf[mcp]`.

## Core Architecture

### Query Pipeline (most important to understand)

All three syntaxes compile to the same JSON AST, then execute through `jaf_eval`:

```
S-expression: (gt? @age 25)     ──→ sexp_parser.py ──→
JSON array:   ["gt?", "@age", 25]  ──→ (already AST) ──→  jaf_eval.py  ──→ result
Infix DSL:    @age > 25          ──→ dsl_compiler.py ──→
```

- **`jaf/sexp_parser.py`** — Parses `(op args...)` Lisp syntax via `lark` grammar → JSON AST
- **`jaf/dsl_compiler.py`** — Compiles infix `@age > 25` via `smart_compile()` → JSON AST
- **`jaf/jaf_eval.py`** — The evaluation engine. Has two categories:
  - `special_forms`: short-circuit (`and`, `or`, `not`, `if`, `@`, `exists?`, etc.)
  - `funcs`: regular operators that evaluate all args first (all predicates, math, string ops)
- **`jaf/console_script.py`** — CLI uses `smart_compile()` to auto-detect syntax before eval

### Path System

Path notation `@field` desugars to AST `["@", [["key", "field"]]]`. The path system spans:

- **`jaf/path_evaluation.py`** — `eval_path()` resolves paths against JSON objects
- **`jaf/path_types.py`** — `PathValues` (multi-value results from wildcards), `MISSING_PATH` sentinel
- **`jaf/path_conversion.py`** — Converts between path AST and `@dotted.string` notation

Key design: `MissingPath` sentinel distinguishes "path doesn't exist" from `None`/empty values.

### Streaming Architecture

Lazy evaluation — operations build pipelines, nothing executes until `.evaluate()`:

- **`jaf/lazy_streams.py`** — `stream()` factory + `LazyDataStream` (base), `FilteredStream`, `MappedStream`
- **`jaf/streaming_loader.py`** — `StreamingLoader` dispatches to source-specific loaders by `type` field in source dict
- **`jaf/lazy_ops_loader.py`** — Implements stream operations (take, skip, batch, groupby, join, distinct, intersect, except). Windowed operations accept `window_size` parameter
- **`jaf/probabilistic.py`** — `BloomFilter`, `CountMinSketch`, `HyperLogLog` for `strategy="probabilistic"` windowed ops

### Integration Layers

- **`jaf/api.py`** — FastAPI REST server with streaming endpoints + WebSocket (optional: `jaf[api]`)
- **`jaf/mcp_server.py`** — Model Context Protocol server for LLM integration (optional: `jaf[mcp]`)

## Development Patterns

### Adding New Operators

1. Add to `jaf/jaf_eval.py` — in `funcs` dict (regular) or `special_forms` set + handler method
2. Use `adapt_jaf_operator(arity, lambda)` from `jaf/utils.py` for simple operators
3. Write tests covering edge cases (null, missing paths, type mismatches)
4. Path expressions (`@field`) must work as arguments

### Modifying the Path System

- `MissingPath` sentinel is critical — never confuse missing paths with `None`
- Test wildcards (`@*.field`, `@**.field`) and special paths (regex, fuzzy)
- Path types in `path_types.py`, resolution in `path_evaluation.py`

### Adding Streaming Operations

- Implement in `jaf/lazy_ops_loader.py`, register in `StreamingLoader`
- Operations must be truly lazy — don't consume iterators until `.evaluate()`
- For windowed ops, support `window_size` with default `float('inf')`
- For CLI exposure, add subparser in `jaf/console_script.py` with `--eval` flag

### Testing Conventions

- TDD: write tests first
- API tests use `pytest.importorskip("fastapi")` — no markers needed
- MCP tests are skipped via `conftest.py` `pytest_ignore_collect` unless run directly
- `pytest.ini_options`: `asyncio_mode = "auto"` — async tests just work

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/queelius)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/queelius)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
