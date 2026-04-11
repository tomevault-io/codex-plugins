# AI Agent Instructions — Dear Apple

**Intended audience:** Public contributors and automation agents.

Purpose: quick, actionable guidance to make AI agents productive in this repository.
Focus on discoverable patterns, formatting expectations, and non-negotiable rules.

Do this repo’s work:
- This is a documentation-first repo. Primary artifacts live in `analyses/` and `letters/`.
- Use `analyses/TEMPLATE.md` and `letters/TEMPLATE.md` as the canonical templates for analyses and letters respectively. Files should be placed under `analyses/YYYY-MM-DD-short-title.md` or `letters/` as appropriate.

What an AI agent may safely do
- Structural edits: ensure files follow the folder templates (`analyses/TEMPLATE.md` or `letters/TEMPLATE.md`) (headings, Observation status & confidence, Evidence, Open questions, Updates).
- Consistency checks: detect duplicated sections, contradictory statements, missing fields from the template, or repeated reproduction steps.
- Editorial refinement: improve clarity, fix grammar, normalize headings and formatting to match existing style.
- Add metadata stubs: add an explicit "Author's note on methodology" header when missing and a preface pointing to `AI_TOOLS_USAGE.md`.
- Add a short, neutral "Evidence (stub)" when no logs exist, but clearly label it as hypothetical / unexecuted.
- Create PRs and branch names using the project's pattern (e.g. `add/analysis-2025-12-24-safari-external-link`). Ensure PR description follows the repository's PR checklist (see `CONTRIBUTING.md`).

What an AI agent must NOT do
- Generate or fabricate observational data, logs, or test results. All technical observations must be clearly attributed to a human contributor who performed the test on real hardware.
- Compose exploit code or publish steps that could enable harmful actions. For any security-possible finding, follow `SECURITY.md` vendor-first guidance.
- Merge or accept PRs that lack required human-verified evidence or that fail to document uncertainty/confidence clearly.

Repository-specific patterns & examples
- Repro steps should be a numbered list exactly as in `TEMPLATE.md` → example block under "Reproduction steps (observed)" in existing analysis.
- Include an "Observation status & confidence" block with the following fields: Reproducible by author, Reproduced on multiple devices, Hardware dependency suspected, Regression vs previous versions, Confidence level, Uncertainty summary, Open questions.
- Keep an "Evidence" section even when empty: mark as `hypothesized` or `not yet executed` and list the minimal reproducible probe ( eg. cookie set + echo endpoint ) when applicable.
- Use `TEMPLATE.md` fields order and headings—review `analyses/2025-12-24-macos-safari-external-link-routing-private-context.md` as canonical example.

Safety and governance
- All contributions must avoid PII. If PII is present, redact before committing and flag maintainers for help (see `DISCLAIMER.md`).
- If a contribution references possible security-impacting behavior, add a note and follow `SECURITY.md` before any public disclosure or exploit details.
- Disclose AI usage: place a short note in the document header if AI tools assisted editorially; follow `AI_TOOLS_USAGE.md` for required language.
- Prefer ASCII punctuation and plain text in technical content — avoid non-ASCII punctuation (non-breaking hyphens, typographic quotes, en/em dashes) and emoji in `analyses/`, `letters/`, and governance files where possible. Use UTF-8 only for controlled, decorative contexts (e.g., social media-ready headings), and document any intentional exceptions in the PR description.
- **Use English for official repository content.** All technical documentation, templates, issue titles, PR descriptions, and governance files should be written in English. Optional translations are allowed but must be provided in addition to — not instead of — English; document any intentional exceptions in the PR description.

PR checklist for agents to validate (automatable checks)
- File placed under correct directory: `analyses/` or `letters/`
- `TEMPLATE.md` headings are present and complete
- `Observation status & confidence` block exists and is populated
- `Evidence` section exists and is clearly labelled (hypothesized / collected / attached)
- No PII patterns (email addresses, IPs, serial numbers) are present
- If security-sensitive, `SECURITY.md` steps are referenced
- PR description follows the PR checklist in `CONTRIBUTING.md` and includes required checkboxes

If you are uncertain about a change
- Don't guess test results. Mark as `hypothesized` and add a suggested minimal test (server echo, header/cookie probe) for a human to run.
- Ask reviewers to confirm provenance (who ran which test, on which hardware/OS).

Contact/maintenance
- For policy or governance questions, reference `AI_TOOLS_USAGE.md`, `CONTRIBUTING.md`, and `CODE_OF_CONDUCT.md`.
- If you're an external contributor and unsure about redaction or publication, open an issue (do **not** include sensitive data) and maintainers will advise.
- Tag maintainers and request guidance for redaction, sensitive disclosure, or legal questions.

---

Keep instructions concise, non-prescriptive, and always prioritize human verification over automation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/overbog)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/overbog)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
