## DATABASE

- Use simple SQL comands, so that solution is db agnostic (ie. works with Postgress or H2)
- `./be/dailogi-server/src/main/java/com/github/vvojtas/dailogi_server/db/entity` - database models
- `./be/dailogi-server/src/main/java/com/github/vvojtas/dailogi_server/db/repository` - database repositories
- `./be/dailogi-server/src/main/resources/db/migration` - flyway migrations
- `./ai/db-plan.md` - db schema description
- When modyfying java entities / flyway migrations / db schema - remember to update others
- When updating flyway migration ask whether to modify existing one or create new migration file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vvojtas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vvojtas)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
