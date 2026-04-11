# Omnimeter - Copilot Instructions

## Project Overview
ESP32-S2 precision voltmeter firmware (Lolin S2 Mini) with 5-range auto-ranging (0-60V), web interface, and USB serial output. Production-ready embedded C++ using PlatformIO/Arduino framework.

## Architecture

### Core Components
- **Voltmeter** ([src/Voltmeter.cpp](src/Voltmeter.cpp)) - ADC driver with 5 parallel voltage dividers, 13-bit resolution, 64-sample oversampling, hysteresis-based auto-ranging
- **WebInterface** ([src/WebInterface.cpp](src/WebInterface.cpp)) - ESPAsyncWebServer, dual-mode WiFi (AP/Station), embedded HTML, JSON REST API at `/api/voltage`
- **CalibrationManager** ([src/Calibration.cpp](src/Calibration.cpp)) - NVS-persistent two-point calibration per range, serial command interface
- **main.cpp** - Non-blocking task scheduler using `millis()` timing

### Data Flow
```
ADC Pins (GPIO1-5) → Voltmeter.measure() → MeasurementResult struct
                                              ↓
                              Serial output ← main loop (250ms)
                              Web JSON API ← AsyncWebServer (on request)
```

## Build & Upload

```bash
# Build firmware
pio run

# Upload to device (hold BOOT button, press RST, release BOOT)
pio run --target upload

# Serial monitor (115200 baud, USB CDC)
pio device monitor

# Upload web files to LittleFS (if using external HTML)
pio run --target uploadfs
```

## Code Conventions

### Memory Management
- **Static allocation preferred**: Use `static` objects, avoid `new` in `setup()`
- **No String concatenation in loops**: Use `snprintf()` with pre-allocated `char[]` buffers
- **PROGMEM for constants**: Large strings like HTML use `PROGMEM` storage

### Non-Blocking Pattern
All timing uses `millis()` with interval constants - never use `delay()` in `loop()`:
```cpp
constexpr uint32_t INTERVAL_MS = 100;
if (currentTime - lastUpdate >= INTERVAL_MS) {
    lastUpdate = currentTime;
    // ... task code
}
```

### Calibration Safety
- Gain defaults to `1.0f`, offset to `0.0f` if NVS is empty/corrupt
- All float inputs validated via `parseVoltage()` - rejects NaN, Inf, multiple decimals
- Magic number (`0xCA11B001`) validates stored calibration data

### ADC Configuration
- Resolution: 13-bit (`analogReadResolution(13)`)
- Attenuation: 11dB for ~2.5V max range
- Factory eFuse calibration used when available

## Key Constants
| Constant | Location | Purpose |
|----------|----------|---------|
| `PIN_RANGE_*` | Voltmeter.h | ADC GPIO pins (1-5) |
| `MULTIPLIER_*` | Voltmeter.h | Voltage divider scaling factors |
| `AP_SSID/AP_PASSWORD` | WebInterface.h | Default WiFi AP credentials |
| `*_INTERVAL_MS` | main.cpp | Task timing intervals |

## Serial Commands
- `HELP` - List available commands
- `STATUS` - System status (uptime, heap, voltage)
- `CAL:ZERO` - Calibrate zero offset (inputs shorted)
- `CAL:REF:<voltage>` - Calibrate with reference voltage
- `CAL:SHOW` - Display calibration values
- `CAL:RESET` - Factory reset calibration

## Hardware Notes
- **USB CDC**: Native USB serial, no UART bridge (`ARDUINO_USB_CDC_ON_BOOT=1`)
- **LED**: GPIO15, active LOW (Lolin S2 Mini built-in)
- **Boot mode**: Hold BOOT → Press RST → Release BOOT to enter upload mode
- **Voltage dividers**: 100kΩ high-side resistor on all ranges except 2.5V (direct)

## Testing Approach
No automated test framework. Validate via:
1. Serial output verification (`pio device monitor`)
2. Web API response check (`curl http://192.168.4.1/api/voltage`)
3. Known reference voltage measurements for calibration verification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NortonTech3D)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/NortonTech3D)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
