# vstack Copilot Instructions

You are assisting in the development of **vstack**, a VS Code–native AI engineering workflow system inspired by gstack.

## Identity

vstack provides structured skills for backend/microservice development, executable via GitHub Copilot Agent Mode. It exposes six delivery roles — product, architect, designer, engineer, tester, release — plus a planner coordinator agent, each backed by hand-authored agent files that invoke skill templates.

## Core Principles

- **Template-driven install model.** Skills are Markdown templates (`src/vstack/_templates/skills/<name>/template.md` — source of truth). Agent config is in `src/vstack/_templates/agents/<name>/config.yaml` + `template.md`. Generated output is produced at install time. In the vstack source repo, do not treat `.github/` artifacts as source of truth and do not edit them directly. In downstream consumer repos that use vstack, installed `.github/` artifacts are effective project configuration and are usually committed.
- **VS Code native.** Skills run in Copilot Chat / Agent Mode. No assumptions about Claude Code or CLI-only flows.
- **Backend first.** Prioritize API correctness, reliability, observability, CI/CD, contracts, performance, and security. Browser automation is optional and pluggable.

## Artifact Choice Policy

- Agents are for roles and handoffs.
- Skills are for reusable procedures.
- Instructions are for always-on policies.
- Prompts are for one-shot task framing.
- Hooks are for repository-level automation.

When proposals span multiple artifact types, split them into small independent changes.

## Reasoning & Critical Thinking

Evidence-based reasoning is available as an explicit opt-in framework via `prompt/reasoning`.

When `prompt/reasoning` is invoked, apply these principles:
- **Truth > agreement** — evaluate correctness, not user preference
- **Evidence > opinion** — support claims with facts, logic, or explicit assumptions
- **Critical thinking > people-pleasing** — challenge premises before accepting them

For structured adversarial review workflows, agents invoke `skill/advise`, which can reference `prompt/reasoning` when deeper critical analysis is needed.

**For downstream users:** Add `prompt/reasoning` to your vstack installation (in your `.vstack/vstack.json` manifest or via your preferred vstack distribution mechanism), then run `vstack install` to generate `.github/prompts/reasoning.prompt.md` in your repository. Agents and skills can invoke it by reference.

## System Structure

```
src/vstack/      ← Python package (source of truth)
src/vstack/_templates/
├── skills/<name>/
│   ├── template.md             ← skill body (edit these)
│   └── config.yaml
├── skills/_partials/           ← shared partial snippets
├── agents/<name>/
│   ├── template.md             ← agent instructions body
│   └── config.yaml             ← agent frontmatter fields
├── instructions/<name>/
│   ├── template.md             ← instruction file body
│   └── config.yaml
└── prompts/<name>/
    ├── template.md             ← prompt file body
    └── config.yaml
docs/            ← overview.md, design.md, skills.md, workflow.md, roadmap.md, adr/
.github/         ← generated output (never edit directly*)
├── skills/<name>/SKILL.md
├── agents/<name>.agent.md
├── instructions/<name>.instructions.md
└── prompts/<name>.prompt.md
.vstack/         ← project-scope vstack state and config
└── vstack.json                 ← install manifest (generated)
```

*Hand-authored exceptions in `.github/`: `copilot-instructions.md`, `CODEOWNERS`,
`PULL_REQUEST_TEMPLATE/`, `ISSUE_TEMPLATE/`, `workflows/`.

Generated files are written to `.github/` at install time:

```bash
python3 -m vstack install
```

## In-repo Installation

When vstack is installed inside its own repo (`.github/agents/` and `.github/skills/` exist):

**Source of truth is always in `src/vstack/_templates/`.** After every change:

| What changed | Command to run |
|---|---|
| `src/vstack/_templates/agents/<name>/config.yaml` or `template.md` | `python3 -m vstack install` |
| `src/vstack/_templates/skills/<name>/template.md` or `_partials/*.md` | `python3 -m vstack install` |
| `src/vstack/_templates/instructions/<name>/template.md` | `python3 -m vstack install` |
| `src/vstack/_templates/prompts/<name>/template.md` | `python3 -m vstack install` |

A single command handles all artifact types:

```bash
python3 -m vstack install
```

Never edit `.github/agents/`, `.github/skills/`, `.github/instructions/`, or `.github/prompts/` directly — changes will be overwritten.

## Execution Model

**Option A (current):** Single Copilot context window. A role agent reads the relevant skill and executes in one call.

**Option B (future):** Sequential multi-agent pipeline — `[vision → architecture → verify → release]`. Each stage is a separate model call; output artifacts feed the next stage. Design all config formats and generators to support this with minimal refactoring.

## Verification (Microservices-First)

`verify` and `verify-lite` must cover:

- Unit, integration, and contract tests (OpenAPI / JSON Schema / Protobuf)
- API smoke tests, migrations, retries/backoff, timeouts, circuit breakers
- Observability (logs, metrics, traces, dashboards)
- Security scanning, semver/API compatibility, linting, typechecking, packaging

Browser/E2E steps are optional.

## Documentation Rules

Update docs whenever a change affects system structure, skill definitions, execution flow, or the Option B path.

| Change | File to update |
|--------|---------------|
| High-level structure | `docs/architecture/overview.md` |
| Generator, loaders, builders | `docs/design/overview.md` |
| Execution flow / pipeline | `docs/design/workflow.md` |
| Skill names and behavior | `docs/design/skills.md` |
| Option B milestones | `docs/product/roadmap.md` |
| Any significant decision | `docs/architecture/adr/NNN-<slug>.md` |

Every ADR must include: context, decision, alternatives considered, rationale, and impact on the Option B pipeline.

## Markdown Documentation Style

For hand-authored Markdown in this repository, treat baseline docs as a coherent documentation system rather than isolated files.

- Apply the same documentation style to non-generated, non-template Markdown files such as `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CHANGELOG.md`, `docs/**/*.md`, and ADRs.
- Do not apply these rules by editing generated artifacts under `.github/` or source templates under `src/vstack/_templates/` unless the task is explicitly about those sources.
- Use Mermaid for process, flow, interaction, lifecycle, and decision diagrams when the visual structure matters more than exact monospace layout.
- Keep ASCII/text trees for repository structure, directory layout, and similar scan-friendly file hierarchies where Mermaid would reduce readability.
- Keep plain code fences for literal examples, terminal transcripts, frontmatter samples, JSON, YAML, and other content that is data rather than a diagram.
- Prefer consistency across related docs: if `README.md`, architecture docs, and workflow docs describe the same flow, they should use compatible terminology and diagram style.
- When modernizing older docs, convert diagrams selectively rather than mechanically. Preserve useful content first, then improve presentation.

## PyPI README Sync Rules

- `README.md` is the canonical full documentation for GitHub rendering.
- `README-pypi.md` is the PyPI-compatible long description source referenced by `pyproject.toml`.
- Whenever `README.md` changes in a way that affects installation, usage, positioning, links, or badges, update `README-pypi.md` in the same change.
- Keep `README-pypi.md` minimal and PyPI-safe: no Mermaid blocks, no relative documentation links, and no `<picture>` markup.
- Use absolute GitHub URLs in `README-pypi.md` for cross-document links and images.

## Python Docstring Style

For Python modules in this repository, treat code as the source of truth and keep docstrings aligned with shipped behavior.

- Use **PEP 257** as the baseline: complete sentences, correct one-line vs multi-line structure, and a concise summary line first.
- Keep docstrings **reStructuredText-compatible** per **PEP 287**. Use reST roles such as ``:class:`...``` when helpful and avoid Markdown formatting inside docstrings.
- Prefer **Google-style sections** when additional structure adds value, especially `Args:`, `Returns:`, and `Raises:` on public APIs with non-trivial behavior.
- Do not add section headers mechanically. For simple helpers, a precise one-line docstring is preferred over verbose boilerplate.
- Module docstrings should explain responsibility and key concepts. Class docstrings should describe the abstraction. Function and method docstrings should describe behavior and observable effects rather than implementation trivia.
- Avoid placeholder docstrings such as "Initialize instance state" or "Build parser". Describe intent and contract instead.
- When a behavior, public API, or exception contract changes, update the corresponding docstring in the same change.

## Development Commands

| Task | Command |
|------|---------|
| Fast tests (current interpreter only) | `make test-local` |
| Full quality gate (format-check + lint + typecheck + test) | `make check` |
| Lint | `make lint` |
| Type-check | `make typecheck` |
| Format Python + Markdown | `make format` |
| Regenerate `.github/` artifacts from templates | `python3 -m vstack install` |

Test coverage is enforced at **100%** (`--cov-fail-under=100`). Every behavioral change must keep all checks green. See [CONTRIBUTING.md](../CONTRIBUTING.md) for full dev setup.

## CLI Architecture

CLI commands live in `src/vstack/cli/`, one file per command (e.g. `install.py`, `verify.py`, `validate.py`).

- Each command subclasses `BaseCommand` and implements `run(*, context: CommandContext) -> int`.
- `CommandLineInterface` (`interface.py`) is the parsing/dispatch facade — no business logic.
- `CommandService` (`service.py`) constructs and dispatches commands.
- `COMMAND_CATALOG` (`catalog.py`) is the registration point for all commands.

When adding a new CLI command: create the command class in a new file, then register it in `COMMAND_CATALOG`.

## Work Style

- Produce small, reviewable changes.
- Always edit `src/vstack/_templates/skills/<name>/template.md`; generate output at install time with `python3 -m vstack install`.
- Default to microservices and libraries unless UI is explicitly present.
- After each change: summary + files changed + how to test.

## Pull Request Response Format

- When asked to provide PR content, strictly follow the repository template at `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md`.
- Always provide PR content in English.
- Always provide PR content as copy-paste-ready Markdown in a fenced `markdown` code block.
- Do not add extra sections that are not present in the active PR template.

## Context Exclusions

When searching or reading repository content, ignore generated and third-party directories by default:

- `.venv/`, `venv/`, `env/`
- `node_modules/`
- `__pycache__/`
- `dist/`, `build/`
- `.git/`

Only include these paths when the user explicitly asks to inspect them.

## Safety

Before executing any destructive command (`rm -rf`, `DROP TABLE`, `git push --force`,
`git reset --hard`, `kubectl delete`, production config changes, database migrations):

1. Stop.
2. Explain exactly what will be permanently lost.
3. Ask for explicit confirmation.
4. Only proceed if the user confirms.
5. Never use workarounds to avoid this confirmation.

---
> Source: [eschaar/vstack](https://github.com/eschaar/vstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
