# FastGen – Copilot Instructions

## Project Overview
FastGen is an NVIDIA PyTorch framework for fast generation via diffusion-model distillation. It supports 10B+ parameter models across T2I, I2V, V2V tasks with methods like DMD2, sCM, MeanFlow, Self-Forcing, and more.

## Architecture (3-layer config → model → trainer)

```
train.py  →  instantiate(config.model_class)  →  Trainer(config).run(model)
```

1. **Config** (`fastgen/configs/`): 3-level hierarchy using `attrs` + OmegaConf.
   - `config.py` — Base classes: `BaseConfig > BaseModelConfig > BaseTrainerConfig`
   - `methods/` — Method-level defaults (e.g., `config_dmd2.py` adds discriminator/fake_score fields)
   - `experiments/<Arch>/` — Final experiment configs override network, data, and hyperparams
2. **Model** (`fastgen/methods/`): `FastGenModel` base → method subclasses (e.g., `DMD2Model`). Each implements `build_model()` and `training_step()`.
3. **Trainer** (`fastgen/trainer.py`): Receives a fully-built model; manages training loop, DDP/FSDP wrapping, checkpointing, and callbacks.
4. **Network** (`fastgen/networks/`): `FastGenNetwork` (ABC) per architecture (EDM, Wan, Flux, CogVideoX…). Each implements `forward(x_t, t, condition)`.

## Critical Pattern: LazyCall (`L`)

`L(Class)(arg=val)` produces a `DictConfig` with `_target_=Class`, **not** an instance. Actual construction happens later via `instantiate()`. This is core to the entire config system.

```python
from fastgen.utils import LazyCall as L, instantiate
model_class: DictConfig = L(DMD2Model)(config=None)  # stores intent, doesn't call
model = instantiate(model_class)                       # actually calls DMD2Model(config=...)
```

Nested LazyCall dicts are resolved recursively. Always use `L()` for deferred construction in configs.

## Config Conventions

- **`attrs.field(factory=lambda: copy.deepcopy(X))`** — Every mutable config default MUST use this pattern to avoid shared-state bugs across instances.
- **Experiment configs** define a `create_config()` function that calls the parent method's `create_config()` then overrides fields:
  ```python
  def create_config():
      config = config_dmd2_default.create_config()  # inherit method defaults
      config.model.net = Wan_1_3B_Config             # swap network
      config.model.net_optimizer.lr = 1e-5           # override specific hyperparam
      config.dataloader_train = VideoLoaderConfig     # swap data loader
      return config
  ```
- **CLI overrides** use dash separator: `python train.py --config=path.py - key=value nested.key=value`
- **`--dryrun`** flag prints resolved config without training.

## Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `FASTGEN_OUTPUT_ROOT` | Output dir (default: `./FASTGEN_OUTPUT`) |
| `DATA_ROOT_DIR` | Dataset root |
| `CKPT_ROOT_DIR` | Pretrained checkpoint root |
| `HF_HOME` | HuggingFace cache |
| `LOCAL_FILES_ONLY` | Set `1` to prevent network downloads |

## Developer Workflow

```bash
pip install -e .           # install in dev mode
make format                # ruff format + fix (excludes third_party/)
make lint                  # ruff check
make mypy                  # type check via mypy
make pytest                # run tests (ignores FASTGEN_OUTPUT, runs, tmp, third_party)
```

- **Training:** `python train.py --config=fastgen/configs/experiments/EDM/config_dmd2_test.py`
- **Multi-GPU:** `torchrun --nproc_per_node=8 train.py --config=... - trainer.ddp=True` (or `trainer.fsdp=True` for large models)
- **Signed commits required** — use `git commit -s`.

## Adding a New Experiment

1. Create `fastgen/configs/experiments/<Arch>/config_<method>.py`
2. Import the method's base `create_config` and the target network/data configs
3. Override: `net`, `input_shape`, `discriminator`, `dataloader_train`, learning rates, `log_config.group`
4. Keep method-agnostic defaults (callbacks, warm-up) in the method config layer

## Adding a New Method

1. Create `fastgen/methods/<category>/<method>.py` subclassing `FastGenModel`
2. Implement `build_model()`, `training_step()`, and optionally `on_train_begin()`
3. Create `fastgen/configs/methods/config_<method>.py` with a `ModelConfig(BaseModelConfig)` and `Config(BaseConfig)`
4. Register extra optimizers/schedulers as `attrs.field(factory=lambda: copy.deepcopy(BaseOptimizerConfig))`

## Code Style Notes

- License header required: `# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES.`
- `fastgen/third_party/` is excluded from all linting/formatting
- Type hints used throughout; `str | None` union syntax (Python 3.10+)
- Precision is controlled per-context: `precision`, `precision_amp`, `precision_amp_infer`, `precision_amp_enc`

---
> Source: [csy2077/data-forcing-distillation](https://github.com/csy2077/data-forcing-distillation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-24 -->
