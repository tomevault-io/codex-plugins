# VosDroits MCP Server - GitHub Copilot Instructions

This is a Model Context Protocol (MCP) server written in Go that provides search and retrieval capabilities for French public service information from service-public.gouv.fr.

## Project Overview

- **Language**: Go 1.23+
- **Framework**: MCP Go SDK (`github.com/modelcontextprotocol/go-sdk`)
- **Purpose**: MCP server for searching French public service procedures and articles
- **Deployment**: Docker container published to GitHub Packages
- **CI/CD**: GitHub Actions workflows

Always use Context7 to use the latest best practices and versions.
Generated Git Copilot Commit messages should follow conventional commits format (short 1 liner but explicit).
Documentation should be clear and concise and in the docs/ folder (as subfolders as needed).

## Core Functionality

The server provides three main tools:
1. `search_procedures` - Search for procedures on service-public.gouv.fr
2. `get_article` - Retrieve detailed information from a specific article URL
3. `list_categories` - List available categories of public service information

## Development Principles

### Code Quality
- Write idiomatic Go code following [Effective Go](https://go.dev/doc/effective_go)
- Use the official MCP Go SDK patterns and best practices
- Prioritize type safety with struct-based inputs/outputs
- Handle errors explicitly and provide informative error messages
- Use JSON schema tags for comprehensive API documentation

### Architecture
- Keep the codebase simple and maintainable
- Separate concerns: HTTP client, MCP tools, main server logic
- Use dependency injection for testability
- Follow standard Go project layout conventions

### Testing
- Write unit tests for all tool handlers
- Use table-driven tests for multiple scenarios
- Test both success and error paths
- Ensure context cancellation is properly handled

### Docker & Deployment
- Use multi-stage Docker builds for minimal image size
- Build statically-linked binaries for scratch-based images
- Tag Docker images appropriately for versioning
- Publish to GitHub Container Registry (ghcr.io)

### Security
- Validate all external inputs
- Use HTTPS for external API calls
- Handle rate limiting gracefully
- Sanitize user-provided URLs and queries

## File Organization

```
mcp-vosdroits/
├── .github/
│   ├── copilot-instructions.md          # This file
│   ├── instructions/                     # Language-specific guidelines
│   ├── prompts/                          # Reusable prompts
│   ├── chatmodes/                        # Specialized chat modes
│   └── workflows/                        # GitHub Actions workflows
├── cmd/
│   └── server/
│       └── main.go                       # Server entry point
├── internal/
│   ├── tools/                            # MCP tool implementations
│   ├── client/                           # HTTP client for service-public.gouv.fr
│   └── config/                           # Configuration management
├── Dockerfile                            # Multi-stage Docker build
├── go.mod                                # Go module definition
├── go.sum                                # Go dependencies checksum
└── README.md                             # Project documentation
```

## Related Instructions

- [Go Development Guidelines](instructions/go.instructions.md)
- [Go MCP Server Best Practices](instructions/go-mcp-server.instructions.md)
- [Testing Standards](instructions/testing.instructions.md)
- [Docker Guidelines](instructions/docker.instructions.md)
- [Security Best Practices](instructions/security.instructions.md)
- [Documentation Standards](instructions/documentation.instructions.md)

## Environment Variables

- `SERVER_NAME`: Name of the MCP server (default: "vosdroits")
- `SERVER_VERSION`: Server version (default: "v1.0.0")
- `LOG_LEVEL`: Logging level (debug, info, warn, error)
- `HTTP_TIMEOUT`: Timeout for HTTP requests to external services

## Quick Start Commands

```bash
# Install dependencies
go mod download

# Run tests
go test ./...

# Build locally
go build -o bin/mcp-vosdroits ./cmd/server

# Build Docker image
docker build -t mcp-vosdroits .

# Run with stdio transport
./bin/mcp-vosdroits

# Run with HTTP transport
HTTP_PORT=8080 ./bin/mcp-vosdroits
```

---
> Source: [guigui42/mcp-vosdroits](https://github.com/guigui42/mcp-vosdroits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
