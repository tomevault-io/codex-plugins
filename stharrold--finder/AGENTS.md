# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Automated daily search across online marketplaces to locate specific items. Supports multiple search profiles:

**Ring Search** (config.yaml): Lost antique ring ("The Giulia Ring" - 10K yellow gold with amethyst and seed pearls, size 7, lost in Indianapolis).

**Bike Search** (bike_config.yaml): Trek Allant+ 7S electric bike (Class 3, 625Wh battery, range extender, Large frame, within 300mi of Indianapolis).

**Search Coverage**:
- **Fixed Adapters**: ShopGoodwill, eBay, Etsy, Craigslist (24 regions/300mi), Ruby Lane, Mercari, Poshmark, Pinkbike, Trek Red Barn Refresh
- **Adaptive Discovery**: Any marketplace via DuckDuckGo + Facebook Marketplace, OfferUp, Nextdoor

**Configuration**: Edit `config.yaml` (ring) or `bike_config.yaml` (bike) to customize search keywords, scoring weights, marketplace priorities, and discovery settings.

## Development Commands

```bash
# Install dependencies
uv sync --extra dev
uv run playwright install chromium

# Run CLI (note: global options before subcommand)
# Ring search
uv run ring-search -c config.yaml run              # Daily ring search
uv run ring-search -c config.yaml run --headed     # With visible browser
uv run ring-search -c config.yaml run --adaptive   # With adaptive discovery

# Bike search (Trek Allant+ 7S)
uv run ring-search -c bike_config.yaml run         # Daily bike search
uv run ring-search -c bike_config.yaml run --headed  # With visible browser

# Common commands
uv run ring-search -c config.yaml check-urls urls.txt  # Check specific URLs
uv run ring-search report                          # View most recent summary

# Testing
uv run pytest                    # Run all tests
uv run pytest tests/test_scoring.py  # Run single test file
uv run pytest -k "test_high"     # Run tests matching pattern
uv run pytest --cov=src          # With coverage

# Linting and type checking
uv run ruff check src/           # Lint
uv run ruff format src/          # Format
uv run mypy src/                 # Type check
```

## Architecture

```
SearchOrchestrator (src/ring_search.py)        # Ring search orchestrator
BikeSearchOrchestrator (src/bike_search.py)    # Bike search orchestrator
├── MarketplaceAdapter (src/adapters/base.py) - Abstract interface
│   ├── ShopGoodwillAdapter, EbayAdapter, EtsyAdapter, CraigslistAdapter
│   ├── RubyLaneAdapter, MercariAdapter, PoshmarkAdapter
│   ├── PinkbikeAdapter, TrekRedBarnAdapter    # Bike-specific adapters
├── SearchDiscovery (src/discovery/base.py) - Search engine discovery
│   ├── GoogleDiscovery, DuckDuckGoDiscovery
│   └── MarketplaceFilter - URL filtering and prioritization
├── AdaptiveExtractor (src/extractors/base.py) - Universal listing extraction
│   ├── StructuredDataExtractor - JSON-LD, OpenGraph, microdata
│   ├── LegacyAdapterBridge - Routes to existing adapters
│   └── GenericListingExtractor - Heuristic fallback
├── RelevanceScorer (src/scoring.py) - Configurable weights for ring attributes
├── BikeRelevanceScorer (src/bike_scoring.py) - Bike-specific scoring
├── ScreenshotCapture (src/capture.py) - Full-page screenshots via Playwright
├── DedupManager (src/dedup.py) - Persistent URL tracking
└── SearchLogger (src/logger.py) - JSON logs and markdown summaries
```

**Adding a new marketplace**: Subclass `MarketplaceAdapter`, implement `search()` and `get_listing_details()`, register in `src/adapters/__init__.py` and add to `ADAPTER_MAP`.

**Adaptive mode**: Enable with `--adaptive` flag or set `discovery.enabled: true` in config.yaml. Discovers listings via search engines and extracts data using structured markup or heuristics.

**Bike scoring weights** (from bike_config.yaml):
- Model match (Allant+ 7S): 40%
- Class 3 confirmation: 20%
- 625Wh battery: 20%
- Range extender: 15%
- Large frame: 5%

## Data Directory Structure

```
finder/
└── YYYYMMDD_type_description/
    ├── input/    # Source materials (PDFs, images)
    └── output/   # Processed/generated files
```

### Naming Conventions

**Item folders**: `YYYYMMDD_type_description` (e.g., `20251201_ring_vintage-amethyst-pearl-gold`)

**Input files**:
- Product PDFs: Full product page title from source
- Product images: `ProductName_view.ext` where view is `top`, `side`, `hand_zi`, `hand_zo`
- Order confirmations: `YYYYMMDD_source - subject.pdf`

## Branch Structure

```
main                           ← Production (tagged vX.Y.Z)
  ↑
develop                        ← Integration branch
  ↑
contrib/<gh-user>             ← Personal contribution branch
```

### Branch Protection Policy

**Protected branches** (merge via PR only):
- `main` - Production branch
- `develop` - Integration branch

**Editable branches** (direct commits allowed):
- `contrib/*` - Personal contribution branches

### PR Flow

All changes flow: `contrib/<user>` → `develop` → `main`

## Scheduled Automation

Daily search runs at 8:00 AM via macOS LaunchAgent:
- **Plist**: `~/Library/LaunchAgents/com.stharrold.ring-search.plist`
- **Logs**: `output/logs/launchd.log`
- **Manual trigger**: `launchctl kickstart gui/$(id -u)/com.stharrold.ring-search`

## Critical Guidelines

- **ALWAYS prefer editing existing files** over creating new ones
- **End on editable branch**: All work should end on `contrib/*` (never `develop` or `main`)
- **Follow naming conventions** for item folders and files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stharrold) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
