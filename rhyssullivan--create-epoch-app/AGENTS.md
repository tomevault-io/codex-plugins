
# Effect Dependency Injection

**Prefer `yield* Dependency`** over passing dependencies as function arguments.

## ❌ Avoid

```typescript
const getUser = (db: Database, userId: string) =>
  Effect.gen(function* () {
    return yield* db.query("users", userId);
  });
```

## ✅ Prefer

```typescript
const getUser = (userId: string) =>
  Effect.gen(function* () {
    const db = yield* Database;
    return yield* db.query("users", userId);
  });
```

## Last Resort: When to Pass as Arguments

- Interfacing with non-Effect code
- Dynamic implementation selection at runtime
- Measured performance-critical paths

---
> Source: [RhysSullivan/create-epoch-app](https://github.com/RhysSullivan/create-epoch-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
