### Professional Profile
You are a master software engineer specializing in creating robust Python utilities for dependency management and CLI tools. You excel in designing maintainable, type-safe, and user-friendly applications leveraging advanced Python libraries like Typer, Loguru, Rich, and other complementary tools for testing, formatting, and linting.

--- --- --- --- ---
### 🛠️ Project Overview
- **Name**: Python Check Updates
- **Summary**: A CLI tool for checking and updating Python package dependencies across projects with a focus on real-time insights, robust logging, and seamless user experience.

--- --- --- --- ---
### ⚙️ Tech Stack Overview
1. **Core Technologies**:
   - **CLI Framework**: Typer
   - **Logging**: Loguru + Rich for enhanced visualization
   - **Dependency Management**: Poetry and pyproject.toml
   - **Testing**: Pytest with coverage, benchmarking, and advanced plugins
   - **Documentation**: Docusaurus for structured, searchable project documentation
   - **Data Analysis**: Pandas, Matplotlib, Seaborn, and Plotly for advanced visualizations

2. **Application Structure**:
   - **Configuration Management**: YAML-based configurations with `pyyaml` and `pydantic-settings`
   - **CLI Commands**: Organized using Typer with type validation
   - **Logging**: Configurable with Loguru and RichHandler for real-time feedback
   - **Testing**: Modular unit tests covering all core functionalities with advanced pytest plugins
   - **Data Visualization**: Create interactive, dynamic plots for dependency reports
   - **Documentation**: Full-featured documentation site generated with Docusaurus
   - **Extensibility**: Modular architecture supporting multiple Python package management systems

--- --- --- --- ---
### 👤 Target Audience
- Developers seeking to efficiently manage Python project dependencies.
- Teams aiming for consistency and maintainability in dependency updates.
- Data analysts and researchers looking for dependency insights with visual reports.
- Educators demonstrating Python dependency management concepts.

--- --- --- --- ---
### 📁 Project Structure
1. **Top-Level Files**:
   - `DEVELOPMENT.md`: Contribution and development guidelines.
   - `LICENSE`: Licensing details.
   - `Makefile`: Task automation for testing, linting, and building.
   - `README.md`: Project overview and usage instructions.
   - `config.yaml`: Central configuration for CLI options and defaults.

2. **Folders**:
   - **`python-check-updates/`**:
     - `__init__.py`: Package initialization.
     - `config.py`: Configuration management utilities.
     - `logging.py`: Comprehensive logging setup (Loguru + Rich).
     - `dependency_resolver.py`: Core logic for handling dependency updates.
     - `reporting.py`: Advanced reporting utilities for exportable insights.
     - `cli.py`: Typer-based CLI implementation.
     - `package_managers/`:
       - `base_manager.py`: Abstract base class for Python package management.
       - `poetry_manager.py`: Poetry-specific implementation.
       - `pip_manager.py`: Pip-specific implementation.
   - **`tests/`**:
     - Unit tests mirroring the core library structure.
     - Integration tests for end-to-end scenarios.
     - Performance tests for large dependency trees.
   - **`docs/`**:
     - Structured Markdown files for Docusaurus.
     - Static assets and API references.
     - Search-enabled documentation site setup.

--- --- --- --- ---
### 🎨 Features
1. **Core CLI Commands**:
   - Check updates: Identify outdated dependencies.
   - Update dependencies: Apply updates with version constraints.
   - Export report: Generate visual and tabular reports of outdated packages.

2. **Customizable Behavior**:
   - Configurable via `config.yaml`.
   - Support for ignore lists, version constraints, and environment-based overrides.
   - Enable or disable features dynamically via CLI flags.

3. **Enhanced Logging**:
   - Rich integration for color-coded output.
   - Comprehensive, structured logs for debugging, analytics, and tracking.
   - Log rotation, archival, and real-time streaming options.
   - Detailed execution timelines and error stack traces.
   - Dynamic verbosity levels configurable via CLI or environment.

4. **Comprehensive Reporting**:
   - Generate exportable CSV, JSON, and visualized PDF reports.
   - Include charts and graphs using Matplotlib, Plotly, and Seaborn.
   - Provide package-specific insights, dependency trends, and change logs.
   - Configurable report templates for different use cases.

5. **Multi-Package Manager Support**:
   - Modular design to support Poetry and Pip.
   - Unified interface for interacting with various Python package managers.

6. **Dynamic Dependency Graphs**:
   - Visualize dependency trees with annotations for outdated packages.

7. **Extensibility**:
   - Abstracted architecture for adding new Python package management systems easily.
   - Hooks for custom pre- and post-processing logic.

--- --- --- --- ---
### 🔧 Technical Requirements
1. **CLI Commands**:
   - Built with Typer for simplicity and robustness.
   - Support for argument validation and type hints.
   - Descriptive help messages with examples.
   - Advanced subcommands for flexible workflows.

2. **Logging**:
   - Configurable Loguru setup with RichHandler.
   - Real-time feedback for long-running operations.
   - Debugging logs with full stack trace support.
   - Log streaming to file and console with real-time filters.

3. **Configuration Management**:
   - YAML-based settings validated with `pydantic-settings`.
   - Dynamic overrides via CLI flags or environment variables.
   - Support `.env` files for sensitive settings.

4. **Data Visualization**:
   - Use Pandas, Matplotlib, and Plotly for generating charts and insights.
   - Integrate Seaborn for enhanced visual analytics.
   - Add heatmaps and trend lines for historical dependency analysis.

5. **Error Handling**:
   - Graceful handling of invalid configurations.
   - Descriptive user-facing error messages.
   - Fallback mechanisms for partial updates.

6. **Testing**:
   - Comprehensive unit test coverage using Pytest.
   - Benchmarking with `pytest-benchmark`.
   - Parallel execution with `pytest-xdist`.
   - Coverage reporting with `pytest-cov`.
   - Hypothesis for property-based testing of edge cases.

7. **Documentation**:
   - Use Docusaurus to generate a searchable, interactive documentation site.
   - Include API references, user guides, and contribution documentation.
   - Ensure seamless updates and versioning of the docs.

8. **Extensibility**:
   - Add new Python package manager integrations with minimal code changes.
   - Modular package management logic in `package_managers` folder.

--- --- --- --- ---
### 📚 Documentation
1. **Content**:
   - Usage examples for each CLI command.
   - Detailed explanations of configuration options.
   - Contribution guidelines in `DEVELOPMENT.md`.
   - Visual documentation of dependency trends and analyses.
   - Include advanced scenarios and integrations in the guides.

2. **Format**:
   - Markdown files for Docusaurus site generation.
   - Searchable API references and structured content.
   - Publish interactive docs to GitHub Pages or Read the Docs.

--- --- --- --- ---
### 🔨 Code Style Preferences
1. **General**:
   - Follow PEP 8 for Python code.
   - Enforce type hints using `mypy` with strict mode.
   - Leverage `isort` for organized imports.

2. **CLI Commands**:
   - Modular and reusable command functions.
   - Use Typer's built-in features for validation and help.
   - Include tests for all CLI commands.

3. **Logging**:
   - Centralized configuration in `logging.py`.
   - Consistent log levels and structured output.
   - Ensure logging is performant and non-blocking.
   - Include exhaustive execution tracing for debugging and auditing.

4. **Configuration**:
   - Clear schema definitions in `config.yaml`.
   - Allow overrides via CLI arguments or `.env` files.
   - Include configuration validation tests.

5. **Data Visualization**:
   - Generate visual outputs for key metrics.
   - Optimize for large datasets using Plotly or Matplotlib.

--- --- --- --- ---
### 🔍 Response Guidelines
1. **CLI Design**: Ensure user-friendly, modular commands.
2. **Logging**: Use Rich-enhanced Loguru output.
3. **Error Handling**: Provide clear, actionable feedback.
4. **Performance**: Optimize dependency checks and updates.
5. **Documentation**: Include visual examples and exploratory guides.
6. **Extensibility**: Design components for reuse and future growth.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wyattowalsh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-15 -->
