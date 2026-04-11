# Schorm AI Agent Instructions

## Project Overview
Schorm is a CLI-driven static-site generator for SCORM 2004 e-learning packages. Think "Hugo for SCORM" - it transforms Markdown lessons, YAML quizzes, and Handlebars templates into LMS-ready content.

**Current Status**: Early development (v0.1) - core architecture established, MVP commands implemented.

## Architecture

### Core Pipeline
```
Source Files → Course Model → Template Render → SCORM Manifest → ZIP Package
```

**Key Directories:**
- `src/cli.ts` - Commander.js entry point, registers all commands
- `src/commands/` - Thin command wrappers (init, build, preview, package, validate, new)
- `src/core/` - Business logic (config, course-model, markdown, templates, manifest, media)
- `theme-default/` - Default Handlebars templates + SCORM runtime JS
- `test/commands/` - Integration tests using Vitest + execSync

### Design Principles
1. **Commands delegate to core modules** - Commands in `src/commands/` are orchestrators, real work happens in `src/core/`
2. **Deterministic builds** - Same input files → same output, no side effects
3. **File-first, CLI-first** - Everything lives in files (Markdown, YAML), no database
4. **TypeScript interfaces as contracts** - See `src/core/course-model.ts` for canonical data structures
5. **ES modules** - Uses `.js` extensions in imports (Node16 module resolution)

## Development Workflows

### Running Commands in Development
```powershell
npm run dev -- init my-project    # Run via tsx (no build needed)
npm run build                     # Compile TypeScript to dist/
node bin/schorm build             # Run built CLI
```

### Testing Strategy
- **Integration tests** in `test/commands/` use `execSync` to invoke the CLI binary
- Tests create temporary projects in `test-output/` (cleaned in afterEach)
- Use Vitest: `npm test` (run once), `npm run test:watch` (watch mode)
- Pattern: init project → modify files → run command → assert output

Example test structure:
```typescript
const projectPath = path.join(TEST_DIR, 'my-test');
execSync(`${CLI_PATH} init my-test`, { cwd: TEST_DIR });
// modify course.yml, add content
execSync(`${CLI_PATH} build`, { cwd: projectPath });
expect(fs.existsSync(path.join(projectPath, 'build', 'imsmanifest.xml'))).toBe(true);
```

## Key File Patterns

### Course Model (`src/core/course-model.ts`)
Central data structures: `Course`, `Module`, `Lesson`, `Quiz`, `Question`
- Load from YAML with `loadCourse(coursePath)`
- All content items must have `id` and `title`
- Modules contain `items[]` array referencing lesson/quiz IDs

### Config System (`src/core/config.ts`)
- `schorm.config.yml` sets `scorm_version` and `theme` path
- Defaults: `scorm_version: '2004-4th'`, `theme: 'theme'`
- Config loaded at start of every build command

### Markdown Processing (`src/core/markdown.ts`)
- Uses `gray-matter` for frontmatter + `markdown-it` for rendering
- Frontmatter must include: `id`, `title`, optionally `module`, `duration`, `objectives[]`
- `parseLesson()` returns a `Lesson` object with HTML content
- `findLessons()` recursively scans `content/` directory

### Template Engine (`src/core/templates.ts`)
- Handlebars-based, wrapped in `TemplateEngine` class
- Load partials from `theme/partials/` via `loadPartials()`
- Layouts in `theme/layouts/` (base.html, lesson.html, quiz.html)
- Custom helpers registered in `registerHelpers()` (currently just `uppercase`)

### SCORM Manifest (`src/core/manifest.ts`)
- Generates `imsmanifest.xml` using `xmlbuilder2`
- `generateManifest(options)` takes `organizations[]` and `resources[]`
- Organizations = hierarchical structure (modules/items)
- Resources = physical files (HTML, media, assets)

## Common Patterns

### Adding a New Command
1. Create `src/commands/your-command.ts`
2. Export a Commander Command object: `export const yourCommand = new Command('your-command')`
3. Import and register in `src/cli.ts`: `program.addCommand(yourCommand)`
4. Delegate logic to `src/core/` modules
5. Add integration tests in `test/commands/your-command.test.ts`

### Working with Paths
- Use `path.resolve()` for absolute paths from CLI arguments
- Theme location: `path.resolve(config.theme)`
- Build output: Always configurable via `--output` option (default: `'build'`)
- Media files: Expect `media/` in project root

### Error Handling
- Throw descriptive errors from core modules (e.g., `throw new Error('Course must have an id')`)
- Commands should catch and log errors clearly
- Config/course validation happens at load time, not during render

## SCORM-Specific Knowledge

### Minimal SCORM 2004 Subset
Focus on these data model elements:
- `cmi.completion_status`, `cmi.success_status`, `cmi.score.scaled`
- `cmi.location`, `cmi.suspend_data`, `cmi.progress_measure`

### Runtime JS (`theme-default/assets/schorm-runtime.js`)
- Handles SCORM API discovery (searches for `window.API_1484_11`)
- Provides `SchormRuntime` object with methods: `init()`, `setValue()`, `getValue()`, `commit()`, `terminate()`
- Must be copied to build output as static asset
- Preview server injects mock SCORM API for local testing

## File Conventions

### Import Extensions
Always use `.js` extensions in TypeScript imports due to Node16 module resolution:
```typescript
import { loadConfig } from '../core/config.js';  // ✓ Correct
import { loadConfig } from '../core/config';     // ✗ Wrong
```

### YAML Structure
- `course.yml`: Course metadata + modules list
- `schorm.config.yml`: Build configuration
- Quizzes: Individual YAML files in `quizzes/` directory
- Use `yaml` package for parsing: `yaml.parse(content)`

### Build Output
Always follows this structure:
```
build/
  *.html              # Lesson/quiz HTML files
  imsmanifest.xml     # SCORM manifest
  assets/             # Theme CSS + runtime JS
  media/              # Copied media files
```

## References
- Architecture details: `docs/architecture.md`
- Product requirements: `docs/prd.md`
- Example tests: `test/commands/build.test.ts` (shows full workflow)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/splittist)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/splittist)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
