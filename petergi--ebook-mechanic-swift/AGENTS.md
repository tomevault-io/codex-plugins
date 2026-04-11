# Copilot Instructions for EbookMechanic

## Project Overview

**EbookMechanic** is a Swift-native ebook library management toolkit with modular architecture. It validates ebook files (EPUB, MOBI, AZW3, AZW4, PDF), detects corruption, auto-repairs damage, normalizes EPUB structure, and generates reports. The project uses **Swift Package Manager (SPM)** with 5 independent packages and Swift 6 strict concurrency throughout.

## Architecture

**Modular Package Structure** (under `Apps/` and `Packages/`):

1. **EbookMechanicCore** - Core library with scanning/validation/repair logic using Swift actors
2. **EbookMechanicCLI** - Full-featured CLI for all ebook types
3. **EbookMechanicEPUBCLI** - Lightweight EPUB-only CLI
4. **EbookMechanicPDFCLI** - Lightweight PDF/AZW4-only CLI
5. **EbookMechanicApp** - Native macOS SwiftUI app

**Key Design Principles**:
- **Actor-based concurrency** (`FileScanner` is an `actor`) for thread-safe operations without mutex locks
- **No external dependencies** - pure Swift implementation of ZIP handling, format validation, repair logic
- **Single-responsibility separation** - `FileValidator`, `FileRepairer`, `EPUBArchiveNormalizer`, `MarkdownReportGenerator` are distinct types
- **Sendable types everywhere** - models use `Sendable` for safe cross-actor communication

## Critical Implementation Details

**File Format Validation** (`Packages/EbookMechanicCore/Sources/EbookMechanicCore/Validation/`):
- **EPUB**: Valid ZIP + `mimetype` file with exact content `application/epub+zip` + `META-INF/container.xml`
- **MOBI/AZW3**: PalmDB header signature (`BOOKMOBI` or `TEXtREAd` at bytes 60-68)
- **AZW4**: Uses PDF validation (Kindle PDF wrapper)
- **PDF**: Header `%PDF-` AND `%%EOF` marker in last 1KB (detects email/FTP corruption, UTF-8 BOM, junk prefixes)

**Actor-Based Scanning** (`FileScanner.swift`):
- `public actor FileScanner` - automatically enforces Sendable data flow
- Use `progress` handler callback to send `ProgressEvent` during scanning
- Automatically skips `CORRUPTED/` directory to avoid re-scanning moved files
- `scanForCorruption()` and `deleteEmptyFolders()` are async methods

**Auto-Repair** (`Packages/EbookMechanicCore/Sources/EbookMechanicCore/Repair/`):
- **EPUB**: Adds missing `mimetype` and `META-INF/container.xml` if absent
- **PDF**: Repairs header corruption (email wrappers, BOM, junk prefixes); restores `%%EOF` marker
- Always creates `.backup` files before repairs
- Returns `RepairResult` for each file with status and reason

**EPUB Normalization** (`EPUBArchiveNormalizer.swift`):
- Pre-checks if EPUB is already normalized to avoid redundant work
- Normalizes manifest IDs, file extensions (`.htm` → `.xhtml`), and ZIP structure
- Uses `OPFValidator` and `HTMLValidator` for validation
- Follows Sigil ebook editor standards

## Development Workflow

**Build Targets** (from `Makefile.swift`):
```bash
make -f Makefile.swift build-core          # Build core library
make -f Makefile.swift build-cli           # Build main CLI
make -f Makefile.swift build-app           # Build macOS SwiftUI app
make -f Makefile.swift build-specialized   # Build EPUB + PDF specialized CLIs
make -f Makefile.swift build-all           # Build everything
```

**Test Commands**:
```bash
make -f Makefile.swift test-core          # Test core library (26 tests)
make -f Makefile.swift test-cli           # Test CLI (102 tests)
make -f Makefile.swift test-all           # Run all tests
```

**Run/Install**:
```bash
make -f Makefile.swift run-cli ARGS="-dir ~/Books"     # Run CLI
make -f Makefile.swift run-app                         # Launch SwiftUI app
make -f Makefile.swift install                         # Install all CLIs to /usr/local/bin
```

## Adding New Features

**New File Format**:
1. Add case to `EbookFileType` enum in `Models.swift`
2. Create validator function in `FileValidator.swift` (match pattern `validate(epub:)`)
3. Add repair logic to `FileRepairer` if applicable
4. Update tests in `EbookMechanicCoreTests/`

**Extending CLI**:
- All CLIs depend on `EbookMechanicCore` - add logic there, not in CLI
- Use `ArgumentParser` pattern from existing CLIs
- Update shell completions in `completions/` if adding new flags

## Command-Line Interface

The tool supports both TUI and simple modes via flags:

- `-dir` - Root directory to scan (default ".")
- `-corrupted-dir` - Directory for corrupted files (default "CORRUPTED")
- `-repair` - Attempt to repair corrupted files before moving them
- `-normalize-epub` - Normalize EPUB files to Sigil standards (implies -repair)
- `-force-normalize` - Force normalization even if EPUB appears already normalized
- `-keep-backups` - Keep .backup files after successful operations
- `-clean-backups` - Remove existing .backup files in directory
- `-dry-run` - Scan only, no file modifications
- `-corruption-only` - Skip empty folder operations
- `-empty-folders-only` - Skip corruption detection
- `-no-confirm` - Skip deletion confirmation prompts
- `-no-tui` - Simple text output mode

**Enhanced EPUB Normalization Features**

- Scans ALL EPUB files in directory (not just corrupted ones)
- Pre-checks if EPUBs are already normalized to avoid unnecessary work
- Use `-force-normalize` to bypass pre-checks and normalize everything
- Significantly improves performance for large libraries with mixed normalized/unnormalized files
## Testing Patterns

### Unit Tests (Validators & Transformers)

Located in `EbookMechanicCoreTests/`, test individual components in isolation:

**File Validation Tests** (`ValidationTests.swift`):
- Test each format validator independently: `testValidateEPUB()`, `testValidateMOBI()`, etc.
- Use `TestFixtures` helpers to create corrupted variants (missing files, wrong signatures, truncated headers)
- Validate both **positive cases** (valid files pass) and **negative cases** (corruption detected with correct reason)
- Example: `createEPUBWithoutMimetype()`, `createPDFWithoutEOF()` create specific corruption scenarios

**Repair Tests** (`RepairTests.swift`, `EnhancedRepairTests.swift`):
- Test repair logic in isolation: missing EPUB metadata, PDF header corruption, etc.
- Verify `.backup` files are created before repair attempts
- Confirm repaired files pass validation after repair
- Example: Create a PDF without `%%EOF`, repair it, then validate it passes

**Report Generation** (`ReportGeneratorTests.swift`):
- Test markdown report formatting with mock data
- No file I/O needed - pass in `ScanResult` structs directly

### Integration Tests (Actor Workflows)

Located in `ScannerTests.swift`, test multi-step workflows using actual `FileScanner` actor:

**Actor Concurrency Pattern**:
- All `FileScanner` methods are `async` - call them with `try await`
- Test methods must be marked `async`: `func testScanForCorruptionAndEmptyFolders() async throws`
- Create temporary directory (`temporaryRoot()`) for each test to avoid file system interference

**Multi-Step Workflow Tests**:
```swift
// Example: Full pipeline test
let scanner = FileScanner(rootDirectory: tempDir)
let result = try await scanner.scanForCorruption()        // Step 1: detect corruption
let (repairs, count) = await scanner.repairCorruptedFiles()  // Step 2: repair
try await scanner.moveCorruptedFiles()                       // Step 3: organize
try await scanner.deleteEmptyFolders()                       // Step 4: cleanup
```

**Key Integration Test Patterns**:
- **Temporary filesystem setup** - Use `temporaryRoot()` and create ebook fixtures with `TestFixtures`
- **Progress callback verification** (optional) - Pass progress handler to `scanForCorruption(progress:)` to verify progress events fire
- **State preservation** - After calling `scanner.scanForCorruption()`, access results via `scanner.lastResult`
- **Sequential operation chaining** - Verify that results from one operation feed correctly into the next

### Test Fixture Library (`Support/TestHelpers.swift`)

Essential helpers for creating realistic test files:

**EPUB Fixtures**: 
- `createValidEPUB()` - Proper ZIP + mimetype + container.xml (uncompressed mimetype)
- `createEPUBWithoutMimetype()` - Missing mimetype file
- `createEPUBWithLateMimetype()` - mimetype not first entry (invalid per EPUB spec)
- `createEPUBMissingManifest()` - Missing manifest entries in OPF

**PDF Fixtures**:
- `createPDFWithHeader()` - Valid PDF with `%PDF-` and `%%EOF`
- `createPDFWithoutEOF()` - Missing EOF marker
- `createPDFWithJunkPrefix()` - Email/FTP corruption wrapper

**MOBI/AZW Fixtures**:
- `createMOBI(with: "BOOKMOBI")` - Valid MOBI signature
- `createMOBI(with: "TEXtREAd")` - Valid TEXT format

### When Adding New Features

**Format Validation**: Add unit test in `ValidationTests.swift`
```swift
func testValidateNewFormat() throws {
    let tempDir = try temporaryDirectory()
    let validFile = tempDir.appendingPathComponent("valid.newformat")
    try TestFixtures.createValidNewFormat(at: validFile)
    XCTAssertTrue(validator.validate(url: validFile, as: .newformat).isValid)
    
    let corruptedFile = tempDir.appendingPathComponent("corrupt.newformat")
    try TestFixtures.createCorruptNewFormat(at: corruptedFile)
    XCTAssertFalse(validator.validate(url: corruptedFile, as: .newformat).isValid)
}
```

**Repair Logic**: Test in isolation, then in `ScannerTests` with full workflow
- Unit test: Repair algorithm produces correct output structure
- Integration test: `scanner.repairCorruptedFiles()` finds and repairs the file

**Actor Methods**: Always test async operations with `async throws` and `try await`
- Don't create separate mocks - actors are efficient enough for tests with temporary directories
- Verify actor isolation by confirming `Sendable` constraint compliance at compile time

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/petergi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/petergi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
