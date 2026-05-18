# OpenSloth Agent Guide

- **Mission**: Scale Unsloth SFT training across GPUs via `torchrun`; most logic lives in `src/opensloth/patching` and `src/opensloth/utils`.
- **DDP Patch Flow**: `opensloth.patching.ddp_patch.ddp_patch()` must run once per process right after importing Unsloth; it pins `cuda:{LOCAL_RANK}`, logs device info, and applies the Trainer loss fix.
- **Batch Repacking**: `patch_optimize_sft_trainer_batch_samples()` swaps `SFTTrainer.get_batch_samples` with a DP-based packer; new training scripts should opt-in after `ddp_patch()` when targeting multi-GPU efficiency.
- **Trainer Template**: Follow `examples/qwen_sft.py` or `train_scripts/train_ddp.py`; the canonical setup is `ddp_patch(); patch_optimize...; FastLanguageModel.from_pretrained(...device_map=f"cuda:{local_rank}")`.
- **LoRA Defaults**: `FastLanguageModel.get_peft_model` always targets `["q_proj","k_proj","v_proj","o_proj"]`, rank 8, alpha 16, and `use_gradient_checkpointing="unsloth"`—mirror these unless intentionally diverging.
- **World Size Awareness**: Scripts derive `WORLD_SIZE` from env; keep `per_device_batch_size` inversely proportional and enforce `ddp_find_unused_parameters=False` when `world_size>1`.
- **Env Vars**: DDP relies on `LOCAL_RANK`, `RANK`, `WORLD_SIZE`; `_check_distributed_environment` in `utils/dataset_cache.py` enforces integer values—propagate those when writing CLI wrappers.
- **Dataset Prep**: Capybara chat data is formatted with `tokenizer.apply_chat_template` and stored under `text`; reuse the map/remove_columns pattern in `train_scripts/` to stay consistent.
- **Caching Wrapper**: `opensloth.utils.simple_cache_dataset_wrapper` decorates trainer factories; it expects the wrapped callable signature `(model, tokenizer, train_dataset, val_dataset, world_size, ...)` and relies on HuggingFace `save_to_disk`. Ensure new wrappers respect that contract.
- **Cache Keys**: Metadata includes dataset sizes and `cache_key_params`; update `cache_key_params` when adding knobs that change tokenization to avoid stale caches.
- **Debugging Tooling**: `opensloth._debug_dataloader.debug_chat_dataloader_for_training` writes HTML previews to `.log/`; use it when touching dataloaders to spot label masking issues.
- **Packing Utilities**: `opensloth.utils.packing.pack` implements best-fit bin packing with causal masks; if you reuse it, honor the expected batch dict keys (`input_ids`, `attention_mask`, `labels`).
- **Multi-GPU Workflow**: Run via `torchrun --nproc_per_node=N train_scripts/train_configurable.py ...`; `scripts/sanity_check_training.sh` demonstrates the supported matrix and is the quickest regression check.
- **Single GPU**: Plain `python train_scripts/train_configurable.py --no_patches` keeps behavior close to multi-GPU for parity testing.
- **Precomputed Data**: `train_scripts/precompute_dataset.py` bootstraps tokenized datasets and saves under `data/precomputed/.../train|val`; downstream scripts expect that directory layout.
- **Logging & Outputs**: Training defaults to TensorBoard logging under `outputs/<run_name>`; maintain `run_name=f"{experiment}_ws{world_size}"` so multi-GPU runs stay separate.
- **Versioning**: Project uses `uv` + `poetry`; local installs follow `uv pip install -e .`. Keep Python ≥3.12 per `pyproject.toml`.
- **External Deps**: Core runtime depends on `unsloth`, `trl`, `datasets`, `transformers`, `bitsandbytes`; avoid importing heavy libs at module import time (see comment in `ddp_patch.py`).
- **Legacy Code**: Historical experiments live under `legacy/`; avoid spreading new changes there unless backporting.
- **Style**: Code assumes ASCII logging emojis already in use; minimal inline comments only when logic is non-obvious (e.g., DP packing).
- **Validation**: After modifying training logic, smoke test with `torchrun --nproc_per_node=2 train_scripts/train_ddp.py` to catch DDP regressions before pushing.
- **Docs**: README and `TRAINING_README.md` are authoritative for user-facing flows—sync changes there when adjusting scripts or flags.

---
> Source: [anhvth/opensloth](https://github.com/anhvth/opensloth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
