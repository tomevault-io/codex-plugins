# GitHub Copilot Instructions for App-Lycan

> Repository-specific instructions for GitHub Copilot coding agent.

## Project Overview

**App-Lycan** is a privacy-first, browser-based CV and cover letter builder with AI assistance. Everything runs client-side — no backend, no database. All user data lives in `localStorage`. AI features require the user's own OpenRouter API key.

### Tech Stack

| Layer     | Technology                                    |
| --------- | --------------------------------------------- |
| Framework | Vue 3 (Composition API with `<script setup>`) |
| Build     | Vite 7                                        |
| State     | Pinia (Composition API style)                 |
| Styling   | Tailwind CSS v4 (dark mode via `dark:` prefix)|
| Icons     | Lucide Vue                                    |
| Rich Text | TipTap                                        |
| PDF       | html2pdf.js                                   |
| AI        | OpenRouter SDK (`@openrouter/sdk`)            |

## Branch Strategy & Workflow

| Branch               | Purpose                                                       |
| -------------------- | ------------------------------------------------------------- |
| `master`             | Stable released code. Only updated via merges from `develop`. |
| `develop`            | Integration branch for WIP. All feature/fix PRs target this. |
| feature/fix branches | Created from `master`; PR back to `develop`.                  |

**Workflow for any change:**
1. Branch off `master`.
2. Commit using [Conventional Commits](https://www.conventionalcommits.org/) format.
3. Open a PR **targeting `develop`**, never `master` directly.
4. CI (lint + unit + e2e tests) runs on every PR to `develop` or `master`.
5. Releases are triggered automatically by merging `develop` into `master`.

Do **not** commit directly to `master` or `develop`.

## Commit Messages

Every commit must follow [Conventional Commits](https://www.conventionalcommits.org/) — enforced by `commitlint`. Non-conforming commits will fail CI.

```
<type>(<optional scope>): <short description>
```

| Type       | Description                  | Release bump       |
| ---------- | ---------------------------- | ------------------ |
| `feat`     | New feature                  | **minor** (1.x.0)  |
| `fix`      | Bug fix                      | **patch** (1.0.x)  |
| `perf`     | Performance improvement      | **patch** (1.0.x)  |
| `revert`   | Revert a previous commit     | **patch** (1.0.x)  |
| `docs`     | Documentation only           | none               |
| `style`    | Formatting, whitespace       | none               |
| `refactor` | Code restructure, no logic   | none               |
| `test`     | Tests only                   | none               |
| `ci`       | CI/CD config                 | none               |
| `chore`    | Build, deps, housekeeping    | none               |

**Scope** (optional): `ai`, `cv`, `cover-letter`, `workspace`, `settings`, `profile`, `router`, `store`, `ui`, `deps`.

**Rules:**
- Imperative mood, lowercase first letter, no trailing period.
- Subject line ≤ 72 characters.
- Never use vague messages like `fix stuff`, `WIP`, or `update`.

## Commands

```bash
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build
npm run lint         # ESLint with auto-fix
npm run format       # Prettier format src/
npm run test:unit    # Run unit tests (Vitest + jsdom)
npm run test:e2e     # Run end-to-end tests (Playwright + Chromium)
```

## Key Directories

```
src/
├── components/          # Reusable Vue components
├── views/               # Route-level page components
├── stores/              # Pinia state stores (source of truth)
├── services/            # Business logic & external APIs
├── composables/         # Vue composables (hooks)
└── router/              # Vue Router configuration
```

## Code Patterns

### Vue Component Structure

```vue
<script setup>
import { ref, computed } from 'vue'
import { useYourStore } from '../stores/yourStore'

const store = useYourStore()
const localState = ref(null)
const derivedValue = computed(() => /* ... */)

const handleAction = () => { /* ... */ }
</script>

<template>
  <!-- Template here -->
</template>
```

> **Important:** The `@` path alias is **not** configured — always use relative imports.

### Store Pattern (Pinia Composition API)

```javascript
export const useYourStore = defineStore('yourStore', () => {
    const state = ref(initialValue)
    const derived = computed(() => /* ... */)
    const action = () => { /* mutate state */ }
    return { state, derived, action }
})
```

All stores follow the same localStorage persistence pattern: load on init, `watch(state, persist, { deep: true })`.

### AI Streaming Pattern

```javascript
const generator = streamChat(messages, { model, apiKey })
for await (const chunk of generator) {
    if (chunk.type === 'content') {
        content += chunk.content
    } else if (chunk.type === 'tool_call') {
        // Handle tool call
    }
}
```

## State Management

| Store           | localStorage key           | Purpose                                  |
| --------------- | -------------------------- | ---------------------------------------- |
| `settings`      | `app-lycan-settings`       | API key, model preferences               |
| `userProfile`   | `app-lycan-user-profile`   | User's professional info for AI context  |
| `systemPrompts` | `app-lycan-system-prompts` | Custom prompts per AI command type       |
| `chat`          | `app-lycan-chat-history`   | Unified AI chat sessions across views    |
| `workspace`     | `workspaces`               | Job application containers + AI context  |
| `cv`            | `cvData-{uuid}`            | CV document content                      |
| `cvMeta`        | `cv-list`                  | CV metadata index                        |
| `coverLetter`   | `coverLetters`             | Cover letter content                     |

## AI Services

| File               | Purpose                               | Key Exports                                                                        |
| ------------------ | ------------------------------------- | ---------------------------------------------------------------------------------- |
| `ai.js`            | OpenRouter SDK wrapper                | `streamChat()`, `streamWithTools()`, `chatWithTools()`, `fetchAvailableModels()`   |
| `aiToolkit.js`     | Tool definitions for function calling | `AI_TOOLS`, `executeToolCall()`, `setupToolHandlers()`                             |
| `aiCommands.js`    | Slash command implementations         | `performCvAiAction()`, `executeAiCommand()`                                        |

## Routing

```
/                                              → WorkspaceDashboard
/workspace/:workspaceName                      → Dashboard (per-workspace)
/workspace/:workspaceName/edit/:name           → DocumentEditor (CV)
/workspace/:workspaceName/cover-letter/:name   → DocumentEditor (cover letter)
/profile                                       → UserProfile
/settings                                      → Settings
```

Route guards validate workspace/document existence and redirect to `/` if not found.

## Testing

```
src/test/setup.js          # Global Vitest setup (localStorage mock, crypto.randomUUID, Pinia reset)
src/test/factories.js      # Data factories for test data
src/stores/__tests__/      # Store unit tests
src/services/__tests__/    # Service unit tests
e2e/                       # Playwright end-to-end tests
```

**Unit test conventions:**
- Use relative imports — `@` alias is not configured in tests
- Use factories from `src/test/factories.js` for test data
- `setActivePinia(createPinia())` + `localStorage.clear()` run automatically before each test

**E2E conventions:**
- Call `await clearAppData(page)` in `test.beforeEach` for localStorage isolation
- Prefer `getByRole()` and `getByPlaceholder()` over CSS class selectors
- Use `exact: true` with `getByText()` to avoid strict-mode violations

**Key testing gotchas:**
- `workspace.js` always creates "My Workspace" on init — tests expecting an empty store must reset manually
- Mocking `ai.js` requires ALL exports (`RECOMMENDED_MODELS`, `WEB_SEARCH_MODELS`, etc.) — partial mocks cause Vitest to throw
- Pinia `watch()` fires asynchronously — add `await nextTick()` before asserting `localStorage.setItem`

## Key Gotchas

- **`toRaw()`**: Use when passing Pinia reactive store data to external APIs (OpenRouter SDK)
- **localStorage limits**: ~5–10MB; large CV collections may hit this
- **Tool calling**: Not all OpenRouter models support it; check model capabilities before using `streamWithTools`
- **Web search**: Enabled per-model via `plugins: [{ id: 'web', max_results: 5 }]`; supported by Anthropic, OpenAI, Perplexity, X-AI providers only
- **Path aliases**: `@` is **not** configured in `vite.config.js` — always use relative imports

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aruizrab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/aruizrab)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
