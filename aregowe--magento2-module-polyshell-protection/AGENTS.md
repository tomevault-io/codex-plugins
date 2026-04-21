# Copilot Instructions — Aregowe_PolyShellProtection

## Project Identity

This is `Aregowe_PolyShellProtection`, a **high-risk ecommerce security module** for Adobe Commerce / Magento Open Source. It closes the PolyShell unrestricted file upload vulnerability (APSB25-94) via defense-in-depth: eight layered Magento plugins and three security models that block polyglot file attacks and webshell uploads across multiple interception points.

**Treat every change as a production security patch.** Correctness is more important than speed. A missed edge case can result in remote code execution on a merchant's server.

---

## Mandatory Review-First Collaboration Protocol

**Do not write code without explicit approval.** For every issue or recommendation:

1. Explain the concrete tradeoffs.
2. Give an opinionated recommendation.
3. Ask for input before assuming a direction.

### Per-Issue Response Format

For every specific issue (bug, smell, design concern, or risk):

- Describe the problem concretely, with file and line references.
- Present 2–3 options, including "Do Nothing" where reasonable.
- For each option: implementation effort, risk, impact on other code, maintenance burden.
- Give a recommended option and why, mapped to the engineering preferences below.
- Explicitly ask whether the user agrees or wants a different direction before proceeding.

### Review Workflow

When asked to review, work through sections interactively (Architecture → Code Quality → Tests → Performance), pausing after each section. Number issues, use lettered options per issue, and label recommendations clearly.

---

## Engineering Preferences

These guide all recommendations and code generation:

- **DRY is important.** Flag repetition aggressively. Shared constants already live in `FileUploadGuard` (e.g., `ALLOWED_IMAGE_EXTENSIONS`, `MIME_EXTENSION_MAP`, `BLOCKED_EXTENSION_PATTERN`, `BASE_ALLOWED_EXTENSIONS`, `BASE_BLOCKED_EXTENSIONS`). Shared utility methods like `FileUploadGuard::inferExtensionFromMimeType()`, `FileUploadGuard::inferExtensionForFileName()`, `FileUploadGuard::getAllowedExtensions()`, and `FileUploadGuard::getBlockedExtensions()` centralize cross-plugin logic. Never duplicate them.
- **Follow modern code standards** for PHP 8.1-compatible code, XML, JSON, and Adobe Commerce conventions.
- **Well-tested code is non-negotiable.** Favor more tests over fewer. Every public method, every edge case, every failure path.
- **Keep code engineered enough.** Avoid under-engineered fragility and over-engineered premature abstraction.
- **Err on the side of handling more edge cases, not fewer.** Thoughtfulness over speed.
- **Bias toward explicit over clever.** No magic, no implicit behavior, no "cute" one-liners that sacrifice readability.
- **Prefer minimal, targeted changes** with strong backward compatibility.

---

## Architecture Overview

### Defensive Layers

The module implements four defensive layers, each blocking attacks at a different depth:

| Layer | Purpose | Plugins | Sort Order |
|-------|---------|---------|------------|
| 1 — Request Path Blocking | Block HTTP requests to vulnerable media paths before routing | `BlockSuspiciousMediaPathPlugin`, `BlockSuspiciousMediaAppPathPlugin` | 5 |
| 2 — Controller & Upload Blocking | Block file upload controllers and file processor methods for dangerous entity types | `BlockCustomerAttributeFileUploadControllerPlugin`, `BlockCustomerFileUploadPlugin` | 1 |
| 3 — Custom Option Upload Validation | Filename validation and deep content scanning on custom option file uploads via REST API | `ValidateUploadedFileNamePlugin`, `ValidateUploadedFileContentPlugin`, `ValidateCustomOptionUploadPlugin` | 5, 10 |
| 4 — Framework-Level Image Hardening | Extension allowlisting, double-extension detection, polyglot scanning at the framework image validator/processor | `HardenImageContentValidatorPlugin`, `HardenImageProcessorPlugin` | 10 |

### Model Dependency Graph

```
FileUploadGuard (orchestrator)
├── PolyglotFileDetector (image + embedded PHP detection)
├── AttackPatternDetector (known filenames / regex patterns)
├── ScopeConfigInterface (admin-configured extension allowlist/blocklist)
│
└── Used by: ValidateUploadedFileNamePlugin,
             ValidateUploadedFileContentPlugin,
             ValidateCustomOptionUploadPlugin,
             HardenImageContentValidatorPlugin,
             HardenImageProcessorPlugin

SecurityPathGuard (path prefix blocking)
└── Used by: BlockSuspiciousMediaPathPlugin,
             BlockSuspiciousMediaAppPathPlugin

SecurityLogSanitizer (log context sanitization)
└── Used by: all plugins that log
```

### Fail-Closed vs Fail-Open Decisions

- **Path blocking plugins:** Fail-closed. If reflection or detection fails, the request is blocked.
- **Media app path plugin:** Fail-open with warning log. If reflection on `Media::$relativeFileName` fails, the request passes but a warning is logged. Other layers enforce restrictions.
- **Image processor reflection:** Fail-open with warning log. If locking uploader extensions via reflection fails, the request passes but other layers enforce restrictions.

**Any change to fail-closed/fail-open behavior requires explicit discussion and approval.**

---

## PHP Coding Standards

### Strict Mode

Every PHP file **must** begin with:

```php
<?php

declare(strict_types=1);
```

No exceptions.

### Namespace and Autoloading

- Root namespace: `Aregowe\PolyShellProtection\`
- PSR-4 autoloading with the root namespace mapped to the module root directory.
- Directory structure must mirror namespace segments exactly.

### Type System

- For **new and modified code**, function parameters should have type declarations wherever the signature is under our control.
- **All** methods should have return type declarations unless a Magento/framework compatibility constraint prevents it.
- **All** class properties must have typed declarations.
- Use union types (`string|null`) or nullable types (`?string`) where appropriate. Prefer `?string` for single-type nullable.
- Do not use `mixed` unless it is truly necessary, justified, and preferably documented inline.
- **Magento plugin/signature compatibility exceptions:** when implementing `before*`, `around*`, `after*`, observers, framework interfaces, or other interception points, preserve the signature shape required for compatibility with the intercepted method/framework contract. Do not add parameter types or narrow types if doing so would break interception compatibility. In those cases, prefer the framework-compatible signature over this default typing preference.

### Constructor Style

Use **traditional constructor assignment**, not constructor promotion:

```php
// CORRECT
private PolyglotFileDetector $polyglotDetector;
private AttackPatternDetector $patternDetector;

public function __construct(
    PolyglotFileDetector $polyglotDetector,
    AttackPatternDetector $patternDetector
) {
    $this->polyglotDetector = $polyglotDetector;
    $this->patternDetector = $patternDetector;
}

// WRONG — do not use constructor promotion
public function __construct(
    private PolyglotFileDetector $polyglotDetector,
    private AttackPatternDetector $patternDetector
) {}
```

### Visibility

- **Prefer `private`** for properties, methods, and constants unless there is a concrete reason for wider visibility.
- Use `public` only for constants that are shared across classes (e.g., `FileUploadGuard::ALLOWED_IMAGE_EXTENSIONS`) and for method signatures required by Magento plugin contracts.
- Do not use `protected` unless a class is explicitly designed for inheritance (currently none are).

### Constants

- Use class constants for fixed values. Group related constants together.
- Public constants for values shared across classes. Private constants for internal use.
- Use hash-map arrays (`['key' => true]`) for O(1) membership lookups instead of flat arrays with `in_array()`.

### Static Properties

- Use `private static` properties only for reflection caching to avoid repeated `ReflectionClass` instantiation:

```php
private static ?\ReflectionProperty $uploaderProperty = null;
```

### No Enums, No Final, No Readonly

The codebase does not use PHP enums, `final` classes, or `readonly` properties. Do not introduce these without discussion.

---

## Magento Plugin Conventions

### Plugin Naming

- File: `Plugin/{Verb}{Target}Plugin.php` (e.g., `BlockSuspiciousMediaPathPlugin`, `ValidateUploadedFileNamePlugin`, `HardenImageProcessorPlugin`)
- DI name: `aregowe_polyshell_{snake_case_descriptor}` (e.g., `aregowe_polyshell_block_suspicious_media_paths`)

### Plugin Method Patterns

- `around{Method}`: Used when the plugin must conditionally prevent the original method from executing (e.g., blocking a request entirely).
- `before{Method}`: Used when the plugin validates or modifies arguments before the original method runs.
- `after{Method}`: Used when the plugin validates the result after the original method completes.

### Sort Order

- Lower sort order = earlier execution. Path blockers use `sortOrder="5"`, controller blockers use `sortOrder="1"`, framework hardeners use `sortOrder="10"`.
- When adding a new plugin, consider where it fits in the defensive layer hierarchy and choose an appropriate sort order.

### Reflection Usage

Some plugins use `\ReflectionClass` and `\ReflectionProperty` to access private Magento framework properties. This is a necessary evil for deep interception. When using reflection:

- Cache `\ReflectionProperty` instances in `private static` properties.
- Always wrap reflection in try/catch with a graceful fallback.
- Log a warning on reflection failure.
- Document the fail-open/fail-closed decision in a comment.

---

## Model Conventions

### Naming

- `Model/{Noun}{Noun}.php` — descriptive compound nouns (e.g., `FileUploadGuard`, `PolyglotFileDetector`, `AttackPatternDetector`, `SecurityLogSanitizer`, `SecurityPathGuard`).

### Responsibilities

- Each model has a single, well-defined responsibility.
- `FileUploadGuard` is the orchestrator: it normalizes filenames, checks extensions, and delegates to `PolyglotFileDetector` and `AttackPatternDetector`.
- Do not add validation logic directly to plugins. Delegate to models.

### Exception Handling

- Throw `Magento\Framework\Exception\InputException` for validation failures (blocked uploads, bad filenames).
- Throw `Magento\Framework\Exception\LocalizedException` for entity-level operation failures.
- Throw `Magento\Framework\App\Request\Http\NotFoundException` for blocked path requests (404 the attacker).
- Never catch and swallow exceptions silently. Always log before re-throwing or returning an error response.

---

## Logging Standards

### Channel

All security events log to the `polyshell_security` channel via `Aregowe\PolyShellProtection\Logger\Logger`, which writes to `var/log/polyshell_security.log`.

### Sanitization

**Always** sanitize log context values through `SecurityLogSanitizer::sanitizeString()` before logging. This removes control characters, collapses whitespace, and truncates to 256 characters. Never log raw user input.

```php
// CORRECT
$this->logger->warning('Blocked upload', [
    'filename' => $this->logSanitizer->sanitizeString($fileName),
    'ip' => $this->remoteAddress->getRemoteAddress(),
]);

// WRONG — raw user input in log context
$this->logger->warning('Blocked upload', ['filename' => $fileName]);
```

### Log Levels

- `warning`: Blocked attacks, suspicious activity, reflection failures that fall back gracefully.
- `error`: Unexpected failures that indicate a bug or misconfiguration.
- `info`: Reserved for future use (e.g., module startup diagnostics).

### Log Message Prefixes

Use `'PolyShellProtection: '` or `'PolyShell guard '` as message prefixes for grep-ability.

---

## Testing Standards

### Framework

- PHPUnit `^10.5` (as declared in `composer.json`).
- All tests extend `PHPUnit\Framework\TestCase`.
- Test location: `Test/Unit/` with directory structure mirroring production code.

### Test File Naming

- `Test/Unit/Model/{ClassName}Test.php`
- `Test/Unit/Plugin/{ClassName}Test.php`
- Standalone validation tests: `Test/Unit/{DescriptiveNameTest}.php`

### Test Method Naming

- `test{DescriptiveBehavior}(): void` — camelCase, starts with `test`, describes what is being tested and the expected outcome:

```php
public function testBlocksUnicodeEscapedPhpExtension(): void
public function testAllowsSafePngName(): void
public function testValidImagePassesThrough(): void
```

### Mocking

- Use **PHPUnit mocks exclusively**: `$this->createMock()`, `$this->getMockBuilder()->disableOriginalConstructor()->getMock()`.
- Do not use Mockery or any other mocking library.
- Use `disableOriginalConstructor()` for classes with complex constructors (e.g., `Logger`).
- Use `createMock()` for interfaces and simple classes.
- For models with no dependencies (e.g., `PolyglotFileDetector`, `AttackPatternDetector`, `SecurityLogSanitizer`), instantiate the real object instead of mocking.

### Mock Property Annotation

Use the dual-type PHPDoc annotation for mock properties:

```php
/** @var Logger|\PHPUnit\Framework\MockObject\MockObject */
private Logger $logger;
```

### setUp() Pattern

```php
protected function setUp(): void
{
    $this->mockDependency = $this->getMockBuilder(SomeClass::class)
        ->disableOriginalConstructor()
        ->getMock();

    $this->subject = new ClassUnderTest(
        $this->mockDependency,
        new RealSimpleDependency()
    );
}
```

### Data Providers

- Use `@dataProvider` for parameterized tests covering multiple inputs.
- Data provider methods are `public static` and return `array<string, array>` with descriptive string keys:

```php
public static function allowedExtensionsProvider(): array
{
    return [
        'jpg'  => ['jpg'],
        'jpeg' => ['jpeg'],
        'gif'  => ['gif'],
    ];
}
```

### Exception Testing

```php
public function testBlocksDisallowedExtension(): void
{
    $this->expectException(InputException::class);
    $this->expectExceptionMessage('expected message fragment');

    $this->subject->methodThatShouldThrow($badInput);
}
```

### Assertion Patterns

- `assertTrue()` / `assertFalse()` for boolean results.
- `assertSame()` for strict equality (preferred over `assertEquals()`).
- `assertInstanceOf()` for type checks.
- `assertSameSize()` for collection length comparisons.
- `$this->once()`, `$this->never()`, `$this->callback()` for mock expectations.

### Test Coverage Requirements

- **Every public method** must have at least one test.
- **Every exception path** must be tested.
- **Every validation branch** (allow and block) must be tested.
- **Edge cases are mandatory:** empty strings, null values, unicode obfuscation, double extensions, no-extension filenames, boundary-length inputs, known attack filenames.
- **Data providers** should cover the full matrix of allowed and blocked values.
- When adding a new attack pattern or validation rule, add tests for both the positive (blocked) and negative (allowed) cases.

---

## Security-Specific Guidelines

### Defense in Depth

Multiple layers intentionally overlap. **Do not remove a "redundant" check** without understanding the full attack surface. A check that seems redundant may be the only layer protecting a specific code path.

### Attack Pattern Updates

When adding new patterns to `AttackPatternDetector::ATTACK_FILENAMES` or `ATTACK_PATTERNS`:

1. Add the pattern with a comment explaining the attack it targets.
2. Add a corresponding test case in `AttackPatternDetectorTest`.
3. Verify the pattern does not false-positive on legitimate filenames (test both block and allow).

### Extension Allowlists and Blocklists

- `FileUploadGuard::BASE_ALLOWED_EXTENSIONS` — base general upload allowlist (documents + images), code-defined.
- `FileUploadGuard::BASE_BLOCKED_EXTENSIONS` — base blocklist of executable/script extensions (including PHP include files `.inc`, PHP source `.phps`, and module files `.module`), code-defined.
- `FileUploadGuard::getAllowedExtensions()` — returns the merged allowlist: base + admin-configured additions, minus any blocked extensions. The blocklist always overrides.
- `FileUploadGuard::getBlockedExtensions()` — returns the merged blocklist: base + admin-configured additions.
- `FileUploadGuard::ALLOWED_IMAGE_EXTENSIONS` — image-only allowlist used by image hardening plugins (not configurable via admin).
- Admin-configured extensions are read from `ScopeConfigInterface` at runtime (`XML_PATH_ADDITIONAL_EXTENSIONS`, `XML_PATH_ADDITIONAL_BLOCKED_EXTENSIONS`).
- These are the single source of truth. **Never hardcode extension lists in plugins.**

### Content Scanning

`PolyglotFileDetector` scans file content for:

1. Image file signatures (magic bytes) — to identify polyglot candidates.
2. PHP code patterns embedded within image files — the actual threat.
3. Known campaign-specific attack signatures.

When adding new scan patterns, ensure they do not trigger on legitimate binary content (e.g., compressed data that coincidentally contains `<?php` bytes).

### Filename Normalization

`FileUploadGuard::normalizeFileName()` is a **private** method that applies multi-pass normalization:

1. Unicode escape decoding (`\u002e` → `.`)
2. Multi-pass URL decoding (`%2e` → `.`)
3. CR/LF/TAB replacement with spaces (other control characters are preserved and rejected by `assertSafeFileName()`)
4. Whitespace collapse

Plugins must not call `normalizeFileName()` directly. Instead, use the public methods `assertSafeFileName()` or `inferExtensionForFileName()`, which normalize internally and enforce safety validation.

**Any filename validation must use the normalized form.** Never validate against the raw input.

---

## Dependency Injection (di.xml) Conventions

- All plugin registrations go in `etc/di.xml`.
- Use descriptive `name` attributes prefixed with `aregowe_polyshell_`.
- Specify `sortOrder` on every plugin declaration.
- Inject dependencies via constructor arguments in `<type>` declarations.
- Use `xsi:type="object"` for class dependencies, `xsi:type="string"` for scalar values, `xsi:type="array"` for collections.

---

## Admin Configuration Conventions

### Config Paths

- Config paths use the `aregowe_polyshell/` section prefix (e.g., `aregowe_polyshell/upload_settings/additional_allowed_extensions`).
- Path constants are defined as public constants on the relevant model (e.g., `FileUploadGuard::XML_PATH_ADDITIONAL_EXTENSIONS`).
- Use `ScopeConfigInterface::getValue()` to read config values at runtime. Cast to `(string)` and handle empty values gracefully.

### ACL

- The admin configuration section is protected by `Aregowe_PolyShellProtection::config`, declared in `etc/acl.xml`.
- New admin config sections must reference this ACL resource (or a child of it).

### system.xml

- All admin-facing settings live in `etc/adminhtml/system.xml`.
- The section ID is `aregowe_polyshell`, placed in the `security` tab.
- Use `<field type="note">` for read-only informational items (e.g., displaying base code-defined extension lists).
- Use `<comment>` with CDATA for HTML formatting.

### config.xml

- Default values for all config fields are declared in `etc/config.xml`.
- Empty-string defaults are expressed as self-closing elements (e.g., `<additional_allowed_extensions/>`).

### Blocklist-Overrides-Allowlist Rule

- The blocked extension list **always** overrides all allowlists. If an extension appears in both, it must be blocked.
- Extensions matching `BLOCKED_EXTENSION_PATTERN` (executable/script patterns) must be rejected regardless of admin config.
- This invariant is enforced in `FileUploadGuard::getAllowedExtensions()` and documented in inline comments.

---

## Composer Conventions

- Package: `aregowe/magento2-module-polyshell-protection`
- Type: `magento2-module`
- PHP constraint: `^8.1`
- Framework constraint: `magento/framework ^103`
- The module declares `"replace": {"markshust/magento2-module-polyshell-patch": "*"}` because it fully supersedes that package. Maintain this replacement declaration.

---

## File Organization Rules

```
/
├── composer.json
├── registration.php
├── README.md
├── etc/
│   ├── module.xml
│   ├── di.xml
│   ├── acl.xml
│   ├── config.xml
│   └── adminhtml/
│       └── system.xml
├── Logger/
│   ├── Logger.php
│   └── Handler/
│       └── SecurityHandler.php
├── Model/
│   ├── AttackPatternDetector.php
│   ├── FileUploadGuard.php
│   ├── PolyglotFileDetector.php
│   ├── SecurityLogSanitizer.php
│   └── SecurityPathGuard.php
├── Plugin/
│   ├── Block*.php          (request/upload blockers)
│   ├── Harden*.php         (framework hardeners)
│   └── Validate*.php       (content/name validators)
└── Test/
    └── Unit/
        ├── Model/          (mirrors Model/)
        └── Plugin/         (mirrors Plugin/)
```

- Every production class gets a corresponding test file.
- Test directory structure mirrors production directory structure.
- No integration or functional tests exist yet; if added, place them in `Test/Integration/` and `Test/Functional/`.

---

## Change Checklist

Before submitting any change, verify:

- [ ] `declare(strict_types=1);` is present in every new/modified PHP file.
- [ ] All parameters, return types, and properties are fully typed.
- [ ] No duplicated constants or extension lists — reference `FileUploadGuard` constants.
- [ ] All user-facing input in log messages is sanitized via `SecurityLogSanitizer`.
- [ ] New validation logic lives in models, not plugins.
- [ ] New plugins are registered in `di.xml` with appropriate sort order.
- [ ] Fail-closed/fail-open behavior is explicitly documented in comments.
- [ ] Unit tests cover all new public methods, all exception paths, and edge cases.
- [ ] Data providers are used for parameterized test cases with descriptive keys.
- [ ] No `protected` visibility unless justified by a concrete inheritance need.
- [ ] No constructor promotion — use traditional assignment.
- [ ] Backward compatibility is preserved; no breaking changes to public constants or method signatures.
- [ ] `composer.json` constraints are satisfied (`php ^8.1`, `magento/framework ^103`).
- [ ] PHPUnit tests pass: `vendor/bin/phpunit Test/`

---

## Anti-Patterns — Do Not

- **Do not bypass validation layers.** Even if a check seems redundant, it serves a defense-in-depth purpose.
- **Do not log unsanitized user input.** Always use `SecurityLogSanitizer`.
- **Do not hardcode extension lists in plugins.** Use `FileUploadGuard` constants.
- **Do not use `in_array()` for membership checks on large lists.** Use hash-map arrays with `isset()`.
- **Do not swallow exceptions.** Log and re-throw or return an appropriate error.
- **Do not use `mixed` types** without justification.
- **Do not introduce new dependencies** (Mockery, external libraries) without discussion.
- **Do not use constructor promotion** — the codebase uses traditional assignment consistently.
- **Do not use `final`, `readonly`, or `enum`** without explicit approval.
- **Do not assume a single layer will catch an attack.** Always consider what happens if your layer is the only one that fires.

---
> Source: [aregowe/magento2-module-polyshell-protection](https://github.com/aregowe/magento2-module-polyshell-protection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-21 -->
