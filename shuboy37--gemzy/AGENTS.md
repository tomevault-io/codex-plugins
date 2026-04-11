# Copilot Instructions for Gemzy

## Project Overview

Gemzy is a full-stack AI chat application that integrates multiple LLM providers (Google Gemini and Groq/Llama) with file upload capabilities. The project follows a client-server architecture with a React TypeScript frontend and Express.js backend.

## Architecture & Key Patterns

### Backend Structure (`geminiProject/backend/`)

- **Middleware-first routing**: All requests flow through `upload.middleware.js` → `model.middleware.js` → controllers
- **Intelligent model detection**: `detectModel` middleware uses Gemini to analyze prompts for image generation intent
- **Service layer pattern**: Controllers delegate to services (`geminiSdkService.ts`, `groqServices.ts`, `imageServices.ts`)
- **Client utilities**: Separate client setup files (`geminiClient.ts`, `groqClient.ts`) with configured instances

### Frontend Structure (`geminiProject/frontend/`)

- **Radix + Lucide ecosystem**: UI built with `@radix-ui` primitives, `lucide-react` icons, and custom components
- **Motion animations**: Uses `framer-motion` for sophisticated animations in placeholders component
- **Layout wrapper pattern**: `DarkRadialLayout` component provides gradient background using CSS-in-JS
- **Conditional state management**: Complex boolean state interactions (`isOpen`, `isHoverable`) with conditional event handlers

## Development Workflow

### Starting the development environment:

```bash
# Backend (runs on port 3000)
cd geminiProject/backend && pnpm dev

# Frontend (runs on port 5173)
cd geminiProject/frontend && pnpm dev
```

### Key dependencies:

- Backend: `@google/genai`, `groq-sdk`, `multer`, `tsx` with nodemon
- Frontend: Vite, React 19, Radix UI, Tailwind CSS, `class-variance-authority`

## Critical Integration Points

### Smart Model Selection Logic

```typescript
// Backend automatically switches models based on:
- PDF URLs detected via regex → forces Gemini
- File uploads with Llama model → forces Gemini
- Gemini analyzes text for image generation intent → switches to image generation model
```

### Dual Response Patterns

- **Gemini**: Returns JSON with `response`, `imageDataSrc`, `model`, `textWithPic` fields
- **Groq**: Streams NDJSON with `{type: "delta", content}` and `{type: "meta", model}`

### File Processing Pipeline

1. `multer` → `upload.middleware` → file type detection
2. Images vs PDFs handled differently (immediate vs. processing state polling)
3. Google File API with state monitoring for PDFs

## Component Patterns

### Conditional UI with Template Literals:

```tsx
className={`${isOpen ? "w-60" : "w-14"} h-screen relative bg-neutral-900/85`}
```

### Radix Composition Pattern:

```tsx
// All UI components follow Radix primitive + forwardRef pattern
const Component = React.forwardRef<ElementRef, Props>(
  ({ className, ...props }, ref) => (
    <Primitive.Component className={cn("base-styles", className)} {...props} />
  )
);
```

### State-Driven Event Handling:

```tsx
// Conditional event handlers based on component state
onMouseEnter={!isOpen ? () => setIsHoverable(true) : undefined}
```

### Dark Mode & Styling:

- Use `dark:` prefixes consistently
- `dark:invert` for SVG color switching
- Neutral color palette (`neutral-900/85`, `neutral-800`)
- `cn()` utility combines `clsx` and `tailwind-merge`

## Environment & Configuration

- Backend env in `src/config/.env`: `GROQ_API_KEY_LLAMA3_8B`, `GEMINI_APIKEY`
- TypeScript with ES modules (`"type": "module"`)
- Both projects use `pnpm` with `packageManager` field

## Testing & Debugging

- Backend logs model selection and file processing states
- Frontend console logs FormData and streaming chunks
- Network tab shows different response formats (JSON vs NDJSON streams)
- File processing includes polling with 5-second intervals

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuboy37)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/shuboy37)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
