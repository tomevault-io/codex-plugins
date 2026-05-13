# PetBhai - AI Coding Agent Instructions

## Project Overview
PetBhai is a hybrid web application for pet care (shop, vet services, adoption, community). It consists of a **React (Vite)** frontend and an **Express/Node.js** backend.

## Tech Stack
- **Frontend**: React 18, TypeScript, Tailwind CSS, Vite, React Router (HashRouter).
- **Backend API**: Node.js, Express, TypeScript, Zod (validation), JWT (auth), @google/genai.
- **State Management**: React Context (\contexts/\) + \useReducer\ + \localStorage\.
- **Testing**: Jest + React Testing Library (Unit/Component tests).
- **Deployment**:
  - Frontend: GitHub Pages (static).
  - Backend: Vercel/Node environment (supplies \/api\ endpoints).
  - **Persistence**: File-based JSON Database (Backend) + localStorage (Frontend Cart/Auth).

## Critical Rules: NO MOCKS
- **"Real Data" Policy**: The application must behave like a production system.
- **Backend First**: All data must come from the Backend API.
- **No Frontend Fallbacks**: Do NOT use mock data fallbacks in \catch\ blocks in React components/contexts. If the API fails, display a proper error state or notification. This ensures we detect backend issues immediately.
- **Persistence**: The backend must persist data modifications (orders, users, inventory) to disk (JSON DB) so changes survive server restarts.
- **Mobile First**: The website should focus mainly on mobile users. Default to mobile-first layouts, tap targets, safe-area handling, and form UX before enhancing tablet or desktop views.

## Architecture & Data Flow

### 1. Hybrid State & Data Patterns
- **Context-First**: Logic resides in \contexts/\.
  - **AuthContext**: Syncs with Backend API (\/auth/login\, \/auth/signup\) using JWT. Falls back to purely local/mock for \socialLogin\.
  - **CartContext**: Purely client-side with \localStorage\ persistence (\petbhai_cart_items\).
  - **Data Providers**: \ProductContext\, \VetContext\ etc., MUST fetch from API.
- **Persistence**: Hybrid approach. Auth tokens and cart items are stored in \localStorage\.
- **API Communication**:
  - Frontend uses relative \/api\ paths.
  - Development: \ite.config.ts\ proxies \/api\ -> \http://localhost:5000\.
  - Production: \API_URL\ env var points to the live backend.

### 2. Backend Structure (\ackend/\)
- **Location**: All server-side logic is in \ackend/\.
- **Core Routes**: \src/routes/\ (\uth\, \i\, \product\, \et\, etc.).
- **Data Source**: Use \src/db.ts\ which manages the persistent JSON store. DO NOT import from \mockData.ts\ directly in routes.
- **Security**: Uses \helmet\, \cors\, and \
ateLimiter\.
- **AI Service**: The backend handles Gemini API calls (\@google/genai\) to protect secrets.
  - Frontend \services/geminiService.ts\ delegates to \POST /api/ai/*\.

### 3. Component Architecture
- **Pages**: Lazy-loaded in \App.tsx\ (\React.lazy\).
- **Shared Components**: \components/\ (atomic UI elements).
- **Skeleton Loading**: Use \<PawHeartLoader>\ or \Skeletons.tsx\ components.
- **Icons**: Custom SVG components in \components/icons.tsx\.

## Developer Workflows

### Running the Project
- **Full Stack Dev**: \
pm run dev:full\ (Recommended: runs React dev server + Backend watch).
- **Frontend Only**: \
pm run dev\ (Vite on port 3000).
- **Backend Only**: \
pm run server\ (Express on port 5000).

### Testing
- **Command**: \
pm test\ (Runs Jest).
- **Structure**:
  - \__tests__/components/\: UI component tests.
  - \__tests__/contexts/\: State logic tests.
- **Requirement**: Write tests for new utility functions and complex context logic.

### Linting & Code Quality
- **Strict Linting**: \
pm run lint\ enforces 0 warning policy.
- **HTML Validation**: \
pm run check:html\ (runs in CI).
- **Formatting**: Prettier is standard.

## Implementation Guidelines

### Adding New Features
1. **Types First**: Define interfaces in \	ypes.ts\.
2. **Backend (if needed)**: Add routes in \ackend/src/routes/\ and controllers.
3. **Frontend Service/Context**: Update \contexts/\ or create a service to call the API.
4. **UI Component**: Build components in \components/\ using Tailwind.
5. **Page**: Add lazy-loaded route in \App.tsx\.

### AI Integration
- **Do NOT** call Gemini API directly from React components.
- **DO** use \services/geminiService.ts\, which calls the Backend.
- **Backend Implementation**: \ackend/src/controllers/aiController.ts\ handles the prompt/response.

### Common Conventions
- **Naming**: PascalCase for Components (\VetCard.tsx\), camelCase for helpers (\alidateInput.ts\).
- **State**: Prefer \Context\ for global data, \useState\ for local component UI state.
- **Validation**: Use \zod\ on the backend, custom validators (\lib/validations.ts\) on frontend.
- **Environment**: Access vars via \import.meta.env\ (Frontend) or \process.env\ (Backend).

## File Map
- \App.tsx\: Main routing and provider wrapping.
- \	ypes.ts\: Global TypeScript definitions.
- \constants.ts\: DEPRECATED for data. Use only for configuration/static-labels. 
- \ackend/\: Express server source.
- \contexts/\: React Context definitions.
- \__tests__/\: Jest test suites.

---
> Source: [myselfrezaul7/PetBhai](https://github.com/myselfrezaul7/PetBhai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
