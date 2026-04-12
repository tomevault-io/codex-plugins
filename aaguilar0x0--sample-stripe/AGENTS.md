This project is using supabase and there is a file [database.types.ts](mdc:src/lib/extern/db/supabase/database.types.ts) that describes the database schema.

It is then decoupled and abstracted as Data Transfer Object (DTO) for the architecture, and is using zod for schema validation

each entity has a DTO file in `src/lib/core/dtos` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AAguilar0x0)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/AAguilar0x0)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
