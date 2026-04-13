# Copilot Instructions for Mallorn Challenge

## Project Overview
- **Purpose:** Time-series classification using astronomical lightcurve data.
- **Major Components:**
  - `src/`: Production code (feature engineering, preprocessing, training)
  - `data/`: Raw and processed data (not versioned)
  - `notebooks/`: EDA, feature analysis, and experiment documentation
  - `output/`: Model artifacts and submission files

## Key Workflows
- **Feature Engineering:**
  - Main script: `src/make_features.py`
  - Aggregates features from grouped lightcurve data (see `features/` submodules)
  - Output: `data/processed/train_final_<version>.csv`, `test_final_<version>.csv`
- **Preprocessing:**
  - Script: `src/preprocessing.py`
  - Handles split-wise data merging, extinction correction, and data cleaning
- **Model Training:**
  - Script: `src/train.py`
  - Loads processed features, runs LightGBM with 5-fold cross-validation
  - Outputs OOF predictions, feature importances, and submission files

## Data Flow
- Raw data in `data/raw/` → Preprocessing → Feature engineering → Model training → Output in `output/`
- Data splits (`split_*/`) are merged and processed for both train/test

## Conventions & Patterns
- **Config:** All paths and directories are set in `src/config.py` (import as needed)
- **Vietnamese comments:** Some code and notebooks use Vietnamese for documentation and logs
- **Feature selection:** See `notebooks/correlation-reduction-strategy.md` for domain-driven feature dropping
- **Notebooks:** Used for EDA and diagnostics, not for production pipelines
- **No test suite:** Manual validation via notebook and script outputs

## Examples
- To generate features: `python src/make_features.py`
- To preprocess splits: `python src/preprocessing.py`
- To train model: `python src/train.py`

## Integration Points
- Uses `lightgbm`, `scikit-learn`, and `extinction` (for astrophysics)
- Feature modules in `src/features/` can be extended for new feature types

## Tips for AI Agents
- Always check `src/config.py` for path conventions
- Respect the data flow: do not write directly to `data/raw/` or `output/` except via scripts
- Reference `notebooks/` for feature selection rationale and EDA context
- Scripts are designed to be run from the project root

---
For questions or unclear conventions, review the latest `README.md` and `notebooks/` for up-to-date project context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZukaNoPro2k5)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ZukaNoPro2k5)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
