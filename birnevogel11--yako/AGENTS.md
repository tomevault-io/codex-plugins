# Copilot Instructions for Yako

Yako is a mock-based testing framework for Ansible playbooks. It intercepts Ansible task execution via a custom callback plugin to run assertions and inject mocks, enabling unit-style testing of roles and playbooks without real infrastructure.

## Build & Test Commands

This project uses [uv](https://docs.astral.sh/uv/) for package management and requires Python 3.13+.

```shell
# Install dependencies
uv sync

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/unit/test_given.py

# Run a single test by name
uv run pytest -k "test_parse_files_field"

# Run only unit or integration tests
uv run pytest tests/unit/
uv run pytest tests/integration/ -m integration

# Lint
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/

# Type check
uv run mypy src/
```

## Architecture

### Core Flow

```
CLI (yako test)
  → Config loading (yako.yaml + env)
  → Test discovery (find test*.yaml files)
  → TestSuite building (modules → cases, expand parametrize)
  → Runner selection (local or docker)
  → For each TestCase:
      → Runner.init() — resolve roles, generate ansible.cfg with yako plugins
      → Runner.run() — execute ansible-playbook as subprocess
      → yako_callback plugin intercepts tasks → runs mocks & assertions
  → Report results
```

### Key Modules

- **`cli/main.py`** — Typer CLI entry point. Commands: `test`, `list`, `test-callback`.
- **`config.py`** — Pydantic-based config from `yako.yaml`, `yako_local.yaml`, env vars. Three-level merge: global → module → case.
- **`test_module.py`** / **`test_case.py`** — Test discovery and data models. A `TestModule` is one YAML file containing multiple `TestCase`s.
- **`given.py`** — Test setup: files to copy, extra_vars to inject, mock_tasks to intercept. Merges across config/module/case levels.
- **`assert_check.py`** — Assertion logic with modes: `==`, `!=`, `<`, `>`, `in`, `not_in`, `is_none`, `is_true`, etc.
- **`runner/`** — `LocalTestCaseRunner` and `DockerTestCaseRunner` behind a `TestCaseRunner` protocol. Selected via `runner_mode` in config.
- **`plugins/callback/yako_callback.py`** — Ansible callback plugin that intercepts task execution to apply mocks and run `assert_inputs`/`assert_outputs`.
- **`plugins/module/yako_mock.py`** — Ansible module that replaces real modules with mocked results.
- **`plugins/module/yako_assert.py`** — Ansible module for running assertions as tasks.
- **`repo.py`** — Git repo cloning and caching via `diskcache` at `~/.cache/yako/repos/`.

### Test YAML Structure

See README.md for full documentation on writing test cases. Tests are YAML files (`test_*.yaml`) under `tests/yako/` with this structure:

```yaml
given:           # Module-level (applies to all test cases in file)
  extra_vars: {}
  files: []
  mock_tasks: []

test_cases:
  - name: "test_name"
    given:       # Case-level (merges with module and global)
      mock_tasks:
        - name: "Task name to mock"
          mock: { changed: true, result_dict: {} }
          assert_inputs: [{ name: "var", value: "expected", mode: "==" }]
          assert_outputs: [{ name: "var", value: "expected", mode: "==" }]
    playbooks: ["playbook.yaml"]   # OR tasks: [{...}], not both
    parametrize:
      variant_1:
        extra_vars: { key: value }
```

## Conventions

- **Pydantic everywhere** — All configs, test models, and data structures are Pydantic `BaseModel`s with validators. Use `model_validate()` for construction from dicts/YAML.
- **Three-level config merging** — `given` blocks merge from global (`yako.yaml`) → module-level → case-level. `files` and `mock_tasks` concatenate; `extra_vars` uses `ChainMap` (later wins).
- **Ruff with ALL rules** — Linting uses `ruff` with `select = ["ALL"]` and a large explicit ignore list. Check `pyproject.toml` for which rules are disabled.
- **`S101` allowed in tests** — `assert` statements are permitted in test files via per-file-ignores.
- **inline-snapshot for tests** — Unit tests use the `inline-snapshot` library for snapshot assertions. Update snapshots with `--inline-snapshot=update`.
- **Versioning via setuptools-scm** — Version is derived from git tags, not hardcoded.
- **Runner protocol** — New runners implement the `TestCaseRunner` protocol (`init()` and `run()` methods).

---
> Source: [birnevogel11/yako](https://github.com/birnevogel11/yako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
