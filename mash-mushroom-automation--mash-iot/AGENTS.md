# M.A.S.H. IoT Project - AI Coding Agent Instructions

## Project Overview
**Mushroom Automation Smart Hydro-Environment (M.A.S.H.)** - A two-layer IoT system for automated mushroom cultivation monitoring and control.

**Architecture:** Arduino (hardware layer) ↔ USB Serial ↔ Raspberry Pi (control layer) ↔ Firebase/MQTT (cloud)

## Critical Architecture Patterns

### Two-Layer Command Pattern
- **Arduino (`arduino_firmware/`)**: Hardware interface ONLY. Reads sensors, controls relays, implements safety watchdog.
- **Raspberry Pi (`rpi_gateway/`)**: Business logic, automation rules, web UI, cloud sync, database.
- **Communication**: JSON over USB Serial (`{"fruiting": {"co2": 800}, "spawning": {"co2": 2000}}`).
- **Safety**: Arduino turns off ALL relays if serial disconnects (watchdog pattern in `safety.cpp`).

### Dual I2C Sensor Setup (Critical Implementation Detail)
- **Sensor 1 (Fruiting Room)**: Hardware I2C (A4/A5) via Grove Hub
- **Sensor 2 (Spawning Room)**: Software I2C (D10/D11) - uses SoftWire library
- Both are SCD41 sensors at address 0x62, differentiated by I2C bus, NOT address
- Tag data: `room: "fruiting"` vs `room: "spawning"` in JSON payloads

### Offline-First Data Flow
1. Arduino → Serial JSON → RPi receives
2. **IMMEDIATELY** save to SQLite (`rpi_gateway/app/database/db_manager.py`)
3. **THEN** attempt Firebase sync (mark rows as synced)
4. If offline: data persists locally, syncs when connection restored

## Development Workflows

### Arduino Development (PlatformIO)
```bash
cd arduino_firmware
pio run           # Compile
pio run -t upload # Flash to Arduino
pio device monitor # Serial monitor (9600 baud)
```

**Key Files:**
- `src/main.cpp`: Main loop (read sensors + listen for serial commands)
- `src/sensors.cpp`: SCD41 dual-sensor logic (SoftWire + Wire)
- `src/actuators.cpp`: Relay control (Fan, Mist, Light)
- `src/safety.cpp`: Watchdog to disable relays on serial disconnect

**Serial Configuration:** 9600 baud, 8N1 (8 data bits, no parity, 1 stop bit)

### Raspberry Pi Development (Flask + Python)
```bash
cd rpi_gateway
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python -m app.main        # Start orchestrator
```

**Project Structure:**
- `app/main.py`: Orchestrator (starts web server + serial loop)
- `app/core/serial_comm.py`: Arduino USB communication
- `app/core/logic_engine.py`: Automation brain (threshold comparisons)
- `app/web/routes.py`: Flask endpoints
- `app/utils/wifi_manager.py`: NetworkManager-based WiFi provisioning

### WiFi Provisioning System (IMPORTANT)
The WiFi manager uses **NetworkManager CLI (nmcli)**, NOT wpa_supplicant:
- Hotspot SSID: `RPi_IoT_Provisioning` (open, no password)
- Hotspot IP: `10.42.0.1`
- Web UI accessible at `http://10.42.0.1:5000` when in hotspot mode
- On successful WiFi connection, hotspot auto-disables
- Failsafe: If connection fails after 30s, hotspot restarts

**Critical Functions in `wifi_manager.py`:**
- `start_hotspot()`: Creates AP mode connection
- `connect_to_wifi(ssid, password)`: Switches from hotspot to client mode
- `get_wifi_list()`: Scans available networks

## Project-Specific Conventions

### File Placement Rules
- **Root-level legacy files**: `app.py`, `wifi_manager.py` are TESTED prototypes. DO NOT delete.
- **New structure**: Migrate logic INTO `rpi_gateway/app/` modules, but keep root files as reference.
- **Web templates**: Must be in `rpi_gateway/app/web/templates/` for Flask Blueprint routing.
- **Config files**: `rpi_gateway/config/config.yaml` (thresholds), `.env` (secrets).

### Serial Communication Pattern
**Arduino sends JSON:**
```json
{"fruiting": {"temp": 23.5, "humidity": 85, "co2": 800}, "spawning": {"temp": 24.0, "humidity": 90, "co2": 1200}}
```

**RPi sends commands (plain text):**
```
FAN_ON
FAN_OFF
MIST_ON
LIGHT_OFF
```

### Flask Blueprint Pattern
Use `Blueprint` for all routes (see `rpi_gateway/app/web/routes.py`):
```python
web_bp = Blueprint('web', __name__, template_folder='templates')
```
Register in `main.py` with: `app.register_blueprint(web_bp)`

### Web UI Resolution Constraints (CRITICAL)

**Target Display: 1024x600** (Raspberry Pi Official 7" Touchscreen)

The Flask web interface runs on the same display as the Flutter mobile app. **ALL UI elements MUST be readable with the naked eye at 1024x600 resolution**.

**Design Guidelines:**
- **Minimum font sizes:**
  - Body text: 14px minimum
  - Headings: 18px minimum
  - Sensor values/metrics: 20px minimum
  - Button labels: 16px minimum

- **Touch targets:**
  - Minimum button/link area: 48x48px (Material Design standard)
  - Spacing between interactive elements: 8px minimum

- **Layout considerations:**
  - Use responsive CSS (flexbox, grid) for different viewports
  - Prioritize critical sensor data in above-the-fold area
  - Avoid excessive padding that wastes screen real estate
  - Test all templates at 1024x600 during development

- **Common issues to avoid:**
  - Text smaller than 14px (hard to read at arm's length)
  - Buttons smaller than 48px (difficult to tap accurately)
  - Fixed-width layouts that require horizontal scrolling
  - Critical controls hidden below the fold without scroll indicators

**Testing:**
```bash
# Test on RPi display
python -m app.main
# Access at http://localhost:5000 on RPi touchscreen

# Or test on desktop with browser window set to 1024x600
```

## Hardware Constraints

### Relay Module (8-Channel, Active Low)
- **Trigger**: Writing `LOW` to pin activates relay
- **Safety**: Arduino resets all relays to `HIGH` (OFF) on startup
- **Mapping**: Define in `actuators.cpp` (e.g., Relay 1 = Fruiting Fan, Relay 2 = Mist Maker)

### Voltage Levels
- **RPi GPIO**: 3.3V logic
- **Arduino GPIO**: 5V logic
- **Communication**: USB Serial (no level shifter needed)
- **Relay trigger**: 5V (Arduino-controlled, safe)

## Configuration Files

### Firebase Setup (`rpi_gateway/config/firebase_config.json`)
Contains production credentials. DO NOT commit to public repos. Use `.gitignore`.

### MQTT Broker (`rpi_gateway/config/.env`)
HiveMQ Cloud credentials for low-latency control:
- Broker: `290691cd2bcb4d7faf979db077890beb.s1.eu.hivemq.cloud:8883`
- TLS required, credentials in `.env`

### Thresholds (`rpi_gateway/config/config.yaml`)
Example structure:
```yaml
fruiting_room:
  temp_target: 24
  humidity_target: 90
  co2_max: 1000
spawning_room:
  temp_target: 25
  humidity_target: 95
```

## Testing Patterns

### Arduino Serial Testing
1. Upload firmware
2. Open Serial Monitor (9600 baud)
3. Expect JSON output every 5 seconds
4. Send commands: `FAN_ON`, `MIST_OFF` → verify relay clicks

### RPi Integration Testing
1. Mock Arduino with `pyserial` loopback
2. Test routes: `curl http://localhost:5000/`
3. Check SQLite: `sqlite3 rpi_gateway/data/sensor_data.db`
4. Test ML predictions: `python -m app.core.logic_engine` (runs inference on test data)

## Migration Notes (from old repo)
- `database/db_manager.py`: SQLAlchemy ORM → migrate existing schema
- `cloud/firebase.py`: Admin SDK initialization → use `firebase_config.json`
- `utils/wifi_manager.py`: Already migrated and TESTED ✓

## Common Pitfalls
1. **Don't use `sudo` for nmcli**: Configure polkit rules instead
2. **Serial buffer overflow**: Arduino should flush buffer before reading
3. **GPIO cleanup**: Always reset relay pins to HIGH on Arduino reboot
4. **Flask static files**: Must be in `rpi_gateway/app/web/static/` with correct blueprint config

## Machine Learning Architecture

### Two-Stage ML Pipeline (Scikit-Learn)
**Why scikit-learn?** Lightweight, runs on RPi 3B (1GB RAM), no GPU needed, inference < 10ms

**Stage 1: Anomaly Detection**
- Algorithm: Isolation Forest (`sklearn.ensemble.IsolationForest`)
- Purpose: Filter sensor noise/hardware faults (e.g., temp spike to 70°C)
- Model: `rpi_gateway/data/models/isolation_forest.pkl`
- Training: Weekly re-training on RPi during low-activity periods

**Stage 2: Actuation Control**
- Algorithm: Decision Tree Classifier (`sklearn.tree.DecisionTreeClassifier`)
- Purpose: Automate relay states (Fan/Mist/Light) based on env conditions
- Model: `rpi_gateway/data/models/decision_tree.pkl`
- Features: `[temp, humidity, co2, time_of_day]`
- Output: `[fan_state, mist_state, light_state]`

**Implementation:** See `docs/MACHINE_LEARNING.md` for training code and baseline data

## Database Architecture

### Local (Offline-First)
- **SQLite**: `rpi_gateway/data/sensor_data.db`
- Schema: See `docs/SCHEMA.md` for full table definitions
- Critical tables: `sensor_data`, `device_commands`, `sync_queue`

### Cloud (Sync Layer)
- **PostgreSQL (Neon)**: Production backend database
- **Firebase Realtime DB**: IoT device messaging
- **Backend API**: Node.js/Express REST API (see `docs/API.md`)

## Deployment & System Integration

### Auto-Start on Boot (Planned)
- Systemd service for `rpi_gateway/app/main.py`
- Chromium kiosk mode auto-launch to `http://localhost:5000`
- Custom boot splash screen (replace Raspberry Pi logo with M.A.S.H. branding)

### Kiosk Mode Setup
- Browser: Chromium in fullscreen kiosk mode
- Target: `http://localhost:5000` (Flask web UI)
- Configuration script: `scripts/setup_kiosk.sh` (in progress)

## Version Management (MANDATORY)

**CRITICAL: ALL code changes MUST update version numbers**

### When to Bump Version (Semantic Versioning)
Follow the versioning file: `rpi_gateway/app/core/version.py`

**PATCH (x.y.Z)** - Bug fixes and minor improvements:
- Bug fixes that don't change API behavior
- Documentation updates
- Performance optimizations
- Code refactoring without feature changes
- Example: 2.0.0 -> 2.0.1

**MINOR (x.Y.0)** - New features (backward compatible):
- New API endpoints
- New actuator/sensor support
- New configuration options
- UI feature additions
- Enhanced error handling
- Example: 2.0.1 -> 2.1.0

**MAJOR (X.0.0)** - Breaking changes:
- API contract changes (endpoint removal/modification)
- Serial protocol changes (Arduino <-> RPi)
- Database schema migrations
- Configuration file format changes
- Example: 2.1.0 -> 3.0.0

### Version Update Workflow (MANDATORY)
1. **Before writing code**: Determine if change is MAJOR/MINOR/PATCH
2. **Update version.py**: Increment MAJOR, MINOR, or PATCH
3. **Update RELEASE_DATE**: Set to current date
4. **Update RELEASE_NAME**: Descriptive name for the release
5. **Update FEATURES dict**: Add/modify feature flags as needed
6. **Add CHANGELOG_REGISTRY entry**: Add a new entry at the TOP of the list in version.py
7. **Create changelog file**: Create `changelogs/vX.Y.Z.md` (see template below)
   - **NEVER modify previous changelog files** (integrity)
   - Use standardized tags and user-friendly language
8. **Verify**: Run `python rpi_gateway/app/core/version.py` to confirm

### Changelog File Template (Strict)

Each version gets its own file at `rpi_gateway/app/core/changelogs/vX.Y.Z.md`:

```markdown
## vX.Y.Z (YYYY-MM-DD) - Release Name
Priority: high|medium|low

Brief summary of the release milestone in user-friendly language.

### New
- Feature description (no function names or technical jargon)

### Improved
- Enhancement description

### Fixed
- Bug fix description (describe the user-visible problem, not the code fix)

### Changed
- Behavior change description (non-breaking)

### Breaking
- Breaking change description (MAJOR version only)

### Security
- Security fix description

### Known Limitations
- Limitation description
```

**Standardized Tags** (use only these, omit sections with no entries):

| Tag | Purpose |
|-----|---------|
| `New` | New features or capabilities |
| `Improved` | Enhancements to existing features |
| `Fixed` | Bug fixes |
| `Changed` | Behavior changes (non-breaking) |
| `Breaking` | Breaking changes (MAJOR only) |
| `Security` | Security fixes |
| `Known Limitations` | Known issues or limitations |

**Priority Levels** (for future OTA update notifications):
- `high` - Required update, security fix, or critical bug fix
- `medium` - Recommended update, new features
- `low` - Optional, documentation, minor improvements

**Language Rules**:
- Write for users, not developers
- BAD: "Fixed check_connection() timing bug in backend_api.py"
- GOOD: "Fixed connection retry logic that caused excessive server requests"
- Never reference function names, file paths, or variable names

### Examples of Version Bumps

**PATCH Example (2.0.0 -> 2.0.1):**
- Fix WiFi reconnection bug
- Improve logging clarity
- Update documentation typos

**MINOR Example (2.0.1 -> 2.1.0):**
- Add `/api/version` endpoint
- Implement on-screen keyboard support
- Add mDNS service discovery
- New feature flags in config

**MAJOR Example (2.1.0 -> 3.0.0):**
- Change serial protocol from JSON to Protocol Buffers
- Rename API endpoints (breaking mobile app compatibility)
- Require new config.yaml structure

### Enforcement Rules
1. **NO PR/commit without version bump** (except docs-only changes)
2. **NEVER edit old changelog files** - create new files only
3. **ALWAYS update RELEASE_DATE** to match current date
4. **Test version endpoint**: Verify `/api/version` returns correct info
5. **Run version.py**: Confirm output shows updated version

### Version Coordination
Components with direct communication require coordinated updates:
- **RPi ↔ Arduino**: MAJOR version changes in serial protocol require both to update
- **Mobile ↔ RPi**: MINOR version changes in local API require mobile app update
- **Mobile ↔ Backend**: API changes require backend + mobile coordination

## Key Decision Rationale
- **Why separate Arduino/RPi?** Isolation: hardware failures don't crash business logic
- **Why SQLite + Firebase?** Offline-first: data survives network outages
- **Why NetworkManager?** Modern, handles conflicts better than `wpa_supplicant`
- **Why JSON over Serial?** Human-readable debugging, extensible schema
- **Why scikit-learn ML?** Lightweight enough for RPi 3B, interpretable models (white-box), no GPU required

---

**Documentation:** 
- `README.md` - Hardware specs, project overview
- `STRUCTURE.md` - Folder layout
- `docs/MACHINE_LEARNING.md` - ML training/inference code
- `docs/SCHEMA.md` - Database schema (SQLite + PostgreSQL)
- `docs/API.md` - Backend REST API reference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MASH-Mushroom-Automation)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MASH-Mushroom-Automation)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
