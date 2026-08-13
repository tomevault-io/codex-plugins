## Copilot guide: Adlist-Parser (concise)

Purpose: Merge 50+ adlists into two normalized outputs for DNS blockers (Pi-hole/AdGuard) using only the Python stdlib. Typical run: ~50–60s, ~1.6M entries. Current version: **3.0.0**.

Quick rules (TL;DR for AI changes)

- Do keep stdlib-only. Never add runtime dependencies or change `pyproject.toml` beyond the existing optional `dev` extras.
- Do preserve the stable contracts and file boundaries below; treat them as public APIs.
- Do push heavy CPU/IO off the event loop with `asyncio.to_thread()` and cap concurrency to ≤16 workers.
- Do write LF-only files, keep deterministic case-insensitive sorting, and regenerate headers only in post-processing.
- Do maintain Windows-safe path/URL handling in `io.py` and avoid breaking relative path resolution.
- Do keep `cache/`, `output/`, and bulk list files in `data/` tracked via Git LFS (configured in `.gitattributes`); never remove or bypass the LFS tracking for those paths.
- Don’t widen domain-matching regexes or IDN heuristics; keep `_maybe_extract_domain` and `DOMAIN_RE` conservative.
- Don’t swap output formats or semantics (no element-hiding rules, ABP options stripped, `@@||` → whitelist).
- Don’t remove the “merge with existing adlist” step; it’s required to preserve entries across transient source failures.

### Architecture & flow

- `adparser/cli.py::main()` runs two pipelines in parallel via `asyncio.gather()`:
  - Adlist → `output/adlist.txt`
  - Whitelist → `output/whitelist.txt`
- Stages (heavy work off the loop via `asyncio.to_thread()`; live progress via `status.GroupedStatusDisplay`):
  1. `io.load_sources(JSON)` → List[Source] (paths resolved relative to the JSON file)
  2. `fetcher.fetch(sources, progress_cb)` → `(results, failed)` (≤16 workers)
  3. `content.generate_list(...)` → `(domains, abp_rules, failed)`
  4. Adlist only: merge with existing `output/adlist.txt` before write to preserve entries across transient source failures (whitelist writes directly)
  5. `io.write_output(path, domains+abp, header)` (LF-only)
  6. Post-process: `content.separate_blocklist_whitelist(adlist, whitelist)`; then re-write both files with fresh headers, counts, and case-insensitive sorting
  7. Redundancy + summary: `redundancy.generate_redundancy_report(...)`, `reporting.generate_report(...)`

### Key modules (single-responsibility)

- `content.py`: normalization, ABP handling, Pi-hole regex → ABP, dedupe
- `fetcher.py`: concurrent HTTP/local fetch, progress callback, UA header; ≤16 workers
- `io.py`: JSON loaders, path resolution (Windows-safe URL vs path), LF writes
- `status.py`: grouped spinners; `spinner.show_progress()` around async ops
- `redundancy.py`: duplicate/overlap analysis (incl. local-file coverage)
- `reporting.py`: emoji summary; wide-char-safe width calc
- `constants.py`: default `data/` inputs, `output/` paths, and `cache/` dir
- `models.py`: `Source` dataclass (`raw`, `resolved_path`, `is_url()`, `is_file_url()`)
- `conflicts.py`: detects entries present in both `data/blacklist.txt` and `data/whitelist.txt`; canonical comparison via `_convert_regex_to_abp`
- `duplicates.py`: `--remove-duplicates` pipeline; removes local entries already covered by remote sources or broader rules within each file

### Stable contracts (use as-is)

- `fetcher.fetch(sources, cb)` → `(results, failed)` where `cb(completed, total)` is called by workers
- `content.generate_list(results, sources, failed)` → `(domains, abp_rules, failed)`
- `content.separate_blocklist_whitelist(adlist, whitelist)` → `(clean_adlist, clean_whitelist)`
- `io.write_output(path, lines, header)` → LF-only; headers are regenerated after post-processing

### Parsing rules (project-specific)

- Hosts: `0.0.0.0 example.com` or `127.0.0.1 example.com` → `example.com`; leading IP anywhere is stripped
- Wildcards kept as tokens; output strips leading `*.` (e.g., `*.a.b` → `a.b`)
- ABP: `@@||example.com^` → whitelist; `||example.com^` → adlist; strip `$options`; element-hiding rules are dropped
- ABP cleanup (conservative): `||*cdn.site^` → `||*.cdn.site^`; `||app.*.adjust.com^` → `||*.adjust.com^`; `||domain.google.*^` → `||domain.google^`; `@@|domain.com^|` → `@@||domain.com^`
- Regex → ABP: anchored Pi-hole `(^|\.)domain\.com$` → `||domain.com^`; complex/JS-like regex discarded
- IDN kept via `_maybe_extract_domain` (punycode); don’t widen `DOMAIN_RE`

### Conventions & workflows

- Run: `python -m adparser` (PowerShell task: "Adlist-Parser") or installed script `adlist-parser` → `adparser.cli:main`
- Git LFS: `cache/`, `output/`, and bulk list files in `data/` are tracked via Git LFS. Run `git lfs install` once after cloning.
- Python 3.14+: `cli.py` uses `asyncio.run()` only — `asyncio.get_event_loop()` (removed in 3.14) is not used.
- `--remove-duplicates`: removes local entries in `data/blacklist.txt` / `data/whitelist.txt` that are covered by remote sources or by broader rules in the same file
- `--check-conflicts`: detects and reports entries that appear in both `data/blacklist.txt` and `data/whitelist.txt` (exits after report, does not modify files)
- Tests: Pytest with coverage (task: "Tests: Pytest (coverage)", config in `pyproject.toml`)
- Lint: Pylint report (task: “Lint: Pylint (report)”, output to `pylint-report.txt`)
- Fast dev loop: set `data/adlists.json` and `data/whitelists.json` to `{ "lists": ["blacklist.txt"], "urls": [] }` etc. for sub-second runs
- Source JSON keys accepted: `lists`, `urls`, `adlists`, `sources`; relative paths resolve against the JSON file’s location
- Post-processing: moves `@@||` from adlist to whitelist; drops shadowed blocks; writes LF-only outputs with regenerated headers

### VS Code tasks and quality gates

- Tasks (Windows PowerShell):
  - “Adlist-Parser” → runs `python -m adparser`
  - “Tests: Pytest (coverage)” → runs `pytest` with coverage as configured in `pyproject.toml`
  - “Lint: Pylint (report)” → runs pylint and writes `pylint-report.txt`
- Quality gates expectations:
  - Build: N/A (pure Python). Runtime entry point is `adparser.cli:main`.
  - Tests: PASS (fast subset with local data; full run may download sources and take ~50–60s).
  - Lint: PASS is ideal; if non-zero exit, report is still generated for review.

### Edge cases and invariants to keep

- Treat leading IPs anywhere in a hosts line as removable; keep only the hostname token.
- Wildcards are tokens only; output strips leading `*.` but does not expand wildcards.
- ABP cleanup stays conservative: avoid broadening patterns (e.g., prefer `||*.cdn.site^` over `||*cdn.site^`).
- Regex → ABP conversion only for simple anchored Pi-hole forms like `(^|\.)domain\.com$`.
- Sorting is case-insensitive; deduping must be case-insensitive and whitespace-trimmed.
- Network concurrency: ≤16; respect retry/backoff behavior in `fetcher.py` if present.

### Contribution tips

- Prefer small, pure functions with unit tests under `tests/` (pytest). Keep side effects (IO/prints) localized to `io.py`, `status.py`, and `cli.py`.
- Keep messages and summaries wide-char safe (see `reporting.py`).
- When touching fetch logic, keep UA header behavior and progress callback contract intact.

Examples: `example.com` kept; `*.sub.domain` → token; `(\.|^)domain\.com$` → `||domain.com^`.

---
> Source: [Toomas633/Adlist-Parser](https://github.com/Toomas633/Adlist-Parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
