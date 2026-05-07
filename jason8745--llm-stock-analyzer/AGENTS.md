# .github/copilot-instructions.md

## Project Context
This repository manages prompt templates, LLM integration logic, and best practices for a stock analysis API service. The project is built with FastAPI, LangChain, and related Python tools. The main goal is to provide a maintainable, modern, and robust API for LLM-powered stock analysis, not to serve as an application codebase.

## Project Structure (2025-06-30)
- `app/main.py` — FastAPI app entrypoint, config, logger
- `app/api/v1/endpoints.py` — API endpoints (LLM analysis, health)
- `app/configs/config.py` — Pydantic config, YAML loading
- `app/internal/analysis/indicators.py` — Technical indicator functions
- `app/internal/llm/chain.py` — LLM chain, prompt formatting, output parsing
- `app/internal/shioaji/stock_data.py` — Shioaji (TW market) data logic
- `app/internal/yfinance/stock_data.py` — YFinance (global market) data logic
- `app/services/analysis/stock_trend_pipeline.py` — Data pipeline, indicator enrichment
- `app/services/analysis/trend_analysis.py` — Trend signal generation
- `app/utils/logger.py` — Structured logging
- `app/assets/stock_analyzer_prompt.md` — Versioned, documented prompt template
- `tests/` — Pytest test cases for all modules and endpoints
- `.github/workflows/ci.yml` — CI workflow for lint, type check, and tests

## Used Technologies
- **Python**: 3.12+
- **FastAPI**: >=0.115.14
- **Pydantic**: v2+
- **LangChain**: >=0.3.27 (langchain-openai)
- **Pandas**: >=2.3.0
- **YFinance**: >=0.2.64
- **Shioaji**: >=1.2.6
- **Uvicorn**: >=0.34.3
- **python-dotenv**: >=1.1.1

## Prompt & Documentation Style Guidelines
- Write all prompt templates and documentation in clear, concise, and professional English (or Traditional Chinese if the context requires).
- Use Markdown for documentation, with proper headings, code blocks, and bullet points.
- For prompt templates, use explicit variable placeholders (e.g., `{{variable}}`) and provide context for each variable.
- Always include a short comment or docstring at the top of each prompt or template file describing its purpose and usage.
- For LLM prompt design, prefer explicit instructions, clear role definition, and step-by-step breakdowns.
- When documenting API endpoints, use OpenAPI/Swagger-compatible docstrings and include example requests/responses.

## Preferred Patterns
- Organize all API endpoints using FastAPI's `APIRouter` and group by logical domain (e.g., `/api/v1/analysis`).
- Use async/await for all I/O-bound operations and endpoint handlers.
- Use Pydantic v2 models for all request/response schemas.
- Separate business logic (e.g., stock analysis, LLM calls) into service modules under `app/services/`.
- Store configuration and environment logic in `app/configs/config.py`.
- Place all prompt templates and LLM-related logic in dedicated files or modules (e.g., `app/internal/llm/chain.py`).
- Write tests in the `tests/` directory, using pytest.
- Use a centralized logger (`app/utils/logger.py`) for all logging.
- Use GitHub Actions for CI (see `.github/workflows/ci.yml`).

## LLM Integration Rules
- All LLM calls (e.g., via LangChain) must be wrapped in service functions/modules, not directly in API endpoints.
- Prompt templates should be versioned and documented.
- Always validate and sanitize LLM inputs/outputs.
- For prompt chaining or multi-step reasoning, use LangChain's official patterns and document the flow.
- If using OpenAI or other providers, keep API keys and secrets in environment variables (never hardcode).

## Development Practices
- Follow PEP8 and Black formatting for all Python code.
- Use type hints throughout the codebase.
- Write docstrings for all public functions, classes, and modules.
- Use descriptive variable and function names (English preferred).
- Keep all dependencies up to date as per `pyproject.toml`.
- Document any non-obvious logic or business rules inline with comments.
- Use Git for version control and keep commit messages clear and descriptive.

## Things to Avoid
- Do not hardcode secrets, API keys, or credentials in any file.
- Do not place application code in this repository unless it is directly related to prompt or LLM management.
- Avoid mixing prompt templates with business logic—keep them modular and maintainable.
- Do not use deprecated FastAPI, Pydantic, or LangChain patterns—always follow the latest stable documentation.
- Do not use synchronous I/O in API endpoints or LLM calls.
- Avoid ambiguous variable names or undocumented code.
- Do not answer Copilot Chat queries without first stating you have read and understood this instructions file.

---

# Copilot Chat Behavior Rule
**Every time you respond to a user query in Copilot Chat:**
- Begin by stating: _“I've reviewed and understood the project's Copilot instructions. Based on your codebase’s style and declared dependencies, here’s the best solution...”_
- Do not answer any question until this declaration is made and all suggestions follow the documented style.

## Additional Rules
- Only use English for all code comments and docstrings. Do not use Chinese or other languages for comments in any code file.

---
> Source: [jason8745/llm-stock-analyzer](https://github.com/jason8745/llm-stock-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
