# Wisp — SoundTouch Service

Self-hosted home automation service for Bose SoundTouch speakers. Provides scheduled on/off routines with volume fade-in/fade-out, a PWA web UI, and a REST API. Deployed as a Docker container on a Synology NAS.

## Essential Commands

```bash
# Run (always from repo root)
python -m app.main                         # default port 9001
PORT=8080 python -m app.main               # override port

# Test
pytest -q                                  # unit + validation tests
python tests/test_api.py --speaker "Name"  # live hardware smoke test (not pytest)

# Lint / type-check
ruff check .
mypy                                       # only checks files listed in pyproject.toml
pre-commit run --all-files                 # runs ruff + mypy via hooks

# Docker (host networking required for mDNS)
cp deployment/config.json ./config.json
docker run -d --name wisp --network host \
  -v $(pwd)/config.json:/workspace/config.json \
  -p 9001:9001 brisebois/soundtouch-service:latest

# Docker Compose (Synology NAS)
cd deployment && docker compose up -d
```

## Architecture

| Component | File | Role |
|---|---|---|
| App factory | `app/main.py` | Flask app creation, Blueprint registration, daemon startup |
| API routes | `app/api/routes.py` | All REST + PWA endpoints (`api_bp` Blueprint) |
| Scheduler | `app/scheduler/jobs.py` | 10 s poll loop; `auto_on_job` / `auto_off_job` via `ThreadPoolExecutor` |
| Discovery | `app/core/discovery.py` | mDNS scan via zeroconf; 5-min background IP cache refresh |
| Speaker cache | `app/core/speaker_cache.py` | Persistent WebSocket per speaker; real-time status/volume |
| Status | `app/core/status.py` | HTTP GET to SoundTouch `/now_playing` + `/volume`; XML → dict |
| Control | `app/core/control.py` | HTTP POST to SoundTouch `/key` and `/volume` |
| Models | `app/core/models.py` | TypedDict types: `Schedule`, `SchedulePayload`, `ConfigDocument` |
| Constants | `app/core/constants.py` | All numeric tunables (ports, timeouts, defaults) |

**Startup sequence:** `create_app()` → `jobs.start_daemon()` (scheduler thread + config-write worker thread) → `discovery.start_device_cache()` (immediate mDNS scan + 5-min refresh loop → `speaker_cache.start_ws_listeners`).

**Config writes** are serialized through `queue.Queue[ConfigMutation | None]` with a dedicated worker thread to prevent concurrent file corruption.

## Code Conventions

- **Framework:** Flask 3 + Flasgger (Swagger UI at `/apidocs`). Not FastAPI.
- **Models:** `TypedDict` — not Pydantic or dataclasses.
- **Validation:** Manual helpers (`_coerce_int`, `_coerce_non_negative_number`, `_validate_schedule_payload`). No validation library.
- **Globals:** Module-level singletons (`current_config`, `_device_cache`, `_speaker_state`) protected by `threading.Lock()`.
- **Logging:** `logging.getLogger(__name__)` per module; `_log_with_fields(level, msg, **fields)` for structured extra fields. No `print()` calls.
- **Error handling:** Catch `RequestException` / `ET.ParseError`; return `None` or `{"status": "OFFLINE"}`. Exceptions must not bubble to callers.
- **Naming:** `snake_case` everywhere; private helpers prefixed with `_`.
- **Line length:** 100. **Target:** Python 3.14+.
- **Scheduler:** Hand-rolled `while True` / `time.sleep(10)` — no APScheduler.
- **Commit style:** `feat:`, `fix:`, `docs:` prefixes.

## Pitfalls

- **Import path:** always `python -m app.main` from repo root — `python app/main.py` breaks relative imports.
- **Host networking:** `--network host` is required for mDNS speaker discovery. Local containers without host networking will fail device discovery — this is expected, not a startup regression.
- **Timezone:** `TZ` env var is critical for schedule matching (e.g. `TZ=America/New_York`). Wrong or missing `TZ` causes silent scheduling skew.
- **`source` vs `preset`:** Mutually exclusive in schedule config; `source` only accepts `"AUX"`.
- **mypy scope:** Only checks the 6 files listed in `pyproject.toml` — not the full codebase.
- **`test_api.py`:** Standalone live-hardware smoke test — not a pytest file. Requires real speakers on the network.

## Key References

- Config schema & environment variables: [README.md](../README.md)
- REST API surface: Swagger at `http://<host>:9001/apidocs` or [README.md](../README.md)
- Contributing workflow: [CONTRIBUTING.md](../CONTRIBUTING.md)
- Architecture review skill: [.github/skills/python-architecture-review/SKILL.md](skills/python-architecture-review/SKILL.md)
- Completed feature history: [task.md](../task.md)

---
> Source: [AlexandreBrisebois/soundtouch-service](https://github.com/AlexandreBrisebois/soundtouch-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-28 -->
