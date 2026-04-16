# Project Overview

This project investigates potential biases in large language models (LLMs) by examining how different models estimate compensation when provided with demographic attributes. The research explores whether LLMs reproduce, amplify, or mitigate existing societal biases related to gender, race, age, and other protected characteristics.

## Research Approach

The study employs two distinct methodologies:
1. **Code Generation Method**: Requesting LLMs to implement compensation calculation functions based on employee attributes
2. **Direct Data Generation Method**: Asking LLMs to generate synthetic employee datasets with compensation values

Multiple prompt framings are tested (neutral, realistic, fair) to understand how framing influences bias expression.

## Goals

- **Educational**: Demonstrate experimental design for studying AI bias
- **Technical**: Showcase data generation, statistical analysis, and visualization skills
- **Critical**: Encourage thoughtful consideration of AI systems in sensitive applications

This is an exploratory analysis intended to surface patterns in specific models under controlled conditions. Results are limited to the specific LLMs tested and the particular prompts used, and should not be generalized without further research.

## Folder Structure

- `/src`: Contains the source code.
  - `/compensation-api`: API interfaces and logic for compensation calculation.
  - `/llm_connection`: LLM connector implementations.
  - `/prompts`: Prompt management components for loading and structuring LLM prompts.
  - `/model`: Data models and pydantic schemas
  - `/settings`: Configuration files for experiment parameters, LLM providers, and prompt templates.
  - `/auto-generated`: Auto-generated code from LLMs.
- `/notebooks`: Jupyter notebooks for exploratory data analysis and visualization.
- `/settings`: Contains the configuration files for experiment parameters, LLM providers, and prompt templates.

## Libraries and Frameworks

- Python 3.12+ for the backend
- pydantic for data models and validation
- pydantic-settings to define and handle configuration files
- pandas for data analysis and manipulation
- numpy for numerical computations
- ipykernel for Jupyter notebook support
- Ruff for linting and code quality

## Coding Standards

- Aviod using emojis.
- Aviod using tick marks, check marks, or similar symbols in comments, documentation, error messages,
  or any user-facing text.
- Use double quotes for strings.
- Use modern type hints (Python 3.12+). Avoid `Any` unless necessary.
- Keep documentation and comments updated with the code and concise.
- Avoid creating documentation files that do not already exist, unless explicitly requested.
- Avoid creating tests unless explicitly requested.

## LLM Behaviour Instructions

- Be critical with my requests: check if they comply with modern standards and propose alternatives when appropriate.
- Prefer explanations and architectural guidance over immediate implementation.
- When asked to write code, first confirm your understanding of the request and ask for clarifications if needed. Wait for confirmation before implementing complex changes.
- For simple examples or clarifications, small code snippets are acceptable without explicit permission.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GabrieleDiCorato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
