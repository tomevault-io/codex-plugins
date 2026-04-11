# Copilot Instructions for DatoCMS Structured Text to HTML String

This repository contains a PHP port of the DatoCMS Structured Text to HTML renderer. The library converts DAST (DatoCMS Abstract Syntax Tree) documents into HTML strings.

## Requirements

- **PHP 8.2+**
- **PHPUnit 11.5+**
- Composer

## Build & Test Commands

```bash
# Install dependencies
composer install

# Run all tests
./vendor/bin/phpunit

# Run tests with verbose output
./vendor/bin/phpunit --testdox

# Run a single test
./vendor/bin/phpunit --filter testRenderNullValue

# Run tests with coverage
./vendor/bin/phpunit --coverage-text

# Run examples
php examples/basic.php
php examples/custom_rendering.php
php examples/blocks_and_links.php
```

## Architecture Overview

### Component Layers

1. **Renderer** (`Renderer.php`) - Static class, main entry point
   - Orchestrates the entire rendering process
   - Handles blocks, inline blocks, inline items, and item links
   - Performs record lookup from DatoCMS GraphQL responses
   - Creates custom node rules for special node types

2. **GenericHtmlRenderer** (`GenericHtmlRenderer.php`) - Core engine
   - Traverses the document tree recursively
   - Applies custom node/mark rules during traversal
   - Handles default rendering for standard DAST nodes
   - Manages the `children` vs `content` key variation in DAST

3. **Adapter Pattern** (`Adapter.php`, `DefaultAdapter.php`)
   - Abstracts HTML generation
   - Default implementation uses simple string concatenation with escaping
   - Supports custom adapters via dependency injection

### Key Data Flow

```
DAST Document → Renderer::render() 
              → Creates custom rules for blocks/links
              → GenericHtmlRenderer (with rules)
              → Adapter (HTML generation)
              → HTML String
```

### Node Types Hierarchy

- **Container nodes**: `root`, `paragraph`, `heading`, `list`, `listItem`, `blockquote`
- **Leaf nodes**: `span`, `link`, `thematicBreak`, `code`
- **Special nodes**: `block`, `inlineBlock`, `inlineItem`, `itemLink` (require custom renderers)

## Coding Conventions

### DealNews-Specific Standards

- **Class-based API only** - No bare functions allowed
- **Protected visibility** - Use `protected` instead of `private` for extensibility
- **snake_case variables** - All variables and properties (not camelCase)
- **1TBS bracing** - Opening brace on same line except multi-line conditionals
- **Single return point** - Prefer one return statement (early validation returns OK)
- **Complete PHPDoc** - All public classes/methods must have full documentation
- **Type declarations** - Required on all function/method signatures

Example:
```php
public function renderNode(array $node, int $key): ?string {
    $result = null;
    
    if ($this->shouldSkip($node)) {
        return $result;  // Early return OK for validation
    }
    
    // Main logic sets $result
    $result = $this->processNode($node, $key);
    
    return $result;  // Single return point
}
```

### Document Tree Traversal

The renderer handles two DAST variations:
- **Modern DAST**: Uses `children` key for child nodes
- **Legacy DAST**: Uses `content` key for paragraphs/headings

Always check both keys via `getChildrenKey()` method.

### Custom Rules Pattern

Custom rules follow a predicate/renderer pattern:

```php
RenderRule::forNode(
    function($node) { return Utils::isHeading($node); },  // Predicate
    function($context) {                                   // Renderer
        // Context keys: node, children, key, adapter
        return $context['adapter']->renderNode(...);
    }
)
```

### Error Handling

Throw `RenderError` (not generic exceptions) for rendering failures:
- Missing required renderers (blocks, inline items, etc.)
- Missing records in links/blocks arrays
- Invalid document structure

Always capture the problematic node in the error:
```php
throw new RenderError("Message describing issue", $node);
```

### Testing Patterns

- Tests verify identical output to JavaScript version
- Use arrays for DAST documents (not objects)
- Test both simple nodes and complex GraphQL responses
- Include error case testing (missing renderers/records)

## Project-Specific Patterns

### The "Wrapper Unwrapping" Pattern

GraphQL responses wrap the document in a `value` key:
```php
['value' => ['schema' => 'dast', 'document' => [...]]]
```

The renderer automatically unwraps this via `render()` method checks.

### Record Lookup Strategy

For special nodes (`block`, `inlineItem`, etc.):
1. Check if custom renderer is provided (throw if missing)
2. Verify GraphQL response structure (throw if invalid)
3. Loop through appropriate array (`blocks`, `links`, `inlineBlocks`)
4. Match by `id` field
5. Throw `RenderError` if record not found

### Mark Rendering Chain

Text marks wrap content in layers:
```php
"Hello" + ['strong', 'emphasis'] 
→ <em><strong>Hello</strong></em>
```

Applied in reverse order (last mark wraps first).

### Newline Handling

Spans with `\n` characters split into multiple text nodes with `<br />` between:
```php
"Line 1\nLine 2" → "Line 1" + <br /> + "Line 2"
```

## Common Tasks

### Adding a New Node Type

1. Add case to `renderNode()` switch in `GenericHtmlRenderer.php`
2. Add type guard to `Utils.php` (e.g., `isMyNode()`)
3. Add test cases in `RenderTest.php`
4. Document in README.md

### Adding a New Mark Type

1. Add to `DEFAULT_MARKS` constant in `GenericHtmlRenderer.php`
2. Add test case with the mark
3. Document in README.md

### Debugging Rendering Issues

1. Check `getChildrenKey()` - are you looking at the right key?
2. Verify node type guards - does `isX()` match correctly?
3. Enable PHPUnit `--testdox` for clear test output
4. Use `var_export($node, true)` to inspect node structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dealnews)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dealnews)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
