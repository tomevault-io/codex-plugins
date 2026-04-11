# Project Instructions for GitHub Copilot

## Code Quality Checks

- Always run `rake test` after making changes if there are tests
- Check Ruby syntax with `ruby -c` for modified files
- Ensure proper 2-space indentation
- Make sure that the server can still run by executing `./bin/dev` after changes
- When testing endpoints which require exchange and/or symbol, use 'us' and 'aapl' as test values

## Zeitwerk Conventions

- Follow Zeitwerk conventions in naming inside `app/`
- Files in `app/config/` map to `App::Config::*` namespace
- There should be only one Zeitwerk dir, `app/`, meaning no subdirs

## Before Completing Tasks

- [ ] Run syntax check on all modified Ruby files
- [ ] Run full test suite
- [ ] Verify no references to old class names remain

## Naming Conventions

- Use snake_case for file names
- No `_util` or `_helper` suffixes (namespace provides context)
- File names should not be redundant, meaning use `routes/root.rb` instead of `routes/root_routes.rb`
- Test files mirror source files: `date.rb` → `date_test.rb`
- Test dirs mirror source dirs: `app/config/config.rb` → `test/config/config_test.rb`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrzli)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mrzli)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
