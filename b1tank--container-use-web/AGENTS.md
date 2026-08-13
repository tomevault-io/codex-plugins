# General Coding Practices

- Use clear and descriptive variable and function names.
- Write modular and reusable code.
- Include comments and documentation for complex logic.
- Follow consistent code formatting and style guidelines.

# Preferred Technologies

- Use commander for CLI development.
- Use esbuild for bundling CLI and backend, and vite for bundling frontend.
- Use node.js and hono for backend development.
- Use TypeScript and tanstack router/query for frontend development.
- Use React and popular libraries for building user interfaces.
- Use pnpm for package management for both frontend and backend.
- Use Tanstack router for routing in React applications.
- Use latest version of Shadcn UI for UI components (install cmd, e.g. `pnpm dlx shadcn@latest add accordion`).
- Use Tailwind CSS for styling.
- Use the latest version of all libraries and frameworks (use fetch tool to check for updates).

# Action Requirements

- no need to run `pnpm run dev` or similar commands, just focus on the code changes.
- wrap functions in `useCallback` when passing them as props to child components.
- wrap functions in `useMemo` when they return a value that is used in the component.
- use `useEffect` for side effects, such as data fetching or subscriptions.
- use `useState` for managing local component state.
- avoid circular dependencies in when using `useState` and `useEffect`.
- use `useRef` for mutable references that do not trigger re-renders.
- use `useContext` for accessing global state or context providers.
- For end-to-end features, always first implement the backend API, then generate the client sdk using `./scripts/generate-client.sh`, and finally implement the frontend UI.
- Before implementing any UI feature, always check if there is shadcn UI component available that fits the requirement directly or can be easily adapted.
- Always summarize your implementation plan before starting to write code.
- You summarization should be presented in a way that is easy for me to check or dispute item by item.
- You summary after implementation should include a lean statement summarizing TL;DR of what you did, a list of all dependencies, folders, files that were created, modified or deleted, with a brief description of each, and next steps if applicable.

---
> Source: [b1tank/container-use-web](https://github.com/b1tank/container-use-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
