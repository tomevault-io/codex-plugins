# Comet History Extractor Development Instructions

**ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**

## Project Overview

Comet History Extractor is a Python tool that extracts browser history from Comet Browser's SQLite database and formats it for AI processing. This is a **single-file Python application** with no external dependencies, designed for simplicity and cross-platform compatibility.

## Quick Start Development Setup

### Prerequisites & Installation
- **Python 3.8+** (tested with Python 3.12.3)
- **No external dependencies required** - uses Python standard library only
- Git for version control

### Bootstrap Commands
```bash
# Clone and enter repository (if not already done)
git clone <repository-url>
cd comet-history-extractor

# Verify Python version
python3 --version  # Should be 3.8+

# Test basic functionality (will fail without database - this is expected)
python3 extract_comet_history.py
# Expected output: "❌ Database not found at: comet_history_temp.db"
```

### Build Process
**There is no build process** - this is a direct-execution Python script using only standard library modules.

## Testing & Validation

### Manual Testing (Primary Method)
Since there are no automated tests, **ALWAYS** manually validate changes:

```bash
# 1. Create test database
sqlite3 test.db "
CREATE TABLE urls (id INTEGER PRIMARY KEY, url TEXT, title TEXT, visit_count INTEGER, typed_count INTEGER, last_visit_time INTEGER, hidden INTEGER);
CREATE TABLE visits (id INTEGER PRIMARY KEY, url INTEGER, visit_time INTEGER, visit_duration INTEGER, transition INTEGER, external_referrer_url TEXT);  
CREATE TABLE keyword_search_terms (url_id INTEGER, term TEXT);
INSERT INTO urls VALUES (1, 'https://github.com', 'GitHub', 5, 2, 13394145923000000, 0);
INSERT INTO visits VALUES (1, 1, 13394145923000000, 120000, 805306368, '');
INSERT INTO keyword_search_terms VALUES (1, 'github');
"

# 2. Test with small dataset (should take < 1 second)
cp test.db comet_history_temp.db
python3 extract_comet_history.py
# Expected: Successful extraction with 1 URL

# 3. Verify output files created
ls -la *.json *.csv
# Expected: comet_history_complete.json, comet_history_summary.csv, comet_history_statistics.json

# 4. Test error handling
rm comet_history_temp.db
python3 extract_comet_history.py
# Expected: "❌ Database not found" error message

# Cleanup test files
rm -f test.db *.json *.csv *.db
```

### Performance Testing
```bash
# Create large test database (10,000 entries)
python3 -c "
import sqlite3
conn = sqlite3.connect('large_test.db')
cursor = conn.cursor()
cursor.execute('CREATE TABLE urls (id INTEGER PRIMARY KEY, url TEXT, title TEXT, visit_count INTEGER, typed_count INTEGER, last_visit_time INTEGER, hidden INTEGER)')
cursor.execute('CREATE TABLE visits (id INTEGER PRIMARY KEY, url INTEGER, visit_time INTEGER, visit_duration INTEGER, transition INTEGER, external_referrer_url TEXT)')
cursor.execute('CREATE TABLE keyword_search_terms (url_id INTEGER, term TEXT)')
for i in range(1, 10001):
    cursor.execute('INSERT INTO urls VALUES (?, ?, ?, ?, ?, ?, 0)', (i, f'https://example{i}.com', f'Site {i}', i%20+1, i%5, 13394145923000000+i*1000))
    cursor.execute('INSERT INTO visits VALUES (?, ?, ?, ?, ?, ?)', (i, i, 13394145923000000+i*1000, 120000, 805306368, ''))
    cursor.execute('INSERT INTO keyword_search_terms VALUES (?, ?)', (i, f'search{i}'))
conn.commit()
conn.close()
print('Large test database created')
"

# Test performance (should complete in < 1 second)
cp large_test.db comet_history_temp.db
time python3 extract_comet_history.py
# Expected: ~0.5 seconds for 10,000 entries

# Cleanup
rm -f large_test.db *.json *.csv *.db
```

### Validation Scenarios
**ALWAYS run through these scenarios after making changes:**

1. **Basic Functionality**: Script runs with small database (1-10 URLs)
2. **Large Dataset**: Script handles 10,000+ URLs in < 1 second  
3. **Error Handling**: Script gracefully handles missing/corrupted database
4. **Output Integrity**: JSON and CSV files are valid and contain expected data
5. **Cross-Platform**: Script works on Windows, macOS, and Linux (test when possible)

## Code Quality & Linting

### Style Checking
```bash
# Install linting tools (optional, for development)
pip3 install flake8 mypy

# Check code style (will show many violations in current code)
python3 -m flake8 extract_comet_history.py

# Type checking
python3 -m mypy extract_comet_history.py --ignore-missing-imports

# Basic syntax validation
python3 -m py_compile extract_comet_history.py
```

**Current State**: The codebase has many flake8 violations (whitespace, line length) and some mypy issues. These are acceptable for now but should be fixed for new code.

## Application Architecture

### Core Components
- **`HistoryEntry`** dataclass: Data model for browser history entries
- **`HistoryExtractor`** class: Main extraction logic with methods:
  - `extract_data()`: Reads SQLite database 
  - `save_to_json()`: Exports comprehensive JSON
  - `save_to_csv()`: Exports summary CSV
  - `generate_summary()`: Creates statistics

### Database Schema (Comet Browser)
- **`urls`**: Main URL table with visit counts and metadata
- **`visits`**: Individual visit records with timestamps
- **`keyword_search_terms`**: Search terms associated with URLs

### Output Formats
- **`comet_history_complete.json`**: Full data for AI processing
- **`comet_history_summary.csv`**: Simplified spreadsheet format
- **`comet_history_statistics.json`**: Summary statistics and insights

## Development Workflow

### Making Changes
1. **Always test manually** before and after changes
2. **Keep changes minimal** - this is a working single-file tool
3. **Preserve backward compatibility** with existing database schema
4. **Test error conditions** - database missing, corrupted, permission denied

### Common Tasks

#### Adding New URL Categories
Edit the `_categorize_url()` method in `HistoryExtractor` class:
```python
# Add new domain patterns
if 'your-domain.com' in domain:
    return "Your Custom Category"
```

#### Modifying Output Format
- JSON: Edit `save_to_json()` method
- CSV: Edit `save_to_csv()` method  
- Statistics: Edit `generate_summary()` method

#### Database Schema Changes
- Modify SQL queries in `extract_data()` method
- Update `HistoryEntry` dataclass if needed
- **Always test with real Comet Browser database**

## File Structure
```
comet-history-extractor/
├── extract_comet_history.py    # Main application (366 lines)
├── README.md                   # User documentation
├── CONTRIBUTING.md             # Development guidelines
├── examples/                   # Sample outputs
├── guides/                     # Usage guides
├── templates/                  # AI prompt templates
└── .github/                    # GitHub templates
```

## Timing Expectations

### Expected Performance
- **Small datasets (1-100 URLs)**: < 0.1 seconds
- **Medium datasets (1,000 URLs)**: < 0.1 seconds  
- **Large datasets (10,000 URLs)**: < 0.5 seconds
- **Very large datasets (50,000+ URLs)**: < 2 seconds

**NEVER CANCEL** - All operations complete very quickly. Maximum expected time is under 5 seconds even for extremely large databases.

### No Long-Running Operations
Unlike many projects, this tool has:
- **No build process** (direct Python execution)
- **No compilation step** (interpreted language)  
- **No dependency installation** (uses standard library)
- **No test suite execution** (manual testing only)
- **No CI/CD pipeline** (currently)

## Error Conditions & Troubleshooting

### Common Issues
```bash
# Database not found (expected when no Comet data)
❌ Database not found at: comet_history_temp.db
# Solution: Copy real Comet database or create test database

# Permission denied (database locked by Comet Browser)
❌ Database error: unable to open database file
# Solution: Close Comet Browser before running script

# Import errors (rare, indicates Python installation issue)
❌ ModuleNotFoundError: No module named 'sqlite3'
# Solution: Reinstall Python with sqlite3 support
```

### Debugging Tips
- **Use small test databases** to isolate issues
- **Check file permissions** if database access fails
- **Verify Python version** (must be 3.8+)
- **Test on different platforms** when possible

## Platform-Specific Notes

### Windows
- Use `python` instead of `python3` if Python 3 is default
- Database path may use backslashes: `C:\Users\...\History`

### macOS  
- Comet database typically in: `~/Library/Application Support/Comet/Default/History`
- Use `python3` command explicitly

### Linux
- Comet database typically in: `~/.config/comet/Default/History`
- Ensure `python3-sqlite` package installed

## Contributing Guidelines

### Code Standards
- **Follow existing patterns** in the single Python file
- **Keep it simple** - avoid adding external dependencies
- **Test manually** with various database sizes
- **Document any breaking changes** to database schema expectations

### Validation Requirements
Before submitting changes:
- [ ] Script executes successfully with test database
- [ ] Performance remains under 1 second for 10K entries
- [ ] Error handling works for missing database
- [ ] Output files are valid JSON/CSV
- [ ] No new external dependencies added
- [ ] Code follows existing style patterns

## Key Locations

### Frequently Modified Files
- **`extract_comet_history.py`**: Main application logic (lines 59-112 for categorization, lines 114-218 for extraction)
- **`README.md`**: User-facing documentation 
- **`examples/`**: Sample output files for testing

### Important Code Sections
- **URL Categorization**: Lines 59-112 in `_categorize_url()` method
- **Database Extraction**: Lines 127-217 in `extract_data()` method  
- **Output Generation**: Lines 220-307 for JSON/CSV export
- **Error Handling**: Throughout, especially lines 115-120 and 214-218

## Final Reminders

- **This is a fast, simple tool** - don't overcomplicate it
- **Manual testing is critical** - no automated test suite exists
- **Performance is excellent** - operations complete in milliseconds  
- **Cross-platform compatibility** - test on multiple OS when possible
- **Keep external dependencies at zero** - this is a key project constraint

Always validate functionality with real database files when making significant changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucaspsrrio)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lucaspsrrio)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
