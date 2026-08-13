# Foxglove Retail Extension - AI Coding Guidelines

## Architecture Overview
This is a Foxglove extension providing a retail management interface with multiple specialized views. The extension uses a single-panel architecture where `ExamplePanel.tsx` serves as the main container with a sidebar navigation switching between different page components in `src/views/`.

## Key Components & Patterns

### Panel Structure
- **Main Panel**: `src/ExamplePanel.tsx` - Single panel with sidebar navigation
- **Views**: `src/views/` - Individual page components (ModelPage, ConfigPage, etc.)
- **API Layer**: `src/api/driveApi.ts` - File operations via REST API to cloud storage

### UI Patterns
- **Material-UI**: Use `@mui/material` components with `styled()` for custom styling
- **Navigation**: Sidebar with `ListItemButton` components for view switching
- **Layout**: `Box` components with flexbox layout (`display: 'flex'`)
- **Feedback**: `Snackbar` with `Alert` components for user notifications
- **Loading States**: Boolean state variables with disabled buttons during async operations

### File Operations
- **Base URL**: `http://172.16.10.50:30001/hos-service/api/v1/taskmanager/drive/storage`
- **Drive ID**: `hos-24:41:8c:18:6c:99` (constant in driveApi.ts)
- **Authentication**: Bearer tokens from `localStorage.getItem('userToken')`
- **Paths**: Use `/retail/` prefix (e.g., `/retail/goods-models`, `/retail/grasp-config`)

### Code Conventions
- **Language**: Chinese comments and UI text throughout
- **Error Handling**: Try/catch blocks with snackbar notifications
- **State Management**: React hooks (`useState`, `useEffect`, `useRef`)
- **File Inputs**: Hidden inputs with `useRef` for programmatic triggering
- **Async Operations**: Loading states with `setLoading(true/false)`

## Development Workflow
- **Local Development**: `npm run local-install` installs to Foxglove desktop
- **Building**: `npm run build` compiles TypeScript to `dist/`
- **Packaging**: `npm run package` creates `.foxe` extension file
- **Linting**: `npm run lint` uses Foxglove's ESLint config with React/TypeScript rules

## Adding New Views
1. Create component in `src/views/` following existing patterns
2. Add navigation item to `navItems` array in `ExamplePanel.tsx`
3. Add case to `renderContent()` switch statement
4. Import component at top of `ExamplePanel.tsx`

## API Integration
- Extend `driveApi.ts` for new file operations
- Follow existing pattern: async functions with error handling
- Use `getAuthHeaders()` for authentication
- Return structured data or throw errors

## Common Patterns
- **File Upload**: `uploadFile(DRIVE_ID, file, path)` with FormData
- **File Download**: `downloadFile(DRIVE_ID, path)` returns Blob
- **File Listing**: `listFiles(DRIVE_ID, path)` returns filtered arrays
- **Notifications**: `setSnackbarMessage()`, `setSnackbarSeverity()`, `setSnackbarOpen(true)`

## Dependencies
- **Foxglove SDK**: `@foxglove/extension` for panel context and lifecycle
- **Monaco Editor**: For YAML configuration editing in ConfigPage
- **Material-UI**: Complete component library with theming
- **React**: Functional components with hooks (no class components)</content>
<parameter name="filePath">/home/nav/Downloads/1-HOS/foxglove-extensions/foxglove-retail/.github/copilot-instructions.md

---
> Source: [deng-h/foxglove-retail](https://github.com/deng-h/foxglove-retail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
