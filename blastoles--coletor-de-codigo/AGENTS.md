## Quick instructions for AI coding agents

This repository provides a deterministic, JSON-based project snapshot used to avoid hallucinations.
Follow these project-specific rules exactly — they are *enforced by the extraction tool*.

- Key files to read before making changes:
  - `extrair.py` — main extractor (run with Python; GUI folder picker). Produces `contexto_ia_<project>.json`.
  - `PROMPT_DEFINITIVO_INICIAL.md` — mandatory prompt to paste immediately after the JSON.
  - `PROMPT_TRABALHO.md`, `PROMPT_EMERGENCIA.md`, `PROMPT_EXCELENCIA_MAXIMA.md` — follow-up prompts for tasks.
  - `README.md` — usage examples and recommended flow.

- Minimal workflow (always follow):
  1. Run: `python extrair.py` and select the project root. The extractor writes `contexto_ia_<project>.json`.
  2. Open the JSON and COPY ALL content to the AI input (Ctrl+A / Ctrl+C).
  3. Paste `PROMPT_DEFINITIVO_INICIAL.md` exactly after the JSON and require the model to confirm per that file.
  4. For ordinary tasks, use `PROMPT_TRABALHO.md`. For errors use `PROMPT_EMERGENCIA.md`.

- Absolute naming rule (project-specific):
  - Never change identifiers. Use the exact names found in the JSON `source_files` → `files` → `content`.
  - Example: if the code shows `def Criar_conta_dados(...):` you must reference `Criar_conta_dados` exactly (case and underscores).

- Required response format when referencing code (enforced by prompts):
  - Include: `Arquivo: [path]`, `Hash: [file_hash]`, `Linha aproximada: [n]`, `Nome EXATO: [0name0]` and a code excerpt copied from `content`.
  - Always validate with `file_hash` from the JSON to ensure you opened the correct file.

- Project conventions and gotchas (discoverable from `extrair.py` and README):
  - The extractor ignores common vendored dirs (`node_modules`, `.git`, `venv`, etc.).
  - Files > 2MB are skipped by default. Encoding fallback: utf-8 → latin-1.
  - The produced JSON contains `ai_instructions` which define the verification checklist. Consult it before any change.

- Developer workflows & environment notes:
  - Requires Python 3.7+. `tkinter` is used for folder selection; install system tkinter if GUI fails.
  - No external pip dependencies are required for extraction.
  - For debugging, run `python extrair.py` in a terminal to see progress and any read errors; read the generated JSON `errors` section.

- Integration points:
  - The extractor is the single source-of-truth. All AI tasks must begin by consuming the generated JSON.
  - Do not rely on unconstrained model knowledge or higher-level refactor suggestions that change identifier names.

If anything in this document is unclear or you want a stricter/looser agent behavior, tell me which section to adjust and I will iterate.

---
> Source: [Blastoles/Coletor_De_Codigo](https://github.com/Blastoles/Coletor_De_Codigo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-09 -->
