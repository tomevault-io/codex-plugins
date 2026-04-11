# JPR Election Results Scraper - AI Coding Agent Guide

## Project Overview

This is an **election results scraping and visualization system** that:
1. Fetches real-time election results from Oregon and California state APIs
2. Processes and stores data in JSON (for history) and CSV (for analysis)
3. Auto-updates Datawrapper charts for publication
4. Runs automatically via GitHub Actions triggered by cron-job.org

**Primary Use Case**: Election night live results tracking for JPR (Jefferson Public Radio)

## Architecture & Data Flow

### Two Main Scripts (Election-Specific)

1. **`JPRscraper.py`** - General election scraper (Nov 2024)
2. **`Mayscraper.py`** - May primary scraper (May 2025)

Each follows the same pattern but with election-specific race IDs and API parameters.

**Data Flow**:
```
API Request → JSON Storage (jsons/*.json) → CSV Processing → Datawrapper Update → Auto-publish
```

### API Structure Differences

**California** (uses SOS API):
- Single API call returns all desired races
- URL format: `https://api.sos.ca.gov/returns/[endpoint]`
- Race IDs hardcoded in URL as array: `["13000001000059","13000002000059",...]`

**Oregon** (uses orresultswebservices API):
- Iterative approach: one API call per race ID
- URL format: `https://orresultswebservices.azureedge.us/ResultsAjax.svc/GetMapData?type=TYPE&category=CATEGORY&raceID=ID&osn=0&county=0&party=0`
- Race IDs read from `oregon_raceids.txt` or hardcoded arrays
- Critical `type` parameter values:
  - `MEASURE` - Statewide measures
  - `LMEA` - Local measures (requires specific raceID + category=CTY)
  - `HOUSE` / `SENATE` - State legislature
  - `SWPAR` - Statewide partisan races

**Oregon API Quirk**: Yes/No votes for measures are returned as separate "candidates" in the response. Code must iterate through all candidates for a measure, match by `RaceID`, and combine Yes/No data.

## Critical Configuration Points

### Before Each Election

1. **Update Race IDs**:
   - Oregon: Edit `oregon_raceids.txt` or hardcoded arrays in scripts (search for `oregon_ids`, `oregon_measure_ids`)
   - California: Update race ID arrays in API URLs (search for `api.sos.ca.gov/returns/query?r=`)

2. **API URL Verification**:
   - Both states may change API endpoints between elections
   - Contact state election offices before election day (especially Oregon - no public documentation)
   - Marked with `#NOTE:` comments in code

3. **Datawrapper Chart IDs**:
   - Create charts manually first in Datawrapper
   - Copy chart IDs from Datawrapper URLs
   - Update all `chart_id` parameters in `dw.add_data()`, `dw.update_chart()`, and `dw.publish_chart()` calls

4. **Timezone Handling**:
   - All timestamps use Pacific Time (`pytz.timezone('US/Pacific')`)
   - **PST vs PDT**: Manually update timezone abbreviation in chart metadata based on election date
   - Search for `"notes": f"Last updated: {latest_time} PST"` - change `PST` to `PDT` if needed

### Environment Variables

- `DATAWRAPPER_API_KEY` - Required for chart updates
  - Set in GitHub Secrets for Actions
  - Or set locally as environment variable for manual runs

## Key Conventions

### File Naming Pattern
```python
timenow = datetime.datetime.now(tz=pacific_tz).strftime("%Y-%m-%d_%H-%M")
filename = f"jsons/oregon_measures_{timenow}.json"
```

### CSV Structure
- **Measures**: `["Measure", "Yes Votes", "Yes %", "No Votes", "No %"]`
- **Candidate Races**: `["Race", "Candidate", "Party", "Votes", "Percent"]`

### Data Processing Pattern (Oregon)
```python
# For each race ID:
# 1. Fetch JSON from API
# 2. Append/update combined JSON file
# 3. Extract & append to CSV
# 4. Sort DataFrame and overwrite CSV
# 5. Update Datawrapper chart
```

### Percentage Handling
- **2024 (JPRscraper.py)**: API returns decimals → multiply by 100
- **2025 (Mayscraper.py)**: Same pattern
- Always check API response format in new elections

### JSON File Cleanup
Scripts auto-delete JSON files older than 24 hours:
```python
for filename in os.listdir('jsons/.'):
    if (now - file_time).total_seconds() > 24 * 3600:
        os.remove(f'jsons/{filename}')
```

## Running the Scraper

### Local Development
```powershell
# Install dependencies
pip install requests datawrapper pandas pytz

# Set API key
$env:DATAWRAPPER_API_KEY = "your_key_here"

# Run scraper
python Mayscraper.py  # or JPRscraper.py
```

### Automated Execution
- GitHub Action: [.github/workflows/scraper-app.yml](\workflows\scraper-app.yml)
- Triggered by: cron-job.org via workflow_dispatch (more reliable than GitHub's built-in scheduler)
- Default: 3 minutes past every hour
- Timeout: 5 minutes
- Runs: Ubuntu latest, Python 3.10
- **Important**: Update `run: python Mayscraper.py` to correct script name in workflow file

## Common Modifications

### Adding a New Race
1. Find race ID from state results website or API export
2. Add to relevant array (e.g., `oregon_ids`) or `oregon_raceids.txt`
3. Verify API `type` parameter matches race type
4. Test locally before election

### Creating New Charts
1. Run scraper to generate CSV
2. Manually create chart in Datawrapper using CSV
3. Copy chart ID from Datawrapper URL
4. Add chart update code block (see existing patterns)

### Debugging API Issues
- Check API response structure changes: `json.dumps(response.json(), indent=4)`
- Verify all query parameters present in URL
- Oregon: Confirm `type` matches race category
- Test with `raceID=0` to see all available races for that type

## Notable Code Patterns

### Party Normalization (California)
```python
if candidate.get("Party") == "Dem":
    party = "Democratic"
elif candidate.get("Party") == "Rep":
    party = "Republican"
```

### Incumbent Detection
```python
if candidate.get("incumbent") == True:
    name = candidate.get("Name") + " (Incumbent)"
```

### Measure Number Extraction (Oregon 2024)
```python
race_name = re.search("Measure ...", measure["RaceName"]).group(0)
```

## Dependencies
- `requests` - API calls
- `datawrapper` - Chart updates
- `pandas` - CSV manipulation & sorting
- `pytz` - Timezone handling
- Standard library: `datetime`, `json`, `csv`, `re`, `os`

## Important Notes
- Scripts designed for **specific elections** - not plug-and-play
- Always test with real API before election night
- Keep `oregon_raceids.txt` updated and version controlled
- JSON files are for debugging/history; CSVs drive charts
- All `#NOTE:` comments mark election-specific configuration points

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/battaglir)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/battaglir)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
