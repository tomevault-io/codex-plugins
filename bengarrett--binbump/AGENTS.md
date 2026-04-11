# Copilot Instructions for BINbump

## Overview

BINbump is a Go library that converts binary screen dumps (IBM PC text/graphics in CodePage 437) to HTML `<div>` output with proper color attributes and styling. The core use case is rendering ANSI and binary artwork from retro PC systems as HTML for archival/display.

## Build, Test & Lint

### Testing
- **Full test suite**: `go test -count 1 ./...`
- **With race detection**: `go test -count 1 -race ./...`
- **Via Task**: `task test` or `task testr`

### Linting & Formatting
- **Full check**: `task lint` (runs `gofumpt`, `golangci-lint`)
- **Nil analysis**: `task nil` (uses `go.uber.org/nilaway`)
- **Patch dependencies**: `task patch` or `task pkg-patch`
- **Update dependencies**: `task update` or `task pkg-update`

### Documentation
- **Serve godoc locally**: `task doc` (opens `pkgsite` at `localhost:8090`)

## Architecture

### Core Types & Flow

**Decoder** (`binbump.Decoder`): Main stateful processor
- Maintains character/color state, position tracking (row/column), and buffered output
- Processes 2-byte pairs: character byte + attribute byte (IBM video RAM format)
- Builds HTML line-by-line with span optimization (groups consecutive same-color characters)

**Palette** (uint enum): Color scheme selector
- `StandardCGA`: IBM's original 1981 CGA palette
- `RevisedCGA`: Corrected palette (VilaR reference)
- Custom `Colors [16]Color` can be provided

**Public API Entry Points**:
- `WriteTo(reader, writer)`: Stream processing for simplicity
- `String(reader)`: Returns complete HTML div as string
- `Bytes(reader)`: Returns complete HTML div as bytes
- `Buffer(reader, width, maxRows, palette, charset)`: Full control with custom charset

### Input Processing

1. Reader splits input into 2-byte tokens via `splitTwoBytes()` scanner split function
2. Each pair: `chr` (character byte) + `atr` (attribute byte)
3. `decodeAttr(byte)` extracts FG (0-15) and BG (0-7) color indices from attribute bits
4. Character converted via charmap (default: CodePage437)
5. HTML span generated with CSS color styles or appended to current span if colors match

### Output Characteristics

- Wraps content in outer `<div>`
- Line breaks as newlines within div (column wrap at specified width, default 160)
- Color optimization: consecutive chars with same `style=` attribute share a single `<span>`
- Debug mode: wraps every character in individual span with `data-xy` position attribute
- HTML escaping applied to all characters

## Key Conventions

### Byte Pair Format (IBM Video RAM)
- **Byte 0**: Character (CodePage 437 code)
- **Byte 1 (Attribute)**: `[Blink(1bit)|BG(3bits)|Intensity(1bit)|FG_Low(3bits)]`
  - Foreground: 4 bits (low 3 + intensity)
  - Background: 3 bits only (bits 4-6, no intensity)
  - Blink bit ignored (not rendered in HTML)

### Color Handling
- Colors are 3-6 char hex strings (e.g., `"000"`, `"c4c4c4"`)
- `Color.FG()` and `Color.BG()` methods return CSS property strings
- 16-color palette always used, accessed by index (0-15)

### Width & Row Handling
- Default width: 160 columns (80 for some APIs with default 0)
- `maxRows > 0`: Early termination after specified rows (handles corrupted trailing data)
- `maxRows == 0`: Process entire input (default behavior)

### Testing Pattern
- Tests use hardcoded byte slices (`[]byte{0x41, 0x00, ...}`)
- Assertions compare generated HTML strings to expected output
- Examples double as tests (ExampleXxx functions validate output)

### HTML Output Format (No Outer Wrapper in API)
Most functions return only the inner content (`<span>...`, no outer `<div>`).
The `Write()` method wraps in `<div>{{ . }}</div>` template.
Intended use: embed in `<pre>` tag with appropriate CSS.

### Error Handling
- ErrReader: nil reader passed
- ErrAttribute: decoded color value > 15 (malformed data)
- Writer errors wrapped with context prefix (e.g., "buffer write to:")

## Linting Notes

- **Disabled linters** (.golangci.yaml): depguard, exhaustruct, funlen, nlreturn, noinlineerr, varnamelen, wsl
- **Format exclusions**: Generated files, third_party, builtin, examples
- **Nil analysis enabled**: Uses `nilaway` tool to catch potential nil dereferences
- **Line length rule disabled** for `_test.go` files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bengarrett)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/bengarrett)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
