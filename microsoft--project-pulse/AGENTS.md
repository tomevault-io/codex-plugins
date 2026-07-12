# Project Overview

# Key terminology

- **Workspaces**: A workspace is a logical grouping of related agents and tasks. It serves as a container for organizing and managing your development environments.
- **Agents**: Agents are individual development environments or containers that run specific tasks. Each agent can be configured with different settings and resources.
- **Tasks**: Tasks are specific actions or jobs that agents perform. They can include running scripts, building applications, or any other automated processes.

# Features

- Create, manage, and delete workspaces.
- Create, manage, and delete agents within workspaces.
- Create, manage, and delete tasks associated with agents.
- Start, stop, and monitor the status of agents.
- View logs and outputs from tasks executed by agents.

# Instructions
## Build
1. Ensure you have Node.js 22 and npm installed on your machine.
2. Run `npm install` to install all dependencies.
3. Run `npm run build` to compile the TypeScript code.

## Testing
1. Run `npm test` to execute the test suite.

# Notes
- Place notes as markdown files in the `docs` folder.

# React UI/UX Code Guidelines

## Component Structure & Organization
- **Single Responsibility**: Each component should have one clear purpose. Break down complex components into smaller, focused ones.
- **Consistent Naming**: Use PascalCase for components (`UserProfile`) and camelCase for props/variables (`isLoading`, `userData`).
- **File Organization**: Place components in dedicated folders with descriptive filenames (`components/UserProfile/UserProfile.tsx`).

## Code Readability & Maintainability
- **TypeScript First**: Always use TypeScript with proper interface definitions for props and state.
- **Descriptive Prop Names**: Use clear, self-documenting prop names (`onUserSelect` instead of `onClick`, `isSubmitting` instead of `loading`).
- **Extract Custom Hooks**: Move complex logic to custom hooks for reusability and testability.
- **Avoid Inline Styles**: Use CSS modules, styled-components, or Tailwind classes instead of inline styles.

## UI/UX Best Practices
- **Accessibility**: Include proper ARIA labels, semantic HTML, keyboard navigation, and screen reader support.
- **Responsive Design**: Use mobile-first approach with proper breakpoints and flexible layouts.
- **Loading States**: Always provide feedback for async operations with loading spinners, skeletons, or progress indicators.
- **Error Handling**: Implement user-friendly error messages and fallback UI states.
- **Consistent Spacing**: Use design system tokens or Tailwind's spacing scale for consistent margins and padding.

## Performance Considerations
- **Memoization**: Use `React.memo`, `useMemo`, and `useCallback` appropriately to prevent unnecessary re-renders.
- **Lazy Loading**: Implement code splitting for large components using `React.lazy` and `Suspense`.
- **Optimize Bundle Size**: Avoid importing entire libraries when only specific functions are needed.

## State Management
- **Local vs Global State**: Keep state as local as possible, only lift up when necessary.
- **Immutable Updates**: Always update state immutably, especially with arrays and objects.
- **Form Handling**: Use controlled components with proper validation and error states.

## Code Examples & Patterns
When generating React code, follow these patterns:

```typescript
// Good component structure
interface UserCardProps {
  user: User;
  onSelect: (userId: string) => void;
  isSelected?: boolean;
}

export const UserCard: React.FC<UserCardProps> = (props) => {
  const { user, onSelect, isSelected = false } = props;

  // Implementation
};

// Good hook usage
const useUserData = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  // Implementation
  
  return { user, isLoading, error };
};
```

## Testing Considerations
- **Testable Components**: Write components that are easy to test with clear inputs and outputs.
- **Mock External Dependencies**: Ensure components can be tested in isolation.
- **Accessibility Testing**: Include tests for keyboard navigation and screen reader compatibility.

# Patterns

## Retrieving access token

- Use `window.electronAPI.getAzureToken(resource)` to retrieve an access token for a specified resource. This function calls into the main process to get the token using Azure CLI, so it assumes the user is logged in.

---
> Source: [microsoft/project-pulse](https://github.com/microsoft/project-pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-12 -->
