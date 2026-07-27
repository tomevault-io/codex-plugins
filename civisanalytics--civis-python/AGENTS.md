# GitHub Copilot Instructions

## Coding Style Guidelines

### Python
*   **Formatter:** Use the `black` formatter for all Python code under `src/` and `tests/`.
    Adhere strictly to `black`'s style, including a default line length of 88 characters.
*   **Linter:** Use `flake8` for linting.
    Ensure all code under `src/` and `tests/` passes `flake8` checks with the following configurations:
      - max-line-length = 88
      - extend-ignore = E203
*   **Best Practices:** Prioritize clean, readable, and compliant code suggestions that follow these rules automatically.

---
> Source: [civisanalytics/civis-python](https://github.com/civisanalytics/civis-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
