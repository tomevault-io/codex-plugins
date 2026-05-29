# Copilot Instructions

## General Rules

- Never commit changes unless explicitly asked to do so.
- **Never deploy to production without explicit user confirmation.** Before invoking the `deploy` skill or running any `swa deploy`, `docker push`, or `az containerapp` update command, always stop and ask the user: "Ready to deploy to production?" and wait for a clear yes.
- **Always validate UI changes locally before marking a task complete.** After any change to Razor components, CSS, or JavaScript, start the local stack (use the `run-locally` skill) and use Playwright to verify the behavior in the browser. Do not consider a UI task done until it has been visually confirmed working locally.

## Architecture

Three-tier application hosted on Azure:

```
Browser → Azure Static Web Apps (Easy Auth)
              → /api/* → Azure Functions proxy (api/)
                            → ASP.NET Core backend (src/Server/)
                                  → SQLite database
```

- **`src/WasmApp/`** — Blazor WebAssembly frontend (C#, Razor, Bootstrap)
- **`src/Server/`** — ASP.NET Core Web API backend with SQLite
- **`src/Shared/`** — Shared DTOs/contracts used by both frontend and backend
- **`api/`** — Azure Functions v4 (Node.js) API proxy
- **`test/SerializerTests/`** — MSTest unit tests (net9.0)
- **`infrastructure/`** — Bicep templates for Azure Container Apps + SWA

## Build & Test

```powershell
# Build the whole solution
dotnet build rss-reader.sln

# Run all tests
dotnet test rss-reader.sln

# Run a single test class
dotnet test test/SerializerTests --filter "ClassName=SerializerTests.SerializerTests"

# Build Docker image for the backend (run from repo root)
docker build -f src/Server/Dockerfile -t rss-reader-api:local .

# Build and deploy the frontend (run from rss-reader/)
swa build
swa deploy --env production
```

**Local dev**: Hit F5 in Visual Studio — launches frontend and backend together.

For containerized backend testing, see [LOCAL-TESTING.md](../LOCAL-TESTING.md).

## ⛔ Local Dev Build Rule: Always Debug, Never Release

**When running locally, ALWAYS use `dotnet build -c Debug`. NEVER run `dotnet publish -c release` or `dotnet build -c Release` for local testing.**

The release build excludes `appsettings.Development.json` from the output (`CopyToPublishDirectory = Never`). This file contains `EnableTestAuth: true`, which is what allows the test user (`testuser2`) to bypass SWA Easy Auth locally. Without it, every API call returns 401 and the app appears broken even though the code is correct.

The `swa start rss-reader-local` config uses `dotnet watch run` (Kestrel dev server on port 8443) as the app source — it serves files directly from the source `wwwroot/`, not from a publish output directory. The `outputLocation` in `swa-cli.config.json` for the `rss-reader-local` config points to `bin/debug/net9.0/wwwroot` (the debug build output). Never use a release build or create a release placeholder directory for local testing.

## Authentication Flow

Authentication is a two-stage chain:

1. **Azure SWA Easy Auth** authenticates the browser via AAD (login at `/.auth/login/aad`). SWA injects `x-ms-client-principal` (base64 JSON) into requests to the Function proxy — browsers cannot forge this.

2. **Azure Functions proxy** (`api/src/functions/ApiProxy.js`) extracts the Easy Auth principal, then forwards requests to the backend with:
   - `X-Gateway-Key`: a shared secret (env var `RSSREADER_API_KEY`)
   - `X-User-Id`: the raw base64 `x-ms-client-principal` value

3. **Backend** (`StaticWebAppsAuthenticationHandler`) validates `X-Gateway-Key` against `RSSREADER_API_KEY` env var, then parses `X-User-Id` to build the `ClaimsPrincipal`.

For local development, set `RssAppConfig__IsTestUserEnabled=true` to bypass auth with a fake test user (`testuser2`).

## Key Conventions

### Shared Contracts
All DTOs live in `src/Shared/Contracts/`: `NewsFeed`, `NewsFeedItem`, `RssUser`, `OpmlImport`. Both the frontend (`FeedClient`/`UserClient`) and backend controllers use these same types.

### Repository Pattern
The backend uses interfaces (`IFeedRepository`, `IItemRepository`, `IUserRepository`) with SQLite implementations in `src/Server/Data/`. Repositories are registered as singletons and initialize their own database tables on construction. The order of singleton creation matters (see `Program.cs` — feed → user → item).

### Background Work
Feed refresh runs via a `BackgroundWorkQueue` + `BackgroundWorker` hosted service. Enqueue work items to `BackgroundWorkQueue`; `FeedRefresher` handles the actual HTTP fetch and parse. Don't call `FeedRefresher` directly from controllers — enqueue instead.

### Configuration
App config is loaded into `RssAppConfig` from `appsettings.json` under the `RssAppConfig` section. The backend reads `DbLocation` for the SQLite path (`/tmp/storage.db` in Docker, copied to `/data/` for persistence). Frontend config is in `RssWasmConfig`.

### DatabaseBackupService
A periodic image-sync + system-stats service. The SQLite database lives at `/tmp/storage.db` (ephemeral container storage) and is replicated to Azure Blob Storage by **Litestream** (the sole DB backup path). On container startup, `Program.cs` calls `RestoreFromBackupAsync` before any repository is instantiated to copy cached images from `/data/images/` (Azure Files) into `wwwroot/images/`. Every 5 minutes it then syncs new images back from `wwwroot/images/` to `/data/images/` (only new files, never overwrites) and records a system-stats snapshot via a read-only query against the active DB.

This service must NEVER write to `/tmp/storage.db` — Litestream owns the DB, and any write here would generate WAL/LTX traffic. The tests assert this contract explicitly.

### OPML Import/Export
OPML import/export is handled by the static `OpmlSerializer` class (`src/Server/Services/Serialization/OpmlSerializer.cs`), shared between the backend and tests.

- **Export** (`GET /api/feed/exportOpml?userId=...`): Fetches the user's feeds and serializes to OPML 2.0 XML. Feed tags are written as a comma-separated `category` attribute on each `<outline>` element.
- **Import** (`POST /api/feed/importOpml`): Parses OPML XML, reads `category` attribute for tags, and bulk-inserts feeds via `feedRepository.AddFeeds`. Both endpoints enforce that the requesting user can only import/export their own data (comparing `ClaimTypes.Name` against the requested `userId`).

Security: The XML parser has DTD processing disabled and a 1MB content size limit to prevent XXE and DoS attacks. Do not change these settings.

### HTTP Redirects
`RedirectDowngradeHandler` is registered on the `RssClient` `HttpClient` to prevent HTTPS→HTTP redirect downgrade attacks when fetching external RSS feeds.

### Frontend HTTP Clients
The Blazor app uses named `HttpClient` instances via `IHttpClientFactory`. The client named `"ApiClient"` is used by `FeedClient` and `UserClient` to call the backend through the SWA proxy.

## Agents

Specialized agents live in `.github/agents/`. Each has a `name`, `description`, `model`, and `tools` front-matter, followed by detailed system instructions. Prefer these agents over the base Copilot model for tasks in their domain.

| Agent | File | When to use |
|---|---|---|
| `azure-deployer` | `agents/azure-deployer.agent.md` | Deploy the app, read Azure Container App / SWA logs, troubleshoot deployment or runtime errors, check replica/scaling status, diagnose Azure infrastructure issues. Uses Azure MCP tools for structured diagnostics; falls back to `az` CLI. |
| `full-stack-developer` | `agents/full-stack-developer.agent.md` | Implement features or fix bugs spanning frontend and backend — Blazor WASM, ASP.NET Core API, SQLite, Azure Functions proxy. Always adds MSTest unit tests and validates frontend changes with Playwright. |
| `ui-developer` | `agents/ui-developer.agent.md` | Improve look-and-feel, fix layout/spacing, make pages mobile-responsive, improve color schemes, enhance accessibility, or rework navigation. Always validates with Playwright at desktop (1280px) and mobile (375px). |

## Skills

Skills live in `.github/skills/<name>/SKILL.md`. They are step-by-step runbooks executed by invoking the `skill` tool with the skill name.

| Skill | File | What it does |
|---|---|---|
| `deploy` | `skills/deploy/SKILL.md` | Full end-to-end production deployment: resolves prerequisites (Node 20, GITHUB_USERNAME, Docker, az CLI, SWA CLI) → builds & pushes the backend Docker image to GHCR → updates the Azure Container App → builds & deploys the SWA frontend → **validates the deployment** (Azure resource health via MCP + browser smoke test via Playwright). |
| `run-locally` | `skills/run-locally/SKILL.md` | Start the full local dev stack: Docker backend container + SWA emulator frontend at `http://localhost:4280`. Auth is bypassed via `RssAppConfig__IsTestUserEnabled=true` (test user `testuser2`). |
| `stop-local` | `skills/stop-local/SKILL.md` | Stop all locally running servers: SWA emulator (port 4280), Azure Functions host (port 7071), Blazor dev server, `dotnet watch` processes, and the backend Docker container. |
| `playwright-browse` | `skills/playwright-browse/SKILL.md` | Start an interactive Playwright browser session. Navigates to a URL (default: `https://rss.brandonchastain.com`), bypasses the Blazor service worker cache, takes snapshots/screenshots, and interacts with the page. Handles the login prompt if the user is not authenticated. |
| `develop` | `skills/develop/SKILL.md` | **Default workflow for all code changes.** End-to-end TDD: Plan → Red (failing tests) → Green (implement) → Build & Test → Local Smoke Test (Playwright) → PR → Merge → Deploy → Production Smoke Test. Always use this skill when asked to implement a feature or fix a bug. |

### Deploy skill — validation step

After deploying, the `deploy` skill runs a two-part validation:

1. **Azure health check (MCP)** — calls `resourcehealth availability-status get` for `rss-reader-api` to confirm `Available` status, then checks revision traffic with `az containerapp revision list`.
2. **Browser smoke test (Playwright)** — navigates to `https://rss.brandonchastain.com`, clears the Blazor service worker cache, confirms the homepage loads, detects login state (and prompts the user to log in if needed), then exercises `/feeds` and `/timeline` and takes a screenshot.

---
> Source: [brandonchastain/rss-reader](https://github.com/brandonchastain/rss-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-29 -->
