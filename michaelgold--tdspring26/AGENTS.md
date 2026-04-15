# Copilot Instructions

## Project Context

This is a Blender Technical Direction course repository. Scripts automate Blender scene setup, rendering, cameras, lighting, and pipeline tasks using the `bpy` Python API.

## Environment

- Python **3.11.9** with a local `.venv/` virtual environment
- `bpy` is installed in `.venv/` — always activate the project venv, never system Python
- CLI tooling is built with **Typer** (`typer` is in `requirements.txt`)
- **Do NOT use headless Blender** (`blender --background --python ...`). Scripts are executed directly via the venv: `.venv/bin/python script.py`
- Install dependencies with: `pip install -r requirements.txt`

## Writing bpy Code

- **Always use Context7** to look up current `bpy` API documentation before writing or modifying any `bpy` method, property, or operator call. Query it with the library name `bpy` or `blender`.
- Prefer `bpy.data`, `bpy.ops`, and `bpy.context` patterns consistent with bpy 5.x.
- Do not guess at API signatures — verify them via Context7 first.

## Task Workflow

- All tasks are tracked in `backlog.md` at the repo root.
- Before starting any implementation work, read `backlog.md` and identify the next uncompleted task.
- Work on one task at a time. Mark it as complete in `backlog.md` when done.
- If `backlog.md` does not exist, ask the user what to work on before proceeding.

## Testing

- After implementing each feature or fix, run the script with `.venv/bin/python <script.py>` to verify it executes without errors.
- Describe what to check or observe in Blender to confirm the feature works as expected.
- Wait for the user to confirm the result before proceeding.

## Commits

- After the user confirms a feature or fix works, suggest a [Conventional Commits](https://www.conventionalcommits.org/) message.
- Format: `<type>(<scope>): <short description>` — e.g. `feat(camera): add automated lighting rig helper`
- Common types: `feat`, `fix`, `chore`, `refactor`, `docs`
- Do not commit automatically — always present the message for the user to review and run.

## Project Structure

```
exercises/      weekly exercise scripts and assets
assets/         diagrams and shared reference files
requirements.txt  Python dependencies
backlog.md      task queue — read this before starting work
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michaelgold) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
