# Project Guidelines

## Mandatory Development Checklist

Before finishing work, run `uv run ruff check .`, `uv run pytest`, and verify the app starts with `uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`. In this Windows workspace, if `uv` is not on `PATH`, use `py -m uv ...`.

## Architecture

Soc Ops is a small FastAPI app that renders Jinja2 templates and uses HTMX for fragment updates instead of a JSON API frontend. Keep route wiring in `app/main.py`, session state in `app/game_service.py`, pure bingo rules in `app/game_logic.py`, and shared typed models in `app/models.py`.

## Conventions

Keep handlers thin and move state transitions into `GameSession`. Preserve the server-rendered HTMX flow: endpoints should return HTML partials, usually from `app/templates/components/`, not JSON. Reuse the existing CSS utilities in `app/static/css/app.css` and keep new helpers typed and minimal.

## Design Guide

Use a clear, modern visual direction with intentional typography, color, and spacing instead of generic default styles.

- Default aesthetic for this repo: editorial-tech with ocean/cyan accents.
- Keep core UI shell and game states visually distinct, but consistent through shared CSS variables and utility classes.
- Prefer expressive display typography for headings and readable sans-serif body text; maintain strong contrast and hierarchy.
- Build atmosphere with layered gradients/surfaces and restrained motion; include reduced-motion-safe behavior.
- Preserve interaction semantics while restyling: keep `#game-container`, `hx-*` attributes, and accessibility attributes (`aria-*`, disabled states).
- Keep test-sensitive copy stable unless tests are updated in the same change (for example: `Soc Ops`, `Start Game`, `How to play`, `FREE SPACE`, `← Back`).
- For styling changes, extend `app/static/css/app.css` utilities and tokens first, then apply class changes in templates.
- Ensure responsive behavior for mobile and desktop; avoid fixed sizes that break the board or modal at narrow widths.

## References

Link to existing docs instead of duplicating them: `README.md` covers the repo, `workshop/GUIDE.md` covers the lab flow, and `.github/instructions/css-utilities.instructions.md` covers styling utilities. Prefer extending existing assets under `.github/agents/` and `.github/prompts/` rather than introducing parallel conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rathinarasa7Nimal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
