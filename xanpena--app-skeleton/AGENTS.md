# Copilot Repository Instructions

## ROLE

You are a senior developer and code reviewer for this repository. Be concise, follow our workflow strictly, and respect the style guide.

## MANDATORY WORKFLOW

1. Read `README.md` and use `make help` to see available commands
2. Use `make` commands instead of manual npm/docker commands
3. All commands must be executed inside the Docker container and never locally
4. Never insert comments into the code, only in configuration files and markdown files
5. Never create sh scripts that the user has not specified
6. Do not install dependencies without user authorization
7. You cannot run commands that are not in the `Makefile` or `VS Code tasks`
8. You can ask the user to add commands to the `Makefile`
9. Always add strict typing to all code

## SECURITY & PRIVACY

- Never open/read/include content from files matching:
  - `.env`, `.env.*`
  - `*.key`, `*.pem`
  - `secrets/`
- Do NOT expose credentials, tokens, API keys, or personal data in any output.
- This repo uses `.copilotignore` to enforce exclusions.

## Architecture Documentation

Multi-platform React Native application targeting **iOS**, **Android**, **Web**, and **Desktop**. Containerized development with Expo Development Build for local native app generation without cloud dependencies. Desktop applications powered by Tauri with Rust backend for optimal performance and security.

### Structure

```
src/
+-- app/                    # Expo Router pages
+-- assets/                 # Images, icons (PNG optimized)
+-- modules/                # Bounded contexts with hexagonal architecture

android/                    # Generated native Android code (git ignored)
src-tauri/                  # Tauri desktop application (Rust backend)
+-- src/                    # Rust source code
+-- icons/                  # Desktop app icons
+-- tauri.conf.json        # Tauri configuration

app.json                    #
Dockerfile                  # Single Docker image for development
docker-compose.yml          # Container orchestration
Makefile                    # Command automation
metro.config.js             #
package.json                #
tsconfig.json
```

### Hexagonal Architecture with CQRS

This app implements a **Hexagonal Architecture** (Ports & Adapters) combined with **CQRS** (Command Query Responsibility Segregation) pattern. The application is organized in **bounded contexts** that contain complete business functionality.

### Key Architecture Principles

1. **Dependency Inversion**: Domain layer doesn't depend on infrastructure
2. **Single Responsibility**: Each class has one clear purpose
3. **Command/Query Separation**: Write and read operations are separated
4. **Value Objects**: Encapsulate business rules and validation
5. **Result Pattern**: Explicit error handling without exceptions
6. **Repository Pattern**: Abstract data access behind interfaces
7. **Domain Services**: Business logic that doesn't belong to entities

### Bounded Context Structure

Each bounded context follows this standardized structure:

```
src/modules/{context}/
+-- domain/              # Business logic layer
¦   +-- model/          # Domain entities and value objects
¦   +-- repository/     # Repository interfaces (ports)
¦   +-- service/        # Domain services
+-- application/        # Application layer (use cases)
¦   +-- command/        # Commands (write operations)
¦   +-- query/          # Queries (read operations)
+-- infrastructure/    # Infrastructure layer (adapters)
¦   +-- repository/    # Repository implementations
+-- ui/                # Presentation layer
    +-- controllers/   # UI controllers
    +-- components/    # React components
    +-- views/         # Screen components
```

### Creating a New Bounded Context: CRUD Example

Follow these steps to implement a new bounded context called `crud`:

### Fast modules

We can simplify bounded context generations with this structure reusing shared utilities:

```
src/modules/{context}/
+-- domain/
¦   +-- model/
¦       +-- {Entity}.ts
+-- ui/
¦   +-- types/
¦   ¦   +-- {Entity}.types.ts
¦   +-- components/
¦   ¦   +-- {Entity}Editor/
¦   ¦   ¦   +-- {Entity}Editor.component.hook.tsx
¦   ¦   ¦   +-- {Entity}Editor.component.tsx
¦   ¦   +-- {Entity}Item/
¦   ¦       +-- {Entity}Item.component.tsx
¦   +-- views/
¦       +-- {entity}sList/
¦       ¦   +-- {Entity}sList.view.tsx
¦       +-- create{Entity}/
¦       ¦   +-- Create{Entity}.view.tsx
¦       +-- edit{Entity}/
¦           +-- Edit{Entity}.view.tsx
```

#### 1. Domain Layer

```
src/modules/crud/domain/
+-- model/
   +-- CrudItem.ts                              # Domain entity interface
+-- repository/
¦   +-- crud/
¦       +-- Add.repository.ts                    # Add operation interface
¦       +-- Update.repository.ts                 # Update operation interface
¦       +-- Delete.repository.ts                 # Delete operation interface
¦       +-- Search.repository.ts                 # Search operation interface
+-- service/
    +-- crud/
        +-- Add.service.ts                       # Add business logic
        +-- Update.service.ts                    # Update business logic
        +-- Delete.service.ts                    # Delete business logic
        +-- Search.service.ts                    # Search business logic
```

#### 2. Application Layer

```
src/modules/crud/application/
+-- command/
¦   +-- addCrudItem/
¦   ¦   +-- AddCrudItem.command.ts              # Add command interface
¦   ¦   +-- AddCrudItem.command.handler.ts     # Add command handler
¦   +-- updateCrudItem/
¦   ¦   +-- UpdateCrudItem.command.ts          # Update command interface
¦   ¦   +-- UpdateCrudItem.command.handler.ts # Update command handler
¦   +-- deleteCrudItem/
¦       +-- DeleteCrudItem.command.ts          # Delete command interface
¦       +-- DeleteCrudItem.command.handler.ts # Delete command handler
+-- query/
    +-- searchCrudItems/
        +-- SearchCrudItems.query.ts           # Search query interface
        +-- SearchCrudItems.query.handler.ts  # Search query handler
```

#### 3. Infrastructure Layer

```
src/modules/crud/infrastructure/
+-- repository/
    +-- add/
    ¦   +-- Add.repository.ts                    # Add repository implementation
    ¦   +-- Request.mapper.ts                    # Request mapping
    ¦   +-- Request.interface.ts                 # Request interface
    +-- update/
    ¦   +-- Update.repository.ts                 # Update repository implementation
    ¦   +-- Request.mapper.ts                    # Request mapping
    ¦   +-- Request.interface.ts                 # Request interface
    +-- delete/
    ¦   +-- Delete.repository.ts                 # Delete repository implementation
    ¦   +-- Request.mapper.ts                    # Request mapping
    ¦   +-- Request.interface.ts                 # Request interface
    +-- search/
        +-- Search.repository.ts                 # Search repository implementation
        +-- Request.mapper.ts                    # Request mapping
        +-- Request.interface.ts                 # Request interface
        +-- Response.mapper.ts                   # Response mapping
```

#### 4. UI Layer

```
src/modules/crud/ui/
+-- controllers/
¦   +-- AddCrudItem.controller.ts               # Add controller
¦   +-- UpdateCrudItem.controller.ts            # Update controller
¦   +-- DeleteCrudItem.controller.ts            # Delete controller
¦   +-- SearchCrudItems.controller.ts           # Search controller
+-- components/
¦   +-- CrudItemCard.tsx                        # Item display component
¦   +-- CrudItemForm.tsx                        # Item form component
¦   +-- CrudItemList.tsx                        # Items list component
+-- views/
    +-- CrudItemsScreen.tsx                     # Main screen
    +-- AddCrudItemScreen.tsx                   # Add item screen
    +-- EditCrudItemScreen.tsx                  # Edit item screen
```

---
> Source: [xanpena/app-skeleton](https://github.com/xanpena/app-skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
