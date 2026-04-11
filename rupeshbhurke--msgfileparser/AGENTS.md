# Copilot Instructions for MsgFileParser

## Project Overview
- **Purpose:** Parse Microsoft Outlook MSG files and export their contents to text or HTML using a .NET 8.0 console app.
- **Architecture:** Modular, with clear separation:
  - `Program.cs`: CLI entry, argument parsing, error handling, orchestration
  - `ParserArguments.cs`: Flexible CLI argument parsing
  - `FileValidator.cs`: Input/output validation
  - `MsgFileParser.cs`: MSG file parsing logic
  - `MsgExportService.cs`: Export logic (text/HTML)
  - `MsgFileInfo.cs`: Data class for parsed MSG info
  - `ExitCode.cs`: Centralized exit code enum

## Key Patterns & Conventions
- **Error Handling:**
  - All exit codes use the `ExitCode` enum (see `ExitCode.cs`).
  - Output messages follow strict format: `ERROR: <CODE> - <desc>`, `SUCCESS: ...`, `WARNING: ...`, `INFO: ...`.
  - Always use `Environment.Exit((int)ExitCode.<Name>)` for process termination.
- **Argument Parsing:**
  - Use `ParserArguments` for all CLI parsing; supports flexible flag positions and output path logic.
- **Validation:**
  - Use `FileValidator` for input/output checks; do not duplicate validation logic in other files.
- **Export Logic:**
  - Use `MsgExportService` for all file writing; supports both text and HTML export.
- **Data Flow:**
  - `Program.cs` → `ParserArguments` → `FileValidator` → `MsgFileParser` → `MsgExportService`

## Developer Workflows
- **Build:**
  - `dotnet build` (from project root)
- **Run:**
  - `dotnet run "input.msg" [output_path] [--text|--html]`
- **Test:**
  - `./test_msgparser.sh` (validates all error/success scenarios)
- **Debug:**
  - Use CLI arguments and check structured output for error diagnosis.
- **Linux Dependency:**
  - Install `libgdiplus` for graphics support: `sudo apt install libgdiplus libc6-dev`

## Integration Points
- **External:**
  - Uses NuGet packages: `MSGReader`, `System.Text.Encoding.CodePages`
- **Automation:**
  - Test script returns exit codes for CI/CD integration

## Examples
- **Success:**
  - `SUCCESS: Processing completed - Output: /path/to/output.txt`
- **Error:**
  - `ERROR: FILE_NOT_FOUND - missing.msg`
  - Exit code: 3

## File References
- See `README.md` for usage, error codes, and integration details
- See `ExitCode.cs` for all exit codes
- See `test_msgparser.sh` for test scenarios

## Contribution
- Follow modular structure and error handling conventions
- Update `ExitCode.cs` and documentation for new error codes
- Run `./test_msgparser.sh` before submitting changes

---
For more, see the full README.md and source files for implementation details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rupeshbhurke)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rupeshbhurke)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
