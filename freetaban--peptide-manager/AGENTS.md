# GitHub Copilot Instructions - Peptide Management System

## 🎯 Mission Critical Rules

### 1. NEVER Delete Code Without Explicit Verification
- Before removing ANY code block, verify it's not used elsewhere
- Use `grep_search` or `list_code_usages` to check references
- If unsure about code purpose, ask user before deletion
- **LESSON LEARNED**: Commit 9817859 deleted AlertDialog code causing production break

### 2. One Problem = One Commit
- **NEVER** mix multiple fixes in single commit
- Each commit should address exactly ONE issue
- Refactoring must be separate from bug fixes
- If commit touches >100 lines, stop and split it

### 3. Test Before Commit - ALWAYS
```powershell
# Before EVERY commit:
python -m pytest tests/ -v
python scripts/smoke_test_gui.py  # If GUI modified
```

### 4. Read Full Context Before Editing
- Always read entire function/class before modifications
- Never edit based on partial context
- Use `read_file` with sufficient line range
- Verify surrounding code won't break

---

## 📋 Pre-Modification Checklist

Before ANY code change:

### ✅ Phase 1: Context Gathering
1. [ ] Read complete function/method being modified
2. [ ] Check for usages: `list_code_usages` for functions/classes
3. [ ] Search for string patterns: `grep_search` for related code
4. [ ] Review related test files in `tests/`
5. [ ] Check if change affects database schema

### ✅ Phase 2: Planning
1. [ ] Identify exact scope of change (single function? single file?)
2. [ ] Plan commits: split if >1 issue
3. [ ] Identify tests to run post-change
4. [ ] Consider backwards compatibility

### ✅ Phase 3: Implementation
1. [ ] Make minimal change required
2. [ ] Preserve all existing code unless explicitly removing
3. [ ] Keep original formatting/indentation
4. [ ] Add comments for non-obvious changes

### ✅ Phase 4: Validation
1. [ ] Run affected tests: `pytest tests/test_<module>.py -v`
2. [ ] Check syntax: No errors in `get_errors`
3. [ ] If GUI: run `python scripts/smoke_test_gui.py`
4. [ ] Visual inspection: does change make sense?

---

## 🚫 Critical Anti-Patterns (NEVER DO)

### ❌ Incomplete Function Edits
```python
# BAD - Missing function closing
def show_dialog(self):
    field = ft.TextField(...)
    # WHERE IS THE REST? AlertDialog? page.update()?
```

### ❌ Multi-Issue Commits
```
BAD: "Fix multi-prep, refactor dashboard, fix floating point"
     (touching 3 files, 200+ lines)
```

### ❌ Blind Copy-Paste Refactoring
```python
# BAD - Copying code without understanding
# Might miss dependencies, break references
```

### ❌ Editing Without Full Context
```python
# BAD - Only reading 20 lines of a 100-line function
# Risk: accidentally remove critical code
```

---

## 🔍 Project-Specific Rules

### GUI Modifications (`gui.py`)

**CRITICAL**: All `show_*_dialog()` functions MUST:
1. Create `ft.AlertDialog` object
2. Add to `self.page.overlay.append(dialog)`
3. Set `dialog.open = True`
4. Call `self.page.update()`

**Before editing any dialog function:**
```python
# Verify complete pattern exists:
grep_search("def show_add_preparation_dialog", includePattern="gui.py")
# Read ENTIRE function - not just first 50 lines!
read_file("gui.py", offset=<start>, limit=150)  # Get full function
```

**After editing:**
```powershell
pytest tests/test_gui_dialogs.py::test_add_preparation_dialog_has_alertdialog -v
```

### Database Modifications

**ALWAYS:**
1. Create migration script in `migrations/`
2. Test on development DB first: `data/development/`
3. Document in migration comment what changed
4. Provide rollback SQL
5. Run `scripts/check_admin_columns.py` after schema changes

**NEVER:**
1. Modify production DB directly
2. Delete columns without migration
3. Change column types without data conversion plan

### Backend Logic (`peptide_manager/`)

**Floating Point Operations:**
```python
# ALWAYS use explicit rounding for volumes/money
volume = round(calculated_volume, 2)  # 2 decimals for ml
# SQL: ROUND(volume_ml, 2)
```

**Multi-Prep Distribution:**
- Use FIFO by `preparation_date` (not expiry_date)
- Always check `volume_remaining_ml > 0.01`
- Consider `wastage_ml` in calculations

### Testing Strategy

**Test Pyramid:**
1. **Unit Tests** (`tests/test_*.py`): Test individual functions
2. **Integration Tests** (`scripts/integration_smoke.py`): Test workflows
3. **Smoke Tests** (`scripts/smoke_test_gui.py`): Test GUI opens/works

**Before Commit:**
```powershell
# Minimum
pytest tests/ -v

# If GUI changed
python scripts/smoke_test_gui.py

# If backend changed
pytest tests/test_database.py tests/test_calculator.py -v

# Full suite (before push)
pytest tests/ -v --cov=peptide_manager --cov-report=term
```

---

## 🛠️ Workflow for Bug Fixes

### Step-by-Step Process:

1. **Understand the Bug**
   ```
   - Read user report carefully
   - Reproduce bug if possible
   - Identify root cause with grep_search/read_file
   ```

2. **Plan the Fix**
   ```
   - Identify exact file(s) and functions to modify
   - Check if fix requires tests update
   - Determine if breaking change
   ```

3. **Implement Minimally**
   ```python
   # Change ONLY what's needed
   # Keep everything else intact
   # Add TODO if refactoring needed later
   ```

4. **Validate Thoroughly**
   ```powershell
   # Run tests
   pytest tests/ -v
   
   # Check no new errors
   get_errors
   
   # If GUI: smoke test
   python scripts/smoke_test_gui.py
   ```

5. **Commit Atomically**
   ```bash
   git add <single_file>
   git commit -m "Fix: specific bug description"
   # Pre-commit hook validates automatically
   ```

---

## 🎓 Learning from Past Incidents

### Incident: Commit 9817859 (Nov 30, 2025)

**What happened:**
- During multi-prep fix, `show_add_preparation_dialog()` lost last 13 lines
- AlertDialog creation code deleted accidentally
- Bug undetected until production use

**Root causes:**
1. ❌ Too many changes in one commit (3 files, 176 lines deleted)
2. ❌ No test coverage for dialog completeness
3. ❌ Incomplete function read during refactoring
4. ❌ No smoke test before commit

**Preventions NOW in place:**
1. ✅ Test: `tests/test_gui_dialogs.py` checks all dialogs complete
2. ✅ Pre-commit hook validates dialog integrity
3. ✅ Smoke test script for GUI validation
4. ✅ This document for future reference

**How to prevent:**
- Always read ENTIRE function before editing
- Run `python scripts/smoke_test_gui.py` before commit
- Split large refactors into multiple commits
- Let pre-commit hook validate (don't use `--no-verify`)

---

## 📞 When to Ask User

**ALWAYS ask before:**
- Deleting any function/class/method
- Changing function signatures (breaking change)
- Modifying database schema
- Refactoring >100 lines
- Changing behavior of existing features
- If uncertain about code purpose

**Questions to ask:**
- "Should I split this into multiple commits?"
- "This function seems unused, should I verify before deleting?"
- "This change affects X, Y, Z - proceed with all or separately?"

---

## 🚀 Command Reference

### Essential Commands
```powershell
# Test all
pytest tests/ -v

# Test specific module
pytest tests/test_gui_dialogs.py -v

# Smoke test GUI
python scripts/smoke_test_gui.py

# Check for errors
get_errors

# Search for code usage
grep_search "function_name" --includePattern "**/*.py"

# Install/reinstall hooks
python scripts/install_hooks.py
```

### Git Workflow
```bash
# Before starting work
git status
git pull

# Atomic commits
git add <single_file>
git commit -m "Fix: specific issue"
# Hook runs automatically

# Emergency bypass (use sparingly)
git commit --no-verify -m "Emergency fix"

# Check what changed
git diff
git diff --staged
```

---

## 📚 Key Files Reference

- `gui.py` - Main Flet GUI (CAREFUL: all dialogs must be complete)
- `peptide_manager/__init__.py` - Main adapter class
- `peptide_manager/models/` - Data models and repositories
- `tests/test_gui_dialogs.py` - Dialog integrity tests
- `scripts/pre_commit_check.py` - Pre-commit validation
- `scripts/smoke_test_gui.py` - GUI smoke tests
- `DEVELOPMENT_GUIDE.md` - Full development workflow
- `LESSONS_LEARNED.md` - Past incidents and learnings

---

## ✅ Success Criteria

A change is ready for commit when:

1. ✅ All tests pass: `pytest tests/ -v`
2. ✅ No syntax errors: `get_errors` returns clean
3. ✅ Pre-commit hook passes (or intentionally bypassed with reason)
4. ✅ Change is minimal and focused
5. ✅ Commit message is clear and specific
6. ✅ User approved if breaking change
7. ✅ Documentation updated if API changed

---

## 🎯 Remember

> **"Make the change you intend, and ONLY that change."**

> **"Test before commit, ALWAYS."**

> **"When in doubt, ask the user."**

> **"One problem, one commit, one test run."**

---

*Last updated: Dec 3, 2025 after commit 9817859 regression analysis*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Freetaban) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
