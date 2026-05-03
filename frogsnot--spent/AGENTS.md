# Spent - Personal Finance Tracker

## Project Overview
Minimalist, local-first personal finance application for Linux desktop with cross-platform support.

## Technology Stack
- Backend: Tauri v2 (Rust) with SQLite (rusqlite)
- Frontend: Svelte + TypeScript
- Styling: TailwindCSS with dark mode
- Database: SQLite for local storage

## Architecture
- All data stored in local SQLite database
- No cloud sync, no accounts required

## Key Features
- Offline operation
- Keyboard navigation (Ctrl+N for quick entry)
- Transaction tracking with categories
- Monthly balance calculations
- Dark mode

## Development Guidelines
- Store money as integers (cents) to avoid floating-point errors
- All database operations in Rust backend
- Frontend communicates via Tauri commands
- Minimize dependencies for small binary size

---
> Source: [FrogSnot/Spent](https://github.com/FrogSnot/Spent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
