# ARS Project Instructions

## The most important aspect!
- **AVOID OVERENGINEERING**: Always choose the simplest solution that works. Avoid unnecessary complexity, Keep solutions simple, functional, and direct.
- **Minimalism**: Less code is better. Avoid unnecessary abstractions or complex patterns unless absolutely required.
- **Pragmatism**: Focus on working features over theoretical purity.

## Project Overview
ARS is a Python-based 3D application built with **PyQt6** (UI) and **Vispy** (3D rendering). It features a unique "Floating Bubble" interface and integrates with **ComfyUI** for rendering workflows.

## Architecture
- **Entry Point**: `main.py` initializes `Application` and `MainWindow`.
- **UI Framework**: PyQt6. Main window logic in `ui/main_window.py`.
- **3D Engine**: Located in `ars_3d_engine/`.
  - `ViewportWidget` (`ars_3d_engine/viewport.py`): Core 3D view using `vispy.scene`.
  - `CObjectManager`: Manages 3D objects.
  - `GizmoController`: Handles object manipulation gizmos.
- **Command System**: Located in `ars_cmds/`. Implements a command pattern where features are modular "Bubbles".
- **Theme & Resources**: `theme/` handles fonts and icons. `res/` contains assets.
- **Hotkeys**: `hotkeys/` manages input profiles.

## Development Conventions

### Command / Bubble Pattern
New features are often added as "Bubbles" in `ars_cmds/bubble_cmds/`.
To add a new command:
1. Create a new `.py` file in `ars_cmds/bubble_cmds/`.
2. Define a configuration dictionary named `BBL_<NAME>_CONFIG`:
   ```python
   from theme.fonts import font_icons as ic
   BBL_MYCMD_CONFIG = {
       "symbol": ic.ICON_NAME, # Use constant from font_icons
       "hotkey": "Ctrl+K"      # Optional default hotkey
   }
   ```
3. Define the entry function `BBL_<NAME>`:
   ```python
   from ars_cmds.core_cmds.run_ext import run_ext
   def BBL_MYCMD(*args):
       run_ext(__file__) # Delegates to execute_cmd
   ```
4. Define `execute_cmd(ars_window)` for the actual logic (typically opens a context menu):
   ```python
   from ui.widgets.context_menu import ContextMenuConfig, open_context
   
   def execute_cmd(ars_window):
       config = ContextMenuConfig()
       config.options = {
           ic.ICON_OPTION_1: 'Option 1',
           ic.ICON_OPTION_2: 'Option 2'
       }
       config.callbackL = {
           ic.ICON_OPTION_1: lambda: print("Op 1"),
           ic.ICON_OPTION_2: lambda: print("Op 2")
       }
       open_context(config)
   ```

### Icons & Fonts
- **Source**: Icons are defined in `theme/fonts/style.css` and parsed by `theme/fonts/font_icons.py`.
- **Usage**: Import `theme.fonts.font_icons` as `ic` and use `ic.ICON_<NAME>` constants.
- **Loading**: `theme/fonts/new_fonts.py` handles font loading (defaulting to Arial if missing).

### Hotkeys & Bubbles
- **Registration**: `ars_cmds/core_cmds/define_hotkeys.py` dynamically scans `ars_cmds/bubble_cmds/` for `BBL_*` functions and registers hotkeys from their `*_CONFIG`.
- **UI Distribution**: `ars_cmds/core_cmds/distribute_bubbles.py` loads bubbles into the overlay.
  - **Left Click**: Executes the `BBL_*` function.
  - **Right Click**: Opens a dropdown to edit the bubble's code (`r_dropdown`).

## Workflows
- **Run Application**: `python main.py`
- **Environment**: Requires `PyQt6`, `vispy`, `pygame`, `numpy`, `scipy`.
- **Assets**: Resources are in `res/` (icons, sounds, meshes).

## Key Files
- `main.py`: Application bootstrap.
- `ui/main_window.py`: Main UI layout and initialization.
- `ars_3d_engine/viewport.py`: 3D scene setup.
- `ars_cmds/bubble_cmds/`: Directory for adding new application commands.
- `theme/fonts/font_icons.py`: Icon definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/merkvilson)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/merkvilson)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
