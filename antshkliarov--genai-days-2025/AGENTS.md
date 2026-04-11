# Copilot Instructions for Gen AI Days 2025 Demo

## Project Overview
This is a **TodoMVC demonstration project** showcasing AI coding agent exercises with a React app and Playwright testing framework. The codebase has two main parts:
- `todomvc-react/`: A React TodoMVC implementation using useReducer pattern
- `excercises/`: Structured Playwright test exercises for AI agent training

## Architecture Patterns

### React App (`todomvc-react/`)
- **State Management**: Uses `useReducer` with action constants in `src/todo/constants.js`
- **Component Structure**: Header → Main → Footer pattern with shared `dispatch` prop
- **Build System**: Webpack with separate dev/prod configs
- **ID Generation**: Custom nanoid implementation in `reducer.js` (not external dependency)

### Test Structure (`excercises/`)
- **Page Object Model**: Centralized in `fixtures/todo-page.js` with reusable methods
- **Test Data**: Shared constants in `data/test-constants.js` (use `TODO_ITEMS` array)
- **Exercise Pattern**: Each numbered folder represents a structured AI exercise with `input.md`, `prompts/`, and implementation files

## Critical Workflows

### Development Commands
```bash
# React app
cd todomvc-react
npm run dev          # Development server (webpack-dev-server)
npm run serve        # Production build server on :7002

# Testing  
cd excercises
npx playwright test  # Run tests against localhost:8080
```

### Test Execution Context
- **Base URL**: Tests expect app running on `http://localhost:8080`
- **Browser**: Chromium only (see `playwright.config.js`)
- **Page Object**: Always use `todoPage` fixture, never direct page interactions

## Project-Specific Conventions

### Test Writing
- **Use shared constants**: Import `TODO_ITEMS` from `data/test-constants.js`
- **Page Object pattern**: All DOM interactions through `TodoPage` class methods
- **LocalStorage validation**: Tests include `checkTodosInLocalStorage()` assertions
- **Test structure**: Follow the 4-step pattern in prompts: Analyze → Plan → Implement → Validate

### React Component Patterns
- **No external state management**: Pure useReducer, no Redux/Zustand
- **Action dispatch**: Pass `dispatch` down through props, never use context
- **CSS**: Single `app.css` file, no CSS modules or styled-components
- **Testing**: Uses `data-testid` attributes (e.g., `todo-item`, `todo-title`, `todo-count`)

### Exercise Structure
```
exercises/tests/N-topic-name/
├── input.md          # Exercise description and requirements
├── prompts/          # Structured prompts for AI agents
├── data/            # Test constants and shared data
├── fixtures/        # Page Object Models and test utilities
└── specs/           # Actual test implementations
```

### MCP Integration (Exercise 4)
- Uses Playwright MCP for browser automation
- Tests against `https://demo.playwright.dev/todomvc` (external site)
- Focuses on DOM analysis and localStorage behavior

## Key Files to Reference
- `todomvc-react/src/todo/reducer.js`: Core state management logic
- `excercises/tests/2-planning/fixtures/todo-page.js`: Page Object Model template
- `excercises/tests/2-planning/data/test-constants.js`: Shared test data
- `todomvc-react/src/todo/app.jsx`: Main component architecture

## Testing Anti-Patterns to Avoid
- Direct `page.locator()` calls in test specs (use Page Object methods)
- Hardcoded todo text (use `TODO_ITEMS` constants)
- Missing localStorage assertions in todo creation tests
- Testing against wrong base URL (should be localhost:8080 for local tests)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AntShkliarov)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/AntShkliarov)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
