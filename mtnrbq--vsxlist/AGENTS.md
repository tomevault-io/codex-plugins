# VSXList - AI Coding Agent Instructions

## Agent Personality & Approach
- **Peer Principal Engineer**: Communicate as an experienced peer, not obsequious or overly praising of straightforward interactions
- **Test-Driven Development**: Default to TDD approach when building tests, writing failing tests first to expose bugs and guide implementation
- **Structured Testing**: Prefer well-organized projects that enable comprehensive unit testing of services and functions using NUnit-based test projects
- **No Emojis**: NEVER use emojis in code, responses, or otherwise unless EXPRESSLY instructed to

## Project Overview
VSXList is a .NET 9.0 CLI tool that extracts VS Code extension lists from user profiles and outputs them as CSV files. The tool helps developers inventory, backup, and analyze their VS Code extension configurations across different profiles.

**Current Status**: Production-ready CLI tool with Core library architecture, comprehensive testing, and published on GitHub. Ready for enhanced features and community contributions.

## Architecture & Structure
- **Console Application**: Built on .NET 9.0 with modern C# features
- **Core Library**: Separate Core.csproj with business logic for reusability
- **VS Code Profile Reader**: Discovers and parses VS Code user data directories
- **Extension Parser**: Extracts extension metadata from profile configurations
- **CSV Writer**: Outputs formatted extension data for analysis/backup
- **Cross-platform**: Supports Windows, macOS, and Linux VS Code installations
- **Test Coverage**: 4 passing NUnit tests covering core functionality

## Development Workflows
```bash
# Build and run locally
dotnet build
dotnet run

# Run tests (currently 4 passing)
dotnet test

# Create release build
dotnet publish -c Release -r win-x64 --self-contained
dotnet publish -c Release -r linux-x64 --self-contained
dotnet publish -c Release -r osx-x64 --self-contained
```

## Technology Stack
- **.NET 9.0** - Latest LTS framework with modern C# features
- **System.CommandLine** - Command-line argument parsing
- **CsvHelper** - CSV file generation and formatting
- **NUnit** - Unit testing framework with 4 comprehensive tests
- **Cross-platform** - Windows, macOS, and Linux support

## Current Development State
**Completed Features:**
- ✅ Core library architecture with separation between CLI and business logic
- ✅ VS Code profile discovery across all supported platforms
- ✅ Extension metadata extraction with rich data parsing
- ✅ CSV export with multiple output formats (single file, per-profile, summary)
- ✅ Command-line interface with comprehensive options
- ✅ Error handling and validation with TDD approach
- ✅ Cross-platform compatibility testing
- ✅ Professional documentation and GitHub publication

**Next Development Priorities:**
1. Enhanced CSV output formats with additional metadata fields
2. Performance optimization for large extension collections
3. Plugin architecture for custom export formats
4. Integration testing with real VS Code profiles
5. Release automation and packaging improvements

## Git Conventions
- **Succinct commit messages**: Keep commit messages brief and clear (e.g., "Add extension icon support", "Fix profile parsing bug")
- **Succinct PR messages**: Title and description should be concise but complete
- Recent commits: Initial implementation → Core refactoring → Bug fixes → GitHub publication

## Agent Mode Workflow
When working in agent mode you MUST create a new branch for the work from main BEFORE starting on any change session, where a change session is a set of changes and interactions designed to effect a feature or similar. When it is indicated that we're done, e.g. 'lets button this up', or 'we're done' or similar, prepare a commit with a succinct message, commit the change to the working branch, then create a PR to main and wait for manual review/merging. Once the PR has been merged by a maintainer, offer to remove the old branch.

**CRITICAL**: The main branch is protected and requires Pull Requests for ALL changes. Direct pushes to main are NEVER allowed. Force pushes are prohibited. Always work on feature branches and create PRs for review and merging. Agents CANNOT merge their own PRs.

## Coding Conventions
- Use modern C# features (records, pattern matching, file-scoped namespaces)
- Async/await for file I/O operations
- Dependency injection for testability
- Command-line parsing with `System.CommandLine` library
- TDD methodology: Write failing tests first, implement to make them pass, then refactor
- Internal visibility for service methods to enable comprehensive unit testing

## Project Structure & Key Files
```
vsxlist.sln                    # Solution file with multiple projects
├── vsxlist.csproj             # Main console application
├── Program.cs                 # Entry point with System.CommandLine setup
├── Core/                      # Business logic library
│   ├── Core.csproj            # Reusable business logic project
│   ├── Constants/
│   │   └── VsCodePaths.cs     # Platform-specific VS Code path constants
│   ├── Models/
│   │   ├── VsCodeExtension.cs # Extension data model with 13+ properties
│   │   └── VsCodeProfile.cs   # Profile data model with extension collections
│   └── Services/
│       ├── ProfileReaderService.cs  # Discovers and reads VS Code profiles
│       └── CsvWriterService.cs      # Handles CSV output in various formats
├── Tests/                     # NUnit test project
│   ├── Tests.csproj           # Test project targeting .NET 9.0
│   └── UnitTest1.cs           # 4 comprehensive tests covering core functionality
├── Data/                      # Output directory (gitignored except anchor)
├── Directory.Build.props      # MSBuild properties shared across projects
└── README.md                  # Comprehensive user documentation
```

## Core Architecture
**VSXList Console App**:
- System.CommandLine integration for argument parsing
- Dependency injection setup for service classes
- Error handling and user-friendly output

**Core Library** (Reusable business logic):
- **ProfileReaderService**: Discovers VS Code installations, reads profile configurations
- **CsvWriterService**: Generates CSV files in multiple formats (single, per-profile, summary)
- **Models**: Strongly-typed data structures for extensions and profiles
- **Constants**: Cross-platform path resolution for VS Code user data

## Testing Strategy
- **4 comprehensive NUnit tests** covering core service functionality
- **TDD methodology** with failing tests first approach
- **Cross-platform validation** for path resolution and profile reading
- **Error condition testing** for missing profiles and invalid data
- All tests passing with good coverage of business logic

## Integration Points
- **VSXViz Integration**: Designed to output CSV files consumed by VSXViz Flutter app
- **VS Code Compatibility**: Reads standard VS Code profile and extension formats
- **Cross-platform support**: Windows (`%APPDATA%`), macOS (`~/Library`), Linux (`~/.config`)
- **CLI Integration**: Can be integrated into build scripts and automation workflows

## Production Deployment
- **Self-contained executables** for all major platforms
- **No runtime dependencies** when published with `--self-contained`
- **GitHub releases** ready for automated distribution
- **CSV output** designed for spreadsheet import and analysis tools

---

## Instructions for Updating This File
As an AI coding agent working on this project:

1. **Update this file** whenever you discover or implement significant architectural patterns
2. **Document workflows** that aren't obvious from file inspection (build commands, test patterns, deployment)
3. **Note conventions** that differ from standard practices in the chosen technology stack
4. **Reference specific files** that exemplify important patterns
5. **Keep it concise** - focus on what makes THIS project unique

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mtnrbq)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mtnrbq)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
