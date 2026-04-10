## Monorepo Conventions and Guidelines

- All packages are located in the `packages/` directory and are TypeScript ESM modules.
- Each package must have its own `tsconfig.json`, `package.json`, and (if needed) `eslint.config.js`.
- Tests
    - For small packages, tests may live alongside the source files.
    - For larger packages, tests must be placed in a separate `test/` directory within each package; do not mix tests with source files.
- Use npm workspaces for dependency and script management across packages.
- The build process prioritizes packages with names starting with `@dfb/` before building others. See `scripts/build-all.sh`.
- Use the `global_env.sh` script to set up `.env` symlinks in each package so all packages share the root environment configuration.

### Shared @dfb/* Packages
| Package         | Path                          | Purpose                                      |
|----------------|-------------------------------|----------------------------------------------|
| @dfb/ai        | packages/@dfb/ai              | Core AI utilities and OpenAI integration     |
| @dfb/db        | packages/@dfb/db              | TypeORM-based database models and logic      |
| @dfb/finddb    | packages/@dfb/finddb          | Utility for finding and working with DB files|
| @dfb/octokit   | packages/@dfb/octokit         | GitHub API and workflow integration          |

### Database & Entities
- Use `snake_case` for all database column names.
- For TypeORM entities, use `@PrimaryColumn` for string IDs; do not use `@PrimaryGeneratedColumn` for string IDs, as it always forces the type to `number`.
- Store arrays (especially vectors) as JSON strings in text columns for non-vector DB stores (e.g., native SQLite). When sending to Cosmos DB or other vector-aware stores, convert these to the expected `number[]` type.

### Coding Standards
- Always use `async/await` for asynchronous code.
- Format code with Prettier and lint with ESLint before submitting.
- Imports must always be at the top of the file, only preceded by comments.

### Stack & Tooling
- Preferred stack: TypeScript, GraphQL, Vitest, npm workspaces.
- Use GraphQL for API layers where applicable.

### Miscellaneous
- When adding new packages, ensure they follow the monorepo conventions.
- Document any scripts or utilities added to the `scripts/` directory or root `package.json`, including their purpose and usage.
- If using Cosmos DB, ensure string IDs are used for compatibility.
- When creating new packages, refer to the example structure below.

### Example: New Package Structure
```
packages/new-package/
  src/
    index.ts
  test/
    index.test.ts
  package.json
  tsconfig.json
  eslint.config.js (if needed)
```

### Example: Entity Definition
```typescript
// ...comments...
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity('my_table')
export class MyEntity {
  @PrimaryColumn()
  id!: string;
  @Column({ type: 'text', nullable: true })
  my_vector?: string; // store as JSON string in SQLite, parse to number[] for Cosmos
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dfberry)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dfberry)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
