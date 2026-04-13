# MicroMouse AI Coding Assistant Instructions

## Project Overview
This is an **ATmega328P (Arduino Nano) embedded C project** for a maze-solving MicroMouse robot. The robot uses:
- **Left-hand rule algorithm** for maze navigation
- **Dual feedback control**: Encoder-based positioning + ultrasonic wall-following (PID)
- **TB6612FNG motor driver** with dual DC motors
- **3 HC-SR04 ultrasonic sensors** (left, front, right)
- **Quadrature encoders** for precise turning and drift correction

## Architecture & Key Components

### Hardware Abstraction Layers
- **[src/motors.c](../src/motors.c)**: TB6612FNG driver control, PWM setup (Timer0/Timer2), motor direction logic
- **[src/sensors.c](../src/sensors.c)**: Ultrasonic pulse timing, distance calculation (echo duration → cm)
- **[src/uart.c](../src/uart.c)**: Serial debug output (115200 baud), float/int printing utilities
- **[src/utils.c](../src/utils.c)**: Timing primitives (`millis()`, `micros()`, `delay_ms()`)

### Main Control Loop ([src/main.c](../src/main.c))
- **Encoder ISR** (PCINT1_vect): Quadrature decoding on Port C pins (A0-A3)
- **PID wall-following**: Maintains `TARGET_WALL_DISTANCE` from walls (8cm default)
- **Fusion control**: Combines encoder correction (40%) + wall PID (60%) via `ENCODER_WEIGHT`/`WALL_WEIGHT`
- **Maze decision logic**: Priority = LEFT > FORWARD > RIGHT > TURN_AROUND (180°)
- **Opening detection**: 500ms delay (`MAZE_OPENING_DELAY`) before turning at gaps to avoid false triggers

### Configuration System ([include/config.h](../include/config.h))
**All tunable parameters live here** - never hardcode values in `.c` files. Critical defines:
- **Motor speeds**: `MOTOR_SPEED` (110), `TURN_SPEED` (80)
- **Turn calibration**: `TICKS_PER_REVOLUTION` (356), `TURN_90_COMPENSATION` (30.0f), `TURN_180_COMPENSATION` (85.0f)
- **PID gains**: `PID_KP` (1.2), `PID_KI` (0.03), `PID_KD` (0.6)
- **Maze thresholds**: `MAZE_LEFT_THRESHOLD` (20cm) for opening detection
- **Control timing**: `CONTROL_PERIOD_MS` (50), `SENSOR_UPDATE_MS` (100)

## Critical Build Workflow

### Commands (Windows shell via Makefile)
```bash
make build    # Compile all src/*.c → build/*.o → micromouse.hex
make flash    # Upload via avrdude to PORT=COM3 (change in Makefile)
make clean    # Remove build artifacts
make size     # Show Flash/RAM usage (target: <32KB Flash, <2KB RAM)
```

**PORT CONFIGURATION**: Before flashing, update `PORT = COM3` in [Makefile](../Makefile) to your actual COM port.

### Compilation Constraints
- **ATmega328P limits**: 32KB Flash, 2KB RAM (current: ~37% Flash, ~90% RAM)
- **Always check size**: Run `make size` after changes - RAM near capacity
- **Optimize for size**: `-Os` flag essential, avoid large buffers/arrays
- **No dynamic allocation**: No `malloc()` - use stack or global static variables

## Code Conventions

### Pin Access Pattern
```c
// Read: Use PIN register
uint8_t state = LEFT_ECHO_PINREG & (1 << LEFT_ECHO_PIN);

// Write: Use PORT register
LEFT_TRIG_PORT |= (1 << LEFT_TRIG_PIN);   // Set HIGH
LEFT_TRIG_PORT &= ~(1 << LEFT_TRIG_PIN);  // Set LOW
```

### Motor Direction Logic (TB6612FNG)
- **Forward**: `IN1=HIGH, IN2=LOW, PWM=speed`
- **Backward**: `IN1=LOW, IN2=HIGH, PWM=speed`
- **Brake**: `IN1=HIGH, IN2=HIGH, PWM=255`
- **Stop**: `IN1=LOW, IN2=LOW, PWM=0`

### Encoder Polarity
**Left encoder reversed** in ISR (lines 142-154 in main.c) - left motor counts backward during forward motion. Right encoder normal. When modifying encoder logic, preserve this inversion.

### Turn Calculations
Auto-calculated from physical params:
```c
// 90° ticks = (π × wheelbase / 4) / (π × wheel_diameter) × ticks_per_rev + compensation
TICKS_90_DEGREE = calculated_value + TURN_90_COMPENSATION
```
**Never edit calculated macros** - adjust `TURN_90_COMPENSATION` and `TURN_180_COMPENSATION` instead.

## Calibration & Tuning

### Common Tuning Tasks
1. **Turns overshooting/undershooting**: 
   - Adjust `TURN_90_COMPENSATION` or `TURN_180_COMPENSATION` in [config.h](../include/config.h)
   - See [TURN_CONFIG_SIMPLIFIED.md](../TURN_CONFIG_SIMPLIFIED.md) for test procedures

2. **Wall following oscillation**:
   - Reduce `PID_KP` or increase `PID_KD`
   - Lower `WALL_MAX_CORRECTION` to limit aggressive steering

3. **Straight-line drift**:
   - Increase `ENCODER_CORRECTION_GAIN` (0.0-1.0 range)
   - Verify encoder wiring/polarity if drift is severe

4. **False opening detection**:
   - Increase `MAZE_OPENING_DELAY` (500ms default)
   - Adjust `MAZE_LEFT_THRESHOLD`/`MAZE_RIGHT_THRESHOLD`

**Comprehensive parameter guide**: See [CONFIG_PARAMETERS_GUIDE.md](../CONFIG_PARAMETERS_GUIDE.md) for detailed explanations of all 40+ tunable parameters.

## Integration Points

### Sensor Data Flow
1. `sensors_read_all()` → `sensor_data_t` struct (left/front/right distances)
2. `determine_wall_state()` → `wall_state_t` enum (BOTH_WALLS, LEFT_WALL, etc.)
3. `apply_integrated_control()` → Fuses encoder + PID corrections → motor speeds

### Control Fusion Algorithm
```c
encoder_correction = (left_ticks - right_ticks) × ENCODER_CORRECTION_GAIN;
wall_correction = PID_output;
total_correction = (encoder_correction × ENCODER_WEIGHT) + (wall_correction × WALL_WEIGHT);
// Apply to motors: left_speed -= correction, right_speed += correction
```

### Maze Decision Trigger
`check_maze_decision_needed()` uses **time-delayed detection**:
- Detects opening (sensor > threshold)
- Waits `MAZE_OPENING_DELAY` (500ms) while still sensing opening
- If opening persists, calls `maze_solve_decision()`
- Prevents reacting to transient sensor noise or narrow gaps

## Debugging Patterns

### Serial Output
Robot continuously prints status every 500ms:
```
ENC L:1234 R:1230 D:4 | LEFT L:7cm
```
- `ENC` = encoder counts (L=left, R=right, D=difference)
- State = `BOTH`, `LEFT`, `RIGHT`, `NO_WALLS`, or `FRONT`
- Distance readings for active sensors

### Turn Verification
After each turn, prints:
```
Left Turn 90° complete! Ticks: 287 (Target: 282)
```
Use this to calibrate compensation values - adjust until actual ≈ target.

### Debug Mode
Uncomment `#define DEBUG_MODE` in [config.h](../include/config.h) to enable `DEBUG_PRINT()` macros throughout code. **WARNING**: Excessive serial output can slow control loop.

## Sample Code Locations

- **90° turn implementation**: See [samples/90_turning_and_wall_following/main.c](../samples/90_turning_and_wall_following/main.c)
- **Optimized wall following**: See [samples/optimized_wall_following_with_ultrasonic/main.c](../samples/optimized_wall_following_with_ultrasonic/main.c)
- **Arduino test code**: See [arduino_codes/wall_following_tune.ino](../arduino_codes/wall_following_tune.ino)

## Important Constraints

1. **No floating-point in ISRs**: Encoder ISR is performance-critical - use only integer arithmetic
2. **Sensor timing**: Ultrasonic sensors need 50ms+ between readings on same sensor
3. **PWM pins fixed**: Motor A = D5 (OC0B), Motor B = D3 (OC2B) - hardware-locked to timers
4. **No blocking delays in main loop**: Use `millis()` checks, not `delay_ms()`, except in turn/decision functions
5. **UART buffer limits**: `uart_println()` has small buffer - keep messages concise

## When Modifying Code

- **Adding parameters**: Put in [config.h](../include/config.h) with detailed comment explaining range/effect
- **Changing control logic**: Test `apply_integrated_control()` impact on both encoder and wall modes
- **Adjusting thresholds**: Document reasoning in comments - future tuning depends on understanding *why*
- **Memory-intensive changes**: Always run `make size` and verify RAM usage stays below 2000 bytes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ravinx001)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Ravinx001)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
