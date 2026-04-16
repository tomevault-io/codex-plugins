# RScraper — GitHub Copilot Instructions

## Project

Travel data scraping and visualization solution for monitoring trip pricing across multiple destinations and time periods.

Stack (scraper): Python 3.8+, `requests`, `beautifulsoup4`.
Stack (web interface): TypeScript, React 18, Vite, Material-UI (MUI) v5.

## Language Rules

- Communicate with the human in Polish.
- Keep code, comments, documentation, and commit messages in English.

## Custom Agents

Canonical agent location for this repository: `.claude/agents/`

Workflow artifact locations:

- `docs/knowledge/lessons-learned.md` for reusable lessons
- `docs/tasks/TASK-###-plan.md` and `docs/tasks/TASK-###-complete.md` for task lifecycle artifacts
- `docs/reviews/TASK-###-review.md` for review reports
- `docs/testing/TASK-###-test.md` for test plans and results
- `docs/reviews/TEMPLATE-review.md` and `docs/testing/TEMPLATE-test.md` as reusable report templates

Use these custom agents:

- `.claude/agents/orchestrator.agent.md` as the main orchestration agent (human-driven, approval gates)
- `.claude/agents/autonomous-orchestrator.agent.md` as a fully autonomous variant (no approval gates, runs commits)
- `.claude/agents/ninja.agent.md` as a solo autonomous executor that plans, implements, tests, and commits itself while delegating review only
- `.claude/agents/planning-subagent.agent.md` for research
- `.claude/agents/implement-subagent.agent.md` for implementation
- `.claude/agents/code-review-subagent.agent.md` for review
- `.claude/agents/testing-subagent.agent.md` for testing

If your Copilot build does not auto-discover agents from `.claude/agents/`, register these files manually via the custom agent configuration UI or copy them to the user-level Copilot prompts location.

Subagent troubleshooting checklist:

- Ensure the selected chat mode is `orchestrator` or `autonomous-orchestrator` (custom agent), not generic chat.
- If the selected chat mode is `Ninja`, expect it to handle planning, implementation, testing, and commits itself while still delegating code review.
- In Chat -> Tools, confirm `runSubagent` is enabled for the current session.
- If needed, use `/agents` to reconfigure and reselect custom agents.
- If the agent reports unavailable tools, rerun once in a new chat session after reselecting the orchestrator agent.
- If `runSubagent` still fails, treat it as a runtime/tooling issue and report concrete failure evidence. Do not switch the orchestrator into single-agent execution.
- If repository search is unstable, prefer `fileSearch` + `listDirectory` + `readFile` + `textSearch` over one broad search request.
- Use `code-review-subagent` only for reviewing concrete current changes, not for general architecture exploration.
- If `planning-subagent` fails twice to return final findings, the orchestrator must stop that phase as blocked with evidence instead of performing the planning research itself.
- When adding or porting autonomous agents, align their workflow with the subagents and validation surfaces that actually exist in `.claude/agents/`.

## Coding Standards

**Python (RScraper/):**

- Target Python 3.8+; use only `requests` and `beautifulsoup4` for scraping — no new dependencies without justification.
- Data output goes to CSV files in `data/`; never modify existing CSV structure without review.
- Configuration stays in `sources.json`; no hardcoded URLs or trip data in source files.

**TypeScript / React (RDisplay/):**

- TypeScript strict; no implicit `any`.
- Functional components and hooks only.
- Material-UI (MUI) v5 components and theme; no inline style overrides that bypass the theme.

## Security

- Never hardcode secrets or API keys in source files
- Validate and sanitize any external data before processing or persistence

## Guardrails

- Prefer minimal, focused changes
- Check `docs/knowledge/lessons-learned.md` before work on a known area and append reusable lessons after non-trivial fixes
- Use task IDs in the form `TASK-001`, `TASK-002`, ... for task-scoped docs
- Run relevant checks after code changes (e.g. `python -m py_compile RScraper/*.py generate_deals.py` for scraper changes, `npm run build` for web interface changes)
- Prepare work for the human by default: stage the commit candidate, write review/test docs, and provide exact run/commit commands
- Keep fixes, re-review, re-testing, and documentation updates under the same `TASK-###` as the original implementation thread
- Before any commit handoff, ensure the full task candidate is cached: code + updated docs for the same `TASK-###`, with no unstaged leftovers for that task
- Before review, enforce staging-scope hygiene: staged diff must contain only task-relevant files and must match expected review scope
- `orchestrator` never runs `git commit`; `autonomous-orchestrator` may commit only after passing review, completing required testing, and only when no manual test stop is still pending; `Ninja` is an explicit exception and may commit autonomously after passing its mandatory review and testing loop
- At handoff, prefer returning file handles (`docs/tasks/TASK-###-complete.md`, `docs/testing/TASK-###-test.md`, `docs/reviews/TASK-###-review.md`) instead of relying only on chat text for commit/test instructions

## Temporary Files Policy

- Never create temporary files under `/tmp`.
- Use repository-local temporary files only under `./tmp/`.
- Ensure `./tmp/` exists before use (`mkdir -p ./tmp`).
- Clean up temporary artifacts after use and before final handoff.
- If environment policy blocks write operations outside the workspace, treat that as expected and retry using `./tmp/` paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filipbiernat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
