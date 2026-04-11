# GitHub Copilot - Project Instructions

## Project Documentation

Full project context is available in [`docs/dev/`](../docs/dev/):

- **[docs/dev/README.md](../docs/dev/README.md)** - Overview and navigation guide
- **[docs/dev/architecture.md](../docs/dev/architecture.md)** - Tech stack, file structure, routing
- **[docs/dev/conventions.md](../docs/dev/conventions.md)** - Code style and patterns
- **[docs/dev/roadmap.md](../docs/dev/roadmap.md)** - Planned features
- **[docs/dev/known-issues.md](../docs/dev/known-issues.md)** - Current limitations

## Project Overview

**Firebase Barillets Editor** - A web app for creating and managing "barillets" (improvisation theater theme collections).

### Tech Stack

- Vue 3 + TypeScript (Composition API with `<script setup>`)
- Vite 7.x build tool
- Firebase (Firestore + Authentication)
- Tailwind CSS + shadcn-vue components
- Vue Router 4.x

### Key Concepts

- **Barillet**: Collection of 18 themes for an improv show
- **Theme**: Individual game with type (Mixte/Comparée), title, participation, category, duration
- **User-facing language**: French (all UI text must be in French)

## Code Style Guidelines

### Vue Components

```vue
<script setup lang="ts">
import { ref } from 'vue';

interface Props {
  title: string;
}

const props = defineProps<Props>();
const emit = defineEmits<{
  save: [];
}>();
</script>

<template>
  <div>{{ title }}</div>
</template>
```

### Naming Conventions

- **Components**: PascalCase (`ThemeCard.vue`)
- **Composables**: camelCase with `use` prefix (`useBarillets.ts`)
- **Types**: PascalCase (`Barillet`, `Theme`)

### Import Order

1. Vue imports
2. Third-party libraries
3. Type imports
4. Local composables
5. Local components

## Critical Patterns

### Date Handling

**IMPORTANT**: The app uses JavaScript `Date` objects internally, converted at the Firestore boundary.

```typescript
// In composables/useBarillets.ts
// Reading: Firestore Timestamp → JavaScript Date
const convertTimestampToDate = (data: any): Barillet => ({
  ...data,
  date: data.date?.toDate() || null,
  createdAt: data.createdAt?.toDate() || null,
  updatedAt: data.updatedAt?.toDate() || null,
});

// Writing: JavaScript Date → Firestore Timestamp
const convertDateToTimestamp = (barillet: Partial<Barillet>) => ({
  ...barillet,
  date: barillet.date ? Timestamp.fromDate(barillet.date) : null,
  updatedAt: Timestamp.now(),
});
```

**In Vue components**: Always use `Date` objects, never Firestore `Timestamp`.

### Error Handling

```typescript
try {
  await updateBarillet(id, data);
  // Success handling
} catch (error) {
  console.error('Failed to update:', error);
  // User-facing error in French
  alert('Erreur lors de la sauvegarde');
}
```

### Form Validation

- Use `@blur` for real-time validation
- Provide French error messages
- Validate before Firestore writes

## File Structure

```
src/
├── views/           # Route-level components (HomeView.vue, LoginView.vue)
├── components/      # Reusable UI (ThemeCard.vue, BarilletCard.vue)
├── composables/     # Shared logic (useAuth.ts, useBarillets.ts)
├── types/           # TypeScript definitions (barillet.ts)
├── router/          # Vue Router config
└── firebase-app.ts  # Firebase initialization
```

## Development Commands

```bash
npm run dev         # Development server
npm run build       # Production build
npm run type-check  # TypeScript validation
npm run format      # Prettier formatting
npm run lint:fix    # ESLint auto-fix
```

## Common Tasks

### Creating a New Component

1. Use `<script setup lang="ts">` syntax
2. Define props with TypeScript interface
3. Use French for all user-facing text
4. Follow Tailwind CSS for styling
5. Export component from `.vue` file

### Working with Firestore

- Use `useBarillets()` composable for CRUD operations
- Dates: Always `Date` objects in components
- Real-time updates: `onSnapshot` listener active in composable
- Validation: Client-side before write, types enforce structure

### Adding a Route

1. Create view in `src/views/`
2. Add route to `src/router/index.ts`
3. Set `meta.requiresAuth` for protected routes
4. Set `meta.requiresLayout` for pages using AppLayout

## Resources

- [Vue 3 Docs](https://vuejs.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Firebase Docs](https://firebase.google.com/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

**For detailed architecture, patterns, and conventions**, see [docs/dev/](../docs/dev/).

**Last Updated**: 2025-10-17

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tchapeaux)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tchapeaux)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
