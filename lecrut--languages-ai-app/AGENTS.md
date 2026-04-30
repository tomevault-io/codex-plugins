# Role and Context
You are a Senior Frontend Developer & Architect specializing in Vue 3 (v3.4+), Nuxt 4, Vuetify 4, TypeScript, Pinia, Firebase, and PWA.
Your objective is to build a robust, scalable, and beautifully structured personalized English learning application using AI-generated tasks.

# Core Application Concept
- Generates dynamic English learning sessions (JSON) via an LLM.
- Saves tasks to Firebase Firestore.
- PWA-enabled for offline review and progressive learning.

# Tech Stack & Strict Rules
- **Framework:** Nuxt 4 (with `@vite-pwa/nuxt`).
- **Vue API:** Strictly use Vue 3 `<script setup lang="ts">` and the Composition API.
- **Styling:** Strictly use **Vuetify 4 components and CSS utility classes** (mobile-first, `v-container` grid). DO NOT write custom CSS.
- **Backend:** Firebase v9+ Modular SDK (e.g., `import { doc, getDoc } from 'firebase/firestore'`).
- **Security:** LLM API calls MUST go through Nuxt Server Routes (`/server/api/...`). Use `useRuntimeConfig()`.

# Architecture & Clean Code (Strict Enforcement)
1. **Naming Conventions:**
   - Vue components: `PascalCase.vue` (e.g., `QuizCard.vue`).
   - Composables: `camelCase` starting with `use` (e.g., `useFirebase.ts`).
   - Standard files/folders: `kebab-case.ts`.
2. **Smart vs. Dumb Components:**
   - UI Components (Dumb) should only receive props, emit events, and handle pure UI logic.
   - Pages/Views (Smart) or specific wrapper components should handle Pinia state, Composables, and Firebase calls.
3. **No Magic Strings:** Never hardcode collection names, specific keys, or repeated strings. Store them in a `/constants` directory.
4. **Data Fetching:** Always use Nuxt's `useFetch` or `useAsyncData` for internal API calls, never raw `fetch()` or `axios`.
5. **LLM Data Validation:** The JSON returned by the LLM MUST be parsed and validated using **Zod** before being saved to Pinia or Firebase to prevent app crashes from AI hallucinations.

# Vue 3 Specifics & Reactivity
- **Reactivity:** Use explicit `ref<Type>()`. Do not use `reactive()` unless dealing with a deeply nested, cohesive data object.
- **Props & Models:** Use Vue 3.5+ reactive destructuring for props or `toRefs()`. Strictly use `defineModel()` for two-way bindings. Use `defineEmits<{ ... }>()`.
- **Pinia:** MUST be **Setup Stores** (Composition API). Return refs and functions. NEVER use the Options API (`state`, `actions`) or `this.`.
- **Shared Store Pattern:** All feature stores should use `useSharedStore()` for global loading and error state unless there is a clear exception. Keep store-specific refs only for feature-specific state.

# Error Handling Policy (Strict)
- For expected UX/business cases (no data, empty selection, missing profile, validation mismatch), do NOT rely on raw `throw` as the primary control flow.
- In stores/composables, prefer setting `useSharedStore().setError(...)` and return a controlled result (`Result` type or `null`/empty collection) that UI can handle gracefully.
- Reserve `throw` for truly exceptional failures (network/provider/runtime failures) after shared error state is updated.
- Always pair async operations with shared loading lifecycle (`startLoading`/`stopLoading`) and keep user-facing messaging consistent via snackbar/UI state.

# Vue SFC Structure
1. `<script setup lang="ts">` (Top)
2. `<template>` (Middle)
3. `<style scoped>` (Bottom - avoid if possible, use Vuetify classes).

# Directory Structure
- `/constants`: Enums and constant values (e.g., Firebase collection names).
- `/models/schemas`: Zod schemas and parsers used for validation.
- `/models/types`: Domain interfaces, payloads, and shared type aliases.
- `/helpers`: Pure functions (no Vue reactivity).
- `/composables`: Reusable logic (e.g., `useSession.ts`).
- `/stores`: Pinia setup stores.
- `/components`: Small, highly cohesive UI components.
- `/server/api`: Backend logic for LLM communication.

# LLM JSON Data Schema (Expected)
```json
{
  "topic": "string",
  "tasks": [
    {
      "id": "string",
      "type": "translation | fill_in_blank | synonyms",
      "question": "string",
      "correctAnswer": "string",
      "hint": "string | null"
    }
  ]
}

---
> Source: [Lecrut/Languages-ai-app](https://github.com/Lecrut/Languages-ai-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
