# Copilot Instructions — crawl4md

## Project Overview

crawl4md is a Python library for crawling websites and extracting content as Markdown-formatted text files. It wraps Crawl4AI with a synchronous API designed for non-technical Jupyter Notebook users.

## Data Flow

```
SiteCrawler.crawl()
  ├─ Crawl4AI (async) → CrawlResult (raw HTML per page)
  ├─ ContentExtractor  → ExtractedPage (clean Markdown per page)
  ├─ FileWriter         → size-limited content files + URL lists
  └─ ContentSorter      → sorted final files grouped by URL path
```

Each crawl creates a timestamped output directory. Results pass through multiple rounds (initial + retries), then are merged, deduplicated, and sorted.

## File Layout

```
src/crawl4md/
├── __init__.py       # Public API exports
├── config.py         # Pydantic config models
├── crawler.py        # SiteCrawler class
├── extractor.py      # ContentExtractor class
├── sorter.py         # ContentSorter class
├── writer.py         # FileWriter class
└── progress.py       # ProgressReporter class
```

## Architecture

### Config models (`config.py`)

Pydantic v2 models — all user-facing parameters are validated here.

- **CrawlerConfig** — fields: `urls`, `exclude_paths`, `include_only_paths`, `limit`, `max_depth`, `flush_interval`, `delay`, `stealth`, `headers`, `max_retries`. Accepts CSV strings for list fields; validates regex patterns.
- **PageConfig** — fields: `exclude_tags`, `include_only_tags`, `wait_until`, `wait_for`, `timeout`, `max_file_size_mb`, `extract_main_content`, `output_extension`, `separate_items`, `item_selector`, `js_code`, `scan_full_page`, `scroll_delay`.
- **CrawlResult** — per-page output: `url`, `html`, `markdown`, `success`, `error`, `redirected_url`, `is_pdf`.
- **ExtractedPage** — post-extraction output: `url`, `title`, `markdown`.

**Constraints:**

- Cannot set both `exclude_tags` and `include_only_tags` on PageConfig.
- `headers` is a free-form `dict[str, str]` forwarded to Crawl4AI's `BrowserConfig`.

### SiteCrawler (`crawler.py`)

Synchronous wrapper around Crawl4AI's async crawler. Uses `nest_asyncio` for Jupyter compatibility; on Windows uses `ProactorEventLoop` in a `ThreadPoolExecutor`.

**Multi-round strategy:**

1. Round 1 — crawl seed URLs with link discovery (respects `max_depth` and `limit`). Per-page jitter applied only when `delay > 0`.
2. Rounds 2+ — retry failed URLs with link discovery. Cooldown pause between rounds. Escalating delay with jitter.
3. WAF back-off — immediate back-off on WAF detection; escalates after consecutive blocks.
4. Cross-round state — `all_generated` and `url_depths` shared across rounds for dedup and depth tracking.
5. Session persistence — single `AsyncWebCrawler` instance shared across all rounds (cookies persist).
6. Early exit — skip remaining retries if all pages succeed.

**Constraints:**

- URL filtering: same-domain only (with `www.` stripping), skips boilerplate domains and static asset extensions (excluding `.pdf`), applies `include_only_paths`/`exclude_paths` regex, skips template placeholders (`{{`, `{%`).
- PDF handling: layered detection — URL extension fast path (`.pdf`) + Content-Type HEAD-request fallback when crawl returns empty content. PDFs are downloaded via httpx (bypassing the browser) and converted to Markdown via pymupdf4llm. `CrawlResult.is_pdf` flag routes extraction through `_extract_pdf_page()` which skips HTML preprocessing. OCR for scanned/image-only PDFs is controlled by `PageConfig.ocr_languages` (default `["eng", "msa"]`); empty list disables OCR. Tesseract-not-found errors are caught once per crawl with a warning — OCR silently degrades to plain extraction.
- WAF detection is two-stage: HTML signature check + post-extraction content-length check.
- `_ROUND_COOLDOWN` is patched to 0 in tests via autouse fixture in `conftest.py`.

### ContentExtractor (`extractor.py`)

Converts crawled HTML to Markdown. Three modes: trafilatura (default, `extract_main_content=True`) with markdownify fallback, markdownify-only (`extract_main_content=False`), or PDF mode (`is_pdf=True` — skips HTML preprocessing, derives title from URL filename).

**Pipeline order:** pre-processing → extraction → supplementary section recovery (trafilatura mode) → table fixing → post-processing (`_clean_markdown`) → mdformat validation.

**Constraints:**

- Pre-processing is always-on (tag filtering, strikethrough preservation, heading spacing, empty link population, item separation, table fixing) — read the function docstrings for details.
- `_populate_empty_links()` is always-on with no config flag.
- `_insert_item_separators()` uses `_ITEM_SENTINEL` marker that must survive trafilatura extraction.
- mdformat validation has a content-preservation guard — falls back to original text if any content is lost.
- Supplementary sections (FAQs, accordions) and product metadata are recovered in trafilatura mode only.

### ContentSorter (`sorter.py`)

Groups extracted pages by URL path for natural display order. Stable sort preserves crawl order for identical paths.

### FileWriter (`writer.py`)

Combines extracted pages into size-limited output files. Never splits a single page across files. Oversized pages get their own file with a `UserWarning`.

**Constraints:**

- Batch mode (`write()`) and incremental mode (`add()`/`flush()`/`reset()`) — both must be maintained.
- A symmetric `_fail_writer` handles failed-page output.
- Output extension is configurable via `PageConfig.output_extension`.

### ProgressReporter (`progress.py`)

Real-time progress display. Auto-detects Jupyter vs terminal.

**Constraints:**

- Jupyter widget uses `_repr_html_()` — all CSS classes must be prefixed with `c4md-`.
- Colors are centralised in `_LIGHT_COLORS` and `_DARK_COLORS` module-level dicts. Standard CSS uses `_LIGHT_COLORS` as defaults with `@media (prefers-color-scheme: dark)` overrides.
- **Colab compatibility:** `_repr_html_colab()` uses only inline `style="..."` attributes — no `<style>` blocks, no `@keyframes`, no `position: absolute`, no `filter`, no `animation`. Colab's HTML sanitizer strips all of these.
- `set_activity()` / `update_activity_label()` are optional — reporters that don't call them still produce a valid widget.

## Output File Naming

Timestamped folder (`YYYY-MM-DD_HH-MM-SS`) with three tiers:

```
round_{N}_{success|fail}_content_{NNN}.ext   # Per-round
round_{N}_{success|fail}_urls.txt
final_{success|fail}_content_{NNN}.ext       # Merged after all rounds
final_{success|fail}_urls.txt
sorted_final_{success|fail}_content_{NNN}.ext  # Sorted final
sorted_final_{success|fail}_urls.txt
```

`{N}` = round number (1-based), `{NNN}` = file index (001, 002, …), `ext` = `PageConfig.output_extension`.

## Coding Conventions

- Python 3.10+, type hints on all public APIs.
- Pydantic v2 for data models (use `model_validator`, `field_validator`).
- Linting via ruff (config in `pyproject.toml`).
- Tests use pytest with mocked HTTP calls — never make real network requests in tests.
- Keep the notebook UX simple: plain language, no jargon, no code explanations.
- **No inline magic values:** All behavioral thresholds, tag lists, regex patterns, and repeated string literals must be defined as module-level `_UPPER_SNAKE_CASE` constants with a descriptive comment. Group constants categorically at the top of the file (after imports). Regex patterns should be `re.compile()`d at module level for clarity and performance. **Exempt:** Pydantic field defaults, standard Python idioms (e.g. `divmod(x, 60)`, `stacklevel=2`), spec-defined keys used once (JSON-LD properties, HTML attribute names in BeautifulSoup calls), and trivial markdown formatting strings (`"- "`, `"### "`).

## Dependencies

Key deps: crawl4ai, trafilatura, markdownify, pydantic, nest-asyncio, beautifulsoup4, mdformat (with mdformat-gfm), pymupdf4llm, httpx. See `pyproject.toml` for the full list.

## Testing

- **Every code change must include unit tests** — new features need tests for the happy path and key edge cases; bug fixes need a test that reproduces the bug.
- **Before making changes or implementing a plan, always start with a TODO list** — write the planned steps first, then execute and update the list as work progresses.
- **All tests must pass before a task is considered complete.** Run `python -m pytest tests/ -q` and confirm zero failures.
- **After tests pass, run linting:** `ruff check src/ tests/` and `ruff format --check src/ tests/`. Fix any errors and re-run both tests and linting until **both pass with zero errors**. A task is not complete until tests AND linting are clean.
- **For test and lint verification, always delegate to the `test-runner` agent** instead of running commands directly. This agent uses a two-pass strategy: quiet run first, then verbose re-run of failures only.
- `_ROUND_COOLDOWN` is globally patched to 0 via an autouse fixture in `conftest.py`. No per-test patching is needed.
- When running tests in a terminal, **always wait for the command to finish and return its full output** before re-running, retrying, or drawing any conclusions. Do not start a new test run while one is still in progress.

## Maintaining This File

**Target size:** This file must stay under 250 lines. If an update would exceed that, condense or remove existing content to make room.

**What belongs here:** Constraints the agent can't discover from source code — conventions, gotchas, cross-module rules, validation constraints, testing policy. Structural overview (file layout, data flow, output naming patterns).

**What does NOT belong here:** Implementation details discoverable by reading source — function behavior, algorithm steps, constant values, CSS specifics, rendering logic, fallback chains. If the agent needs those details, it can read the relevant file.

**When to update this file:** Only when a change introduces a new *constraint* or *convention*, removes one, or makes existing content factually wrong. Do NOT add descriptions of new features — only add rules about how to work with them correctly.

**When to update README.md:** When user-facing behavior changes (new config params, changed defaults, new features, installation steps). README describes *what the tool does for users*; this file describes *how to work on the code correctly*.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prakosd)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/prakosd)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
