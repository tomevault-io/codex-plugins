Repository: Launchpad Student Feedback CSVs

## Overview
- This repository contains 10 CSV exports of student feedback surveys (filenames begin with "All Student Feedback - ...") from the Launchpad program (Foundations, 101, Summer sessions).
- **Total data**: 446 student responses across 179 unique questions
- **Workflow**: Extract questions inventory → Merge all surveys → Analyze responses
- Two main Python scripts automate data processing (see `scripts/` folder)

## Architecture & Data Flow
```
Raw CSVs (10 files)
    ↓
scripts/generate_questions_summary.py → questions_summary.csv (question inventory)
    ↓
scripts/merge_all_surveys.py → data/processed/merged_surveys.csv (consolidated data)
```

## Key Files & Patterns
**Input Files** (repository root):
- `All Student Feedback - *.csv` (10 files)
- Examples: `All Student Feedback - 10End of 101 Feedback survey .csv`

**Scripts** (`scripts/` folder):
- `generate_questions_summary.py` - Extracts all unique questions and shows which surveys contain each
- `merge_all_surveys.py` - Consolidates all surveys into single CSV with provenance tracking

**Output Files**:
- `questions_summary.csv` - Question inventory (179 questions × survey sources)
- `data/processed/merged_surveys.csv` - Consolidated dataset (446 rows × 181 columns)

**CSV Header Patterns**:
- Standard: `What is your name?`, `Email Address`, `Timestamp`
- Likert scales: `Launchpad has helped me develop a clear idea of the careers I am interested (or not interested) in pursuing.`
- NPS: `On a scale of 0 (not at all likely) to 10 (extremely likely), how likely is it that you would recommend Launchpad to a friend or classmate?`
- Free text: `How has Launchpad supported your career and education plans?`
- Staff feedback: `What is one thing [Name] does well?`, `What is one thing [Name] can do better?`

## Critical Implementation Details
**Header Normalization** (see `normalize_header()` function):
```python
# Strip whitespace, collapse multiple spaces, normalize quotes
normalized = header.strip()
normalized = re.sub(r'\s+', ' ', normalized)  # "What  is   your name?" → "What is your name?"
normalized = normalized.replace('"', '"').replace('"', '"')  # Smart → straight quotes
```

**Provenance Tracking** (see `merge_all_surveys.py`):
- Every merged row includes `_source_file` (filename) and `_source_row` (original CSV row number)
- Example: `_source_file="All Student Feedback - 10End of 101 Feedback survey .csv", _source_row=3`

**CSV Parsing** (critical settings):
```python
# Use these pandas settings to handle embedded newlines and preserve data
pd.read_csv(file, encoding='utf-8', keep_default_na=False, na_values=[''], dtype=str)
```

## Developer Workflows

**Setup** (first time):
```bash
pip install pandas  # Only dependency
```

**Generate Question Inventory**:
```bash
python scripts/generate_questions_summary.py
# Output: questions_summary.csv (shows all 179 questions + which surveys have them)
```

**Merge All Surveys**:
```bash
python scripts/merge_all_surveys.py
# Output: data/processed/merged_surveys.csv (446 rows, 181 columns)
```

**Inspect Headers Quickly**:
```bash
# Bash/Git Bash
head -n 1 "All Student Feedback - 10End of 101 Feedback survey .csv"

# PowerShell
Get-Content "All Student Feedback - 10End of 101 Feedback survey .csv" -TotalCount 1
```

## Conventions & Gotchas
- **Filename numbering is arbitrary** - Don't rely on sort order for chronology
- **Headers contain embedded newlines** - Use proper CSV parsers (csv.reader, pandas)
- **Inconsistent spacing/quotes** - Always normalize before matching headers
- **Free-text responses** - Contain commas, newlines, quotes; never split on delimiters manually
- **Missing columns are normal** - Different surveys have different questions (use outer join)
- **Preserve original CSVs** - Never modify raw data; write outputs to `data/processed/`

## Code Pattern Examples

**Read headers safely**:
```python
import csv
with open(csv_path, 'r', encoding='utf-8', newline='') as f:
    reader = csv.reader(f)
    headers = next(reader)  # Handles embedded newlines correctly
```

**Merge with provenance**:
```python
df['_source_file'] = filename
df['_source_row'] = df.index + 2  # +2: row 1=headers, pandas 0-indexed
merged_df = pd.concat(all_dfs, ignore_index=True, sort=False)  # Outer join
```

## When to Ask the User
- If creating new analysis scripts that require specific question filtering
- If merging logic needs to handle duplicate questions differently
- If output format needs to change (e.g., JSON instead of CSV)

## Quality Checks
After running scripts, verify:
- `questions_summary.csv` has 179 rows (one per unique question)
- `merged_surveys.csv` has 446 rows (total student responses)
- Common questions (Timestamp, Name, Email) have 100% coverage
- Provenance columns (`_source_file`, `_source_row`) are populated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MO-fr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MO-fr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
