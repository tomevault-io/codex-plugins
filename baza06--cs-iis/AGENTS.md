# Copilot Instructions for CS IIS

## Project Overview
This repository contains multiple C# console applications organized by topic and exercise. Each subfolder (e.g., `Sección1`, `Sección2`, `Prueba/Ejercicio3`, `Viernes/Ejercicio1`) is a standalone .NET project with its own `Program.cs` and `.csproj` file. The solution file `CS IIS.sln` ties these projects together for easy management.

## Architecture & Structure
- **Solution File**: `CS IIS.sln` is the entry point for managing all projects.
- **Projects**: Each folder under the root (and under `Prueba/` and `Viernes/`) is a separate C# project. Projects do not share code; each is self-contained.
- **Entry Point**: Every project uses a single `Program.cs` as its main entry point.
- **Build Output**: Compiled binaries are placed in `bin/Debug/net9.0/` within each project folder.

## Developer Workflows
- **Build & Run**: Use `dotnet run` from the project directory (e.g., `cd Sección2 && dotnet run`).
- **Debugging**: Standard .NET debugging tools apply; no custom launch configurations detected.
- **Testing**: No test projects or test files found; this repo is focused on small exercises, not automated tests.

## Conventions & Patterns
- **Naming**: Projects and files are named by topic or exercise (e.g., `Ejercicio3`, `Sección1`).
- **Code Style**: Follows basic C# conventions. Comments are used to explain logic and syntax, especially for beginners.
- **No Shared Libraries**: All code is local to each project; avoid cross-project dependencies.
- **Version**: Projects target .NET 9.0 (see output folders).

## Integration Points
- **External Dependencies**: No NuGet packages or external libraries detected; all code is standard C#.
- **No Web/API/Database**: Projects are pure console applications with no external integration.

## Examples
- To add a new exercise, create a new folder with a `.csproj` and `Program.cs`, then add it to the solution using `dotnet sln add <path-to-csproj>`.
- To run an exercise: `cd <ProjectFolder> && dotnet run`

## Key Files & Directories
- `CS IIS.sln`: Solution file
- `<ProjectFolder>/Program.cs`: Main code for each exercise
- `<ProjectFolder>/<ProjectName>.csproj`: Project configuration
- `<ProjectFolder>/bin/Debug/net9.0/`: Build output

---

**For AI agents:**
- Treat each project as independent; do not refactor across folders unless explicitly requested.
- When editing, preserve beginner-friendly comments and code style.
- If adding new projects, follow the existing folder and file naming conventions.
- If you find a README or instructions file in a new project, merge its content here.

---
> Source: [BaZa06/CS-IIS](https://github.com/BaZa06/CS-IIS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-21 -->
