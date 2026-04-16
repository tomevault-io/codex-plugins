# Build & Run Workflow

This project uses TypeScript for the canonicalizer script and its libraries.

**Critical:** You **MUST** build the TypeScript source before running the script or tests.

## Build Command

Use `pnpm build:canon` to compile the relevant TypeScript files (`lib/**/*.ts`, `scripts/**/*.ts`) using [tsconfig.canon.json](mdc:tsconfig.canon.json) into the `dist/canon/` directory.

```bash
pnpm build:canon
```

## Running the Script

Always execute the *compiled* JavaScript version:

```bash
node dist/canon/scripts/canonicalize.js [options]
```

Refer to the script's help (`--help`) or the main [README.md](mdc:README.md) for available options.

## Running Tests

Use the project's test script, which typically runs `vitest`:

```bash
pnpm test
```


This command implicitly relies on the code having been built correctly beforehand using `pnpm build:canon`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsallc)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tsallc)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
