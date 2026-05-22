# Project Guidelines

## Code Style
- Follow existing Python style in this repo: simple classes, explicit state fields, and minimal abstraction.
- Keep puzzle logic inside classes under `mqtt/puzzles/`; do not move game-state logic into Flask routes.
- Keep frontend puzzle logic in paired files: `templates/puzzleN.html`, `static/js/puzzleN.js`, `static/css/puzzleN.css`.
- Preserve current naming patterns:
  - Python modules and files: snake_case
  - Puzzle files: `puzzle1.py` ... `puzzle12.py`
  - MQTT payload format: comma-separated values (`P{id},...`)

## Architecture
- Core flow: Flask (`app.py`) -> MQTT client (`mqtt/client.py`) -> puzzle classes (`mqtt/puzzles/*.py`) -> SSE updates (`/state_stream`) -> browser clients (`static/js/*.js`).
- `app.py` owns routing, view rendering, and SSE fan-out queues.
- `mqtt/client.py` owns broker connectivity (`localhost:1883`), topic subscription (`TO_FLASK`), and dispatch to puzzle handlers.
- Puzzle classes inherit `BasePuzzle` in `mqtt/puzzles/base.py` and should implement `reset`, `handle_message`, `get_state`, and `stop` consistently.
- Puzzle order lives in `config.py` (`PUZZLE_ORDER`).
- Tutorial and final selectors live in `config.py` (`PUZZLE_TUTORIAL`, `PUZZLE_FINAL`) and stay outside `PUZZLE_ORDER`.

## Build And Test
- Python dependencies:
  - `python3 -m pip install -r requirements.txt`
- Start MQTT broker locally before running the app (expected host/port: `localhost:1883`).
- Run Flask app:
  - `flask --app app run`
- There is no automated test suite currently.
- For manual MQTT-driven checks, use scripts in `scriptsBash/` (example: `scriptsBash/puzzle3Trivial.sh`).

## Conventions
- Any new puzzle should follow the existing end-to-end pattern:
  - Add puzzle class in `mqtt/puzzles/`
  - Register it through puzzle factory flow used by `create_puzzles(...)`
  - Add matching template/js/css files
  - Ensure state updates are sent via `mqtt_client.push_update(...)`
- Keep SSE payloads JSON-serializable and stable for existing frontend listeners.
- Respect hardware integration constraints:
  - Inbound topic: `TO_FLASK`
  - Outbound start topic: `FROM_FLASK`
  - State mirror topic: `puzzles/{puzzle_id}`
- Avoid introducing blocking operations in Flask request handlers; puzzle timing/state should remain in puzzle classes or background-safe logic.

## Pitfalls
- Repo assets under `static/audios/`, `static/images/`, and `static/videos/` are ignored in Git and may be missing locally.
- If MQTT is unavailable, puzzle interactions fail even when Flask routes render successfully.
- Keep thread-safety in mind for shared puzzle state (`threading.Lock` is used in current puzzle implementations).

---
> Source: [thepapallot/jocPro](https://github.com/thepapallot/jocPro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
