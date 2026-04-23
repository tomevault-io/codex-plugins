# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A production-ready **FastAPI REST API template** for the KAOS platform. The app lives under `app/` (Python package with its own `pyproject.toml`). All development commands run from inside `app/`.

## Commands

```bash
# Setup
cd app && poetry install && poetry shell

# Run locally (auto-reload)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run full observability stack (API + Loki + Prometheus + Tempo + Grafana)
docker-compose up -d

# Tests
cd app && pytest
cd app && pytest tests/test_health.py -v          # single file
cd app && pytest --cov=app --cov-report=term-missing  # with coverage

# Lint
cd app && flake8 app/

# Version bump (managed by commitizen — do not edit __version__.py manually)
cd app && cz bump
```

## Architecture

```
app/
├── app/
│   ├── main.py               # FastAPI app init, router registration, uvicorn entry
│   ├── config.py             # AppConfig (env vars) + RBAC access_requirements dict
│   ├── __version__.py        # Semver string managed by commitizen
│   └── core/
│       ├── fastapi/
│       │   ├── models/       # Pydantic request/response schemas
│       │   └── routers/      # One file per feature group; register in main.py
│       ├── otlp/utils.py     # PrometheusMiddleware, setting_otlp(), metrics endpoint
│       └── zitadel/
│           ├── auth.py       # get_current_user() — JWT validation via Zitadel JWK
│           └── access_control.py  # authorize_access() — RBAC enforcement
└── tests/
```

### Key Patterns

**Adding a new endpoint:**
1. Create models in `app/core/fastapi/models/`
2. Create router in `app/core/fastapi/routers/` using `APIRouter()`
3. Register in `main.py`: `app.include_router(my_router.router)`

**RBAC:** Endpoint access requirements are declared as a dict in `config.py` under `access_requirements`. Each entry maps an endpoint name to `{role, power}`. Power hierarchy: `muscles < clever < all`. Enforcement happens via `authorize_access()` injected as a FastAPI dependency.

**Observability is opt-in:** Set `OBSERVABILITY=true` to activate `PrometheusMiddleware`, the `/metrics` endpoint, and OpenTelemetry tracing to Tempo. Off by default to keep the container lightweight.

**Authentication:** All protected endpoints use `Depends(get_current_user)` which validates a Zitadel-issued JWT (RS256) via JWK. Configure `ZITADEL_DOMAIN`, `ZITADEL_JWK_URL`, and related vars (see `config.py` or `docker-compose.yaml` for the full list).

**Version management:** `app/__version__.py` is the single source of truth. CI reads it for image tags. Use `cz bump` (commitizen) to increment — it reads conventional commit prefixes and updates the file automatically.

## Release & GitOps

All CI/CD is in `.github/workflows/`. The release flow is:

```
feature/* → PR → dev  →  dev-release.yaml  →  image: dev-vX.Y.Z-{ts}-{sha}
                                              →  dispatch: dev-environment-update → GitOps repo
[add create-rc🚀 label or run manual-create-rc workflow]
rc/vX.Y.Z → PR → main  →  rc-release.yaml   →  image: vX.Y.Z-rcN → staging
[merge RC PR]           →  prod-release.yaml →  image: vX.Y.Z + latest → prod + GitHub Release
hotfix/* → PR → main   →  hotfix-release.yaml → skips staging, direct to prod
```

GitOps repo (`KUBECORE_KUBEPROJECT_REPO`) receives `repository_dispatch` events. It updates `kubeapps/{app_name}/overlays/{env}/` and ArgoCD syncs the cluster.

**Required repo secrets/variables:** `KUBECORE_REGISTRY`, `KUBECORE_REGISTRY_UNAME`, `KUBECORE_REGISTRY_PWD`, `KUBECORE_KUBEPROJECT_REPO`, `KUBECORE_APP_ID`, `KUBECORE_APP_PKEY`.

## Claude Code Entrypoints

### `/release-manager` skill

Use `/release-manager` to get release guidance inline. The skill handles:
- Status checks, tag format questions, and pipeline explanations directly
- Complex multi-step operations (end-to-end promotion, hotfix orchestration, pre-prod checklist validation) by delegating to the **release-manager sub-agent**

### `release-manager` agent (`.claude/agents/release-manager.md`)

A specialized sub-agent for autonomous release orchestration. Invoked automatically by the skill for complex tasks, or explicitly via the Task tool with `subagent_type: release-manager`. Knows overlay paths, dispatch event types, and how to verify end-to-end with `gh` and `kubectl`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/novelcore) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
