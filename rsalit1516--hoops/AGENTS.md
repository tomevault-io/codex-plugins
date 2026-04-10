# CLAUDE.md

This file provides guidance to Claude Code when working with the Hoops repository. For frontend-specific rules see `hoops.ui/CLAUDE.md`. For backend-specific rules see `src/CLAUDE.md`.

## Project Overview

Hoops is a youth basketball league management system with a .NET 9 backend API, Azure Functions, and an Angular 20 frontend. The application manages people, households, teams, divisions, seasons, and game scheduling for a basketball league.

## Technology Stack

- **Backend**: .NET 9.0 (C#) — see `src/CLAUDE.md`
- **Frontend**: Angular 20 with Material Design and Tailwind CSS — see `hoops.ui/CLAUDE.md`
- **Database**: SQL Server with Entity Framework Core 9.0
- **Cloud**: Azure (App Services, Functions, SQL Server)
- **CI/CD**: Azure Pipelines

## Repository Structure

```
Hoops.sln               ← Solution file
src/                    ← .NET backend (API, Functions, Core, Application, Infrastructure)
tests/                  ← Backend test projects
hoops.ui/               ← Angular frontend
docs/                   ← Architecture and technical documentation
SQL/                    ← Database scripts
scripts/                ← Automation scripts
azure-pipelines.yml     ← CI/CD pipeline definition
```

## Core Domain Model

Key entities and relationships — understand these before making any cross-cutting changes:

- `Person` ↔ `Household` (many-to-one)
- `Season` → `Division` → `Team` → `Player`
- `Division` → `Game` (schedule games and playoff games)
- `Person` → `Player` (one-to-many, across seasons)

## Authentication & Authorization

- **Mechanism**: Cookie-based authentication (20-minute sliding window)
- **Cookie name**: `hoops.auth`
- **Backend**: Configured in `Startup.cs` with `CookieAuthenticationDefaults`
- **Frontend**: Route guards for admin access control in feature modules
- **Authorization**: Policy-based — `[Authorize(Roles = "Admin")]` on controllers
- **NEVER** introduce JWT authentication — the system uses cookies only

## API Conventions

- JSON serialization: Newtonsoft.Json (configured in `Startup.cs`)
- Reference handling: `ReferenceLoopHandling.Ignore` to prevent circular references
- JSON property convention: camelCase
- Swagger available at root (`/`) when running API locally
- CORS configured for `localhost:4200` and production domains

## CI/CD Pipeline

Azure Pipelines configuration in `azure-pipelines.yml`:

**Build & Test Stage:**

1. Restore and build .NET solution (Release config)
2. Run fast backend tests (`--filter TestCategory!=Slow`)
3. Install npm dependencies and build Angular app
4. Run Angular tests in CI mode
5. Publish test results and code coverage

**Deploy Stage:**

- Branch mapping: `develop` → dev environment, `master`/`main` → prod
- Publishes Azure Functions on deploy

**Main branch is `master`.** PRs should target `master`.

## Azure Best Practices

When generating Azure-related code or running Azure terminal commands, always follow Azure best practices. Reference `.github/copilot-instructions.md` for specifics.

## Work Item Management

**Azure DevOps is the single source of truth** for epics, user stories, tasks, and backlog.

**Product Area Codes:**

- `APM` - Admin People Management
- `AGM` - Admin Game Management
- `AHM` - Admin Household Management
- `APL` - Admin Player Management
- `PLY` - Playoff Management
- `USR` - User Management
- `RPT` - Reports & Analytics
- `SYS` - System Administration
- `INF` - Infrastructure & DevOps

**Work Item ID Structure:**

- Epics: `{AREA}-###` (e.g., `APM-045`, `AGM-001`)
- Stories: `{AREA}{TYPE}-###` (e.g., `APMF-001` for features, `APMB-001` for bugs)

When working on a story, copy the story content from Azure DevOps and provide it directly. Claude Code will apply the architectural standards defined in this file and the relevant sub-CLAUDE.md files.

## Technical Documentation

- `docs/architecture-overview.md` — System design and ER diagrams
- `docs/testing/` — Test specifications and strategy
- `docs/technical-requirements.md` — Functional and non-functional requirements
- `docs/templates/` — Templates for documentation and technical specifications

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rsalit1516)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rsalit1516)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
