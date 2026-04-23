# PDF Diff - AI Agent Instructions

## Project Overview
PDF Diff is a **dual-target codebase**: a privacy-focused web app for comparing PDFs in the browser AND a CLI tool published to npm. All PDF processing happens client-side (web) or locally (CLI) with zero server uploads.

**Tech Stack:** React 19 + TypeScript + Vite (web), Node.js ESM (CLI), PDF.js for rendering, diff library for text comparison.

## Critical Architecture Patterns

### Dual Build System
This project has **two separate build outputs**:
1. **Web App** (`npm run build`): Vite builds React SPA to `dist/`
2. **CLI Tool** (`npm run build:cli`): Custom esbuild script bundles Node CLI to `dist-cli/cli.mjs`

**Key:** CLI code in `src/cli/` shares utility logic but cannot import React components. Web code in `src/` and `src/utils/` is browser-only.

### Code Duplication by Design
`src/utils/diffUtils.ts` and `src/cli/diffUtils.ts` are **intentionally duplicated**. The CLI version has fewer functions (no `filterAdditionsOnly`, `filterRemovalsOnly`) because CLI doesn't need all view modes. Do NOT try to share these via imports - build targets are separate.

Same pattern applies to:
- `src/utils/pdfUtils.ts` (browser) vs `src/cli/pdfUtils.ts` (Node.js with fs)
- Both use PDF.js but different initialization (canvas vs filesystem)

### Version Management
The CLI version is hardcoded in `src/cli/index.ts` as `const VERSION = '1.0.4'`. When bumping versions:
1. Update `package.json` version
2. Update `VERSION` constant in `src/cli/index.ts`
3. Run `npm run build:cli`
4. Commit, tag with `v<version>`, push tag

**Example workflow:**
```bash
npm run build:cli
git add -A
git commit -m "Bump to 1.0.4 - <reason>"
git tag v1.0.4
git push origin main && git push origin v1.0.4
```

## TypeScript Configuration

This is a **project references** setup with 3 configs:
- `tsconfig.app.json`: Web app (DOM types, React)
- `tsconfig.cli.json`: CLI code (Node types, no DOM)
- `tsconfig.node.json`: Build scripts (Node)

Never import DOM APIs in CLI code or Node.js modules in React components.

## Component Architecture

### State Management Pattern
Main app state lives in `App.tsx` with hooks. No Redux/Zustand - intentionally simple for auditable privacy claims.

**PDF Processing Flow:**
1. User drops files → `PDFDropZone` component
2. Files passed to `extractTextFromPDF()` → parses with PDF.js
3. Results stored as `PDFDocument` in state (`originalDoc`, `modifiedDoc`)
4. On render, `useMemo` computes diffs via `computeTextDiff()` from diff library
5. `DiffView` component renders with view mode prop

### View Modes
Five modes controlled by `ViewModeTabs`:
- `side-by-side`: Split view, removals left, additions right
- `unified`: Inline with color coding
- `additions`: Green only
- `removals`: Red only  
- `changes-only`: Side-by-side per page

**Implementation:** Mode passed as prop to `DiffView.tsx` which conditionally renders. Each mode filters/displays `DiffPart[]` differently.

## CLI Design Patterns

### Commander.js Structure
Arguments before options: `pdf-diff <original> <modified> [options]`

Optional interactive mode if no args: `pdf-diff --interactive` uses @inquirer/prompts for UX.

### Report Generation
Three formats (text/json/junit) + visual reports (HTML/PDF):
- Text/JSON/JUnit printed to stdout
- HTML/PDF saved to `--out` directory (default: `./pdf-diff-report`)
- PDF generation uses Playwright's Chromium (auto-installed on first use)

**Key:** `reportGenerator.ts` has functions for each format. HTML report is generated first, then Playwright converts to PDF.

### Exit Codes
- `0`: Success or no differences (unless `--fail-on-diff` with diffs)
- `1`: Differences found with `--fail-on-diff`, or fatal errors

## Development Workflows

### Running Locally
```bash
npm run dev          # Vite dev server at localhost:5173
npm run build        # Production web build
npm run preview      # Preview production build
npm run lint         # ESLint (must pass before commits)
```

### Testing CLI Changes
After editing `src/cli/`, always rebuild:
```bash
npm run build:cli
node dist-cli/cli.mjs test.pdf test2.pdf  # Test locally
```

**Do NOT use `npm link`** - just test the built output directly.

### Publishing to npm
`prepublishOnly` script runs `build:cli` automatically. Only `dist-cli/` is published (see `files` in package.json).

## Privacy-First Conventions

### Never Add Server Communication
This is a **100% client-side/local tool**. Do NOT add:
- API calls (except demo PDF loading from `/public`)
- Analytics beyond what's in HTML
- Cookies for functionality (theme uses localStorage)

### File Handling
Web: Files processed via FileReader API, never uploaded.
CLI: Files read from disk via `fs`, never transmitted.

## Common Pitfall: Import Paths

CLI code requires `.js` extensions in imports because it's ESM:
```typescript
// ✅ Correct in src/cli/
import { extractTextFromPDFFile } from './pdfUtils.js';

// ❌ Wrong - TypeScript will complain or runtime breaks
import { extractTextFromPDFFile } from './pdfUtils';
```

Web code (Vite) does NOT need `.js` extensions.

## Performance Patterns

### Large PDFs
- `extractTextFromPDF()` processes pages sequentially (not parallel) to avoid memory spikes
- CLI spinner feedback during long operations via `ora`
- Web shows "Processing PDFs..." state during extraction

### Show All Pages Mode
Computes diffs for ALL pages at once - can be slow on 100+ page docs. Stats combined via `combineStats()`.

## Styling Convention

CSS modules are NOT used - plain CSS files imported per component. Theme switching via `data-theme` attribute on `<html>`.

Colors: Green (#22c55e) for additions, Red (#ef4444) for deletions. Dark mode inverts with different shades.

## Key Files Reference

- [src/App.tsx](src/App.tsx): Main React app, state management
- [src/cli/index.ts](src/cli/index.ts): CLI entry point, Commander setup
- [src/utils/diffUtils.ts](src/utils/diffUtils.ts): Browser diff utilities
- [src/cli/diffUtils.ts](src/cli/diffUtils.ts): CLI diff utilities (duplicate by design)
- [scripts/build-cli.mjs](scripts/build-cli.mjs): esbuild configuration for CLI
- [package.json](package.json): Defines both web and CLI scripts/dependencies

---
> Source: [jamesmontemagno/pdf-diff](https://github.com/jamesmontemagno/pdf-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-23 -->
