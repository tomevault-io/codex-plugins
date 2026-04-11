# Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a Flutter memory matching game (Hafıza Oyunu) project built with MVVM architecture and Riverpod state management.

## Architecture Guidelines
- **MVVM Pattern**: Use Models for data structures, Views for UI components, and ViewModels for business logic
- **State Management**: Use Riverpod for all state management operations
- **Code Generation**: Use riverpod_generator for provider generation

## Key Features
- Card matching game with 4x4 grid (expandable)
- Multiple themes: Animals, Fruits, Countries, Colors
- Collection system to track matched items
- Information panels with details about matched items
- Score tracking and completion screen

## Coding Standards
- Use Turkish comments for UI text and game content
- Follow Flutter/Dart naming conventions
- Implement proper error handling
- Use animations for card flips and transitions
- Maintain responsive design principles

## File Organization
- `/lib/models/` - Data models and entities
- `/lib/views/` - UI screens and widgets
- `/lib/viewmodels/` - Business logic and state management
- `/lib/services/` - External services and APIs
- `/lib/utils/` - Helper functions and constants
- `/lib/data/` - Static data and theme content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Berkant-Y)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Berkant-Y)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
