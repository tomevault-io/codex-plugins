<!-- Custom instructions for Outlook AI Assistant project -->

## Project Overview
This is an Outlook Add-in that provides AI-powered email, calendar, and task management. Built with React, TypeScript, Office.js, and Express.

## Tech Stack
- **Frontend**: React 18, TypeScript, Fluent UI, Zustand
- **Backend**: Node.js, Express, TypeScript
- **Office Integration**: Office.js API
- **AI Providers**: OpenAI GPT-4, Anthropic Claude

## Key Files
- `manifest.xml` - Office Add-in manifest (defines permissions and entry points)
- `src/client/` - React frontend application
- `src/server/` - Express API server
- `src/shared/types.ts` - Shared TypeScript interfaces

## Development Guidelines

### Office.js API
- Always check `Office.context.mailbox.item` before accessing email properties
- Use async patterns with `getAsync()` methods
- Handle cases where Office.js is not initialized (development mode)

### AI Integration
- Never expose API keys in client-side code
- All AI calls go through the backend server
- Provide fallback/mock responses for development

### UI/UX
- Use Fluent UI components for consistency with Office apps
- Keep the task pane compact (limited width)
- Support both light and dark themes

### Automation Rules
- Rules are stored in memory (implement persistent storage for production)
- Evaluate conditions in order of priority
- Log all rule triggers for debugging

## Running the Project
```bash
npm install
npm run dev
```

## Building for Production
```bash
npm run build
npm start
```

## Testing with Outlook
1. Sideload the manifest.xml in Outlook
2. The add-in appears in the ribbon when reading/composing emails
3. Click the AI Assistant button to open the task pane

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rrrr022) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
