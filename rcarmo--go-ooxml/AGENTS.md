# Go OOXML Library - Copilot Instructions

## Build and Test Commands

Use the Makefile for all standard operations:

```bash
make help          # Show all available targets
make build-all     # Full build (clean + deps + lint + test + build)
make test          # Run all tests
make coverage      # Run tests with coverage
make lint          # Run golangci-lint
make format        # Format code with gofumpt
make check         # Run lint + tests
```

## Makefile-First Workflow

If you need a new workflow step, add a Make target rather than running ad-hoc commands.

## CI/CD Convention

CI should call `make check` (or `make lint` + `make test` if `check` is unavailable).

For targeted tests, use `go test` directly:

```bash
go test -v ./pkg/document -run TestParagraph_SetText   # Single test
go test -v ./pkg/spreadsheet -run TestCell             # Tests matching pattern
```

## Architecture

This library implements Office Open XML (OOXML) support for Word (.docx), Excel (.xlsx), and PowerPoint (.pptx) with a layered architecture:

```
High-Level API (document.Document, spreadsheet.Workbook, presentation.Presentation)
    ↓
Element Wrappers (Paragraph, Cell, Slide, Shape, etc.)
    ↓
OOXML Types (pkg/ooxml/wml, sml, pml, dml - low-level XML structs)
    ↓
Packaging (pkg/packaging - OPC/ZIP handling, relationships, content types)
    ↓
Go Standard Library (archive/zip, encoding/xml)
```

### Package Responsibilities

- **`pkg/packaging`** - OPC package handling (ZIP, relationships, `[Content_Types].xml`)
- **`pkg/ooxml/*`** - Low-level XML types matching ECMA-376 spec (CT_* structs)
  - `wml/` - WordprocessingML
  - `sml/` - SpreadsheetML  
  - `pml/` - PresentationML
  - `dml/` - DrawingML (shared across formats)
- **`pkg/document`** - High-level Word API
- **`pkg/spreadsheet`** - High-level Excel API
- **`pkg/presentation`** - High-level PowerPoint API
- **`pkg/utils`** - Shared utilities (EMU conversions, color parsing, cell references)
- **`internal/testutil`** - Shared test infrastructure

## Key Conventions

### Zero External Dependencies
Use only Go standard library (`archive/zip`, `encoding/xml`, `io`, `path`). Do not add third-party dependencies.

### Consistent Document Lifecycle
All document types follow the same pattern:
```go
// Create new
doc, err := document.New()   // or spreadsheet.New(), presentation.New()

// Open existing  
doc, err := document.Open("file.docx")

// Open from reader
doc, err := document.OpenReader(reader, size)

// Save
doc.Save()           // Save to original path
doc.SaveAs("new.docx")

// Always close
defer doc.Close()
```

### Table-Driven Tests
Use parameterized tests with the shared test infrastructure:
```go
func TestParagraph_SetText(t *testing.T) {
    for _, tc := range CommonTextCases {
        t.Run(tc.Name, func(t *testing.T) {
            h := NewTestHelper(t)
            doc := h.CreateDocument(nil)
            defer doc.Close()
            // test logic
        })
    }
}
```

Common test data is defined in `internal/testutil/testutil.go`:
- `CommonStringCases`, `CommonNumericCases`
- `CommonFormatCombinations`
- `CommonCellRefCases`, `CommonRangeCases`

### XML Struct Naming
Low-level OOXML types use ECMA-376 naming conventions prefixed with `CT_`:
- `CT_P` (paragraph), `CT_R` (run), `CT_Tbl` (table)
- `CT_Cell`, `CT_Slide`, `CT_Shape`

### EMU Units
Measurements use English Metric Units (EMUs). Use `pkg/utils/emu.go` for conversions:
- 914400 EMUs = 1 inch
- 12700 EMUs = 1 point

### Error Handling
Custom error types are defined in `pkg/utils/errors.go` and `pkg/spreadsheet/errors.go`.

## Test Fixtures

Test files are in `testdata/`. See `testdata/FIXTURES.md` for the fixture documentation. When testing document manipulation, prefer round-trip tests: Open → Modify → Save → Re-open → Verify.

---
> Source: [rcarmo/go-ooxml](https://github.com/rcarmo/go-ooxml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
