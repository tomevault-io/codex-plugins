## osu! codebase: AI Agent Guidance

Short, actionable instructions for AI coding agents working on this repository. Focus on project-specific architecture, workflows, and conventions. Reference key files and patterns for maximum productivity.

### Architecture Overview
- **Core projects:**
	- `osu.Game`: Game engine, ruleset system, and gameplay logic
	- `osu.Desktop`: Desktop client entry point
	- `osu.Game.Rulesets.*`: Individual ruleset implementations (see `Ruleset`, `DrawableRuleset`, `RulesetInputManager`, `Mods`)
	- `osu.Game.Tests`: Visual and unit tests (see `Visual/` for UI test scenes)
- **UI/DI pattern:**
	- Use `[BackgroundDependencyLoader]` for dependency injection and initialization
	- Use `LoadComplete()` for event binding and post-load logic
	- Reactive state: use `IBindable<T>`, `BindValueChanged(..., true)` for initialization
- **Partial classes:**
	- Large components are split into partial classes (search for `partial class`). Always check for sibling partial files before editing.

### Developer Workflows
- **Build:**
	- Desktop: `dotnet build osu.Desktop` or VS Code task `Build osu! (Debug)`
	- Tests: `dotnet build osu.Game.Tests` or VS Code task `Build tests (Debug)`
- **Test:**
	- Visual/unit: Run via IDE test runner or `dotnet test osu.Game.Tests`
- **Format & static analysis:**
	- Run `dotnet format` and `./InspectCode.ps1` (Windows) or `./InspectCode.sh` (Unix)
- **Local framework/resources:**
	- Use `UseLocalFramework.ps1` and `UseLocalResources.ps1` to develop against local `osu-framework`/`osu-resources`

### Project Conventions
- **PRs:**
	- Single-purpose, small, Angular-style commit messages: `type(scope): subject`
	- UI changes: add/update a visual test in `osu.Game.Tests/Visual/`
	- Star-rating/PP changes: use `pp-dev` branch (not `master`)
- **Code style:**
	- Run `dotnet format` and `InspectCode.ps1` before PRs
	- Avoid banned APIs in `CodeAnalysis/BannedSymbols.txt`
- **Testing:**
	- Add visual tests for UI, unit tests for logic. See `osu.Game.Tests/Visual/` for examples.

### Key Patterns & Examples
- **Screens/UI:** See `osu.Game/Screens/` for `[BackgroundDependencyLoader]`, `LoadComplete()` usage
- **Ruleset/scoring:** See `osu.Game/Rulesets/*/`, `osu.Game/Rulesets/Difficulty/`, `osu.Game/Rulesets/Scoring/`
- **Partial class usage:** Search for `partial class` in large components
- **Build/test scripts:** See root scripts (`InspectCode.ps1`, `UseLocalFramework.ps1`)

### If Unsure
- Check `CONTRIBUTING.md` and `CodeAnalysis/BannedSymbols.txt` for policy and forbidden APIs
- Ask maintainers via repo issues or development Discord

---
For more examples or clarification, request expansion of specific sections.

---
> Source: [ShikkesoraSIM/torii-osu](https://github.com/ShikkesoraSIM/torii-osu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-05 -->
