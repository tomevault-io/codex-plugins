<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# AutoTablature - AI-Powered Guitar Tablature Generation

## Project Overview
This project is an AI-powered system that analyzes MP3 files and generates guitar tablature in Guitar Pro format. It includes modules for audio processing, instrument separation, pitch detection, rhythm analysis, and tablature generation.

## Code Architecture
- `src/audio/`: Audio loading and processing
- `src/separation/`: Instrument track separation using machine learning
- `src/models/`: AI models for pitch and rhythm detection
- `src/tablature/`: Guitar tablature generation
- `src/utils/`: Utility functions for configuration, logging, etc.

## Libraries and Technologies
- PyTorch and TensorFlow for machine learning models
- Librosa for audio analysis
- Guitar Pro format libraries for tablature output
- Music21 for music theory operations

## Development Focus
Focus on making the code:
- Accurate in detecting notes, timing, and assigning them to appropriate strings/frets
- Efficient in separating instrument tracks and processing audio
- Extensible to support different instruments and music styles
- Well-documented with clear comments and typing

## Code Style Guidelines
- Follow PEP 8 conventions
- Use type hints
- Write docstrings for classes and functions
- Include unit tests for key functionality
- Keep model architectures clear and well-commented

## Domain-Specific Knowledge
Remember that guitar tablature has specific constraints:
- Notes should be assigned to appropriate strings and frets based on playability
- Guitar standard tuning is E, A, D, G, B, E (from lowest to highest)
- Rhythm notation should follow standard music notation conventions
- Guitar Pro format supports multiple tracks, effects, and playing techniques

---
> Source: [webprofusion/autotablature](https://github.com/webprofusion/autotablature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
