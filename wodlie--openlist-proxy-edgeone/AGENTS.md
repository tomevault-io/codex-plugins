<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# OpenList Proxy for EdgeOne Pages

This is an EdgeOne Pages project that serves as a proxy for OpenList file services. The project uses EdgeOne Pages Functions to handle API requests and provide file download capabilities with signature verification.

## Architecture

- **EdgeOne Pages Functions**: Serverless functions that handle HTTP requests on 3200+ edge nodes
- **Modular Design**: Separated concerns into different modules for better maintainability
- **ES6 Modules**: Using modern JavaScript with import/export syntax
- **Environment Configuration**: Runtime configuration through EdgeOne Pages environment variables
- **Edge Computing**: Ultra-low latency with global distribution

## Key Components

1. **Functions Directory**: Contains request handlers
   - `index.js`: Root path handler for service info and homepage
   - `[[path]].js`: Catch-all routing for all other requests
2. **Lib Directory**: Contains shared modules for constants, verification, handlers, and utilities
3. **Public Directory**: Optional static files served directly by EdgeOne Pages
4. **Environment Variables**: Configuration through EdgeOne Pages dashboard

## Development Guidelines

- Use ES6 module syntax (import/export)
- Follow the existing modular structure
- Add proper JSDoc comments for functions
- Use EdgeOne Pages Functions format with `onRequest` handlers
- Handle errors gracefully with proper HTTP status codes
- Use the utility functions for consistent error responses
- Test both with and without signature verification enabled
- Consider CORS requirements for cross-origin requests
- Leverage EdgeOne's global edge network for optimal performance

## EdgeOne Pages Functions Format

Functions should export `onRequest` or specific HTTP method handlers:

```javascript
// Handle all HTTP methods
export async function onRequest(context) {
  const { request, params, env, waitUntil } = context;
  // ... handler logic
}

// Handle specific HTTP methods
export async function onRequestGet(context) {
  // ... GET handler logic
}

export async function onRequestPost(context) {
  // ... POST handler logic
}
```

## Environment Variables

Required:
- `ADDRESS`: OpenList backend server address
- `TOKEN`: API access token for OpenList server

Optional:
- `PAGES_ADDRESS`: Full address of your EdgeOne Pages site
- `DISABLE_SIGN`: Set to "true" to disable signature verification (not recommended for production)

## EdgeOne Pages Advantages

- **Global Distribution**: 3200+ edge nodes worldwide
- **Ultra-low Latency**: Request processing at the edge
- **Elastic Scaling**: Automatic load balancing and scaling
- **Serverless Architecture**: Zero maintenance, pay-per-use

## Security Considerations

- Always validate input paths to prevent directory traversal
- Use signature verification in production environments
- Implement proper CORS headers
- Handle sensitive data (tokens) securely through environment variables
- Leverage EdgeOne's built-in security features

## Local Development

Use EdgeOne CLI for local development:
1. `npm install -g edgeone`
2. `edgeone pages init`
3. `edgeone pages link`
4. `edgeone pages dev`

---
> Source: [Wodlie/openlist-proxy-edgeone](https://github.com/Wodlie/openlist-proxy-edgeone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
