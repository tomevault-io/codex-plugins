# SSH Device Manager - AI Coding Instructions
# SSH Device Manager - AI Coding Instructions

## Architecture Overview

SSH_DeviceManager is a **Tkinter GUI application** for remote device management via SSH/SFTP. The codebase is organized as a Python package (`ssh_device_manager/`) with a thin backward-compatibility launcher (`SSH_DeviceManager.py`).

### Package Layout

```
ssh_device_manager/          # Main package
    __init__.py              # Re-exports public API
    models.py                # ActionButton, ButtonSection, ToolTip
    ssh_manager.py           # SSHManager (Paramiko wrapper)
    themes.py                # THEMES dictionary (18 built-in themes)
    config.py                # App config / profile persistence
    constants.py             # Shared app constants (limits, file paths)
    paramiko_compat.py       # Clean import when paramiko is absent
    sections_loader.py       # JSON section loading + handler resolution
    validation.py            # Input validation helpers
    output.py                # OutputManager (log queue, append, clear, copy, save)
    app.py                   # SSHGuiApp (Tkinter orchestrator)
    controllers/             # Focused controllers
        connection.py        # Connection lifecycle
        actions.py           # SSH actions and file uploads
        profiles.py          # Profile CRUD
        sections.py          # Section loading, rendering, file watching

SSH_DeviceManager.py         # Thin launcher / backward-compat shim
test_SSH_DeviceManager.py    # 100 unit + integration tests
customizer.py                # Standalone sections.json editor
docs/                        # Test matrix, Gherkin specs, reading guide
```

### Module Responsibilities

| Module | Responsibility | Key Classes/Functions |
|---|---|---|
| `models.py` | UI data structures | `ActionButton`, `ButtonSection`, `ToolTip` |
| `ssh_manager.py` | SSH/SFTP via Paramiko | `SSHManager` |
| `themes.py` | Color theme definitions (18 themes) | `THEMES` dict |
| `config.py` | Profile persistence to JSON | `load_app_config()`, `save_app_config()` |
| `constants.py` | Shared app constants | `COMMAND_HISTORY_LIMIT`, `APP_CONFIG_FILE`, `DEFAULT_SECTIONS_FILE` |
| `paramiko_compat.py` | Safe paramiko import | `paramiko` (real or stub) |
| `sections_loader.py` | Button definitions from JSON | `load_sections_from_file()` |
| `validation.py` | Connection form validation | `get_connection_inputs()`, `parse_int_input()`, `get_host_key_mode()` |
| `output.py` | Thread-safe terminal output | `OutputManager` |
| `app.py` | Tkinter GUI orchestrator | `SSHGuiApp` |
| `controllers/` | Delegated app behaviors | `ConnectionController`, `ActionsController`, `ProfilesController`, `SectionsController` |

## Critical Architecture Patterns

### Thread-Safe Logging with Queue
All network operations run in daemon threads that communicate with the main UI thread via `self.log_queue` (a `queue.Queue`). The `log()` method safely timestamps and queues messages; the log poller runs on the main thread every 80ms to dequeue and display output.

**Example:** `SSHGuiApp.run_ssh_command()` spawns a thread, logs via `self.log()`, not direct output writes.

### Modular Button Sections
Buttons are organized into `ButtonSection` objects. Sections can be defined:
- **Built-in:** via `_define_sections()` in `app.py` (fallback)
- **External JSON:** via `sections_loader.load_sections_from_file()` with handler tokens (`__upload_template__`, `__send_file__`, `__custom_command__`, `run:command`)

Each section has `title`, `max_buttons` (hard limit), and a list of `ActionButton` objects. Disabled buttons are excluded from rendering.

### Theme System
`THEMES` dictionary in `themes.py` maps theme names to color dictionaries. `apply_theme()` in `app.py` modifies `ttk.Style` and raw Tk widgets. Themes are applied dynamically via menu selection.

### Host History (Combobox)
The host field uses `ttk.Combobox` with a special `<Clear History>` option. History persists only during runtime; it's not saved to disk. Capped at 10 entries.

## Key Developer Workflows

### Running Tests
```powershell
python -m unittest test_SSH_DeviceManager.py
```
Tests use `unittest.mock` to mock `paramiko`, `tkinter`, and file dialogs. Tkinter is mocked at import time in the test file. All 100 tests (84 unit + 16 integration) run in under 1 second.

### Adding a New Button
1. In `_define_sections()` in `app.py`, add an `ActionButton` to the appropriate `ButtonSection`
2. Or add to `sections.json` � the app auto-reloads on file changes
3. Set `enabled=True` and provide a handler function
4. If max_buttons is exceeded, excess buttons are truncated with a warning logged

### Extending SSH Commands
Commands execute via `SSHManager.run_command()` in `ssh_manager.py` which uses `client.exec_command()` (non-interactive). For interactive shells (network device paging), override with `client.invoke_shell()`.

### File Transfer
Two methods exist in `app.py`:
- `upload_config_template()`: Fixed remote path (/tmp/uploaded_config.txt)
- `send_file_scp()`: User-specified remote path via dialog
Both use SFTP under the hood (`SSHManager.upload_file()` in `ssh_manager.py`).

## Project-Specific Conventions

### Connection Management
- `SSHManager.connect()` defaults to `WarningPolicy` for host keys (configurable: strict, warning, auto)
- Timeout defaults: 10s for connection, 30s for commands
- `disconnect()` uses `contextlib.suppress()` to silently handle cleanup errors
- The `clear_creds_var` checkbox optionally clears password on disconnect

### Output Handling
- Terminal output is read-only by default (disabled state)
- Editing only occurs in `_append_output()` method
- Output can be cleared, copied to clipboard, or saved to file via buttons
- All timestamps are added by `log()` in HH:MM:SS format

### Error Handling
- Connection errors are logged, not raised (background threads suppress exceptions)
- Disconnection is graceful (uses `suppress()` for both SFTP and SSH cleanup)
- Invalid UTF-8 bytes in output are replaced with U+FFFD (see `run_command()` decode with `errors="replace"`)

### Testing Pattern
Tests mock external dependencies (Paramiko, Tkinter, file dialogs) to avoid side effects. The test file mocks tkinter before importing the main module to prevent GUI initialization. All UI state is mocked; threading is mocked to run synchronously.

**Important:** When patching `SSHManager` for tests that exercise `test_connection()` (which creates a *new* SSHManager instance), patch `ssh_device_manager.app.SSHManager`, not `SSH_DeviceManager.SSHManager`, because the app module has its own import reference.

## References

### Key Files
- `ssh_device_manager/app.py`: Main GUI orchestrator
- `ssh_device_manager/ssh_manager.py`: Paramiko wrapper
- `ssh_device_manager/models.py`: Data models (ActionButton, ButtonSection)
- `ssh_device_manager/themes.py`: Theme definitions
- `ssh_device_manager/validation.py`: Input validation
- `ssh_device_manager/config.py`: Profile/config persistence
- `ssh_device_manager/sections_loader.py`: JSON section loading
- `ssh_device_manager/output.py`: Output manager
- `SSH_DeviceManager.py`: Thin launcher / backward-compat shim
- `test_SSH_DeviceManager.py`: 100 unit + integration tests
- `docs/TEST_MATRIX.md`: Test IDs, descriptions, requirements traceability
- `docs/TEST_GHERKIN.md`: Gherkin behavioral specifications
- `docs/READING_GUIDE.md`: How to navigate the test documentation

### Constants
- `COMMAND_HISTORY_LIMIT = 500`: Max recent commands cached
- Host history max: 10 entries (hardcoded in `on_connect()`)
- Default theme: "Default" (light colors)
- Queue poll interval: 80ms (see `_start_log_poller()`)

## Common Tasks

**Add a status check button:** Create an ActionButton in "Status" section with `handler=lambda: self.run_ssh_command("show status")`

**Add a new theme:** Add entry to `THEMES` dict in `themes.py`, then select via Theme menu (auto-updates)

**Test a connection without connecting:** Use "Test Connection" button which spawns a temporary SSHManager, connects, and disconnects

**Access output programmatically:** Use `self.output_text.get("1.0", "end-1c")` to read all content as string

---
> Source: [esanacore/SSH_DeviceManager](https://github.com/esanacore/SSH_DeviceManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
