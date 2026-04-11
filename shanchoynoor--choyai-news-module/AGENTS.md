# ChoyNewsBot AI Coding Agent Instructions

## Project Overview
ChoyNewsBot is an AI-powered Telegram bot delivering real-time breaking news, crypto intelligence, and weather data across multiple daily time slots (8am, 1pm, 7pm, 11pm BDT). It features zero-duplicate news delivery using SQLite deduplication and DeepSeek AI analysis.

## Core Architecture

### Module Structure (Post-v2.0 Reorganization)
- `core/` - Main business logic (news fetching, AI analysis, bot commands)
- `data_modules/` - SQLite database models (subscriptions, logs, deduplication)
- `services/` - External service integrations (Telegram API, scheduling)
- `api/` - API handlers (telegram.py for message processing)
- `utils/` - Shared utilities (config, logging, time handling)
- `config/` - Environment-specific configurations (dev/prod/base)

### Key Data Flows
1. **News Pipeline**: RSS sources → `fetch_breaking_news_rss()` → SQLite dedup → AI analysis → Telegram formatting
2. **Crypto Intelligence**: CoinGecko API → DeepSeek AI analysis → Formatted market updates
3. **User Management**: Telegram updates → `data_modules/models.py` → SQLite (subscriptions, logs, timezones)

## Critical Development Patterns

### News Deduplication System
- Uses `core/advanced_news_fetcher.py` with SQLite hash tracking
- News items hashed by `title + source` to prevent repeats across time slots
- Always call `mark_news_as_sent()` after delivering news
- Database: `data/news_history.db` (auto-created)

### RSS Source Management
```python
# News sources organized by category with reliability scoring
bd_sources = {"The Daily Star": "https://thedailystar.net/rss.xml", ...}
global_sources = {"BBC": "https://feeds.bbci.co.uk/news/rss.xml", ...}
# Each category fetches exactly 5 items with smart fallbacks
```

### AI Integration Points
- DeepSeek API for crypto analysis via `DEEPSEEK_API` env var
- Sentiment analysis in `analyze_news_item()` 
- Market predictions in `fetch_crypto_market_with_ai()`

### Configuration Management
- Multi-tier config: `config/base_config.py` → `config/dev_config.py`/`config/prod_config.py`
- Environment variables loaded via `utils/config.py`
- Database paths auto-constructed relative to project root

## Essential Commands & Workflows

### Development Setup
```bash
make install-dev          # Install all dependencies including dev tools
make test                 # Run pytest test suite
make lint                 # Run flake8 + pylint
make run                  # Start bot in development mode
```

### Production Deployment
```bash
make docker-build         # Build production container
make docker-run           # Run with docker-compose
make logs                 # View application logs
```

### Database Operations
```python
# Always initialize DBs before operations
from data_modules.models import init_user_subscriptions_db, init_user_logs_db
init_user_subscriptions_db()
init_user_logs_db()

# User subscription pattern
users = get_users_for_scheduled_times(hour=8, minute=0)  # Get 8am subscribers
```

## Key File Dependencies

### Critical Path Files
- `bin/choynews` - Main entry point, handles both interactive bot and scheduled news
- `core/advanced_news_fetcher.py` - Primary news engine with deduplication
- `services/bot_service.py` - Telegram message handling and command routing
- `data_modules/models.py` - All database operations and user management

### Configuration Chain
- `.env` → `utils/config.py` → `config/base_config.py` → app modules
- Always validate config with `Config.validate()` before operations

### API Integration Points
- Telegram: `api/telegram.py` handles all bot messaging
- CoinGecko: Embedded in `core/advanced_news_fetcher.py` for crypto data
- DeepSeek: Rate-limited requests in `_rate_limited_post()`
- Weather: `get_dhaka_weather()` via WeatherAPI
- Calendar: `get_bd_holidays()` via Calendarific

## Common Debugging Patterns

### News Fetching Issues
```python
# Check RSS feed status
logger.info(f"Fetched {len(entries)} entries from {successful_sources}/{len(sources)} sources")
# Verify deduplication
news_hash = get_news_hash(title, source_name)
# Check time parsing
time_ago = get_hours_ago(published_time_str)
```

### Database Problems
- SQLite DBs auto-create in `data/` directory
- Check `USER_SUBSCRIPTIONS_DB` and `USER_LOGS_DB` paths in logs
- Use `cleanup_old_news_history(days_back=7)` to prevent bloat

### AI/API Failures
- All external APIs have fallback handling and rate limiting
- Check environment variables: `DEEPSEEK_API`, `WEATHERAPI_KEY`, etc.
- Crypto data caches for 2-5 minutes to prevent API abuse

## Testing & Quality Assurance

### Test Structure
- Unit tests in `tests/unit/` for core functions
- Integration tests in `tests/integration/` for API interactions
- Sample data in `tests/fixtures/sample_data.py`

### Code Quality
- Use `make lint` before commits (flake8 + pylint)
- Follow existing patterns for error handling and logging
- All database operations should have try/catch with proper logging

## Docker & Deployment

### Container Architecture
- Multi-stage build in `Dockerfile.production`: base → development → production
- Development target includes dev dependencies and debugger support
- Production target is optimized with minimal dependencies
- Health checks via `Config().validate()`
- Volume mounts: `./data:/app/data`, `./logs:/app/logs`
- Redis and PostgreSQL services in `docker-compose.yml`

### Development vs Production
- Development: `docker-compose.dev.yml` with live code reload and debugger ports
- Production: `docker-compose.yml` with optimized image and restart policies
- Use `Dockerfile.production` for both environments with different targets

### Environment Variables
Required: `TELEGRAM_TOKEN`, `DEEPSEEK_API`  
Optional: `WEATHERAPI_KEY`, `CALENDARIFIC_API_KEY`, `TWELVE_DATA_API_KEY`

When modifying this codebase, always maintain the 5-item guarantee per news category, preserve the deduplication system, and ensure proper error handling for all external API calls.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shanchoynoor)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shanchoynoor)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
