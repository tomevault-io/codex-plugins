# Copilot Instructions for Spada Libreria

## Project Overview

Spada Libreria is a platform for studying Bolognese fencing treatises. The application displays historical fencing texts in Italian, French, and English with interactive glossary tooltips.

## Tech Stack

- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Data Format**: YAML files for content (glossary and treatises)
- **Package Manager**: npm (for Node.js), uv (for Python)
- **Python**: 3.13+ (for data extraction scripts)

## Project Structure

```
src/
├── app/           # Next.js App Router pages and API routes
│   ├── api/
│   │   ├── annotations/  # Annotation persistence endpoints
│   │   └── content/      # Content loading endpoints
├── components/    # React components
│   ├── Term.tsx                      # Glossary tooltip component
│   ├── TextParser.tsx                # Text parsing with glossary links
│   ├── BolognesePlatform.tsx         # Main platform component
│   ├── SearchBar.tsx                 # Search interface with options
│   ├── AnnotationPanel.tsx           # Annotation editing panel
│   ├── AnnotationBadge.tsx           # Annotation tag display
│   ├── AnnotationDisplaySettings.tsx # Annotation config UI
│   ├── TagFilter.tsx                 # Tag-based filtering
│   ├── ColorPicker.tsx               # Color selection for annotations
│   ├── StatisticsModal.tsx           # Statistics display modal
│   ├── ComparisonModal.tsx           # Translation comparison modal
│   ├── MeasureProgressBar.tsx        # Visual measure progression
│   └── TextEditor.tsx                # Rich text editor
├── contexts/      # React contexts for state management
│   ├── AnnotationContext.tsx         # Annotation state and persistence
│   ├── AnnotationDisplayContext.tsx  # Annotation display configuration
│   └── SearchContext.tsx             # Search state and index
└── lib/           # Utilities and data loading functions
    ├── dataLoader.ts                 # YAML data loading
    ├── searchEngine.ts               # Search logic
    ├── searchIndex.ts                # Search index construction
    ├── highlighter.ts                # Search term highlighting
    ├── termTypeMapping.ts            # Term type utilities
    ├── localStorage.ts               # Local storage utilities
    └── annotation/                   # Annotation class architecture
        ├── Annotation.ts             # Abstract base class
        ├── AnnotationRegistry.ts     # Factory/registry pattern
        ├── Weapons.ts                # Weapons annotation class
        ├── WeaponType.ts             # Weapon type annotation class
        ├── Guards.ts                 # Guards annotation class
        ├── Techniques.ts             # Techniques annotation class
        ├── Measures.ts               # Measures annotation class
        ├── Strategy.ts               # Strategy annotation class
        ├── Strikes.ts                # Strikes annotation class
        ├── Targets.ts                # Targets annotation class
        └── index.ts                  # Public exports
data/
├── glossary.yaml  # Glossary terms with FR/EN definitions
└── treatises/     # YAML files containing treatise sections
scripts/
├── extract_book.py  # Python script for extracting text from PDFs
└── yaml_annotate.py # Python script for adding annotations and glossary links
```

## Key Architecture Principles

1. **Content/Code Separation**: All content (treatises, glossary) is stored in YAML files in the `data/` directory. Never hardcode content in TypeScript/React files.

2. **Glossary Terms**: Terms in treatise text are marked with `{term_name}` syntax and are automatically linked to glossary entries.

3. **Multi-language Support**: The application supports Italian (original), French, and English translations. English supports multiple translator versions.

4. **Search System**: The application includes a cross-treatise search system with:
   - **SearchIndex**: Built at application load from all treatise sections
   - **SearchEngine**: Supports Match Case, Match Whole Word, and Regex options
   - **Highlighter**: Real-time highlighting of search terms in text
   - **SearchContext**: Manages search state and results across the application

5. **Annotation Class Architecture**: Annotations use an object-oriented architecture:
   - **Annotation (abstract)**: Base class defining common interface
   - **AnnotationRegistry**: Factory/registry pattern providing singleton instances
   - **9 Concrete Classes**: Weapons, WeaponType, Guards, Techniques, Measures, Strategy, Strikes, Targets, Note
   - Each class encapsulates its own styling (`getChipStyle()`, `getTextStyle()`) and validation logic
   - Used by `ColorPicker`, `AnnotationPanel`, and `AnnotationDisplaySettings` components

6. **LocalStorage Utility**: All browser storage operations must use the centralized `@/lib/localStorage` utility:
   - **NEVER** use `localStorage` or `window.localStorage` directly
   - **ALWAYS** use `LocalStorage.getItem<T>()` and `LocalStorage.setItem()` from the utility
   - The utility provides automatic error handling, type safety, and backward compatibility
   - All stored data is wrapped in a `StorageItem<T>` structure with timestamp metadata
   - Storage operations gracefully handle quota exceeded, parse errors, and unavailability

## Commands

### Node.js / Next.js
```bash
npm install    # Install dependencies
npm run dev    # Start development server at http://localhost:3000
npm run build  # Build for production
npm run lint   # Run ESLint
```

### Python Scripts
```bash
uv sync                    # Synchronize Python dependencies
uv add <package>           # Add a new Python dependency
uv run scripts/extract_book.py   # Run the extraction script
uv run scripts/yaml_annotate.py  # Run the annotation script
```

## Code Style Guidelines

### TypeScript / React
- Use TypeScript with proper type definitions
- Use `'use client'` directive for client-side components
- Use Tailwind CSS for styling (no CSS modules)
- Follow React functional component patterns with hooks
- Use absolute imports with `@/` prefix (e.g., `@/lib/dataLoader`)
- **Storage**: Use `LocalStorage` utility from `@/lib/localStorage` (never direct `localStorage` access)
- **Null Safety**: Always provide default values with nullish coalescing (`??`) for storage reads

### Python
- Use Python 3.13+ features
- Follow PEP 8 style guide
- Use type hints where appropriate
- Use `argparse` for command-line scripts (not `input()`)

## Dependency Management

### **CRITICAL RULE**: When adding a Python dependency:

1. **NEVER** manually edit `pyproject.toml` dependencies
2. **ALWAYS** use: `uv add <package-name>`
3. This ensures proper dependency resolution and lockfile updates

Example:
```bash
# ✅ Correct
uv add pyyaml

# ❌ Wrong
# Manually editing pyproject.toml
```

### Node.js Dependencies
Use standard npm commands:
```bash
npm install <package>
```

## Data File Formats

### Glossary Entry (`data/glossary.yaml`)

```yaml
term_key:
  term: Display Term
  type: Category
  definition:
    fr: French definition
    en: English definition
  translation:
    fr: French translation
    en: English translation
```

### Treatise Section (`data/treatises/*.yaml`)

```yaml
- id: unique_section_id
  title: Section Title
  metadata:
    master: master_name
    work: Work Name
    book: 1
    chapter: 1
    year: 1536
  content:
    it: Italian original text with {glossary_terms}
    fr: French translation with {glossary_terms}
    en_versions:
      - translator: "Translator Name"
        text: English translation with {glossary_terms}
  annotation:
    id: anno_1234567890_unique
    weapons: [Spada sola]
    weapon_type: "Épée aiguisée"  # or "Épée émoussée"
    guards_mentioned: [Coda Longa e Stretta]
    techniques: [Stringere]
    measures: [Largo, Mezzo]
    strategy: [provocation]
    strikes: [Mandritto, Fendente]
    targets: [Tête, Bras]
    guards_count: {"Coda Longa e Stretta": 2}
    techniques_count: {"Stringere": 1}
    strikes_count: {"Mandritto": 3}
    targets_count: {"Tête": 2}
```

**Important**: The annotation fields (weapons, guards_mentioned, techniques, measures, strategy, strikes, targets, weapon_type, *_count) are in the `annotation` section, NOT in `metadata`. The `metadata` section contains only bibliographic information.

## Python Scripts Guidelines

### Extract Book Script

The `extract_book.py` script extracts text from PDF treatises and should:

1. Use `argparse` for command-line arguments
2. Accept `--pdf` and `--pages` arguments
3. Generate YAML files in the format described above
4. Save to `data/treatises/{author}_{book}.yaml`
5. NOT include `annotation` fields in output

Example usage:
```bash
uv run extract-book marozzo --pages "34-102"
```

## Testing

Jest with React Testing Library is configured for JavaScript/TypeScript. Use pytest for Python scripts.

### Running Tests
```bash
npm test                  # Run all tests
npm test -- --watch       # Watch mode for development
npm test -- --testPathPattern=fileName  # Run specific test file
```

### LocalStorage Testing
- Tests use `jest-localstorage-mock` for localStorage mocking
- `setupTests.js` automatically clears localStorage before each test
- Always test both success and error scenarios (quota exceeded, corrupted data)
- Verify backward compatibility with legacy data formats

## Important Notes

- The project is primarily in French (documentation, comments)
- API routes are in `src/app/api/` and use Next.js App Router conventions
- Data loading uses server-side functions with `fs` and `js-yaml`
- Client components must not import server-side modules like `fs`
- Python environment is managed by `uv`, not pip or conda directly

- **Audience**: This is a learning project for beginners in web technologies (transitioning from C/systems programming)
- **Deployment**: This is a local-only application. It will never be deployed online. All development and usage is on the developer's machine

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Manuel-Fostier)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Manuel-Fostier)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
