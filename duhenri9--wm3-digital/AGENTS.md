# Project Rules and Guidelines

> Auto-generated from .context/docs on 2026-01-17T14:35:00.432Z

## README

```markdown
# WM3 Digital Documentation

## Overview

Welcome to the WM3 Digital repository. This documentation serves as a comprehensive knowledge base for developers and contributors, providing essential information about the architecture, development workflows, and key features of the project.

## Architecture

### Structure

The codebase is organized into several key components:

- **Utils**: Provides utility functions and helpers.
- **Services**: Contains business logic and integrations with external services.
- **Repositories**: Manages data access and persistence.
- **Controllers**: Handles user input and processing.
- **Components**: UI elements built using React.

### Core Exports

Key components and utility functions that you will interact with:

- **Components**: 
  - `AboutSection`: Information section about the application.
  - `Card`, `CardContent`, `CardHeader`, `CardTitle`: UI building blocks for displaying information cards.
  - `ComparisonTable`: Component for displaying side-by-side comparisons.

- **Services**:
  - `AIServiceDefinition`, `AIServiceError`: Definitions and error handling for AI services.
  - `generateBrandSnapshot`, `generateQRCodeHero`: Functions for generating brand snapshots and QR codes.

- **Utils**:
  - `cn`: Utility for handling class names.
  - `appendCorsHeaders`: Function to manage CORS headers for API responses.

## Public API

The following exports are available for use within this project:

- **Components**:
  - `Card`, `FloatingElements`, `ComparisonTable`: UI components for the application's front end.
  
- **Services**:
  - `analyzeRaioX`, `calculateCost`, `generateLandingBlueprint`: Core functions for AI analytics and service costs.
  
- **Types**:
  - Multiple types such as `AIServiceDefinition`, `FinalRequest`, `CheckoutSessionRequest` which define the structure for data being passed around.

## Key Dependencies

This project utilizes various libraries and components. Notable dependencies include:

- **src/lib/service-cache.ts**
- **src/lib/leads.ts**
- **src/lib/ai/tema-360.ts**

These files handle core functionality such as caching, lead processing, and AI integrations.

## Guides and Navigation

Visit the following detailed guides for further information:

- [Project Overview](./project-overview.md)
- [Architecture Notes](./architecture.md)
- [Development Workflow](./development-workflow.md)
- [Testing Strategy](./testing-strategy.md)
- [Glossary & Domain Concepts](./glossary.md)
- [Data Flow & Integrations](./data-flow.md)
- [Security & Compliance Notes](./security.md)
- [Tooling & Productivity Guide](./tooling.md)

## Repository Snapshot

A high-level overview of files in the repository:

- `CHANGELOG.md`
- `CONTRIBUTING.md`
- `README.md`
- `src/` — Contains all the TypeScript source files.
- `public/` — Static assets served by the application.
- `package.json` — Project dependencies and scripts.

### Document Map

| Guide                     | File                          | Primary Inputs                                    |
|---------------------------|-------------------------------|---------------------------------------------------|
| Project Overview          | `project-overview.md`        | Roadmap, README, stakeholder notes               |
| Architecture Notes        | `architecture.md`            | ADRs, service boundaries, dependency graphs      |
| Development Workflow      | `development-workflow.md`    | Branching rules, CI config                        |
| Testing Strategy          | `testing-strategy.md`        | Test configs, known flaky suites                 |
| Glossary & Domain Concepts| `glossary.md`                | Business terminology, user personas               |
| Data Flow & Integrations  | `data-flow.md`               | System diagrams, integration specs                |
| Security & Compliance     | `security.md`                | Auth model, secrets management                    |
| Tooling & Productivity    | `tooling.md`                 | CLI scripts, IDE configs                          |

This concludes the core documentation for the WM3 Digital project.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duhenri9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-15 -->
