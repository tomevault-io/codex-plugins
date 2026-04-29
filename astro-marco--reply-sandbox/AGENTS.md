# Copilot Instructions for `reply-sandbox`

## Project scope and source of truth

- `Sandbox_2026_V3.pdf` (repo root) is the challenge specification.
- `public_lev_1/` contains Level 1 training inputs:
  - `status.csv`
  - `users.json`
  - `locations.json`
  - `personas.md`
- `how-to-track-your-submission/` contains operational helper material, including `main.py` for Langfuse trace inspection.

This repository is still in a build phase: the challenge solution pipeline is not fully implemented yet.

## Commands currently available

Use these commands for the existing helper script:

```bash
cd how-to-track-your-submission
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 main.py <session_id>
```

There is currently no configured build system, lint command, or automated test suite in this repository.
For challenge development, prefer Python 3.13 (Python 3.14 is noted as problematic in the official guidelines).

## Challenge and submission constraints

- Challenge URL: `https://challenges.reply.com/challenges/ai-agent/home/`
- How-it-works reference: `https://challenges.reply.com/challenges/ai-agent/how-it-works/`
- Allowed challenge models (whitelist): `https://cdn.reply.com/documents/challenges/02_26/api_model_whitelist.html`
- API/Langfuse guidelines reference: `https://cdn.reply.com/documents/challenges/02_26/api_guidelines.html`
- From the challenge flow/spec:
  - Build an adaptive AI/multi-agent system that classifies instances with:
    - `0` = continue standard monitoring
    - `1` = activate personalized preventive support pathway
- LLM/API constraints from challenge docs:
  - Use the provided challenge API key and route LLM calls through OpenRouter.
  - Use only model IDs from the whitelist URL above.
  - The LLM must be the main decision/orchestration layer (not a mostly deterministic pipeline with superficial LLM usage).
  - Langfuse is mandatory for tracking/validation; include the Langfuse `session_id` in submissions.
  - Prefer Langfuse SDK v3 for compatibility with the challenge platform.
  - Official scoring is based on submitted output files (not Langfuse metrics directly).
  - Expected env vars for challenge runs include `OPENROUTER_API_KEY`, `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_HOST`, `TEAM_NAME`, and `LANGFUSE_MEDIA_UPLOAD_ENABLED=false`.
  - `session_id` should follow `{TEAM_NAME}-{ULID}` (no spaces in `session_id`; normalize spaces in team name to `-`).
- From private dashboard submission instructions:
  1. Upload output file(s) first as UTF-8 plain text.
  2. Upload source code after output upload.
  3. After source upload, no additional submissions are allowed.
- Submission workflow constraints from challenge docs:
  - Training datasets: output uploads are iterative/unlimited.
  - Evaluation datasets: final submission is one-shot and must include output + source code zip.
  - Submitted source zip must be runnable/comprehensive (code, dependencies, config, instructions).
  - Initial token budget applies to datasets 1-3, with additional budget unlocked for datasets 4-5 after evaluation submission for datasets 1-3.

## High-level architecture context (current state)

- `public_lev_1/` is multimodal input data to be fused by future solution code:
  - longitudinal status events (`status.csv`)
  - user profile/residence metadata (`users.json`)
  - geospatial/temporal trajectories (`locations.json`)
  - narrative behavioral context (`personas.md`)
- Existing Python runtime code is `how-to-track-your-submission/main.py`, which is an observability utility (not the predictive pipeline):
  - loads `.env` values with `python-dotenv`
  - initializes a Langfuse client
  - paginates trace listing (`limit=100`)
  - fetches trace details and aggregates `GENERATION` observations (model count, cost, timing)
  - prints summary output including shortened first input / last output previews

## Repository-specific conventions

- Langfuse credentials are required for the helper script:
  - `LANGFUSE_PUBLIC_KEY`
  - `LANGFUSE_SECRET_KEY`
  - optional `LANGFUSE_HOST` (defaults to `https://cloud.langfuse.com`, but challenge submissions should use `https://challenges.reply.com/langfuse`)
- `main.py` fails fast if required credentials are missing.
- Keep challenge submission outputs UTF-8 plain text files compatible with the upload flow above.
- Treat missing output files, missing source zip (for evaluation), missing Langfuse session ID, or incomplete/corrupted source package as submission blockers.
- In challenge pipeline code that performs LLM calls, use `@observe()` + `CallbackHandler()` and flush traces (`langfuse_client.flush()`) before process exit so session tracking is complete.

---
> Source: [astro-marco/reply-sandbox](https://github.com/astro-marco/reply-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
