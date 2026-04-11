# SAGE Eval Copilot Instructions

## Scope
- Package `isage-eval` (`sage_eval`) is an L3 evaluation library.
- Provides reusable metrics, profilers, and LLM judge implementations.

## Critical rules
- This is not a benchmark runner; keep experiment orchestration out of this repo.
- Keep runtime-neutral design and avoid L4+ dependencies.
- Flownet-aligned ecosystem direction; do not add `ray` imports/dependencies.
- Do not create new local virtual environments (`venv`/`.venv`); use the existing configured Python environment.
- In conda environments, use `python -m pip`.
- No silent fallback logic.

## Architecture focus
- Metrics (`metrics/`), profilers (`profiler/`), judges (`judge/`), registration (`_register.py`).
- Keep APIs typed and composable for upstream benchmark repos.

## Workflow
1. Make minimal implementation changes.
2. Add unit tests for edge cases and API behavior.
3. Keep docs/examples aligned with public API.

## Polyrepo coordination (mandatory)

- This repository is an independent SAGE sub-repository and is developed/released independently.
- Do not assume sibling source directories exist locally in `intellistream/SAGE`.
- For cross-repo rollout, publish this repo/package first, then bump the version pin in `SAGE/packages/sage/pyproject.toml` when applicable.
- Do not add local editable installs of other SAGE sub-packages in setup scripts or docs.

## 🚫 NEVER_CREATE_DOT_VENV_MANDATORY

- 永远不要创建 `.venv` 或 `venv`（无任何例外）。
- NEVER create `.venv`/`venv` in this repository under any circumstance.
- 必须复用当前已配置的非-venv Python 环境（如现有 conda 环境）。
- If any script/task suggests creating a virtualenv, skip that step and continue with the existing environment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intellistream)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/intellistream)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
