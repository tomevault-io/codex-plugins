# Copilot Instructions

## Commit instructions

- Use conventional commits
- Run `npm run lint` and `npm run format` before committing to ensure code quality and consistency.

## Commit Message Format

Always use conventional commits format when creating commits. Follow this structure:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files

### Scope

The scope should indicate the package or area affected (e.g., `emitter`).

### Examples

```
feat(emitter): add support for video endpoints
fix(emitter): correct enum generation for renamed values
docs(README): update installation instructions
ci(release): configure automated release workflow
```

### Breaking Changes

If a commit introduces a breaking change, add `BREAKING CHANGE:` in the footer or append `!` after the type/scope:

```
feat(identity-emitter)!: change output format for models

BREAKING CHANGE: The emitter now generates TypeScript interfaces instead of types
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BinkyLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
