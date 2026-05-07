# EntraPulse Lite Copilot Instructions

## Project Overview
EntraPulse Lite is a freemium desktop application that provides natural language querying of Microsoft Graph APIs through local LLM integration. It's a lightweight derivative of the EntraPulse project.

## Architecture
- **Platform**: Electron desktop application
- **Language**: TypeScript
- **Build Tool**: Webpack
- **Framework**: Electron Forge
- **Authentication**: Microsoft MSAL for Electron
- **LLM Integration**: Local models via Ollama/LM Studio
- **MCP Servers**: Built-in Lokka MCP for Microsoft Graph and Fetch MCP for Microsoft Learn documentation

## Key Features
1. Microsoft Business Account authentication
2. Natural language Microsoft Graph API queries
3. Local LLM integration with Microsoft Entra knowledge
4. Chat interface with trace visualization
5. Freemium model (basic free, advanced paid)

## Development Guidelines

### File Structure
```
src/
├── main/                 # Main process code
├── renderer/             # Renderer process code
├── shared/               # Shared utilities
├── mcp/                  # MCP server integration
├── auth/                 # Authentication logic
├── llm/                  # Local LLM integration
├── types/                # TypeScript definitions
└── tests/                # Unit and integration tests
```

### Coding Standards
- Use TypeScript for type safety
- Follow Microsoft Graph API best practices
- Implement proper error handling for network requests
- Use async/await for asynchronous operations
- Follow Electron security best practices

### Authentication
- Use MSAL for Electron for Microsoft authentication
- Support both work/school and personal accounts
- Handle token refresh automatically
- Secure token storage in Electron

### MCP Integration
- Built-in Lokka MCP server for Microsoft Graph API access
- Built-in Fetch MCP server for Microsoft Learn documentation and Permissions Explorer
- Local communication with MCP protocol
- Error handling for MCP server failures
- Support for multiple concurrent MCP connections

### UI/UX
- Modern, clean interface similar to Claude Desktop
- Dark/light theme support
- Responsive design for different window sizes
- Accessibility compliance

### Security
- Follow Electron security guidelines
- Secure IPC communication
- Proper CSP implementation
- No node integration in renderer by default

## Dependencies to Use
- `@azure/msal-electron` for authentication
- `@microsoft/microsoft-graph-client` for Graph API
- Material-UI or similar for components
- MCP TypeScript SDK for server integration
- Axios/fetch for HTTP requests

## Testing
- Unit tests for core functionality
- Integration tests for authentication flow
- E2E tests for critical user journeys
- Mock Microsoft Graph responses for testing

---
> Source: [darrenjrobinson/EntraPulseLite](https://github.com/darrenjrobinson/EntraPulseLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
