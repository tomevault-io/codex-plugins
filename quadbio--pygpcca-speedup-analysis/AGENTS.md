# Copilot Instructions for pyGPCCA Speedup Analysis

## What this repo does

Benchmarks comparing pyGPCCA optimization methods (NM, L-BFGS-B, multi-start) on synthetic and real CellRank datasets. See README for benchmark overview and key findings.

## Critical rules
- **Don't update Jupyter notebooks** — I manage them myself
- Use `pixi run <command>` or `pixi shell` for all commands
- Summarize in chat, don't create markdown summary files

## Sibling repos (editable installs via pixi.toml)

- **`../pyGPCCA`** (`feature/gradient-optimization`): Jacobian, L-BFGS-B support, multi-start SO(k) perturbation in `_gpcca.py`. See its copilot instructions for algorithm details.
- **`../cellrank`**: GPCCA estimator with `optimizer`, `n_starts`, `perturbation_scale`, `seed` passthrough in `compute_macrostates()`.

## Benchmark conventions

- Scripts live in `analysis/<NN>_<topic>/benchmark_<name>.py`
- All scripts are standalone — run with `pixi run python analysis/.../script.py`
- Scripts print Markdown tables to stdout (Tables A/B/C pattern)
- Datasets come from `cr.datasets.pancreas(kind="raw")` and `cr.datasets.bone_marrow()`
- Both use `PseudotimeKernel` with `palantir_pseudotime`

## Key parameters

| Parameter | Recommended | Notes |
|-----------|-------------|-------|
| `optimizer` | `"L-BFGS-B"` | Gradient-based, uses analytical Jacobian |
| `n_starts` | `10` | Multi-start with SO(k) rotation perturbation |
| `perturbation_scale` | `0.1` | Angular scale; 0.1 rad best across sweeps |
| `seed` | `0` | For reproducibility |
| `method` (Schur) | `"krylov"` for m>20 | Requires PETSc/SLEPc (provided by pixi env) |

## Quick reference

| Task | Command |
|------|---------|
| Run Python | `pixi run python script.py` |
| Run tests | `pixi run test` |
| Reinstall after pyGPCCA/CellRank changes | `pixi install` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quadbio)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/quadbio)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
