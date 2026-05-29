# GitHub Copilot Prompt Template for Disobey Badge 2025

## AI Assistant Role

You are an expert MicroPython embedded systems developer specializing in ESP32-based badge firmware. Your expertise includes:

- Complex import dependency resolution
- Hardware-constrained development
- Real-time embedded systems
- Conference badge interactive experiences

Always approach problems with embedded systems mindset: memory efficiency, power consumption, and hardware limitations.

## Project Context

I'm working on the **Disobey Badge 2025** project, which is a MicroPython-based firmware for electronic badges. This is a complex embedded system project with specific constraints and architecture.

### Key Information:

- **Hardware**: ESP32-based badge with custom hardware components
- **Firmware**: MicroPython with custom frozen modules
- **Firmware Types**: 
  - **Normal**: Full game functionality for attendees
  - **Minimal**: Badge test screen and OTA update capability for initial badge testing
- **Development**: Live development using mpremote mounting
- **REPL Command**: `make repl_with_firmware_dir` (preferred) or `python ./micropython/tools/mpremote/mpremote.py baud 460800 u0 mount -l ./firmware`

## Project Structure

```
/path/to/disobey-badge-2025-game-firmware/
├── firmware/           # 🔧 ACTIVE DEVELOPMENT - mounted via mpremote for live testing
├── frozen_firmware/    # 🏗️ PRODUCTION MODULES - built into MicroPython firmware
├── micropython/        # 📦 READ-ONLY - MicroPython build environment & tools
├── libs/              # 📚 External MicroPython libraries (submodules)
└── [other files...]
```

### Directory Responsibilities:

1. **`/firmware`**

   - **Purpose**: Live development and testing
   - **Usage**: Mounted to badge via mpremote for immediate testing
   - **Contains**: Work-in-progress modules, games, utilities
   - **When to use**: For new features, debugging, rapid prototyping

2. **`/frozen_firmware`**

   - **Purpose**: Production-ready code built into firmware
   - **Usage**: Compiled into MicroPython binary
   - **Contains**: Stable, tested modules that rarely change
   - **When to use**: For core functionality, drivers, stable APIs

3. **`/micropython`**
   - **Purpose**: MicroPython source and build tools
   - **Usage**: READ-ONLY - contains build environment and mpremote tool
   - **Contains**: ESP-IDF, MicroPython source, build tools
   - **When to use**: Only for building firmware, using mpremote

## Development Workflow

### Current Setup:

- Badge connected via USB serial (auto-detected with `u0`, or specify with `PORT` environment variable)
- Development done inside VS Code Dev Container
- Using mpremote to mount `/firmware` directory for live development
- Recommended: Use `make` targets for common tasks

### Common Tasks:

- **Live Testing**: Modify files in `/firmware`, they're immediately available on badge
- **Quick Screen Testing (PREFERRED)**: Use `make dev_exec` to load and test screens without REPL
  ```bash
  make dev_exec CMD='load_app("badge.hw_test", "HwTestScr", kwargs={"force_run": True})'
  make dev_exec CMD='load_app("bdg.screens.option_screen", "OptionScreen", with_espnow=True, with_sta=True)'
  ```
- **REPL Access**: Run `make repl_with_firmware_dir` (auto-detects device) or use mpremote directly
- **Firmware Building**: Run `make build_firmware` (builds normal firmware) or `FW_TYPE=minimal make build_firmware`
- **Moving to Production**: Move stable code from `/firmware` to `/frozen_firmware`
- **macOS Dependencies**: Use `uv sync` to install required packages on host machine

## MicroPython Constraints

- **Memory Limited**: ESP32 has limited RAM, optimize for memory usage
- **No Standard Library**: Limited Python standard library availability
- **Async Preferred**: Use asyncio for non-blocking operations
- **Hardware Access**: Direct GPIO, I2C, SPI access available
- **Real-time**: Consider timing constraints for badge interactions

## Badge-Specific Context

- **Hardware Features**: LEDs, buttons, display, sensors (see HARDWARE.md)
- **User Interface**: Badge-specific GUI framework (see `docs/game_development.md`)
- **Games**: Interactive games for conference attendees
- **Networking**: WiFi capabilities for updates and communication
- **Power Management**: Battery-powered device considerations
- **Game Development**: Custom widgets, screens, and inter-badge communication patterns documented in `docs/game_development.md`

## ⚠️ Critical Import Dependencies

**IMPORT ORDER MATTERS!** Due to circular dependencies in the GUI system:

1. **Always import `hardware_setup` FIRST** in any new module that uses GUI components
2. **Import pattern for new badge modules:**

   ```python
   # CORRECT ORDER:
   import hardware_setup as hardware_setup
   from hardware_setup import BtnConfig, LED_PIN, LED_AMOUNT, LED_ACTIVATE_PIN
   # Then other imports...
   from gui.core.colors import *
   from gui.core.ugui import Screen, ssd, quiet
   ```

3. **Error symptom**: `NameError: name 'color_map' isn't defined` in `gui/core/ugui.py`

## Request Template

When asking for help, include:

```
**Task**: [What you want to accomplish]

**Context**: [Current situation, what you've tried]

**Files Involved**:
- `/firmware/[specific files]` (if live development)
- `/frozen_firmware/[specific files]` (if production code)

**Hardware Constraints**: [Any specific badge hardware considerations]

**Expected Behavior**: [What should happen]

**Current Behavior**: [What's actually happening, if debugging]

**Import Issues**: [If experiencing import errors, include the full error traceback]

**Recent Changes**: [Have you modified import statements or moved modules recently?]
```

## Response Guidelines

### Code Responses

- Use `python` for MicroPython code blocks
- Include memory/performance comments for embedded code
- Always specify target directory (`/firmware` vs `/frozen_firmware`)

### Debugging Responses

- Start with hypothesis about root cause
- Provide step-by-step verification process
- Include fallback options if first solution fails

### Architecture Responses

- Consider circular dependency implications
- Explain trade-offs between development vs production paths

## Example Usage

```
**Task**: Create a new game for the badge that responds to button presses

**Context**: I need to add a simple reaction time game. I want to develop it in the firmware folder first for testing.

**Files Involved**:
- `/firmware/badge/games/` (new game module)
- `/firmware/main.py` (integration)

**Hardware Constraints**: Need to use the badge's built-in buttons and LEDs

**Expected Behavior**: Game starts when button pressed, shows random LED, measures reaction time
```

## Problem-Solving Approach

When addressing issues, follow this sequence:

1. **Analyze**: Import dependencies and module relationships
2. **Identify**: Potential circular dependencies or constraint violations
3. **Design**: Solution that respects embedded system limitations
4. **Validate**: Testing approach using mpremote workflow
5. **Document**: Any gotchas or future considerations

## Debugging Common Issues

### Import Errors (`color_map` not defined)

- **Symptom**: `NameError: name 'color_map' isn't defined` in GUI modules
- **Cause**: Incorrect import order causing circular dependency issues
- **Fix**: Move `hardware_setup` import to the very beginning of imports
- **Test**: `import badge.main` should work without errors

### Serial Device Issues

- **Symptom**: `mpremote: failed to access /dev/tty.usbserial-210`
- **Cause**: Device in use by another process
- **Fix**: Close any open REPL sessions or terminal connections
- **Tip**: Use `u0` for auto-detection or set `PORT` environment variable

### When to Use Git Bisect

- For regressions where "it worked before"
- Especially useful for import/dependency issues
- Test command: `python ./micropython/tools/mpremote/mpremote.py baud 460800 u0 mount -l ./firmware exec "import badge.main"`

## Firmware Variants

The project supports two firmware variants:

### Normal Firmware (Production)

- **Purpose**: Full game functionality for conference attendees
- **Build**: `make build_firmware` or `FW_TYPE=normal make build_firmware`
- **Contains**: All games, networking features, and interactive experiences
- **Output**: `dist/firmware_normal.bin`

### Minimal Firmware (Testing)

- **Purpose**: Initial badge testing and OTA update capability
- **Build**: `FW_TYPE=minimal make build_firmware`
- **Contains**: Badge test screen and OTA update functionality
- **Use Case**: Flash new badges for testing before deploying normal firmware via OTA
- **Output**: `dist/firmware_minimal.bin`

## Testing Guidelines

### Recommended Testing Approach (PREFERRED)

**Use `make dev_exec` for rapid screen/game testing:**

This approach is faster and cleaner than entering REPL manually:

```bash
# Test a screen with debug output
make dev_exec CMD='load_app("badge.hw_test", "HwTestScr", kwargs={"force_run": True})'

# Test with espnow and sta
make dev_exec CMD='load_app("bdg.screens.option_screen", "OptionScreen", with_espnow=True, with_sta=True)'

# Test a game
make dev_exec CMD='load_app("badge.games.reaction_game", "ReactionGameScr", args=(None, True))'

# Specify PORT if device not auto-detected
make PORT=/dev/tty.usbserial-210 dev_exec CMD='load_app("badge.hw_test", "HwTestScr")'
```

**Workflow for debugging:**
1. Copy problematic file from `/frozen_firmware` to `/firmware` for live development
2. Add debug print statements
3. Test with `make dev_exec CMD='load_app(...)'`
4. Watch console output for debug messages
5. Make changes, test again immediately (no rebuild needed)
6. Once fixed, copy back to `/frozen_firmware`

### Alternative: Manual REPL Testing

Before Making Changes, test import success:

```bash
python ./micropython/tools/mpremote/mpremote.py baud 460800 u0 mount -l ./firmware exec "import badge.main"
```

Or use the Makefile target for REPL:

```bash
make repl_with_firmware_dir
# Then in REPL:
import badge.main
# Or use load_app:
load_app("badge.hw_test", "HwTestScr", kwargs={"force_run": True})
```

### Expected Success Output

```
->Badge<-
Bind btn_a, pin Pin(13)
# ... button bindings ...
MAC: mac=b'...'
BootScr: nick=...
# Badge should start successfully
```

## Architecture Gotchas

### Circular Dependencies

- `gui.core.colors` imports `hardware_setup`
- `hardware_setup` calls `init_display()` which imports from `gui.core.ugui`
- `gui.core.ugui` imports from `gui.core.colors`
- **Solution**: Import `hardware_setup` first to break the cycle

### Development vs Production Code Paths

- `/firmware`: Uses mpremote mounting, different module resolution
- `/frozen_firmware`: Compiled into firmware, different import behavior
- **Testing**: Always test in development environment first

## Important Notes

- Always specify which directory (`/firmware` vs `/frozen_firmware`) when discussing code
- Remember MicroPython syntax differences from standard Python
- Consider memory and performance implications
- Test on actual hardware using mpremote mount before moving to frozen firmware
- Badge hardware has specific GPIO mappings and capabilities

## When Uncertain

- Ask for specific error messages and full tracebacks
- Request project state (git status, recent changes)
- Suggest incremental testing steps with `make dev_exec`
- Recommend git bisect for regressions

---

## Quick Reference Commands

**Testing and Debugging (PREFERRED):**
```bash
# Load and test a screen/game
make dev_exec CMD='load_app("badge.hw_test", "HwTestScr", kwargs={"force_run": True})'

# With specific port
make PORT=/dev/tty.usbserial-210 dev_exec CMD='load_app("module.path", "ClassName")'
```

**REPL Access:**
```bash
make repl_with_firmware_dir
```

Or using mpremote directly (inside Dev Container):

```bash
python ./micropython/tools/mpremote/mpremote.py baud 460800 u0 mount -l ./firmware
```

---
> Source: [disobeyfi/disobey-badge-2025-game-firmware](https://github.com/disobeyfi/disobey-badge-2025-game-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-29 -->
