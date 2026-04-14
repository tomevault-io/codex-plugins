# GitPulse MCP Server - Copilot Instructions

This project is a Model Context Protocol (MCP) server that monitors GitHub repositories and delivers summarized updates via Poke Automations.

## Project Overview
- **Type**: MCP Server (TypeScript/Node.js)
- **Purpose**: GitHub repository monitoring and automated updates
- **Integration**: Poke Automations platform
- **Key Features**: GitHub API integration, authentication, LLM summarization, real-time updates

## Development Guidelines
- Use TypeScript for type safety
- Follow MCP server protocol specifications
- Implement secure GitHub token authentication
- Use modern Node.js patterns and async/await
- Include comprehensive error handling
- Write unit tests for core functionality

## Code Style
- Use ESLint and Prettier for code formatting
- Follow conventional commit messages
- Use descriptive variable and function names
- Add JSDoc comments for public APIs
- Prefer composition over inheritance

## Architecture Components
- GitHub API connector with rate limiting (using Octokit)
- Personal Access Token authentication
- Event filtering and prioritization logic
- Markdown-based content summarization
- MCP protocol tools, resources, and prompts
- State management for incremental updates

## Current Implementation
The server provides:
- **Tools**: fetchNewCommits, fetchNewPRs, fetchNewIssues, fetchNewReleases, getRepositoryInfo
- **Resources**: repo-activity-summary with github://activity/{owner}/{repo} URIs
- **Prompts**: planRepositoryMonitoring for interactive planning

## Configuration
- Uses GITHUB_TOKEN environment variable for authentication
- Supports stdio transport for MCP communication
- Configurable via .vscode/mcp.json for VS Code integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyler505) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
