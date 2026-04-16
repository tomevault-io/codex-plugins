## Quick orientation for AI coding agents

This repository is a Streamlit app that creates RPG characters from PDF reference materials (D&D and "Ordem Paranormal"). The UI sits in `main.py`; PDF parsing and search helpers are implemented in `functions.py`, `dndfunctions.py`, and `opfunctions.py`. Reference PDFs live under `assets/` and there is a lightweight parser repro in `scripts/test_extract.py` plus a pytest in `tests/test_extract_pytest.py`.

Essential facts
- Entry point: `main.py` — run with Streamlit for interactive work: `streamlit run main.py`.
- PDF-first parsing: code tries `pdfplumber.Page.extract_tables()` first; when that fails it converts pages to images and uses `pytesseract` OCR. Look for `.to_image()` and `.original` patterns.
- Class mapping: `dndfunctions.class_to_file_map` maps display names (often accented) to files in `assets/`. Update that map when adding/removing class PDFs.
- OCR defaults: many calls use Portuguese (`lang='por'`) — prefer `por` by default when editing OCR behavior.

Key functions & contracts
- `choose_dnd_class()` (in `dndfunctions.py`): streamlit chooser; when invoked programmatically it returns `(character_class, class_to_file_map)`; otherwise it renders UI.
- `extract_table_page_2(pdf_path, ocr_lang='por', ocr_resolution=300)` -> list[list[str]] | None: returns rows (header optional) or None. UI callers expect page numbers in human-facing 1-indexed form.
- `_ensure_tesseract()` respects `SKIP_OCR=1|true` to opt out of OCR in CI or on runners without system Tesseract.

Project conventions to preserve
- Page indexing: UI and returned page numbers are 1-indexed; internal PdfReader/pages loops are 0-based. Keep this mapping.
- Encoding: filenames and keys contain accents (UTF-8). Don’t normalize/strip accents when modifying filenames or keys.
- Testing: parsing fixes should include a small repro in `scripts/test_extract.py` or an extra assertion in `tests/test_extract_pytest.py` so reviewers can verify behavior without running the full UI.

Where to look first (recommended order)
- `dndfunctions.py` — class mapping, extraction helpers, and table heuristics (header-slicing, fallback split on multi-space).
- `functions.py` — bookmark/header search helpers, OCR integration utilities.
- `main.py` — UI wiring and how chooser functions are called.
- `scripts/test_extract.py` / `tests/test_extract_pytest.py` — quick reproducer and CI test patterns.

Practical run & debug notes (Windows PowerShell)
- Create venv & install pinned deps (project includes `requirements.txt`):

```powershell
python -m venv .venv
& .\.venv\Scripts\python.exe -m pip install --upgrade pip
& .\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

- Run Streamlit UI:

```powershell
& .\.venv\Scripts\python.exe -m streamlit run main.py
```

- Quick parser repro:

```powershell
& .\.venv\Scripts\python.exe .\scripts\test_extract.py
```

Non-obvious patterns and gotchas
- Header-slicing heuristic: `extract_table_page_2` may compute fixed column start indices from a header line and slice subsequent OCR lines by character offsets. If OCR merges columns, the fallback is splitting on 2+ spaces.
- OCR can be flaky: use `SKIP_OCR` to skip OCR work in CI. Local debugging often requires installing the Tesseract system binary (not just the Python wrapper).
- Use `pypdf` (PdfReader) and `pdfplumber` together: `pypdf` is used for bookmarks/outline inspection; `pdfplumber` is used for page images and text/tables.

Minimal PR guidance for agents
- Make small, focused changes. When you alter parsing heuristics, add or update `scripts/test_extract.py` with a targeted print/assert so maintainers can reproduce the change quickly.
- Preserve Streamlit flow: chooser functions should return values when possible to allow unit testing instead of only side-effecting UI calls.

If anything here is unclear, tell me which section (OCR heuristics, run/debug steps, or where tests live) to expand and I will iterate.
## Quick orientation for AI coding agents

This repository is a small Streamlit-based Python app to help create RPG characters from PDF reference materials (D&D and "Ordem Paranormal"). The UI is in `main.py`; PDF parsing and helpers are in `functions.py` and `dndfunctions.py`. There are example assets under `assets/` and a small test harness in `scripts/test_extract.py`.

Key facts an agent should know (short):
- Entrypoint: `main.py` — a Streamlit app. Typical run: `streamlit run main.py`.
- PDF helpers: `functions.py` and `dndfunctions.py` implement most parsing logic. They prefer `pdfplumber` table extraction and fall back to `pytesseract` OCR.
- Class-to-PDF mapping lives in `dndfunctions.py` as `class_to_file_map` (look here when adding/removing class PDFs).
- Test harness: `scripts/test_extract.py` — imports project root and runs `extract_table_page_2` against an asset. Use it to reproduce parsing bugs.

Contract & important function shapes
- choose_dnd_class() (in `dndfunctions.py`): side-effecting Streamlit UI function. Returns `(character_class, class_to_file_map)` when invoked programmatically; otherwise it builds UI.
- extract_table_page_2(pdf_path, ...) -> list[list[str]] | None: returns rows (header row optional) or None on failure. Caller expects 1-indexed page numbers elsewhere.

Project-specific patterns and conventions
- PDF-first: functions try `pdfplumber.Page.extract_tables()` first; when that yields nothing they convert pages to images and run OCR via `pytesseract` (look for `.to_image()` and `.original` patterns).
- OCR language: many calls use Portuguese (`lang='por'`) in tests and functions — prefer `por` as a default when adding OCR-related fixes.
- Page indexing: when communicating page numbers back to UI, the code uses 1-indexed numbers; internal `PdfReader.pages` uses 0-based indexing. Preserve this mapping.
- Filenames contain accented characters (e.g., `D&D-Barbaro.pdf` with `Bárbaro` as a key). Use UTF-8-safe handling and avoid normalizing filenames unless necessary.

How to run locally (Windows PowerShell)
- Install dependencies inferred from imports (no requirements.txt present):

  pip install streamlit pdfplumber pytesseract pypdf pillow pandas

- Run the UI:

  streamlit run main.py

- Run the parser test (fast reproduce of extraction issues):

  python .\scripts\test_extract.py

Notes about debugging parsing
- Use `scripts/test_extract.py` to reproduce table extraction from a specific PDF. It prints the first extracted rows.
- Add `print()` or temporary `pdb.set_trace()` in `dndfunctions.basic_traits_table_extraction` or `extract_table_page_2` to inspect OCR output and header slicing.
- When OCR produces merged columns, inspect `lines` (the OCR text) and the header detection logic that slices fixed column positions based on keyword locations.

Integration and change points
- Adding new class PDFs: update `dndfunctions.class_to_file_map` and add the file in `assets/`.
- If adding a new RPG system, follow the `main.py` pattern: add a new selectbox option and call the corresponding chooser function.

Minimal PR guidance for agents
- Small, focused changes only. When changing parsing logic, add or update `scripts/test_extract.py` to include a targeted assertion (or at least a reproducible print-out) so maintainers can verify behavior.
- Preserve existing Streamlit UI flow — `main.py` wires selection -> chooser; prefer returning values from chooser functions so they can be unit-tested.

Files to inspect for examples
- `main.py` (UI wiring)
- `dndfunctions.py` (class mapping + extraction routines)
- `functions.py` (PDF search helpers)
- `scripts/test_extract.py` (parser reproduction harness)

If anything is ambiguous, ask these short questions:
1. Which RPG systems should be prioritized for bug fixes (D&D or Ordem Paranormal)?
2. Are we allowed to add a requirements.txt or pin dependency versions?

End of file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MattDSantosDev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
