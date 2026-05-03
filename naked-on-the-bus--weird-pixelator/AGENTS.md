# Project Guidelines

## Build and Run
- Create and activate a virtual environment before installing dependencies.
- Install runtime dependencies with `pip install -r requirements.txt`.
- Run the app with `python main.py`.
- Install build dependencies with `pip install -r requirements-build.txt`.
- Build desktop artifacts with `pyinstaller --noconfirm --windowed --name "Weird Pixelator" main.py`.
- On macOS, prefer `./scripts/build_macos.sh` when producing the `.app` package.
- Build each platform on that platform (Windows on Windows, macOS on macOS, Linux on Linux).

## Architecture
- `main.py` is the Tkinter application layer and owns UI state, control wiring, preview updates, and save/export flow.
- `image_effects.py` is the image processing layer; keep effect functions stateless and focused on pixel transformations.
- `image_object.py` is a lightweight image container used to pass image metadata and pixel data through the pipeline.
- Preserve the current flow: UI values -> update method in `main.py` -> effect functions in `image_effects.py` -> render/export image.

## Code Conventions
- Follow existing Python style in this repo (simple functions, explicit names, minimal abstraction).
- In effect functions, keep early guards like `if image is None: return None`.
- Use NumPy array operations for pixel edits; avoid per-pixel Python loops unless unavoidable.
- Preserve alpha channel behavior when modifying RGB data.
- Keep preview responsiveness in mind (`preview_delay_ms` in `main.py`); avoid expensive recomputation during rapid slider changes.

## Packaging and Naming
- Keep the visible app name as `Weird Pixelator` in UI and build outputs.
- Do not remove or rename `icon.png` without updating macOS build flow in `scripts/build_macos.sh`.
- Keep both dependency manifests up to date when introducing new packages:
  - `requirements.txt` for runtime.
  - `requirements-build.txt` for packaging/build tools.

## Practical Guardrails
- Do not introduce commercial-license assumptions in docs or UI text; this repository is source-available with non-commercial restrictions in `LICENSE`.
- Prefer incremental edits in `main.py`; it is a large, central file where unrelated refactors can easily break UI behavior.

---
> Source: [naked-on-the-bus/Weird-Pixelator](https://github.com/naked-on-the-bus/Weird-Pixelator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
