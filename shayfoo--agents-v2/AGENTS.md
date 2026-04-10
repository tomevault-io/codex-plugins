
# JSDoc for Exported Functions

Every **exported** function (including methods on exported classes) must have a JSDoc comment.

## Required elements

- **`@param`** for each parameter (name, type in braces if helpful, description)
- **`@returns`** (or `@return`) describing the return value
- One-line **summary** as the first line (optional but preferred for public APIs)

## Examples

```typescript
// ❌ BAD — exported with no JSDoc
export function fetchUser(id: string) {
  return db.users.findById(id);
}

// ✅ GOOD
/**
 * Loads a user by ID from the database.
 * @param id - Unique user identifier
 * @returns The user record, or undefined if not found
 */
export function fetchUser(id: string) {
  return db.users.findById(id);
}
```

```typescript
// ❌ BAD — exported class method with no JSDoc
export class UserService {
  async getProfile(userId: string) {
    return this.repo.findOne(userId);
  }
}

// ✅ GOOD
export class UserService {
  /**
   * Fetches the full profile for a user.
   * @param userId - The user's ID
   * @returns The user profile or null
   */
  async getProfile(userId: string) {
    return this.repo.findOne(userId);
  }
}
```

## Notes

- Non-exported (private/internal) functions may omit JSDoc unless they are complex.
- Use `@throws` when the function can throw documented errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shayFoo)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shayFoo)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
