# Copilot Instructions

## Architecture & Runtime
- `cmd/mcm/main.go` boots `internal/app.App` for dependency init, then exposes `http.Server` on `:8080` using `internal/transport/http.NewMux` (which now needs both a schema readiness callback and a `course.Store`).
- Keep lifecycle logic (`Start`/`Stop`) inside `internal/app/app.go`; wire new resources there so the graceful shutdown block can drain them.
- HTTP handlers currently live in `internal/transport/http/handler.go` using the stdlib mux. Prefer small handler funcs registered inside `NewMux` and call domain code in lower layers (e.g., `pkg` packages) instead of embedding logic inline. `/api/courses` exposes CRUD over the in-memory store for now.
- Health endpoints `/livez` and `/readyz` are consumed by Kubernetes probes in `deploy/kustomize/base/mcm-deployment.yaml`; keep them cheap and always-on.

## Coding Patterns
- Project targets Go 1.25.3 (`go.mod`); use `slog` for logging and pass contexts explicitly when adding long-running work.
- Shared, importable helpers belong under `pkg/` (see `pkg/greet` and the new `pkg/course` types + validation). Code that should remain internal to the service belongs under `internal/` (e.g., `internal/store/memory` for the dev-only store backing the `/api/courses` endpoints).
- Stick with standard library HTTP primitives; no third-party router is wired today, so adding one requires updating `httpx.NewMux` and the server wiring in `main.go`.

## Local Dev & Testing
- Run `scripts/dev.sh` before committing; it enforces `go fmt ./...`, `go vet ./...`, and `go test ./... -race`.
- For quick iterations, `go run ./cmd/mcm` starts the API locally; use `curl :8080/hello` or the probe endpoints to verify responses.
- Add table-driven tests mirroring `pkg/greet/greet_test.go` when extending pure functions or handler helpers.
- Integration tests can spin up ephemeral Postgres instances via Testcontainers (`internal/testsupport/postgres`); Docker must be available or run `go test -short` to skip them. See `internal/app/app_integration_test.go` for a pattern that preps `schema_migrations` and exercises `ensureSchema`.

## Database & Migrations
- SQL migrations live in `db/migrations` using timestamped `up/down` files (see `0001_init_schema` for the courses/students/teams schema).
- `build/docker/Dockerfile.migrate` builds a tiny image bundling the `migrate` CLI plus SQL; `make migrate-build` tags it as `$(DOCKERHUB_USER)/mcm-migrate:dev` by default.
- Kubernetes job manifests sit under `deploy/kustomize/base/migrate-job/`; the main dev/prod overlays already include this job so `make dev-apply` / `make prod-apply` will trigger migrations automatically. Use `make dev-migrate` or `make prod-migrate` when you need to run the job standalone.

## Containers & Tooling
- Multi-stage Docker build lives in `build/docker/Dockerfile` (builder: `golang:1.25`, runtime: `distroless`). If you add CGO deps, adjust `CGO_ENABLED`/base image accordingly.
- The `Makefile` wraps image builds and cluster deploys. Key targets:
  - `dev-build`: `docker build` tagging `$(DOCKERHUB_USER)/mcm:dev` via `build/docker/Dockerfile`.
  - `dev-apply` / `prod-apply`: run `kubectl kustomize` against overlays and apply into the chosen context/namespace.
  - `dev-diff`: previews Kustomize changes with `kubectl diff`.
  - `migrate-build`, `dev-migrate`, `prod-migrate`: build the migration image and launch the K8s job overlays.

## Deployment Layout & Config
- Kustomize base (`deploy/kustomize/base/`) defines the namespace, API deployment/service/ingress, Postgres (statefulset + services), Keycloak, and an init DB configmap.
- Overlays in `deploy/kustomize/overlays/dev|prod` patch the base. Dev overlay flips Keycloak to embedded `start-dev` mode via `patches/keycloak-env.yaml`; prod overlay injects resource/storage overrides.
- The API deployment expects `mcm-db-secret` (key `DATABASE_URL`) and references Keycloak at `http://mcm-keycloak.mcm.svc.cluster.local:8080`. Ensure new env vars are surfaced in both the deployment manifest and any overlays that override images or probes.
- Service names pick up the `namePrefix` (`mcm-`) defined in base kustomization, so DNS inside the namespace follows `mcm-<component>.mcm.svc.cluster.local`—mirror that when adding new cross-service calls.

## Secrets & Release Flow
- Secrets are intentionally out-of-tree. For prod, provision `postgres-superuser`, `mcm-db-secret`, `keycloak-db-secret`, and `keycloak-admin` before running `make prod-apply`.
- CI/CD expectations (per `deploy/kustomize/README.md`): set `GIT_SHA` when applying overlays from automation so clusters can trace the running image tag.

---
> Source: [vsamtuc/mcm](https://github.com/vsamtuc/mcm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
