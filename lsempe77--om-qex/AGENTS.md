# GitHub Copilot Instructions for OM_QEX Project

## Quick Reference

**Main Documentation**: See `README.md` for project overview, architecture, and baseline performance  
**Technical Reports**: See `BASELINE_PERFORMANCE_REPORT.md` and `HUMAN_COMPARISON_RESULTS.md`  
**This File**: Development-specific technical details, troubleshooting, coding standards

## Shell Environment

- **Always use PowerShell syntax** (not bash)
- Shell: Windows PowerShell v5.1
- Use `;` to join commands on a single line
- Use `type` instead of `cat`, `ls` instead of `ls`, etc.
- Path separators: backslash `\` (though forward slash `/` works in most contexts)

## Code Standards

### Python

- Use `pandas` for CSV/data manipulation
- Use `pathlib.Path` for file paths
- Type hints encouraged (`from typing import Dict, List, Optional`)
- Logging: Use `logger.info()` not `print()` in production code
- Error handling: Specific exceptions, informative messages

### File Operations

- **ALWAYS archive old files** before major changes:
  ```powershell
  mkdir archive/old_prompts -Force
  mv prompts/extraction_prompt.txt archive/old_prompts/extraction_prompt_v1.txt
  ```

- **Use descriptive filenames** with timestamps:
  ```
  outputs/extractions_2025-11-10/
  archive/comparer_v1_2025-11-08.py
  ```

### Git Workflow

- Commit frequently with descriptive messages
- `.gitignore` excludes: PDFs, API keys, `__pycache__`, outputs
- Document major changes in README or dedicated markdown files

## Data Files Quick Reference

### ID System
- **Study ID** (e.g., `121294984`): EPPI-Reviewer identifier in CSVs
- **Key** (e.g., `PHRKN65M`): Filename for TEI/TXT files
- **Mapping**: Master CSV links Study ID ↔ Key

### Key Files
- **TEI XML**: `data/grobid_outputs/tei/{key}.tei.xml` - Structured paper text
- **Plain Text**: `data/grobid_outputs/text/{key}.txt` - Backup text source
- **Master CSV**: `data/raw/Master file of included studies (n=114) 11 Nov(data).csv`
  - Contains study IDs and metadata
  - 114 studies total, 95 with complete metadata
- **Human QEX**: `data/human_extraction/8 week SR QEX Pierre SOF and TEEP.csv` (skiprows=2)
- **Human OM**: `data/human_extraction/OM_human_extraction.csv`

### Test Papers
- **PHRKN65M**: Study 121294984 (Burchi 2018, Malawi TEEP) - 9 human outcomes
- **ABM3E3ZP**: Study 121058364 (Maldonado 2019, Paraguay SOF)

## Running Extractions

```powershell
cd om_qex_extraction

# Two-stage pipeline (recommended)
python run_twostage_extraction.py --keys PHRKN65M

# Single modes
python run_extraction.py --mode om --keys PHRKN65M   # Find all outcomes
python run_extraction.py --mode qex --keys PHRKN65M  # Extract details

# Batch processing
python run_extraction.py --sample 5  # 5 random papers
python run_extraction.py --all       # All 95 papers
```

## Common Issues and Troubleshooting

### HTTP Timeout/Connection Errors
**Symptom**: `KeyboardInterrupt` or connection errors during API response reading  
**Root Cause**: Network instability during response streaming (misleading exception name)

**Traceback shows**: `File "...\ssl.py", line 1106, in read` → `KeyboardInterrupt`

**Solutions Applied**: 
- Timeout configuration: `connect=15s, read=300s, write=15s, pool=15s`
- `max_retries=5` with exponential backoff
- KeyboardInterrupt handler for socket timeouts
- Location: `src/extraction_engine.py::_initialize_client()`

**Workaround**: If two-stage fails, run OM and QEX modes separately

### Unicode Encoding Errors (Windows)
**Symptom**: `UnicodeEncodeError: 'charmap' codec can't encode character`  
**Solution**: Force UTF-8 encoding in `run_extraction.py`:
```python
if sys.platform == 'win32':
    sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```

### Reading Data Files

```powershell
# Master CSV
python -c "import pandas as pd; df = pd.read_csv('data/raw/Master file of included studies (n=114) 11 Nov(data).csv'); print(df.head())"

# Extraction output
python -c "import pandas as pd; df = pd.read_csv('om_qex_extraction/outputs/extractions/extracted_data.csv'); print(df[['study_id', 'outcome_name', 'effect_size']].head(10))"

# Human extraction (QEX)
python -c "import pandas as pd; df = pd.read_csv('data/human_extraction/8 week SR QEX Pierre SOF and TEEP.csv', skiprows=2); print(df[df['StudyID']=='121294984'])"
```

### JSON Inspection

```powershell
cat outputs\extractions\json\PHRKN65M.tei.json | python -m json.tool | Select-Object -First 80
```

## Prompt Engineering

### Prompt Files
- **OM Prompt**: `prompts/om_extraction_prompt.txt` - Find all outcomes with statistics
- **QEX Prompt**: `prompts/extraction_prompt.txt` - Extract detailed statistical data
- **QEX Focused**: `prompts/qex_focused_prompt.txt` - QEX with OM guidance (two-stage)

### Prompt Structure
1. Task description and context
2. JSON schema with field definitions
3. Extraction rules and criteria
4. Examples (1-2 outcomes)
5. Mental verification checklist (for OM)

### Verification Fields (Added Nov 10, 2025)
Every outcome includes fields for manual verification:

- **literal_text**: Exact quote from paper (e.g., `"Project | 4,408** (1,369)"`)
- **text_position**: Precise location (e.g., `"Table 6, Row 'Project', Column '(3) Amount savings (MWK)'"`)

**Implementation**: Added to JSON schema, CSV flattening, and prompt examples

### Testing Workflow
1. **Modify prompt** - Update extraction criteria or field definitions
2. **Test on PHRKN65M** - Paper with 9 human-extracted outcomes
3. **Check output** - Verify JSON structure, outcome count, field completeness
4. **Compare results** - Did change improve/degrade extraction quality?
5. **Document** - Note what worked/didn't in prompt comments or changelog

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lsempe77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
