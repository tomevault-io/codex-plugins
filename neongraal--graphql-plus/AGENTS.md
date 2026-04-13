# Copilot Instructions for GraphQL+

## Project Overview

GraphQL+ is a successor to GraphQL, implementing a parser, merging verifier, and modeller in C# .NET. The project provides tools for parsing, verifying, and modeling GraphQL+ schemas and queries.

## Tech Stack

**Language & Framework:**

- C# with latest language features enabled
- Source projects target .NET Standard 2.0
- Test projects multi-target .NET 10.0, 9.0, and 8.0
- Nullable reference types enabled
- File-scoped namespaces preferred

**Testing Frameworks:**

- XUnit v3 for test framework
- AutoFixture with AutoData for test data generation
- NSubstitute for mocking
- Shouldly for assertions
- Verify for snapshot testing

**Other Key Dependencies:**

- YamlDotNet for YAML processing
- Fluid for templating
- System.Text.Json for JSON processing

**Build & Development Tools:**

- dotnet CLI for build, test, and restore
- Prettier for non-C# formatting (via `format.ps1`)
- dotnet format for C# code formatting
- PowerShell scripts for automation
- dprint for additional formatting

## Build, Test, and Validation

**Install Dependencies:**

```powershell
dotnet restore
```

**Build:**

```powershell
dotnet build
```

**Run All Tests:**

```powershell
./test.ps1
```

**Run Class Tests Only:**

```powershell
./test.ps1 -ClassTests
```

**Run Tests for Specific Framework:**

```powershell
./test.ps1 -Framework "10.0"
```

**Format Code:**

```powershell
./format.ps1
```

This runs:

- Prettier for markdown, JSON, YAML, and other non-C# files
- `dotnet format whitespace` for whitespace formatting
- `dotnet format style` for code style
- `dotnet format analyzers` for analyzer fixes

**Coverage:**

```powershell
./coverage.ps1
```

## Coding Standards and Conventions

**Read `Style-Guide.md` for repository-wide coding style and `Conventions.md` for test-specific conventions before making changes.**

### Code Style

- Use file-scoped namespaces (enforced via `.editorconfig`)
- Maximum line length: 150 characters
- Indent with 2 spaces (not tabs)
- Use `using` directives outside namespace
- Prefer expression-bodied members only when they stay simple and readable
- Prefer primary constructors where they match the type shape
- Prefer explicit variable types while still using target-typed `new()` where the declaration already makes the type obvious
- TreatWarningsAsErrors is enabled

### Testing Standards

**When writing tests use XUnit and AutoData.**

**Class tests** use NSubstitute and Shouldly:

- Use fields or properties for the instance being tested
- Store any constructor parameters of the instance being tested in private fields
- NSubstitute Returns should always use a local variable
- String and other primitive constants should be specified via test parameters
- Tests with parameters should have Theory and RepeatData attributes

**Component tests** use Shouldly or Verify.

**Renderer class tests:**

- Should inherit from RendererClassTestBase
- Use its Renderer property for the instance being tested
- Check the Structured output using `.ToLines(false)`

**Test naming convention:** Follow the `MethodUnderTest_StateUnderTest_ExpectedResult` pattern as documented in Conventions.md.

### Important Restrictions

- Avoid using `dynamic` keyword unless absolutely necessary
- All code must build without warnings (TreatWarningsAsErrors is enabled)
- Follow the existing test structure and patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NeonGraal)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/NeonGraal)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
