# PackDev - GitHub Copilot Instructions

This project is **PackDev**, a modern TypeScript CLI tool that replaces `npm link`, `yarn link`, and manual git dependency management for Node.js package development.

## Project Context

PackDev is a CLI tool written in TypeScript that helps developers manage local and git dependencies during package development. It allows seamless switching between published packages, local file paths, and git repositories without global state or manual package.json editing.

## Technology Stack & Standards

We use TypeScript with strict configuration targeting ES2022 and CommonJS modules. Our build system uses the TypeScript compiler (tsc) with source maps and declarations. We follow these specific conventions:

- **TypeScript**: Strict mode enabled with comprehensive compiler options including `noImplicitAny`, `noUnusedLocals`, `exactOptionalPropertyTypes`
- **Module System**: CommonJS with Node.js resolution
- **CLI Framework**: Commander.js for command-line interface
- **Testing**: Custom Node.js test runner (no external test framework like Jest or Mocha)
- **Code Style**: Use semicolons, prefer `const` over `let`, use async/await over promises
- **Error Handling**: Always return success/error objects with descriptive messages rather than throwing exceptions
- **Branding**: Always use "PackDev" (capital P, capital D) in documentation and user-facing text, but the CLI binary is `packdev` (lowercase)

## Project Structure

The main source files are in `src/`:
- `src/index.ts` - CLI entry point with Commander.js setup
- `src/packageManager.ts` - Core business logic for dependency management
- `src/utils.ts` - Shared utility functions

Tests are in `test/` directory using custom test runners, not standard frameworks.

## Core Concepts

PackDev manages dependencies through a `.packdev.json` configuration file that tracks:
- **Local dependencies**: File system paths to local packages
- **Git dependencies**: Git repository URLs (supports HTTPS, SSH, GitHub/GitLab shorthand)
- **Original versions**: Backup of npm registry versions for restoration

The tool operates in two modes:
- **Development mode** (`packdev init`): Replaces package.json versions with local paths or git URLs
- **Production mode** (`packdev finish`): Restores original npm registry versions

## CLI Commands

All CLI commands should support both local file paths and git repository URLs. When generating code or documentation:
- Use "local and git dependencies" not just "local dependencies"
- Support both `file:../path` and `git+https://` URL formats
- Include examples for both local and git workflows

## File Management Patterns

When working with file operations, always:
- Use `fs/promises` for asynchronous file operations
- Check file existence before reading/writing
- Validate JSON structure when parsing configuration files
- Create backup files before modifying package.json
- Use relative paths and validate they exist for local dependencies

## Git Integration

PackDev includes git hooks functionality that prevents accidental commits of development dependencies. Git hooks are installed in `.git/hooks/` (not `.github/hooks/`). Always reference the correct git hooks directory in code and documentation.

## Testing Approach

Use the custom test runner in `test/` directory. Tests should:
- Use simple Node.js assert module
- Run actual CLI commands to test integration
- Validate both success and error scenarios
- Test both local file and git URL workflows
- Check that package.json changes are properly applied and restored

## Documentation Standards

When writing or updating documentation:
- Always use "PackDev" branding (capital P, capital D)
- Include examples for both local file paths and git repositories
- Show practical scenarios with real-world use cases
- Include troubleshooting sections for common issues
- Reference the correct file paths (`.git/hooks/` not `.github/hooks/`)

## Package Manager Support

PackDev detects and works with npm, yarn, and pnpm. When generating package management code:
- Auto-detect the package manager from lock files
- Support `npm install`, `yarn install`, and `pnpm install`
- Handle different package.json dependency sections (dependencies, devDependencies, peerDependencies)

## Error Handling Best Practices

Always return structured result objects instead of throwing exceptions:
```typescript
interface Result {
  success: boolean;
  error?: string;
  // other result data...
}
```

Include descriptive error messages that help users understand what went wrong and how to fix it.

## Git URL Support

PackDev supports multiple git URL formats:
- HTTPS: `https://github.com/user/repo.git`
- SSH: `git@github.com:user/repo.git`
- GitHub shorthand: `user/repo`
- GitLab shorthand: `gitlab:user/repo`
- With branches/tags: `user/repo#branch-name`
- With commits: `user/repo#commit-hash`

When working with git URLs, always validate the format and provide helpful error messages for invalid URLs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dionmaicon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
