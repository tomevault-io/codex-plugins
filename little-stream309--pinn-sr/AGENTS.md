## Quick orientation

This repository implements a Physics-Informed Deep Learning (PiDL) workflow to discover the Burgers PDE with a source term. Key artifacts:

- `code/Burgers_Source.py` — main training script. Loads data from `Burgers_SineSource.mat`, builds the model and runs the training/diagnostics pipeline.
- `code/Utils_Burgers_Source.py` — contains the `PiDL` class (model, losses, training loop, STRidge / ADO logic).
- `code/PDE_FIND.py` — numerical utilities for differentiation and library construction used during equation discovery.
- `data/` — expected place for input data (.mat files). The main script expects `Burgers_SineSource.mat` in cwd or `data/`.

## Environment & dependencies (discoverable from imports)

- Python 3.x with packages: `tensorflow==1.15.0` (code uses TF1 placeholders / tf.Session / tf.contrib), `numpy`, `scipy`, `matplotlib`, `pyDOE`, `tqdm`.
- GPU support: scripts use `with tf.device('/device:GPU:0')` and create a `tf.Session` with GPU growth enabled. Keep TF1 GPU setup if you want GPU training.

Suggested minimal pip requirements (for maintainers/agents):

```
tensorflow==1.15.0
numpy
scipy
matplotlib
pyDOE
tqdm
```

## How to run (what agents should do)

- To run training locally (from repo root):

```
python code/Burgers_Source.py
```

- The script saves results to the working directory: `FullField_ModelPred.png`, `FullField_GroundTruth.png`, `Pred.mat`, `LambdaEvolution.mat`.

If the dataset file (`Burgers_SineSource.mat`) is stored in `data/`, run the script from repo root and ensure the path `data/Burgers_SineSource.mat` is used or copied to the working directory.

## Project-specific patterns & important invariants

- TensorFlow 1.x API: code uses placeholders (`tf.placeholder`), `tf.Session`, `tf.contrib.opt.ScipyOptimizerInterface` (L-BFGS). Do NOT convert code to TF2 idioms unless you update all training and optimizer usages.
- Library size / term index: `PiDL.lambda1` is defined as shape `[29,1]`. The code and post-processing assume a 29-term library. If you change `Phi` / library construction, update `lambda1` and the downstream post-processing (e.g., indexing used in `Burgers_Source.py`).
- Training flow (compound algorithm):
  - Adam pretraining (many epochs)
  - L-BFGS-B (via SciPy interface) pretrain
  - ADO loop: alternating Adam, L-BFGS, and STRidge (sparse regression) updates
  Preserve this flow when editing optimization logic.
- Random seeds: reproducibility relies on `np.random.seed(1234)` and `tf.set_random_seed(1234)` in the code. Keep these for deterministic experiments.

## Where to make common changes

- Network architecture: change `layers = [2] + 8*[80] + [1]` in `code/Burgers_Source.py` (or accept external config) and pass to `PiDL(...)`.
- Collocation / measurement points: `N_f`, `N_s`, `N_t` and sampling logic live in `code/Burgers_Source.py`.
- Library terms: see `PiDL.physics_residue()` in `code/Utils_Burgers_Source.py` — trig and polynomial terms are concatenated there and `library_description` is set.

## Debugging hints for agents

- TF device placement and logging: `ConfigProto(allow_soft_placement=True, log_device_placement=True)` will print device placement — useful when debugging GPU/CPU fallbacks.
- If L-BFGS throws errors, check that `tf.contrib` is available (TF1); L-BFGS options are passed to `ScipyOptimizerInterface` and may need parameter tuning.
- When changing the library or lambda shape, update the post-processing in `code/Burgers_Source.py` where `lambda_true` and nonzero indices are compared.

## Small examples to reference

- To inspect the library description at runtime, print `model.library_description` after model initialization (the `PiDL` instance populates this in `physics_residue`).
- To change the random train/val split alter `Split_TrainVal` in `code/Burgers_Source.py`.

## Keep in mind

- This repo targets reproducible experiments with TF1 and SciPy L-BFGS; avoid migrating to TF2 without a full, tested conversion.
- Document any changes to the library terms and lambda indexing — downstream scripts expect a fixed ordering/length.

If any of the above assumptions are incorrect or you want the instructions expanded (examples, explicit requirements file, or CI tips), tell me which area to expand and I will update the file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Little-Stream309)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Little-Stream309)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
