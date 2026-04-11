# CanvasBT - EPaper BLE Image Transfer App

## Project Overview
Flutter mobile app that sends images to Arduino-powered 6-color e-paper displays (800x480) via Bluetooth Low Energy. The app processes images using a specific 6-color hardware palette and transfers them with OTA update capability.

## Architecture

### Core Components
- **Flutter Mobile App** (`Flutter/`) - Complete Flutter project with all UI and business logic
  - `Flutter/lib/main.dart` - Single-file monolithic app for learning purposes
  - `Flutter/lib/ble_manager.dart` - Handles Bluetooth communication with Arduino devices
  - `Flutter/lib/image_processor.dart` - Converts images to 6-color e-paper format
- **Arduino Firmware** (`Arduino/Spectra6/`) - ESP32-based receiver with deep sleep and touch wake
- **Python Tools** (`Python/`) - Desktop image conversion and BLE transfer utilities

### Hardware Color Palette
Six specific colors with exact hardware codes (critical for display compatibility):
```dart
static const List<ColorMap> hwPalette = [
  ColorMap(0x00, Color.fromRGBO(0, 0, 0, 1)),       // Black
  ColorMap(0xFF, Color.fromRGBO(255, 255, 255, 1)), // White  
  ColorMap(0xFC, Color.fromRGBO(255, 255, 0, 1)),   // Yellow
  ColorMap(0xE0, Color.fromRGBO(210, 0, 0, 1)),     // Red
  ColorMap(0x03, Color.fromRGBO(0, 0, 180, 1)),     // Blue
  ColorMap(0x1C, Color.fromRGBO(0, 150, 0, 1)),     // Green
];
```

## Development Workflow

### Running the Flutter App
```bash
cd Flutter
flutter pub get
flutter run  # Android device with BLE required
```

### Key Constants
- **Display Resolution**: 800x480 pixels (landscape e-paper)
- **BLE Chunk Size**: 480 bytes (optimized for MTU)
- **Transfer Types**: `0x10` (image), `0x20` (OTA update)
- **BLE UUIDs**: Nordic UART service (`6E400001-B5A3-F393-E0A9-E50E24DCCA9E`)

### Arduino Development
```bash
# Navigate to Arduino firmware directory
cd Arduino/Spectra6
# Upload firmware to ESP32 using Arduino IDE
# Uses ArduinoBLE library
# Supports SPIFFS for image persistence
# Deep sleep with 5-day refresh cycle
```

### Python Tools
```bash
# Navigate to Python tools directory
cd Python
# Run BLE image tool
python epd_image_tool_ble.py
# Run standalone image converter
python epd_image_tool.py
```

## Critical Integration Points

### BLE Protocol
- **Handshake**: Size header (4 bytes) → ACK → chunked data transfer
- **Acknowledgments**: `0x01` (size received), `0x02` (progress), `0x03` (complete), `0xFF` (error)
- **Battery Status**: `0xB0` + percentage byte sent at connection
- **Device Discovery**: Scans for devices with "EPD" in advertisement name

### Image Processing Pipeline
1. **Load** → **Enhance** (brightness/contrast/saturation) → **Resize** (fit/stretch modes)
2. **Quantize** to 6-color palette with optional Floyd-Steinberg dithering
3. **Pack** pixels (2 per byte) → **Optional 180° rotation** → **BLE transfer**

### State Management Patterns
- Uses Flutter's `setState()` pattern throughout main file
- BLE connection state tracked with periodic timers
- Image library stored in memory with JSON persistence to local storage
- Pan/zoom/rotate gestures for precise image framing

## Project-Specific Conventions

### File Organization
- **Three-tier structure**: `Arduino/`, `Flutter/`, `Python/` for clean separation
- **Monolithic Flutter**: All UI logic in `Flutter/lib/main.dart` for educational clarity
- **Empty directories**: `Flutter/lib/src/` and `Flutter/lib/widgets/` exist but unused
- **Asset structure**: `Flutter/FramePics/`, `Flutter/OTAFile/` for bundled resources

### Error Handling
- BLE errors show user-friendly dialogs with retry options
- Image processing failures display toast messages
- Arduino disconnection triggers automatic reconnection attempts

### Performance Optimizations
- **High-speed BLE**: 512-byte chunks with 200-chunk acknowledgment threshold
- **Image caching**: Processed images stored in memory library
- **Background processing**: Image conversion runs on separate isolates

### Debugging
- Extensive console logging in both Flutter and Arduino code
- Real-time transfer speed display during BLE operations
- Battery percentage monitoring from Arduino device

When modifying this codebase, maintain the educational single-file structure in `main.dart` and ensure BLE protocol compatibility between Flutter and Arduino components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abubakar1075)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/abubakar1075)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
