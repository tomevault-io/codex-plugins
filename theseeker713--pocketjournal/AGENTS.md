# PocketJournal - AI Coding Agent Instructions

## ⚠️ Development Status
**This application is currently in active development and may contain bugs, incomplete features, or unexpected behavior. Use at your own discretion.**

## Project Overview
PocketJournal is a Windows-first personal journaling application built with PySide6, developed by DigiArtifact. The app features a modern, writing-focused interface with a unique micro-launcher system supporting two dock modes: corner launcher (floating circular button) and system tray mode.

## Architecture & Key Components

### Core Structure
- **Main App**: `src/pocket_journal/main.py` - Entry point with `PocketJournalMainWindow`
- **Launcher System**: `ui/launcher_manager.py` coordinates between `micro_launcher.py` (circular UI) and `editor_panel_integrated.py` (expandable panel)
- **Settings**: `settings.py` uses platformdirs for cross-platform data storage with comprehensive defaults
- **Entry Management**: `data/entry_manager.py` handles journal entries with YAML front-matter metadata
- **Smart Formatting**: `core/smart_formatting.py` provides display-time text formatting without mutating raw content

### UI Architecture Pattern
The app uses a manager pattern where `LauncherManager` orchestrates:
- Corner mode: `CircularLauncher` + expandable `IntegratedEditorPanel`
- Tray mode: `SystemTrayManager` + panel access
- Shared: Settings, formatting, autosave, and entry management

### Version Management
- `version.json` at root defines version/build info
- `app_meta.py` loads and exposes constants (`APP_NAME`, `VERSION`, `CHANNEL`)
- Supports dev/beta/release channels for different build types

## Development Workflow

### Environment Setup
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -e .  # Development install
```

### Testing
- Uses pytest with pytest-qt for PySide6 testing
- Test files follow `test_*.py` pattern in `tests/`
- Example: `test_main.py` tests window creation and basic functionality
- Run: `pytest` from project root

### Project Scripts
- `launch.ps1` / `launch.bat` - Development launchers
- `scripts/build.py` - Build automation
- Entry points: `pocket-journal` command or `python -m pocket_journal`

## Critical Patterns

### Settings Management
Settings use a centralized pattern with `DEFAULT_SETTINGS` dict in `settings.py`. Access via:
- `get_setting(key, default)` - Retrieve with fallback
- `set_setting(key, value)` - Persist changes
- Settings automatically persist to user data directory

### Entry Format
Journal entries use YAML front-matter with `EntryMetadata` dataclass:
```python
@dataclass
class EntryMetadata:
    id: str
    created_at: str  # ISO UTC timestamp
    updated_at: str
    title: str
    # ... more fields
```

### Component Communication
Use Qt signals extensively for loose coupling:
- `LauncherManager` emits `panel_expanded`, `panel_collapsed`, `dock_mode_changed`
- `SmartFormatter` emits `formatting_changed`, `rules_updated`
- System tray emits `show_window_requested`, `settings_requested`

### Step-Based Development
The project uses incremental "steps" (see `STEP*_SUMMARY.md` and `demo_step*.py`). Each step builds specific features with acceptance criteria. When implementing new features, follow this pattern of focused, testable increments.

## Key Dependencies
- **PySide6**: UI framework (≥6.6.0)
- **platformdirs**: Cross-platform app data directories
- **tinydb**: Lightweight JSON database (currently unused but planned)
- **markdown-it-py**: Markdown processing
- **pyyaml**: YAML front-matter parsing

## Testing Strategy
- Unit tests for core logic (entry management, formatting rules)
- Qt-based widget tests using pytest-qt
- Mock external dependencies (file system, settings)
- Test both corner and tray dock modes for launcher features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheSeeker713) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
