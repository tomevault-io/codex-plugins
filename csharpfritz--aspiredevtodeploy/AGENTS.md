# Copilot Instructions for .NET Aspire Lecture Repository

## Repository Overview

This repository contains reference materials for a live-coding lecture about **.NET Aspire**, demonstrating the journey from a basic web application to a fully distributed, cloud-ready application with AI capabilities.

### Repository Structure

- **`AspireFromDevToDeployment.pptx`** - Presentation slides for the session introduction
- **`speaker-notes.md`** - Detailed step-by-step instructions for progressing through each demo stage
- **`start/`** - Basic MyWeatherHub web application (no Aspire)
- **`demo1/`** - Aspire-enabled with service discovery and dashboard
- **`demo2/`** - Added health checks and external service monitoring
- **`demo3/`** - Added Redis caching for performance
- **`demo4/`** - Added AI capabilities with GitHub Models
- **`complete/`** - Final deployment-ready application with Azure configuration

### Application: MyWeatherHub

The demo application is a weather application consisting of:
- **Api** - Backend weather API service
- **MyWeatherHub** - Frontend Blazor web application
- **AppHost** - .NET Aspire orchestration project (added in demo1+)
- **ServiceDefaults** - Shared Aspire service defaults (added in demo1+)

## Demo Progression

Each demo folder represents a savepoint in the live-coding session:

1. **start → demo1**: Add Aspire orchestration, service discovery, and dashboard
2. **demo1 → demo2**: Add health checks for the external NWS Weather API
3. **demo2 → demo3**: Add Redis caching with `Aspire.Hosting.Redis`
4. **demo3 → demo4**: Add AI-powered forecast summarization with GitHub Models
5. **demo4 → complete**: Add Azure deployment configuration (`azure.yaml`)

## Key Technologies

- **.NET Aspire** - Cloud-ready distributed application framework
- **Blazor** - Frontend web framework
- **Redis** - Distributed caching
- **GitHub Models** - AI integration (gpt-4o-mini)
- **Azure Container Apps** - Deployment target
- **Azure Developer CLI (azd)** - Deployment tooling

## Code Conventions

- Use `builder.AddServiceDefaults()` in all ASP.NET Core projects
- Use `app.MapDefaultEndpoints()` for health and readiness endpoints
- Use Aspire service discovery URLs (e.g., `"https+http://api"`) instead of hardcoded URLs
- Reference external APIs using `builder.AddExternalService()`
- Include health checks for all external dependencies

## When Assisting with This Repository

1. **Understand the demo flow** - Changes should align with the progressive demonstration from start → complete
2. **Preserve demo boundaries** - Each folder is a distinct savepoint; don't merge content across folders
3. **Reference speaker-notes.md** - Contains the detailed steps and key talking points for each demo stage
4. **Maintain consistency** - The Api and MyWeatherHub projects should have parallel structure across all demo folders
5. **Consider live-coding context** - Code should be simple and demonstrable in a live session

## Prerequisites for Running Demos

- .NET 10 SDK
- Docker Desktop (required for Redis in demo3+)
- Visual Studio 2026 or VS Code with C# extension
- Azure CLI (for deployment demo in complete/)
- GitHub account with GitHub Models access (for AI demo in demo4+)

## Common Tasks

- **Adding a new demo stage**: Create a new folder, copy from previous stage, apply incremental changes
- **Updating NuGet packages**: Update across all demo folders that use the package for consistency
- **Modifying speaker notes**: Keep in sync with actual code changes in demo folders

---
> Source: [csharpfritz/AspireDevToDeploy](https://github.com/csharpfritz/AspireDevToDeploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-02 -->
