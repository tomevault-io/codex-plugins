# Project Guidelines

## Overview

easyquotation is a Python library for fetching real-time stock quotations from Chinese financial data sources (Sina, Tencent, Jisilu, Bank of China). The primary runtime dependency is `requests`.

## Build and Test

```bash
make init              # Set up Pipenv + dev deps + pre-commit
pipenv shell           # Enter virtual environment
pytest                 # Run tests
pytest --cov           # Run tests with coverage
mypy easyquotation     # Type check (ignore_missing_imports enabled)
```

Formatting: `black -l 79`. Import sorting: `isort`.

## Architecture

**Two class patterns coexist:**

1. **`BaseQuotation` subclasses** (template method): `Sina`, `Tencent`, `HKQuote`, `DayKline` — share stock code loading, batched threaded HTTP fetching, and a common `real()`/`market_snapshot()` API. Subclasses override `stock_api` (URL), `format_response_data()` (parsing), and optionally `_gen_stock_prefix()`, `_get_headers()`, `max_num`.

2. **Standalone classes**: `Jsl` and `Boc` have completely different interfaces (`jsl.etfindex()`, `jsl.cb()`, `boc.get_exchange_rate()`). They do NOT inherit from `BaseQuotation` and do NOT implement `real()` or `market_snapshot()`.

**Factory entry point:** `easyquotation.use(source)` in `api.py` maps source strings to classes.

**Stock codes:** Loaded from `stock_codes.conf` (a JSON file with `.conf` extension). `helpers.py` manages code loading, updating from remote, and determining market prefix (`sh`/`sz`/`bj`).

## Conventions

- `# coding:utf8` header on every source file (legacy convention, maintain it)
- Sina/HKQuote use English dict keys; Tencent uses Chinese keys (`"涨跌(%)"`, `"流通市值"`) — be source-aware
- Response parsing is regex-based (Sina) or split-based (Tencent) — fragile to API format changes
- `requests.Session` is reused in `BaseQuotation`; `Jsl` creates ad-hoc sessions; `Boc` uses plain `requests.get`
- Concurrency via `multiprocessing.pool.ThreadPool` for batched HTTP fetches

## Testing

- Framework: `unittest` classes run with `pytest`
- Unit tests pass mock data strings directly to `format_response_data()`, bypassing HTTP
- `TestEasyquotation` contains integration tests that hit live Sina/Tencent APIs — these fail offline
- No tests exist for `Jsl`, `Boc`, or `Tencent`

## Pitfalls

- **`build/` directory is stale** — never edit files there; authoritative source is `easyquotation/`
- **No HTTP timeouts or retries** — requests can hang; adding timeouts changes behavior
- **Jsl `__cb_url` mutation** — `self.__cb_url.format(...)` consumes the `{ctime:d}` placeholder on first call, breaking reuse on the same instance
- **Live API tests** — network-dependent tests may fail; don't treat as code bugs
- **`stock_codes.conf` loaded at init** — if missing/corrupt, all `BaseQuotation` subclasses fail at construction

---
> Source: [shidenggui/easyquotation](https://github.com/shidenggui/easyquotation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-29 -->
