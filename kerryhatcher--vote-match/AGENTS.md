# Copilot Instructions for Vote Match

## Project Overview

**Vote Match** is a Python CLI tool for processing voter registration records for GIS applications. It:

1. **Loads** voter data from CSV files into PostGIS database
2. **Geocodes** addresses via multiple services (Census, Nominatim, Mapbox, Geocodio, Google Maps, Photon)
3. **Validates** addresses via USPS API
4. **Compares** voter district assignments against spatial boundaries (GeoJSON/Shapefile)
5. **Generates** interactive Leaflet maps with district mismatches
6. **Uploads** maps to Cloudflare R2 storage

### Key Use Case

Identify voters whose registered district (per Secretary of State) does not match their physical location (per GIS boundaries). Supports **all 15 district types** in Georgia voter data: congressional, state senate, state house, county commission, judicial, school board, city council, municipal school board, water board, super council, super commissioner, super school board, fire district, municipality, county precinct.

---

## Language & Runtime

- **Python 3.13+** (required)
- **Package manager**: `uv` ONLY (never `python -m pip`, `pip`, or `poetry`)
- **Build**: `pyproject.toml` with hatchling backend
- **CLI framework**: Typer with Rich for styled terminal output
- **Dependencies**: pandas, loguru, Jinja2, SQLAlchemy 2.0, GeoAlchemy2, Alembic, httpx

### Installation Commands

```bash
# Install dependencies
uv sync

# Run CLI
uv run vote-match --help

# Run tests
uv run pytest

# Lint/format
uv run ruff check
uv run ruff format
```

---

## Code Style & Linting

### Ruff Configuration

- **Line length**: 100 characters (hard limit)
- **Target**: Python 3.13
- **Config**: `[tool.ruff]` in `pyproject.toml`

### Style Guidelines

```python
# ✅ CORRECT: Python 3.13 union syntax
def get_voter(id: str | None) -> Voter | None:
    ...

# ❌ WRONG: Optional type (outdated)
from typing import Optional
def get_voter(id: Optional[str]) -> Optional[Voter]:
    ...

# ✅ CORRECT: String column for district IDs
district = Column(String, nullable=True)  # Preserves "026" vs "26"

# ❌ WRONG: Integer column (loses leading zeros)
district = Column(Integer, nullable=True)  # "026" becomes 26
```

### Import Order

1. Standard library (`os`, `pathlib`, `json`)
2. Third-party (`typer`, `loguru`, `sqlalchemy`)
3. Local modules (`vote_match.models`, `vote_match.processing`)

---

## Database & ORM

### Technologies

- **Database**: PostgreSQL with PostGIS extension
- **ORM**: SQLAlchemy 2.0+ (declarative style with `declarative_base()`)
- **Geometry**: GeoAlchemy2 for PostGIS columns
- **Migrations**: Alembic for schema versioning
- **CRS**: EPSG:4326 (WGS84) for all spatial data

### Key Models

```python
# src/vote_match/models.py

Base = declarative_base()

# District type key → Voter column name mapping
DISTRICT_TYPES: dict[str, str] = {
    "county": "county",
    "congressional": "congressional_district",
    "state_senate": "state_senate_district",
    "state_house": "state_house_district",
    "county_commission": "county_commission_district",
    "school_board": "school_board_district",
    # ... 15 types total
}

class Voter(Base):
    """Voter registration record with geocoding results."""
    __tablename__ = "voters"
    
    # All columns are String (not Integer) to preserve leading zeros
    voter_registration_number = Column(String, primary_key=True)
    congressional_district = Column(String, nullable=True)  # "026" not 26
    residence_zipcode = Column(String, nullable=True)  # "30901" not 30901
    
    geom = Column(Geometry("POINT", srid=4326), nullable=True)

class GeocodeResult(Base):
    """Multi-service geocoding results."""
    __tablename__ = "geocode_results"
    
    voter_id = Column(String, ForeignKey("voters.voter_registration_number"))
    service_name = Column(String(50))  # 'census', 'nominatim', etc.
    status = Column(String(20))  # 'exact', 'interpolated', 'approximate', 'no_match', 'failed'
    
class DistrictBoundary(Base):
    """Universal district boundary model (replaces legacy CountyCommissionDistrict)."""
    district_type = Column(String(50), nullable=False)  # Key from DISTRICT_TYPES
    geom = Column(Geometry("MULTIPOLYGON", srid=4326))

class VoterDistrictAssignment(Base):
    """Tracks spatial district assignments and mismatches."""
    voter_id = Column(String, ForeignKey("voters.voter_registration_number"))
    district_type = Column(String(50))
    spatially_assigned_district_id = Column(String(10))
    registered_district_id = Column(String(10))
    is_mismatch = Column(Boolean)
```

### Critical Rules

1. **ALL string columns must be `String`** (never `Integer`) to preserve leading zeros in:
   - Voter registration numbers
   - District IDs ("026", "07", "003")
   - ZIP codes ("30901", "01234")
   - FIPS codes ("13021", "01001")

2. **All geometry must be EPSG:4326** (WGS84 lat/lon)
   - Reproject shapefiles/GeoJSON before import
   - Use `geopandas.to_crs("EPSG:4326")` for conversions

3. **Batch operations commit in chunks** (default 1000 records)
   ```python
   for i, batch in enumerate(chunks(records, 1000)):
       session.bulk_save_objects(batch)
       session.commit()
   ```

4. **Use PostgreSQL upserts** for bulk loads:
   ```python
   from sqlalchemy.dialects.postgresql import insert as pg_insert
   
   stmt = pg_insert(Voter).values(records)
   stmt = stmt.on_conflict_do_update(
       index_elements=["voter_registration_number"],
       set_=dict(residence_city=stmt.excluded.residence_city)
   )
   session.execute(stmt)
   ```

5. **Close sessions and dispose engines** in CLI commands:
   ```python
   @app.command()
   def my_command():
       engine = get_engine(settings)
       session = get_session(engine)
       try:
           # ... work ...
           session.commit()
       finally:
           session.close()
           engine.dispose()
   ```

### Migrations

- **Every schema change requires an Alembic migration**
- PostGIS extension is NOT managed by Alembic (requires superuser)
- Migrations are in `alembic/versions/`

```bash
# Create new migration
vote-match db-migrate "Add county_boundary table"

# Apply migrations
vote-match db-upgrade head

# Rollback one migration
vote-match db-downgrade -1
```

**Important**: Review that `upgrade()` and `downgrade()` are inverses. Flag destructive operations (DROP COLUMN) that don't migrate data first.

---

## Architecture & File Structure

### Core Modules

```
src/vote_match/
├── cli.py              # Typer commands (thin wrappers)
├── processing.py       # Business logic (geocoding, district comparison, map generation)
├── models.py           # SQLAlchemy models + DISTRICT_TYPES mapping
├── config.py           # Pydantic settings from environment
├── database.py         # Engine/session creation, init_database()
├── csv_reader.py       # Load voter CSV files
├── migrations.py       # Alembic wrapper functions
├── logging.py          # Loguru configuration
├── r2_storage.py       # Cloudflare R2 upload
├── usps_validator.py   # USPS address validation
├── county_linking.py   # County FIPS code utilities
├── geocoder.py         # Legacy Census-only geocoder (deprecated)
├── geocoding/
│   ├── base.py         # GeocodeService ABC, StandardGeocodeResult
│   ├── registry.py     # GeocodeServiceRegistry for service discovery
│   └── services/       # One module per geocoding provider
│       ├── census.py
│       ├── nominatim.py
│       ├── mapbox.py
│       ├── geocodio.py
│       ├── google_maps.py
│       └── photon.py
```

### Design Patterns

#### 1. CLI Commands (cli.py)

- **Thin wrappers** around processing functions
- Handle argument parsing, settings loading, progress bars
- Always dispose of database resources

```python
@app.command()
def geocode(service: str = "census"):
    settings = get_settings()
    engine = get_engine(settings)
    try:
        geocode_voters(engine, settings, service)
    finally:
        engine.dispose()
```

#### 2. Business Logic (processing.py)

- Core functions for geocoding, district comparison, map generation
- Accept `Session` or `Engine` as parameter (not global)
- Return structured data (not printing directly)

```python
def compare_voter_districts(
    session: Session,
    district_type: str,
    batch_size: int = 1000
) -> tuple[int, int, int]:
    """Compare voters against spatial boundaries.
    
    Returns:
        (total_compared, matched, mismatched)
    """
```

#### 3. Geocoding Services (geocoding/)

All services inherit from `GeocodeService` ABC and register via decorator:

```python
from vote_match.geocoding.base import GeocodeService, StandardGeocodeResult
from vote_match.geocoding.registry import GeocodeServiceRegistry

@GeocodeServiceRegistry.register
class MyService(GeocodeService):
    @property
    def service_name(self) -> str:
        return "myservice"
    
    @property
    def service_type(self) -> GeocodeServiceType:
        return GeocodeServiceType.BATCH  # or INDIVIDUAL
    
    @property
    def requires_api_key(self) -> bool:
        return True
    
    def prepare_addresses(self, voters: list[Voter]) -> Any:
        """Format addresses for API."""
        ...
    
    def submit_request(self, prepared_data: Any) -> Any:
        """Call external API."""
        ...
    
    def parse_response(self, response: Any, voters: list[Voter]) -> list[StandardGeocodeResult]:
        """Parse API response into standard format."""
        ...
```

**Key points**:
- Return `StandardGeocodeResult` with `GeocodeQuality` enum (exact/interpolated/approximate/no_match/failed)
- Store raw API response in `raw_response` field for debugging
- Handle rate limits via `config.rate_limit_delay`
- Process in batches via `config.batch_size`

---

## Configuration Management

### Pydantic Settings (config.py)

All configuration comes from environment variables or `.env` file:

```python
from pydantic import Field
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str = Field(default="postgresql+psycopg://...")
    default_geocode_service: str = Field(default="census")
    
    # Nested service configs
    geocode_services: GeocodeServicesConfig = Field(default_factory=GeocodeServicesConfig)
    
    model_config = SettingsConfigDict(env_file=".env", env_nested_delimiter="__")

# Access via singleton
settings = get_settings()
api_key = settings.geocode_services.mapbox.api_key
```

### Environment Variables

```bash
# Database
DATABASE_URL=postgresql+psycopg://user:pass@localhost:5432/vote_match

# Geocoding
DEFAULT_GEOCODE_SERVICE=census
GEOCODE_SERVICES__MAPBOX__API_KEY=pk.ey...
GEOCODE_SERVICES__GEOCODIO__API_KEY=abc123...
GEOCODE_SERVICES__NOMINATIM__EMAIL=user@example.com

# USPS
USPS_CLIENT_ID=abc123
USPS_CLIENT_SECRET=xyz789

# Cloudflare R2
R2_ENABLED=true
R2_ENDPOINT_URL=https://abc.r2.cloudflarestorage.com
R2_ACCESS_KEY_ID=xxx
R2_SECRET_ACCESS_KEY=yyy
R2_BUCKET_NAME=vote-match-maps
R2_PUBLIC_URL=https://maps.example.com
```

**Critical**: Never commit `.env`, `sample.csv`, or voter data files to git.

---

## Security & Privacy

### PII Protection

- **Voter data files** (`*.csv`) must be in `.gitignore`
- `sample.csv` is for testing only (sanitized data)
- CLI has `--redact-pii` flag for public maps (removes names/addresses)

### Secrets Management

- **API keys** come from environment (never hardcoded)
- Use `Field(default="")` for optional keys
- Check `config.enabled` before using service

### SQL Injection Prevention

```python
# ✅ CORRECT: Use bind parameters
session.execute(text("SELECT * FROM voters WHERE county = :county"), {"county": county})

# ❌ WRONG: F-string interpolation
session.execute(text(f"SELECT * FROM voters WHERE county = '{county}'"))
```

---

## Testing

### Framework

- **pytest** with coverage tracking (`pytest-cov`)
- Tests in `tests/` directory
- Fixtures in `tests/conftest.py`

### Running Tests

```bash
# All tests with coverage
uv run pytest

# Specific test file
uv run pytest tests/test_processing.py

# With verbose output
uv run pytest -v

# Coverage report in HTML (htmlcov/index.html)
uv run pytest --cov-report=html
```

### Test Database

Tests use a separate database (`vote_match_test`):

```python
@pytest.fixture
def test_settings() -> Settings:
    return Settings(
        database_url="postgresql+psycopg://test:test@localhost:5432/vote_match_test",
        log_level="DEBUG",
    )
```

### Test Strategy

1. **Unit tests** for processing functions (normalize_district_id, address building)
2. **Integration tests** for database operations (loading CSV, geocoding)
3. **Mock external APIs** (Census, USPS) to avoid rate limits

**Important**: Flag new processing functions that lack tests.

---

## Common CLI Commands

### Database Initialization

```bash
# Create schema and run migrations
vote-match init-db

# Drop existing tables and start fresh (DESTRUCTIVE)
vote-match init-db --drop
```

### Loading Data

```bash
# Load voter CSV
vote-match load-csv voters.csv

# Load with progress bar
vote-match load-csv voters.csv --batch-size 1000
```

### Geocoding

```bash
# List available services
vote-match geocode --service list

# Geocode with Census (default)
vote-match geocode

# Geocode with alternative service (processes only no_match records)
vote-match geocode --service nominatim

# Sync best results to Voter.geom (REQUIRED for QGIS)
vote-match sync-geocode

# Force re-sync all voters
vote-match sync-geocode --force
```

**Cascading strategy**:
1. Census geocodes all ungeocoded voters
2. Nominatim/Mapbox/etc. geocode only `no_match` records
3. `sync-geocode` selects best result (exact > interpolated > approximate) across all services

### District Comparison

```bash
# Import district boundaries
vote-match import-geojson districts.geojson county_commission

# List available district types
vote-match import-geojson list

# Compare voters against boundaries
vote-match compare-districts county_commission

# Legacy command (backward compatible)
vote-match compare-districts --legacy
```

### Map Generation

```bash
# Generate map of mismatches
vote-match generate-map --mismatch-only --district-type county_commission

# Upload to Cloudflare R2
vote-match generate-map --upload-to-r2

# Redact PII for public sharing
vote-match generate-map --redact-pii
```

---

## District Types & DISTRICT_TYPES Mapping

The `DISTRICT_TYPES` dict in `models.py` maps district type **keys** to Voter model **column names**:

```python
DISTRICT_TYPES = {
    "county_commission": "county_commission_district",
    "congressional": "congressional_district",
    "state_senate": "state_senate_district",
    # ... 15 types total
}
```

**Usage**:

```python
# Get voter's registered district for a type
district_type = "county_commission"
column_name = DISTRICT_TYPES[district_type]  # "county_commission_district"
voter_district = getattr(voter, column_name)  # voter.county_commission_district

# Compare against spatial assignment
spatial_district = assignment.spatially_assigned_district_id
is_mismatch = normalize_district_id(voter_district) != normalize_district_id(spatial_district)
```

**Important**: Always use `DISTRICT_TYPES` keys, never hardcode column names.

---

## Common Mistakes to Flag

### 1. Wrong Column Types

```python
# ❌ WRONG: Loses leading zeros
district = Column(Integer)

# ✅ CORRECT: Preserves "026" as "026"
district = Column(String)
```

### 2. Using Legacy Models

```python
# ❌ WRONG: Deprecated single-purpose model
from vote_match.models import CountyCommissionDistrict

# ✅ CORRECT: Generic multi-purpose model
from vote_match.models import DistrictBoundary
```

### 3. Not Closing Database Resources

```python
# ❌ WRONG: Resource leak
@app.command()
def my_command():
    engine = get_engine(settings)
    session = get_session(engine)
    # ... work ...
    return  # LEAK!

# ✅ CORRECT: Always close
@app.command()
def my_command():
    engine = get_engine(settings)
    session = get_session(engine)
    try:
        # ... work ...
        session.commit()
    finally:
        session.close()
        engine.dispose()
```

### 4. Forgetting session.commit()

```python
# ❌ WRONG: Changes not saved
session.bulk_save_objects(voters)
# Missing commit!

# ✅ CORRECT: Explicitly commit
session.bulk_save_objects(voters)
session.commit()
```

### 5. Wrong CRS

```python
# ❌ WRONG: UTM projection (wrong CRS)
gdf = gpd.read_file("districts.shp")  # May be EPSG:32617
# Store directly (WRONG!)

# ✅ CORRECT: Always reproject to EPSG:4326
gdf = gpd.read_file("districts.shp")
gdf = gdf.to_crs("EPSG:4326")  # WGS84
# Now store
```

### 6. Hardcoding District Type Strings

```python
# ❌ WRONG: Hardcoded column name
voter_district = voter.county_commission_district

# ✅ CORRECT: Use DISTRICT_TYPES mapping
column_name = DISTRICT_TYPES["county_commission"]
voter_district = getattr(voter, column_name)
```

### 7. Omitting --legacy Support

When modifying `import-geojson` or `compare-districts`, preserve backward compatibility:

```python
# ✅ CORRECT: Support both old and new syntax
if legacy:
    # Old behavior (county_commission hardcoded)
    district_type = "county_commission"
else:
    # New behavior (any district type)
    if district_type is None:
        raise ValueError("--district-type required (or use --legacy)")
```

### 8. Not Normalizing District IDs

```python
# ❌ WRONG: "026" != "26" (false mismatch)
if voter.district == spatial_district:
    ...

# ✅ CORRECT: Normalize before comparison
if normalize_district_id(voter.district) == normalize_district_id(spatial_district):
    ...
```

---

## Logging

Uses **loguru** for structured logging:

```python
from loguru import logger

logger.info("Processing {} voters", len(voters))
logger.debug("Geocoding voter {id}", id=voter.voter_registration_number)
logger.warning("API rate limit hit, sleeping {}s", delay)
logger.error("Geocoding failed: {}", error)
```

**Configuration**:
- `LOG_LEVEL` environment variable (DEBUG/INFO/WARNING/ERROR)
- `LOG_FILE` path (default: `logs/vote-match.log`)
- `--verbose` CLI flag enables DEBUG level

---

## Cloudflare R2 Integration

Vote Match can upload generated maps to Cloudflare R2 (S3-compatible storage):

```bash
# Upload map to R2
vote-match generate-map --upload-to-r2
```

**Configuration** (`.env`):

```bash
R2_ENABLED=true
R2_ENDPOINT_URL=https://<account-id>.r2.cloudflarestorage.com
R2_ACCESS_KEY_ID=xxx
R2_SECRET_ACCESS_KEY=yyy
R2_BUCKET_NAME=vote-match-maps
R2_PUBLIC_URL=https://maps.example.com
R2_FOLDER=secret-folder-path  # Optional
```

**Implementation** (`r2_storage.py`):

```python
import boto3

def upload_to_r2(local_path: Path, settings: Settings) -> str:
    """Upload file to R2 and return public URL."""
    s3 = boto3.client(
        "s3",
        endpoint_url=settings.r2_endpoint_url,
        aws_access_key_id=settings.r2_access_key_id,
        aws_secret_access_key=settings.r2_secret_access_key,
    )
    
    key = f"{settings.r2_folder}/{local_path.name}" if settings.r2_folder else local_path.name
    s3.upload_file(str(local_path), settings.r2_bucket_name, key, ExtraArgs={"ContentType": "text/html"})
    
    return f"{settings.r2_public_url}/{key}"
```

---

## USPS Address Validation

Vote Match integrates with USPS API for address standardization:

```bash
# Validate addresses via USPS
vote-match validate-usps
```

**Configuration**:

```bash
USPS_CLIENT_ID=abc123
USPS_CLIENT_SECRET=xyz789
USPS_BASE_URL=https://apis.usps.com/addresses/v3  # Production
# USPS_BASE_URL=https://apis-tem.usps.com/addresses/v3  # Test environment
```

**Fields added to Voter model**:

- `usps_validation_status`
- `usps_validated_street_address`
- `usps_validated_city`
- `usps_validated_state`
- `usps_validated_zipcode`
- `usps_validated_zipplus4`
- `usps_dpv_confirmation` (Delivery Point Validation)
- `usps_business` (Y/N)
- `usps_vacant` (Y/N)

---

## Map Generation Features

Generated Leaflet maps support:

- **Marker clustering** (configurable via `MAP_CLUSTER_*` settings)
- **Spiderfying** (spread out overlapping markers)
- **Dynamic cluster radius** (zoom-dependent)
- **PII redaction** (`--redact-pii` flag)
- **District filtering** (`--district-type`, `--include-districts`)
- **Mismatch highlighting** (red = mismatch, green = match)
- **Popup tooltips** (voter info, district assignments)

**Template**: `web/template.html` (Jinja2)

---

## Performance Considerations

1. **Batch processing**: Default 1000 records per commit
2. **Index usage**: Indexes on `geocode_status`, `county`, `county_precinct`, `district_mismatch`
3. **Spatial indexes**: PostGIS automatically creates spatial indexes on `Geometry` columns
4. **Geocoding rate limits**: Respect service `rate_limit_delay` (especially Nominatim: 1 req/sec)
5. **Progress tracking**: Use Rich progress bars for long operations

---

## Git & Version Control

### .gitignore

**Must ignore**:
- `*.csv` (voter data files)
- `.env` (secrets)
- `*.html` (generated maps)
- `logs/` (log files)
- `htmlcov/` (coverage reports)
- `__pycache__/`

**Exception**: `sample.csv` (sanitized test data)

### Commit Messages

Use conventional commit format:

```
feat: add Photon geocoding service
fix: normalize district IDs before comparison
docs: update README with sync-geocode requirement
refactor: migrate from CountyCommissionDistrict to DistrictBoundary
test: add unit tests for normalize_district_id
```

---

## Key Dependencies

### Production

- **SQLAlchemy 2.0+**: Database ORM
- **GeoAlchemy2**: PostGIS integration
- **Alembic**: Schema migrations
- **Typer**: CLI framework
- **Rich**: Terminal styling
- **httpx**: HTTP client (async support)
- **pandas**: CSV processing
- **geopandas**: Geospatial data handling
- **shapely**: Geometry operations
- **loguru**: Structured logging
- **Pydantic**: Settings validation
- **boto3**: S3/R2 uploads
- **Jinja2**: HTML templating

### Development

- **pytest**: Testing framework
- **pytest-cov**: Coverage tracking
- **ruff**: Linting and formatting

---

## Workflow Summary

1. **Initialize database**: `vote-match init-db`
2. **Load voter CSV**: `vote-match load-csv voters.csv`
3. **Geocode addresses**:
   - `vote-match geocode --service census`
   - `vote-match geocode --service nominatim` (for no_match records)
   - `vote-match sync-geocode` (REQUIRED for QGIS)
4. **Validate addresses**: `vote-match validate-usps` (optional)
5. **Import district boundaries**: `vote-match import-geojson districts.geojson county_commission`
6. **Compare districts**: `vote-match compare-districts county_commission`
7. **Generate map**: `vote-match generate-map --mismatch-only --upload-to-r2`
8. **Check status**: `vote-match status`

---

## Additional Resources

- **Project docs**: `docs/` directory
  - `PROJECT_SUMMARY.md` - Full project background and findings
  - `CLI_README_BEST_PRACTICES.md` - CLI design guidelines
  - `GA_Maps.md` - Georgia GIS data sources
  - `nominatim.md` - Nominatim geocoding notes
- **Data files**: `data/` directory (shapefiles, CSV metadata)
- **Web maps**: `web/` directory (Leaflet templates, static assets)
- **Alembic migrations**: `alembic/versions/`

---

## When in Doubt

1. **Check DISTRICT_TYPES** before adding district logic
2. **Use String columns** for all ID fields (never Integer)
3. **Reproject to EPSG:4326** before storing geometry
4. **Close sessions and engines** in CLI commands
5. **Test with sample data** (not full voter roll)
6. **Run pytest before committing**
7. **Never commit PII or secrets**

---

## Contact & Contribution

- **Author**: Kerry Hatcher
- **License**: See LICENSE file
- **Repository**: GitHub (kerryhatcher/vote-match)

For questions about Georgia voter data, contact the Secretary of State's office: <https://sos.ga.gov/>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kerryhatcher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
