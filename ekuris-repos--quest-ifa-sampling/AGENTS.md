<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project: Quest IFA Sampling

Azure Functions app (.NET 8 Isolated) demonstrating Application Insights adaptive sampling configuration that can be toggled on/off via app settings without redeployment — only a restart is required.

### Key Design Principle
- Sampling configuration (enabled/disabled, percentage, excluded types) is driven by **app settings / environment variables**, NOT hardcoded values.
- In Azure: change Configuration > Application Settings, then restart the Function App. No code redeploy needed.

### Project Structure
- `Program.cs` — Configures Application Insights with sampling settings read from `IConfiguration`
- `host.json` — Minimal host config (sampling controlled via code, not host.json)
- `local.settings.json` — Local environment variables including sampling toggles
- `Functions/` — Sample HTTP trigger functions that generate telemetry
- `TelemetryConfig/` — Custom telemetry initializer and sampling configuration

### Conventions
- Use .NET 8 isolated worker process model
- Read all sampling config from `IConfiguration` (app settings)
- Use `Sampling__Enabled`, `Sampling__Percentage`, `Sampling__ExcludedTypes` app settings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ekuris-repos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
