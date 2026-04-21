# Claude Code Context — CryptoApp

## Developer Preferences

- **Package manager:** `uv` — always use `uv run`, `uv add`, `uv sync` (never pip)
- **No emojis:** Never use emoji characters anywhere — not in JS strings, HTML templates, Python log messages, or route responses. Use plain text instead (e.g. `STOPPED` not `🛑 STOPPED`, `Auto-approving` not `🤖 Auto-approving`).
- **Code style:** Keep the project tidy — no redundant files, dead code, or orphaned imports
- **Git workflow:** `dev` branch for work, merge to `main` for releases. Commit + push after every completed change. Descriptive commit messages with bullet-point body.
- **Deployment:** Raspberry Pi 4 — if running locally use `ssh pi "cd ~/CryptoApp && git pull && sudo systemctl restart cryptoapp"`. If already on the Pi, run `git pull && sudo systemctl restart cryptoapp` directly.

## Verification Commands

After making changes, verify with:
- **Syntax check:** `uv run python -m py_compile <file>.py` (also runs automatically via hook on every edit)
- **Import check:** `uv run python -c "import app"` — catches missing deps and circular imports
- **Run app locally:** `uv run python app.py` (dev mode, port 5001)
- **Health check:** `curl -s http://localhost:5001/` — should return 200
- **Lint:** `uv run flake8 ml/ routes/ --max-line-length=120 --ignore=E501`

## Context Compaction

When compacting, ALWAYS preserve:
- The list of files modified in this session and what changed
- Any in-progress task or feature being built
- Current error messages or test failures being debugged
- Trade safety mechanism states (kill switch, budget, approval settings)

## Project Overview

AI-powered low-cap cryptocurrency analysis and automated trading system. Uses a 5-agent Google ADK (Gemini) architecture to discover undervalued coins, analyse them, and execute live trades on Kraken. Runs headlessly on a Raspberry Pi 4 for ~£2.50/month.

**Key flow:** CoinGecko data → 5 AI agents analyse → trading engine proposes → auto-approve or email → Kraken execution → portfolio tracking → sell automation monitors exits.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.13 |
| Web | Flask + Gunicorn (1 worker, 4 gthread) |
| AI Agents | Google ADK + `gemini-2.0-flash` (configurable via `ORCHESTRATOR_MODEL`) |
| Exchange | Kraken via ccxt |
| ML | scikit-learn (training), ONNX Runtime (inference), Q-learning RL |
| Frontend | Vanilla JS + Jinja2 templates + CSS |
| Data | JSON file persistence, optional Redis cache |
| Deploy | systemd + nginx + Let's Encrypt SSL |
| Scheduling | `schedule` library in background threads |

## Directory Structure

```
app.py              — Flask app factory, extension init, scheduler starts
wsgi.py             — Gunicorn WSGI entry (wsgi:app)
gunicorn.conf.py    — Pi-optimised config (1 worker, 120s timeout)
extensions.py       — Flask-Limiter, Flask-CORS (avoids circular imports)

ml/                 — All ML, trading, and agent logic
  scan_loop.py      — Automated scan pipeline (every 12h)
  market_monitor.py — Between-scan price checks at 5, 15, and 30-min intervals
  trading_engine.py — Proposals, budget, approval, execution, kill switch
  exchange_manager.py — Kraken via ccxt, pair cache, FX, orders
  sell_automation.py  — Exit triggers (profit/stop-loss/trailing/agent)
  q_learning.py        — Q-learning RL (buy/skip, reward shaping, ε-greedy)
  portfolio_tracker.py — Holdings, cost basis, unrealised P&L
  portfolio_manager.py — Batch portfolio analysis via ADK
  orchestrator_wrapper.py — Thin ADK adapter for portfolio batch analysis
  training_pipeline.py — RandomForest → ONNX export
  onnx_inference.py    — ONNX Runtime inference
  backtesting.py       — Backtesting framework
  data_pipeline.py     — Data ingestion and feature engineering
  gem_score_tracker.py — Tracks gem scores over time
  error_handling.py    — Shared error handling utilities
  agent_memory.py      — Short/long-term agent context
  scheduler.py         — Weekly retrain + performance reports
  doc_updater.py       — Auto-updates documentation
  agents/official/     — 5 ADK agents + orchestrator + quick_screen

routes/             — Flask Blueprints (coins, health, ml, symbols, trading)
services/           — Shared app state + optional Redis cache
src/core/           — Config, CryptoAnalyzer model, CoinGecko fetcher
src/web/            — Jinja2 templates + static JS/CSS
data/               — Runtime JSON state (gitignored)
deploy/             — systemd unit, nginx config, SSL script
docs/               — Markdown documentation
```

## Key Patterns

- **Singletons:** `get_trading_engine()`, `get_scan_loop()`, `get_exchange_manager()` — module-level globals with lazy init
- **State init:** `services/app_state.py` → `init_all()` called once at startup
- **Error handling:** try/except with `logger.warning()` fallback; graceful degradation if dependencies missing
- **Logging:** Per-module `logger = logging.getLogger(__name__)`
- **Data models:** `@dataclass` for core models, Pydantic `BaseModel` for agent I/O
- **Section separators:** `# ─── Section Name ───────────────────`
- **Auth:** Bearer token (`TRADING_API_KEY`) on all trading POST endpoints
- **Frontend:** Vanilla JS modules in `src/web/static/js/`, no build step

## API Cost Rules

Gemini is the primary cost driver. Every code change that touches agent calls, scan loops, or sell automation MUST be evaluated for API cost impact before merging.

| Guard | Value |
|-------|-------|
| `GEMINI_DAILY_BUDGET_GBP` | £1.00 default — hard ceiling on Gemini spend per day |
| `SCAN_MAX_COINS` | 10 — coins reaching full agent analysis per scan |
| `max_full_analysis` | 3 — max coins analysed by full 5-agent chain per scan |
| `SCAN_COOLDOWN_HOURS` | 1h — minimum gap between any two scans |
| `RECYCLE_MIN_GBP` | £2.00 — minimum sell value to trigger a post-sell recycle scan |

**Rules for new code:**
- Never call an agent inside a hot loop or a path that runs more than once per scan cycle.
- Prefer data already in `coin_data` / `holding` dicts over a fresh API call (e.g. `price_change_7d` for stagnation checks).
- Raise quick-screen thresholds in bear regimes — fewer full analyses saves budget.
- Any new background scan trigger MUST check the scan cooldown and must be bounded by `GEMINI_DAILY_BUDGET_GBP`.
- Log a warning whenever the Gemini budget exceeds 80% — don't silently burn spend.
- New env vars that control agent call frequency must be documented in `.env.example`.

## Safety Mechanisms

| Mechanism | Default |
|-----------|---------|
| Kill switch | Halts all trading instantly |
| Daily budget | `DAILY_TRADE_BUDGET_GBP` (£3 default) |
| Per-trade max | 50% of daily budget |
| Approval threshold | Trades above `APPROVAL_THRESHOLD_GBP` (£50) always require email approval |
| Proposal expiry | 1 hour (proposal auto-cancels if unactioned) |
| Trade cooldown | 60 min between new proposal generation |
| Scan cooldown | 1 hour between scans |
| Max proposals/scan | 3 |
| Conviction threshold | ≥55% (agent) / regime-aware scan: bull=40%, neutral=45%, bear=60% |
| Min hold period | 72h before profit/trailing triggers |
| Tiered exits | Tier 1: 75% profit → sell 33%; Tier 2: 150% → sell 50% of remaining |
| Balance check | Kraken balance verified before every order |
| Email approval | HMAC-signed links; sells always require manual approval |
| Memory guard | systemd MemoryMax=1G, Gunicorn max_requests=500 |
| Rate limiting | 200 req/hour (Flask-Limiter) |
| Audit trail | `data/trades/audit_log.jsonl` + daily scan logs |

## Environment Variables

See `.env.example` for the full list. Key groups:
- **Required:** `GOOGLE_API_KEY` (Gemini); `COINGECKO_API_KEY` optional (free tier works without it)
- **Trading:** `KRAKEN_API_KEY`, `KRAKEN_PRIVATE_KEY`, `TRADING_API_KEY`, `DAILY_TRADE_BUDGET_GBP`
- **Approval:** `BUY_AUTO_APPROVE`, `SELL_REQUIRE_APPROVAL`, `TRADE_NOTIFICATION_EMAIL`, SMTP settings
- **Scan:** `SCAN_ENABLED`, `SCAN_INTERVAL_HOURS`, `SCAN_MAX_COINS`
- **Sell:** `SELL_TIER1_PCT` (75%), `SELL_TIER1_FRACTION` (0.33), `SELL_TIER1_TRAILING_PCT` (20%), `SELL_TIER2_PCT` (150%), `SELL_TIER2_FRACTION` (0.50), `SELL_TIER2_TRAILING_PCT` (15%), `SELL_STOP_LOSS_PCT`, `SELL_TRAILING_STOP_PCT`, `SELL_MIN_HOLD_HOURS`
- **Scan thresholds:** `SCAN_QUICK_SCREEN_BULL` (60%), `SCAN_QUICK_SCREEN_NEUTRAL` (70%), `SCAN_QUICK_SCREEN_BEAR` (72%)

## Domain Knowledge

Load these when working on the relevant area (use `@` to import):
- @.github/instructions/trading.instructions.md — Trading engine, proposals, approval flow, execution
- @.github/instructions/agents.instructions.md — ADK agent architecture, orchestrator, sub-agents
- @.github/instructions/scanning.instructions.md — Scan loop, market monitor, scheduling
- @.github/instructions/deployment.instructions.md — Pi setup, systemd, nginx, Tailscale SSH
- @.github/instructions/frontend.instructions.md — Templates, JS modules, API endpoints, dashboard

Broader architecture context: `docs/architecture/`.

## Skills & Agents

- `/deploy` — Deploy to Pi and verify the service is healthy
- `/backtest` — Run backtesting framework with optional custom thresholds
- `security-reviewer` subagent — Ask: *"use a subagent to review this code for security issues"*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finn6666) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
