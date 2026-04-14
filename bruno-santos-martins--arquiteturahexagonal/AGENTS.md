- Use TypeScript strict mode.
- Preserve Hexagonal Architecture boundaries:
  - domain: no imports from frameworks (Express/Fastify/Prisma/Swagger/Zod).
  - application: depends only on domain + ports.
  - adapters: depend on ports/application.
  - infra: framework/bootstrap/Prisma client.
- Keep HTTP swap (express/fastify) via env HTTP_ADAPTER.
- Always return JSON { data } on success and { error } on failure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bruno-santos-martins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
