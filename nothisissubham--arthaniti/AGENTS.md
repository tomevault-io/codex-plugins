# Copilot Instructions for Arthaniti Platform

## Project Overview
**Arthaniti** is a Bengali-language financial literacy platform built with React + TypeScript + Vite. It educates users about savings, investments, and financial stability through an interactive learning experience. The app uses Google's Gemini API to power an AI mentor ("Boro Kaku") who provides personalized financial guidance.

## Architecture & Key Patterns

### State Management
- **App-level state**: Single `AppState` type ('intro' | 'hero' | 'learning') managed in [App.tsx](../App.tsx)
- Navigation between states via `setState` callbacks passed to child components
- No external state management; all state is lifted to App root

### Component Structure
- **Page flow**: IntroSlides → Hero → LearningFlow (sequential state transitions)
- **Persistent UI**: Navbar and Footer render conditionally (not during 'intro' state)
- Components are functional with React.FC<Props> pattern
- Props are passed as callback functions (e.g., `onStart`, `onComplete`)

### AI Integration (Gemini API)
- Service file: [services/gemini.ts](../services/gemini.ts)
- Uses `@google/genai` SDK with environment variable `VITE_GEMINI_API_KEY` (client-side, Vite-prefixed)
- System instruction sets tone: calm, conversational Bengali mentoring (no formal jargon)
- Response accessed via `.text` property (not a method) on response object
- ElderMentor component manages message history and feeds it to `getElderResponse()` for context

### Styling Approach
- **Tailwind CSS** for utilities (in HTML classnames)
- **Inline styled components** for complex animations (typewriter, hover effects, hexagon floats)
- Color palette: stone-900, stone-800, black backgrounds; white/stone text
- Dark theme dominates; light sections (stone-50) used for contrast

## Critical Workflows

### Development
```bash
npm install          # Install dependencies (React, Google GenAI, dev tools)
npm run dev          # Start Vite dev server (hot reload)
npm run build        # Compile TypeScript + build static bundle
npm run preview      # Preview production build locally
```

### Environment Setup
- Create `.env.local` with `VITE_GEMINI_API_KEY=<your-api-key>`
- Vite automatically exposes vars prefixed with `VITE_` to client code
- Missing API key gracefully handled; mentor responds with fallback message

## Language & Cultural Conventions

- **Primary language**: Bengali (all user-facing text)
- **Tone**: Warm, familial, conversational ("Boro Kaku" = wise elder uncle)
- **Financial concepts explained via stories**, not technical jargon
- **Educational content** in [components/LearningFlow.tsx](../components/LearningFlow.tsx) uses highlight spans (`highlight-target`, `highlight`) for key ideas
- **Quiz system** reinforces learning with immediate explanations

## Type Definitions
All types in [types.ts](../types.ts):
- `Message`: { role: 'user'|'model', text: string } (chat history)
- `Chapter`: Includes id, title, content (ReactNode), quiz object
- `Quiz`: question, options array, explanation
- `Lesson`: Metadata for learning modules (title, description, readTime, image)

## Key Files & Their Responsibility

| File | Purpose |
|------|---------|
| [App.tsx](../App.tsx) | Root component; manages app state flow; renders conditional sections |
| [components/Hero.tsx](../components/Hero.tsx) | Hero section with typewriter animation and call-to-action |
| [components/IntroSlides.tsx](../components/IntroSlides.tsx) | Splash screen with timed animations |
| [components/LearningFlow.tsx](../components/LearningFlow.tsx) | Chapter content + interactive quizzes |
| [components/ElderMentor.tsx](../components/ElderMentor.tsx) | AI chat interface; manages message state & API calls |
| [services/gemini.ts](../services/gemini.ts) | Gemini API integration; system prompt + response handling |
| [types.ts](../types.ts) | Central type definitions for all data structures |

## Common Development Tasks

**Adding a new lesson chapter**: Define object in CHAPTERS array in [LearningFlow.tsx](../components/LearningFlow.tsx) with content (ReactNode) and quiz (QuizOption[]). Content uses `.highlight-target` and `.highlight` spans.

**Modifying mentor behavior**: Edit `SYSTEM_INSTRUCTION` in [gemini.ts](../services/gemini.ts). Keep tone warm and conversational; avoid financial jargon.

**Styling changes**: For simple utility changes, use Tailwind classes. For animations (fade-ins, hovers), add inline `<style>` blocks with CSS (see Hero.tsx for examples).

**Testing environment variables**: Ensure `.env.local` exists. If API key missing, mentor falls back gracefully (no crashes).

## Dependencies
- **react** (v19): UI framework
- **@google/genai** (v1.34): Gemini API client
- **typescript**: Type safety
- **vite**: Build tool with fast HMR
- **tailwindcss**: Implicit via Tailwind integration

## Non-Standard Patterns
- Inline styles for complex animations (not external CSS files) to keep component logic self-contained
- Components directly define their content arrays (CHAPTERS, LESSONS) rather than fetching from API
- Quiz results not persisted; purely in-session learning

---
> Source: [NoThisIsSubham/Arthaniti](https://github.com/NoThisIsSubham/Arthaniti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-06 -->
