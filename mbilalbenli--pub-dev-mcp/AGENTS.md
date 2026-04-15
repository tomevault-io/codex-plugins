# pub_dev_mcp Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-09-25

## Active Technologies
- C# 13 on .NET 9.0 (AOT-ready) + `System.Net.Http`, Polly resilience policies, Serilog with source generators, OpenTelemetry SDK/Exporter, MediatR (vertical slices), FluentValidation (source generated), `Microsoft.Extensions.Caching.Memory`, `System.Text.Json` for schema validation, BenchmarkDotNet for NFR validation (001-build-an-mcp)
- No persistent datastore; short-lived in-memory caches (`IMemoryCache`) for score and dependency memoization (001-build-an-mcp)

## Project Structure
```
src/
├── PubDevMcp.Server/
├── PubDevMcp.Application/
├── PubDevMcp.Domain/
└── PubDevMcp.Infrastructure/

tests/
├── contract/
├── integration/
├── compliance/
└── performance/
```

## Commands
dotnet restore
dotnet build
dotnet test

## Code Style
C# 13/.NET 9: Follow dotnet-format/.editorconfig conventions, favor `readonly record struct` for value objects, use source-generated logging APIs.

## Recent Changes
- 001-build-an-mcp: Captured .NET 9 vertical-slice architecture, resilience, telemetry, and caching stack

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbilalbenli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
