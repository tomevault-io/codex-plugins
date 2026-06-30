# Copilot Instructions

* Write code in English.
* Write comments in English.
* You must write comments to explain non-trivial parts of the code.
* All features or bug fixes must be tested by one or more specs (unit-tests).
* Public methods must include parameters, returns, and possible exceptions.
* All methods must have docstrings.
* All methods must have type hints.
* The names of variables, functions, classes, files and modules should be descriptive.
* Keep functions and methods focused on a single task; avoid large monolithic functions.
* Use environment variables or configuration files (e.g., `.env`) to manage sensitive data.
* No secret keys, passwords, or sensitive information should be committed to the repository.
* Use the python virtual environment (in `.venv`). Do not use global Python.
* When adding new dependencies, add them to the `pyproject.toml` file using Poetry.
* Whenever possible, use the Makefile to run commands. This ensures that the commands are run with the correct environment and settings.
* If you changed any code, run `make format`, `make lint` and `make test` before committing.
* Aim to keep test coverage as close to 100% as possible.
* We follow the [PEP8 Style Guide][https://peps.python.org/pep-0008/] for general coding.
* We follow the [Numpy Docstring Style Guide](https://numpydoc.readthedocs.io/en/latest/format.html) for code documentation.
* The commit message should be concise and follow the format: `<type>: <short description>`. For example: `feat: add new API endpoint for user authentication`.
* Use the appropriate commit type (feat, fix, perf, docs, test, chore) to indicate the nature of the change.
* Remember to update the documentation (e.g., README.md, docs/) whenever you add new features or change existing ones.
* Always make sure you're following the guidelines described in the CONTRIBUTING.md file.

---
> Source: [jonatasgrosman/findpapers](https://github.com/jonatasgrosman/findpapers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-29 -->
