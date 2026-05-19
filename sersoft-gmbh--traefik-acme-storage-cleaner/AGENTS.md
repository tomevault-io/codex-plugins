# Traefik ACME Storage Cleaner - Copilot Instructions

## Repository Overview

This is a small, focused Go CLI application that cleans expired certificates from Traefik ACME storage JSON files. The application is designed for simplicity and performance, processing multiple files in parallel using worker pools.

**Repository Statistics:**
- Language: Go 1.25
- Size: ~14 files (excluding dependencies)
- Type: CLI application
- Main Dependencies: github.com/traefik/traefik/v3 v3.6.7
- Test Coverage: 65.1%

## Project Structure

```
.
├── main.go                 # Main application code (single file application)
├── main_test.go            # Unit tests for main application
├── go.mod                  # Go module definition
├── go.sum                  # Go dependencies checksums
├── Dockerfile              # Multi-stage, multi-arch Docker build
├── README.md               # User documentation
├── LICENSE                 # License file
└── .github/
    ├── workflows/
    │   ├── build.yml       # CI build workflow (multi-OS, multi-arch)
    │   ├── test.yml        # CI test workflow (runs tests with coverage)
    │   ├── docker-publish.yml  # Docker image publishing on release
    │   └── enable-auto-merge.yml
    ├── dependabot.yml      # Dependabot configuration
    └── CODEOWNERS          # Code ownership configuration
```

## Building and Testing

### Prerequisites
- Go 1.25 or later (specified in go.mod)
- No additional tools required for basic builds

### Build Commands

**Standard build:**
```bash
go build .
```
This creates the `traefik-acme-storage-cleaner` binary in the current directory.

**Build time:** Approximately 2-3 minutes on first build (downloads dependencies), ~5-10 seconds on subsequent builds.

**Cross-platform builds:**
The project supports multiple platforms via environment variables:
```bash
# Linux AMD64 (default)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build .

# macOS ARM64
GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build .

# Windows AMD64
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build .
```

**Important:** CGO_ENABLED=0 must be set to disable glibc linking for portability.

### Docker Build

**Build Docker image:**
```bash
docker build -t traefik-acme-storage-cleaner .
```

The Dockerfile uses a multi-stage, multi-architecture build:
1. Stage 1: golang:1.25-alpine builder with cross-compilation support
   - Uses `--platform=$BUILDPLATFORM` for native builder execution
   - Supports TARGETOS and TARGETARCH build arguments for cross-compilation
   - Supports linux/amd64 and linux/arm64 architectures
2. Stage 2: scratch base for minimal image size

**Build time:** ~3-5 minutes on first build, faster with Docker layer caching.

**Multi-arch build:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t traefik-acme-storage-cleaner .
```

### Testing

**Unit tests exist in main_test.go** with comprehensive test coverage.

**Run tests:**
```bash
go test -v ./...
```

**Run tests with coverage:**
```bash
go test -v -race -coverprofile=coverage.out -covermode=atomic ./...
go tool cover -func=coverage.out
```

**Test Coverage:** 65.1% (minimum threshold: 60%)

**Test coverage includes:**
- Core functions: `isExpired()`, `formatDomain()`, `defaultWorkers()`
- File processing: Expired cert removal, permission preservation, multi-resolver support, error handling
- Concurrency: Parallel file processing with configurable workers
- Edge cases: Empty files, invalid JSON, missing files, malformed certificates

**Test certificates:** Generated programmatically using `crypto/x509` with adjustable `NotBefore`/`NotAfter` timestamps. Helper function `mustGenerateTestCertificate()` ensures test failures on cert generation errors.

**Manual testing:**
1. Build the binary: `go build .`
2. Create a test ACME JSON file with sample certificates
3. Run: `./traefik-acme-storage-cleaner test-acme.json`
4. Verify the output shows certificate processing

### Running the Application

```bash
# Basic usage
./traefik-acme-storage-cleaner /path/to/acme.json

# Multiple files
./traefik-acme-storage-cleaner file1.json file2.json file3.json

# Custom worker count
./traefik-acme-storage-cleaner -workers 8 /path/to/acme.json

# Docker
docker run --rm -v /path/to/acme:/data ghcr.io/sersoft-gmbh/traefik-acme-storage-cleaner:latest /data/acme.json
```

## CI/CD Pipeline

### Build Workflow (.github/workflows/build.yml)
- Triggers on: push to main, pull requests to main
- Tests: Builds for 6 combinations (linux/darwin/windows × amd64/arm64)
- Go version: Read from go.mod using actions/setup-go@v6
- Each build runs with `CGO_ENABLED=0` and uses Go module caching

### Test Workflow (.github/workflows/test.yml)
- Triggers on: push to main, pull requests to main
- Runs tests with race detector: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
- Displays coverage breakdown: `go tool cover -func=coverage.out`
- Enforces 60% minimum coverage threshold
- Reports total coverage percentage
- Fails if coverage drops below threshold

### Docker Publish Workflow (.github/workflows/docker-publish.yml)
- Triggers on: Release published
- Publishes to: GitHub Container Registry (ghcr.io)
- Multi-architecture support: linux/amd64 and linux/arm64
- Uses QEMU for emulation: `docker/setup-qemu-action@v3`
- Uses buildx for multi-platform builds: `docker/setup-buildx-action@v3`
- Tags generated:
  - `vX.Y.Z` (full semver)
  - `vX.Y` (major.minor)
  - `vX` (major)
  - `sha-<commit>` (git SHA)

## Code Architecture

### Main Components

**main.go** contains all application logic:
- `main()`: Entry point, handles CLI flags and orchestrates processing
- `processFiles()`: Worker pool implementation for parallel file processing
- `processFile()`: Processes a single ACME storage file
- `isExpired()`: Parses and checks certificate expiration
- `formatDomain()`: Formats domain information for display

### Key Data Structures

The application uses Traefik's ACME data structures:
- `map[string]*acme.StoredData`: Top-level storage format (resolver name → data)
- `acme.StoredData`: Contains certificate array and account info
- `acme.CertAndStore`: Individual certificate with domain and store info
- `types.Domain`: Domain with Main and SANs (Subject Alternative Names)

### Parallel Processing

The application uses a worker pool pattern:
- Default workers: `runtime.GOMAXPROCS(0)` (respects container CPU limits)
- Configurable via `-workers` flag
- Files processed concurrently using channels and goroutines
- Results collected and summarized at the end

### Error Handling

- File read errors: Reported and skipped
- Certificate parsing errors: Warning printed, certificate treated as expired
- Partial failures: Application continues processing remaining files
- Exit code: 0 if all files processed successfully, 1 if any failures

## Coding Conventions

### Style Guidelines
- Standard Go formatting (use `go fmt`)
- Minimal comments (code is self-documenting)
- Error messages written to `os.Stderr`
- Progress/status messages to `os.Stdout`
- Use emoji in output: ✓ for success, ❌ for errors

### Testing Guidelines
- All new features must have corresponding unit tests in main_test.go
- Maintain test coverage at ≥60% (enforced by CI)
- Use `mustGenerateTestCertificate()` helper for test certificate generation
- Test edge cases: empty data, invalid input, error conditions
- Tests run with race detector in CI: write thread-safe code

### File Handling
- Preserve original file permissions when writing
- Use `os.ReadFile` and `os.WriteFile` (not deprecated `ioutil`)
- Only write file if modifications were made

### Dependencies
- Minimize external dependencies
- Primary dependency: github.com/traefik/traefik/v3 for ACME types
- Use Go standard library whenever possible

## Common Tasks

### Adding New Features
1. Implement the feature in main.go
2. Add comprehensive tests in main_test.go
3. Run `go test -v ./...` to verify tests pass
4. Check coverage: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
5. Ensure coverage remains ≥60%
6. Format code: `go fmt main.go main_test.go`
7. Commit with descriptive message

### Adding New Command-Line Flags
1. Declare flag variable in main()
2. Use `flag.IntVar`, `flag.StringVar`, etc.
3. Ensure `flag.Parse()` is called before accessing
4. Update usage message if needed
5. Add tests for the new flag behavior

### Modifying Certificate Logic
- Certificate expiration check: See `isExpired()` function
- Handles both PEM-encoded and DER format certificates
- Uses Go's `crypto/x509` package for parsing

### Updating Dependencies
```bash
# Update specific dependency
go get github.com/traefik/traefik/v3@latest

# Update all dependencies
go get -u ./...

# Tidy modules
go mod tidy
```

### Creating a Release
1. Tag the commit: `git tag vX.Y.Z`
2. Push tag: `git push origin vX.Y.Z`
3. Create GitHub release (triggers docker-publish workflow)
4. Docker images automatically published to ghcr.io

## Important Notes

### Always Remember
- This is a **single-file application** (main.go with tests in main_test.go) - keep it simple
- **Tests exist in main_test.go** - always run `go test -v ./...` after changes
- **Test coverage must be ≥60%** - enforced by CI
- **CGO_ENABLED=0** must always be set for builds to ensure static linking
- Worker count defaults to available CPUs (respects container limits)
- The application modifies files in-place (preserves permissions)
- Files are processed in parallel - order is not guaranteed
- Docker builds support multi-arch: linux/amd64 and linux/arm64

### Known Limitations
- No dry-run mode
- No backup functionality (modifies files directly)
- No validation that input files are valid ACME storage format before writing
- Error handling treats unparseable certificates as expired (safe default)

### Validation Steps for Changes
Always run automated tests first, then perform manual validation:
1. Run unit tests: `go test -v ./...`
2. Run tests with coverage: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./...`
3. Verify coverage: `go tool cover -func=coverage.out` (must be ≥60%)
4. Build the application: `go build .`
5. Manual validation (if needed):
   - Create or obtain a test ACME JSON file
   - Run the application on the test file
   - Verify:
     - Application completes without crashing
     - Output messages are correct and formatted properly
     - File is modified correctly (if certificates were expired)
     - File permissions are preserved
     - Multi-file processing works correctly
6. Test with edge cases (covered by unit tests):
   - Empty ACME file: `{}`
   - File with no expired certificates
   - File with all expired certificates
   - Multiple files at once
   - Invalid JSON (should fail gracefully)

### Build Troubleshooting
- If build fails with dependency errors: Run `go mod download` first
- If build is slow: Go is downloading dependencies (first build only)
- If binary doesn't run on target system: Ensure CGO_ENABLED=0 was set during build
- If Docker build fails: Check that go.mod and go.sum are in sync

## Example Workflow for Code Changes

1. Modify main.go
2. Update or add tests in main_test.go if needed
3. Format code: `go fmt main.go main_test.go`
4. Run tests: `go test -v ./...`
5. Check coverage: `go test -v -race -coverprofile=coverage.out -covermode=atomic ./... && go tool cover -func=coverage.out`
6. Build: `go build .`
7. Test manually with sample files (if needed)
8. If all tests pass and coverage is ≥60%, commit changes
9. Push to branch and create PR
10. CI will automatically:
    - Build for all 6 platform combinations
    - Run tests with race detector
    - Check coverage threshold (≥60%)
    - Build multi-arch Docker images (on release)
11. Merge when all CI checks pass

---
> Source: [sersoft-gmbh/traefik-acme-storage-cleaner](https://github.com/sersoft-gmbh/traefik-acme-storage-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
