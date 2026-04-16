# Copilot Instructions for BotFlight6

## Project Overview
BotFlight6 is a NEAT-driven 2D drone simulator with a Python NEAT engine and a Rust physics/scoring backend. Python orchestrates evolution, population management, and visualization. Rust handles fast, batched physics simulation and scoring. Both layers share configuration via TOML files.

## Architecture & Data Flow
- **Python (`python/`)**: Implements NEAT (population, speciation, mutation, crossover, checkpoints), manages orchestration, and provides visualization (Pygame).
- **Rust (`rust_scorer/`)**: Library for physics simulation and batch scoring. Exposes FFI via PyO3.
- **Config (`data/simulation.toml`)**: Centralized parameters for physics, NN architecture, and evolution. Always read from config, never hard-code.
- **Persistence (`data/`)**: Stores logs, checkpoints, genomes, and stats for reproducibility.
- **Scripts (`scripts/`)**: Entrypoints for training, testing, and visualization.


## Integration & Communication
Python and Rust communicate via FFI (PyO3). Refer to project files for details on serialization and orchestration.

## Developer Workflows
All configuration is managed in `data/simulation.toml`. Python tests are located in `python/tests/`. Visualization is handled in `python/visualize.py`.

## Conventions & Patterns
- Keep Python and Rust code modular and minimal.
- Never hard-code parameters; always read from TOML config.
- Persist all important run artifacts under `data/`.
- Document new integration points and update this file as needed.


## Example: Workflow
Refer to orchestration scripts and config files for the typical run and evaluation process.

## Key Files
- `python/main.py`: Orchestration entrypoint.
- `python/genotype.py`: Genome representation.
- `python/utils.py`: Helpers for serialization, FFI.
- `python/visualize.py`: Pygame visualization.
- `rust_scorer/`: Rust physics/scoring lib.
- `data/simulation.toml`: Shared config.

---
Update this file with new patterns, integration points, or workflow changes as the project evolves.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linduix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
