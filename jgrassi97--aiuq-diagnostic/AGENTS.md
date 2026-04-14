
# Pre-Commit Automation Workflow

Whenever you mention **commit**, **push**, **review**, or **pre-commit** in the context of this AIUQ repository, follow this workflow:

## Automatic Workflow Trigger

### Phase 1: Pre-Commit Check
1. Run `git status --short` to detect modified files
2. For each unstaged file, generate `git --no-pager diff --` to analyze changes
3. For each staged file, generate `git --no-pager diff --staged --` for review
4. Run error/linting checks on touched files to catch syntax issues

### Phase 2: Code Review
- **High Priority**: Flag breaking changes, unsafe patterns, or logic regressions
- **Medium Priority**: Note style inconsistencies, performance concerns, missing tests
- **Info**: Document new features, refactorings, or dependency changes
- **Edge Cases**: Identify potential issues with restart/resume logic, empty states, race conditions

### Phase 3: Notion Update - Commit Check
- Identify the **current branch** (e.g., `features/amip-simulations`)
- Search Notion for a page matching the branch name or context (e.g., "AMIP Simulation")
- If found: create a sub-page titled `Commit Check - [DATE] ([BRANCH])`
- Document:
  - Files changed (count + paths)
  - Main changes summary
  - Pre-commit findings (risks, suggestions, info)
  - Suggested commit message
  - Follow-up items

### Phase 3b: Notion Update - Release Notes (Post-Commit)
*Only after commit is successful and staged*
- Determine the current branch before any changelog action
- If the current branch is `main` or `master`: do not create, update, or append any changelog / release notes entry
- If the current branch is not `main` / `master`: search Notion for the release notes page specific to that branch context under the branch's parent (e.g., "Release Notes - AMIP Simulations" under "AMIP Simulation")
- If found: append a new entry at the top of "Latest Releases" section with:
  - **Date**: Current date in `[YYYY-MM-DD]` format
  - **Title**: The commit message
  - **Commit hash**: Short hash (7 chars) from `git log -1 --format=%h`
  - **Changes**: Bullet-point summary from review findings
  - **Impact**: One-liner about what this enables/fixes
  - **Breaking Changes**: Yes/No with details
- If Release Notes page not found: create one under the branch parent with the template structure
- Never write a feature-branch commit into a generic `main` changelog, and never maintain a changelog while operating directly on `main`

### Phase 4: Suggestions & Approval
Before the user pushes:
1. Present a summary of findings
2. Suggest fixes or improvements with concrete locations (file + line range)
3. **Ask for explicit confirmation** before allowing Push:
   - "Ready to push? (yes/no)"
   - If fixes suggested: "Apply fixes? (yes/no)"
4. Only proceed with `git push` after user confirms

---

## Details

### Code Review Focus
- **Deterministic vs Non-deterministic code**: Check for hardcoded paths, missing env vars, or unsafe defaults
- **State management**: Verify checkpoint resume paths, cleanup logic, and edge cases
- **API changes**: Surface breaking changes to function signatures, config keys
- **External dependencies**: Flag new imports and check for version constraints

### Notion Structure
- **Parent**: Auto-detect from current branch (e.g., "AMIP Simulation" for `features/amip-simulations`)
- **Release Notes scope**: Must match the active branch context; do not reuse `main` or unrelated branch changelogs
- **Format**: Markdown with code blocks, bullet lists, and links to files/lines
- **Auto-fix notes**: Mark sections as "[DONE - automatic fix]" if fixes were already applied

### Git Commands Used
```bash
git status --short
git --no-pager diff --stat
git --no-pager diff [--staged] -- <file>
git --no-pager log -n 5 --oneline
git push origin $(current_branch)
```

### Skip Conditions
- If no files are staged or unstaged: report "No changes detected"
- If `.gitignore` is the only change: optionally skip Notion update (lightweight)
- If the current branch is `main` or `master`: skip changelog / Release Notes updates entirely
- If user explicitly says "skip review": proceed to push without workflow

---

## Example Interaction

**User**: "I'm ready to commit the AMIP changes"

**Agent** (automatically):
1. ✓ Detects 3 files modified (runscripts/sim_neuralgcm.py, templates/config.yml, .gitignore)
2. ✓ Analyzes diffs → finds edge case in checkpoint resume logic
3. ✓ Creates page "Commit Check - 2026-03-18 (features/amip-simulations)" in Notion under AMIP Simulation
4. ✓ Suggests fix with line numbers
5. ✓ Updates the branch-specific Release Notes page with commit entry (post-commit)
6. 📋 Asks: "**Apply fix to checkpoint edge case?** (yes/no)"
7. 📋 Asks: "**Ready to stage & push?** (yes/no)"

---

## Behavior Notes

- Always operate within the AIUQ workspace repository
- Use Notion integration for all documentation (never suggest external tools)
- **Release Notes**: Maintained separately from Commit Checks for merge/changelog purposes
  - Populated automatically after each successful commit only for non-`main` branches
  - Must be specific to the active branch context
  - Used as source for PR descriptions and version tagging
- If Notion page not found: create a new one under "Commit Logs" or ask user where to put it
- Keep notes concise: focus on actionable items, not verbose prose
- Mark manual user decisions clearly (e.g., "User chose not to apply fix")

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JGrassi97) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
