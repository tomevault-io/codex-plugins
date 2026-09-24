# Copilot instructions for Peekd

## Build, test, and checks

The repository requires Go 1.26 as declared in `go.mod`.

```bash
make build       # builds ./peekd; VERSION comes from git describe unless set
make test        # verbose: go test -v ./...
make vet         # go vet ./...
make check       # fmt-check, vet, and tests
make run         # serves ROOT (default .) at ADDR (default :8090)
```

Run one test with the standard Go test filter:

```bash
go test ./internal -run '^TestZIPFileListsEntries$' -v
go test ./internal/middleware -run '^TestNewBasicAuth$' -v
```

`make fmt` runs `gofmt` on tracked Go files. `make fmt-check` expects a clean
working tree before running, then verifies that formatting produces no diff.
The CI workflow runs `make check` and `make build`.

## Architecture

`main.go` is the executable entry point. It parses CLI flags, resolves the
root directory, reads `PEEKD_AUTH_USER` and `PEEKD_AUTH_PASSWORD`, embeds
`assets/*`, and passes all runtime configuration to `internal.Run`.

`internal.NewHandler` builds the HTTP handler and parses every embedded HTML
template. It mounts embedded CSS and templates under `/__peekd_assets/`,
handles directory rendering, and routes file requests. The request path is
mapped beneath `Config.RootDir`; directories render an index page, while
files either use the direct `http.FileServer` path or a browser document
preview.

Preview selection first checks the filename extension and then falls back to
content sniffing. Text, Markdown, and JSON content is read through the shared
bounded preview reader. JSON is formatted with the standard library and falls
back to text when invalid. Markdown is rendered with Goldmark. CSV/TSV previews
use `encoding/csv` parsing and render escaped tables. They are bounded by the
configured preview file size, without separate row or column limits. Images,
audio/video, PDF, and ZIP/TAR/TAR.GZ archives each have format-specific
renderers and templates. Archive previews list entries without extracting
files.

Direct file responses intentionally remain on `http.FileServer` so range
requests and platform file-serving optimizations are preserved. `raw=1`
always selects this path. Otherwise, document previews depend on
`Sec-Fetch-Dest=document`; when browsers omit that header for LAN navigation,
`Upgrade-Insecure-Requests: 1` is the fallback signal. Non-document requests
serve the original file so preview subresources do not recurse into previews.

Middleware is assembled in `internal/server.go` as:

```text
request:  access log -> Basic Auth -> mux
response: mux -> Basic Auth -> access log
```

`internal/middleware/auth.go` implements optional HTTP Basic Auth. Both
environment variables must be set to enable it; setting only one is a startup
error. `internal/middleware/accesslog.go` records status, response bytes,
duration, client address, Referer, and User-Agent while bypassing embedded
asset requests.

## Repository-specific conventions

- Keep server implementation files under `internal/` with `package internal`.
  Middleware is the intentional exception and lives under
  `internal/middleware` with `package middleware`.
- Add a format-specific preview implementation in its own `internal/preview_*.go`
  file and add its embedded HTML template under `assets/`. Wire template
  parsing in `NewHandler`, preview detection in both extension/content paths
  when appropriate, and dispatch in the single `switch preview`.
- Preserve graceful fallback behavior: unsupported files and failed archive
  parsing use direct file serving; oversized or invalid text-like content
  falls back to direct serving or the text preview as already defined.
- Keep embedded asset names synchronized with `fstest.MapFS` entries in
  `internal/server_test.go`; handler construction deliberately fails if a
  required template is missing.
- Use second precision for displayed file modification times:
  `2006-01-02 15:04:05`.
- Keep preview resource URLs using `?raw=1` where a template embeds or
  downloads the original file.
- Port binding uses `listenWithFallback`: an occupied configured port is
  retried at successive ports until one is available or the port range ends.
- User-facing behavior and examples belong in `README.md`; technical details
  about direct serving and preview planning belong in `docs/`.

---
> Source: [jiacai2050/peekd](https://github.com/jiacai2050/peekd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-24 -->
