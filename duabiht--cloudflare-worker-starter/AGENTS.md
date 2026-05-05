# Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a Cloudflare Worker project using JavaScript. The worker provides:
- Basic HTTP request handling
- Authentication middleware
- Environment variable configuration
- Response utilities

## Development Guidelines
- Use modern JavaScript features (ES6+)
- Follow Cloudflare Worker API patterns
- Implement proper error handling
- Use environment variables for configuration
- Write clean, maintainable code

## Key Files
- `src/worker.js` - Main worker entry point
- `wrangler.toml` - Cloudflare configuration
- `.env.example` - Environment variables template

## Development Commands
- `npm run dev` - Start development server with Wrangler
- `npm run deploy` - Deploy to Cloudflare
- `npm test` - Run tests

---
> Source: [duabiht/cloudflare-worker-starter](https://github.com/duabiht/cloudflare-worker-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
