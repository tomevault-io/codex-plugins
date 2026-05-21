# Copilot Instructions — Fuse (ONNX DSL)

Purpose: give concise, actionable context so an AI coding agent can be immediately productive in this repo to produce modular, production, non-boilerplate, DRY code that is free from tech debt.

Quick reference
- Canonical spec: `SPEC.md` (grammar, type/shape rules, lowering invariants).
- Key code: `src/parser.py` (EBNF grammar + AST transformer), `src/lowering/main.py` (FuseLowerer & ops lowering), `src/graph_context.py` (GraphContext), `src/import_fusion.py` (ImportManager / import fusion).
- See `examples/golden/` and `jupyter/cookbook/` (use `./scripts/run_examples.sh --validate`).
- Changelog: `CHANGELObinG.md` (recent changes, important notes).
- no backward-compatiblity and deprecation messages.
- ALWAYS run your tests / scripts / python using `./.venv/bin/python`
- and/or use `make setup` to setup the development environment.

Big picture
- Fuse is a small cognitive compiler that lowers to ONNX via these stages: Source → AST → Resolved AST → Typed/shape-checked IR → ONNX → .onnx.
- Design invariants: determinism (same source → identical bytes), stable SSA-style names, params→initializers, consts folded early.
- **make gold**: Run `make gold` to compile all `examples/golden` models and verify ONNX outputs (does *not* run the full test suite).  ``make test`` performs all unit tests.


## Big picture & architecture 🧭
- The pipeline is: **source.fuse → parser (AST) → normalize (lambdas/type aliases) → lowering (FuseLowerer) → GraphContext → ONNX ModelProto**.
- **Determinism** is a first-class invariant: same source must produce identical bytes. Tests rely on stable naming and ordering (SSA-style names, deterministic name allocation).
- **Separation of concerns**: parsing + AST transforms are in `src/parser.py` and `src/ast/`; lowering and op-specific lowering live under `src/lowering/`; import and variant fusion are in `src/import_fusion.py` and `src/imports/`.
- **GraphContext** centralizes naming, value-types and metadata (e.g., `trainables`), and provides `add_param`, `add_const`, and `build_model()` helpers used by `FuseLowerer`.

## Project-specific conventions & patterns 📐
- **Namespacing enforced by default**: When lowering from a file source, an explicit `@domain` is required.
- **Params vs Consts**: `param` with a default value is treated as a constant initializer; `const` becomes an initializer. `trainable` flags are serialized into `ModelProto.metadata_props` as `trainables` (stable JSON map).
- **Imports / Variants**: `@import` declares remote/local ONNX models with `@variant` entries. Import fusion prefixes domains deterministically and wires inputs/outputs. External tensor references (`external_data`) are supported and may be written out via `--externalize`.
- **Lowering helpers**: inline lambdas are normalized into named functions before lowering to keep lowering deterministic (see `src/ast/normalize_lambdas.py`).
- ALWAYS run your tests / scripts / python using .venv:

## Testing & developer workflow 🧪
- Do NOT write standalone scripts; add to `./tests/smoke/` and run via `make smoke-test` instead.
- Use `uv` for all development and tests.
- Use `make setup` to setup the development environment.
- Use `make test` for unit tests
- Use `make smoke-test` for quick smoke tests (./tests/smoke/)
- Use `make test-lowering` for lowering tests
- Use `make test-parsing` for parsing only
- Use `make test-autodoc` for testing built-in fuse docs generation (--docs CLI)
- Use `make test-golden` for testing golden examples (./examples/golden/*.fuse)
- Use `make test-jupyter` for unit testing jupyter application
- Use `make test-decompile` for auditing/decompiling emitted ONNX back to Fuse source.
- Use `make test-server` for unit testing server without blocking.
- Use `make test-all` for full test run (all above).
- Use `make golden` for a clean build + test-all.
- Use `make jupyter` to run jupyter container (if not running).
- Use `make jupyter-test` for jupyter unit tests.
- Use `make jupyter-lsp` for jupyter LSP tests.
- Use `make jupyter-llm` for jupyter LLM tests.
- Use `make jupyter-stop` for a clean build + test-all.
- Use `make jupyter-docker` for a full test then docker build.
- Use `make package` clean, build, test and mint packages.
- ALWAYS run your tests / scripts / python using .venv:

## Integration points & external dependencies 🔗
- ONNX: opset compatibility is enforced (`src/onnx_opset.py`); prefer pinned opset versions for imports. Use `scripts/update_onnx_ops.py` to refresh operator lists.
- ORT Web bundles: helper scripts `scripts/install_ort_web.sh` and `src/ort_web_install.py` manage pinned runtimes under `third_party/ort_web/` (CI locks via `LOCK.json`).
- Remote imports: `@import from "https://..."` supported — `RemoteImportManager` validates variants and URLs.

## Implementation checklist for agents ✅
When changing code, prefer small, well-scoped PRs and follow this checklist:
1. Preserve determinism (don't change emitted model ordering or names without justification).
2. Add focused tests (unit for logic; golden/integration for emitted ONNX bytes if relevant).
3. Use `name_allocator` or `GraphContext` helpers to keep emitted names deterministic in tests.
4. Run `make test-lowering` → `make gold` locally before submitting PR.
- All temporary scripts, work files, adhoc tests MUST be in `./tmp/` (not `/tmp/`).
- All plans, reviews, progress / summary reports MUST be in `./todo/` (not `/`).

*Last note:* keep edits compact, add tests for every behavioral change, and prefer adding examples under `./examples/fuse/` for language-level features.

---
> Source: [onnx-cloud/fuse](https://github.com/onnx-cloud/fuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-21 -->
