# AI Coding Agent Instructions for GPS-Project-Jetson

## Project Overview

**GPS-Project-Jetson** is an autonomous vehicle navigation system combining:
- **Jetson Nano** (Linux, Python) running GPS-based waypoint navigation
- **Arduino Nano** (C++) controlling motor/steering via binary serial protocol
- **Ublox ZED-F9R** GPS/IMU module communicating over `/dev/ttyTHS1` at 38400 baud

The system supports both **RC manual override** and **autonomous Jetson control** with priority switching logic.

---

## Architecture & Data Flow

### Multi-Component System
1. **Jetson (`jetson_control.py`)**: Maintains GPS state in thread-safe dictionary, calculates bearing/distance, commands Arduino
2. **Arduino (`RCandJetsoncontrol/RCandJetsoncontrol.ino`)**: Arbitrates between RC receiver and Jetson serial packets, controls motor/servo
3. **GPS Module (`ublox_gps/`)**: Modified SparkFun library (port: `/dev/ttyTHS1`, baud: 38400)
4. **Data Recording (`Record_data.py`)**: Records GPS tracks to CSV/GPX in `results/` directory

### Serial Communication
- **Jetson ↔ Arduino**: 115200 baud on `/dev/ttyUSB0`, binary protocol (6 bytes)
- **Jetson ↔ GPS**: 38400 baud on `/dev/ttyTHS1`, Ublox UBX protocol (hardware-dependent; may change if GPS module is swapped)
- **Arduino ↔ RC Receiver**: PWM input on pins 2/3, millisecond pulse width parsing

### Control Mode Arbitration
- **RC Active**: Reads PWM pulse from RC receiver; overrides Jetson until neutral for 2 seconds
- **Jetson Active**: Accepts binary packets when RC neutral; falls back to RC if signal lost
- **Failsafe**: Motor stops if no valid command for 500ms

---

## Key Code Patterns & Conventions

### Serial Protocol (Jetson → Arduino)
Binary packet format: `[0xAA, throttle, steering, reserved, checksum, 0x55]`
- **Throttle**: int8 (-255 to 255), converted to unsigned byte for negative values
- **Steering**: uint8 (0–180 degrees), 90 = neutral
- **Checksum**: `(throttle + steering + reserved) & 0xFF`

Example: [jetson_control.py](jetson_control.py#L82-L108)

### GPS State Management
Thread-safe dictionary pattern with lock:
```python
gps_state = {"lat": None, "lon": None, "ele": None, "heading": None, "aligned": False, "last_update": 0.0}
gps_lock = threading.Lock()
```
Always access within `with gps_lock:` context. Check `aligned` status (ESF-ALG message = 3) before navigation.

Example: [jetson_control.py](jetson_control.py#L10-L23)

### Distance/Bearing Calculations
Uses **flat-earth approximation** (suitable for small areas <100 km):
- Bearing: `math.atan2(sin(dlon)*cos(lat2), cos(lat1)*sin(lat2) - sin(lat1)*cos(lat2)*cos(dlon))`
- Distance: Scale degrees to meters (~111,320 m/deg lat; lon scale shrinks by `cos(latitude)`)

Note: ZED-F9R heading auto-calibrates to **true north**; no BNO055 IMU calibration needed.

Examples: [jetson_control.py](jetson_control.py#L136-L147) and [jetson_control.py](jetson_control.py#L235-L252)

### Steering Angle Mapping
Normalize relative angle to [-180, 180], then clamp to max steering range (typically 45°):
```python
def map_angle_to_steering(relative_angle, max_steering=45):
    return max(-max_steering, min(max_steering, relative_angle))
```
Adaptive throttle multiplier based on steering magnitude reduces speed on sharp turns.

Example: [jetson_control.py](jetson_control.py#L175-L191)

### Waypoint Navigation Loop
1. Read GPS state with timeout check (`age > 1.0s` triggers warning)
2. Calculate distance and bearing to waypoint
3. Compute relative angle, map to steering, apply adaptive throttle
4. Send binary packet to Arduino at 10 Hz (`time.sleep(0.1)`)
5. Stop when distance < 1.0 meter (current test threshold; subject to change based on real-world accuracy testing)

Example: [jetson_control.py](jetson_control.py#L258-L328)

### GPX File Input
Parser extracts `<trkpt lat="..." lon="...">` tags using simple string splitting:
```python
if '<trkpt' in line:
    parts = line.split('"')
    lat, lon = float(parts[1]), float(parts[3])
```
No XML library—manual parsing. Coordinates appended to list for multi-waypoint navigation.

Example: [jetson_control.py](jetson_control.py#L59-L74)

---

## Setup & Configuration

### Jetson Serial Port Setup
**GPIO pins 8 & 10** (UART) connected to ZED-F9R:
```bash
sudo systemctl stop nvgetty
sudo systemctl disable nvgetty
sudo systemctl disable serial-getty@ttyTHS1.service
```
Verify: `ls -la /dev/ttyTHS1`

### Python Dependencies
```bash
pip install pyserial
```
Ublox library is bundled in `ublox_gps/` (SparkFun port).

### Arduino Compilation
Pin assignments (Arduino Nano):
- Pin 2: RC throttle input
- Pin 3: RC steering input
- Pin 4: Motor direction (LOW=fwd, HIGH=rev)
- Pin 5: Motor PWM (0–255)
- Pin 6: Steering servo signal

Baud: 115200. Default PWM frequency used (no custom settings).

---

## Common Tasks & Workflows

### Adding a New Navigation Algorithm
- Modify `navigate_to_waypoint()` in `jetson_control.py`
- Respect 10 Hz update rate constraint; stale GPS data check must remain
- Always use `gps_lock` when accessing `gps_state`

### Debugging GPS Lock Issues
- Check alignment status: `gps_state["aligned"]` should equal True (NAV-ATT accuracy < 5°)
- Inspect raw UBX messages: `gps.geo_coords()` returns None until valid fix
- Monitor serial errors: High baud rate (38400) can fail with long cable runs

### Recording Waypoint Tracks
- `Record_data.py` records at user-defined frequency to `results/gps_log_*.{csv,gpx}`
- CSV columns: Timestamp, Lat, Lon, Elevation (m), Heading (°)
- GPX includes elevation and course (heading) in `<trkpt>` elements

### Testing Serial Protocol
Use `send_command(serial_conn, throttle, steering)` in Python or read Arduino debug output:
```
<MODE:RC;SPD:128;DIR:FWD;STR:90;RCOK:1;JETOK:1;thrPulse;1495>
```
Debug output format may evolve; check [RCandJetsoncontrol.ino](RCandJetsoncontrol/RCandJetsoncontrol.ino#L198-L237) for current format.

---

## Files Reference

| File | Purpose |
|------|---------|
| [jetson_control.py](jetson_control.py) | Main autonomous navigation & Jetson-Arduino communication |
| [RCandJetsoncontrol.ino](RCandJetsoncontrol/RCandJetsoncontrol.ino) | Arduino control logic, RC/Jetson arbitration |
| [Record_data.py](Record_data.py) | GPS data logging to CSV/GPX |
| [ublox_gps/](ublox_gps/) | Modified SparkFun Ublox GPS Python library |
| [setupnotes.txt](setupnotes.txt) | Hardware setup checklist for Jetson serial ports |

---

## Known Limitations & TODOs

- GPS config in u-center not yet documented
- BNO055 IMU support added but not integrated (ZED-F9R heading sufficient)
- ROS2 node migration planned
- No path smoothing or obstacle avoidance
- Flat-earth approximation valid only for small areas

---

## Testing & Debugging Tips

1. **Serial connection fails**: Verify `/dev/ttyUSB0` permissions. Add user to `dialout` group: `sudo usermod -a -G dialout $USER`
2. **GPS stale warnings**: Check antenna placement and sky visibility; ZED-F9R needs clear line-of-sight
3. **Steering jitter**: Deadband values in Arduino (`SERVO_DEADBAND`, `MOTOR_DEADBAND`) reduce servo chatter
4. **RC override not working**: Confirm PWM pulses arrive on pins 2/3; check neutral pulse range (1490–1550 µs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fa1conX)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Fa1conX)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
