
# Cursor Workflow Bridge

This repository includes AI assets under `.github/` that were authored for VS Code Copilot customization.
In Cursor, treat those files as project guidance and workflow templates.

## How to use in Cursor

- For full-stack work, follow `.github/prompts/AddFeature.prompt.md`.
- For backend entity work, follow `.github/prompts/AddEntity.prompt.md`.
- For testing work, follow `.github/prompts/WriteTests.prompt.md`.
- For review work, follow `.github/prompts/ReviewCode.prompt.md`.

## Specialist routing behavior

- Backend-focused tasks: prioritize `.github/agents/BackendDeveloper.agent.md`.
- Frontend-focused tasks: prioritize `.github/agents/FrontendDeveloper.agent.md`.
- Test/review-focused tasks: prioritize `.github/agents/TestingEngineer.agent.md`.
- Mixed full-stack tasks: use `.github/agents/FantasyBaseballOrchestrator.agent.md` phase order:
  planning -> implementation -> testing -> final review.

## Compatibility notes

- Do not assume VS Code-only slash commands are available.
- Re-express prompt workflows as normal chat instructions when needed.
- Keep all standards from `.github/instructions/` enforced during code changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinMgreer)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kevinMgreer)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
