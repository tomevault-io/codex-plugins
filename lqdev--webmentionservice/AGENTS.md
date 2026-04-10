# WebmentionService - Copilot Instructions

## Repository Overview

**WebmentionService** is an Azure Functions-based service written in **F#** that processes [Webmentions](https://www.w3.org/TR/webmention/) - a web standard for decentralized notifications between websites. The service validates incoming webmention requests, stores them in Azure Table Storage, and generates RSS feeds from collected mentions.

### Project Characteristics
- **Language**: F# (functional programming)
- **Framework**: Azure Functions v4, .NET 10.0 LTS (isolated worker)
- **Platform**: Azure Flex Consumption (Linux)
- **Size**: Small codebase (~600 lines total, 6 F# source files)
- **Type**: Serverless microservice / Azure Functions application
- **No Tests**: This repository does not have a test suite
- **Deployment**: `lqdevwebmentions-flex` at `https://webmentions.lqdev.tech/api/inbox`

### Key Dependencies
- `lqdev.WebmentionFs` (v0.0.7) - Core webmention validation library
- Azure Functions Extensions for HTTP, Storage, Blobs, and Tables
- Azure.Data.Tables for Table Storage operations

## Architecture

### Main Components

1. **ReceiveWebmention Function** (`ReceiveWebmention.fs`)
   - HTTP POST endpoint at `/inbox`
   - Validates and stores incoming webmentions
   - Uses Azure Table Storage with source/target URLs as keys

2. **WebmentionToRss Function** (`WebmentionToRss.fs`)
   - Timer-triggered (daily at 3 AM UTC: `0 0 3 * * *`)
   - Generates RSS feeds from stored webmentions
   - Outputs to Azure Blob Storage

3. **Domain Models** (`Domain.fs`)
   - `WebmentionEntity`: Table entity for Azure Storage

4. **Services**
   - `RssService.fs`: RSS feed generation logic
   - `Program.fs`: Entry point and dependency injection configuration

### File Structure
```
/
├── Domain.fs                  # Data models
├── RssService.fs             # RSS generation service
├── Startup.fs                # DI configuration
├── ReceiveWebmention.fs      # HTTP endpoint function
├── WebmentionToRss.fs        # Timer-triggered RSS function
├── WebmentionService.fsproj  # F# project file
├── host.json                 # Azure Functions host config
├── .vscode/                  # VS Code tasks and settings
└── .devcontainer.json        # Dev container configuration
```

## Build & Development Instructions

### Prerequisites

**CRITICAL**: This project requires **.NET 10.0 runtime** to build successfully.

**Installation Command** (if .NET 10.0 is missing):
```bash
wget https://dot.net/v1/dotnet-install.sh -O /tmp/dotnet-install.sh
chmod +x /tmp/dotnet-install.sh
/tmp/dotnet-install.sh --channel 10.0
```

Verify installation:
```bash
dotnet --list-sdks | grep "10.0"
```

### Build Commands

**IMPORTANT**: Always run commands in this exact order:

1. **Restore dependencies** (always run first):
   ```bash
   dotnet restore
   ```
   - Takes ~6-10 seconds on first run
   - Must complete before building

2. **Build** (Debug):
   ```bash
   dotnet build
   ```
   - Takes ~2-8 seconds after restore
   - Output: `bin/Debug/net10.0/WebmentionService.dll`
   - **Will fail** if .NET 10.0 runtime is not installed

3. **Clean**:
   ```bash
   dotnet clean
   ```
   - Removes `bin/` and `obj/` directories

4. **Publish** (Release):
   ```bash
   dotnet publish --configuration Release
   ```
   - Output: `bin/Release/net6.0/publish/`
   - Used for Azure deployment

### VS Code Tasks

The repository includes pre-configured VS Code tasks in `.vscode/tasks.json`:
- `clean (functions)`
- `build (functions)` (default, includes clean)
- `publish (functions)` (includes clean release)

### Local Development Setup

To run locally, create `local.settings.json` (not checked in):
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "PERSONAL_WEBSITE_HOSTNAMES": "localhost:3000,127.0.0.1:3000"
  }
}
```

**Note**: Azure Functions Core Tools (`func`) is optional for local testing but not required for builds.

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `AzureWebJobsStorage` | Yes | Azure Storage connection string |
| `PERSONAL_WEBSITE_HOSTNAMES` | Yes | Comma-separated hostnames to accept webmentions for |
| `FUNCTIONS_WORKER_RUNTIME` | Yes | Must be `dotnet-isolated` |

Configured in `Startup.fs` during dependency injection.

## Testing & Validation

**No automated tests exist** in this repository. Validation must be done manually:

1. Build successfully: `dotnet build`
2. Publish successfully: `dotnet publish --configuration Release`
3. Review changes to ensure:
   - No syntax errors in F# code
   - Function attributes remain intact
   - Azure bindings are not modified incorrectly

## Common Issues & Workarounds

### Issue 1: Build Fails with .NET 6.0 Not Found

**Error**:
```
error : Framework: 'Microsoft.NETCore.App', version '6.0.0' (x64) not found
```

**Solution**: Install .NET 6.0 runtime using the command in Prerequisites section above.

### Issue 2: local.settings.json Missing

The `local.settings.json` file is git-ignored. Create it manually using the template in the "Local Development Setup" section.

### Issue 3: F# Compilation Order

F# requires files to be compiled in dependency order. The `.fsproj` file lists files in the correct order:
1. Domain.fs
2. RssService.fs
3. Startup.fs
4. ReceiveWebmention.fs
5. WebmentionToRss.fs

**Never reorder files** in the project file unless you understand F# dependency requirements.

## Coding Guidelines

### F# Specific
- This codebase uses functional programming patterns
- Pattern matching is preferred over if-else
- Use pipelines (`|>`) for data transformations
- Functions are typically immutable

### Azure Functions Conventions
- Function names use `[<FunctionName("Name")>]` attribute
- HTTP triggers use `[<HttpTrigger(...)>]`
- Timer triggers use `[<TimerTrigger("cron")>]`
- Storage bindings use `[<Table(...)>]` and `[<Blob(...)>]`

### Key Code Patterns

**Dependency Injection** (Startup.fs):
- Services registered in `Configure` method
- Uses `AddScoped<T>` for services

**Table Storage Keys**:
- PartitionKey = URL-encoded target URL
- RowKey = URL-encoded source URL

**Error Handling**:
- Returns `OkObjectResult` for success (200)
- Returns `BadRequestObjectResult` for errors (400)

## Deployment

**Primary deployment method**: Azure Functions Core Tools
```bash
func azure functionapp publish <your-function-app-name>
```

**Pre-deployment**: The VS Code setting `azureFunctions.preDeployTask` is configured to run `publish (functions)` automatically.

## CI/CD

**No GitHub Actions workflows** are currently configured in this repository. All validation must be done manually before committing changes.

## Quick Reference

**Most common workflow**:
1. Ensure .NET 6.0 is installed
2. `dotnet restore` (if dependencies changed)
3. `dotnet build` (verify changes compile)
4. Make code changes
5. `dotnet build` (verify again)
6. `dotnet publish --configuration Release` (before deployment)

**Critical reminders**:
- Always install .NET 6.0 runtime before building
- The project targets .NET 6.0 specifically (not 8.0 or later)
- No tests exist - validate manually
- F# file order in .fsproj matters
- local.settings.json is git-ignored - create it manually

## Additional Resources

- README.md - Detailed usage documentation and API examples
- .vscode/tasks.json - Build task definitions
- host.json - Azure Functions runtime configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lqdev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lqdev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
