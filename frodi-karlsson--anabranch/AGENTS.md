# Conventions

- Private class members have no `_` prefix.
- Exported symbols not exposed in `index.ts` get a `_` prefix.
- Newsletter ordering: things that consume go above things they consume. Public
  exports first, then helpers, then interfaces. Example:
  ```ts
  // Public exports (things that consume)
  export function createXxx(options: XxxOptions): XxxConnector { ... }
  export class DB { ... }

  // Helpers (internal utilities)
  function toConfig(options: XxxOptions): Config { ... }

  // Interfaces
  export interface XxxConnector { ... }
  export interface XxxOptions { ... }
  ```
- Prefer function declarations over `const` arrow functions.
- No default values for internal parameters where the caller can always be
  explicit; use destructured objects for boolean flags (e.g.
  `{ skipFilter }: { skipFilter: boolean }`).
- Configurable classes use a private constructor, a static `create()` factory,
  and immutable `with*()` chaining methods instead of a public constructor with
  an options bag. Each `with*()` method returns a new instance. The options
  interface is internal — do not export it.
- Test names follow the pattern `Container.method - should description`.
- All exported symbols must have JSDoc with a one-line summary. Use `@default`
  for default values and `@example` for usage examples. Avoid `@param`,
  `@returns`, and redundant type information in documentation.
- Commit messages use the format `type(scope): description`. For management
  tasks use `project` as the scope. Multiple scopes can be comma-separated:
  - `feat(fs): add readLines function`
  - `fix(anabranch): channel race condition`
  - `chore(project): update ci workflows`
  - `feat(fs,broken-link-checker): add new shared utils`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frodi-karlsson)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/frodi-karlsson)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
