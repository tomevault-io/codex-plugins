# dsnj-homie Copilot Instructions

## Project Overview
IoT home automation system for monitoring heating, hot water, solar production, and weather data. Data flows from multiple sensors and APIs into a MySQL database for analysis and control decisions.

## Architecture & Data Flow

**Data Collection Stack:**
- **Temperature Sensors** (Python): `fetch1PMplus.py`, `fetch1PMplusAussen.py` - Read heat pump storage temps
- **Heating System** (JavaScript): `fetchTOventrop.js`, `fetchTOventropOutside.js` - Scrape Oventrop portal via Puppeteer
- **Solar & Energy** (Python): `fetch3EMSolar.py`, `getStromrechnungElektra.py` - Query solar inverters and electricity usage
- **Weather** (Python): `getWeather.py` - OpenWeatherMap API integration
- **Control Logic** (Python): `startWP.py`, `stopWP.py` - Heat pump automation based on thresholds

**Database:** MySQL (`solar` database) with tables for temperatures, weather, and billing data.

## Critical Patterns & Conventions

### Process Locking
All long-running scripts use file-based locking to prevent duplicate execution:
```python
def instance_already_running():
    lock_file_pointer = os.open(f"/tmp/{script_name}.lock", os.O_WRONLY | os.O_CREAT)
    try:
        fcntl.lockf(lock_file_pointer, fcntl.LOCK_EX | fcntl.LOCK_NB)
        return False
    except IOError:
        return True
```
Always add this check to any new background task.

### Configuration Management
Sensitive credentials stored in `config.py` (imported via `import config`):
- `DB_NAME`, `DB_USER`, `DB_PASS` - MySQL connection
- Environment variables loaded via `.env` for Node scripts (dotenv)

### Logging Convention
All scripts log to dated files in project root:
- Python: `logging.basicConfig(filename='script.log', ...)`
- Shell: `>> script.log` appends
- Check for hung processes by analyzing log frequency (see `startGetTemp.sh` pattern)

### Database Patterns
- Prepared statements for SQL injection prevention: `cursor.execute(query, (id, ext_temp, ...))`
- Timestamps: Use `datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")`
- Always call `connection.commit()` after INSERT/UPDATE; use try/except with `connection.rollback()`

### Scheduling
Shell scripts wrapped with process health checks:
- `startGetTemp*.sh` - Check if process already running, kill if hanging
- Bash: Use `ps -x | grep 'process_name' | grep -v grep` pattern
- Node.js: Wrapped by Python subprocess calls (`checkProcess.py`)

## Key Dependencies

**Node.js:** Puppeteer (web scraping), Sequelize (ORM), mysql2, dotenv
**Python:** requests, mysql-connector, logging, fcntl (file locking), bs4 (BeautifulSoup)

## Heat Pump Control Logic
`startWP.py` implements seasonal thresholds:
- **Winter** (Nov-Feb): `min_temp=50, solar_prod_level=1000, red_for_bad_weather=750`
- **Summer**: `min_temp=50, solar_prod_level=1300, red_for_bad_weather=1000`
- Decision: Turn on/off based on storage temperature, solar production, and weather conditions

## Common Tasks

**Add a new sensor:** Create fetch script with locking + logging, insert via prepared statement, schedule in shell wrapper.

**Debug data flow:** Check logs in project root (`.log` files), validate DB connection in `config.py`, verify cron/schedule invocations.

**Modify thresholds:** Edit winter/summer conditions in `startWP.py` or solar constants in `fetch1PMplus.py`.

## Development Notes
- No test framework configured (`"test": "echo 'no test specified'"`)
- Mixed Python/Node.js codebase - coordinate dependencies separately
- Puppeteer requires special flags for containerized environments (see existing `args` in `fetchTOventrop.js`)

---
> Source: [grooveculture/solar_heat_pump](https://github.com/grooveculture/solar_heat_pump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
