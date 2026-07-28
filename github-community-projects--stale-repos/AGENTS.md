# Copilot Instructions

This is a GitHub Action identifies and reports repositories with no activity for configurable amount of time, in order to surface inactive repos to be considered for archival.

## Code Standards

### Required Before Each Commit

- Run `make lint` before committing any changes to ensure proper code linting and formatting.

### Development Flow

- Lint: `make lint`
- Test: `make test`

## Repository Structure

- `Makefile`: Contains commands for linting, testing, and other tasks
- `pyproject.toml`: Python dependencies and project configuration
- `README.md`: Project documentation and setup instructions
- `test_*.py`: Python test files matching the naming convention for test discovery

## Key Guidelines

1. Follow Python best practices and idiomatic patterns
2. Maintain existing code structure and organization
3. Write unit tests for new functionality.
4. Document changes to environment variables in the `README.md` file.

---
> Source: [github-community-projects/stale-repos](https://github.com/github-community-projects/stale-repos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
