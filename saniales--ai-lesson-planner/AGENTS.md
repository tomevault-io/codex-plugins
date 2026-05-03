# Project Guidelines

## Code Style
- Validate lesson identifiers exactly as scripts do: lesson number must be `01..23` (two digits) and lesson id must be lowercase kebab-case (`[a-z0-9-]+`).
- Preserve naming/status consistency between `lessons/README.md` and `.lesson-config.json`.
- Enforce language policy: use `course.metadata.language` when available, default to English, and always honor explicit user language override.
- Enforce section-count policy: follow the section count decided in planning/highlights/user input; templates are guidance only, never fixed counts.

## Architecture
- Use the chat-first flow: `@course-planner` -> `@discussion-moderator` -> `@lesson-planner` (optional) -> `@slides-maker` -> `./scripts/generate-slides-pdf.sh`.
- Course metadata source is root `.lesson-config.json` (scripts fail if missing for course scaffolding).
- Lesson artifacts live under `lessons/lesson-{number}-{id}/.lesson/artifacts/` with `highlights/`, `discourse/`, `slides/`; PDF output is `lessons/lesson-{number}-{id}/SLIDES.pdf`.
- Use templates from `.lesson/templates/` (`HIGHLIGHTS.md`, `DISCOURSE.md`, `SLIDES.md`, `.lesson-config.json`) as structure baselines only.

## Build and Test
- Prerequisites used by scripts: Bash, `jq` (required for full-course scaffolding), `marp` (PDF export), `gh` (GitHub auth/repo ops).
- Single-lesson scaffolding: `./scripts/generate-lesson.sh <number> <id> "<title>"`.
- Full-course scaffolding from config: `./scripts/generate-course-scaffolding.sh` (or `--force` to overwrite existing lesson dirs).
- Range options are not implemented (`--range/--from/--to` not supported in current scripts).
- Export slides PDF: `./scripts/generate-slides-pdf.sh lessons/lesson-XX-lesson-id`.
- No automated test suite is defined in this repository; validate by script exit status and expected file outputs.

## Project Conventions
- Keep workflow outputs in lesson artifact folders; do not invent alternate locations.
- `generate-lesson.sh` updates lesson status in `lessons/README.md` and updates `.lesson-config.json` status only when `jq` is available.
- Repository currently ignores `.github/`, `tmp/`, and media/PDF files via `.gitignore`; avoid relying on ignored artifacts as durable outputs.
- For publishing workflow, `repository-creator` expects `tmp/highlights/`, `tmp/discourse/`, `tmp/slides/` and then creates a separate repo from template.

## Integration Points
- GitHub CLI integration: `./scripts/prepare-git.sh` + `gh` commands in repository creation flow.
- MARP CLI integration: PDF generation from concatenated `slides/*.md`.
- JSON integration: `jq` for reading/updating `.lesson-config.json`.

## Security
- Before GitHub operations, run `./scripts/prepare-git.sh` to clear conflicting `GITHUB_TOKEN` and configure `gh` auth.
- Respect `.gitignore` and publishing rules: no PDFs/media artifacts in commits unless explicitly required; no agent/workflow instructions in published lesson README.
- Destructive operations require confirmation (existing lesson dir overwrite; existing target repo destruction).

---
> Source: [saniales/ai-lesson-planner](https://github.com/saniales/ai-lesson-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
