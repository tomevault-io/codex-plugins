# Search Engine Workspace Instructions

Scope
- This workspace uses a simple control layer for consistent execution and handoff.
- Preserve the existing runtime workflow: notebook build path + Flask runtime path.
- Runtime implementation authority is defined by: `app.py`, `templates/index.html`, `Untitled4 (8).ipynb`, and root runtime artifacts (`vectorizer.pkl`, `pos_weight_vector.pkl`, `doc_ids.pkl`, `doc_titles.pkl`, `doc_abstracts.pkl`, `tfidf_base_matrix.npz`, `tfidf_pos_matrix.npz`, `click_store.pkl`).
- Treat `school_docs/files/*` and `school_docs/notes/*` as examples/courseware/reference context only unless explicitly requested otherwise.

Behavior Rules
- Do not change ranking logic, score formulas, or sort behavior unless explicitly requested.
- Do not redesign frontend UX/CSS unless explicitly requested.
- Do not refactor notebook pipeline unless explicitly requested.
- Prefer small, surgical edits over broad rewrites.
- Keep artifact compatibility with current files: vectorizer.pkl, pos_weight_vector.pkl, doc_ids.pkl, doc_titles.pkl, doc_abstracts.pkl, tfidf_base_matrix.npz, tfidf_pos_matrix.npz, click_store.pkl.

Execution Policy
- For review/explanation tasks, use Ask-mode behavior: analyze first, edit only if requested.
- For implementation tasks, use Autopilot behavior: implement end-to-end and verify.
- When changing behavior, log rationale in 00_admin/decision_log.md and change in 00_admin/change_log.md.
- Track open blockers in 00_admin/unresolved_issues.md.
- Track executable next actions in 00_admin/backlog.md.

Out of Scope By Default
- Architecture migrations
- Framework swaps
- New services/databases
- Dataset replacement

Active Control Surfaces
- Scoped instructions:
	- .github/instructions/python-runtime.instructions.md (applies to app.py)
	- .github/instructions/notebook.instructions.md (applies to **/*.ipynb)
	- .github/instructions/frontend.instructions.md (applies to templates/**/*.html)
- Agents:
	- .github/agents/search-ask.agent.md
	- .github/agents/search-autopilot.agent.md
	- .github/agents/search-release-check.agent.md
- Prompt templates:
	- .github/prompts/auto-implement.prompt.md
	- .github/prompts/review.prompt.md
	- .github/prompts/debug.prompt.md
- Skills:
	- .github/skills/handoff-quality/SKILL.md
	- .github/skills/search-runtime-triage/SKILL.md
	- .github/skills/notebook-build-check/SKILL.md
- Hook wiring:
	- .github/hooks/hooks.json

Invocation Guidance
- Use search-ask for review/explanation/risk triage and planning-only work.
- Use search-autopilot for implementation/fix/apply-change requests.
- Use search-release-check for read-only pre-handoff quality checks.
- Use prompt templates as follows:
	- auto-implement.prompt.md for implementation handoff framing.
	- review.prompt.md for regression/risk review framing.
	- debug.prompt.md for repro/root-cause/minimal-fix framing.

Validation Cadence
- Run school_docs/files/validate_customization.py after control-surface edits (customization governance validation only; not runtime behavior validation).
- Keep file_map.md and governance logs current when control surfaces change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peachyplayzrb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
