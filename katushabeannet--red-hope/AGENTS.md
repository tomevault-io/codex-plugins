# Red_Hope Copilot Instructions

Use these rules when making changes in this workspace.

## Project Structure
- Backend API is in `System/backend` (Spring Boot, Java 17, Maven).
- Frontend app is in `System/red-hope` (React + TypeScript + Vite).
- Dataset and model artifacts are in `DATASET` and `ELEGIBILITY MODELS`.

## Hard Rules
- Do not modify files in `DATASET` or `ELEGIBILITY MODELS` unless the user explicitly requests data/model updates.
- Keep changes minimal and targeted; avoid unrelated refactors.
- Keep backend and frontend concerns separated unless the task explicitly requires coordinated changes.

## Preferences
- For backend changes, preserve package structure under `com.blooddonation.backend` and existing controller/service/repository layering.
- For frontend changes, keep TypeScript strictness and existing ESLint compatibility.

## Validation
- For backend code changes, run tests with Maven from `System/backend`.
- For frontend code changes, run lint and build checks from `System/red-hope`.
- If validation cannot be run, clearly report what was not validated.

## Output Rules
- Explain changes with concrete file paths and why each change was required.
- Flag assumptions before implementing behavior that is not explicitly specified.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/katushabeannet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
