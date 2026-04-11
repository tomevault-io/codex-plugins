# AI Coding Agent Instructions - SQL KPI Pipeline

## Project Overview
This is a **daily KPI reporting and anomaly detection system** for e-commerce metrics. It executes a three-stage pipeline:

1. **runner.py**: Query SQL metrics files → `metrics_<date>.json`
2. **evaluator.py**: Analyze metrics against KPI rules + statistical anomalies → `eval_<date>.json`
3. **report_writer.py**: Generate markdown report (with LLM fallback) → `report_<date>.md`

All outputs land in `outputs/` with date-based filenames (YYYY-MM-DD).

## Architecture & Data Flow

### Stage 1: Metrics Collection (runner.py)
- **Input**: SQL files in `metrics/` (numbered 01-06)
- **Process**: Execute each `.sql` file against SQLite database (`database/olist.sqlite`)
- **Output**: JSON with `metrics` dict mapping `{metric_name: [rows]}`
- **Key Pattern**: Row dict uses `metric_date` as the timestamp key; all other keys are numeric value_keys

### Stage 2: Evaluation (evaluator.py)
- **Input**: `metrics_<date>.json` + KPI rules from `kpi_rules.yaml`
- **Process**:
  - Load KPI rules (id, metric_file, value_key, condition, threshold)
  - Compute stats over trailing 28-day window: mean, std, min, max, z-score
  - Detect anomalies: z-score >= 2.0, record_high_28d, record_low_28d
  - Evaluate KPI conditions (>=, <=, >, <, ==, !=)
- **Output**: JSON with `summary`, `kpis` (PASS/FAIL/MISSING), `alerts`, `metric_insights`
- **Critical Detail**: Z-score uses population std (not sample); trailing window **includes report_date**

### Stage 3: Report Writing (report_writer.py)
- **Input**: `eval_<date>.json`
- **Process**:
  - Build plain-text "facts" from KPIs and alerts (used as context)
  - If `REPORT_WRITER_NO_LLM=false`: Call Ollama LLM to generate markdown
  - If `REPORT_WRITER_NO_LLM=true`: Generate deterministic fallback markdown
- **Output**: Markdown report with fixed sections (Executive Summary, KPI Status, Notable Movements, etc.)
- **LLM Config**: Uses Ollama with `mistral:7b-instruct-q4_K_M` by default; configurable via env

## Developer Workflows

### Running the Pipeline
```bash
# Single day (defaults to today)
python runner.py --date 2018-08-01
python evaluator.py --date 2018-08-01
python report_writer.py --date 2018-08-01 [--dry-run] [--verbose]

# All three in order (date defaults to today)
python runner.py && python evaluator.py && python report_writer.py
```

### Adding a New Metric
1. Create `metrics/NN_<metric_name>.sql` (numbered sequentially; must query 1+ rows with `metric_date` column)
2. Add `value_key` columns (any numeric columns besides `metric_date` will auto-detect)
3. Test: `python runner.py --date <test_date>` → inspect `outputs/metrics_<date>.json`

### Adding a New KPI Rule
1. Edit `kpi_rules.yaml`: add `{id, metric_file, value_key, condition, threshold}`
2. Example: `id: orders_min; metric_file: 01_orders_daily; value_key: orders; condition: ">="; threshold: 1000`
3. Test: `python evaluator.py --date <test_date>` → inspect `outputs/eval_<date>.json` for `kpis[]` array

## Code Patterns & Conventions

### Date Handling
- **Consistent**: Always use `date.isoformat()` for JSON/output filenames (YYYY-MM-DD format)
- **Trailing Window**: `start = report_date - timedelta(days=27)` (28 days *inclusive*)
- **Prev Day/Week**: Subtracted as timedeltas from report_date, never as absolute dates

### Value Coercion
- `_coerce_number(value)` → `Optional[float]` (handles int, float, string "123", None)
- **Key pattern**: SQL `.sql` files must include `metric_date` column; all numeric columns auto-detect as value_keys
- Missing data: None values → N/A in output, skipped in statistical windows

### Statistics
- **Mean**: `sum(values) / len(values)` (arithmetic mean)
- **Std**: Population std (`sqrt(sum((v-mean)²) / N)`), not sample
- **Z-score**: `(today_value - mean) / std` (triggers alert if |z_score| >= 2.0)
- **DoD/WoW**: "Abs" = absolute diff; "Pct" = percentage change (e.g., `(today - prev) / prev`)

### Formatting
- **Monetary**: `f"${num:,.2f}"` (revenue, AOV)
- **Percentage**: `f"{num*100:.1f}%"` (late_delivery_rate); deltas as `pp` (percentage points)
- **Scores**: `f"{num:.2f} / 5"` (avg_review_score)
- **Counts**: `f"{int(round(num)):,}"` (orders, active_customers)
- **Generic**: `f"{num:.4g}"` (fallback)

### KPI Rule Evaluation
- Condition parser: `_evaluate_condition(value, condition, threshold)` supports: `>=`, `<=`, `>`, `<`, `==`, `!=`
- Status logic:
  - `PASS`: value satisfies condition
  - `FAIL`: value violates condition
  - `MISSING`: value is None (no data for that metric/value_key on report_date)

### Alert Types
- `kpi_breach`: KPI threshold violation (separate, rarely shown in report narrative)
- `record_high_28d` / `record_low_28d`: Highest/lowest in 28-day window
- `zscore_alert`: |z_score| >= 2.0 (statistical outlier)

### LLM Integration (report_writer.py)
- **Fallback**: If Ollama unavailable or `NO_LLM=true`, use `generate_markdown_fallback()` (deterministic, no LLM)
- **System Prompt**: Instructs LLM to translate anomalies for non-technical management
- **User Prompt**: Structured as "BRIEF" (plain-text facts) → "Write markdown with 7 sections"
- **LangChain**: Uses `ChatOllama` + `langsmith` for tracing (optional but recommended)

## Environment & Dependencies
- **Python**: 3.7+
- **Key packages**: `pyyaml`, `langchain-ollama`, `langchain-core`, `langsmith` (latter three optional for LLM mode)
- **SQLite**: `database/olist.sqlite` (Olist e-commerce sample database)
- **Env vars**:
  - `OLLAMA_BASE_URL` (default: `http://localhost:11434`)
  - `OLLAMA_MODEL` (default: `mistral:7b-instruct-q4_K_M`)
  - `REPORT_WRITER_NO_LLM=true` (force fallback markdown; no LLM call)

## Common Pitfalls & Troubleshooting

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| "Missing eval file" | evaluator.py not run yet | Run `python evaluator.py --date <date>` first |
| Metric value is None | SQL query returned no rows or empty `metric_date` | Debug SQL: ensure GROUP BY includes all rows for target date |
| KPI shows MISSING | report_date not in SQL result set | Verify SQL generates rows for all dates in range |
| Z-score alerts missing | Std is 0 (all window values identical) | Z-score calc is skipped; check data variance |
| Report shows "N/A" | Missing metrics in metric_insights | Likely: metric not in kpi_rules.yaml or value_key mismatch |
| LLM report never appears | Ollama not running | Either start Ollama or set `REPORT_WRITER_NO_LLM=true` |

## Testing & Validation
- **Quick validation**: Run pipeline with `--dry-run` flags (outputs to stdout instead of files)
- **Date-based testing**: Use past dates to test with real historical data (e.g., `--date 2018-08-01`)
- **Verbose mode**: `--verbose` includes PASS KPIs and detailed stats in facts
- **Output inspection**: JSON is human-readable; grep for `"status"` or `"type"` to debug logic

## Key Files Reference
- [runner.py](runner.py) – Metric query executor
- [evaluator.py](evaluator.py) – KPI & anomaly evaluator (~515 lines, core logic)
- [report_writer.py](report_writer.py) – Markdown generator with LLM fallback (~450 lines)
- [kpi_rules.yaml](kpi_rules.yaml) – KPI threshold definitions
- [metrics/](metrics/) – SQL query templates (01-06)
- [outputs/](outputs/) – JSON and markdown reports (date-keyed)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jana-go)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jana-go)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
