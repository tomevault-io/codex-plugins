# EasyCut Development Guide

EasyCut is a Windows desktop YouTube downloader, live stream recorder, and live clipper built with Python 3.10+ and Tkinter. All source code lives in a flat `src/` directory.

## Build, Test, and Lint

```bash
# Install dependencies (use venv)
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Run the application
python main.py

# Run all tests
.\venv\Scripts\python.exe -m pytest tests/

# Run a single test
.\venv\Scripts\python.exe -m pytest tests/test_config_manager.py::TestConfigManagerDefaults::test_default_config_is_dict -v

# Run tests matching a pattern
.\venv\Scripts\python.exe -m pytest tests/ -k "config" -v

# Build standalone executable (requires PyInstaller and build_config.json with OAuth credentials)
pip install pyinstaller
python build.py
# Output: dist/EasyCut.exe
```

## Architecture Overview

### Module Responsibilities

| Module | Purpose |
|--------|---------|
| `main.py` | Entry point: sets window icon, creates Tk root, launches `EasyCutApp` |
| `src/easycut.py` | Main orchestrator (~6,800 lines): 8-tab UI, download/batch/live/clip logic, threading |
| `src/i18n.py` | Translation engine with 509+ keys across 7 languages (EN, PT, ES, FR, DE, IT, JA) |
| `src/design_system.py` | Design tokens: 6 accent color families, gradients, typography, spacing, ttk themes |
| `src/modern_components.py` | 16 custom Tkinter widgets (ModernButton, ModernCard, ScrollableFrame, etc.) |
| `src/video_player.py` | Embedded mpv player via subprocess + JSON IPC over Windows Named Pipes |
| `src/oauth_manager.py` | YouTube OAuth 2.0 authentication |
| `src/post_processor.py` | Post-download format conversion, trimming, audio extraction |
| `src/channel_monitor.py` | YouTube channel monitoring for new uploads |
| `src/icon_renderer.py` | SVG Feather icons → Pillow → PhotoImage (no external SVG deps) |
| `src/ui_enhanced.py` | ConfigManager, LogWidget, StatusBar utilities |

### Key Data Flows

**Downloads**: User action → `EasyCutApp.start_download()` → background thread → `yt_dlp.YoutubeDL().download()` → progress logged to `LogWidget` → saved to `config/history_downloads.json`

**Theme/Language Switch**: Toggle → save to `config/config.json` → `setup_ui()` rebuilds entire UI → instant hot-reload (~200ms)

**Live Recording**: Verify stream → optional embedded player preview → background thread records with yt-dlp → clipper marks start/end points → ffmpeg extracts clips

### Threading Model

- All downloads, recordings, and processing run in background threads
- UI updates from background threads **must** use `root.after()` for thread safety
- The embedded mpv player communicates via a background IPC thread

### Configuration Files (runtime, in `config/`)

- `config.json` — User settings (theme, language, output folder)
- `credentials.json` — OAuth credentials (developers only, gitignored)
- `youtube_token.json` — Saved OAuth tokens (gitignored)
- `yt_cookies.txt` — Cookies for yt-dlp authentication (gitignored)
- `history_downloads.json` — Download history entries

## Key Conventions

### Translation System

Use the singleton translator for all user-facing text:

```python
from i18n import translator as t
label_text = t.get("btn_download")  # Returns translated string
```

When adding new text, add keys to `TRANSLATIONS` dict in `src/i18n.py` for all 7 languages.

### Theme System

Access colors via `DesignTokens`:

```python
from design_system import ModernTheme, DesignTokens
tokens = DesignTokens(dark_mode=True)
bg = tokens.get_color("bg_primary")
accent = tokens.get_color("accent_primary")
```

Both dark and light palettes have 60+ semantic color tokens including 6 accent families (Blue, Purple, Orange, Red, Rose, Cyan).

### Icons

Two icon systems are available:

```python
# SVG icons (preferred) — renders Feather icons via Pillow
from icon_renderer import render_feather_icon
icon = render_feather_icon("download", size=20, color="#8B5CF6")

# PNG icons with emoji fallback
from icon_manager import icon_manager
icon = icon_manager.get_icon("download", size=24)
```

### Adding a New Tab

1. Create `create_my_tab(self)` method in `easycut.py`
2. Register in `self.sections` dict in `setup_ui()`
3. Add translation keys to `src/i18n.py` for all 7 languages

### External Dependencies

- **yt-dlp**: Video downloading
- **FFmpeg**: Required for post-processing, audio conversion, trimming (install via `choco install ffmpeg` or `winget install FFmpeg`)
- **mpv**: Optional, for embedded video player preview

### OAuth Setup (Developers)

Copy `config/credentials_template.json` to `config/credentials.json` and fill in your Google OAuth credentials.

### Build for Distribution

The `build.py` script embeds OAuth credentials into the executable. Requires `build_config.json` with your production OAuth credentials.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dekouninter)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dekouninter)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
