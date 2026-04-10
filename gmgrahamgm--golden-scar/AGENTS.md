## Summary
This repository is a testing suite to build a LightGBM antivirus detection model using EMBER2024.
Culling scripts attempt to find the most representative subsample from the full EMBER2024 dataset.
Drifting attempts to use real world data to analyze concept drift and create "expert" models.
Routing uses the data and expert models alongside DSEL and KNORA-U to create a dynamic ensemble selection model.

## Architecture
LightGBM will be used for training.
- Hyperparameter optimization will be done via random search with cross-validation.
EMBER2024 will be used as the main dataset.
- EMBER2024 is approximately 48GB, 4.68 million samples, and 2500 features.
- HDBSCAN and DBSCAN will be used for clustering.
Use the virtual python environment located in `~/golden-scar/.venv/`.
thrember (EMBER2024) is a python package located in `~/golden-scar/EMBER2024/`. It contains scripts to load and process the EMBER2024 dataset.
Whenever you wish to use Python, you must use `source ~/golden-scar/.venv/bin/activate && python3 ...` to ensure you are using the correct environment.

## Task planning and problem-solving
- Before each task, you must first complete the following steps:
  1. Provide a full plan of your changes.
  2. Provide a list of behaviors that you'll change.
- Before you add any code, always check if you can just re-use or re-configure any existing code to achieve the result.

## Coding guidelines
- Always focus on simplicity and precision and not comprehensiveness.
- Do not create a markdown file explaining changes unless prompted.
- Do not use emojis.
- When fixing a bug, always write a failing test first.
- Always follow the DRY principle and avoid code duplication.
- Avoid hard-coded numbers and use shared constants instead.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gmgrahamgm)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gmgrahamgm)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
