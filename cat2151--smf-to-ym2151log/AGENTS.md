## Project snapshot

This repository converts Standard MIDI Files (SMF) to a YM2151 register-write log JSON used by `ym2151-zig-cc`.

Key elements:
- Pass A (MIDI -> events): `src/midi_parser.py` (entry via `parse_midi_file`) — produces an `events` list and metadata (`ticks_per_beat`, `tempo_bpm`).
- Pass B (events -> YM2151): `src/ym2151_converter.py` (entry via `convert_to_ym2151_log`) — consumes the Pass A structure and emits YM2151 register writes.
- Helpers: `src/midi_utils.py` (note -> KC/KF, tick/sample conversions) and `src/ym2151_init.py` (channel initialization events).

## Critical runtime conventions (concrete rules for code edits)

- Event shape (Pass A -> Pass B): each event is a dict. Examples in code/tests:
  - tempo: `{ 'type': 'tempo', 'ticks': <int>, 'tempo_bpm': <float> }` (`src/midi_parser.py`)
  - note_on: `{ 'type': 'note_on', 'ticks': <int>, 'channel': <int>, 'note': <int>, 'velocity': <int> }`
  - note_off: `{ 'type': 'note_off', 'ticks': <int>, 'channel': <int>, 'note': <int> }`

- YM2151 output format (Pass B output JSON):
  - Top-level: `{ 'event_count': <int>, 'events': [ {time, addr, data}, ... ] }` (`README.md`, tests)
  - `time` is integer sample count (sample rate 55930 Hz is used by default in `midi_utils.ticks_to_samples`).
  - `addr` and `data` are hex strings like `'0x08'` or `'0xC7'` (see `src/ym2151_init.py` and `src/ym2151_converter.py`).

## Architecture and data flow (quick mental model)

1. CLI (`smf_to_ym2151log.py`) calls `parse_midi_file` (Pass A). Output saved as `<base>_events.json` for debugging.
2. CLI passes that structure to `convert_to_ym2151_log` (Pass B). Output saved as `<base>_ym2151.json`.
3. `midi_utils` converts ticks -> seconds -> samples (55930 Hz default) and MIDI note -> YM2151 KC/KF. `ym2151_init` builds channel init register writes.

When editing conversion logic, always keep the two-pass model intact: unit tests rely on Pass A producing the canonical intermediate structure.

## Project-specific patterns and gotchas

- Files are intentionally small (~100 lines) and focused; prefer adding a new small module rather than expanding one huge file.
- The converter currently maps all notes to YM2151 channel 0 (mono). If you change channel assignment, update both `active_notes` logic in `src/ym2151_converter.py` and tests that assume mono.
- `midi_utils.midi_to_kc_kf` clamps octave to 0-7 and subtracts 1 from MIDI note for octave alignment — changing this will shift many tests and output JSON addresses.
- `addr`/`data` are formatted as `'0x{val:02X}'` strings. Maintain this formatting for compatibility with downstream `ym2151-zig-cc` consumers and tests.

## Tests and developer workflows

- Install deps: `pip install -r requirements.txt` (repo uses `mido`).
- Run tests: `python -m pytest tests/ -v` (used in README and CI). Unit tests validate both Pass A and Pass B shapes and a small happy-path conversion.
- Quick local run: `python smf_to_ym2151log.py <file.mid>` — produces `<base>_events.json` and `<base>_ym2151.json`.

## Examples to cite when making edits

- Add a tempo-handling change: update `src/midi_parser.py` to emit tempo events and `src/ym2151_converter.py` to read `event['tempo_bpm']` (search both files for `tempo`).
- Change sample rate: update `src/midi_utils.py` functions `seconds_to_samples` / `ticks_to_samples` and verify tests that assert integer `time` values in `tests/test_ym2151_converter.py`.

## Integration points and external expectations

- Downstream: output JSON is consumed by `ym2151-zig-cc` (see README). Keep `addr`/`data` hex strings and sample-time integer semantics.
- CI: there are GitHub Actions workflows under `.github/workflows/` — they assume tests are runnable via `pytest` and README examples.

## When to ask humans / not to change automatically

- Don't change the two-pass file outputs or event shapes without coordination — tests and downstream tools expect these exact fields.
- If you need to change YM2151 operator defaults in `ym2151_init.py`, open an issue so maintainers can confirm sonic expectations.

---
If any section is unclear or you want additional examples (e.g., common refactors or style choices), tell me which area to expand and I'll iterate.

# userからの指示
- commit前にruff formatとruff check --fixを実行する
- 作業報告は、プルリクエストのコメントに書く。document作成禁止
  - DRY原則に準拠し、「codeやbuild scriptと同じことを、documentに書いたせいで、そのdocumentが陳腐化してハルシネーションやuserレビューコスト増大や混乱ほか様々なトラブル原因になる」を防止する
  - なおissue-notes/は、userがissueごとの意図を記録する用途で使う

---
> Source: [cat2151/smf-to-ym2151log](https://github.com/cat2151/smf-to-ym2151log) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-10 -->
