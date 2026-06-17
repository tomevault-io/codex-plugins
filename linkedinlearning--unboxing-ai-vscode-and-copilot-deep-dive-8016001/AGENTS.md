
# MCP Demo Project - Copilot Instructions

This project demonstrates building an advanced MCP server using FastMCP combined with weather and mapping APIs.

## Project Overview

- **Technology Stack**: FastMCP (Python), TypeScript/Node (optional clients), OpenWeatherMap API, OpenStreetMap Nominatim API
- **Purpose**: Educational demo showing Copilot integration with MCP server development
- **Key Artifact**: Weather & Mapping MCP server with 5 primary tools

## Core Development Principles

### 1. MCP Protocol Compliance

- All tools must follow MCP protocol specifications
- Tool schemas must be complete and valid (auto-generated from FastMCP docstrings)
- Tools should follow semantic versioning for parameters and return types

### 2. Tool Design Standards

- Every tool must have a comprehensive docstring explaining:
  - Purpose and use case
  - All parameters with types and constraints
  - Return value structure
  - Potential errors and edge cases
- Use type hints exclusively (FastMCP requires them for schema generation)
- Async functions for all I/O operations
- Proper error handling with meaningful error messages

### 3. Documentation First

- Docstrings are primary documentation (used by FastMCP for schema generation)
- Examples in docstrings help agents understand tool usage
- Tool names should be descriptive and follow snake_case convention

### 4. Testing Requirements

- All tools should have unit tests
- Integration tests verify tool interactions
- Mock external APIs (OpenWeatherMap, OpenStreetMap) in tests
- Aim for 80%+ code coverage

### 5. Error Handling

- All external API calls must have try-catch blocks
- Return structured error responses (never raise exceptions to caller)
- Include context in error messages: what was attempted, why it failed, suggestions

## Tool Implementation Checklist

When creating or modifying MCP tools, ensure:

- [ ] Docstring explains purpose, parameters, returns, errors
- [ ] Type hints on all parameters and return values
- [ ] Async for I/O operations
- [ ] Error handling for external API calls
- [ ] Validation of input parameters
- [ ] Meaningful return structure
- [ ] Examples in docstring or tool description
- [ ] Tests written before or immediately after
- [ ] Tool name follows snake_case convention

## Code Organization

- `/src/server/` - FastMCP server code
- `/tests/` - Test suite
- `/docs/` - Documentation

## When to Reference Context Files

Consult these files for guidance:

- `.github/copilot/tech-stack.md` - Exact versions and API details
- `.github/copilot/architecture.md` - MCP protocol and design patterns
- `.github/copilot/exemplars.md` - High-quality tool examples to emulate

---
> Source: [LinkedInLearning/unboxing-ai-vscode-and-copilot-deep-dive-8016001](https://github.com/LinkedInLearning/unboxing-ai-vscode-and-copilot-deep-dive-8016001) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
