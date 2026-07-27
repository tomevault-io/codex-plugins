# OpenSplitDeck Controller Firmware - AI Coding Agent Instructions

## Project Overview
This is firmware for a **split wireless game controller** running on Seeed Xiao BLE nRF52840 boards using the **Zephyr RTOS** and **Nordic ESB (Enhanced ShockBurst)** wireless protocol. Each controller half (LEFT/RIGHT) is a separate nRF52840 board that communicates with a central dongle.

## Critical Architecture Concepts

### Controller Identity System
- **CONTROLLER_ID macro** in [main.c](src/main.c#L16) determines LEFT (1) or RIGHT (0) controller
- Controllers have distinct radio pipelines and respond only to their ID
- The `flags` byte in `esb_controller_data_t` uses bit 7 to indicate LEFT (0x80) or RIGHT (0x00)
- **Always respect this ID when modifying ESB communication or initialization code**

### Driver-Based Architecture
This firmware uses a **modular driver pattern** - each peripheral has a dedicated driver with init/update/get_data pattern:

| Driver | Header | Purpose | Key Functions |
|--------|--------|---------|---------------|
| `esb_comm_driver` | [esb_comm_driver.h](src/esb_comm_driver.h) | Wireless communication via Nordic ESB | `esb_comm_send_data()`, adaptive timing with ACK payloads |
| `analog_driver` | [analog_driver.h](src/analog_driver.h) | Thread-based ADC reads for sticks/triggers | `analog_driver_get_controller_data()`, background sampling |
| `button_driver` | [button_driver.h](src/button_driver.h) | Debounced GPIO button handling | `button_driver_read_all()`, edge detection |
| `imu_driver` | [imu_driver.h](src/imu_driver.h) | LSM6DS3 accelerometer/gyro | `imu_get_controller_data()`, complementary filter |
| `haptic_driver` | [haptic_driver.h](src/haptic_driver.h) | DRV2605 LRA motor control | `haptic_setup_external_trigger()`, GPIO trigger mode |
| `display` | [display.h](src/display.h) | SSD1306 OLED (128x32) | Thread-safe screen management |
| `controller_storage` | [controller_storage.h](src/controller_storage.h) | NVS-based persistent config | Calibration, bindings, preferences |
| `power_mgmt_driver` | [power_mgmt_driver.h](src/power_mgmt_driver.h) | Sleep/wake, button combos | Peripheral shutdown orchestration |

**Pattern**: When adding features, use the existing driver interfaces rather than direct hardware access. Drivers handle thread safety and error recovery.

### Multi-Threading Architecture
The firmware uses **three concurrent threads** coordinated via shared memory with atomic updates:

```c
Main Thread (priority 0):     ESB transmission @ ~125Hz, button reads, sleep combo detection
Trackpad Thread (priority 5): IQS7211E polling @ 10ms, haptic pulse management  
Display Thread (priority 6):  Screen rendering @ 30Hz, lockless screen switching
```

**Critical**: `trackpad_thread_heartbeat` and `display_thread_heartbeat` are monitored in main loop - frozen threads trigger warnings. Never block these threads with long sleeps.

### ESB Adaptive Timing System
The ESB driver implements **dongle-controlled transmission timing** via ACK payloads:

1. Controller sends data at adaptive rate (default 8ms)
2. Dongle responds with `ack_timing_data_t` containing `next_delay_ms`
3. Controller adjusts sleep delay based on dongle requirements
4. **Never hardcode sleep delays in main loop** - use `esb_comm_get_next_delay()`

See [main.c lines 2368-2385](src/main.c#L2368-L2385) for timing implementation. The system includes radio busy backoff and latency spike detection.

## Build System (Zephyr + West)

### Building the Firmware
```bash
# Build for specific board (from controller/ directory)
west build -b seeed_xiao_ble_nrf52840_sense

# Flash via USB bootloader (double-tap reset button)
west flash

# Clean build
west build -t clean
```

### Key Build Files
- **CMakeLists.txt**: Source file list (comment out files to disable modules)
- **prj.conf**: Kconfig options - logging levels, peripheral enables, ESB config
- **xiao_ble_nrf52840_sense.overlay**: Device tree hardware config (I2C pins, ADC channels, display)
- **boards/seeed_xiao_ble.overlay**: Board-specific variant

**Important**: When modifying prj.conf, changes take effect on next build. Common issues:
- `CONFIG_I2C_NRFX_TRANSFER_TIMEOUT=100` prevents I2C freezes
- `CONFIG_NFCT_PINS_AS_GPIOS=y` required for DRV2605 (uses NFC pins)
- Logging disabled in production (`CONFIG_LOG=n`) for performance

## Calibration & Storage System

### Interactive Calibration Mode
Hold **STICK_CLICK + PAD_CLICK for 3 seconds** to enter calibration:
- Phase 0 (2s): Center stick, hold trigger released
- Phase 1 (8s): Move stick in circles, pull trigger fully

Code: [main.c lines 2161-2280](src/main.c#L2161-L2280). Uses `analog_driver_begin_calibration_collection()` → `analog_driver_update_calibration_data()` → `analog_driver_finalize_calibration()`.

**Calibration data persists in NVS flash** via `controller_storage_save_calibration()`. Always load on boot with `controller_storage_load_calibration()`.

### Storage Architecture
Uses Zephyr Settings subsystem (NVS backend) with three namespaces:
- `controller/calibration`: Stick centers, min/max, deadzone, trigger range
- `bindings/bindings`: Button mappings (currently default only)
- `preferences/preferences`: Display brightness, controller name, sleep timeout

**Pattern**: Modify storage structure requires updating both header struct AND default initializers in [controller_storage.c](src/controller_storage.c#L200-L250).

## Hardware-Specific Patterns

### Battery Voltage Reading (XIAO Specific)
P0.14 must be configured as **current sink** to enable voltage divider:
```c
gpio_pin_configure_dt(&vbat_enable, GPIO_OUTPUT_INACTIVE | GPIO_OPEN_DRAIN);
gpio_pin_set_dt(&vbat_enable, 0); // Active LOW for sink
```
See [main.c lines 1730-1792](src/main.c#L1730-L1792). This is a XIAO board quirk.

### Haptic Motor (DRV2605 LRA Mode)
**Must perform LRA auto-calibration before external trigger setup:**
```c
haptic_perform_lra_calibration();  // One-time auto-tune
haptic_setup_external_trigger(DRV2605_EFFECT_SHARP_TICK_2);
```
GPIO trigger mode allows ultra-low latency haptic response (<1ms) for trackpad clicks.

### Display Thread Safety
Display uses **lockless double-buffering** pattern:
```c
display_set_screen(DISPLAY_SCREEN_ANALOG, &display_data);  // Thread-safe
```
Never call display API directly - always use `display_set_screen()` which handles atomic screen transitions.

## Debugging & Common Issues

### Freeze Detection
Monitor these indicators if system appears frozen:
- Main loop heartbeat: Logged every 30s with iteration count
- Thread heartbeats: `trackpad_thread_heartbeat`, `display_thread_heartbeat`
- ESB stats: `total_transmissions` counter should increment (watchdog at lines 2255-2290)

**ESB Watchdog**: If stats freeze for 15s despite "successful" sends, driver auto-resets ESB stack.

### I2C Timeouts
`CONFIG_I2C_NRFX_TRANSFER_TIMEOUT=100` prevents indefinite hangs on unresponsive devices. If device isn't present (e.g., trackpad), driver gracefully degrades.

### Sleep/Wake Issues
**Sleep combo**: Hold START + BUMPER for 5s (triple vibrate confirms)
Sleep entry calls `shutdown_all_peripherals()` which:
1. Suspends trackpad/display threads
2. Powers down DRV2605, IMU, display via I2C commands
3. Stops ESB transmission (`esb_comm_enter_sleep()`)
4. Enters System OFF mode (deep sleep)

Wake on BUMPER button press restores full system via `wakeup_all_peripherals()`.

## Code Conventions

### Error Handling Pattern
Drivers return status enums (`_STATUS_OK`, `_STATUS_ERROR`, etc.). **Always check return values**:
```c
analog_status_t status = analog_driver_get_controller_data(&analog_data);
if (status == ANALOG_STATUS_OK) {
    // Use data
}
```

### Logging Discipline
- `LOG_ERR`: Critical failures that affect functionality
- `LOG_WRN`: Degraded operation (missing peripheral, retry attempts)
- `LOG_INF`: Major state changes (init complete, entering sleep, calibration)
- `LOG_DBG`: Verbose debug (disabled in production builds)

**In main loop**: Avoid excessive logging (>1Hz) - use counters with periodic summaries.

### GPIO Button Pattern
All buttons are **active-low with internal pull-ups**:
```c
static const struct gpio_dt_spec button = {
    .port = DEVICE_DT_GET(DT_NODELABEL(gpio1)),
    .pin = 10,
    .dt_flags = GPIO_ACTIVE_LOW
};
```
Button driver handles debouncing - use `button_driver_read_all()` not direct GPIO reads.

## Testing Workflows

### Analog Debug Mode
Hold MODE button to print real-time ADC values and display on OLED:
```
Stick X raw: -2048 to 2047
Stick Y raw: -2048 to 2047  
Trigger raw: 0 to 4095
```
Useful for diagnosing stick drift or calibration issues.

### ESB Link Quality
Check transmission stats every 5s (main loop automatic):
```
ESB Stats: total: 625, success: 623, failed: 2, rate: 99.68%
```
Success rate <95% indicates RF interference or range issues.

## Important Constants

| Symbol | Value | Location | Purpose |
|--------|-------|----------|---------|
| `CONTROLLER_ID` | 0 or 1 | main.c:16 | **Must match physical controller** |
| `ESB_MAX_PAYLOAD_LENGTH` | 32 | prj.conf | Controller data packet size |
| `TRACKPAD_THREAD_PRIORITY` | 5 | main.c:2052 | Higher = more urgent |
| `CONFIG_I2C_NRFX_TRANSFER_TIMEOUT` | 100 | prj.conf | I2C timeout in ms |

## When Modifying Code

1. **Changing CONTROLLER_ID**: Update main.c line 16, rebuild from scratch (`west build -t clean`)
2. **Adding peripherals**: Create driver following analog_driver pattern, register in CMakeLists.txt
3. **Modifying ESB payload**: Update `esb_controller_data_t` struct, coordinate with dongle firmware
4. **Thread timing changes**: Consider impact on ESB transmission jitter (target <1ms variance)
5. **Storage changes**: Increment version or use new namespace to avoid corruption on OTA updates

## Resources
- Zephyr I2C API: https://docs.zephyrproject.org/latest/hardware/peripherals/i2c.html
- Nordic ESB: https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v15.0.0/esb_users_guide.html
- nRF52840 Datasheet: For GPIO, SAADC, and power management specifics

---
> Source: [tommybee456/OpenSplitDeck](https://github.com/tommybee456/OpenSplitDeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
