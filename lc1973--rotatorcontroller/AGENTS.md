# PlatformIO ESP32 Rotator Controller - AI Coding Instructions

## Project Overview
This is a **PlatformIO-based ESP32-S3 rotator controller** for remote antenna rotator management. It provides WiFi web interface, scheduled power management, and ping-based device monitoring with automatic reset capability.

## Architecture & Key Components

### Hardware Abstraction
- **Rotator control**: 6 relays on pins `{36,37,38,39,34,35}` control rotator motors and associated equipment (USB ports, LTE router, network devices, power supplies)
- **Active-high relays**: `RELAY_ON = HIGH`, `RELAY_OFF = LOW` (opposite of typical ESP32 examples)

### Core Systems Integration
1. **Web Server**: Serves HTML/CSS/JS from SPIFFS with live relay status updates via `/api/status` JSON endpoint
2. **Configuration**: JSON-based config in SPIFFS (`/config.json`) with web upload/download capability
3. **Logging**: Dual logging to serial + SPIFFS file with timestamp correlation and BLE UART output

### Development Workflow

**Building & Uploading:**
```powershell
# Serial upload (development)
pio run -t upload

# Monitor serial output
pio device monitor

# OTA upload (field deployment)
# Uncomment OTA lines in platformio.ini, set target IP
```

**Key Build Configurations:**
- Target: `esp32s3dev` (ESP32-S3 DevKit)
- SPIFFS filesystem for web assets and config storage
- Upload speed: 921600 baud
- Serial port typically `COM6` (update in platformio.ini)

## Project-Specific Patterns

### State Management
- **Rotator/relay states** persist in `relayStates[6]` array and sync to GPIO immediately
- **Configuration changes** require `saveConfig()` + `loadConfig()` pattern
- **Schedule-based sleep**: Uses `shouldBeOnBySchedule()` to determine deep sleep timing

### Error Handling & Recovery
- **Ping monitoring**: Configurable per-relay with auto-reset on failure (ping failure = reset cycle)
- **WiFi resilience**: Auto-reboot on connection loss, AP fallback mode for setup
- **LoRa packet validation**: APRS message parsing with callsign filtering to prevent self-loops

### HTML Generation Pattern
```cpp
// Use modular HTML parts from SPIFFS
String html = loadHTMLPart("/header.html");
html += "<!-- page content -->";
html += loadHTMLPart("/footer.html");
// Add dynamic data
html += "<script>window.BUILD_DATE='" + String(buildDate) + "';</script>";
```

## Critical Integration Points

### GPIO Timing Requirements
- **Rotator relay initialization**: Must set `pinMode()` then `digitalWrite()` immediately - any delay causes relay chatter

### SPIFFS File Dependencies
- Web assets: `/header.html`, `/footer.html`, `/style.css`, `/script.js`, `/logo.png`
- Config: `/config.json` (JSON with WiFi, relay labels, schedule, callsign)
- Logs: `/log.txt` (auto-clearing at 2 AM daily)

### Memory Management
- **JSON documents**: Use `JsonDocument` for config (not `DynamicJsonDocument`)
- **String handling**: Extensive use of Arduino String class - watch for fragmentation in long-running operations
- **Log rotation**: Automatically truncates in-memory log at 5000 chars

## Common Debugging Approaches

- **Serial monitoring**: All operations log with millisecond timestamps
- **BLE UART**: Real-time log streaming to mobile devices
- **Web log viewer**: `/log` endpoint with auto-refresh and download capability

## Testing Scenarios
- Power cycle rotator relay sequences while monitoring ping targets
- WiFi disconnect/reconnect behavior in AP fallback mode
- Schedule-based sleep/wake cycles with timer wake
- Config upload/download round-trip validation

---
> Source: [LC1973/RotatorController](https://github.com/LC1973/RotatorController) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
