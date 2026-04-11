# Copilot Instructions

## Project Overview

Customer Intent Scraper is a feedback analytics pipeline for Microsoft 365 products. It scrapes discussions from Microsoft Tech Community and Reddit, runs keyword-based analysis (sentiment, intent, author role, product area, K-Means topic clustering), and surfaces results in a Streamlit dashboard.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt
playwright install chromium   # required for Tech Community spider

# Run the dashboard
streamlit run app.py

# Run scrapers (from sidebar in app, or directly)
scrapy crawl techcommunity
scrapy crawl reddit

# Run analysis over all unanalyzed discussions
python analyze_local.py

# Run tests
pytest tests/

# Run a single test file
pytest tests/test_extract_queries.py -v
```

## Architecture

**Data flow:** Scrapy spiders → `SQLitePipeline` (pipelines.py) → `discussions.db` → `analyze_local.py` (analysis columns added via UPDATE) → Streamlit dashboard (app.py).

**Four DB tables:** `discussions` (main posts with `analysis_*` columns), `replies` (threaded responses, FK → discussions.id), `queries` (extracted questions, FK → discussions.id), `retrievability_results` (TF-IDF scores, FK → queries.id).

**Key modules:**
- `db.py` — pure SQLite data-access layer; no Streamlit imports allowed here
- `analyze_local.py` — keyword dictionaries and K-Means clustering; the production analysis path (not OpenAI)
- `extract_queries.py` — pulls questions out of discussion text; tags each with a `method` field (`title_question`, `content_question`, `title_implicit`)
- `score_retrievability.py` — TF-IDF relevance scoring against local docs in `docs/`
- `customer_intent_scraper/pages/` — scrapy-poet Page Objects for structured extraction
- `app.py` — Streamlit UI; calls `subprocess` to trigger scraper/analyzer from the sidebar

## Key Conventions

**`db.py` rules (enforced by docstring):** No Streamlit imports. Every public function takes an explicit `db_path: str` argument and returns a `pd.DataFrame`. Use `get_db()` context manager for all connections — never open `sqlite3.connect()` directly elsewhere.

```python
with get_db(db_path) as conn:
    df = pd.read_sql_query(sql, conn, params=(...))
```

**Admin-only scraper panel:** Set `ADMIN_PASSWORD=yourpassword` in `.env` (or cloud secrets) to gate the sidebar scraper controls behind a password prompt. If `ADMIN_PASSWORD` is not set, the panel is always visible (local dev default).

**Analysis columns** are prefixed `analysis_` (e.g., `analysis_sentiment`, `analysis_intent`, `analysis_author_role`, `analysis_product_area`, `analysis_cluster_id`). Add new analysis dimensions by adding keyword dictionaries and a new `analysis_*` column in `analyze_local.py`.

**Naming patterns:**
- Internal helpers: `_build_full_text()`, `_parse_replies_from_edges()` (underscore prefix)
- Analysis functions: `analyze_*()` 
- DB query functions: `query_*()` (in db.py)
- Dashboard load functions: `load_*()` (in app.py)

**Tech Community spider** uses `scrapy-playwright` for JavaScript rendering + GraphQL API calls. Reddit spider deliberately skips Playwright (uses the JSON API directly to avoid bot detection).

**Keyword-based analysis is the default** — the `openai`/`analyze_intent.py` path is optional and not wired into the main pipeline. When extending analysis, add to the keyword dictionaries in `analyze_local.py` first.

**`scrapy-poet`** is used for dependency-injected Page Objects in `customer_intent_scraper/pages/`. New spiders for JS-heavy sites should follow the same pattern as `techcommunity.py`; lightweight sites can use plain Scrapy callbacks.

**Fixtures** for tests live in `fixtures/`. Tests use real SQLite in-memory databases or fixture files — no mocking of the DB layer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SharedBench-Studio)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SharedBench-Studio)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
