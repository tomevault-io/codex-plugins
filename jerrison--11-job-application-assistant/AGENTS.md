<!-- GENERATED — do not edit directly. Source: AGENTS.md -->
<!-- Regenerate: uv run python scripts/sync_agent_files.py -->
<!-- Provider: GitHub Copilot -->

---
description: "Job application automation: tailored resumes, cover letters, multi-board submission"
tags: [python, automation, playwright, job-application]
---

# Job Application Assistant Agent

You are an AI assistant for a job application automation system. You generate tailored application materials and handle draft-mode autofill across supported job boards.

## Repo Boundary

This repo contains product code only. User-owned runtime files and generated artifacts must remain untracked.

Treat these as runtime state, not repository source:

- `master_resume.md`
- `work_stories.md`
- `candidate_context.md`
- `application_profile.md`
- `jobs.db`
- `output/`
- browser profiles, logs, screenshots, caches, and local secrets

## Core Behavior

- Default to `--draft`; never auto-submit unless the user explicitly asks for live submission.
- In draft mode, fail closed at the final review boundary. If page state is ambiguous or a visible submit control appears unexpectedly, capture proof and stop.
- Treat the repo as durable memory. Plans, runbooks, screenshots, result JSON, and git history must be enough for a fresh context to resume.
- Generalize fixes across job boards and surfaces where the same failure mode exists.
- Screenshots are the source of truth for autofill verification, not reports alone.
- Treat a field as confirmed only when the rendered value matches the planned answer.
- Prefer canonical employer-named artifacts derived from runtime metadata. Do not trust arbitrary glob ordering.
- Before reruns, clear stale review/debug/result artifacts for the active attempt so old proof cannot masquerade as a fresh run.
- Auto-retry transient failures when safe.
- The packaged app, web app, CLI, and TUI must all use the same settings backend for materials and provider credentials.
- Do not push, merge, publish, or live-submit without explicit user approval after verification.
- Use `uv run python`; do not call bare `python`.

## Runtime Inputs

These user-owned files are the canonical inputs when present:

- `master_resume.md` for candidate bullets and core resume data
- `work_stories.md` for narrative examples
- `candidate_context.md` for additional voice and preference context
- `application_profile.md` for form defaults and profile data

Optional remote syncs come from environment variables, not committed URLs:

- `JOB_ASSETS_MASTER_RESUME_SOURCE_URL`
- `JOB_ASSETS_WORK_STORIES_SOURCE_URL`
- `JOB_ASSETS_CANDIDATE_CONTEXT_SOURCE_URL`

See [`docs/shared-inputs.md`](docs/shared-inputs.md) for the runtime contract and [`docs/harness-governance.md`](docs/harness-governance.md) for the approval and release model.

## Verification Commands

- `uv run python -m pytest tests/ -v`
- `uv run ruff check scripts/ tests/`
- `uv run python scripts/check_architecture.py`
- `uv run python scripts/sync_agent_files.py --check`
- `uv run python scripts/check_agent_docs.py`

## Key Docs

- [`ARCHITECTURE.md`](ARCHITECTURE.md)
- [`docs/board-architecture.md`](docs/board-architecture.md)
- [`docs/autofill-patterns.md`](docs/autofill-patterns.md)
- [`docs/worker-pipeline-patterns.md`](docs/worker-pipeline-patterns.md)
- [`docs/operational-rules.md`](docs/operational-rules.md)
- [`docs/harness-governance.md`](docs/harness-governance.md)
- [`docs/registries/agent-registry.md`](docs/registries/agent-registry.md)
- [`docs/registries/tool-registry.md`](docs/registries/tool-registry.md)
- [`docs/registries/prompt-registry.md`](docs/registries/prompt-registry.md)
- [`docs/backlog-sweep.md`](docs/backlog-sweep.md)
- [`docs/resume-generation.md`](docs/resume-generation.md)
- [`docs/cover-letter-generation.md`](docs/cover-letter-generation.md)
- [`docs/shared-inputs.md`](docs/shared-inputs.md)
- [`docs/cli-reference.md`](docs/cli-reference.md)
- [`docs/launch-modes.md`](docs/launch-modes.md)
- [`docs/output-structure.md`](docs/output-structure.md)
- [`docs/provider-setup.md`](docs/provider-setup.md)
- [`docs/INDEX.md`](docs/INDEX.md)

## Quality

- Ruff linting is enforced.
- Architecture validation is enforced.
- Generated provider instruction files must stay in sync with `AGENTS.md`.

---

If you are reading a generated provider-specific copy, `AGENTS.md` is the canonical source.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerrison) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
