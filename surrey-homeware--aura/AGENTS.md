# Aura ESP32 Weather Widget

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

Aura is an ESP32-based weather widget that runs on ESP32-2432S028R ILI9341 devices with a 2.8" screen (sometimes called "CYD" or Cheap Yellow Display). This is a hardware-specific Arduino project that requires physical ESP32 hardware to build and test.

## CRITICAL LIMITATIONS

**HARDWARE DEPENDENCY**: This project CANNOT be built, tested, or run without the specific ESP32-2432S028R ILI9341 hardware. Do not attempt to build without the hardware.

**NO COMMAND-LINE BUILDS**: This is an Arduino IDE project. There are no Makefiles, package.json, or command-line build tools. All compilation must be done through Arduino IDE.

**NO AUTOMATED TESTING**: There are no unit tests, integration tests, or CI/CD pipelines. All validation must be done manually on the hardware.

## Working Effectively

### Prerequisites
- ESP32-2432S028R ILI9341 device with 2.8" screen (CYD/Cheap Yellow Display hardware)
- Arduino IDE installed and configured
- All required libraries installed in Arduino IDE (see Libraries section below)

### Arduino IDE Configuration
Configure Arduino IDE with these EXACT settings before compiling:
1. Board: "ESP32 Dev Module"
2. Tools -> Partition Scheme: "Huge App (3MB No OTA/1MB SPIFFS)"
3. All other settings can remain at defaults

### Required Libraries Installation
Install these EXACT library versions in Arduino IDE (Sketch -> Include Library -> Manage Libraries):
- ArduinoJson 7.4.1
- HttpClient 2.2.0  
- TFT_eSPI 2.5.43_
- WifiManager 2.0.17
- XPT2046_Touchscreen 1.4
- lvgl 9.2.2

**CRITICAL CONFIGURATION STEP**: After installing libraries, manually copy configuration files:
1. Copy `TFT_eSPI/User_Setup.h` from repository to `~/Documents/Arduino/libraries/TFT_eSPI/User_Setup.h`
2. Copy `lvgl/src/lv_conf.h` from repository to `~/Documents/Arduino/libraries/lvgl/src/lv_conf.h`

### Project Setup
1. Copy the `aura/` folder to `~/Documents/Arduino/aura/`
2. Open `~/Documents/Arduino/aura/weather.ino` in Arduino IDE
3. Verify board and partition scheme settings
4. Connect ESP32 hardware via USB

### Build Process
1. Click "Verify" in Arduino IDE to compile - takes 2-3 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
2. If successful, click "Upload" to flash to device - takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
3. Open Serial Monitor (115200 baud) to view debug output

**Expected Build Time**: 2-3 minutes for verification, 1-2 minutes for upload. NEVER CANCEL builds or uploads.

## Validation

### Hardware Testing Scenarios
After successfully uploading firmware, test these complete scenarios:

1. **Initial Setup Flow**:
   - Power on device
   - Connect to "Aura" WiFi network from phone/laptop
   - Navigate to http://192.168.4.1 in browser
   - Configure WiFi credentials
   - Verify device connects to WiFi and displays weather

2. **Weather Display**:
   - Verify current weather displays with icon and temperature
   - Touch screen to access settings
   - Verify 7-day forecast and hourly forecast views work

3. **Settings Configuration**:
   - Test location search and selection
   - Test brightness adjustment
   - Test language switching (English, Spanish, German, French)
   - Test temperature unit toggle (°C/°F)
   - Test 12/24 hour time format toggle

4. **WiFi Reset**:
   - Test WiFi reset functionality
   - Verify device returns to captive portal mode

### Code Validation
- Always run Arduino IDE's "Verify" before making changes
- Check Serial Monitor output for runtime errors
- Use `extract_unicode_chars.py` when modifying multilingual strings

## Common Tasks

### Key File Locations
```
Repository Structure:
├── aura/
│   ├── weather.ino              # Main Arduino sketch
│   ├── extract_unicode_chars.py # Unicode analysis utility
│   ├── icon_*.c                 # Weather icon assets
│   ├── image_*.c                # Weather image assets
│   └── lv_font_*.c              # LVGL font files
├── TFT_eSPI/
│   └── User_Setup.h             # Display driver configuration
└── lvgl/
    └── src/
        └── lv_conf.h            # LVGL library configuration
```

### Modifying Weather Display
- Main UI creation: `create_ui()` function in weather.ino
- Weather data fetching: `fetch_and_update_weather()` function
- Icon selection: `choose_image()` and `choose_icon()` functions

### Adding New Languages
1. Add language enum to `enum Language`
2. Create new `LocalizedStrings` structure
3. Update language switching logic in settings
4. Run `python3 aura/extract_unicode_chars.py aura/weather.ino` to get required Unicode characters
5. Regenerate LVGL fonts with new character set if needed

### Modifying Display Configuration
- Edit `TFT_eSPI/User_Setup.h` for display driver settings
- Current config: ILI9341_2_DRIVER, 240x320 resolution, specific pin mappings
- Edit `lvgl/src/lv_conf.h` for LVGL library settings
- Current config: 16-bit color depth, ESP32 optimizations

### Font Management
- Custom fonts in `aura/lv_font_*.c` files
- Use LVGL Font Converter tool to generate new fonts
- Include Unicode characters: `°¿ÉÊÍÓÜßáäçèéíñóöúûü‐→` (from extract_unicode_chars.py)

## Unicode Character Analysis
Use the included Python utility to analyze Unicode requirements:
```bash
cd /path/to/aura
python3 aura/extract_unicode_chars.py aura/weather.ino
```
This identifies all non-ASCII characters needed for LVGL font generation.

## Debugging

### Serial Monitor Output
- Set baud rate to 115200
- Look for WiFi connection status
- Monitor weather API responses
- Check for memory allocation errors

### Common Issues
- **Compilation fails**: Verify library versions and configuration files
- **Display blank**: Check TFT_eSPI configuration and pin connections
- **Touch not working**: Verify XPT2046 pin configuration
- **WiFi issues**: Check WifiManager configuration and credentials
- **Weather not updating**: Verify internet connection and API responses

### Hardware Connections
The TFT_eSPI User_Setup.h defines these pin connections:
- TFT_MISO: 12, TFT_MOSI: 13, TFT_SCLK: 14
- TFT_CS: 15, TFT_DC: 2, TFT_RST: -1 (connected to ESP32 reset)
- TOUCH_CS: 33, LCD_BACKLIGHT_PIN: 21

## Project Architecture

### Main Components
- **weather.ino**: Main application logic, UI, and weather fetching
- **LVGL**: UI framework for ESP32 with touchscreen support
- **TFT_eSPI**: Display driver for ILI9341 screens
- **WiFiManager**: Captive portal for WiFi configuration
- **ArduinoJson**: Weather API response parsing
- **XPT2046_Touchscreen**: Touch input handling

### Key Functions
- `setup()`: Initialize hardware, WiFi, display, and UI
- `loop()`: Main application loop with LVGL updates
- `create_ui()`: Build the main weather display interface
- `fetch_and_update_weather()`: Get weather data from API
- `create_settings_window()`: Settings screen with user preferences

### Weather Data Source
- Uses Open-Meteo API for weather data
- Geocoding API for location search
- No API key required
- Updates every 10 minutes (600000ms)

## Development Guidelines

### Making Changes
1. Always test on actual hardware
2. Use Serial Monitor for debugging
3. Keep builds under 3MB (partition scheme limitation)
4. Test all language variants when modifying UI text
5. Verify touchscreen functionality after UI changes

### Code Style
- Follow existing Arduino/C++ conventions
- Use descriptive function names
- Comment complex LVGL UI creation code
- Keep string literals in LocalizedStrings structures

### Memory Management
- ESP32 has limited RAM - be careful with large images/fonts
- Use PROGMEM for static data when possible
- Monitor Serial output for memory allocation failures

## Troubleshooting Build Issues

### Library Version Conflicts
- Always use exact versions specified above
- Clear Arduino library cache if needed
- Reinstall libraries if compilation fails

### Configuration File Issues
- Ensure User_Setup.h and lv_conf.h are copied correctly
- Check that #define settings match hardware requirements
- Verify no conflicting definitions between files

### Hardware-Specific Problems
- Only works with ESP32-2432S028R ILI9341 devices
- Different display modules may require User_Setup.h modifications
- Touch calibration may need adjustment for different screens

---
> Source: [Surrey-Homeware/Aura](https://github.com/Surrey-Homeware/Aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
