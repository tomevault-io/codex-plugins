# Copilot Instructions — concrete_audit

## Project Overview

Concrete specimen fraud detection system (混凝土试块造假识别). Compares **wet-state** (制作时) and **dry-state** (送检时) photos of the same concrete test block using local feature matching to determine if a specimen was swapped.

The system pairs images via QR codes, preprocesses them through ROI extraction and perspective correction, then runs feature matching (SuperPoint/ALIKED/SIFT/LoFTR/RoMa + LightGlue) to produce a SAME/DIFFERENT/INSUFFICIENT/INVALID verdict.

## Running

```bash
# Full multi-method comparison on samples/
python main.py

# Limit to N pairs, specific methods
python main.py --limit 3 --methods sp,aliked

# Use full dataset from data/ directory
python main.py --data

# Include cross-batch negative control
python main.py --cross

# Available --methods values: sp, aliked, sift, hardnet, loftr, roma, all

# Test MatchAnything model separately
python test_matchanything.py

# QR detection diagnostics across pipeline stages
python qr_diagnostic.py
```

No formal test suite, linter, or build system exists.

## Architecture

### Pipeline (`pipeline/`)

The `pipeline/` package is the core. `runner.py` orchestrates everything; other modules are single-responsibility:

```
runner.py          — Orchestrator: pair discovery, preprocessing, alignment, matching loop, result aggregation
config.py          — All constants, device setup, SAM model singleton, rotation/async-save utilities
preprocess.py      — Full preprocessing: load → resize → sticker detect → ROI extract → perspective rectify → CLAHE
sticker.py         — Blue sticker detection (HSV filtering + gradient-based edge refinement)
roi.py             — ROI extraction with 3 strategies: 'face' (SAM-based), 'sticker' (circular), 'square'
rectify.py         — Iterative perspective correction (ellipse → circle via homography, max 3 iterations with oscillation detection)
align.py           — Rotation alignment: rectangle axis-alignment + sticker quadrant matching, falls back to brute-force 4 candidates
matching.py        — Match filtering (exclude sticker zone) and scoring: final_score = 0.5 × confidence + 0.5 × RANSAC inlier_ratio
qr.py              — QR detection with 5-phase escalation (cv2 → upscale → ArUco → pyzbar fallback)
visualization.py   — Match overlays, heatmaps, debug diagnostics (pure output, no logic)
```

### Processing flow per image pair

1. **QR scan** → pair wet/dry images by specimen ID
2. **Sticker detection** → locate blue circular sticker (HSV + gradient refinement)
3. **ROI extraction** → SAM segments concrete face, crop to bounding box
4. **Perspective rectify** → iterative homography to make sticker ellipse circular
5. **Rectangle alignment** → rotate so concrete face edges are axis-aligned
6. **Sticker quadrant alignment** → determine rotation between wet/dry (0°/90°/180°/270°), or try all 4
7. **Feature matching** → extract + match + filter sticker zone + RANSAC scoring
8. **Verdict** → SAME (≥0.60) / DIFFERENT (<0.60) / INSUFFICIENT (<4 matches) / INVALID (no sticker/QR)

### Entry points

- `main.py` — Multi-method comparison runner (SuperPoint, ALIKED, SIFT, DoGHardNet, LoFTR, RoMa)
- `test_matchanything.py` — Standalone MatchAnything (EfficientLoFTR) evaluation; loads model from `MatchAnything_hf/imcui/third_party/MatchAnything/` via `sys.path` injection
- `demo.py` / `demo_v2.py` — Earlier iteration scripts (large, mostly superseded by main.py)

### MatchAnything_hf/

Vendored copy of the [MatchAnything](https://github.com/zju3dv/MatchAnything) repository (universal cross-modality matcher). Used only by `test_matchanything.py`. Model weights live at `MatchAnything_hf/imcui/third_party/MatchAnything/weights/`.

## Key Conventions

### Scoring formula

```python
raw_score = 0.5 * mean_confidence + 0.5 * inlier_ratio
final_score = raw_score * (n_matches / MIN_MATCHES)  # penalty if n < MIN_MATCHES
```

Verdict thresholds: SAME ≥ 0.60, DIFFERENT < 0.60, INSUFFICIENT if filtered_matches < 4.

### Critical constants (`pipeline/config.py`)

| Constant | Value | Purpose |
|----------|-------|---------|
| `MATCH_THRESHOLD` | 0.60 | SAME/DIFFERENT cutoff |
| `MIN_MATCHES` | 15 | Below this, score is linearly penalized |
| `MAX_KEYPOINTS` | 2048 | Feature extraction cap |
| `ROI_SCALE` | 2.5 | ROI radius = sticker_radius × 2.5 |
| `MAX_SIZE_WET` | 1024 | Wet image max dimension |
| `MAX_SIZE_DRY` | 1536 | Dry image max dimension |

### Data layout

- `samples/` — Test images: `{supplier}_{specimen_id}_{dry|wet}{1-3}.{jpg|jpeg}`
- `data/` — Full dataset organized by supplier subdirectories, paired via QR code content
- `output_v2/` — Results: `{method}/{batch}/match_{n}.png`, `heatmap_{n}.png`, `debug/`

### Matcher-specific pipelines

Sparse matchers (SuperPoint, ALIKED, SIFT, DoGHardNet) use `run_method()` → LightGlue produces match indices + scores → `filter_matches()` + `compute_score()`.

Dense matchers (LoFTR) use `run_method_loftr()` → outputs point coordinates directly → `filter_points_by_mask()` + `compute_score_from_points()`.

RoMa uses `run_method_roma()` with its own warp-based dense matching pipeline.

### Dependencies (no requirements.txt — install manually)

Core: `torch`, `opencv-python`, `numpy`, `segment-anything`, `lightglue`, `kornia` (for LoFTR), `romatch` (for RoMa). Optional: `pyzbar` (QR fallback).

SAM checkpoint must be at the path defined in `config.py` (`sam_vit_b_01ec64.pth`).

---
> Source: [keeker765/concrete-audit](https://github.com/keeker765/concrete-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
