# Copilot Instructions for PowerToys-Run-WebSearchShortcut

This repository contains a **Windows App SDK** project that implements a **Command Palette Extension** (likely for PowerToys Run or similar hosts). It runs as a **COM Server**.

## Project Architecture

- **Core Framework**: .NET 9.0, Windows App SDK, `Microsoft.CommandPalette.Extensions`.
- **Entry Point**: `WebSearchShortcut/Program.cs` initializes the COM server (`Shmuelie.WinRTServer`) and registers the `WebSearchShortcut` extension.
- **Extension Logic**:
  - `WebSearchShortcut.cs`: Implements `IExtension`. Serves as the main bridge between the host and the plugin.
  - `WebSearchShortcutCommandsProvider.cs`: Implements `CommandProvider`. Manages the list of available commands (shortcuts) displayed in the palette.
- **Data & Persistence**:
  - `Storage.cs`: Handles loading/saving shortcut configurations (JSON). Contains default providers (Google, Bing, etc.).
  - `WebSearchShortcutDataEntry.cs`: Model for a single shortcut entry.
- **Search Suggestions**:
  - `SuggestionsProviders/`: Contains implementations of `ISuggestionsProvider` for different services (Google, Bing, YouTube, etc.).
  - **Pattern**: Each provider implements `GetSuggestionsAsync` to fetch autocomplete results via HTTP.
- **Browser Integration**: `Browser/` folder handles detecting and launching the default browser.

## Key Patterns & Conventions

- **COM Server Lifetime**: The application runs as a local COM server. `Program.cs` manages the lifetime using `ManualResetEvent`.
- **Toolkit Usage**: Heavily relies on `Microsoft.CommandPalette.Extensions.Toolkit` for implementing interfaces like `ICommandItem` and `CommandProvider`.
- **Async/Await**: All I/O (network requests, file access) must be asynchronous.
- **JSON Handling**: Use `System.Text.Json` for parsing API responses and storage files.
- **Error Handling**: Suggestion providers should catch exceptions and log them via `ExtensionHost.LogMessage` (or return empty results) to avoid crashing the host.

## Development Workflows

- **Build**: Standard `dotnet build`.
- **Debugging**:
  - The app expects to be launched with `-RegisterProcessAsComServer` by the host.
  - To debug, you may need to attach to the process or configure the host to launch your debug build.
- **Adding a New Search Provider**:
  1. Create a new class in `SuggestionsProviders/` implementing `ISuggestionsProvider`.
  2. Implement `GetSuggestionsAsync` to parse the specific API response.
  3. (Optional) Add a default entry in `Storage.cs`.

## File Structure Highlights

- `WebSearchShortcut/`: Main application project.
- `WebSearchShortcut.Tests/`: Unit tests (MSTest).
- `WebSearchShortcut/SuggestionsProviders/`: Search engine implementations.
- `WebSearchShortcut/Commands/`: Command implementations (e.g., `SearchWebCommand`).

---
> Source: [Daydreamer-riri/CmdPal-WebSearchShortcut](https://github.com/Daydreamer-riri/CmdPal-WebSearchShortcut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-29 -->
