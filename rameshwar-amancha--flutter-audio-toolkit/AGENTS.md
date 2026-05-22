# Copilot Instructions for Audio Converter Waveform Plugin

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

This is a Flutter plugin project for

1. audio conversion
2. trim audio file with range selection
3. waveform extraction with visualualization options
4. generate noise levels report as noise meter.
5. audio player with true waveform visualization (customizable waveform styles, play, pause buttons and their positions, sizes)
6. audio player with fake waveform visualization (customizable waveform styles, play, pause buttons and their positions, sizes)

## Key Requirements:

- Use native implementations only (no FFmpeg or deprecated libraries)
- Android: MediaCodec and MediaMuxer for audio conversion, MediaExtractor for waveform data
- iOS: AVAssetExportSession/AVAudioConverter for conversion, AVAssetReader for waveform data
- Support audio formats: mp3, wav, ogg → aac, m4a
- Support audio trimming with range selection
- Extract waveform data for visual display
- Follow Flutter plugin best practices
- Follow Dart file conventions and Flutter guidelines
- Provide comprehensive documentation and examples
- Use proper error handling and async operations
- Implement comprehensive testing

## Platform-specific considerations:

- Android: Handle permissions for audio file access
- iOS: Handle AVAudioSession configuration
- Both: Implement proper memory management for audio processing

---
> Source: [Rameshwar-Amancha/flutter_audio_toolkit](https://github.com/Rameshwar-Amancha/flutter_audio_toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
