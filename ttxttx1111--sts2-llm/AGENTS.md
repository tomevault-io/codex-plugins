# Copilot instructions for this repository

This repository is a GitHub Copilot customization pack for a **Slay the Spire 2 player agent**, not an application codebase.

## Source of truth for customizations

- Put direct-open agents in `.github/agents/`
- Put reusable skills in `.github/skills/<skill-name>/`
- Put persistent STS2 run memory in `.github/sts2-memory/`
- Put shared VS Code MCP server configuration in `.vscode/mcp.json`

## Runtime assumptions

- The local game install is expected at `D:\program\steam\steamapps\common\Slay the Spire 2\`
- Gameplay control is provided by the STS2MCP mod over `localhost:15526`
- In VS Code, the shared MCP setup for this repo lives in `.vscode/mcp.json`, which uses the VS Code-style `servers` key
- In Copilot CLI, equivalent MCP access can come from a temporary current-session attach or from `/mcp add` / `~/.copilot/mcp-config.json`; the CLI home config uses `mcpServers`
- A custom agent or skill may load successfully even when the current session does **not** have STS2 MCP tools attached; in that case live piloting must stop at the MCP-readiness check and report the blocker plainly
- In Copilot CLI, if the current session already has STS2 MCP tools attached, treat that as good enough to play immediately rather than insisting on persistence first
- In Copilot CLI, if live STS2 piloting must also work in a fresh session or custom agent, prefer a **persistent MCP configuration** that is available at session start rather than relying on another session's MCP attachment
- When the only missing piece is that persistent CLI MCP config and the agent can edit local files, prefer repairing `~/.copilot/mcp-config.json` directly before asking the user to do manual setup

## Authoring rules for this repo

- Keep skills and agent instructions **reusable**; do not hardcode a single run, seed, or class unless the file is explicitly scoped that way
- Prefer official Copilot customization formats and frontmatter; do not invent custom control files or ad-hoc schemas
- Encode gameplay guidance as evidence-backed heuristics, and clearly separate general rules from class-specific or relic-specific advice
- Record meaningful design decisions and behavior changes in repo documentation so the reasoning behind updates stays traceable
- Keep a concise update log whenever the player workflow changes materially, so the final merged documentation can be assembled from a clear change history
- Treat `.github/design/README.md` and `.github/design/system-details.md` as the maintained entrypoints for current architecture and workflow design
- Write changelog entries under `.github/changelog/` with **one markdown file per feature or workflow change** plus an updated `.github/changelog/README.md` index
- Do not use `prepare_task/` as the long-term source of truth for feature history; leave `prepare_task/change-log.md` as a compatibility pointer only
- When changing overall workflow, memory structure, review rules, skill boundaries, or agent orchestration, update the design docs in `.github/design/`
- When the workspace is a git repository, make coherent commits for completed milestones instead of leaving important design and workflow changes ungrouped
- Preserve the distinction between:
  - always-on instructions (`copilot-instructions.md`)
  - reusable capabilities (`skills`)
  - persistent persona/tooling (`custom agents`)
- Assume the user may interact in Chinese; default to concise Chinese unless the user switches languages

## Gameplay-specific expectations

- Before acting, verify the STS2 MCP tools are available and the game state can be read
- For local MCP-controlled gameplay in Copilot CLI, prefer a persistent MCP setup that survives new sessions and `/agent` launches
- Persist the current-run playbook in `.github/sts2-memory/current-playbook.md`
- Persist recent run history in `.github/sts2-memory/run-history.md` and `.github/sts2-memory/runs/`
- Persist the cross-run lesson ledger in `.github/sts2-memory/lesson-ledger.md`
- At run start, resume, and before major decisions such as pathing, card rewards, shops, rests, elites, and boss prep, read both files and refresh the most relevant lessons against the current run state
- Treat HP as a resource, but avoid reckless lines that ignore lethal or obvious survivability constraints
- Re-check state after actions that can shift indices or change screens
- In combat, treat hand and target indices as volatile: never chain multiple card plays off one read when a fresh read is cheap
- Before each combat card play, verify the intended card against the latest hand state by card name, cost, and targetability, then use the current index from that fresh state
- When strategy is uncertain, use web research and adapt findings to the current deck, relics, HP, gold, and map rather than copying generic advice blindly
- After notable fights, losses, or wins, trigger review behavior proactively instead of waiting for the user to ask
- After meaningful fights, rewards, route pivots, and rest/shop decisions, update `.github/sts2-memory/current-playbook.md`
- After run start, resume, and meaningful checkpoints, update the active run log with timestamp, floor, deck/relic snapshot, and latest outcome so recent-run questions can be answered directly
- After a run ends, keep a brief entry in `run-history.md` and archive the detailed timeline under `.github/sts2-memory/runs/`
- When a review changes your mind about a prior heuristic, revise the lesson status explicitly in `.github/sts2-memory/lesson-ledger.md` instead of keeping stale advice around
- Keep skills narrow and task-scoped; let the `player` agent orchestrate when to use piloting, review, and research skills instead of collapsing the whole workflow into one skill

---
> Source: [ttxttx1111/sts2-llm](https://github.com/ttxttx1111/sts2-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-24 -->
