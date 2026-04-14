# GitHub Copilot Instructions for Py-AdventOfCode

## General Guidelines

- **Language**: Python 3.14+ only
- **Pure Python**: No external libraries that wrap other languages (C extensions, Cython, etc.)
- **No multiprocessing/multithreading**: Avoid `multiprocessing`, `threading`, or async solutions
- **Code Quality**: All code must pass pre-commit hooks without errors or warnings

## Code Standards

### Pre-commit Requirements
All code must pass these checks:
- `black` - Code formatting
- `ruff` - Linting (no errors, no warnings)
- `pyright` - Type checking
- No trailing whitespace
- Proper end of files
- No debug statements

### Python Style
- Use type hints for all functions
- Follow PEP 8 conventions
- Prefer readability over clever one-liners
- Use descriptive variable names
- Add docstrings for complex functions

### Allowed Libraries
- Standard library (collections, itertools, re, hashlib, etc.)
- `numpy` for numerical operations
- `beautifulsoup4` / `bs4` for HTML parsing
- `requests` for HTTP requests
- `markdownify` for markdown conversion

### Optimization Guidelines
- Prefer algorithmic optimizations over brute force
- Use appropriate data structures (sets, defaultdict, etc.)
- Avoid string concatenation in loops (use lists and join)
- Detect and optimize patterns (e.g., multiplication loops in assembly code)
- Calculate mathematically when simulation is too slow

### Testing
- Verify solutions work with provided examples
- Ensure code runs in reasonable time (<10 seconds per day when possible)
- Check that all type hints are correct

## Project Structure

- Each year has a `main.py` that runs all days and displays timing
- Each day has its own folder with `main.py`, `input`, and optional `example` files
- Solutions should use `part_one()` and `part_two()` functions
- Input reading should use `get_input()` function

## Common Patterns

### Input Reading
```python
def get_input(filename: str) -> list[str]:
    with open(filename, encoding="utf8") as f:
        return [line.strip() for line in f.readlines()]
```

### Pattern Detection
When simulating code (like assembly), detect common patterns:
- Multiplication loops
- Nested operations
- Factorials or mathematical sequences

### Numpy Usage
- Use numpy for numerical arrays and matrix operations
- Convert to native Python types when returning from functions (e.g., `int(np.count_nonzero(...))`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevingrillet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
