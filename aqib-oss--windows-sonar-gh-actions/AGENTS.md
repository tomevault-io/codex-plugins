# Copilot Instructions

## Project Overview

This is a demonstration project showing how to run SonarQube/SonarCloud analysis on Windows runners in GitHub Actions. The project contains a simple Python calculator library as sample code for analysis.

**Key Problem Solved**: Official SonarCloud GitHub Actions only work on Ubuntu (`ubuntu-latest`), but this project demonstrates using SonarScanner CLI directly on Windows runners (`windows-latest`).

## Build, Test, and Lint Commands

### Environment Setup
```bash
# Create and activate conda environment
conda env create --file environment.yml
conda activate conda_sonaronwindows

# Install dependencies
poetry install
```

### Testing
```bash
# Run all tests with coverage
poetry run pytest

# Run specific test file
poetry run pytest tests/sonaronwindows/test_calculator.py

# Run single test function
poetry run pytest tests/sonaronwindows/test_calculator.py::test_sum
```

### Code Quality
```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Individual tools
poetry run black .                 # Format code
poetry run isort .                 # Sort imports
poetry run flake8                  # Linting
poetry run pylint src/             # Additional linting
```

### SonarQube Analysis
The project uses SonarScanner CLI for Windows:
- Scanner is cached in GitHub Actions at `C:\SonarScannerCLI`
- Configuration in `sonar-project.properties`
- Coverage reports generated at `coverage-reports/coverage.xml`

## Architecture

### Directory Structure
- `src/sonaronwindows/` - Main package code
- `tests/sonaronwindows/` - Test files (mirrors src structure)
- `coverage-reports/` - Generated coverage XML for SonarQube
- `.github/workflows/` - CI/CD workflows for Windows-based SonarQube scanning

### Key Components
- **Calculator class**: Simple demonstration class in `src/sonaronwindows/calculator.py`
- **GitHub Actions**: Two workflows demonstrating Windows + SonarQube integration
  - `main-workflow.yaml` - Full CI/CD with semantic versioning
  - `pr-workflow.yaml` - PR validation workflow

### SonarQube Integration
- Uses SonarScanner CLI 6.1.0.4477 for Windows x64
- Configured for SonarCloud.io with organization `aqib-oss`
- Coverage integration via pytest-cov generating XML reports
- Windows-specific path handling in sonar-project.properties (`tests\\**` exclusions)

## Key Conventions

### Testing
- Test files mirror source structure under `tests/`
- Use Arrange-Act-Assert pattern in test methods
- Type hints required in both source and test code
- Coverage minimum threshold: 80%

### Code Style
- Black formatter with 90-character line length
- isort with black profile for import sorting
- Type annotations required for function parameters and returns
- Pre-commit hooks enforce all formatting and linting rules

### Commits & Releases
- Follow Conventional Commits specification
- Semantic versioning managed by python-semantic-release
- Version stored in `pyproject.toml:tool.poetry.version`
- CHANGELOG.md automatically generated
- Use prefixes: `fix:`, `feat:`, `perf:`, `build:`, `ci:`, `refactor:`, `docs:`, `test:`, `chore:`, `style:`
- Breaking changes: Add "BREAKING CHANGE" in commit footer

### Branch Management
- Always create feature branches for changes (never work directly on main)
- Use descriptive branch names with prefixes like `feat/`, `fix/`, `docs/`
- Example: `git checkout -b feat/add-new-feature`

### Windows-Specific Considerations
- PowerShell used exclusively in GitHub Actions
- Conda environment management for Python dependencies
- Path separators in sonar-project.properties use backslashes
- SonarScanner CLI cached to avoid repeated downloads
- Poetry installed via official installer script

---
> Source: [aqib-oss/windows-sonar-gh-actions](https://github.com/aqib-oss/windows-sonar-gh-actions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-11 -->
