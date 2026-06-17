# Blazor project guidelines

## Project overview
- Project name: PencaTime
- Architecture: Clean architecture with CQRS
- Primary goal: maintainable, production-ready code.
- Prefer simple, clear solutions over clever ones.
- Layers:
  - PencaTime.Application: DTOs, Commands, Queries, Handlers, Interfaces
  - PencaTime.Infrastructure: EF Core, Repositories, Identity, Services
  - PencaTime.Web: Blazor components/pages and routing

## Code style
- C# 14.0, .NET 10
- C#: nullable enabled, async all the way, avoid blocking calls.
- Keep components small, extract logic into services.
- Prefer explicit names, avoid abbreviations.
- Add comments only where intent is not obvious.
- Async suffix: `*Async`
- Types/methods/properties: PascalCase
- Private fields: camelCase (no leading underscores)
- Prefer `var` when type is apparent
- Prefer file-scoped namespaces
- Nullability and defensive checks in handlers/components

## Visual Studio Guidance
- Follow the repo `.editorconfig` for formatting/naming.
- Use __Format Document__ after edits.
- Keep changes focused and small; prefer incremental PRs.

## Validation before proposing changes
- Provide step-by-step changes.
- Include build/test commands to run:
  - dotnet build
  - dotnet test
- If you add new UI, include a quick manual test checklist.

## Team Template Guidelines
- For Team Template.csv generation, flag image discovery must scan only files directly in the flags folder and must not include subfolders.

---
> Source: [foluis/Fifa-2026-flags-Getter](https://github.com/foluis/Fifa-2026-flags-Getter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
