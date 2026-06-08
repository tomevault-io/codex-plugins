# GitHub Copilot Instructions for ocabox-tcs

## Repository Overview

This is the **OCM Telescope Control Services (ocabox-tcs)** repository - a universal Python service framework for telescope control systems built around NATS messaging. The framework is execution-method agnostic and supports multiple deployment scenarios.

## Core Architecture

### Universal Service Framework

- **Service Independence**: Services work across manual/process/asyncio/systemd execution methods
- **Service Types**: Permanent, blocking permanent, and single-shot services
- **Decorator-Based**: Modern Python decorators (`@service`, `@config`) for clean registration
- **Filename-Based Discovery**: Service type automatically derived from filename (e.g., `hello_world.py` → `hello_world`)
- **Optional Configuration**: Config classes are optional, services can use base config
- **Distributed Management**: NATS-based service discovery, management, and monitoring
- **Flexible Deployment**: Services in local or external packages

### Key Directories

```
src/
├── ocabox_tcs/
│   ├── base_service.py          # Base classes and decorators
│   ├── service_controller.py    # Single service lifecycle control
│   ├── process_context.py       # Shared resources (Singleton)
│   ├── launchers/               # Execution launchers
│   │   ├── process.py           # Process-based launcher
│   │   ├── asyncio.py           # Asyncio-based launcher
│   │   └── base_launcher.py     # Base classes
│   ├── monitoring/              # Monitoring system
│   │   ├── status.py            # Status enum and StatusReport
│   │   ├── monitored_object.py  # Base monitoring classes
│   │   └── monitored_object_nats.py  # NATS-enabled monitoring
│   └── services/                # Available services
│       ├── hello_world.py       # Canonical template service
│       └── examples/            # Tutorial examples (01-05)
└── tcsctl/                      # CLI tool (separate package)
    ├── app.py                   # Main CLI entry point
    ├── client.py                # Service control client
    └── display.py               # Rich terminal output
```

## Development Guidelines

### Code Style

- **Python Version**: 3.12+ required
- **Formatter**: Black with 100 character line length (`poetry run black .`)
- **Linter**: Ruff for catching real issues (gentle rules, non-annoying)
- **Import Style**: Absolute imports (e.g., `from ocabox_tcs.base_service import service`)
- **Comments**: Only add comments if they match existing style or explain complex logic
- **Naming**: Use descriptive names, follow existing patterns

### Framework Naming Conventions

**Framework-Provided Attributes** (prefixed with `svc_` to avoid collisions):
- `self.svc_logger`: Logger instance configured for the service
- `self.svc_config`: Service configuration object
- `self.controller`: Reference to ServiceController
- `self.monitor`: Monitoring object for status/health reporting
- `self.is_running`: Boolean property indicating if service is running

**Users are FREE to use**:
- `self.logger`, `self.config`, etc. for their own purposes
- Any non-prefixed attribute names

### Service Creation Patterns

#### Non-Blocking Permanent Service (for async tasks/workers)

```python
from ocabox_tcs.base_service import service, BasePermanentService

@service
class MyService(BasePermanentService):
    async def start_service(self):
        # Spawn background tasks/workers
        self.task = asyncio.create_task(self.worker())

    async def stop_service(self):
        # Clean up background tasks
        self.task.cancel()
        await self.task
```

#### Blocking Service with Main Loop (most common pattern)

```python
from ocabox_tcs.base_service import service, BaseBlockingPermanentService

@service
class WorkerService(BaseBlockingPermanentService):
    async def on_start(self):
        """Optional: Called before run_service starts - for setup."""
        pass

    async def run_service(self):
        """REQUIRED: Main service loop - runs in managed task."""
        while self.is_running:
            # Main work loop
            await asyncio.sleep(1)

    async def on_stop(self):
        """Optional: Called after run_service stops - for cleanup."""
        pass
```

**CRITICAL**: `BaseBlockingPermanentService` enforces its API at import time:
- ✅ **Override**: `run_service()` - your main loop (required)
- ✅ **Override**: `on_start()`, `on_stop()` - optional hooks
- ❌ **DO NOT override**: `start_service()`, `stop_service()` - these manage task lifecycle
- Violating this raises `TypeError` at class definition time

#### Single-Shot Service

```python
from ocabox_tcs.base_service import service, BaseSingleShotService

@service
class DataImporterService(BaseSingleShotService):
    async def execute(self):
        # One-time task
        pass
```

### Status Management

**The framework automatically manages status transitions!** Manual calls are usually unnecessary.

**Automatic Status Lifecycle**:
1. `STARTUP` - Set by controller during initialization
2. `OK` - Set by controller after `start_service()` completes
3. `SHUTDOWN` - Set by controller during `stop_service()`
4. `FAILED` / `ERROR` - Set by controller on exceptions

**For Advanced Use Cases** (see `examples/04_monitoring.py`):
- **Healthcheck callback (recommended)**:
  ```python
  def healthcheck(self) -> Status:
      if self.error_count > 0:
          return Status.DEGRADED
      return Status.OK
  ```
- **Manual status override (rarely needed)**: `self.monitor.set_status(Status.ERROR, "message")`

**Most services don't need any status management code!**

### NATS Messaging Patterns

Use `serverish` messenger library for all NATS communication.

**Key Concepts**:
1. **Message Structure**: All messages have `data` and `meta` sections
2. **Timestamp Format**: 7-element int array `[year, month, day, hour, minute, second, microsecond]` (UTC)
3. **Specialized Publishers**:
   - `MsgPublisher` / `MsgReader`: Multiple messages, JetStream persistence
   - `MsgSinglePublisher` / `MsgSingleReader`: Single values (e.g., config)
   - `MsgRpcRequester` / `MsgRpcResponder`: Request/Response (Core NATS)

**NATS Subject Schema**:
- Status updates: `svc.status.<service_name>`
- Registry events: `svc.registry.<event>.<service_name>` (events: `declared`, `start`, `ready`, `stopping`, `stop`, `crashed`, `restarting`, `failed`)
- Heartbeats: `svc.heartbeat.<service_name>`
- RPC commands: `svc.rpc.<service_name>.v1.<command>`
- Service naming: `<service_type>.<instance_context>` (e.g., `guider.jk15`)

**JetStream Streams**:
- `svc_registry`: Lifecycle events (persistent, indefinite retention)
- `svc_status`: Status updates (30 days)
- `svc_heartbeat`: Heartbeat messages (1 day, in-memory)

### Testing Practices

- **Test Suite**: 83 tests covering lifecycle, crash scenarios, monitoring, restart policies
- **Run Tests**: `poetry run pytest` (expect ~6 minutes)
- **Async Tests**: Use `pytest-asyncio` with `asyncio_mode = "auto"`
- **Manual Tests**: Tests requiring external services marked with `@pytest.mark.manual`
- **Test Structure**: 
  - Unit tests: `tests/unit/`
  - Service type tests: `tests/service_types/`
  - Integration tests: Root level test files

### Common Development Commands

```bash
# Install dependencies (includes CLI tools automatically for devs)
poetry install

# Run process launcher (services in separate processes)
poetry run tcs_process --config config/services.yaml

# Run asyncio launcher (services in same process)
poetry run tcs_asyncio --config config/examples.yaml

# List running services
poetry run tcsctl
poetry run tcsctl --detailed
poetry run tcsctl --verbose

# Manual service start (all parameters optional)
python src/ocabox_tcs/services/hello_world.py                          # Minimal: uses defaults
python src/ocabox_tcs/services/hello_world.py prod                     # Custom context, no config
python src/ocabox_tcs/services/hello_world.py config.yaml prod         # Full specification

# Run tests
poetry run pytest

# Code formatting
poetry run black .

# Linting
poetry run ruff check .
```

### Configuration System

- **Priority Order**: Command-line args → NATS config (planned) → YAML file → defaults
- **Main Config**: User creates `config/services.yaml` from `config/services.sample.yaml`
- **Tutorial Config**: `config/examples.yaml` for learning examples
- **Service Matching**: By `type` (filename or subdir.filename) + `instance_context` in services array
- **Config Classes**: Optional, with filtering to valid fields only

### Restart Policies (Systemd-inspired)

Configurable via YAML: `restart`, `restart_sec`, `restart_max`, `restart_window`

- `"no"`: Never restart (default)
- `"on-failure"`: Restart only on non-zero exit codes
- `"on-abnormal"`: Restart on crashes, signals (not clean exits)
- `"always"`: Always restart regardless of exit code

See `doc/restart-policies.md` for complete user guide.

### CLI Tool (tcsctl)

**Service Visibility Rules**:
- **Default mode**: Shows running services (except old ephemeral) + fresh ERROR/FAILED services
- **`--show_all` mode**: Shows declared services + fresh services (hides old ephemeral)
- **Service filtering** (by name): Shows matching services regardless of visibility rules

**Features**:
- Hierarchical display: Services grouped under launchers with tree indentation
- Zombie process detection: ⚠ red for missing heartbeat
- ERROR/FAILED indicator: Red × symbol even for stopped services
- Detailed view: `--detailed` shows runner_id, hostname, PID, timestamps
- Collection statistics: `--verbose` shows message counts and timing

### Important Implementation Notes

1. **Service Type Derivation**: Automatically from filename - no manual type specification needed
2. **Config Heuristics Removed**: No warnings if `@config` decorator not used
3. **External Service Creation**: `config_file` and `instance_context` are optional with smart defaults
4. **API Enforcement**: `BaseBlockingPermanentService` validates correct method overrides at import time
5. **Logger Names**: Compact logger names (`ctx`, `cfg`, `ctrl`, `svc`, `launch`, `run`, `mon`)
6. **Path-Aware Discovery**: Service discovery supports subdirectories

### Dependencies

- **NATS**: Message broker for service communication (configurable host/port, defaults to localhost:4222)
- **ocabox**: Core observatory control library
- **serverish**: NATS messaging integration and helpers

### Example Services

Tutorial examples (`src/ocabox_tcs/services/examples/`):
- `01_minimal.py` - Absolute minimum service (< 30 lines)
- `02_basic.py` - Service with configuration
- `03_logging.py` - Logging best practices
- `04_monitoring.py` - Monitoring and health checks
- `05_nonblocking.py` - Non-blocking service with background workers
- `README.md` - Comprehensive Getting Started guide

### Best Practices

1. **Use existing ecosystem tools**: Prefer scaffolding tools (npm init, yeoman) over manual file creation
2. **Minimal modifications**: Change as few lines as possible to achieve goals
3. **Never remove working code**: Unless absolutely necessary or fixing security vulnerabilities
4. **Follow existing patterns**: Match code style and structure in the repository
5. **Test iteratively**: Run linters, builds, and tests frequently after changes
6. **Progressive examples**: Add simple examples before complex ones
7. **Documentation**: Update docs if directly related to changes
8. **Security**: Always validate changes don't introduce vulnerabilities

### Current Status (as of 2025-11-16)

**Recently Completed**:
- ✅ Framework attribute renaming (v0.4): `logger` → `svc_logger`, `config` → `svc_config`
- ✅ Simplified external service creation: All parameters optional with smart defaults
- ✅ Test suite complete: 83/83 tests passing
- ✅ Crash handling & restart policies: Systemd-inspired with full configuration
- ✅ Ephemeral service filtering in tcsctl: Distinguishes formal config from ad-hoc services
- ✅ CLI tool (tcsctl): Separate package with optional installation
- ✅ Launcher monitoring: Hierarchical display with tree-like indentation

### Next Potential Features

See `doc/requirements-analysis.md` for detailed implementation plans.

Recommended priorities:
1. CLI Continuous Mode - Live-updating display with `--follow` flag
2. Task Context Manager - Zero-boilerplate task metrics tracking
3. Multi-Component Services - Display service sub-components
4. External Service Packages - Support services from external packages
5. RPC Service Control - Remote service control commands

## Common Pitfalls to Avoid

1. **Don't override `start_service()`/`stop_service()` in `BaseBlockingPermanentService`** - Use `run_service()` instead
2. **Don't manually set status in most cases** - Framework handles it automatically
3. **Don't forget `@service` decorator** - Required for service registration
4. **Don't use relative imports** - Always use absolute imports
5. **Don't add config class unless needed** - Base config is sufficient for many services
6. **Don't commit `config/services.yaml`** - Create from `services.sample.yaml` locally
7. **Don't remove tests** - Could lead to missing functionality

## Questions or Issues?

- Check examples in `src/ocabox_tcs/services/examples/`
- Review documentation in `doc/`
- See `README.md` for installation and setup
- Full test suite demonstrates all patterns: `poetry run pytest`

---
> Source: [araucaria-project/ocabox-tcs](https://github.com/araucaria-project/ocabox-tcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-07 -->
