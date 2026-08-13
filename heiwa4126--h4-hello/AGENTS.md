# AI Coding Agent Instructions for h4-hello

## Project Overview

This is a practice project for publishing Python packages to PyPI with PEP740 digital signatures using **uv** as the build backend and package manager. The project contains only a simple `hello()` function but demonstrates modern Python packaging workflows with Trusted Publishing and Sigstore attestations.

## Architecture & Key Components

### Package Structure

- `src/h4_hello/` - Source package using src-layout pattern
- `_core.py` - Core function implementation (`hello()` function)
- `__main__.py` - CLI entry point (exposed as `h4-hello` command)
- `_core_test.py` - Co-located test files (excluded from builds via `tool.uv.build-backend`)
- `__init__.py` - Package initialization, exports `hello` and `__version__`

### Build System

- **uv** as build backend (`uv_build`) - NOT setuptools/hatchling
- Uses modern `pyproject.toml` configuration exclusively
- Test files auto-excluded via `source-exclude` and `wheel-exclude` patterns

## Development Workflows

### Python Environment

This project uses **uv** with a local `.venv` directory. When running shell commands that need Python:

```bash
# Activate the virtual environment first
source .venv/bin/activate

# Then run Python commands
python --version
```

Alternatively, use `uv run` to automatically use the correct environment:

```bash
uv run python --version
```

### Task Management with poethepoet

Use `poe <task>` commands defined in `poe_tasks.toml`:

```bash
poe test          # Run pytest tests
poe check         # Ruff linting
poe mypy          # Type checking
poe format        # Auto-format code (ruff, dprint, textlint)
poe lint          # Run all linters (check, mypy, pep440check, pyproject-lint)
poe lint-full     # Run all linters including actionlint
poe build         # Full build workflow (lint, test, clean, pack, smoke-test)
poe smoke-test    # Test built packages in isolation
poe testpypi      # Deploy to TestPyPI (requires .env)
poe cli           # Run h4-hello CLI
poe ex1           # Run example script
```

### Build & Test Workflow

```bash
# Full build workflow (recommended)
poe build         # Runs lint, test, clean, pack, smoke-test

# Or run steps individually
poe lint          # Run all linters
poe test          # Run pytest tests
uv build          # Creates both .tar.gz and .whl in dist/
poe smoke-test    # Test built packages in isolation

# Individual smoke tests
uv run --isolated --no-project --refresh --no-cache --with dist/*.whl h4-hello -V
uv run --isolated --no-project --refresh --no-cache --with dist/*.tar.gz h4-hello --help
uv run --isolated --no-project --refresh --no-cache --with dist/*.whl examples/ex1.py
```

### Tags & Publishing

- **TestPyPI**: Tag with `test-*` (e.g., `test-0.1.11`) triggers TestPyPI deployment
- **PyPI**: Tag with `v*` semver (e.g., `v0.1.11`) triggers PyPI deployment
- Both use GitHub Actions with Trusted Publishing (no manual tokens)

## Project-Specific Patterns

### Version Management

- Manual version bumps in `pyproject.toml` (no automated versioning)
- Beta versions supported: `0.1.12b1` (PEP440 canonical format only)
- Git tags should match pyproject.toml version

### Testing & Quality

- Tests use simple assert statements (no complex frameworks)
- Co-located test files (`*_test.py`) automatically excluded from builds
- Strict tooling: ruff, mypy, actionlint for workflows
- Development dependencies managed via `[dependency-groups]` (not `[project.optional-dependencies]`)
- Key dev tools: bumpuv, mypy, pep440check, poethepoet, pytest, ruff, validate-pyproject

### Security & Publishing

- **Trusted Publishing only** - no API tokens in workflows
- Owner-only publishing: `if: github.repository_owner != github.actor` security check job
- PEP740 Sigstore attestations enabled by default (via `pypa/gh-action-pypi-publish`)
- Environment-based deployment gates (`environment: testpypi/pypi`)
- Security check job runs before build to prevent unauthorized publishing

## Key Configuration Files

### `pyproject.toml`

- Uses `uv_build` backend (not setuptools)
- CLI entry point: `h4-hello = "h4_hello.__main__:main"`
- Build exclusions handle test files (`**/*_test.py` excluded)
- Test files co-located with source code

### `poe_tasks.toml`

- Executor set to `uv` (not default Python)
- Environment variables for commands
- Grouped tasks for complex workflows

### GitHub Actions

- Reusable workflow pattern: `build-package.yml` called by publish workflows
- Pinned action versions with commit SHAs (security best practice)
- Artifact-based build/publish separation

## Integration Points

- **uv** for all Python operations (no pip/poetry mixing)
- **poethepoet** for task running (not make/npm scripts)
- **GitHub Actions** with OIDC for publishing (no manual secrets)
- **TestPyPI** for validation before PyPI release

## Prompt Files

When creating plan or prompt files in `.github/prompts/`:

- Use the naming pattern: `plan-${camelCaseName}-YYYYMMDD.prompt.md`
- Include the current date as a timestamp (e.g., `plan-goodbyeFeature-20251120.prompt.md`)
- This helps track when plans were created and allows multiple iterations

## Common Pitfalls

- Don't use `pip install -e .` - use `uv sync` for development
- Build backend is `uv_build`, not `setuptools.build_meta`
- Test files are excluded via pyproject.toml, not .gitignore
- Publishing requires repository owner permissions (security feature)
- Pre-release versions must use PEP440 canonical format (e.g., `0.1.14a4` not `0.1.14-alpha4`)

---
> Source: [heiwa4126/h4-hello](https://github.com/heiwa4126/h4-hello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
