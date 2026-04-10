# GitHub Copilot Agent Instructions

These instructions guide automated agents (GitHub Copilot, Claude, etc.) when working on this repository.

## Testing Requirements

### 1. Test Coverage for All Changes

**For any new feature or code change:**
- **MUST** include new tests or update existing tests
- Tests must cover the main functionality and edge cases
- Tests must be deterministic (no flaky tests)
- Tests must pass before finalizing any PR

**For bug fixes:**
- **MUST** include a regression test when feasible
- The regression test should fail on the old code and pass on the fixed code
- Document the bug scenario in the test docstring

### 2. Running Tests

**Before finalizing any PR:**
```bash
# Run all tests
pytest

# Run specific test categories
pytest -m unit          # Unit tests only
pytest -m integration   # Integration tests only
pytest -m regression    # Regression tests only

# Run tests with coverage
pytest --cov=app_mockup --cov-report=term

# Run demo tests (for human verification)
pytest -k demo_pipeline -s
```

**All tests MUST pass before submitting a PR.**

### 3. Test Organization

Tests are organized in the `tests/` directory:
- `test_preprocessing.py` - Tests for preprocessing pipeline (57 tests)
  - Sentence segmentation
  - Discourse marker detection
  - Candidate flagging
  - Integration tests
  - Negative/edge case tests
  - Golden/regression tests
- `test_llm_extractor.py` - Tests for main extraction pipeline
- `test_qa_module.py` - Tests for Q&A module (26 tests)
- `test_synthetic_claims.py` - Tests for synthetic claim generation
- `test_conclusion_inference.py` - Tests for conclusion detection
- `test_llm_integration.py` - Tests for LLM client integration
- `tests/live/` - Live API tests (opt-in only, cost real money)

### 4. Test Markers

Use pytest markers to categorize tests:
- `@pytest.mark.unit` - Unit tests for individual functions
- `@pytest.mark.integration` - Integration tests for full workflows
- `@pytest.mark.regression` - Regression tests with golden outputs
- `@pytest.mark.negative` - Edge cases and error conditions
- `@pytest.mark.demo` - Human-readable demonstration tests
- `@pytest.mark.live_api` - Live API tests (require explicit opt-in via RUN_LIVE_API_TESTS=1)

### 5. Test Quality Standards

**Tests must be:**
- **Deterministic**: Same input always produces same output
- **Isolated**: No side effects between tests
- **Fast**: Unit tests should run in milliseconds
- **Clear**: Descriptive names and docstrings
- **Maintainable**: Easy to understand and modify

**Avoid:**
- Network calls (mock external dependencies)
- Large model downloads (skip or mock if model not available)
- Random behavior (use fixed seeds if randomness needed)
- File system pollution (use temp directories, clean up)

**Live API tests:**
- The `tests/live/` directory contains tests that make real OpenAI API calls
- These tests are **skipped by default** and require explicit opt-in
- To run: `RUN_LIVE_API_TESTS=1 pytest -m live_api -s`
- Never run live tests in CI (they consume real credits)
- Live tests should be minimal and budget-safe

## ROADMAP.md Update Requirements

### 1. When to Update ROADMAP.md

**MUST update ROADMAP.md when:**
- Completing a roadmap task or subtask
- Making significant progress on a roadmap item
- Adding new planned work relevant to milestones
- Changing priorities or timelines

### 2. How to Update ROADMAP.md

**For completed tasks:**
- Change `- [ ]` to `- [x]` for completed checkboxes
- Add brief progress notes if significant (1-2 sentences)
- Update status indicators (✅ Complete, 🔄 In Progress, etc.)

**For new work:**
- Add tasks in the appropriate section
- Use clear, actionable task descriptions
- Include context about why the task is needed

**For changes to plans:**
- Document what changed and why
- Update priority markers if needed

### 3. ROADMAP Structure

The ROADMAP.md follows this structure:
- Milestone status table
- Workstream sections (UI/UX, Extraction Backend, Evaluation, etc.)
- Tasks with checkboxes and priority
- Progress notes and blockers

## IMPLEMENTATION_COMPLETE.md Requirements

### 1. When to Update IMPLEMENTATION_COMPLETE.md

**MUST update IMPLEMENTATION_COMPLETE.md when:**
- Completing any issue or PR
- Making significant implementation progress
- Finishing a feature or bug fix
- Completing work that changes system behavior

### 2. Standard Format for IMPLEMENTATION_COMPLETE.md

**ALWAYS rewrite/update the entire file with the following structure:**

```markdown
# [Issue Title] - COMPLETE ✅

**Issue:** [Issue number and title]  
**Date:** [YYYY-MM-DD]  
**Status:** ✅ Complete / 🔄 In Progress / ⚠️ Blocked

## Summary

[2-3 sentence overview of what was implemented]

## Changes Made

- [Bullet point of change 1]
- [Bullet point of change 2]
- [...]

## Key Files Changed

- `path/to/file1.py` - [Brief description]
- `path/to/file2.py` - [Brief description]
- [...]

## Testing

**How to test:**
```bash
# Commands to run tests
```

**Test results:**
- [Number] tests added/updated
- All tests passing ✓

## Verification

**To verify this implementation:**
1. [Step 1 to verify]
2. [Step 2 to verify]
3. [...]

## Follow-ups / Known Limitations

- [Any follow-up work needed]
- [Any known limitations or edge cases]
- [Future improvements planned]

## Documentation Updated

- [X] README.md
- [X] ROADMAP.md
- [X] Code comments/docstrings
- [X] Other: [specify]
```

### 3. What to Include

**Must include:**
- Issue title and number
- Completion date
- Clear summary of changes (bullets)
- List of key files modified/created
- Testing commands and results
- How to verify/validate the changes
- Any follow-up work or limitations

**Keep it:**
- **Concise**: 1-2 pages max
- **Scannable**: Use bullets and sections
- **Actionable**: Include commands to verify
- **Complete**: Don't omit important details

## Code Change Guidelines

### 1. Minimal Changes

- Make the **smallest possible changes** to achieve the goal
- Don't refactor unrelated code
- Don't fix unrelated bugs unless they block your work
- Keep commits focused and atomic

### 2. Code Quality

- Follow existing code style and patterns
- Add docstrings to new functions and classes
- Comment complex logic only when necessary
- Keep functions small and focused

### 3. Dependencies

- Avoid adding new dependencies unless absolutely necessary
- If adding a dependency, document why it's needed
- Check for security vulnerabilities before adding
- **When adding a new dependency:**
  - **MUST** update `requirements.txt` (for pip users)
  - **MUST** update `environment.yml` (for conda users)
  - Ensure version pins target the same version ranges (note: conda uses `=` or `.*` syntax while pip uses `>=,<`)
- **When changing Python version compatibility:**
  - **MUST** update the Python version in `environment.yml`
  - **MUST** update Python version references in README.md
  - **MUST** test with the new Python version before finalizing

### 4. Documentation

- Update README.md for user-facing changes
- Update relevant docs in `/docs/` for behavior changes
- Keep inline documentation clear and concise

## Pull Request Guidelines

### 1. Before Creating PR

**MUST complete these steps:**
1. ✅ Run `pytest` and ensure all tests pass
2. ✅ Add/update tests for your changes
3. ✅ Update ROADMAP.md if completing planned tasks
4. ✅ Update IMPLEMENTATION_COMPLETE.md with issue summary (see format above)
5. ✅ Update relevant documentation
6. ✅ Review your changes for code quality
7. ✅ Ensure no unintended files are committed

### 2. PR Description

Use the PR template and fill in:
- Clear description of changes
- Related issues (use "Fixes #123" for automatic linking)
- List of main changes
- Testing steps and results
- Documentation updates
- Checklist completion

### 3. Review Checklist

Before requesting review:
- [ ] Code follows project style
- [ ] Self-reviewed for quality
- [ ] Complex areas are commented
- [ ] No new warnings or errors
- [ ] Tests added/updated and passing
- [ ] Documentation updated
- [ ] ROADMAP.md updated (if applicable)
- [ ] IMPLEMENTATION_COMPLETE.md updated with issue summary

## Workflow for Agents

### Standard Development Flow

1. **Understand the task**
   - Read issue description carefully
   - Check related code and tests
   - Review ROADMAP.md for context

2. **Plan the changes**
   - Identify minimal changes needed
   - Plan test additions/updates
   - Check if ROADMAP.md needs updating

3. **Implement changes**
   - Write code changes
   - Add/update tests
   - Run tests frequently during development

4. **Validate changes**
   - Run full test suite: `pytest`
   - Run demo tests: `pytest -k demo_pipeline -s`
   - Manual verification if needed

5. **Update documentation**
   - Update ROADMAP.md if applicable
   - Update IMPLEMENTATION_COMPLETE.md with issue summary
   - Update other docs as needed
   - Review PR template requirements

6. **Finalize PR**
   - Ensure all tests pass
   - Complete PR description
   - Self-review checklist
   - Request review

### Error Handling

**If tests fail:**
- Read the failure message carefully
- Run the specific failing test: `pytest tests/test_file.py::test_name -v`
- Fix the issue or update the test if behavior changed intentionally
- Never skip or remove failing tests without understanding why

**If unsure about changes:**
- Ask for clarification in PR comments
- Document assumptions in code comments
- Add TODO comments for follow-up work

## Project-Specific Context

### Current Phase
- Post-Milestone 2 (Initial Mockup complete)
- Repository cleanup complete (2026-01-27): Removed ~5,000 lines of unused code
- Working toward Milestone 3 (Demo Video) and Milestone 4 (Final Prototype)

### Code Architecture & Hygiene

**Single Sources of Truth:**
1. **Extraction Pipeline**: `app_mockup/llm_extractor.py` is the only extraction implementation
   - Do NOT create alternative extraction pipelines
   - All extraction logic goes through llm_extractor.py
   - Uses OpenAI structured outputs API directly

2. **Graph Data Structures**: `app_mockup/backend/graph_construction.py`
   - Provides GraphNode and GraphEdge classes only
   - Do NOT duplicate these classes elsewhere

3. **Q&A Module**: `app_mockup/backend/qa_module.py`
   - Single Q&A implementation
   - Do NOT create alternative Q&A systems

4. **LLM Client**: `app_mockup/backend/llm_client.py`
   - Single LLM client with caching and budget tracking
   - All LLM calls should go through this client

**Before Adding New Modules:**
- Check if functionality already exists
- If it exists, extend the existing module instead of creating a new one
- If you must create a new module, document why in PR description
- Remove deprecated modules when adding replacements

**Preventing Code Duplication:**
- Do NOT create multiple implementations of the same functionality
- Do NOT create "v2" modules alongside old modules - replace the old one
- Do NOT keep unused "example" or "prototype" files in the main codebase
- Move deprecated code to git history, not to `archive/` folders

### Key Components
1. **Preprocessing** (✅ Complete, ✅ Well-tested)
   - Sentence segmentation
   - Discourse marker detection
   - Candidate flagging

2. **Extraction** (✅ Implemented in llm_extractor.py)
   - LLM-based component classification
   - LLM-based relation extraction
   - Synthetic claim generation
   - Conclusion inference

3. **Q&A** (✅ Implemented)
   - Graph-grounded question answering
   - Chat memory

4. **UI/Frontend** (✅ Basic implementation)
   - Streamlit app (app.py)
   - Graph visualization
   - Node details panel

### Testing Philosophy
- **Comprehensive**: Cover main paths and edge cases
- **Maintainable**: Easy to understand and update
- **Fast**: Quick feedback during development
- **Reliable**: Deterministic, no flaky tests
- **Human-readable**: Demo tests for sanity checking

## Questions or Issues?

If these instructions are unclear or don't cover your use case:
1. Check existing code for patterns
2. Look at recent PRs for examples
3. Add a comment to your PR asking for guidance
4. Don't guess - it's better to ask than to make wrong assumptions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EyalKotlik)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/EyalKotlik)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
