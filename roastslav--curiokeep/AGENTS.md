## Copilot Instructions for CurioKeep

- **Project shape:** Spring Boot 4 backend (Java 21) with a bundled Vite/React TypeScript SPA. The SPA is packaged into the jar via the Maven profile `frontend` and served by [src/main/java/org/rostislav/curiokeep/config/SpaWebConfig.java](src/main/java/org/rostislav/curiokeep/config/SpaWebConfig.java).
- **Primary workflows / commands:**
  - Start DB: `docker compose up db`
  - Run backend locally: `./mvnw spring-boot:run` (env: `DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASS`; defaults in [src/main/resources/application.yml](src/main/resources/application.yml)).
  - Build full artifact (includes frontend): `./mvnw clean package -Pfrontend` (Node/npm auto-managed by Maven wrapper).
  - Frontend dev only: `cd frontend && npm ci && npm run dev`.
- **Data model highlights:** Flyway migrations create users/collections/modules/items ([src/main/resources/db/migration/V1\_\_init.sql](src/main/resources/db/migration/V1__init.sql), [V2\_\_auth_and_users.sql](src/main/resources/db/migration/V2__auth_and_users.sql)). Flexible item attributes live in JSONB `item.attributes`; `item.state_key` is a FK into `module_state`.
- **Core concept — Modules:** Modules are XML contracts compiled to JSON and stored in `module_definition`. The loader scans classpath modules (`classpath*:modules/*.xml`) at startup. See [src/main/java/org/rostislav/curiokeep/modules/ModuleBootstrap.java](src/main/java/org/rostislav/curiokeep/modules/ModuleBootstrap.java).
- **Module ingestion (important):** Per-module transaction handled by [ModuleLoadTx](src/main/java/org/rostislav/curiokeep/modules/ModuleLoadTx.java): XSD validation ([ModuleXsdValidator](src/main/java/org/rostislav/curiokeep/modules/ModuleXsdValidator.java)), parse ([ModuleXmlParser](src/main/java/org/rostislav/curiokeep/modules/ModuleXmlParser.java)), compile ([ModuleCompiler](src/main/java/org/rostislav/curiokeep/modules/ModuleCompiler.java)), JSON Schema validation ([ModuleContractValidator](src/main/java/org/rostislav/curiokeep/modules/ModuleContractValidator.java)) using [src/main/resources/docs/spec/module-contract-v1.schema.json](src/main/resources/docs/spec/module-contract-v1.schema.json), semantic checks, then upsert of module_definition/state/field rows. Checksums avoid redundant writes.
- **Hard rules to respect when authoring or changing modules:**
  - `OWNED` state must exist.
  - State and field keys must be unique and are normalized (often uppercase).
  - Provider mapping paths must start with `/` and reference declared providers.
  - Workflows must reference declared fields/providers.
  - Violations abort startup and return aggregated errors.
    Examples: see [src/main/resources/modules/books.xml](src/main/resources/modules/books.xml) and docs in [src/main/resources/docs/modules.md](src/main/resources/docs/modules.md).
- **Providers:** Implementations live under [src/main/java/org/rostislav/curiokeep/providers/impl](src/main/java/org/rostislav/curiokeep/providers/impl). Each provider implements `MetadataProvider` and returns a `ProviderResult` with `normalizedFields.json` and `ProviderAsset` entries. Provider credentials are configured under `curiokeep.providers.*` in application.yml; runtime skips providers if keys are missing. Tests mock HTTP; see provider tests under [src/test/java/org/rostislav/curiokeep/providers](src/test/java/org/rostislav/curiokeep/providers).
- **Business logic pointers:** `ItemService` enforces collection access, validates required module fields, uppercases/normalizes state keys and assigns default state (first defined or `OWNED`). See [src/main/java/org/rostislav/curiokeep/items/ItemService.java](src/main/java/org/rostislav/curiokeep/items/ItemService.java) and [src/main/java/org/rostislav/curiokeep/collections/CollectionService.java](src/main/java/org/rostislav/curiokeep/collections/CollectionService.java).
- **Testing notes:** Run `./mvnw test`. Tests use H2 in-memory DB and stub module loads; provider tests use `MockRestServiceServer` so they don't require live API keys.
- **Common pitfalls / quick checks before edits:**
  - Module load failures will block application startup — always validate XML against XSD and the JSON Schema before pushing changes.
  - When adding fields/states, ensure `item.state_key` FK and `ItemService` required-field checks remain satisfied.
  - Provider mapping keys must match module mappings exactly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RoastSlav)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/RoastSlav)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
