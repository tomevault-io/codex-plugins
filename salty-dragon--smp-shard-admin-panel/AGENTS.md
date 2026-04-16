# GitHub Copilot Instructions for SMP Shard Admin Panel

## 🎯 Project Overview
This is a **Minecraft SMP Server Administration Panel** built with **Next.js 15**, **TypeScript**, **Prisma**, and **TailwindCSS**. It provides a web interface for managing Minecraft servers via tmux sessions, file management, user authentication with 2FA, and activity logging.

---

## 🚨 CRITICAL CONVENTIONS

### 1. API Path Convention
**ALWAYS use the `/apanel44` base path for ALL routes and API calls:**

```typescript
// ✅ CORRECT
fetch('/apanel44/api/files')
f.../apanel44/dashboard"  // Next.js will add it, causing double prefix
```

**Key Rules:**
- API paths: `/apanel44/api/*`
- Page routes: Next.js handles the prefix automatically from `basePath` in `next.config.ts`
- Never hardcode `/apanel44` in `href` attributes in Link components
- Never put pages in `/pages/apanel44/` directory

### 2. Trailing Slash Requirement
All URLs MUST end with a trailing slash (configured in `next.config.ts`):
- `/apanel44/dashboard/` ✅
- `/apanel44/dashboard` ❌ (will redirect)

---

## 📁 Project Structure

```
smp-shard-admin-panel/
├── .github/
│   └── copilot-instructions.md    # This file
├── src/
│   ├── pages/              # Next.js pages (NOT in /pages/apanel44/)
│   │   ├── api/           # API routes (automatically prefixed with /apanel44/api)
│   │   │   ├── auth/      # NextAuth.js routes
│   │   │   ├── files/     # File management APIs
│   │   │   ├── server/    # Server control APIs (console, status)
│   │   │   ├── monitoring/ # Metrics and monitoring
│   │   │   └── permissions/ # User permission management
│   │   ├── index.tsx      # Landing page → /apanel44/
│   │   ├── login.tsx      # Login page → /apanel44/login/
│   │   └── dashboard.tsx  # Main dashboard → /apanel44/dashboard/
│   ├── components/        # Reusable React components
│   ├── lib/              # Utility functions and helpers
│   │   ├── prisma.ts     # Prisma client singleton
│   │   ├── middleware.ts  # Auth middleware (withAdmin, withSuperAdmin)
│   │   ├── console.ts    # Tmux/server console utilities
│   │   ├── fileUtils.ts  # File management utilities
│   │   └── permissions.ts # Permission definitions
│   ├── styles/           # Global styles and Tailwind config
│   └── types/            # TypeScript type definitions
├── prisma/
│   └── schema.prisma     # Database schema
└── scripts/              # Shell scripts for server management
```

---

## 🔐 Security Patterns

### Authentication & Authorization
1. **Middleware Protection:**
   ```typescript
   import { withAdmin, withSuperAdmin } from '@/lib/middleware';
   
   // Admin or Super Admin required
   export default withAdmin(handler);
   
   // Super Admin only
   export default withSuperAdmin(handler);
   ```

2. **Role Hierarchy:**
   - `SUPER_ADMIN`: Full access to everything
   - `ADMIN`: Access to most features except user management
   - `USER`: Read-only access (if implemented)

3. **2FA Support:**
   - Email OTP via SMTP
   - TOTP (Google Authenticator) support
   - Configured per user in database

### Input Sanitization
**ALWAYS sanitize user inputs** for file operations and command execution:

```typescript
import { sanitizeFilename, sanitizePath, sanitizeCommand } from '@/lib/fileUtils';
import { sanitizeSessionName } from '@/lib/console';

// File operations
const safeName = sanitizeFilename(userInput);
const safePath = sanitizePath(userInput);

// Command execution (tmux)
const safeCommand = sanitizeCommand(userInput);
const safeSession = sanitizeSessionName(sessionName);
```

### Path Traversal Prevention
```typescript
import { validateAndResolvePath, PLUGINS_DIR } from '@/lib/fileUtils';

// Validate paths ALWAYS stay within allowed directory
const fullPath = await validateAndResolvePath(userPath, PLUGINS_DIR);
```

### Command Whitelisting
```typescript
import { ADMIN_ALLOWED_COMMANDS } from '@/lib/console-constants';

// Only allow pre-approved Minecraft commands
if (!ADMIN_ALLOWED_COMMANDS.includes(baseCommand)) {
  throw new Error('Command not allowed');
}
```

---

## 🏗️ Architecture Patterns

### API Route Structure
```typescript
import { NextApiResponse } from 'next';
import { withAdmin, AuthenticatedRequest } from '@/lib/middleware';
import { logActivity } from '@/lib/activity';

async function handler(req: AuthenticatedRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    try {
      // Business logic here
      
      // Log activity
      await logActivity({
        userId: req.user.id,
        actionType: 'action_name',
        resource: 'resource_name',
        details: { /* JSON details */ },
        req,
      });
      
      return res.status(200).json({ data: result });
    } catch (error) {
      console.error('Error:', error);
      return res.status(500).json({ error: 'Internal server error' });
    }
  }
  
  return res.status(405).json({ error: 'Method not allowed' });
}

export default withAdmin(handler);
```

### File Upload Handling
```typescript
import { IncomingForm } from 'formidable';

export const config = {
  api: {
    bodyParser: false,  // Required for file uploads
  },
};

async function handler(req: AuthenticatedRequest, res: NextApiResponse) {
  const form = new IncomingForm({
    maxFileSize: 35 * 1024 * 1024,  // 35MB
    uploadDir: '/tmp',
    keepExtensions: true,
  });
  
  const { fields, files } = await new Promise((resolve, reject) => {
    form.parse(req, (err, fields, files) => {
      if (err) reject(err);
      else resolve({ fields, files });
    });
  });
}
```

### Prisma Usage
```typescript
import prisma from '@/lib/prisma';

// ALWAYS use the singleton instance
const users = await prisma.user.findMany();

// Use transactions for multi-step operations
await prisma.$transaction(async (tx) => {
  await tx.activityLog.create({ /* ... */ });
  await tx.user.update({ /* ... */ });
});
```

### Minecraft Server (Tmux) Interaction
```typescript
import { 
  tmuxSessionExists, 
  sendCommandAndCapture, 
  sanitizeCommand 
} from '@/lib/console';

// Check if server is running
const isRunning = await tmuxSessionExists('minecraft-shard-1');

// Send command and capture output
const output = await sendCommandAndCapture(
  'minecraft-shard-1',
  sanitizeCommand(userCommand),
  5000  // 5 second timeout
);
```

---

## 📝 Common Patterns

### Query Parameter Handling (Sub-paths)
```typescript
// API routes support optional 'path' query parameter for subdirectories
const relativePath = typeof req.query.path === 'string' ? req.query.path : '';

// GET /apanel44/api/files → root directory
// GET /apanel44/api/files?path=subfolder → subfolder
// GET /apanel44/api/files?path=folder/nested → nested folder
```

### Frontend API Calls with Path
```typescript
// List files in subdirectory
const response = await fetch(`/apanel44/api/files?path=${encodeURIComponent(currentPath)}`);

// File operations in subdirectory
await fetch(`/apanel44/api/files/${filename}?path=${encodeURIComponent(currentPath)}`);
```

### Error Handling Standard
```typescript
try {
  // Business logic
} catch (error) {
  console.error('Descriptive error message:', error);
  return res.status(500).json({
    error: 'Internal server error',
    message: 'User-friendly error description'
  });
}
```

### Activity Logging
```typescript
await logActivity({
  userId: req.user.id,
  actionType: 'create' | 'update' | 'delete' | 'execute' | 'list' | 'read',
  resource: 'files' | 'console' | 'users' | 'permissions',
  details: { /* Any JSON-serializable data */ },
  req,  // Automatically extracts IP address
});
```

---

## 🛠️ Technology Stack

### Core Framework
- **Next.js 15** (Pages Router, NOT App Router)
- **React 19**
- **TypeScript 5**
- **TailwindCSS 3**

### Database & ORM
- **Prisma** (PostgreSQL/MariaDB/MySQL)
- **NextAuth.js** for authentication

### Key Libraries
- **bcryptjs**: Password hashing
- **speakeasy**: TOTP 2FA
- **qrcode**: QR code generation
- **formidable**: File upload handling
- **nodemailer**: Email sending

### Server Interaction
- **tmux**: Minecraft server console management
- **child_process**: Shell command execution

---

## 🧪 Development Guidelines

### Type Safety
```typescript
// Always define types for request bodies and responses
interface UploadFileRequest {
  path?: string;
}

interface FileListResponse {
  files: FileInfo[];
  currentPath: string;
}
```

### Environment Variables
```typescript
// Access via process.env (defined in .env.local)
const sessionName = process.env.TMUX_SESSION_NAME || 'minecraft-shard-1';
const pluginsDir = process.env.PLUGINS_DIR || '/path/to/plugins';
```

Required environment variables:
- `DATABASE_URL`
- `NEXTAUTH_SECRET`
- `NEXTAUTH_URL`
- `SMTP_*` (for email)
- `TMUX_SESSION_NAME`
- `PLUGINS_DIR`
- `MINECRAFT_SERVER_DIR`

### Constants
```typescript
// Define constants in dedicated files
// lib/console-constants.ts
export const ADMIN_ALLOWED_COMMANDS = [
  'say', 'list', 'help', 'whitelist', // ... etc
];

// lib/fileUtils.ts
export const MAX_JAR_SIZE = 35 * 1024 * 1024;
export const EDITABLE_EXTENSIONS = ['.yml', '.yaml', '.json', '.txt'];
```

### Component Best Practices
```typescript
import { useCallback, useState, useEffect } from 'react';

// Use useCallback for functions passed to useEffect dependencies
const fetchData = useCallback(async () => {
  // Fetch logic
}, [dependency]);

// Proper cleanup in useEffect
useEffect(() => {
  const interval = setInterval(() => {}, 5000);
  return () => clearInterval(interval);
}, []);

// Accessibility attributes
<button
  onClick={handleClick}
  aria-label="Descriptive action"
  disabled={isLoading}
>
  {isLoading ? 'Loading...' : 'Action'}
</button>
```

---

## 📚 Key Documentation Files

When working on specific features, reference these documentation files:

- **CONSOLE_ARCHITECTURE.md**: Server console implementation details
- **PLUGIN_MANAGEMENT_IMPLEMENTATION.md**: File management system
- **SECURITY_REVIEW.md**: Security audit and best practices
- **TESTING_GUIDE.md**: Manual testing procedures
- **SERVER_MANAGEMENT_GUIDE.md**: Minecraft server operations

---

## 🚀 Quick Reference

### Adding a New API Route
1. Create file in `src/pages/api/`
2. Use `withAdmin` or `withSuperAdmin` middleware
3. Handle methods with `if (req.method === 'GET')`
4. Sanitize ALL user inputs
5. Log activities with `logActivity`
6. Return consistent error format
7. Document in README.md under API Routes section

### Adding a New Page
1. Create file in `src/pages/` (NOT `/pages/apanel44/`)
2. Use relative `href` in Links (e.g., `/dashboard`, not `/apanel44/dashboard`)
3. Protect with `getServerSideProps` if needed:
   ```typescript
   export const getServerSideProps = withAuth(async (context) => {
     // Check authentication
   });
   ```

### Database Schema Changes
1. Update `prisma/schema.prisma`
2. Run `npx prisma generate`
3. Run `npx prisma db push` (dev) or `npx prisma migrate dev` (prod)
4. Update TypeScript types if needed

---

## 🔍 Common Issues & Solutions

### Double Base Path (`/apanel44/apanel44/`)
**Problem:** URL becomes `/apanel44/apanel44/dashboard`
**Solution:** Never hardcode `/apanel44` in `href` or `Link` components. Next.js adds it automatically.

### Redirect Loops
**Problem:** Endless redirects between `/apanel44/dashboard` and `/apanel44/dashboard/`
**Solution:** Ensure `trailingSlash: true` in `next.config.ts` and Nginx is configured correctly.

### File Upload Not Working
**Problem:** Body parser interfering with multipart form data
**Solution:** Add `export const config = { api: { bodyParser: false } }` to route.

### Command Not Executing
**Problem:** Command fails silently or returns empty
**Solution:** 
1. Check tmux session exists with `tmuxSessionExists()`
2. Verify command is in `ADMIN_ALLOWED_COMMANDS`
3. Use `sendCommandAndCapture()` with appropriate timeout
4. Check tmux capture-pane output

---

## ✅ Code Quality Checklist

Before committing, ensure:
- [ ] All user inputs are sanitized
- [ ] Authentication middleware applied to protected routes
- [ ] Activity logged for state-changing operations
- [ ] TypeScript errors resolved (`npm run build`)
- [ ] ESLint warnings addressed
- [ ] API paths use `/apanel44` prefix
- [ ] Links use relative paths without `/apanel44`
- [ ] Error handling implemented with try-catch
- [ ] Console.errors have descriptive messages
- [ ] Accessibility attributes added (`aria-label`, `htmlFor`)

---

**Last Updated:** 2026-02-08 18:40:29
**Project:** SMP Shard Admin Panel
**Repository:** Salty-Dragon/smp-shard-admin-panel

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Salty-Dragon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
