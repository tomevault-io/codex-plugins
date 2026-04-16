# Data Analysis Tool Copilot Instructions

## Overview
PyQt5-based desktop application for importing and analyzing data from Excel files. Organizes data into projects stored in SQLite databases, with table views for exploration.

## Architecture
- **app/**: Application controller, context, and project management (e.g., [app/app_controller.py](../../app/app_controller.py) handles menu actions and import flows).
- **ui/**: PyQt5 UI components including main window with stacked widget, dialogs, and table models (e.g., [ui/main_window.py](../../ui/main_window.py) owns menu bar and bottom status banner).
- **db/**: SQLite utilities and raw data ingestion (e.g., [db/sqlite_utils.py](../../db/sqlite_utils.py) provides lightweight helpers without ORM).
- **core/**: Shared services like logging, settings, name scrubbing, and UI theming.

Data flows from file imports → RawIngestService → SQLite tables → TableModel → QTableView display.

## Key Patterns
- **Logging**: Always use LoggingService for messages; set `scope_project=True` to duplicate to project logs. Errors include `exc=` parameter and traceback.
- **Signals**: Use PyQt signals for UI-controller decoupling (e.g., `request_import_files` from MainWindow to AppController).
- **Raw Imports**: Excel files create `raw_` tables with all columns as TEXT; use NameScrubber for column name cleaning.
- **Project Structure**: Projects at `./app/projects/<name>/db/<name>.db`; logs at `./app/projects/<name>/logs/`.
- **UI Updates**: Update bottom banner via MainWindow methods (e.g., `update_current_task`, `update_status_text`) during operations.

## Workflows
- **Run App**: `python main.py` (activate virtual environment first, as shown in terminal).
- **Import Data**: Select Excel files via menu; creates raw tables, updates banner, auto-closes dialog; failures show FailedImportsDialog.
- **Logs**: Global logs in `./app/logs` (rotated last 30 days); project logs append-only, no UI access.

## Dependencies
- PyQt5 (UI framework)
- openpyxl (Excel reading)
- sqlite3 (built-in Python)

Dependencies listed in [README.md](../README.md); no requirements.txt or pyproject.toml.

## Conventions
- **Error Handling**: Catch exceptions, log with `LoggingService.error`, show `QMessageBox.critical` to user.
- **File Paths**: Use `os.path.join` for cross-platform paths, relative to project root.
- **UI Theme**: Classic theme applied via `apply_classic_theme` in main.py.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JohnERyback) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
