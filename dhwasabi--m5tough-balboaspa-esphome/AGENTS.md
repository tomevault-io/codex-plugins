<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# M5Tough Balboa Spa Monitor - Copilot Instructions

This is an ESP32 Arduino project for the M5Tough device that monitors Balboa spa systems via RS485 communication.

## Project Context
- **Hardware**: M5Tough ESP32 device with RS485 Unit on Grove Port A (TX=32, RX=33)
- **Protocol**: Balboa spa communication protocol implementation
- **Display**: TFT touch screen showing spa status (temperature, jets, lights, etc.)
- **Framework**: Arduino framework with PlatformIO build system

## Code Standards
- Use Arduino-style C++ coding conventions
- Keep hardware-specific code in separate modules (BalboaProtocol, Display)
- Use clear, descriptive variable names for spa states
- Include comprehensive error handling for RS485 communication
- Follow M5Stack library conventions for display and touch handling

## Architecture
- **Main file**: M5Tough-BalboaSpa.ino (Arduino setup/loop structure)
- **Protocol Layer**: BalboaProtocol.h/.cpp (RS485 communication and message parsing)
- **Display Layer**: Display.h/.cpp (M5Tough screen management and UI)
- **Data Structures**: SpaStatus struct for all monitored parameters

## Key Implementation Details
- RS485 communication uses HardwareSerial(2) with specific TX/RX pins
- Message parsing follows ESPHome Balboa protocol implementation
- Temperature conversion handles both Celsius and Fahrenheit scales
- Display updates are optimized to minimize flicker and improve performance
- Touch screen handling is prepared for future control functionality

## Dependencies
- M5Tough library for hardware abstraction
- Built-in ESP32 libraries for serial communication
- No external network dependencies (pure local monitoring)

## When suggesting code changes:
1. Maintain the monitoring-only focus (no control commands yet)
2. Preserve the modular architecture between protocol, display, and main code
3. Consider performance implications for real-time display updates
4. Include appropriate error handling for spa communication timeouts
5. Follow the existing color scheme and layout conventions for the display

---
> Source: [dhWasabi/M5Tough-BalboaSpa-esphome](https://github.com/dhWasabi/M5Tough-BalboaSpa-esphome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
