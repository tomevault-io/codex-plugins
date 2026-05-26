# ss14-editor — Agent Instructions

Standalone visual editor for **Space Station 14 prototype YAML** files. Works against any SS14 fork. Vanilla C# server + vanilla JS frontend, no build step for the UI.

## Stack
- **.NET 10**, `OutputType=Exe`, single `ss14-editor.csproj` at repo root.
- Root namespace `Content.Editor`; classes live in `Content.Editor.Editor` (double word is intentional, do not "fix").
- All non-entrypoint types are `internal`. Tests see them via `[assembly: InternalsVisibleTo("ss14-editor.Tests")]` declared in [src/InternalsVisibleTo.cs](src/InternalsVisibleTo.cs).
- Dependencies: `System.Reflection.MetadataLoadContext` (read DLLs without executing game code), `YamlDotNet`.
- **WebUI is vanilla JS/HTML/CSS**, embedded as resources via `<EmbeddedResource Include="WebUI\**\*">`. No bundler, no transpiler, no npm. Do not introduce one.

## Project layout
```
src/              C# server, organised by domain:
  Program.cs            CLI entry, argument parsing
  InternalsVisibleTo.cs assembly attribute
  Api/                  ApiRouter + partial-class endpoint groups
                        (StatusApi, TreeApi, FileApi, FolderApi,
                         AssetApi, SourceApi, EventsApi)
  Core/                 PathSecurity, EditorContext (+ StaticMime)
  Services/             ProtoIndexService, FileTreeService,
                        FileWatcherService, EventStreamService,
                        SourceLocator, YamlPrototypeScanner
  Metadata/             MetadataExtractor, FieldExtractor,
                        MetadataModels, XmlDocReader
  Http/                 HttpJson, EditorServer (HttpListener glue)
WebUI/            HTML/CSS/JS, embedded into the binary
tests/ss14-editor.Tests/   xUnit project (net10)
publish/, bin/, obj/         build outputs (do not edit)
```

All `src/**/*.cs` share the single namespace `Content.Editor.Editor`, regardless of subfolder. The folders exist for human navigation only — moving a file between them does not require any code changes.

`ApiRouter` is a `partial class` split across `src/Api/*.cs`: the dispatcher table lives in `ApiRouter.cs`, handler methods live in the per-domain files. When adding an endpoint, register it in the dictionary in `ApiRouter.cs` AND add the handler in the matching domain file.

## Build & test
- Windows shell: `dotnet` may not be on PATH; use `& 'C:\Program Files\dotnet\dotnet.exe'`.
- Run all tests: `dotnet test tests\ss14-editor.Tests\ss14-editor.Tests.csproj`.
- The main csproj's default SDK glob would pull in `tests\**\*.cs`. The csproj contains explicit `<Compile Remove="tests\**\*" />` to prevent that. **Do not remove that block** — it will re-break the build (test files reference xUnit, which the main project does not).

## Working principles
1. **Tests first when refactoring.** Before any non-trivial change to a file in `src/`, check whether tests exist in `tests/ss14-editor.Tests/`. If they do, run them after editing. If they don't and the change is risky, add tests first.
2. **Verify before deleting.** Do not delete code claimed to be "dead" without grepping the whole workspace (including `WebUI/`) for the symbol. Many indirect calls exist (route dispatch dictionaries, JS closures, CSS class selectors).
3. **No new comments unless the *why* is non-obvious.** Existing code is sparsely commented; match that style.
4. **Path safety.** All filesystem inputs from HTTP requests go through `PathSecurity.NormalizeRelative` (see [src/Core/PathSecurity.cs](src/Core/PathSecurity.cs)). Never bypass it when adding new endpoints.
5. **No CORS.** The HTTP server intentionally sends NO `Access-Control-Allow-*` headers (see [src/Http/EditorServer.cs](src/Http/EditorServer.cs)). The WebUI is served from the same origin. Do not add CORS headers — they would let any website the user visits hit `/api/file` POST and rewrite prototype files.

## Documentation contract
- New CLI subcommand → update **CLI usage** section in [README.md](README.md).
- New `/api/*` endpoint → consider whether external docs need it (current README does not enumerate endpoints, but the change should still be reflected in tests).
- New required environment variable / runtime flag → update **Quick start** and **Development** sections of [README.md](README.md).

File-scoped instructions in `.github/instructions/` will give you more targeted rules when editing specific files.

---
> Source: [TheShuEd/SS14Editor](https://github.com/TheShuEd/SS14Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-26 -->
