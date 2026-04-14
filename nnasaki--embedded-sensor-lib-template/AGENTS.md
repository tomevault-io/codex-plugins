# Copilot Instructions — EmbeddedSensorLib

## Build & Test

```bash
# Configure and build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Run all tests
cd build && ctest --output-on-failure

# Run a single test binary
./build/test_temperature
./build/test_accelerometer
./build/test_pressure

# Run a specific test case
./build/test_temperature --gtest_filter='TemperatureSensorTest.ReadTemperature_Success'

# Build without tests
cmake -B build -DBUILD_TESTS=OFF
```

## Architecture

This is a C++17 embedded sensor driver library with hardware abstraction via pure-virtual bus interfaces.

**Layering** (top → bottom):
1. **Concrete drivers** (`sensor::drivers::`) — each driver owns a reference to its bus interface, reads calibration on `init()`, and exposes sensor-specific methods
2. **`ISensor`** — abstract base with common lifecycle: `init()`, `reset()`, `isConnected()`, `getDeviceId()`, `getName()`
3. **HAL interfaces** — `II2C` and `ISPI` are pure-virtual; platform code implements them, tests use GoogleMock

**Driver → bus mapping:**
- `TemperatureSensor` (BME280) → `II2C`
- `PressureSensor` (BMP280) → `II2C`
- `Accelerometer` (ADXL345) → `ISPI`

All sensor operations return `sensor::Status` — check every return value; never assume `Ok`.

## Conventions

- **Namespace structure**: interfaces and `Status` live in `sensor::`; driver classes live in `sensor::drivers::`
- **Header layout**: public constants and register maps as `static constexpr`, then enums/structs, then public API, then private helpers and state
- **Drivers are non-copyable, movable** (inherited from `ISensor`)
- **Bus interfaces passed by reference** — the caller owns the bus object's lifetime
- **Calibration pattern**: drivers read factory calibration data during `init()` and store it in a `CalibrationData` struct exposed via a `calibration()` accessor
- **Compensation formulas** use integer arithmetic matching the sensor datasheets (e.g., BME280 integer compensation), then convert to float at the end
- **Compiler warnings**: the project enables strict warnings (`-Wall -Wextra -Wpedantic -Wshadow -Wconversion -Wsign-conversion`); all code must compile clean

## Testing

Tests use GoogleTest + GoogleMock (v1.14, fetched via CMake `FetchContent`).

**Test pattern**: each driver has a test file with:
1. A `Mock*` class for the bus interface (e.g., `MockI2C`, `MockSPI`)
2. A `::testing::Test` fixture holding the mock and driver instance
3. Helper methods in the fixture to set up common expectations (e.g., `expectChipIdRead`, `expectCalibration`, `initSuccessfully`)

When adding a new driver, follow this same pattern: mock the bus, create a fixture, write helpers for init expectations.

## Driver Specifications

`docs/specs/` contains Markdown driver specifications extracted from datasheets. These are the **source of truth** for register maps, communication protocols, compensation algorithms, and power modes. When implementing or modifying a driver, always reference the corresponding spec file:

- `docs/specs/bme280_driver_spec.md` → TemperatureSensor
- `docs/specs/bmp280_driver_spec.md` → PressureSensor
- `docs/specs/adxl345_driver_spec.md` → Accelerometer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nnasaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
