# SWIKIWI Copilot Instructions

## Project Overview

SWIKIWI is a CLI tool for multi-source knowledge search, currently in early development. The project aims to provide unified access to Wikipedia, Britannica, and other configurable knowledge sources through a .NET console application.

## Architecture & Key Components

### Planned Project Structure

```
SWIKIWI/
├── Program.cs              # CLI entry point with command parsing
├── Models/                 # Data structures for search results and configuration
│   ├── SearchResult.cs     # Standardized result format across sources
│   ├── SearchSource.cs     # Source configuration and metadata
│   └── Configuration.cs    # App settings and source configurations
├── Services/               # Core business logic
│   ├── WikipediaService.cs # Wikipedia API integration
│   ├── WebScrapingService.cs # Generic web scraping for sources without APIs
│   └── ConfigurationService.cs # Config file management
├── Commands/               # CLI command implementations
│   ├── SearchCommand.cs    # Main search functionality
│   └── ConfigCommand.cs    # Configuration management commands
└── config.json            # Runtime configuration file
```

### Development Guidelines

#### Key Patterns & Conventions

- **Multi-source Strategy**: Each knowledge source implements a common interface for search operations
- **Configuration-driven**: All sources are configurable via `config.json` with enable/disable flags
- **CLI Architecture**: Uses command pattern for `search`, `config show`, `config enable/disable` operations
- **Result Standardization**: All sources return data in a unified `SearchResult` format for consistent output

#### Command Structure

The CLI follows this pattern:

```bash
swikiwi <command> [arguments] [--options]
# Examples:
swikiwi search "Leonardo da Vinci" --source wikipedia --limit 5
swikiwi config enable britannica
```

#### Technology Stack

- **.NET 9.0** console application
- **HttpClient** for API calls and web scraping
- **System.Text.Json** for configuration and API response parsing
- **System.CommandLine** (recommended) for CLI argument parsing

#### Source Integration Patterns

1. **API-based sources** (Wikipedia): Use official REST APIs with proper rate limiting
2. **Web scraping sources** (Britannica): Implement CSS selector-based content extraction
3. **Custom sources**: Support both API and scraping approaches via configuration

#### Configuration Management

- Runtime configuration in `config.json` with hot-reload capability
- Each source has `enabled`, `url`, `language`, and `timeout` properties
- User-agent customization for web scraping compliance

#### Error Handling & Resilience

- Graceful degradation when sources are unavailable
- Timeout handling for slow sources
- Proper HTTP status code handling with retry logic

### Development Workflow

#### Build & Run

```bash
dotnet build
dotnet run -- search "query" --source wikipedia
```

#### Testing Strategy

- Unit tests for each service with mocked HTTP responses
- Integration tests for actual API calls (rate-limited)
- CLI command testing with various argument combinations

#### Dependencies to Add

When implementing features, consider these NuGet packages:

- `System.CommandLine` for robust CLI parsing
- `HtmlAgilityPack` for web scraping
- `Microsoft.Extensions.Logging` for structured logging
- `Microsoft.Extensions.Configuration` for enhanced config management

### Current State

⚠️ **Early Development**: The project currently contains only a basic Program.cs stub. All architecture described in README.md needs implementation.

### Implementation Priority

1. Basic CLI structure with `System.CommandLine`
2. Configuration system and `config.json` loading
3. Wikipedia API service as first source
4. Search result formatting and display
5. Additional sources and web scraping capabilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TerenceU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
