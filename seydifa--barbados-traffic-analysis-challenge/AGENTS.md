# Copilot Instructions - Barbados Traffic Analysis

## Project Overview
Zindi competition project for predicting traffic congestion levels (`congestion_enter_rating`, `congestion_exit_rating`) from 60-second intersection videos. Uses PyTorch with video understanding via temporal encoders (LSTM/Transformer) on visual features.

## Architecture

### Core Pipeline
```
Video (MP4) → torchcodec decoder → Vision Encoder (timm EfficientNet) → Temporal Encoder (LSTM/Transformer) → Classification Head → 2 outputs × 5 classes
```

### Key Modules (`traffic_analysis/`)
- [config.py](traffic_analysis/config.py) - Dataclass configs: `PathConfig`, `DataConfig`, `ModelConfig`, `TrainingConfig`. Central `Config` aggregates all.
- [datasets.py](traffic_analysis/datasets.py) - `load_video()` with torchcodec, `TrafficVideoDataset`, stratified splits
- [models.py](traffic_analysis/models.py) - `ModelFactory.create_model()`, vision/temporal encoders, attention layers
- [trainer.py](traffic_analysis/trainer.py) - PyTorch training loop with callbacks (EarlyStopping, ModelCheckpoint), AMP support
- [preprocessing.py](traffic_analysis/preprocessing.py) - YOLOv12 tracking with Supervision, saves to Parquet

## Critical Patterns

### Configuration System
All configs use Python dataclasses with env var overrides:
```python
from traffic_analysis.config import DEFAULT_CONFIG, Config
# Override via: TORCH_DTYPE_POLICY, TORCH_COMPILE_ENABLED, TORCH_QUANTIZATION_ENABLED
```

### Video Loading
```python
from traffic_analysis.datasets import load_video
frames = load_video(
    filepath, device="cuda", method="start",  # 'start', 'center', 'end'
    num_frames=128, crop_duration=40.0, image_size=224
)
```
- Uses `torchcodec` for GPU-accelerated decoding (falls back to CPU)
- Extracts temporal crops: first 40s (`start`), middle 40s (`center`), or last 40s (`end`)

### Model Creation
```python
from traffic_analysis.models import ModelFactory
model = ModelFactory.create_model('lstm', config)  # or 'transformer', 'gru'
```
Architecture options: `lstm`, `gru`, `transformer`, `transformer_small`, `hybrid_lstm_attn`

### Trainer Usage
```python
from traffic_analysis.trainer import Trainer, TrainerConfig
trainer_config = TrainerConfig.from_config(config)
trainer = Trainer(model, trainer_config)
trainer.fit(train_loader, val_loader)
```

## Kaggle Environment
- Default paths assume `/kaggle/input/reencoded-barbados-traffic` for data
- Output to `/kaggle/working` (checkpoints, features, logs)
- Switch paths in `PathConfig` for local development

## Dependencies
- `torchcodec` - Video decoding (GPU/CPU)
- `timm` - Vision backbones (EfficientNet default)
- `ultralytics` - YOLOv12 for object tracking
- `supervision` - Detection visualization
- `wandb` - Optional experiment tracking (requires `WANDB_API_KEY`)

## Conventions
- French comments in code (e.g., "# Encodeur temporel")
- Multi-output classification: always predict both enter/exit congestion
- Labels are ordinal (0-4 congestion levels), but treated as classification
- Notebooks numbered sequentially: `001-eda.ipynb`, `002-preprocessing.ipynb`, etc.

## Performance Optimizations
- Mixed precision via `use_amp=True` (default)
- Backbone freezing via `freeze_backbone=True`
- Optional `torch.compile` via `TORCH_COMPILE_ENABLED=1`
- Int8 quantization via `TORCH_QUANTIZATION_ENABLED=1`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Seydifa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
