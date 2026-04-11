# Copilot Instructions for Tilbakemeldinger

## Project Overview

This is a "frackend" application (frontend + backend) for collecting feedback to NAV (Norwegian Labour and Welfare Administration). It handles three types of feedback: service complaints (serviceklage), technical issues (feil-og-mangler), and compliments (ros-til-nav).

**Owner**: Team navno (#team-navno on Slack)

## Architecture

### Tech Stack

-   **Frontend**: Preact with React components
-   **Backend**: Express.js (Node.js)
-   **Language**: TypeScript
-   **Architecture Pattern**: Express server serves the Preact frontend, artifacts AND acts as an API proxy to internal services

### Key Components

1. **Preact Frontend**: User-facing feedback forms
2. **Express Backend**:
    - Serves static frontend assets
    - Exposes three API endpoints (see below)
    - Proxies certain API requests to internal `tilbakemeldingsmottak-api`.

## API Endpoints

All proxying endpoints are POST methods that receive typed payloads from the client and forward them to the tilbakemeldingsmottak-api.

| Endpoint                  | Method | Type Definition                 | Purpose                                                                                      |
| ------------------------- | ------ | ------------------------------- | -------------------------------------------------------------------------------------------- |
| `/mottak/serviceklage`    | POST   | `common/types/ServiceKlage.ts`  | Send service complaints object to tilbakemeldingsmottak-api                                  |
| `/mottak/feil-og-mangler` | POST   | `common/types/FeilOgMangler.ts` | Send technical issues and bugs to tilbakemeldingsmottak-api                                  |
| `/mottak/ros-til-nav`     | POST   | `common/types/RosTilNav.ts`     | Send compliments to NAV to tilbakemeldingsmottak-api                                         |
| `/fodselsnr`              | GET    |                                 | Takes the user token, and pulls out the fødselsnummer in order to display frontend in forms  |
| `/enheter`                | GET    |                                 | Fetches list of Nav-kontore from internal API for populating dropdowns in the frontend forms |

## Type Safety

**IMPORTANT**: Always use the TypeScript type definitions located in `common/types/` when:

-   Creating or modifying API endpoints
-   Handling request/response payloads
-   Building frontend forms
-   Validating data structures

Type definitions are the source of truth for data contracts between frontend, backend, and downstream APIs.

## Coding Guidelines

### General Principles

-   **Type Safety First**: Use TypeScript strictly; avoid `any` types
-   **Proxy Pattern**: Express backend should remain a thin proxy layer - business logic belongs in internal APIs
-   **Error Handling**: Always handle errors gracefully and provide meaningful error messages to users
-   **Validation**: Validate incoming data against type definitions before forwarding
-   **Keep to the scope:** When fixing bugs or errors, stick to the task. Do not introduce new features or improve parts of the code unrelated to the bug or error being fixed.

### Frontend (Preact)

-   Use functional components with hooks
-   Keep components small and focused
-   Follow Preact best practices (similar to React but lighter)
-   Consider accessibility (a11y) for all form inputs

### Backend (Express)

-   Keep route handlers thin - delegate to service/controller layers
-   Use async/await for asynchronous operations
-   Implement proper error middleware
-   Log relevant information for debugging (but avoid logging sensitive data)

### File Structure

-   Type definitions: `common/types/`
-   Shared code should be in `common/` to be accessible by both frontend and backend

## Environment-Specific Behavior

### Development

-   Base URL: `https://www.ansatt.dev.nav.no/person/kontakt-oss/nb/tilbakemeldinger/`

### Production

-   Base URL: `https://www.nav.no/person/kontakt-oss/nb/tilbakemeldinger/`

## Internationalization

The application supports Norwegian (`/nb/`, `/nn/` and `/en/` locale). When adding user-facing text:

-   Language files are located in `common/language/[en|nb|nn].ts`
-   When adding new translations, ensure consistency across all supported languages.

## Testing Considerations

When adding or modifying code:

-   Write unit tests for business logic
-   Consider integration tests for API endpoints
-   Test error scenarios and edge cases
-   Validate against TypeScript type definitions

## Deployment

-   Automated via GitHub Actions
-   Workflows: "Deploy to dev" and "Deploy to prod"
-   Deploys to Nais. See [Nais documentation](https://nais.io/docs/) for details when troubleshooting.
-   See workflows in `.github/workflows/`

## Security and privacy reminders

-   Never log sensitive user information.
-   Application should be privacy first.
-   Do not store tokens locally. Do not log them.
-   Be mindful of GDPR and privacy requirements.

## References

-   [README.md](../README.md) - Project overview and ingresses
-   [CONTRIBUTE.md](../CONTRIBUTE.md) - Local development setup and deployment instructions

## Questions or Issues?

-   GitHub: https://github.com/orgs/navikt/teams/navno
-   Slack: #team-navno (for Nav employees)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navikt)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/navikt)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
