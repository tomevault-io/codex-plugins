# Viessmann Decoder Home Assistant Add-on - AI Coding Assistant Guide

## Architecture Overview

This is a **Home Assistant Add-on** that decodes Viessmann heating system protocols (VBUS, KW-Bus, P300, KM-Bus) and exposes data via a web interface. The project bridges C++ hardware protocol libraries with a Linux web server for Home Assistant integration.

**Key Components:**

- **Core Protocol Library** ([viessmann-decoder/src/vbusdecoder.{cpp,h}](viessmann-decoder/src/vbusdecoder.h)) - Multi-protocol decoder supporting 4 Viessmann/RESOL protocols
- **Linux Abstraction Layer** ([viessmann-decoder/linux/src/](viessmann-decoder/linux/src/)) - Arduino API compatibility for Linux (LinuxSerial, Arduino timing functions)
- **Web Server** ([viessmann-decoder/webserver/main.cpp](viessmann-decoder/webserver/main.cpp)) - C++ libmicrohttpd-based HTTP server with JSON API
- **Add-on Configuration** ([viessmann-decoder/config.yaml](viessmann-decoder/config.yaml), [viessmann-decoder/build.json](viessmann-decoder/build.json)) - Home Assistant Supervisor integration

**Data Flow:** Serial Device → LinuxSerial → VBUSDecoder → HTTP Server → Home Assistant UI

## Critical Development Workflows

### Building & Testing Add-on

**Development container is pre-configured** - use VS Code tasks or these commands:

```bash
# Build addon (creates Docker image)
ha addons build local_viessmann_decoder

# Install and start
ha addons install local_viessmann_decoder
ha addons start local_viessmann_decoder

# View logs during testing
ha addons logs local_viessmann_decoder

# Quick restart after changes
ha addons restart local_viessmann_decoder
```

Access the addon web UI at `http://localhost:7123` (dev container port forwarding).

### Version Release Process

**CRITICAL:** Version must be synchronized across 3 files:

1. [viessmann-decoder/config.yaml](viessmann-decoder/config.yaml) - `version: X.Y.Z`
2. [viessmann-decoder/build.json](viessmann-decoder/build.json) - `io.hass.version: X.Y.Z`
3. [viessmann-decoder/Dockerfile](viessmann-decoder/Dockerfile) - `io.hass.version="X.Y.Z"`

Follow [RELEASE_CHECKLIST.md](RELEASE_CHECKLIST.md) for complete release steps.

## Project-Specific Patterns

### Multi-Architecture Docker Builds

The add-on **must support 5 architectures** (amd64, aarch64, armhf, armv7, i386). All builds use Alpine Linux 3.19 base. The Dockerfile includes:

- Single-layer C++ compilation with optimization flags (`-O2 -std=c++17`)
- Static binary stripping for size reduction
- Build dependency cleanup to minimize image size

### Protocol Enum Pattern

Protocol selection uses `enum ProtocolType` (VBUS=0, KW=1, P300=2, KM=3). Always pass protocol type to `VBUSDecoder` constructor:

```cpp
VBUSDecoder vbus(PROTOCOL_VBUS); // or PROTOCOL_KW, PROTOCOL_P300, PROTOCOL_KM
```

### Serial Configuration

**Two-part serial setup:**

1. **Baud Rate** - Protocol-dependent (VBUS=9600, KW/P300=4800)
2. **Serial Config** - `SERIAL_8N1` or `SERIAL_8E2` (affects parity/stop bits)

Both configured in [config.yaml](viessmann-decoder/config.yaml) schema validation.

### Linux Serial Compatibility Layer

The library was originally Arduino-based. [linux/src/Arduino.cpp](viessmann-decoder/linux/src/Arduino.cpp) and [linux/src/LinuxSerial.cpp](viessmann-decoder/linux/src/LinuxSerial.cpp) provide Arduino API compatibility:

- `millis()` → Linux `gettimeofday()`
- `Serial.begin()` → termios serial port configuration
- `delay()` → `usleep()`

**When modifying serial code**, ensure compatibility with both Arduino and Linux implementations.

### Bus Participant Auto-Discovery

The `VBUSDecoder` class automatically discovers devices on the bus and stores them in `BusParticipant` structures. Discovery happens in `handleIncomingData()` by analyzing packet source addresses.

## Documentation Structure

- **User Documentation** - [README.md](README.md) (main), [viessmann-decoder/INSTALL.md](viessmann-decoder/INSTALL.md), [doc/](doc/) folder
- **Developer Documentation** - [viessmann-decoder/DEVELOPMENT.md](viessmann-decoder/DEVELOPMENT.md), [RELEASE_CHECKLIST.md](RELEASE_CHECKLIST.md)
- **Protocol Specifics** - [doc/VITOTRONIC_200_KW1.md](doc/VITOTRONIC_200_KW1.md), [doc/BUS_PARTICIPANT_DISCOVERY.md](doc/BUS_PARTICIPANT_DISCOVERY.md)

**When adding features**, update relevant documentation in both German (primary) and English (code comments).

## Integration Points

### Home Assistant Supervisor API

The add-on uses Home Assistant Supervisor's:

- **Ingress** - Web UI embedded in HA interface (port 8099)
- **Watchdog** - Health check endpoint `/health` monitors addon status
- **Configuration** - Options exposed in HA UI via `config.yaml` schema
- **Privileged Access** - Requires `full_access: true` and UART permissions for serial device access

### External Dependencies

- **libmicrohttpd** - C HTTP server library (web interface)
- **Alpine packages** - curl, bash, jq, sysfsutils-dev (runtime utilities)
- **Optional: socat** - For USB/IP remote USB device forwarding (experimental)

## Common Gotchas

- **Add-on directory is self-contained** - All source files must be in `viessmann-decoder/` because Home Assistant Supervisor cannot access parent directory context during builds
- **No s6-overlay** - Unlike typical HA add-ons, this uses direct CMD execution (`ENV S6_CMD_WAIT_FOR_SERVICES=0`)
- **Binary executable** - The web server is a compiled C++ binary at `/usr/local/bin/viessmann_webserver`, not a script
- **German primary language** - User-facing docs are German; technical docs/code comments are English

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrTir1995)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MrTir1995)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
