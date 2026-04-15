### QA Bugs — Copilot instructions

Keep instructions short and actionable. Focus on the existing architecture and conventions.

**📝 Documentation Requirements:**
- **ALWAYS update documentation when making code changes**
- Update README.md for user-facing features
- Update docs/*.md for technical documentation
- Keep copilot-instructions.md current with architecture changes
- Add inline comments for complex logic
- Update roadmap files (AI_DATA_ANALYSIS_LAYER.md) when implementing planned features

- Project entry: CLI `qa-bugs` (console script -> `qa_bugs.cli.cli:app`). See `qa_bugs/cli/cli.py`.
- Streamlit UI: `src/qa_bugs/ui/app.py` - Web interface using same AnalysisService as CLI
- Main data flow: CSV -> `Normalizer` -> `apply_filters` -> metric compute -> `ReportBuilder` -> `output/run_*/report.html`.

- Metric contract: every metric returns `MetricResult` (see `qa_bugs/metrics/base.py`). Key fields:
  - `.tables`: dict[str, pandas.DataFrame]
  - `.summary`: short string used in the report
  - use `Metric.payload()` when passing results to the LLM
  - **Field Validation**: All metrics MUST validate required fields before accessing them to prevent KeyError.
    - Check if field exists: `if "field_name" not in df.columns`
    - Return early with empty/zero result and descriptive summary if required fields are missing
    - For optional fields, create null column if missing: `df["field_name"] = pd.NaT`
  - **Mandatory Fields** (blocks analysis if missing):
    - `created_at` - Required for all time-based metrics
    - `resolved_at` - Required for tracking defect resolution and closure times
    - `priority` - Required for priority-based analysis and most metrics
    - `status` - Required for status-based metrics
    - `environment` - Required for environment-based metrics

- Ingestion conventions:
  - Normalize field names via `qa_bugs/ingest/normalizer.py` using `fields_mapping` from config.
  - Timestamps are coerced with `pd.to_datetime(..., errors='coerce')` and timezone handling is intentional.
  - `environment` is upper-cased by the Normalizer.

- Metrics registration: metric classes are listed in `qa_bugs/metrics/__init__.py` as the `METRICS` mapping.
  - Add new metrics by creating a class inheriting from `Metric` and registering its id in that mapping.

- LLM integration:
  - `qa_bugs/llm/service.py` uses Azure OpenAI via `openai.AzureOpenAI` and loads prompts from `qa_bugs/prompts`.
  - LLM is optional; disable with `--llm off` or set `llm.enabled: false` in config.
  - Prompts contain placeholders `{{context_json}}` / `{{metrics_context_json}}` that are replaced with JSON payloads.

- Report rendering: `qa_bugs/report/builder.py` builds Plotly figures and an inline Jinja2 template. When adding tables, ensure column names match report expectations (examples: `age_days`, `priority`, `avg_age`, `opened`, `closed`, `rate_percent`).

- Config & run examples (from `README.md`):
  - Create venv, install editable: `python -m venv .venv` then Windows activate: `.venv\Scripts\activate` and `pip install -e .`
  - Run: `qa-bugs run --config configs/example.config.yml --input data/sample_bugs.csv --since 2025-09-01 --until 2025-09-30 --metrics defect_age,age_by_priority --llm off`

- Tests / quick checks: project is lightweight; verify by importing modules and running `qa-bugs run` on `data/sample_bugs.csv`. Watch for missing env vars for Azure OpenAI (`AZURE_OPENAI_KEY`, `AZURE_OPENAI_ENDPOINT`).

- Style & expectations:
  - Follow existing typing and PEP8 style; the repo uses type hints in key places.
  - Prefer config-driven parameters (see `configs/example.config.yml`).

- AI Data Understanding Layer:
  - `qa_bugs/services/data_profiler.py` - AI-powered semantic classification (Phase 0 ✅)
  - `qa_bugs/services/analysis_service.py` - Main orchestration with profiler integration
  - `qa_bugs/ui/components/results_display.py` - Streamlit UI display components
  - Roadmap: `docs/AI_DATA_ANALYSIS_LAYER.md` - tracks MVP implementation (Phases 1-5 planned)

- Files to inspect when working on features or bugs:
  - `qa_bugs/cli/cli.py` (CLI orchestration)
  - `qa_bugs/ui/app.py` (Streamlit UI)
  - `qa_bugs/services/analysis_service.py` (core analysis workflow)
  - `qa_bugs/ingest/normalizer.py`, `qa_bugs/ingest/filters.py` (data ingestion)
  - `qa_bugs/metrics/*` (metric implementations)
  - `qa_bugs/report/builder.py` (HTML report rendering)
  - `qa_bugs/llm/service.py`, `qa_bugs/prompts/` (LLM behavior)
  - `qa_bugs/services/data_profiler.py` (AI data classification)

**Documentation Files:**
- `README.md` - Main user guide
- `docs/AUTO_MAPPING.md` - Automatic field detection
- `docs/LLM_PROVIDERS.md` - LLM configuration
- `docs/AI_DATA_ANALYSIS_LAYER.md` - MVP roadmap and implementation tracker
- `STREAMLIT_DEPLOYMENT.md` - UI deployment guide

If anything here is unclear or you need more specifics (tests, sample config fields, or examples of adding a metric), tell me which area to expand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deni-m) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
