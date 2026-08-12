# GitHub Copilot Instructions

This file provides context to GitHub Copilot about the FastAPI Endpoint Change Detector project.

## Project Overview

This is a Python CLI tool that analyzes code changes and identifies which FastAPI endpoints are affected. It uses AST parsing, static analysis (mypy, ruff), and dependency graph construction.

## Technology Stack

- **Language**: Python 3.10+
- **CLI Framework**: Click
- **Data Validation**: Pydantic v2
- **Graph Operations**: NetworkX
- **Terminal Output**: Rich
- **Testing**: pytest
- **Type Checking**: mypy (strict mode)
- **Linting**: ruff

## Code Style Guidelines

### General
- Use type hints for all function signatures
- Prefer `pathlib.Path` over `os.path`
- Use f-strings for string formatting
- Maximum line length: 100 characters
- Use absolute imports within the package

### Naming Conventions
- Classes: `PascalCase`
- Functions/methods: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Private members: `_leading_underscore`

### Documentation
- All public functions must have docstrings (Google style)
- Complex logic should have inline comments
- Type hints are required, not optional

## Architecture Patterns

### Data Flow
```
CLI Input → Parser → Analyzer → Change Mapper → Output Formatter → CLI Output
```

### Key Design Decisions
1. **Immutable Data Models**: Use Pydantic models with `frozen=True` where possible
2. **Dependency Injection**: Pass dependencies explicitly, avoid global state
3. **Error Handling**: Use custom exception classes, never catch bare `Exception`
4. **Logging**: Use `structlog` for structured logging

## AST Parsing Patterns

When working with Python AST:

```python
import ast
from pathlib import Path

def parse_file(file_path: Path) -> ast.Module:
    """Parse a Python file into an AST."""
    source = file_path.read_text(encoding="utf-8")
    return ast.parse(source, filename=str(file_path))

class FunctionVisitor(ast.NodeVisitor):
    """Example visitor pattern for AST traversal."""
    
    def __init__(self) -> None:
        self.functions: list[ast.FunctionDef] = []
    
    def visit_FunctionDef(self, node: ast.FunctionDef) -> None:
        self.functions.append(node)
        self.generic_visit(node)
```

## FastAPI Detection Patterns

Detect FastAPI endpoints by looking for:

```python
# Direct decorators
@app.get("/path")
@app.post("/path")
@router.get("/path")

# With dependencies
@app.get("/path", dependencies=[Depends(verify_token)])

# APIRouter includes
app.include_router(router, prefix="/api")
```

## Dependency Graph Construction

Use NetworkX for the dependency graph:

```python
import networkx as nx

# Create directed graph
graph = nx.DiGraph()

# Add nodes with metadata
graph.add_node("module.function", 
               file="path/to/file.py", 
               line=42,
               type="function")

# Add edges representing dependencies
graph.add_edge("endpoint_handler", "service_function")
```

## Testing Conventions

### Unit Tests
```python
import pytest
from fastapi_endpoint_detector.parser import ast_parser

class TestAstParser:
    def test_parse_simple_function(self) -> None:
        """Test parsing a simple function definition."""
        source = "def hello(): pass"
        result = ast_parser.parse_source(source)
        assert len(result.functions) == 1
```

### Fixtures
Place test fixtures in `tests/fixtures/` and load via conftest.py:

```python
@pytest.fixture
def sample_fastapi_app(fixtures_path: Path) -> Path:
    return fixtures_path / "simple_app" / "main.py"
```

## Error Handling

Use custom exceptions:

```python
class EndpointDetectorError(Exception):
    """Base exception for all project errors."""
    pass

class ParseError(EndpointDetectorError):
    """Raised when file parsing fails."""
    def __init__(self, file_path: Path, reason: str) -> None:
        self.file_path = file_path
        super().__init__(f"Failed to parse {file_path}: {reason}")
```

## CLI Patterns

Use Click with proper typing:

```python
import click

@click.command()
@click.option("--app", "-a", type=click.Path(exists=True), required=True)
@click.option("--diff", "-d", type=click.Path(exists=True), required=True)
@click.option("--format", "-f", type=click.Choice(["text", "json", "yaml"]))
def main(app: str, diff: str, format: str | None) -> None:
    """Detect FastAPI endpoints affected by code changes."""
    pass
```

## Performance Considerations

- Cache AST parsing results for large projects
- Use lazy loading for optional integrations (mypy, ruff)
- Profile with `cProfile` for optimization targets
- Target: <30 seconds for 100 endpoints

## Common Tasks

### Adding a New Parser
1. Create module in `src/fastapi_endpoint_detector/parser/`
2. Define Pydantic models in `models/`
3. Add unit tests in `tests/unit/`
4. Export from `parser/__init__.py`

### Adding a New Output Format
1. Create formatter in `src/fastapi_endpoint_detector/output/`
2. Inherit from `BaseFormatter`
3. Register in CLI options
4. Add integration tests

## File Patterns to Know

- Entry point: `src/fastapi_endpoint_detector/cli.py`
- Main orchestration: `src/fastapi_endpoint_detector/cli.py::main()`
- Configuration: `src/fastapi_endpoint_detector/config.py`
- Models are in: `src/fastapi_endpoint_detector/models/`

---
> Source: [shaggitza/test_ast_fastapi](https://github.com/shaggitza/test_ast_fastapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
