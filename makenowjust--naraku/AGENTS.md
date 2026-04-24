# Copilot instructions for Naraku

Naraku is a Ruby/Onigmo-compatible regular expression engine implemented in C.

## Project structure

- `include/`
  - Public headers (`naraku_syntax.h`, `naraku_error.h`, encoding interfaces, node definitions).
- `src/`
  - Core implementation:
    - parser/lexer: `src/parse.c`
    - errors/warnings: `src/error.c`
    - encodings: `src/encoding/*.c`
    - node handling: `src/node.c`
- `mrbgems/mruby-naraku/`
  - mruby binding layer:
    - C bridge: `src/mrb_naraku*.c`
    - mruby wrapper classes/modules: `mrblib/naraku/*.rb`
- `ruby-prototype/`
  - Ruby-side prototyping code (VM/compiler experiments and bridge utilities).
  - `ruby-prototype/lib/mruby-scripts/`: scripts executed by mruby.
  - `ruby-prototype/lib/naraku_ruby/`: CRuby-side helper code for prototype workflow.
  - `ruby-prototype/test/`: prototype tests (run via top-level Rake task).
- `tests/`
  - mruby-based tests (entrypoint: `tests/run_test.rb`)
  - custom framework: `Mtest`

## Current implementation status

- Implemented:
  - encoding layer (multiple encodings)
  - parser/lexer and AST construction
  - parser warning callback pipeline to mruby
- In progress:
  - regex VM/compiler prototyping in Ruby
- Not implemented yet:
  - production VM/compiler in C

## Build and test commands

- List tasks:
  - `bundle exec rake -T`
- Build mruby integration:
  - `bundle exec rake naraku:build_mruby`
- ASan build:
  - `bundle exec rake naraku:build_mruby_asan`
- mruby tests:
  - `bundle exec rake naraku:test_mruby`
  - verbose: `bundle exec rake naraku:test_mruby_verbose`
- Ruby prototype test:
  - `bundle exec rake ruby_prototype:test`
- Lint:
  - `bundle exec rake lint`
- Format (C + Ruby):
  - `bundle exec rake format`

## Parser/lexer implementation guidance

- Prefer Onigmo-compatible behavior unless divergence is explicitly requested.
- Error reporting quality is important:
  - set both `offset` and `length` (span),
  - use meaningful ranges when possible, not only point offsets.
- For new parser errors/warnings:
  - update enums in `include/naraku_error.h`,
  - add message in `src/error.c`,
  - add/adjust parser tests in `tests/test_parser.rb`.
- Keep explicit boundary checks at call sites (`parser->pattern_bytes < parser->pattern_bytes_end`) where context-specific errors are required.
- `peek()` includes a defensive end-of-pattern check as a fallback safety net.
- When adding parsing branches that inspect a “next token” (options/groups/char-class/escapes), ensure end-of-pattern is handled before `peek()`/`consume()`.

## mruby binding guidance

- `Naraku::Parser.new` options are wired through:
  - `mrblib/naraku/parser.rb` -> `_new`
  - `src/mrb_naraku_parser.c` -> `nk_parser_options_t`
- In `bin/mruby`, `Naraku` is built-in; do not call `require 'naraku'`.
- Keep Ruby Proc callbacks alive (e.g., warning callback stored on parser object) to avoid GC issues.
- Free C-side user data correctly in parser free hooks.
- Parse errors exposed to Ruby should preserve `offset`/`length` from parser error span fields.

## Testing expectations

- Add regression tests for every parser bug fix (especially span and boundary bugs).
- For warning behavior, test both message and `(offset, length)`.
- Keep tests mruby-compatible; do not assume CRuby-specific behavior.

---
> Source: [makenowjust/naraku](https://github.com/makenowjust/naraku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
