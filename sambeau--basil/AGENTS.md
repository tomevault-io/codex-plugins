# Copilot Instructions for Basil

## Overview
Basil is a Go web server for the Parsley programming language.

## Before Any Task
1. **Check `git status` for uncommitted work** — commit or stash before starting new work
2. Read `AGENTS.md` at the repository root — it contains build commands, project structure, and workflow rules
3. Read `work/reports/specifications-guide-for-implementors.md` before changing behavior, fixing bugs, or optimizing code that may have historical/design context
4. Check `work/BACKLOG.md` for related deferred items
5. Use the appropriate prompt file for your task type

## ⚠️ Critical: No Test Failures May Be Ignored

**All tests must pass before any commit. There are no acceptable pre-existing failures.**

- **Do NOT label a failing test "pre-existing" and move on.** If a test fails, fix it.
- **Do NOT assume a failure is unrelated to your change.** Even if it appears unrelated, fix it before committing.
- **Do NOT commit with a known failing test under any circumstances**, even in code you did not touch.
- If you cannot fix a failing test yourself, **stop and tell the user** before committing. Do not work around it or suppress it.

The only acceptable state before a commit is: `ok` for every package in `go test ./...`.

---

## ⚠️ Critical: Preserve Work Through Commits

**Work has been lost due to incomplete commits.** Follow these rules:

### Before Starting New Work
- Run `git status` — if there are uncommitted changes from previous work:
  - If tests pass → commit with an appropriate message
  - If tests fail → stash and inform the user
  - Never start new work on top of uncommitted changes from a different feature

### During Implementation
- Commit at logical checkpoints (new function, passing tests, before risky changes)
- When tests pass after a change, commit

### After Completing a Feature/Fix
1. Run `make test` or `go test ./...`
2. Run `make bench-compare` to check for performance regressions
3. If tests pass → **commit all related changes immediately**
4. A feature isn't done until it's committed

### ⚠️ Post-Feature Benchmark Requirement
After implementing any FEAT, run benchmarks to detect regressions:

1. Run `make bench-save` to capture post-implementation performance
2. Run `make bench-compare` to check against baseline
3. Include performance summary in implementation notes:
   - Flag regressions > 5% for human review
   - Note improvements > 5%
   - State "No performance regression detected" if stable

Skip benchmarks for: documentation-only changes, test-only changes, config changes.

## Writing Parsley Code
Before writing any Parsley code (handlers, tests, examples):
- Read `.github/instructions/parsley.instructions.md` for syntax rules
- Key points: tags don't need quotes, singleton tags MUST be self-closing (`<br/>` not `<br>`), use `{var}` for interpolation (not `${var}`)

### ⚠️ Critical: Parsley Uses Expression-Based Output
**`print()` does NOT exist in Parsley!** Values ARE the output:
```parsley
// ❌ WRONG — These will error!
print("hello")
println("hello")

// ✅ CORRECT — The value IS the output
"hello"
<div>"content"</div>

// ✅ For debugging, use log()
log("debug:", someVar)
```

## Debugging and Testing Parsley Code
Use `pars -e` to quickly test and debug Parsley expressions:
- Outputs PLN (Parsley Literal Notation) format by default, showing structure
- Examples:
  - `pars -e "[1, 2, 3]"` → outputs `[1, 2, 3]`
  - `pars -e '"hello"'` → outputs `"hello"`
  - `pars -e "{a: 1, b: 2}"` → outputs `{a: 1, b: 2}`
- Use `--raw` or `-r` for file-like output (e.g., for HTML rendering)
- Matches REPL behavior for consistency

## Parsley API Reference (Source of Truth)
**Always use `pars describe` to get accurate API information.** Documentation may be outdated; the code is truth.

### Looking Up Methods and Builtins
```bash
# Get all methods for a type
pars describe string
pars describe array
pars describe dict
pars describe integer
pars describe float

# Get info about a specific builtin
pars describe len
pars describe range
pars describe match

# Get complete API as JSON (for programmatic use)
pars describe all --json
```

### Verifying Code Before Writing
Before including Parsley code in documentation or examples:
```bash
# Test that an expression works
pars -e '"hello".toUpper()'

# Test HTML output
pars -r -e '<div class="test">"content"</div>'

# Check if a method exists
pars describe string | grep -i "trim"
```

### Key Principle
If documentation conflicts with `pars describe` output, **trust the command**. Update the documentation to match.

## Specification Rationale and Earlier Decisions
Before changing behavior — especially for bug fixes, refactors, performance work, caching changes, auth changes, parser/evaluator semantics, or removing code that looks unnecessary — first look for the earlier decision and rationale.

Start here:
- `work/reports/specifications-guide-for-implementors.md` — high-level map of the spec set, where rationale lives, and how to interpret older vs newer specs
- `work/specs/` — intended behavior and acceptance criteria
- `work/plans/` — implementation reality, phased delivery, and deviations from the original spec
- `work/design/` — tradeoffs and architecture reasoning
- `work/reports/` — audits, investigations, performance analysis, and cleanup rationale
- `CHANGELOG.md` — later breaking changes, removals, and consolidation decisions

### Why this matters
Code that looks redundant, slow, awkward, or over-engineered may be preserving an important semantic, security boundary, compatibility decision, or previously investigated edge case. Do not \"simplify\" or \"optimize\" it away until you understand why it exists.

### Minimum rule for decision-making
When making a non-trivial change, determine:
1. which spec owns the behavior
2. whether a later spec or cleanup changed that contract
3. whether the plan documents a different implementation from the original spec
4. whether a design doc or report explains the tradeoff
5. whether the change crosses the Basil/Parsley boundary

If you can't answer those questions yet, read more before changing the code.

## Workflow Entry Points
- **New Feature**: Use `/new-feature` prompt
- **Bug Fix**: Use `/fix-bug` prompt  
- **Release**: Use `/release` prompt

## Key Conventions
- Features: `FEAT-XXX` in `work/specs/`
- Bugs: `BUG-XXX` in `work/bugs/`
- Plans: `work/plans/`
- IDs: Managed via `work/ID_COUNTER.md`

## Git Rules
- AI commits to feature/bug branches
- AI merges to main after human approval
- AI can push to remote when delegated by human
- Human creates release tags
- Use Conventional Commits format:
  - `feat(scope): description` — New features
  - `fix(scope): description` — Bug fixes  
  - `test(scope): description` — Test changes
  - `docs(scope): description` — Documentation
  - `chore(scope): description` — Maintenance
  - Add `!` for breaking changes: `feat(parsley)!: description`

## ⚠️ Critical: Features Require Tests

**A feature is not complete until it has tests and the tests pass.**

### Definition of Done
1. Implementation code is written
2. Tests are written that exercise the new functionality
3. All tests pass (`go test ./pkg/parsley/...`)
4. Benchmarks checked for regressions (`make bench-compare`)
5. Changes are committed

### Test Requirements
- **New language features**: Add tests to `pkg/parsley/tests/` (integration tests)
- **New methods**: Test both success cases and error cases
- **Bug fixes**: Add regression test that would have caught the bug
- **Edge cases**: Test boundary conditions, empty inputs, error paths

### Running Tests
```bash
# Run all Parsley tests
go test ./pkg/parsley/...

# Run with coverage for evaluator
go test -coverprofile=cov.out -coverpkg=./pkg/parsley/evaluator ./pkg/parsley/...

# Run specific test file
go test ./pkg/parsley/tests -run TestName
```

### What Counts as Tested
- Feature works for typical use cases
- Error conditions return appropriate errors (not panics)
- Edge cases are handled (empty arrays, null values, etc.)

### Do NOT
- Merge or mark complete any feature without tests
- Skip tests "to save time" — untested code causes more work later
- Write tests that only test the happy path

## Documentation
- Update `docs/guide/faq.md` when answering "how do I..." questions
- Add deferred items to `work/BACKLOG.md`

## Parsley Documentation
When documenting Parsley language features:
- When writing manual pages: see `.github/templates/DOC_MAN_BUILTIN.md` and `.github/templates/DOC_MAN_STD.md`
- Manual pages go in `docs/parsley/manual/builtins/` (types) or `docs/parsley/manual/stdlib/` (modules)
- `docs/parsley/reference.md` - Comprehensive reference. All features should be documented here with accurate grammar snippets
- `docs/parsley/CHEATSHEET.md` - AI-focused cheatsheet highlighting differences from other languages, ordered by likelihood of being a pitfall
- `docs/parsley/README.md` - Quick guide with examples (may be outdated)

## Basil Documentation
When documenting Basil server features (image transforms, auth, sessions, database, etc.):
- When writing manual pages: see `.github/templates/DOC_MAN_BASIL.md`
- Manual pages go in `docs/basil/manual/`
- `docs/basil/reference.md` - Comprehensive reference for all Basil server extensions
- Basil features require the server environment — they error in standalone `pars` or the REPL
- Include the `> **Basil-only.**` callout near the top of each manual page

### Parsley vs Basil manual pages
- **Parsley** manual = the language: types, operators, stdlib — works in `pars` standalone → `docs/parsley/manual/`
- **Basil** manual = the server framework: images, auth, sessions, database connections — requires Basil server → `docs/basil/manual/`

### Documentation Accuracy Rules
1. **Verify examples work** — run `pars -e "code"` before committing doc changes
2. **Check method existence** — use `pars describe <type>` to confirm methods exist
3. **Don't invent builtins** — `print()`, `println()`, `printf()` do NOT exist
4. **Code is truth** — if docs conflict with `pars describe`, fix the docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sambeau)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sambeau)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
