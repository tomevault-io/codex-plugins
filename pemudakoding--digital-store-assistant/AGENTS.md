# KoalaStore WhatsApp Bot - AI Agent Instructions

## Project Overview
This is a **WhatsApp Bot for Digital Store Management** built with Node.js ES6 modules, Baileys library, and a sophisticated queue-based architecture. The bot handles customer service automation, product management, group moderation, and order processing for digital stores.

## Core Architecture Understanding

### Message Processing Pipeline
```
WhatsApp → MessageHandler → CommandHandler → Services → Response Queue → WhatsApp
```

All message processing uses **p-queue** for race condition prevention. Never bypass the queue system without `queueHelpers.safeAdd()` wrapper.

### Command Discovery System
Commands are **automatically discovered** from filesystem structure via `CommandRegistry.js`:
- Add new commands: Create `.js` file in appropriate `src/commands/{category}/` folder
- Metadata: Configure in `src/commands/registry/commandsConfig.js`
- Categories: `general`, `admin`, `owner`, `store`, `calculator`
- Hot reload: Use `reloadcommands` command (owner only)

### Context Object Pattern
Every command receives a rich context object with WhatsApp data, services, and utilities:
```javascript
export default async function myCommand(context, args) {
    const { from, sender, isOwner, isGroupAdmin, messageService, listManager } = context;
    // Always use context.messageService.reply() instead of direct client calls
}
```

## Essential Development Patterns

### Data Management via Managers
**Never** access JSON files directly. Always use manager classes:
```javascript
// ✅ Correct
const products = await context.listManager.getListDb();
await context.listManager.saveListDb(updatedProducts);

// ❌ Wrong - bypasses data consistency
const products = JSON.parse(fs.readFileSync('database/list.json'));
```

### Command Argument Parsing
Use pipe-separated arguments for complex data:
```javascript
// Command: addlist Product Name|Description here|25000|Electronics
const { args } = commandHandler.parseMultipleArgs(text, 4);
const [name, description, price, category] = args;
```

### Permission-Aware Operations
Always validate permissions before group operations:
```javascript
if (command.adminOnly && !context.isGroupAdmin) {
    return context.messageService.reply(from, "❌ Admin only command", msg);
}

// For bot group operations, check bot's admin status
const isBotAdmin = await context.groupService.isBotGroupAdmin(groupId);
```

### Queue-First Development
All async operations must use queue helpers:
```javascript
// Message sending
await queueHelpers.safeAdd(messageQueue,
    async () => messageService.reply(from, text, msg),
    async () => messageService.sendTextDirect(from, text) // fallback
);
```

### Group Metadata Validation
All group message sending automatically validates metadata before sending:
```javascript
// Automatically handled in MessageService - ensures group metadata exists
await messageService.sendText(groupId, "Message"); // ✅ Safe for groups
await messageService.reply(groupId, "Reply", msg); // ✅ Metadata validated

// Pattern used internally:
// await this.ensureGroupMetadata(jid); // Validates if jid.endsWith('@g.us')
```

### Group Message Encryption
Group messages automatically include encryption options for better compatibility:
```javascript
// Automatically applied for group messages (jid.endsWith('@g.us'))
const sendOptions = to.endsWith('@g.us') ? {
    ephemeralExpiration: 0,
    messageId: undefined, // Let Baileys generate the message ID
    ...options
} : options;

// All MessageService methods handle this automatically
await messageService.sendText(groupId, "Message"); // ✅ Encryption applied
await messageService.sendImage(groupId, buffer, "Caption"); // ✅ Encryption applied
```## Critical Developer Workflows

### Adding New Commands
1. Create file: `src/commands/{category}/commandname.js`
2. Add metadata: `src/commands/registry/commandsConfig.js`
3. Test with: `npm run dev` (auto-discovery active)
4. Hot reload: Send `reloadcommands` command as owner

### Development Setup
```bash
npm run dev              # Development with auto-restart
npm run pm2:start        # Production deployment
npm run pm2:logs         # Monitor logs
npm run clean:win        # Reset WhatsApp session (Windows)
```

### Debugging Commands
- `botstat` - System health and queue statistics
- `commandinfo` - Registry status and command metrics
- `resetqueue` - Emergency queue reset for stuck operations
- `reloadcommands` - Hot reload all commands

### PM2 Production Patterns
- **Non-interactive setup**: `pm2-windows.bat` for Windows deployment
- **Pairing code mode**: Set `USE_PAIRING_CODE=true` to avoid QR scanning
- **Memory monitoring**: Auto-restart at 1GB RAM usage
- **Log rotation**: Configured in `ecosystem.config.js`

## Project-Specific Conventions

### File Organization
```
src/
├── commands/{category}/     # Auto-discovered commands
├── handlers/               # MessageHandler, CommandHandler
├── services/               # Business logic (MessageService, GroupService)
├── models/                 # Data managers (ListManager, TestiManager)
├── utils/                  # Queue system, logger, helpers
└── config/                 # Settings, messages, constants
```

### Data Consistency Rules
- **AFK System**: Group-scoped (user can be AFK in different groups independently)
- **Welcome Messages**: Stored per-group in `WelcomeManager`
- **Product Keys**: Case-insensitive matching for customer inquiries
- **Order States**: Use `set_proses.json` and `set_done.json` for templates

### Error Handling Patterns
```javascript
// Command-level error handling
try {
    const result = await someOperation();
    return await context.messageService.reply(from, "✅ Success", msg);
} catch (error) {
    logger.error("Operation failed:", error);
    return await context.messageService.reply(from, "❌ Failed. Try again.", msg);
}
```

### Media Processing
- **Image products**: Upload via `MediaService.uploadImage()` then store URL
- **Size limits**: Keep images under 50KB for WhatsApp compatibility
- **Sticker creation**: Use `mediaService.createSticker()` for auto-conversion

## Integration Points

### WhatsApp Connection Management
- **Pairing code**: Preferred for headless deployment (`config.bot.usePairingCode`)
- **Session persistence**: Uses `sessionn/` folder (never delete in production)
- **Reconnection logic**: Exponential backoff with circuit breaker pattern
- **Group metadata validation**: All group messages automatically validate metadata before sending

### External Dependencies
- **Baileys 6.7.18**: WhatsApp Web API (breaking changes in newer versions)
- **p-queue**: Critical for message ordering and rate limiting
- **PM2**: Production process management (required for stability)
- **Pino**: Structured logging (JSON format for log aggregation)

### Database Pattern
JSON file-based storage with manager abstraction layer. Each data type has dedicated manager:
- `ListManager` - Product listings and store catalog
- `TestiManager` - Customer testimonials
- `AfkManager` - Group-scoped AFK status tracking
- `SewaManager` - Subscription/rental tracking

## Common Gotchas

1. **Queue Bypass**: Never call `client.sendMessage()` directly - always use `MessageService`
2. **Path Resolution**: `CommandRegistry` paths are sensitive - update `validCategories` when adding command categories
3. **Self-Message Loop**: Bot ignores messages from itself (`msg.key.fromMe`)
4. **Admin Validation**: Check both user admin status AND bot admin status for group operations
5. **Memory Leaks**: Large media processing can cause restarts - monitor with `botstat`

Always test new features with `npm run dev` first, then deploy with PM2 for production stability.

---
> Source: [pemudakoding/Digital-Store-Assistant](https://github.com/pemudakoding/Digital-Store-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-14 -->
