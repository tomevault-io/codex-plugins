<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Workflow Designer Project

This is a React TypeScript project that implements a visual workflow designer using Material-UI and React Flow.

## Project Structure

- **Components**: All React components are in `src/components/`
  - `WorkflowDesigner/`: Main workflow designer components
  - `Nodes/`: Custom node components for the workflow canvas
- **Types**: TypeScript type definitions in `src/types/`
- **Config**: Configuration files including node templates in `src/config/`

## Key Libraries

- **React Flow**: For the drag-and-drop workflow canvas
- **Material-UI (MUI)**: For consistent UI components and theming
- **TypeScript**: For type safety and better development experience

## Code Conventions

1. Use TypeScript for all new files
2. Follow Material-UI design patterns and components
3. Use functional components with React hooks
4. Implement proper error handling and loading states
5. Keep components modular and reusable
6. Use meaningful variable and function names
7. Add proper TypeScript types for all props and state

## Workflow Designer Features

- Drag and drop nodes from palette to canvas
- Connect nodes to create workflow chains
- Different node types: triggers, actions, logic, data, AI, integrations
- Visual node representation with icons and categories
- Save/load workflow functionality
- Export/import workflows as JSON

## Development Guidelines

- Always use Material-UI components instead of HTML elements
- Maintain consistent theming using the theme.ts file
- Keep the workflow state management clean and predictable
- Add proper accessibility attributes where needed
- Ensure responsive design for different screen sizes

---
> Source: [gampalasrinivas/IQ.Hub.Workflows](https://github.com/gampalasrinivas/IQ.Hub.Workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-17 -->
