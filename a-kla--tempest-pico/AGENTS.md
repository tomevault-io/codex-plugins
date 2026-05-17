# Tempest Pico — Copilot Instructions

## Quick Start

**Tech Stack**: PHP 8.4, Tempest Framework (v2), HtmlViewTree, UnoCSS, Pico CSS, PHPUnit, PHPStan (level 7)

**Yohn's Fork of PicoCSS is used** - see `https://yohn.github.io/PicoCSS/` and `https://github.com/Yohn/PicoCSS/`

**HtmlViewTree**: a alternative to string-based template engines, see **Html View Tree Builder** below.

**QA Workflow**:
```bash
composer qa           # fmt + phpunit + lint (use this before PRs)
composer phpunit      # Run tests with detailed output
composer fmt          # Format with mago
composer lint         # Lint with mago (auto-fixes)
pnpm unocss --watch   # Watch CSS during development
```

**Key Namespaces**:
- `TempestPico\` → `/src/` (main application)
- `Tests\` → `/tests/` (PHPUnit tests)

**PHPStan**: Level 7 strict checking on `src/`, `app/`, `tests/`

---

## Type Hints and common abbreviations

- `MD` → Markdown (GitHub flavored), helper: `MD()`
- `IMD` → Inline Markdown (no Block elements like `<p>`), helper: `IMD()`
- If you see `MD`/`IMD` as Type hint it means the corresponding helper is used on the given `string`   
- (Component-)`Content` one of Type:
  - `string|Stringable`  →  auto-escaped for safe output in HTML
  - `HtmlString`  →  raw HTML, must be used with care -> use `Html(…)` helper instead
  - `Component`  →  any class implementing `IsComponent` (renders to HTML via `getViewTree()`)
  - `View`  →  ! not implemented ! - any other Tempest View (should work - with Issues)
- `VT`/`HtmlViewTree`  →  A Tree of the above types, use `VT()` or `Html(…)` helper to create it


---

## Project Principles

### 1. **Semantic HTML/CSS First**
- Prefer semantic HTML elements over generic divs
- Use Pico CSS utilities + UnoCSS for modifiers (BEM workflow: B for semantic, M/E for utilities)
- Avoid Tailwind-style utility-heavy markup; maintain readability
- Prefer HtmL View Tree Builder over template strings

### 2. **IDE & Static Analysis Clarity**
- All template variables must be IDE-discoverable (no `$$key` magic, explicit type hints)
- Property promoted constructors for class dependency injection
- PHPStan level 7: catches undefined classes, type errors, null access issues
- Common issue: **undefined exception classes** → Always create Exception classes in `Exception/`

### 3. **Component Architecture**
- Components live in `src/Components/` with a `.php` (logic and a `getViewTree()`) and `.view.php` (that start rendering) pair
- Components should also provide:
  - a example returning `VT()` or `Html()` (`src/Components/Examples/*.php`)
  - a description / usage notes (`src/Components/Examples/*.md`)
- Extend `Component` base class and implement `IsComponent` interface

### 4. **Html View Tree Builder**
- **Purpose**: Build HTML trees programmatically without template strings
- **Key Methods**:
  - `__invoke(tag, content[], attributes[])` → Create HTML element (validates tag)
  - `customTag(tag, content[], attributes[])` → Custom tag (no validation)
  - `appendContent()` → Add children to current node
  - `render()` → Convert tree to `HtmlString`
- **Common Patterns**:
  ```php
  Html('div')
    ('h1', ['Headline']);
  ```
- **Known Pitfalls**:
  - **Void tags** (`<br>`, `<hr>`, etc.) cannot have content → throws `VoidWithContent` exception

---

## Common Tasks

### Add a New Component

0. use `src/Components/Accordion.php` and `src/Components/Table.php` as reference
1. Create `src/Components/*.php` (logic + dependencies)
2. Create `src/Components/*.view.php` (only needs: `<?= $this->toHtml();`)
3. Implement `IsComponent` and extend `Component`
4. Use it in other components or controllers
5. Add the `class::name` to `src/Components/Doc.php::COMPONENTS`

### Add Examples, short Documentation and Tests for a Component

- Place usage examples and a additional Notes (`*.md` file) in `src/Components/Examples`
- The Notes don't need more examples or be long. Just known Bugs or a "Read more @"-Link is fine.
- Use the example in test at `test/`.

### Add a Helper Function
- Place HtmlViewTree Helpers in `src/Support/Html/functions.php` (auto-loaded via composer.json)
- Example: `Html()` is aliased to `HtmlViewTree` constructor
- Always import with `use function TempestPico\Support\...`

### Run Tests Before PR
```bash
composer qa
```
This ensures:
- Code is formatted (mago fmt)
- All tests pass (PHPUnit)
- Linting passes (mago lint --fix)

### Debug HtmlViewTree Issues
- Use `render()` return value type (must be `HtmlViewTree`)
- Verify void tags never get `appendContent()` called
- PHPStan will catch missing exception definitions immediately

---

## Architecture Overview

```
src/
  Components/         # Reusable UI components
    *.php             # Logic
    *.view.php        # Render (returns HtmlString)
    Examples/         
      *.php           # Example (used in Tests)
      *.md            # Notes / Documentation
  Layout/             # Page layout (header, nav, footer) and configs
    Page/             # Base Html for pages
  Page/               # Route handlers / page builders / Controllers
  Support/
    Html/             # HTML View Tree Builder, Helper
      Exception/      # Custom exceptions (InvalidTag, VoidWithContent, etc.)
      functions.php   # Helper functions / Shortcuts
tests/                # Test code (PhpUnit)
```

---

## Key Files & References

- [GitHub Page](https://a-kla.github.io/tempest-pico/doc/) the final generated Documentation of this project.
- [Tempest Framework v2](https://tempestphp.com/2.x)  —  Tempest Features
- shell command `./tempest --help` shows Tempest commands
- [Yohn's PicoCSS Fork](https://github.com/Yohn/PicoCSS?tab=readme-ov-file)  —  Styled HTML elements and CSS Classes
- [Accordion](../../src/Components/Accordion.php) and [Table](../../src/Components/Table.php) — Example for Components
- [README.md](../../README.md) — Project rationale & deployment
- [phpstan.neon](../../phpstan.neon) — Static analysis config (level 7)
- [composer.json](../../composer.json) — QA scripts & autoloading

---

## PHPStan Common Fixes

### ❌ "Undefined type 'TempestPico\Support\Html\Exception\InvalidTag'"
**Fix**: Create the exception class in `src/Support/Html/Exception/InvalidTag.php`:
```php
<?php declare(strict_types=1);
namespace TempestPico\Support\Html\Exception;

class InvalidTag extends HtmlBuilderException implements HasContext {
    public function __construct(private string $tag) {
        parent::__construct("The HTML tag {$tag} is unknown.");
    }
    public function context(): array {
        return ['tag' => $this->tag];
    }
}
```

Then import it:
```php
use TempestPico\Support\Html\Exception\InvalidTag;
```

### ❌ "Cannot access offset on mixed"
**Fix**: Add explicit array type hints:
```php
/** @var array<string, string> $config */
$config = $arr->toArray();
```

---

## PHPUnit & Testing

- All tests inherit from `IntegrationTestCase` (unless unit-only)
- Test file naming: `*Test.php` in `/tests/`
- Run with: `composer phpunit`

---

## Deployment & Static Generation

```bash
# Generate static HTML (GitHub Pages)
./tempest static:generate --verbose
pnpm unocss

# Clean after deploy
./tempest static:clean --force
```

---

## Git Workflow

**Current Branch**: `POC--View-Tree-Builder` → PR #1 (View Tree Builder)

Before pushing:
1. Run `composer qa` (must pass)
2. Verify no PHPStan errors: `vendor/bin/phpstan analyse`
3. Commit with short meaningful message using a [Git Emoji](https://gitmoji.dev/)

---

## Tips for AI Agents

- **Run `./tempest routes --json` if you need Data about amiable Pages / Routes** - JSON formatted
- **Short Doc Blocks** — Don't repeat yourself or simple PHP code, only add `@param` etc. if it improve type hinting 
- **Always run `composer qa` after changes** to catch formatting, test, and lint issues
- **PHPStan level 7 is strict** — it will catch undefined classes, missing imports, type mismatches
- **Semantic HTML over generic divs** — review existing Components for patterns
- **HTML View Tree Builder is still evolving** — check tests for expected behavior before guessing
- **Make use of Tempest Helpers** — check `vendor/tempest/framework/packages/support/src/*/functions.php`
- **Don't create loose Exception files** — always implement `HasContext`

---
> Source: [a-kla/tempest-pico](https://github.com/a-kla/tempest-pico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
