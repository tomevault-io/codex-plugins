<!-- Use this file to provide workspace-specific custom instructions to Copilot -->

# Squarespace MCP Editor - Workspace Instructions

## Project Overview
This is a Model Context Protocol (MCP) server that provides live editing capabilities for Squarespace websites through browser automation and intelligent site knowledge management.

## Key Technologies
- TypeScript + Node.js 20+
- @modelcontextprotocol/sdk for MCP server
- Playwright for browser automation
- Cheerio for HTML parsing
- Zod for schema validation
- Pino for structured logging

## Architecture Principles

### 1. Site Brain Knowledge Base
- Maintain complete site understanding through crawling and snapshots
- Store structured data in `./site-brain/`
- Enable search across all content
- Build reusable component models

### 2. Patch-Based Editing
- All changes represented as patch files (JSON)
- Strict schema validation using Zod
- Operations include: updateTextBlock, updateButton, updateImage, etc.
- Support for dry-run and safe-publish modes

### 3. Browser Automation Strategy
- Use Playwright to interact with Squarespace editor UI
- BlockHint system to locate elements (nearText, heading, buttonLabel, selector)
- Session persistence to avoid repeated logins
- Screenshot capture for debugging and verification

### 4. Safety First
- Never auto-publish unless explicitly enabled
- Validate all patches before application
- Log all operations with structured logging
- Store before/after artifacts

## Code Style Guidelines

### TypeScript
- Use strict mode
- Prefer interfaces over types for public APIs
- Use Zod schemas for runtime validation
- Always handle errors explicitly
- Use async/await over Promises

### Error Handling
```typescript
try {
  // operation
} catch (error: any) {
  logger.error({ context, error }, 'Operation failed');
  return { success: false, error: error.message };
}
```

### Logging
```typescript
import { createModuleLogger } from '../utils/logger.js';
const logger = createModuleLogger('module-name');

logger.info({ data }, 'Operation started');
logger.debug({ details }, 'Debug info');
logger.error({ error }, 'Operation failed');
```

### File Operations
```typescript
import { readJson, writeJson, ensureDir } from '../utils/helpers.js';

// Always ensure directory exists before writing
await ensureDir('./path');
await writeJson('./path/file.json', data);
```

## Common Tasks

### Adding a New MCP Tool
1. Define tool in `TOOLS` array in `src/index.ts`
2. Add handler case in `handleToolCall()`
3. Implement logic (may require new modules)
4. Update README documentation

### Adding a New Patch Operation
1. Add to `PatchOperationSchema` in `src/types/schemas.ts`
2. Implement handler in `src/browser/editor.ts`
3. Add case in `src/patch/manager.ts` `applyPatch()`
4. Test thoroughly with dry-run first

### Debugging Browser Automation
- Set `PLAYWRIGHT_HEADLESS=false` to watch browser
- Check screenshots in `./site-brain/cache/screenshots/`
- Increase `PLAYWRIGHT_SLOW_MO` for slower execution
- Review logs for selector matching issues

## Testing Strategy
- Manual testing required (Squarespace is live)
- Always test with `DRY_RUN=true` first
- Use example patches for regression testing
- Verify changes on published site

## File Organization
- `/src` - TypeScript source
- `/dist` - Compiled JavaScript
- `/site-brain` - Runtime knowledge base
- `/.vscode` - Editor configuration

## Build Commands
- `npm run build` - Compile TypeScript
- `npm run watch` - Watch mode compilation
- `npm run lint` - Check code style
- `npm run format` - Auto-format code

## Development Workflow
1. Make changes in `/src`
2. Build with `npm run build`
3. Test with appropriate CLI command
4. Check logs and screenshots
5. Verify on live site (if not dry-run)

## Important Constraints

### Squarespace Limitations
- No direct API for content editing
- Editor UI may change without notice
- Must use browser automation
- Session management is critical

### Safety Constraints
- Never publish without user confirmation
- Always validate patches
- Log all operations
- Store artifacts for rollback

### Performance Considerations
- Crawler respects delays to avoid rate limiting
- Limit concurrent requests
- Cache browser sessions
- Minimize re-crawling

## Module Dependencies
Keep modules loosely coupled:
- `crawler` → utils only
- `browser` → config, utils only  
- `patch` → browser, utils
- `search` → utils only
- `index.ts` → all modules (orchestration)

## When Making Changes
- Maintain backward compatibility with existing patches
- Update schemas carefully (breaking changes require migration)
- Add comprehensive logging
- Update documentation
- Test with real Squarespace site

## Code Review Checklist
- [ ] Type safety maintained
- [ ] Error handling present
- [ ] Logging added
- [ ] Schema validation included
- [ ] Documentation updated
- [ ] Tested in dry-run mode
- [ ] Screenshots reviewed

## Resources
- MCP SDK: https://modelcontextprotocol.io
- Playwright: https://playwright.dev
- Squarespace: https://www.squarespace.com

---

This is an MVP with core functionality working. Future enhancements should maintain the same architectural principles while extending capabilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrew-loftis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/andrew-loftis)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
