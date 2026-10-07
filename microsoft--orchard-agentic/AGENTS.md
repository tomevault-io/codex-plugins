# Copilot Instructions for Orchard-Agentic

## Architecture

This repository is the **Orchard-Agentic** research collection. **Orchard** is
the foundational paper and framework in the collection; established artifact
names such as **Orchard Env**, **Orchard-SWE**, **Orchard-GUI**, and
**Orchard-Claw** remain unchanged. The toolkit trains and evaluates agents in
real, isolated execution environments. Three top-level components:

- **`orchard_env/`** — a Kubernetes-based sandbox orchestration service for multi-turn agent↔sandbox interactions (e.g. SWE-bench). Importable package `orchard_env` (`orchard_env/orchard_env/`). This is what the rest of this document describes.
- **`orchard_eval/`** — the evaluation suite. **The importable package is `orchard_evalkit` (`orchard_eval/orchard_evalkit/`), deliberately not the same name as the directory holding it** — the distribution and CLI are both `orchard-eval`. Runs any harness (`codex`, `claude`, `opencode`, `pi`, `mini-swe-agent`) against SWE-bench Verified / Multilingual / Pro with `orchard-eval run`, and against Harbor-format benchmarks (Terminal-Bench 2.1, DeepSWE 1.1) with `orchard-eval harbor`. Configs live in `orchard_eval/configs/`, driver scripts in `orchard_eval/scripts/`.
  - **`orchard_eval/harbor_orchard/`** — nested inside the suite but a *separate* distribution (`harbor-orchard`, package `harbor_orchard`). It provides the Harbor environment provider that Harbor loads by import path as `harbor_orchard:OrchardEnvironment`, translating a task's Dockerfile into commands and its mounts into transfers so a Harbor task runs on a pod.
- **`trainer/slime/`** — the RL trainer fork, vendored as a git submodule ([MSR-Orchard/slime](https://github.com/MSR-Orchard/slime)). A plain `git clone` leaves it empty; run `git submodule update --init trainer/slime`.

**Unless stated otherwise, all paths below are relative to `orchard_env/`.** Three main layers:

1. **Client SDK** (`orchard_env/client/sandbox_client.py`) — Sync (`SandboxClient`) and async (`AsyncSandboxClient`) Python clients. Both use context managers for lifecycle management. `SandboxInstance` / `AsyncSandboxInstance` handle exec, file ops, and patching. Public API is re-exported from `orchard_env/__init__.py`.

2. **Orchestrator** (`orchard_env/orchestrator/`) — FastAPI service managing sandbox lifecycle. Key components:
   - `api.py` — All routes defined directly (no APIRouter), request-ID middleware, API-key auth via `X-API-Key` header
   - `sandbox_manager.py` — Creates/deletes pods in a shared `sandbox-pods` namespace, manages network policies
   - `exec_manager.py` — Submits exec jobs, runs them under per-sandbox locks via the in-pod agent
   - `agent_client.py` — Direct HTTP calls to pod IPs (bypasses K8s API server for exec/file hot paths)
   - `job_store.py` / `redis_job_store.py` — Job state storage (in-memory or Redis for multi-replica)
   - `pod_watcher.py` — K8s Watch/Informer for cached pod status

3. **Sandbox Agent** (`orchard_env/agent/server.py`) — Lightweight FastAPI server injected into every sandbox pod. Handles `/exec`, `/files/upload`, `/files/download`, `/files/list`. Injected via init container (`Dockerfile.agent-injector`) that bundles a self-contained Python interpreter so it works with ANY user image.

### Exec flow

Client calls `POST /sandboxes/{id}/exec` with `wait=True` → orchestrator runs command via agent HTTP → returns result. If the server-side wait times out (response status is `"running"`), the client falls back to polling `GET /jobs/{id}`. **Critical**: a `"running"` status from the server-side wait means the job is still executing — never treat it as complete.

### Container images

- `Dockerfile` — Orchestrator (`python -m orchard_env.orchestrator.main`); build context is `orchard_env/`
- `Dockerfile.sandbox` — Sandbox with baked-in agent (for known images)
- `Dockerfile.agent-injector` — Init container that copies agent + bundled Python into any user image via emptyDir volume

## Build, Test, and Lint

```bash
# Install (editable, from the repo root)
pip install -e "orchard_env[dev]"

# Lint
ruff check .
black --check .

# Format
black .

# Unit tests (offline, no orchestrator needed) — all live in tests/
python -m pytest

# Run a single test
python -m pytest tests/test_exec_timeout.py::TestJobResult::test_running_is_not_complete -v

# Integration scripts (require a running orchestrator + SANDBOX_BASE_URL/SANDBOX_API_KEY).
# These live in tests/integration/, are NOT named test_*.py, and are excluded
# from pytest collection because importing them fires real network calls.
python tests/integration/soak.py
python tests/integration/sandbox_tools.py
python tests/integration/bench_concurrent.py

# Build container images (run from orchard_env/, requires registry access)
./scripts/build_push.sh
```

### The other components

```bash
# Eval suite — from the repo root. Package `orchard_evalkit`, CLI `orchard-eval`.
# orchard-env is a dependency and lives in this repo, so both go in one command.
pip install -e orchard_env -e "orchard_eval[all]"
cd orchard_eval && python -m pytest -q

# Harbor provider — must be importable by the `harbor` process itself, so both
# land in the same environment.
python -m pip install harbor
python -m pip install -e orchard_env -e orchard_eval/harbor_orchard
python -m pytest orchard_eval/harbor_orchard/tests
```

**Import gotcha**: `orchard_eval/` contains a `harbor_orchard/` directory with no `__init__.py`. When the distribution is not installed, `python -c "import harbor_orchard"` run from there resolves to an empty namespace package instead of failing, which surfaces later as a misleading `no attribute 'OrchardEnvironment'`. Verify with `cd /tmp && python -c "import harbor_orchard as m; assert m.__file__"` — the `cd` is what makes the check meaningful.

## Key Conventions

### Configuration

- **Orchestrator**: `pydantic-settings` (`orchard_env/orchestrator/settings.py`), configured via env vars or `.env` file. Singleton `settings` object imported throughout.
- **Client SDK**: Constructor params take priority over env vars (`SANDBOX_BASE_URL`, `SANDBOX_API_KEY`, `SANDBOX_PREFIX`).

### Error handling

- Client retries on connection errors, timeouts, and 503s with exponential backoff + jitter (3 retries, 1s/2s/4s base).
- Cleanup is always best-effort — exceptions during sandbox deletion are silently caught.
- Orchestrator uses HTTP status codes consistently: 401/403 for auth, 404 for missing resources, 408 for wait timeouts, 503 for transient overload.

### Logging

- Orchestrator uses structured JSON logging by default (`orchard_env/orchestrator/utils.py`). Request IDs are propagated via `ContextVar`.
- Agent uses plain text logging (`%(asctime)s [%(levelname)s]` format).

### Python

- Target: Python 3.11+
- Formatting: `black` (line-length 88)
- Linting: `ruff` (rules: E, F, I, N, W, UP; E501 ignored)
- Async: `pytest-asyncio` for async test support

### Client SDK patterns

- Always use context managers (`with SandboxClient() as client:` / `async with AsyncSandboxClient() as client:`)
- `SandboxInstance.exec()` returns a `JobResult` — check `.succeeded`, `.failed`, or `.is_complete`
- Job statuses: `"queued"` → `"running"` → `"succeeded"` | `"failed"`
- Sync client registers `atexit` + signal handlers for cleanup; async client cleans up on `__aexit__`

### Kubernetes

- Sandboxes are pods in a shared `sandbox-pods` namespace, named `sandbox-{sandbox_id}`
- Network isolation via Calico NetworkPolicy (deny-all-egress default, per-sandbox allow when `block_network=False`)
- Dual node pool architecture: `sys` (system components) + `sbx` (sandbox pods, with `workload=sandbox` node selector)
- K8s API calls are throttled with semaphores and retried with backoff

---
> Source: [microsoft/Orchard-Agentic](https://github.com/microsoft/Orchard-Agentic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-07 -->
