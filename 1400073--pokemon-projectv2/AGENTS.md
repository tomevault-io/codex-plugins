## Quick context

- Root assets cover Run & Bun data/policy work, but almost every script expects the fast `pokemon-python/` simulator to exist as a sibling checkout plus the bundled `Pokemon_Trainer_Tournament_Simulator/` (Python CLI + patched Pokemon Showdown).
- Always scope a change to one surface (sim/data, Run & Bun policy/config, or the tournament/Showdown toolchain) to avoid tangled diffs.

## Layout & surfaces

- `pokemon-python/` — simulator entrypoints (`sim/sim.py`) and data loader (`data/dex.py`) plus the legacy `unittest` suite; clone it beside this repo before running anything.
- `ai_policy.py` + `AI_Document_for_RnB*.txt` + `Mechanic Changes.txt` — Run & Bun heuristics; keep the documented probability splits and seed randomness when adding rules.
- Run & Bun data (`runandbun.lua`, `gen8.js`, `trainer_data.json`, `moves_base.json`, `moves_index.py`, `species_index.py`) feed both Lua/JS clients and diff tooling.
- `tools/` hosts validators (`rnb_diff.py`, `rnb_compare.py`, `rnb_diff_report.json`) that reconcile Run & Bun sources with `pokemon-python/data`.
- `Pokemon_Trainer_Tournament_Simulator/` contains the batch pipeline (`Data/*.py`) and the `pokemon-showdown/` submodule (custom AI in `sim/examples/Simulation-test-1.ts`, request/schema tweaks in `sim/pokemon.ts` + `sim/side.ts`).

## Workflows

- `python rnb_custom_tests.py` exercises dex hydration plus deterministic one-turn/mechanic checks—run it after any simulator/data/policy change.
- Broader simulator regressions live under `pokemon-python/test`; run `python -m unittest discover -s "pokemon-python/test"` or target a file as needed.
- After touching data, regenerate reports with `python tools/rnb_diff.py` (counts) and `python tools/rnb_compare.py` (writes `tools/rnb_full_diff.json` suggestions).
- Tournament runs (inside `Pokemon_Trainer_Tournament_Simulator`): `pip install -r requirements.txt`, then in `pokemon-showdown/` run `npm install` + `node build`, finally `python Data/BuildBattles.py` → `python Data/runSimulations.py` (tune `RUN_N_TIMES`, `noOfThreads`, `setLevel`).

## Conventions & integration

- Data-first edits must update JSON (`pokemon-python/data/*.json`, `moves_base.json`, `trainer_data.json`) and their loaders (`data/dex.py`, `moves_index.py`, `species_index.py`) together so diff scripts stay green.
- Use `dex.Decision` + `sim.structs.PokemonSet` for battle inputs, mirroring `rnb_custom_tests.py`, and disable RNG (`battle.rng = False` or `rng=False`) when asserting mechanics.
- AI policy tweaks should cite the Run & Bun doc, keep 80/20-style weightings, and add deterministic coverage in either `pokemon-python/test` or `rnb_custom_tests`.
- Maintain the schema expectations of `runandbun.lua`/`gen8.js` (`move = {}`, `mons = {}`, `SETDEX_SS` keys) because the diff tools rely on those regexes.
- Showdown-side edits (AI or request payloads in `Simulation-test-1.ts`, `sim/pokemon.ts`, `sim/side.ts`) must be mirrored with a small `Data/runSimulations.py` test run to ensure the enriched fields still parse.

## Before you send a PR

1. Run `python rnb_custom_tests.py` and any focused `python -m unittest pokemon-python/test/...` you touched.
2. If Run & Bun data changed, rerun `python tools/rnb_diff.py` + `python tools/rnb_compare.py` and summarize key deltas from the new JSON reports.
3. For tournament/Showdown edits, rebuild (`npm install; node build`) and run a small `python Data/runSimulations.py` batch; mention thread counts you used.
4. Note explicitly when reviewers need to fetch `pokemon-python/` or rerun the Showdown build so bots don’t get stuck.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1400073)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/1400073)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
