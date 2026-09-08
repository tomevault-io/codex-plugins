# Copilot instructions for tigertrade

Purpose: quick, actionable guidance to help an AI coding agent be immediately productive in this repo.

## Big picture 🧭
- Single-script strategy prototype: most logic lives in `tigertrade/tiger1.py`.
- Two main external integrations:
  - `QuoteClient` (market data) — used for `get_future_bars`, `get_future_brief`, etc.
  - `TradeClient` (orders) — used in `place_tiger_order` and simulated in sandbox.
- Runtime mode set by CLI arg: `d` → demo/sandbox (`./openapicfg_dem`), `c` → production (`./openapicfg_com`).
- Main loop: `grid_trading_strategy()` runs repeatedly (currently `time.sleep(20)` between runs).

## How to run / quick checks ▶️
- Start in sandbox: `python tiger1.py d` (uses `openapicfg_dem` props)
- Start for production: `python tiger1.py c` (uses `openapicfg_com` props)
- Use `verify_api_connection()` to validate credentials and connectivity — it prints sample responses and will indicate issues.
- Run unit tests: `pytest -q` (the repo includes `tests/test_kline_data.py` which covers `get_kline_data` DataFrame/list/generator cases).

## Key files & config 🔧
- `tigertrade/tiger1.py` — all core logic (fetching K-lines, indicator calc, risk rules, order placement).
- `openapicfg_dem/` and `openapicfg_com/` — property files that `TigerOpenClientConfig` reads.
- Keep `FUTURE_SYMBOL`, `FUTURE_MULTIPLIER`, and other contract defaults in the script; `get_future_brief_info()` will override when API data is available.

## Important code patterns / conventions 📋
- get_kline_data behavior (important to replicate in tests/mocks):
  - Accepts `symbol` as string or list-like; calls `quote_client.get_future_bars(symbol_list, period_enum, -1, -1, count, None)`.
  - Accepts either:
    - a pandas.DataFrame (must include `time` column), or
    - an iterable of bar-like objects with `.time, .open, .high, .low, .close, .volume` attributes.
  - Converts times using: parse `time`, localize tz-naive timestamps to `UTC`, then convert to `Asia/Shanghai`.
  - Requires at least 10 bars (threshold currently hardcoded). Returns empty DataFrame on failure.
  - Truncates to the most recent `count` rows via `df.tail(count)`.

## QuoteClient K-line API (行情 API) 📡
- Key methods:
  - `QuoteClient.get_future_bars(identifiers, period=BarPeriod.DAY, begin_time=-1, end_time=-1, limit=1000, page_token=None)` — returns a pandas.DataFrame (rows are returned newest-first, i.e., descending by time).
  - `QuoteClient.get_future_bars_by_page(identifier, period=BarPeriod.DAY, begin_time=-1, end_time=-1, total=10000, page_size=1000, time_interval=2)` — use to page through large historical ranges; returns pandas.DataFrame and `next_page_token` in results.
- Behavior & important caveats:
  - Returned timestamps are bar end-times (ms since epoch). The API returns data from `end_time` backward (reverse chronological order).
  - Minute bars may be missing for minutes with no trades — the latest 1-min bar appears only after the first trade in that minute.
  - Default and maximum `limit` is 1000; to fetch longer spans use the by-page API or iterate with `page_token`/`next_page_token`.
  - `end_time` is exclusive; `begin_time` is inclusive when provided. Passing both -1 uses the `limit` to return the most recent bars.
  - Typical columns: `identifier, time, latest_time, open, high, low, close, settlement, volume, open_interest, next_page_token`.
- Testing and mocking tips:
  - Mock both DataFrame and page-based responses; ensure tests cover tz parsing (ms -> pd.to_datetime -> tz localize UTC -> Asia/Shanghai).
  - Simulate missing 1-min bars (e.g., skip a minute) to validate strategy behavior when the latest minute is not present.
  - For pagination, mock `next_page_token` flows and confirm code stops when no token or requested total reached.
- Short example:

```python
# recent bars (most recent N bars)
bars = quote_client.get_future_bars(['SIL2603'], BarPeriod.ONE_MINUTE, -1, -1, 100, None)
# long history (page-based)
bars_long = quote_client.get_future_bars_by_page('SIL2603', end_time=1648526400000, total=5000, page_size=1000)
```

- Logging/messages: prints use Chinese messages and emoji; preserve when editing for consistency with repo tone.

## Order placement specifics ⚙️
- `place_tiger_order()` tries to use SDK classes but falls back to `SimpleNamespace` if unavailable — do not assume SDK types are imported.
- Account lookup prefers `client_config.account` then `client_config.account_id`.
- Uses `RUN_ENV` (`sandbox` vs `production`) to decide whether to actually place orders or simulate a successful response when `trade_client` fails.
- Simulated response sets `order_id = 'SIMULATED'` in sandbox.

## Tests / mocking guidance 🧪
- When unit testing or mocking `QuoteClient.get_future_bars`, use either:
  - pd.DataFrame with columns `['time','open','high','low','close','volume']` where `time` may be tz-aware or tz-naive, or
  - an iterable of `types.SimpleNamespace(time=..., open=..., high=..., low=..., close=..., volume=...)` objects.
- Test the generator case (non-sized iterable) to confirm the code converts to list when `len()` raises `TypeError`.
- Test timezone normalization (tz-naive → localized to UTC → converted to Asia/Shanghai).
- For `place_tiger_order()`, mock `trade_client.place_order` to raise an exception and validate sandbox simulation path.

## Troubleshooting tips 🔍
- If `get_kline_data()` returns empty DataFrame, check whether the API returned fewer than 10 bars or missing required columns.
- Run `verify_api_connection()` to surface credentials/config problems quickly.
- Note the script does not call `load_dotenv()` (though `dotenv` is imported). If using `.env`, call `load_dotenv()` in your changes.

## Good-first edits for an AI agent ✅
1. Add unit tests for `get_kline_data()` covering DataFrame, list of bars, and generator responses.
2. Add a small integration test that runs `verify_api_connection()` in sandbox with mocked `QuoteClient`.
3. Make the `10`-bar threshold a named constant (`MIN_KLINES`) and document it for easier tuning — implemented in `tigertrade/tiger1.py` as `MIN_KLINES`.

## Continuous integration
- A GitHub Actions workflow is included at `.github/workflows/ci.yml` that runs pytest on push and pull requests (installs pytest via pip). Tests may skip if native SDK dependencies are missing; unit tests mock `quote_client` where possible.

---
If anything here is unclear or you'd like the file written in Chinese instead, tell me which sections to expand or translate and I'll iterate. 👍

---
> Source: [chenxi750328ai/agent-jianghu](https://github.com/chenxi750328ai/agent-jianghu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-11 -->
