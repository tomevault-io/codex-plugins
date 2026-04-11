# Copilot Instructions for Trelliq Python

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Context
This is a Streamlit application for processing Trello board data and generating marketing reports.

## Key Technologies
- **Streamlit**: Modern web interface
- **Pandas**: Data processing and deduplication
- **Plotly**: Interactive charts and visualizations  
- **Python 3.8+**: Core language

## Project Structure
- `app.py`: Main Streamlit application
- `src/`: Source code modules
- `data/`: Sample data files
- `config/`: Configuration files

## Code Style Guidelines
- Use type hints for function parameters and returns
- Follow PEP 8 styling conventions
- Use docstrings for all functions and classes
- Prefer pandas operations over loops for data processing
- Use Streamlit caching (@st.cache_data) for expensive operations

## Key Features to Maintain
- Zero duplications in task reports using pandas groupby operations
- Support for 4 marketing groups with configurable members
- Automatic status detection based on Trello list names
- Export capabilities (Excel, JSON, CSV)
- Responsive design with modern UI components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/svlucas7)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/svlucas7)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
