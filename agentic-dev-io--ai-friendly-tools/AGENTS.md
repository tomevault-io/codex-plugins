# Copilot Repository Instructions

## Project Overview

AIFT (AI-Friendly Tools) is a collection of command-line tools optimized for AI interaction. The project consists of multiple Python packages managed as a UV workspace:

- **core**: Core library with config, logging, and CLI framework
- **web**: Web intelligence suite (search, scrape, API tools)
- **mcp-manager**: DuckDB-based MCP manager with semantic search

Note: The `memo` package (Memory & AI tools) exists in the repository but is not currently included in the UV workspace configuration.

## Build and Test Commands

### Setup
```bash
# Install dependencies using UV (recommended)
uv sync --all-groups

# Alternative: Install workspace packages in development mode using pip
# Note: memo package is not in the workspace, install separately if needed
pip install -e core web mcp-manager
pip install -e memo  # Optional, not in workspace
```

### Testing
```bash
# Run all tests
uv run pytest tests/ -v

# Run tests with coverage (for workspace packages)
uv run pytest tests/ -v --cov=core/src --cov=web/src --cov=mcp-manager/src --cov-report=term-missing

# Run specific test file
uv run pytest tests/test_cli.py -v

# Quick smoke test
aift test
```

### Linting and Formatting
```bash
# Check code style with Ruff
uv run ruff check . --select E,F,I,N,W

# Format code with Ruff
uv run ruff format .

# Check formatting without changes
uv run ruff format . --check
```

### Type Checking
```bash
# Run type checker with MyPy (for workspace packages)
uv run mypy core/src web/src mcp-manager/src --strict
```

### Docker
```bash
# Build Docker image
docker buildx build -t aift-os:latest . --load

# Start services
docker-compose up -d

# Access container shell
docker-compose exec aift-os bash

# View logs
docker-compose logs -f
```

## Code Style and Conventions

### Python Standards
- **Python Version**: >=3.11 (as specified in pyproject.toml; CI tests 3.11 and 3.12)
- **Line Length**: 100 characters (configured in pyproject.toml)
- **Import Ordering**: Use Ruff's isort integration (I rules)
- **Naming**: Follow PEP 8 conventions (N rules)

### Code Quality
- Run Ruff linter before committing (`uv run ruff check .`)
- Format code with Ruff (`uv run ruff format .`)
- Type hints are encouraged; MyPy checks are run in CI with `--strict` flag
- All new features should include tests
- Maintain test coverage for modified code

### Testing Guidelines
- Place tests in the `tests/` directory at repository root
- Follow pytest conventions:
  - Test files: `test_*.py`
  - Test classes: `Test*`
  - Test functions: `test_*`
- Use descriptive test names that explain what is being tested
- Include integration tests for user-facing features

## File Organization

### Project Structure
```
ai-friendly-tools/
├── .github/                 # GitHub configuration and workflows
│   ├── workflows/          # CI/CD workflows
│   └── copilot-instructions.md
├── core/                   # Core library package (workspace member)
│   ├── src/               # Source code
│   ├── pyproject.toml     # Package configuration
│   └── README.md          # Core documentation
├── web/                   # Web tools package (workspace member)
│   ├── src/
│   ├── pyproject.toml
│   └── README.md
├── mcp-manager/           # MCP manager package (workspace member)
│   ├── src/
│   ├── pyproject.toml
│   └── README.md
├── memo/                  # Memory & AI tools package (not in workspace)
│   ├── src/
│   └── pyproject.toml
├── tests/                 # All tests (repository level)
├── docs/                  # Documentation
│   ├── ARCHITECTURE.md
│   ├── DEVELOPMENT.md
│   ├── INSTALLATION.md
│   └── Docker.md
├── Dockerfile            # Docker image definition
├── docker-compose.yml    # Docker services configuration
├── pyproject.toml        # Workspace configuration (Ruff, pytest)
└── uv.lock              # UV lock file
```

### Adding New Code
- **New package modules**: Add to respective package's `src/` directory
- **New tests**: Add to `tests/` directory at repository root
- **Documentation**: Update relevant README.md files and docs/
- **Configuration**: Modify pyproject.toml files as needed

## Development Workflow

### Making Changes
1. Create a feature branch from `main` or `develop`
2. Make changes following code style guidelines
3. Write or update tests for your changes
4. Run linter: `uv run ruff check .`
5. Run formatter: `uv run ruff format .`
6. Run tests: `uv run pytest tests/ -v`
7. Run type checker: `uv run mypy core/src web/src mcp-manager/src --strict` (warnings acceptable)
   - Note: CI also checks memo/src, but it's not in the workspace
8. Commit with descriptive messages
9. Create pull request targeting `main` or `develop`

### CI Pipeline
The CI workflow runs on all pull requests:
- Linting with Ruff (style, errors, imports, naming)
- Format checking with Ruff
- Type checking with MyPy (strict mode, failures are non-blocking)
- Tests with pytest and coverage reporting (workspace packages: core, web, mcp-manager)
- Security scanning with Trivy
- Docker image build (on main branch pushes)

### Dependencies
- Use UV for dependency management
- Add dependencies to appropriate package's `pyproject.toml`
- Run `uv sync` after adding dependencies
- Update `uv.lock` file (handled automatically by UV)

## Key Tools and Libraries

### Core Dependencies
See individual package `pyproject.toml` files for specific version requirements:
- **Pydantic** (>=2.12.5): Data validation and settings
- **Typer** (>=0.21.1): CLI framework
- **Rich** (>=14.2.0): Terminal UI and formatting
- **DuckDB** (>=1.4.3): Embedded SQL database

### Development Tools
See root `pyproject.toml` for version requirements:
- **UV**: Fast Python package manager
- **Ruff** (>=0.14.13): Fast Python linter and formatter
- **MyPy** (>=1.19.1): Static type checker
- **Pytest** (>=9.0.2): Testing framework

## Docker Environment

The project includes a complete Docker environment:
- Base image: Python 3.11-slim-bookworm
- CLI tools included: rg, fd, bat, eza, yq, duckdb
- ML/AI libraries: torch, transformers, sklearn, scipy
- Services: SurrealDB, MCP Manager

### Docker Best Practices
- Test changes in Docker environment when applicable
- Update Dockerfile if adding system dependencies
- Keep Docker image size optimized
- Document any new environment variables

## Documentation

Always update documentation when:
- Adding new features or tools
- Changing command-line interfaces
- Modifying configuration options
- Updating dependencies significantly

Key documentation files:
- `README.md` (root): Quick start and overview
- Package `README.md` files: Specific tool documentation
- `docs/DEVELOPMENT.md`: Development guide
- `docs/ARCHITECTURE.md`: Project structure and design

## Environment Variables

Prefix: `AIFT_`
```bash
AIFT_LOG_LEVEL=DEBUG      # Logging level (DEBUG, INFO, WARNING, ERROR)
PYTHONUNBUFFERED=1        # Python output buffering
CLAUDE_CODE_SANDBOX=true  # Sandbox mode flag
```

Config file location: `~/.config/aift/config.env`

## Common Commands Reference

### Quick Validation
```bash
# Validate everything before commit
uv run ruff check . && uv run ruff format . --check && uv run pytest tests/ -v
```

### Installation Verification
```bash
aift --help
aift version
aift test
```

### Package Testing
```bash
# Test specific package
uv run pytest tests/test_cli.py -v

# Test with specific marker
uv run pytest tests/ -v -m "not slow"
```

## Troubleshooting

### Common Issues
- **Import errors**: Run `uv sync --all-groups` to install all dependencies
- **Container issues**: Run `docker-compose down -v && docker-compose up -d --build`
- **Test failures**: Check logs with `AIFT_LOG_LEVEL=DEBUG pytest tests/ -v -s`
- **Type check errors**: MyPy strict mode failures are non-blocking in CI

### Debug Mode
```bash
# Enable debug logging
AIFT_LOG_LEVEL=DEBUG aift test

# Run tests with output
pytest tests/ -v -s
```

## Security Considerations

- Never commit secrets or credentials
- Use environment variables for sensitive configuration
- Security scanning runs in CI via Trivy
- Review dependency vulnerabilities regularly
- Follow secure coding practices for CLI tools

## Specialized Copilot Agents

This repository includes custom agents with domain-specific expertise. For complex tasks, leverage these specialized agents:

- **Python Expert** ([agents/python-expert.agent.md](agents/python-expert.agent.md)): Python 3.11+, UV workspace, type hints
- **CLI Specialist** ([agents/cli-specialist.agent.md](agents/cli-specialist.agent.md)): Typer, Rich, CLI best practices
- **Testing Specialist** ([agents/testing-specialist.agent.md](agents/testing-specialist.agent.md)): Pytest, coverage, mocking
- **Docker Specialist** ([agents/docker-specialist.agent.md](agents/docker-specialist.agent.md)): Dockerfile, docker-compose, optimization
- **Documentation Writer** ([agents/documentation-writer.agent.md](agents/documentation-writer.agent.md)): READMEs, API docs, guides
- **DuckDB Expert** ([agents/duckdb-expert.agent.md](agents/duckdb-expert.agent.md)): Schema design, queries, semantic search

See [AGENTS.md](AGENTS.md) for detailed agent documentation and usage guidelines.

## Best Practices

1. **Write tests first**: Use TDD approach when possible
2. **Keep changes focused**: Small, incremental changes are easier to review
3. **Document as you go**: Update docs with code changes
4. **Use type hints**: Makes code more maintainable
5. **Follow existing patterns**: Consistency matters
6. **Test in Docker**: Validate changes in containerized environment
7. **Check coverage**: Maintain or improve test coverage with new changes
8. **Descriptive commits**: Write clear, descriptive commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agentic-dev-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
