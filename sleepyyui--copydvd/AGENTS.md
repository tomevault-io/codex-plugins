# GitHub Copilot Instructions for CopyDVD

## Project Context
CopyDVD is a cross-platform DVD ripping application written in Rust with both GUI (egui) and CLI interfaces. It integrates with HandBrake for transcoding and provides multi-threading, upload capabilities, and comprehensive DVD analysis.

## Code Analysis Focus Areas

### When generating release notes, please analyze:

1. **Core Functionality Changes**
   - DVD detection and scanning improvements
   - HandBrake integration enhancements  
   - Ripping workflow optimizations
   - Upload system modifications

2. **User Interface Updates**
   - GUI component changes (egui-based)
   - New tabs or UI elements
   - Accessibility improvements
   - User experience enhancements

3. **Performance & Reliability**
   - Multi-threading optimizations
   - Error handling improvements
   - Memory usage optimizations
   - Cross-platform compatibility fixes

4. **Platform-Specific Changes**
   - macOS security/quarantine handling
   - Windows dependency management
   - Linux distribution support
   - Cross-compilation improvements

5. **Configuration & Settings**
   - New configuration options
   - Settings migration improvements
   - Default value changes
   - Preset modifications

## Release Notes Style Guide

### Preferred Language:
- Use action-oriented language ("Added", "Fixed", "Improved")
- Focus on user benefits rather than technical implementation
- Highlight breaking changes prominently
- Include performance impact when relevant

### Structure Preference:
- Lead with most impactful user-facing changes
- Group related changes together
- Use bullet points for clarity
- Include code examples for API changes

### Technical Detail Level:
- Balance technical accuracy with user-friendliness
- Explain the "why" behind changes when helpful
- Mention affected platforms when relevant
- Include migration guidance for breaking changes

## Key Dependencies to Watch:
- egui/eframe (GUI framework)
- tokio (async runtime)
- HandBrake CLI integration
- clap (CLI parsing)
- Platform-specific system APIs

## Common Change Patterns:
- Toast notification system updates
- HandBrake update mechanism improvements
- Cross-platform build configuration changes
- Error handling and logging enhancements
- Configuration management updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SleepyYui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
