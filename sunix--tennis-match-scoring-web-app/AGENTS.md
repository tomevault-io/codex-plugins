# Copilot Instructions

## Running Tests

This project uses [Vitest](https://vitest.dev/) for testing.

### Install dependencies

```bash
npm install
```

### Run all tests

```bash
npm test
```

This runs `vitest run`, which executes all test files once and exits.

### Run tests in watch mode (during development)

```bash
npx vitest
```

### Run the build

```bash
npm run build
```

This runs TypeScript compilation (`tsc -b`) followed by the Vite production build.

### Run the linter

```bash
npm run lint
```

## CI

Every pull request targeting `main` triggers the **CI** workflow (`.github/workflows/ci.yml`), which runs `npm run build` and `npm test` as required checks before merging.

---
> Source: [sunix/Tennis-Match-Scoring-Web-App](https://github.com/sunix/Tennis-Match-Scoring-Web-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
