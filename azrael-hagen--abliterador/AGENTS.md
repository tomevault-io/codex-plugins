# Project Guidelines

## Code Style
- Keep Python code simple and explicit, matching the current modular style in [../abliterador_app/](../abliterador_app/).
- Preserve naming conventions: `PascalCase` for classes and `snake_case` for methods/functions.
- Preserve Spanish UI text unless a task explicitly requests localization changes.
- Use Qt signal-slot patterns consistent with the existing code (`button.clicked.connect(handler)`).

## Architecture
- Entry point is [../abliterador_studio.py](../abliterador_studio.py), with modular app code in [../abliterador_app/](../abliterador_app/).
- `abliterador_app/domain/` contains contracts, `abliterador_app/services/` contains runtime model orchestration, and `abliterador_app/ui/` contains widgets.
- `ModelSearcher` owns UI setup and user actions, while heavy model operations are delegated to `ModelTaskWorker`.
- Keep entrypoint behavior intact (`if __name__ == "__main__": ...`).
- For larger changes, prefer incremental refactors that do not break current GUI behavior.

## Build and Test
- Run locally with:
  - `python abliterador_studio.py`
  - `launch_abliterador_studio.bat` (Windows launcher)
- There is no test suite yet. If adding non-trivial logic, add focused tests when practical.
- Validate changes by launching the GUI and exercising:
  - model search input validation
  - model selection validation
  - happy path and error path in `apply_heretic`

## Conventions
- Keep user-facing errors in `QMessageBox` for consistency.
- Avoid blocking the UI thread with heavy model operations when introducing new features; use worker-thread patterns if loading/generation gets extended.
- Do not replace the current simulated search behavior unless the task explicitly asks for real model search integration.
- Keep dependencies compatible with current imports: `PySide6`, `transformers`, and `heretic_llm`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Azrael-Hagen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
