# Vocal Reader Project Instructions

This project creates a voice cloning system that can:
1. Record user's voice for training
2. Extract text from uploaded PDF files  
3. Generate synthetic speech that mimics the user's voice reading the PDF content

## Architecture
- **Frontend**: React with drag-and-drop PDF upload and voice recording
- **Backend**: Python Flask API for processing
- **ML Models**: Coqui TTS for voice synthesis and cloning
- **PDF Processing**: PyPDF2 for text extraction
- **Audio**: Web Audio API for recording, librosa for processing

## Key Features
- PDF drag-and-drop interface
- Voice recording with real-time feedback
- Text extraction from PDFs
- Voice training pipeline
- Synthetic speech generation
- Audio playback controls

## Tech Stack
- Python (Flask, PyTorch, Coqui TTS, PyPDF2, librosa)
- React (JavaScript, Web Audio API, File API)
- Open-source ML models for voice cloning

## Development Guidelines
- Use free and open-source tools only
- Prioritize user experience and audio quality
- Implement proper error handling
- Follow security best practices for file uploads
- Optimize for real-time audio processing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JKDrewes)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/JKDrewes)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
