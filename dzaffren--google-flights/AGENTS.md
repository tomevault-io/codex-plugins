# Copilot Instructions for Flight Predictor

## Project Overview

**Flight Predictor** is an Azure Functions project that scrapes Google Flights data and stores flight price information in Azure Blob Storage. The project runs on a timer trigger (daily at 08:00 UTC) to monitor flight prices from Kuala Lumpur (KUL) to London (LHR).

### Architecture

- **function_app.py**: Azure Functions entry point containing the timer-triggered function `google_flights()`
- **core.py**: Core business logic with two main functions:
  - `extract_best_price_flight()`: Parses Google Flights API response (via SerpAPI) and extracts key flight data
  - `upload_to_blob()`: Uploads extracted flight data as JSON to Azure Blob Storage
- **host.json**: Azure Functions runtime configuration (v2.0, Application Insights logging, ExtensionBundle v4-5)
- **requirements.txt**: Python dependencies (azure-functions, serpapi)

### Data Flow

1. Timer trigger fires at 08:00 UTC daily
2. SerpAPI queries Google Flights for KUL→LHR flights (hardcoded departure date: 2026-12-25)
3. Best flight option is extracted from response
4. Flight data (price, airline, stops, carbon emissions, etc.) is formatted as JSON
5. JSON blob is uploaded to Azure Blob Storage with filename: `YYYY-MM-DD.json`

## Build, Test, and Run

### Local Development Setup

```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # macOS/Linux
# or: venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Install Azure Functions Core Tools (if not already installed)
# See: https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local
```

### Running Locally

```bash
# Start the local Azure Functions runtime
func start

# Timer trigger will execute based on the schedule in function_app.py
```

### Environment Configuration

The project requires these environment variables to run:

- `SERPAPI_KEY`: Your SerpAPI API key for Google Flights queries
- `AZURE_STORAGE_CONNECTION_STRING`: Azure Storage account connection string

For local development, create a `local.settings.json` file (template provided in `.gitignore`):

```json
{
  "IsEncEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "SERPAPI_KEY": "your-key-here",
    "AZURE_STORAGE_CONNECTION_STRING": "your-connection-string-here"
  }
}
```

**Note**: `local.settings.json` is git-ignored for security; never commit secrets.

### Testing

There are currently no automated tests. When adding tests:

- Use pytest as the testing framework (industry standard for Python)
- Place test files in a `tests/` directory
- Test `extract_best_price_flight()` with mock SerpAPI responses
- Test `upload_to_blob()` with Azure Storage mocks

## Key Conventions

1. **Hardcoded Values**: The function currently hardcodes route (KUL→LHR) and departure date (2026-12-25). These should be parameterized for flexibility.

2. **Error Handling**: Functions use try-except blocks with logging. Non-fatal errors (e.g., "No flights found") are logged as warnings; fatal errors are logged as exceptions.

3. **Blob Naming**: Flight data is stored with ISO date format filenames (`YYYY-MM-DD.json`), making it easy to query historical data.

4. **Schedule Expression**: The timer trigger uses cron expression `* * 8 * * *` (daily at 08:00 UTC). See [Azure Functions timer trigger reference](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=python#ncrontab-expressions) for format.

5. **Logging**: Uses Python's standard `logging` module. All function entry/exit and errors are logged.

## Deployment

The project is deployed to Azure Functions via GitHub Actions. See `.github/workflows/` for deployment pipelines.

### Azure Setup

- Function App must have storage account and blob container `flightdata` already created
- Application Insights should be configured in `host.json` for monitoring

## Dependencies

- **azure-functions**: Python worker for Azure Functions runtime
- **serpapi**: Google Flights API client (queries via SerpAPI service)
- **azure-storage-blob**: Azure Blob Storage client

See `requirements.txt` for pinned versions. The `azure-functions-worker` is managed by the Azure platform and should not be manually added.

## ML Notebook

A Jupyter notebook for flight price forecasting lives in `notebooks/flight_analysis.ipynb`.

### Setup

```bash
cd notebooks
pip install -r requirements.txt
cd ..
jupyter notebook notebooks/flight_analysis.ipynb
```

Requires `AZURE_STORAGE_CONNECTION_STRING` in a `.env` file at the repo root.

### What it does (6 sections)
1. **Load data** — pulls all JSON blobs from the `flightdata` container into a DataFrame
2. **EDA** — price history, distribution, summary stats
3. **Feature engineering** — `days_to_departure`, `day_of_week`, etc.
4. **Forecasting** — Facebook Prophet trained on `(scrape_date, price)`, forecasts daily up to 2026-12-25
5. **Buy/hold signal** — compares today's price to predicted future minimum; outputs `BUY` if within 5% tolerance
6. **Visualisations** — price history + Prophet forecast with 90% confidence interval

The `notebooks/requirements.txt` is kept separate from the Azure Function's `requirements.txt` to avoid polluting the deployed function with ML packages.

## MCP Servers

The following MCP servers are recommended for enhanced developer experience:

### Azure Observability
Useful for monitoring Application Insights logs, metrics, and traces from the deployed function. Enable this to query logs, check function health, and review performance metrics.

### Filesystem Operations
Standard file management operations that complement the built-in tools for code editing and exploration.

---
> Source: [dzaffren/google-flights](https://github.com/dzaffren/google-flights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-15 -->
