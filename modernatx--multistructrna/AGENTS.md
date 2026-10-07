# MultiStructRNA Copilot Instructions

Here are some strict guidelines, unless told otherwise:

Always write code in python.

Functions should have Typer style type hints for both inputs AND outputs.
List type arguments in Typer should be just strings that are separated with commas with no spaces in there.
When requested, split those arguments using string parsing.

Docstrings are Google-style. Any function that gets written must have a docstring.
Functions should have a function definition on the first line of the function, followed by a line break, followed by
parameters as defined on each line with :param x: definition of x. Argument types should not be included.
Each function should have :raises Error: explanation for the errors/exceptions and have a :returns: what the function returns.


This is an example of how functions should always be written, even functions only for testing:

```python
def hello_world(world: str) -> str:
    """Hello command.

    :param world: A string.
    :returns: A string.
    """
    return f"Hello, {world}!"
```

Some general python guidelines and best practices:

- Tests should always be written with pytest
- Pathlib paths should be used when possible instead of string paths
- loguru's logger should be used instead of print when possible
- Semantic line breaks should be in markdown files, which should be formatted according to the Markdownlint VSCode extension
- Use f-strings instead of .format() or str() for strings when possible
- Curly quotation marks (") are not used, instead use straight quotation marks (")
- Block quotes (indicated with >) are used when appropriate in markdown files
- Use list comprehension instead of for loops when possible
- When using pandas dataframes, use pyjanitor method chaining when possible
- Avoid try-except blocks when possible as they hide errors and make debugging harder

### CLI Structure

- CLI commands follow a consistent pattern using Typer apps and subcommands
- Main CLI entry point is in `multistructrna/cli.py`
- Typer CLI command names will always be kebab-case (e.g. `create-project`, not `create_project`)
- Always provide help text for options and commands

### Error Handling

- Use typer.Exit() with appropriate exit codes for CLI error cases
- Use loguru.logger for error messages with appropriate log levels
- Implement helpful error messages that suggest how to fix the problem

### Documentation

- Documentation is in `docs/` using MkDocs
- Each module should have corresponding API documentation
- Code blocks in docs should use triple backticks with the language specified
- Include a table of contents in documentation files

### Package Structure

```
multistructrna/
├── __init__.py          # Package exports
├── rna.py               # Main RNA class
├── engine.py            # Algorithm implementations
├── orchestration.py     # Mid-level API
├── utils.py             # Utility functions
├── plot.py              # Visualization
├── cli.py               # Command-line interface
├── setup_dependencies.py # Dependency installer
└── precompiled_binaries/ # External binaries
```

---
> Source: [modernatx/MultiStructRNA](https://github.com/modernatx/MultiStructRNA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-06 -->
