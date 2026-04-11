# Copilot Instructions for TopShelfLiveInventory

Do not make extra markdown files or documentation files unless requested. For Example, never make: "docs/notes/implimentation-complete.md" or "docs/notes/summary-of-work.md".
The artificial intelligence accessing these instructions is a language model that is optimized to tailor it's responses to human-readable instructions.
Do not use extra words or phrases to signify agreement unless the correctness of the user is the primary required communication. For example, never say: "You're absolutely correct!" or "I can see the issue!"

This application is in development and is not yet used in production. We are working on a development branch.
Simplicity and modularity are prioritized. Removing unused code is prioritised. Adding new systems should only be done if absolutely necessary.
If possible, always work within the existing application structure and patterns.

## Project Overview

This is Top Shelf Exhibits' modular JavaScript web application for live inventory management. Top Shelf Exhibits structures work into domains:

- Inventories: lists of items and current item information. Items are stored in a warehouse in crates. These inventories are used to manage the items available for shows.
- Pack Lists: lists of items to be packed in crates for a show, including quantities and other details. These lists are used to prepare items to ship to shows and return items from shows.
- Production Schedule: a schedule of shows and their related production tasks. This schedule is used to manage the timing and logistics of shows.

The codebase is organized under `docs/`, with subfolders for CSS, images, JS, and notes. The main logic resides in `docs/js/`, split into functional domains:

- **application/**: The user interface is written using Vue 3. It manages its own reactive data during the application lifecycle and only interacts with persistent data through the API layer. UI logic is split into reusable Vue components by domain (e.g., `InventoryTable.js`, `modalComponent.js`).
- **data_management/**: The API acts as an interface between the application and data manipulation functions. It manages caching and cache invalidation automatically via wrapper methods. All data access and mutation should go through the API, not directly to Google Sheets or the database abstraction.
- **google_sheets_services/**: This layer contains all direct interaction with Google Sheets, including authentication, queries, and Google-specific logic. All persistent data flows through this layer, and it should not be accessed directly by the UI or application logic.

## Architecture & Data Flow

- **Presentation Layer**: The UI is built using Vue 3, which manages its own reactive state and lifecycle. It communicates with the API layer for all persistent data operations.
- **API Layer**: Provides a clean interface for the application to interact with data. Handles caching and cache invalidation automatically using wrapper methods. All data management and manipulation logic is centralized here.
- **Google Sheets Services Layer**: Handles all direct communication with Google Sheets, including authentication and queries. This layer is responsible for Google-specific logic and should not be accessed directly by the UI or API consumers.
- **Data Storage**: User and app data is stored in Google Sheets, accessed only via the Google Sheets services layer.
- **User Data**: Each user has a dedicated tab in the CACHE sheet, managed by utility methods (see `application-utils.js`). Tabs are dynamically created and updated.

## Key Patterns & Conventions (by Domain)

### Vue Application (UI)

- UI components manage their own reactive state and lifecycle using Vue 3.
- All persistent data operations are performed via the API layer; never access data directly.
- UI logic is split into reusable Vue components by domain:
  - docs/js/application/components/content/: contains all components that allow the application to manage its content according to Top Shelf Exhibits work domains. Most API interactions are handled here.
  - docs/js/application/components/dashboard/: contains components and logic that allow users to customize their home page uniquely for their workflow.
  - docs/js/application/components/interface/: contains components and logic to manage reusable UI elements like modals, buttons, tables, and forms.
  - docs/js/application/components/navigation/: contains components and logic to manage the application navigation and routing.

### API/Data Management

- API provides a clean interface for the application to interact with data.
- The API references abstractions based on domains:
  - database.js: contains the database abstraction layer which provides methods for interacting with Google Sheets Services.
  - application-utils.js: contains data management functions for application-specific operations, such as storing and retrieving user data.
  - inventory-utils.js: contains data management functions for managing inventory-related data and workflows.
  - packlist-utils.js: contains data management functions for managing pack lists and their items.
  - production-utils.js: contains data management functions for managing production-related data and workflows.
- All data access and mutation goes through the API to the abstractions and eventually to Google Sheets if necessary.
- Beneath the API, the abstraction layer handles caching and cache invalidation automatically using `wrapMethods` for consistent method exposure.

### Google Sheets Services

- Contains all direct interaction with Google Sheets, including authentication, queries, and Google-specific logic.
- All persistent data flows through this layer, and it should not be accessed directly by the UI or application logic.
- Internally attempts to maintain authentication and session management for Google Sheets.

## Developer Workflow

We are NOT explicitly running a local python or Node.js server.
The VSCode extension LiveServer is running a local server at 'http://127.0.0.1:5500/docs/#'.

### Vue Application (UI)

- No build step: JS/CSS/HTML are used directly; changes are reflected immediately. However, access to Google Sheets data requires the site to be deployed on a web server.
- Debugging: Use browser console logging and Vue devtools for runtime issues.

### API/Data Management

- database abstraction layer automatically switches between fake and real google sheets access based on it's environment.
- Debug using console logs and inspect API responses.
- Manual testing of API endpoints and wrapper methods.

### Google Sheets Services

- Any changes in google sheets services must be reflected in FakeGoogle.js for local testing.
- Debug using logs for Google Sheets queries and authentication.
- Manual verification of data in Google Sheets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danschwab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/danschwab)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
