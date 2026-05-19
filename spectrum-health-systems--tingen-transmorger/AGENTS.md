﻿<!-- Last updated 260422 -->

# Copilot Instructions (Repository Onboarding)

This file tells an automated coding agent how to work safely and quickly in this repository. Trust these
instructions first; perform searches only when the instructions are incomplete or when a command fails.

## Quick facts

- Purpose: This repository contains one or more projects targeting .NET 10.
- Languages: C# (primary), XAML for WPF UI.
- Project type: WPF desktop app (SDK-style projects). C# language level: 14.0.
- Repo size / scope: small-to-medium codebase focused on a desktop UI; most work will be inside `src` or the
  repository root projects.

## Important coding/style rules

- Follow existing C# and XAML styles used in the repo (indentation, naming). When in doubt, mimic nearby files.
- Prefer `var` for local variable declarations when the initialized type is apparent or redundantly specified.
- When there is a block of `something = somethingElse;` statements, align the equals signs vertically.
- Code should be human-reaable and maintainable; prioritize clarity over cleverness or terseness.
- Use descriptive names for variables, methods, and classes. Avoid abbreviations unless they are widely understood.

## XML documentation

- Ensure all types and members have XML documentation comments.
- If XML documentation exists, do not replace it; Only create XML documentation where it is missing.
- Do not create XML documentation for event handlers.
- Use proper grammar, punctuation, and capitalization in XML doc comments.
- Use proper XML formatting and escape special characters.
- XML tags should be properly nested and well-formed.
- XML documentation should not be indented.
- Use valid HTML tags for formatting XML documentation.
- Prefer `<br/>` over `<para></para>` when possible.
- Use the following XML doc tags where appropriate:
  - `<c>code</c>` for inline code references and `<code>` for multi-line code blocks.
  - `<example>` for usage examples
  - `<exception>` to document thrown exceptions
  - `<list>` for bulleted or numbered lists
  - `<paramref>` to reference parameters in descriptions
  - `<see cref="..."/>` for inline cross-references
  - `<seealso cref="..."/>` for related topics
  - `<typeparam>` and `<typeparamref>` for generic type parameters
  - `<value>` for property value descriptions
- XML tags should be in the correct order:
  - `<summary>`
  - `<remarks>`
  - `<example>`
  - `<param>`
  - `<typeparam>`
  - `<exception>`
  - `<value>`
  - `<returns>`
  - `<see>`
  - `<seealso>`

### <summary> guidelines

- Every type and member must have a `<summary>` tag.
- The `<summary>` should clearly and concisely describe the purpose and behavior of the type or member.
- Keep `<summary>` tags on a single line.
- Keep `<summary>` tags under 120 characters.
- Example `<summary>` format:

```xml
/// <summary>Represents a customer in the system.</summary>
```

### <remarks> guidelines

- Use `<remarks>` for all methods, and where additional explanation is needed.
- `<remarks>` should provide useful context or details beyond the summary
- If a `<remarks>` section is 120 characters or less, keep it on a single line; otherwise, use multi-line format.
- `<remarks>` should use `<list>`, `<para>`, and other XMLdoc tags as needed.
- Example of single-line `<remarks>`:

```xml
/// <remarks>This method initializes the customer object with default values.</remarks>
```
- Example of multi-line `<remarks>`:

```xml
/// <remarks>
/// This method performs the following steps:
/// <list type="bullet">
/// <item>Initializes the components.</item>
/// <item>Sets up event handlers.</item>
/// <item>Starts the main processing loop.</item>
/// </list>
/// </remarks>
```

### Other tag guidelines

  - Use `<param>` tags for all parameters.
  - Use `<returns>` tags for methods returning values.

### Example of well-formed XML documentation

```xml
/// <summary>Calculates the total price of items in the cart.</summary>
/// <remarks>
/// This method sums up the prices of all items in the shopping cart, applying any discounts or taxes as necessary.
/// <list type="bullet">
/// <item>Iterates through each item in the cart.</item>
/// <item>Adds the item's price to the total.</item>
/// <item>Applies discounts if applicable.</item>
/// </list>
/// </remarks>
/// <param name="items">The list of items in the cart.</param>
/// <returns>The total price as a decimal value.</returns>
```

## Build & validation (what to run locally)

Prerequisites
- Install the .NET 10 SDK. Verify with: `dotnet --version` (should report a 10.x SDK).

Bootstrap (one-time)
1. From the repository root run: `dotnet restore` (restores NuGet packages for all projects).

Build
1. Run: `dotnet build --configuration Release` from the repository root. This performs restore if needed.
2. Fix compile errors until `dotnet build` succeeds.

Run the app (UI)
1. Identify the startup project (open the solution in Visual Studio or locate the project containing `MainWindow.xaml`).
2. From the command line you can run: `dotnet run --project <path-to-startup-csproj>` to launch the application.

Tests
- No automated test project was detected. If tests are added later, run them with `dotnet test` from the repo root.

Lint / format
- No repository-wide linter configuration detected. If preferred, run `dotnet format` to normalize formatting before
  opening a PR.

CI / workflows
- No GitHub Actions workflows or other CI configuration were detected in the repository root. Assume the
  primary validation is that PRs must build cleanly with `dotnet build` on .NET 10.

## Common pitfalls and helpful checks

- Always ensure `dotnet build` succeeds before creating a PR. The build is the primary gate.
- If you change public APIs or event signatures, search for usages across the repo (IDE reference search or `git grep`).
- If you add NuGet packages, ensure `dotnet restore` completes and tests/build still succeed.
- UI behavior changes should be validated by running the app and manually exercising the feature.

## Guidance for automated edits by an agent

- Make minimal, well-scoped changes.
- Run `dotnet build` after edits and include the build output summary in the PR description.
- If you touch UI XAML, run the app to verify the UX and document manual validation steps in the PR.
- If unknowns remain (missing files, ambiguous startup project, failing build), perform a small targeted search
  for the file or symbol rather than broad repository edits.

## If something fails

- If `dotnet build` fails, include the first 120 lines of compiler output in your PR or issue and stop further
  automated edits until a human reviews.

End of instructions.

---
> Source: [spectrum-health-systems/Tingen-Transmorger](https://github.com/spectrum-health-systems/Tingen-Transmorger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
