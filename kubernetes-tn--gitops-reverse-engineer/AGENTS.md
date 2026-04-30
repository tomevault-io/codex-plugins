This is a Kubernetes ValidatingWebhook admission controller written in Go 1.21.

## Architecture

- The controller intercepts CREATE/UPDATE/DELETE operations and commits resource YAML to a Git repository.
- It **never denies** requests — it is observability-only.
- Resources are cleaned before commit (kubectl-neat: no uid, timestamp, status, managedFields).
- Secret values are obfuscated (`********`) before commit — keys and structure are preserved.

## Code Conventions

- Single-package `main` — all Go files are in the repo root (no `pkg/`, `internal/`, `cmd/`).
- `main.go` — HTTP server, webhook handler, health/metrics endpoints.
- `gitea.go` — GitClient interface implementation (Gitea, GitHub, GitLab providers).
- `neat.go` — Resource cleanup logic (remove runtime fields).
- `config.go` — Configuration loading from environment and ConfigMap.
- `metrics.go` — Prometheus metric definitions.
- Git path structure: `{cluster}/{namespace}/{lowercase-kind}/{name}.yaml` for namespaced resources, `{cluster}/_cluster/{lowercase-kind}/{name}.yaml` for cluster-scoped.
- Resource kinds in file paths are **always lowercase** (`strings.ToLower(kind)`).

## Helm Chart

- Chart is in `chart/` with Bitnami-style values structure.
- Use `helm template ... | kubectl apply -f -` — not `helm install` in the Makefile.
- The chart supports `image.registry`, `image.repository`, `image.tag`, `image.digest`.
- Cluster-specific values follow the convention `values/values.{cluster}.yaml`.

## Testing

- Unit tests: `go test ./...` (no build tags).
- E2E tests: `go test -tags=e2e ./e2e/...` — requires Kind + Gitea (see `e2e/setup.sh`).
- E2E tests use the `e2e` build tag to avoid running during unit test passes.

## CI/CD

- `build.yaml` — CI: vet, test, multi-platform Docker build, Helm lint.
- `release.yaml` — Release: test, multi-platform push to GHCR, Helm OCI push, GitHub Release.
- `e2e.yaml` — E2E: Kind cluster, Gitea, full admission→git pipeline test.

## Dependencies

- `go-git/v5` for Git operations (clone, commit, push).
- `k8s.io/client-go` for e2e test k8s API interaction.
- No external frameworks — standard `net/http`, `encoding/json`, `gopkg.in/yaml.v3`.

## Important Rules

- The webhook must always return `Allowed: true` — never block cluster operations.
- Never commit real secret values to Git — always obfuscate.
- Git author comes from the Kubernetes user/serviceaccount performing the operation.
- CGO is disabled (`CGO_ENABLED=0`) for static binaries.

---
> Source: [kubernetes-tn/gitops-reverse-engineer](https://github.com/kubernetes-tn/gitops-reverse-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
