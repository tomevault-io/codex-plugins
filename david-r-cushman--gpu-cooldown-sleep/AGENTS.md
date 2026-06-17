# GitHub Copilot Instructions For This Repository

This repository is a GitHub template for PowerShell projects. GitHub Copilot should treat these instructions as practical generation and review constraints for authored project code created from this template.

These instructions apply to:

- inline suggestions
- Copilot Chat responses
- pull request review comments
- test generation
- documentation generation

These instructions apply to authored PowerShell project code, tests, and automation. They do not necessarily apply to container bootstrap behavior, editor configuration, or environment initialization messages where limited user-facing console output may be intentional.

When repository examples and this file differ, this file takes precedence unless the repository explicitly documents an approved exception.

## Core Expectations

GitHub Copilot should:

- generate production-quality PowerShell, not demo-style scripts
- follow repository patterns before introducing new ones
- optimize for clarity, determinism, testability, and safe automation behavior
- avoid placeholder logic, fake implementations, or TODO-heavy output unless explicitly requested

## Function Design

- Use advanced functions for reusable PowerShell code.
- Include a `param()` block, even when no parameters are currently required.
- Use approved PowerShell verbs and singular, descriptive nouns.
- Prefer one public function per file when the repository is function- or module-oriented.
- Keep public functions small and composable, and move reusable logic into private helpers.
- Do not place executable business logic at module import time unless module initialization explicitly requires it.

## State Changes And Safety

- State-changing functions must use `[CmdletBinding(SupportsShouldProcess = $true)]`.
- Read-only functions should use `CmdletBinding()` without `SupportsShouldProcess`.
- Any function that creates, updates, deletes, enables, disables, assigns, revokes, imports, exports, or otherwise changes state must support `-WhatIf` and `-Confirm`.
- Use `if ($PSCmdlet.ShouldProcess(...))` around the actual mutation only.
- Do not wrap validation, lookups, or harmless preparation steps in `ShouldProcess`.

## Parameters And Pipeline Behavior

- Use PascalCase parameter names.
- Use descriptive names and avoid unclear abbreviations.
- Use appropriate parameter attributes and validation where relevant.
- Prefer strongly typed parameters over loosely structured input where practical.
- Support pipeline input only when it meaningfully improves usability.
- If pipeline input is supported, implement `begin`, `process`, and `end` blocks correctly.
- Emit one output object per input object unless the contract explicitly requires something else.

## Output And Error Handling

- Return structured objects, preferably `[PSCustomObject]` or other stable object shapes.
- Do not return formatted text intended for humans as the primary output.
- Avoid `Write-Host` in authored project code unless there is a documented exception for interactive environment or bootstrap messaging.
- Use terminating errors for unrecoverable failures.
- Use `try/catch` around operations that can fail, including network calls, file I/O, deserialization, and service operations.
- Error messages must be descriptive, actionable, and include relevant context.
- Do not swallow exceptions without adding value.

## Logging And Security

- Use `Write-Verbose` for diagnostic output and `Write-Information` for user-facing informational messages when appropriate.
- Never log secrets, tokens, credentials, or other sensitive values.
- Never hardcode credentials, secrets, tenant IDs, or environment-specific sensitive values.
- Validate external input, especially file paths, identifiers, and query values.
- Prefer least-privilege access patterns and minimize persisted sensitive data.

## Testing

- Generate Pester tests for new public functions.
- Test files should use the naming convention `<FunctionName>.Tests.ps1`.
- Use `Describe`, `Context`, and `It` blocks.
- Mock external dependencies including file I/O, network calls, service interactions, time-dependent behavior, and environment access.
- Tests should cover parameter validation, error handling, output shape, edge cases, and `ShouldProcess` behavior where relevant.
- Tests must not depend on live external systems.

## Documentation

- Public functions should include comment-based help.
- At minimum, include `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`, and `.OUTPUTS` for public functions.
- Examples should be realistic and aligned with the function's actual contract.
- Update documentation when behavior changes.

## Repository Structure

- Source code should normally reside in `/src`.
- Tests should normally reside in `/Tests`.
- Documentation should normally reside in `/docs`.
- Example scripts, if included, should reside in `/examples`.
- Module manifests and explicit exports should be maintained when the repository is module-based.
- Do not place executable business logic in the repository root.

## PowerShell Version And Compatibility

- Default target is PowerShell 7.4.x unless the repository documents a different target.
- Prefer cross-platform compatible approaches unless a Windows-only dependency is intentional and documented.
- Do not introduce syntax or APIs that conflict with the repository's supported PowerShell version.

## Microsoft Graph And External Services

- Prefer the Microsoft Graph PowerShell SDK over raw REST calls when the SDK supports the required operation.
- If raw REST is required, document why.
- Wrap service interactions in helper functions when doing so improves consistency, mockability, and testability.
- Generated code for external services must remain testable without live service calls.

## Formatting And Style

- Use 4 spaces for indentation in PowerShell files.
- Follow repository formatting rules for other file types such as JSON, YAML, and Markdown.
- Keep opening braces on the same line as the statement.
- Prefer single quotes unless interpolation is required.
- Place comments above the line they describe, not at end of line.
- Avoid trailing whitespace.
- Use LF line endings.

## What Copilot Should Flag In Review

- missing tests
- missing comment-based help for public functions
- analyzer violations
- weak or missing parameter validation
- missing `ShouldProcess` for state-changing functions
- unsafe secret handling
- unmockable external calls
- unstable or human-only output contracts
- speculative refactors outside the requested scope

## Forbidden By Default

Unless explicitly requested and justified, do not generate:

- `Invoke-Expression`
- empty catch blocks
- plaintext secret handling
- hardcoded credentials
- live external service calls in tests
- silent breaking changes to established output contracts
- formatting-only refactors unrelated to the task

## General Expectation

Consistency is more important than novelty. Generated code should align with PowerShell best practices, repository conventions, and the standards defined in this file.

---
> Source: [david-r-cushman/gpu-cooldown-sleep](https://github.com/david-r-cushman/gpu-cooldown-sleep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
