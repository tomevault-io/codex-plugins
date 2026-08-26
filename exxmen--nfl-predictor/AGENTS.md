# NFL Playoff Predictor - AI Coding Instructions

## Project Overview

Monte Carlo simulation engine for NFL playoff probability predictions. Uses EPA (Expected Points Added) analytics, injury adjustments, momentum tracking, and full NFL tiebreaker rules.

## Architecture

```
nfl_predictor/
├── cli.py          # Entry point: `nfl-predict` command
├── scheduler.py    # Entry point: `nfl-scheduled` for CI/cron
├── simulation.py   # Core Monte Carlo engine with EPA-Poisson scoring (945 lines)
├── tiebreakers.py  # NFL tiebreaker rules + Game/TeamStats dataclasses (1003 lines)
├── scraper.py      # Pro-Football-Reference scraper via browser-use
├── epa.py          # EPA data loader from nfl_data_py
├── injuries.py     # ESPN scraper + nfl_data_py fallback for injuries
├── player_impact.py# Position-based injury impact calculation
├── intangibles.py  # Rest days, travel, turnover luck adjustments
├── config.py       # Season/week determination, cache validation
└── backtest.py     # Historical validation against past seasons
```

## Data Flow

1. **Scrape standings** → `scraper.scrape_pfr_standings()` (HTTP to Pro-Football-Reference)
2. **Fetch schedule** → `scraper.scrape_pfr_schedule_simple()` (async, uses browser-use)
3. **Load EPA** → `epa.load_team_epa()` from `nfl_data_py` library
4. **Load injuries** → `injuries.scrape_espn_injuries()` or `nfl_data_py` fallback
5. **Calculate impacts** → `player_impact.get_all_team_impacts()` (snap count weighted)
6. **Run simulation** → `simulation.run_advanced_simulation()` (Poisson-distributed scores)
7. **Apply tiebreakers** → `tiebreakers.NFLTiebreaker` (all 12 division + 11 wild card rules)

## Key Commands

```bash
# Development
uv run nfl-predict                    # Interactive mode (10K sims)
uv run nfl-predict -n 50000           # More simulations
uv run nfl-predict --intangibles      # Enable rest/travel/weather factors

# Production (saves to results/)
uv run nfl-scheduled --simulations 100000

# Testing the model
uv run python -c "from nfl_predictor.backtest import run_backtest; run_backtest(2024)"
```

## Conventions

### Caching Strategy
- Cache files live in `cache/` with week-based invalidation
- Pattern: `{data_type}_{season}_meta.json` stores `{"season": int, "week": int}`
- Validate via `config.is_cache_valid_for_week()` - cache expires when new NFL week starts

### Team Name Handling
- Internal: Full names (`"Kansas City Chiefs"`)
- nfl_data_py: Abbreviations (`"KC"`)
- Normalization maps in `scraper.TEAM_NAME_ALIASES` and `injuries.py`

### Game Representation
```python
# tiebreakers.py defines the Game dataclass
Game(week=1, home_team="Buffalo Bills", away_team="Miami Dolphins",
     home_score=24, away_score=17, completed=True)
```

### Position Impact Weights (player_impact.py)
- QB = 1.0, Edge/CB = 0.45-0.50, OL = 0.35-0.55, Others = 0.15-0.40
- Injury status: Out=100%, Doubtful=85%, Questionable=40%, IR/PUP=0% (already in EPA)

## Important Patterns

### Season Configuration (config.py)
- `NFL_WEEK_STARTS` dict must be updated annually with Thursday dates
- Week logic: `get_current_nfl_week()` returns 1-18 based on current date
- Jan/Feb → previous year's season

### Error Handling
- Scraping failures return empty dicts/lists, not exceptions
- Optional modules (EPA, intangibles) use try/except imports with `*_AVAILABLE` flags
- CLI degrades gracefully: `--no-injuries`, `--no-momentum` flags

### Adding New Intangibles
1. Add field to `IntangiblesConfig` dataclass in `intangibles.py`
2. Implement calculation in `IntangiblesCalculator.calculate_game_adjustments()`
3. Wire to CLI via new `--no-{factor}` flag in `cli.py`

## CI/CD

- GitHub Actions runs Tue/Fri at 10:00 UTC via `.github/workflows/nfl-predictions.yml`
- Results published to Gist (requires `GIST_ID` and `GIST_TOKEN` secrets)
- Output formats: JSON, Markdown, and plain text in `results/`

---
> Source: [exxmen/nfl-predictor](https://github.com/exxmen/nfl-predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
