## GitHub Copilot Guidance — BoardBase

This file sets expectations for AI-generated changes in this repository. Keep the guidelines general and reusable. Documentation content (Quarto files) must remain in Polish, but these instructions are in English.

### Project context

- Purpose: a relational database for board games with information about games, categories, mechanics, ratings, reviews, collections, and game session logs. Typical roles: guest, user, admin.
- Database technology: PostgreSQL (containerized via Docker Compose). Avoid hard-coding engine versions; prefer latest stable supported by the course and image pinning in Compose.
- Documentation: Quarto (`docs/case-study/CaseStudy.qmd`), ERD diagram source in `docs/case-study/ERD.mmd` with a PNG export referenced from the QMD.

### Tooling and configuration

- Docker Compose runs a `postgres` service. A server configuration file (`postgresql.conf`) is mounted into the container and activated via a `-c config_file=...` command argument.
- Use environment variables (optionally via `.env`) for credentials and connection parameters. Do not commit secrets.
- Commit conventions are enforced with commitlint/husky. Follow Conventional Commits strictly (see “Commit conventions” below).

### Assignment deliverables (Stage 1 overview)

Generate changes that help satisfy the requirements listed in `Readme.md`, including:

- Logical schema in 3NF with at least 8 non-associative, non-dictionary base tables; correct data types; primary/foreign keys; UNIQUE and NOT NULL where appropriate.
- Seed data: at least 4 rows per table, logically consistent.
- Users/roles and permissions (at least 2 users) with tested access.
- SQL queries (≥2, non-trivial, at least one with a subquery), views (≥2), and indexes (≥2) with justification and performance checks.
- Clear documentation (Polish): case study, ERD, logical schema with keys, decisions, queries, views, indexes, and permission tests.

## Documentation (Quarto, ERD, citations)

- Documentation language: Polish. Maintain citations and bibliography (`references.bib`, `diabetologia.csl`); acronyms from `glossary.yaml`.
- ERD: edit `docs/case-study/ERD.mmd`, export a PNG as `docs/case-study/ERD.png`, and reference it from the QMD.
- Include sections that justify key design decisions (data types, keys, normalization, indexes with performance results, permissions and tests).

## Good practices (do/don’t)

Do

- Model in 3NF; use dictionaries (e.g., category, mechanic) and association tables where appropriate.
- Use `IDENTITY`, `CHECK`, `UNIQUE`, `NOT NULL`, and FKs with explicit referential actions.
- Create indexes for real query patterns; justify choices and show explain/analyze differences.
- Keep seed data consistent and meaningful; respect FK order.
- Use `COMMENT ON` for important tables and columns to ease documentation.

Don’t

- Avoid quoted identifiers and mixed naming styles.
- Don’t add unnecessary indexes; they have DML maintenance costs.

## Commit conventions (MANDATORY)

All commits must follow Conventional Commits. See the repository rule file:
`/.github/instructions/conventional-commits.instructions.md`.

Key rules

- Format: `<type>[optional scope]: <imperative description>` (no trailing period, ≤100 chars).
- Types: `feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert`.

Examples

- `feat(schema): add rating table with checks`
- `docs(case-study): describe database choice and config`
- `test(sql): add view validation queries`

Examples to avoid

- `Added table`
- `fix: Update.`
- `hotfix: critical issue`

## Docker/Postgres integration tips

- Use environment variables for connection strings and credentials. Prefer a `.env` file consumed by Compose; don’t commit secrets.
- Make SQL scripts idempotent when reasonable (`CREATE ... IF NOT EXISTS`, `DROP ... IF EXISTS`).
- If you add scripts to run against the container, keep them minimal and parameterized.

---

These guidelines are intended to keep generated changes consistent, secure, and aligned with the project’s academic requirements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christopher-dabrowski)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/christopher-dabrowski)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
