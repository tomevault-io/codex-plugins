# Copilot Instructions

## Project Configuration

- **Target Framework**: .NET 10.0
- **SDK Version**: 10.0.0 (set in global.json)
- All new projects in this workspace should target .NET 10.0 by default
- Always use Global Package Versions (Directory.Packages.props) for all dependencies
- Always use Global Build Properties (Directory.Build.props) for all common build settings
- Always use `.editorconfig` for code style settings; if none exists create from `https://github.com/ardalis/CleanArchitecture/blob/main/.editorconfig`

## Model Design

Never have a model that can represent inconsistent states. For example:

```csharp
    public bool IsCompleted { get; set; }
    public DateTime? CompletedDate { get; set; }
```

In this case, if `CompletedDate` has a value then `IsCompleted` must be `true`. Really `IsCompleted` could just be a computed property:

```csharp
    public bool IsCompleted => CompletedDate.HasValue;
```

## Testing Conventions

- Use xUnit for all unit tests.
- Use NSubstitute for mocking dependencies.
- Follow the Arrange-Act-Assert pattern in all test methods (separate with newlines, no comments)
- Use Test Folders that mirror the source folder structure (plus ".Tests" suffix)
- Name Test Classes for the Class and Method being tested; do not suffix with Tests!
    - Example: `OrderService_CreateOrder.cs`
- Name Test Methods using the pattern: `DoesSomething_GivenSomeCondition`
    - Example: `CreateOrder_ReturnsOrder_GivenValidRequest`
    - Example: `CreateOrder_ThrowsException_GivenInvalidRequest`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ardalis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ardalis)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
