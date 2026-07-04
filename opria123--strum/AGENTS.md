# STRUM - Copilot Instructions
## Spectral Transcription & Rhythm Understanding Model

## Project Overview
STRUM is an AI-powered audio-to-game-chart transcription pipeline that converts songs (WAV/MP3) into fully playable Clone Hero / YARG chart packages. The system produces MIDI charts for drums, guitar, bass, vocals, and keys — all from audio alone — with four difficulty levels (Expert/Hard/Medium/Easy).

## Tech Stack
- **Language**: Python 3.11+
- **ML Framework**: PyTorch 2.x
- **Audio Separation**: Demucs v4 (HTDemucs)
- **Pitch Detection**: librosa pYIN
- **Speech-to-Text**: OpenAI Whisper
- **MIDI I/O**: mido
- **Experiment Tracking**: Weights & Biases (W&B)
- **Config Management**: Hydra
- **CLI**: Click + Rich
- **Hardware**: NVIDIA DGX Spark (GB10, CUDA 12.8)

## Architecture Summary
```
Audio → Demucs v4 Separation → 5 Instrument Pipelines → Combined MIDI Chart
  │
  ├─ Drums:  Two-stage CRNN onset detector (93.9% F1) + 6-model ensemble classifier (85.2% F1)
  ├─ Guitar: Neural OnsetCRNN + librosa pYIN pitch + rule-based 5-fret mapping
  ├─ Bass:   Neural OnsetCRNN + librosa pYIN pitch + rule-based 5-fret mapping
  ├─ Vocals: Whisper word timestamps + pYIN pitch contour + dynamic alignment + lyrics
  └─ Keys:   Spectral keyboard detection + onset_detect + piptrack (5-lane + Pro Keys)
```

## Key Components

### Drums Pipeline (Flagship)
1. **V14 TwoStageDrumsCRNN** — Onset detection from mel spectrograms (128 bins, 22050 Hz)
2. **6-Model Ensemble** — OnsetClassifier V2/V4/V6/V12c/V15/V16 with PER_CLASS_WEIGHTS
3. **Spectral disambiguation** — Centroid analysis resolves tom/cymbal confusion
4. **Post-processing** — Bidirectional streak smoothing, kick-suppresses-FloorTom, quantization

### Tempo Detection
- Grid-alignment BPM refinement: ±5 BPM search at 0.1 resolution
- Circular statistics phase coherence for optimal alignment
- Tempo change detection (>3 BPM shift, ≥8 beat persistence)

## File Structure
```
strum/
├── configs/                          # Hydra configs
│   ├── drums_v14.yaml                # Two-stage drums config
│   ├── onset_classifier*.yaml        # Ensemble classifier configs
│   ├── inference.yaml                # Inference settings
│   └── preprocessing.yaml            # Preprocessing settings
├── checkpoints/                      # Trained model weights
│   ├── drums_v14/                    # Onset detector
│   └── onset_classifier_v*/          # Classifier ensemble
├── scripts/
│   ├── batch_pipeline.py             # Full multi-instrument pipeline
│   ├── batch_infer_hybrid.py         # Production drums pipeline
│   ├── guitar_hybrid.py              # Guitar/bass transcription
│   ├── vocals_charter.py             # Vocal transcription + lyrics
│   ├── keys_charter.py               # Keyboard detection + charting
│   ├── chart_postprocess.py          # Post-processing & quantization
│   ├── train_onset_classifier.py     # Classifier training
│   ├── train_guitar_onset.py         # Guitar onset training
│   └── preprocess_onset_windows.py   # Dataset preprocessing
├── src/
│   ├── cli.py                        # Click CLI entry point
│   ├── models/                       # Neural architectures + datasets
│   ├── preprocessing/                # Parsers, alignment, Demucs
│   ├── inference/unified.py          # Unified inference engine
│   ├── export/                       # MIDI + .chart export
│   ├── evaluation/                   # F1, precision, recall
│   └── lyrics/                       # LRCLIB + Lyrics.ovh fetcher
├── docs/                             # ARCHITECTURE.md, ROADMAP.md
└── pyproject.toml                    # Dependencies & config
```

## Coding Standards
- Type hints for all function signatures
- Google-style docstrings
- Config via Hydra (no hardcoded hyperparams)
- Log experiments to W&B
- Graceful error handling with informative messages

## Commands Reference
```bash
# Full pipeline (all instruments)
python scripts/batch_pipeline.py --songs-dir ./songs/ --output-dir ./charts/

# Drums only (production)
python scripts/batch_infer_hybrid.py --songs-dir ./songs/ --output-dir ./charts/

# CLI interface
strum preprocess --input-dir ./raw/ --output-dir ./processed/
strum train drums --config configs/drums_v14.yaml
strum infer drums --input song.wav --output song_drums.mid
strum chart --input song.wav --output-dir ./charts/
strum batch --manifest manifest.json --workers 4
```

## Important Context
- ~5k songs with existing Clone Hero/YARG charts used for training
- Pro drum charts with cymbal markers (primary training data)
- Virtual env: `/home/opria123/autocharter-env/bin/activate`
- Test songs: `/mnt/ml-data/sample-songs/`
- Chart output: `/mnt/ml-data/sample-songs-charts-v11/`
- Clone Hero MIDI: 480 ticks_per_beat, drum notes 96-100, tom markers 110-112

---
> Source: [opria123/strum](https://github.com/opria123/strum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-04 -->
