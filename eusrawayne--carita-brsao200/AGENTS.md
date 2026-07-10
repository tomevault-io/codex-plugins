This repository contains small Python exercise scripts organized into two practice folders.

- **Structure**: `atividades-pratica01/` and `atividades-pratica02/` contain individual exercise scripts named `questao-XX.py` (note: one file is named `questao03.py` without the hyphen).
- **Style & language**: Files include a triple-quoted Portuguese problem description at the top and use plain `print()` statements to show results. Preserve Portuguese text and formatting when editing.

Quick conventions for AI agents
- Work at the file level: each task is a self-contained script (no package/module structure).
- Keep changes minimal and focused: these are student exercise files; prefer small, well-documented edits.
- Preserve the header comment (problem description) and the printed output format — tests (if any) or instructors may depend on exact strings.
- Follow existing naming: `questao-01.py`, `questao-02.py`, etc. If creating a new file, match that pattern and keep the same language (Portuguese) in descriptions and outputs.

How to run examples (PowerShell)
- Run a single script:
```
python .\atividades-pratica01\questao-02.py
```
- Run all `questao-*` files in a folder (PowerShell):
```
Get-ChildItem .\atividades-pratica01 -Filter "questao-*.py" | ForEach-Object { python $_.FullName }
```

What to look for when editing or adding code
- Minimal dependencies: current files use only the Python standard library and plain literals.
- Inputs are currently hard-coded (e.g., `numero1 = 12`). If adding interactive behavior, keep it isolated and documented.
- Keep file-level scripts simple: avoid introducing complex package layouts or heavy refactors unless requested.

Examples from this repo
- `atividades-pratica01/questao-02.py` — adds two integers and prints a Portuguese message: keep exact phrase structure when modifying output.
- `atividades-pratica01/questao-04.py` — prints product details and total price; formatting with `R$` is used.

Developer workflows & missing pieces
- There are no tests, build scripts, or CI configuration in the repository. Do not assume test runners or linters are present.
- If you add tests or CI, include instructions and add a top-level `README.md` describing how to run them.

When merging/updating this file
- If a `.github/copilot-instructions.md` already exists, preserve any human-written guidance and only append or refine project-specific bits above.

Questions for maintainers
- Should outputs be normalized (numbers formatting, decimal separators)?
- Do you want interactive versions of scripts or kept as hard-coded examples?

If anything here is unclear, tell me which exercises or workflows you want the agent to prioritize and I will refine this file.

---
> Source: [eusrawayne/carita-brsao200](https://github.com/eusrawayne/carita-brsao200) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-10 -->
