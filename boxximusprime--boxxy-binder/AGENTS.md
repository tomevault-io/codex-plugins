# Star Citizen Joystick Mapper - Tauri + Rust Project

## Project Overview
A desktop application for managing Star Citizen joystick, keyboard, and mouse key bindings outside of the game.

## Technology Stack
- **Backend**: Rust
- **Frontend**: HTML/CSS/JavaScript (default Tauri setup)
- **Framework**: Tauri 2.0

## Setup Checklist

- [x] Create copilot-instructions.md file
- [x] Get Tauri project setup information
- [x] Install prerequisites (Rust, Node.js, MSVC Build Tools)
- [x] Scaffold Tauri project structure
- [x] Organize project files
- [x] Install dependencies and compile
- [x] Create initial documentation
- [x] Successfully build and run application

## ✅ Setup Complete!

The Tauri + Rust development environment is fully configured and working.

## Project Details

- **Name**: Star Citizen Joystick Mapper
- **Framework**: Tauri 2.0
- **Backend**: Rust
- **Frontend**: Vanilla HTML/CSS/JavaScript
- **Target Platform**: Windows Desktop

## Key Commands

```powershell
cargo tauri dev          # Development mode with hot reload
cargo tauri build        # Build production installer
cargo check              # Check Rust code
cargo fmt                # Format Rust code
cargo clippy             # Lint Rust code
```

## Development Notes
- Tauri allows building lightweight desktop apps with web frontends
- Rust backend handles file I/O for Star Citizen keybinding files
- Frontend provides UI for mapping configuration

## Version Management
- Version is maintained in 4 places: `tauri.conf.json`, `Cargo.toml`, `package.json`, and `index.html`
- The HTML header dynamically loads the version from Cargo.toml at runtime
- To update version: change all 4 files to keep them in sync
- See VERSION.md for details

---
> Source: [BoxximusPrime/Boxxy-Binder](https://github.com/BoxximusPrime/Boxxy-Binder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
