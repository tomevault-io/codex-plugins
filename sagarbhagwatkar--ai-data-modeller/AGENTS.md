<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# AI Data Modeller - Copilot Instructions

This is a Streamlit-based AI data modeling application. When working on this project:

## Project Context
- This app uses LLM (OpenAI GPT-4) to analyze uploaded data files and generate data models
- Primary goal is to detect relationships between tables and generate DDL statements
- Keep the code simple and avoid heavy/complex implementations
- Follow PEP8 standards and include type hints

## Key Components
- **Streamlit Frontend**: Simple, reactive UI with file upload and tabs
- **LLM Analysis**: Use OpenAI API for schema analysis and relationship detection
- **Data Processing**: Pandas for file parsing and data profiling
- **ERD Generation**: Graphviz/PyGraphviz for diagram creation
- **Modular Structure**: Separate concerns into utils.py, llm_analysis.py, modeling.py, visualization.py

## Coding Standards
- Use type hints: `List[str]`, `Dict[str, Any]`, etc.
- Add docstrings and inline comments for all functions
- Include error handling for file processing and API calls
- Use logging module for debugging
- Store API keys in environment variables via .env file
- Follow modular programming principles

## Data Modeling Focus
- Detect primary keys (unique, non-null columns)
- Identify foreign key relationships
- Handle column name variations (e.g., customer_id vs cust_id)
- Support multiple file formats: CSV, XLSX, JSON
- Generate normalized data models (1NF, 2NF, 3NF)
- Export SQL DDL statements and JSON metadata

## Edge Cases to Handle
- Different column names with same semantics
- Missing or unclear primary keys
- Circular foreign key references
- Data format inconsistencies
- Empty or sample-only files
- Many-to-many relationships
- Case sensitivity issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sagarbhagwatkar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
