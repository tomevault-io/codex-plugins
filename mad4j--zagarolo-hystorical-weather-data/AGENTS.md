# Zagarolo Historical Weather Data

Historical weather data collection system that downloads meteorological data from the Open-Meteo API and stores it in SQLite databases for analysis. The system focuses on weather data for Zagarolo, Italy (coordinates: 41.837610, 12.831867) but can work with any geographic location.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Environment Setup
- Ensure Python 3.12+ is available: `python3 --version`
- Install required dependencies: `pip3 install requests pandas`
- NEVER CANCEL: Dependency installation typically takes 1-2 minutes. Set timeout to 5+ minutes.
- Install linting tools: `pip3 install flake8`

### Core Workflow Commands
Bootstrap and validate the repository:
- Syntax check: `python3 -m py_compile *.py` -- takes <1 second
- Lint code: `flake8 *.py` -- takes <1 second
- Test script help: `python3 openmeteo_downloader.py --help` -- takes <1 second

Download weather data (requires internet access):
- Basic download: `python3 openmeteo_downloader.py LATITUDE LONGITUDE YEAR`
- Example: `python3 openmeteo_downloader.py 41.837610 12.831867 2024 --no-hourly --formats json`
- NEVER CANCEL: API downloads typically take 5-30 seconds depending on data size. Set timeout to 2+ minutes.
- Large downloads: Use `--chunked` flag for multi-year requests to avoid timeouts

Work with SQLite database:
- Create database: `python3 create_weather_db.py weather.db` -- takes <1 second
- Import JSON data: `python3 import_weather_json_to_sqlite.py weather_data/FILENAME.json weather.db` -- takes <1 second
- Add additional data: `python3 import_weather_json_to_sqlite.py FILE.json weather.db --no-create-tables`
- Query data: `sqlite3 weather.db "SELECT time, temperature_2m_max FROM daily ORDER BY temperature_2m_max DESC LIMIT 5;"`

## Network Limitations

IMPORTANT: The Open-Meteo API (archive-api.open-meteo.com) may not be accessible in all environments due to network restrictions. If downloads fail with "No address associated with hostname" or connection errors:
- This is expected in restricted environments
- Use the existing sample data in `weather_data/` directory for testing
- Document the network limitation in any code changes
- Focus on database and analysis functionality rather than API downloads

## Validation

ALWAYS run through at least one complete end-to-end scenario after making changes:

1. **Database Creation Test**:
   ```bash
   rm -f test_weather.db
   python3 create_weather_db.py test_weather.db
   ```

2. **Data Import Test**:
   ```bash
   python3 import_weather_json_to_sqlite.py weather_data/weather_2024_41.83761_12.831867.json test_weather.db
   ```

3. **Query Validation**:
   ```bash
   sqlite3 test_weather.db "SELECT COUNT(*) FROM daily;"
   sqlite3 test_weather.db "SELECT time, temperature_2m_max FROM daily ORDER BY temperature_2m_max DESC LIMIT 1;"
   ```

4. **Multi-Year Import Test**:
   ```bash
   python3 import_weather_json_to_sqlite.py weather_data/weather_2023_41.83761_12.831867.json test_weather.db --no-create-tables
   sqlite3 test_weather.db "SELECT COUNT(DISTINCT substr(time, 1, 4)) FROM daily;"
   ```

Always run `flake8 *.py` before committing changes to ensure code style compliance.

## Common Tasks

### Repository Structure
```
.
├── README.md                           # Project documentation
├── LICENSE                            # MIT license
├── notes.txt                         # Usage examples and coordinates
├── openmeteo_downloader.py           # Main weather data downloader
├── create_weather_db.py              # SQLite database schema creator
├── import_weather_json_to_sqlite.py  # JSON to SQLite importer
├── weather_data/                     # Sample JSON weather data (2019-2024)
└── weather.db                        # Pre-existing SQLite database
```

### Sample Data Files
The repository includes sample weather data for Zagarolo (41.837610, 12.831867):
- `weather_data/weather_2019_41.83761_12.831867.json` (96K, daily data)
- `weather_data/weather_2020_41.83761_12.831867.json` (96K, daily data)
- `weather_data/weather_2021_41.83761_12.831867.json` (96K, daily data)
- `weather_data/weather_2022_41.83761_12.831867.json` (96K, daily data)
- `weather_data/weather_2023_41.83761_12.831867.json` (96K, daily data)
- `weather_data/weather_2024_41.83761_12.831867.json` (96K, daily data)

Each JSON file contains ~7000 lines with daily weather metrics including temperature, precipitation, wind, and solar radiation data.

### Key Script Parameters
- `openmeteo_downloader.py`: Supports coordinates, year, output formats (json/csv), timezone, units, chunked downloads
- `--no-hourly` / `--no-daily`: Exclude specific data types to reduce file size
- `--chunked --chunk-size N`: Download large datasets in N-month chunks
- `--save-metadata`: Include additional metadata in output

### Database Schema
- `metadata` table: Location info, timezone, generation details
- `hourly` table: Hour-by-hour weather measurements (when requested)
- `daily` table: Daily aggregated weather statistics
- Dynamic columns added based on available weather variables

### Common Queries
```sql
-- Find hottest day
SELECT time, temperature_2m_max FROM daily ORDER BY temperature_2m_max DESC LIMIT 1;

-- Monthly precipitation totals
SELECT substr(time, 1, 7) as month, SUM(precipitation_sum) FROM daily GROUP BY month;

-- Temperature extremes by year
SELECT substr(time, 1, 4) as year, MAX(temperature_2m_max), MIN(temperature_2m_min) FROM daily GROUP BY year;
```

## Development Notes

- Code is primarily in Italian (comments, variable names in DB creation)
- No existing test suite - validate manually using commands above
- No CI/CD pipeline configured
- Standard Python style with flake8 linting available
- Uses requests for HTTP, pandas for data processing, sqlite3 for database operations
- All operations are very fast (<1 second each) except API downloads which depend on network

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mad4j)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mad4j)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
