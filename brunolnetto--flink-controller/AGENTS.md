
# Copilot Agent Policy for Flink Job Controller

## Project Purpose & Architecture
- Declarative Flink job lifecycle controller with security, resilience, and observability as first-class concerns.
- Core modules: `src/core` (job orchestration), `src/security` (auth, credentials, artifact verification), `src/resilience` (circuit breaker, retry), `src/observability` (logging, metrics).
- Control loop pattern: reconcile desired vs. actual state, with robust error handling and state management.

## Streamlined Developer Workflow
- **Justfile is the primary interface** for all tasks; commands are consistent across platforms and CI/CD.
- **Setup:** `just setup` (installs dependencies, configures environment, pre-commit hooks)
- **Development:** `just dev` (auto-test, auto-format)
- **Testing:**
  - `just test` (all tests: unit + integration)
  - `just test-fast` (unit tests only, <30s)
  - `just security` (security scan)
- **Quality:**
  - `just check` (quality checks)
  - `just fix` (auto-fix issues)
- **Build/Deploy:**
  - `just build` (build app)
  - `just deploy` (deploy to target)
- **Monitoring:**
  - `just monitor-start`, `just monitor-dashboards`, `just monitor-stop` (Prometheus, Grafana, Jaeger)

## Progressive Quality Gates
- Development: 70%+ coverage, optional checks, weekly security scan
- Production: 90%+ coverage, required checks, security scan every commit
- Reality-based testing: prefer real system integration over mocks; security tests use real auth flows

## Security & Observability
- Kerberos, API key, SSL/TLS for all external APIs; no hardcoded secrets
- Input validation, audit logging, credential rotation
- Structured logging (correlation IDs), Prometheus metrics, distributed tracing

## Testing Strategy
- Unit: fast, isolated, mock expensive dependencies
- Integration: real Flink cluster, real authentication
- Security: real auth, input validation, no mocked security
- Use fixtures in `tests/unit/fixtures/` for sample specs and mock responses

## Asset & Documentation Maintenance
- All assets version controlled; update milestone tracker weekly, roadmap/policies per release, workflow docs per feature
- Update README and design docs with new features; keep setup and troubleshooting guides current

## Example Patterns & Key Files
- Circuit breaker: `src/resilience/circuit_breaker.py`
- Credential management: `src/security/credentials.py`
- Artifact verification: `src/security/artifact_verifier.py`
- Job reconciliation: `src/core/reconciler.py`
- Observability: `src/observability/logging.py`, `src/observability/metrics.py`
- TDD/unit tests: `tests/unit/`

## Troubleshooting & Help
- `just status`: Check project status
- `just help`: List all commands
- `just clean`: Remove temp files
- Platform-specific setup: see `scripts/setup.sh` (Linux/macOS/WSL2), `scripts/setup.ps1` (Windows)

## Success Metrics & Best Practices
- Setup time <5 min, TDD cycle <30s, one-command deployment, real-time monitoring
- Progressive enhancement: start simple, improve coverage/quality each phase
- Code: use type hints, docstrings, PEP8, small functions, robust error handling
- Tests: descriptive names, real data, error scenarios, fast execution

---
**Agents must follow streamlined workflow, progressive quality gates, reality-based testing, and keep documentation current. See all referenced docs and the Justfile for details.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunolnetto)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/brunolnetto)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
