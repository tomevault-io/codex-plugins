# Copilot Instructions for THZ Integration

## Project Overview

This is an unofficial Home Assistant custom integration for Stiebel Eltron LWZ / Tecalor THZ heat pumps. The integration enables monitoring and control of heat pump systems via serial protocol (USB or network ser2net).

## General Guidelines

- This is a Home Assistant custom component following the Home Assistant architecture and coding standards
- The project uses async/await patterns throughout for non-blocking operation
- All user-facing strings should support internationalization via `strings.json` and translation files
- Follow Home Assistant's entity naming conventions and device class assignments
- Use type hints for all function parameters and return values
- Target modern Home Assistant versions (2023.x or newer recommended)

## Architecture

### Key Components

- **THZDevice** (`thz_device.py`): Core device communication handler managing serial protocol
- **Config Flow** (`config_flow.py`): UI-based configuration wizard supporting USB and network connections
- **Register Maps** (`register_maps/`): Device register definitions and mappings for different firmware versions
- **Platforms**: Standard Home Assistant platforms (sensor, switch, number, select, time, calendar)

### Data Flow

1. ConfigEntry stores connection configuration
2. THZDevice handles low-level serial communication with device locking
3. DataUpdateCoordinator manages polling intervals per register block
4. Platform entities decode register data and expose it to Home Assistant

## Code Style

- Use Python 3.11+ features where appropriate
- Follow PEP 8 style guidelines
- Use double quotes for strings by default
- German comments are acceptable (legacy from FHEM module origin)
- Use `_LOGGER` for logging (imported from `logging` module)
- Log levels: debug for protocol details, info for important events, warning for issues

## Home Assistant Integration Standards

- Implement `async_setup_entry` for platform setup
- Use `CoordinatorEntity` for entities that poll data
- Register devices properly in device registry with manufacturer, model, and version info
- Use appropriate `SensorDeviceClass`, `NumberDeviceClass`, etc. for entities
- Implement `entity_description` pattern where applicable
- Handle entry unload properly in `async_unload_entry`

## Testing and Validation

- Test with Home Assistant's hassfest validation (workflow: `.github/workflows/hassfest.yaml`)
- Validate with HACS action (workflow: `.github/workflows/validate.yml`)
- Manual testing should verify USB and network connections work correctly
- Ensure entity state updates reflect actual device changes

## Dependencies

- `pyserial`: Required for serial communication
- No additional dependencies beyond Home Assistant core

## Documentation

- Update README.md for user-facing features and setup instructions
- Document compatibility for tested device models and firmware versions
- Maintain HACS compatibility via `hacs.json` and proper manifest

## License

This project is licensed under GNU General Public License v3.0. Ensure all contributions comply with GPL v3 terms.

## Credits

Based on FHEM-Module by Immi. Maintain attribution to original work in documentation.

---
> Source: [bigbadoooff/thz](https://github.com/bigbadoooff/thz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-24 -->
