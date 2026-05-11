<!-- Use this file to provide workspace-specific custom instructions to Copilot. -->

# WHMCS MCP Server

This is a Model Context Protocol (MCP) server for managing WHMCS installations.

## Project Structure

- `src/index.ts` - Main MCP server with all tool registrations
- `src/whmcs-client.ts` - WHMCS API client for making authenticated requests
- `.vscode/mcp.json` - VS Code MCP configuration

## Key Technologies

- TypeScript
- Model Context Protocol SDK (`@modelcontextprotocol/sdk`)
- Zod for schema validation
- WHMCS External API

## WHMCS API Reference

The server uses the WHMCS External API which communicates via POST requests to `/includes/api.php`.

Authentication is done via API credentials (identifier + secret) which can be created in:
WHMCS Admin > Setup > Staff Management > API Credentials

## Adding New Tools

To add a new WHMCS API tool:

1. Add the method to `src/whmcs-client.ts` with proper TypeScript types
2. Register the tool in `src/index.ts` using `server.registerTool()`
3. Define input schema using Zod validators
4. Handle the response appropriately

## Building and Running

```bash
npm install     # Install dependencies
npm run build   # Build TypeScript
npm run dev     # Run in development mode
npm start       # Run compiled version
```

## Environment Variables

Required:
- `WHMCS_API_URL` - WHMCS installation URL
- `WHMCS_API_IDENTIFIER` - API credential identifier
- `WHMCS_API_SECRET` - API credential secret

Optional:
- `WHMCS_ACCESS_KEY` - Additional API access key

---
> Source: [scarecr0w12/whmcs-mcp-tool](https://github.com/scarecr0w12/whmcs-mcp-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-11 -->
