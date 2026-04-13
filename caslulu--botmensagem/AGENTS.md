# GitHub Copilot Instructions

This file contains instructions for GitHub Copilot on how to assist with code generation and suggestions within this repository.

## Context

This is an application used to automate tasks of an Insurance agency. It performs many tasks including sending messages, creating cards, generating quotes, pricing, and creating RTAs.

## Stack & Architecture

1. **Stack**: Electron desktop application with React (TypeScript) for the renderer process and JavaScript for the main process. SQLite (via sql.js) stores user data.
2. **Coding Pattern**: Follow clean code principles with modular components, clear naming conventions, and exception handling where necessary. Avoid excessive comments.
3. **Folder Structure**: Code is organized into distinct modules. Place shared components in a `helpers/` folder when used across multiple modules.
4. **UI Design**: Modern, minimalistic, and professional interface using Tailwind CSS and React component libraries. Target non-technical users with intuitive, accessible UX.

## Modules

### 1. WhatsApp Automation
Sends messages to every archived group. Users select messages and images, then the application sends them to all archived groups.

### 2. Trello Automation
Creates Trello cards from predefined templates. Users fill a form and the application creates cards in the board and stores the quote in the database for future use.

### 3. Quote Automation
Generates insurance quotes. Users select a quote (created via Trello module) and an insurance company (Progressive, Liberty; future support for Allstate, Geico). The application fills out the quote on the insurance company website.

### 4. Price Module
Generates image templates for insurance companies. Users select the quote, insurance company, and fill the price. The application adds taxes, generates the image, saves to disk, and uploads to the Trello card.

### 5. RTA Module
Creates RTA (Registration Title Application) forms. Users fill required information and receive a completed PDF form.

---

## Development Guidelines

### Electron Security & IPC
- Use secure `BrowserWindow` settings: enable `contextIsolation`, disable `nodeIntegration`, enable `sandbox`
- Return structured results via `src/main/utils/result.js` (`createSuccess`, `createError`)
- Register IPC handlers under `src/main/ipc/` following the `register*Handlers` pattern in `src/main/ipc/index.js`
- Keep IPC channels typed and predictable

### Persistence & File System
- Use `src/main/database.js` (sql.js) for all database operations
- Always resolve paths via `src/main/automation/utils/path-resolver.js` (`getUserDataDir`, `getSessionsRoot`)
- Never write files outside the resolved user data directory
- Keep filenames deterministic and avoid unintentional overwrites

### Logging
- Use `src/main/automation/utils/logger.js` for progress/error logs in automation flows
- Emit user-facing logs via its `emitter` when available instead of `console.log` directly
- Include contextual prefixes (profile name, module, etc.)

### React Components & TypeScript
- Place shared React components under `src/renderer/src/components/` or `src/renderer/src/components/layout/`
- Use strict, typed props and IPC bridges defined in `src/preload/preload.js`
- Avoid `any` type; define lightweight types in `src/renderer/src/global.d.ts` when needed
- Follow clean component patterns with clear separation of concerns

### UI & Styling
- Tailwind CSS is the base styling framework
- Prefer headless libraries (Radix UI, Headless UI) for accessible components
- Use controlled form libraries (React Hook Form + Zod) for validation
- Keep UI minimalistic, consistent, and intuitive for non-technical users

### Testing
- Add unit tests (Jest/Vitest) for pure helpers and business logic
- Write e2e tests (Playwright) under `e2e/` for critical user flows
- Keep tests focused and avoid flaky selectors
- Test security boundaries and error states

### Error Handling
- Surface actionable messages in the renderer
- Return machine-friendly structures via IPC: `{ success, error, ... }`
- Use `createError` with safe messages; avoid leaking stack traces to UI
- Handle errors gracefully with user-friendly messages

### Module Boundaries
- Respect existing module structure under `src/main/*` (whatsapp, trello, quotes, price, rta)
- New quote providers go in `src/main/automation/quotes/providers/`
- Expose consistent interfaces: login, form fill, submission
- Keep module dependencies minimal and explicit

### Configuration & Secrets
- Read Trello credentials from `src/main/config/trello-config.js` (copy from `.example`)
- Never hardcode secrets in source code
- Use environment fallbacks only when explicitly supported
- Keep sensitive configuration out of version control

### Assets & Output
- Price images/output write to `PathResolver.getUserDataDir()/price/output`
- Follow patterns in `src/main/price/services/priceService.js`
- Use deterministic filenames for predictable file management

### Performance & Rate Limiting
- Implement rate limiting/backoff for WhatsApp automation to avoid throttling
- Batch operations where possible
- Reuse browser sessions via `PathResolver.getSessionsRoot()`
- Profile performance-critical paths

### Code Style
- Follow ESLint/Prettier defaults if present
- Use clear, descriptive names
- Prefer early returns over nested conditionals
- Avoid inline comments except where strictly necessary
- Keep functions small and focused (single responsibility)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caslulu)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/caslulu)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
