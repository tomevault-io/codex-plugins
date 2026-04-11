# Copilot Instructions

## Overview
- Goal: Maintain a Dockerized Postgres 17 instance (Debian trixie) with pgvector compiled and available by default.
- Key files:
  - `Dockerfile` (multi-stage: builds pgvector in builder, copies into final runtime)
  - `docker-compose.yml` (builds from local Dockerfile, tags `lightrag-pgvector`)
  - `db/init/001_extensions.sql` (creates `vector` and optional extensions)

## Generation Rules
- Keep changes minimal and scoped to the task.
- Do not introduce unrelated tools or refactors.
- Keep builder and final stages on the same Postgres major/OS tag: `postgres:17-trixie`.
- Respect existing env vars and defaults (`POSTGRES_DB/USER/PASSWORD`).
- Do not remove the `db/init` semantics; scripts should run only on first init.

## Dockerfile Guidance
- Multi-stage build only:
  - Builder: install `build-essential` and `postgresql-server-dev-17`; compile pgvector (pin via `ARG PGVECTOR_VERSION`).
  - Final: copy `vector.so` and extension SQL/control files into the correct Postgres paths.
- Do not ship build tools or apt caches in the final image.
- Keep `FROM postgres:17-trixie` for both stages and `ARG PG_MAJOR=17`.

## Compose Guidance
- Use `build: .` and `image: lightrag-pgvector`.
- Map `${POSTGRES_PORT:-5432}:5432`.
- Mount a named volume for `/var/lib/postgresql/data`.

## Extensions
- Default: ensure `CREATE EXTENSION IF NOT EXISTS vector;` runs via `db/init/001_extensions.sql`.
- Optional: include commented examples for `pg_trgm`, `pgcrypto`, `uuid-ossp`, `citext`, `ltree`.

## Versioning and Upgrades
- To bump Postgres: change both `FROM` lines and `ARG PG_MAJOR`, and ensure dev package name matches.
- To bump pgvector: update `ARG PGVECTOR_VERSION`.

## Non-goals
- Do not add app code, ORMs, or external services.
- Do not change licensing or repository structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pcnate)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/pcnate)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
