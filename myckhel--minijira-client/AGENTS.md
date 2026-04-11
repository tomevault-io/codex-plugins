# Senior React Developer Copilot Instructions - Mini Jira Frontend

You are an expert in TypeScript, React, React Router, Zustand, Ant Design, and Tailwind CSS.

## Project Overview

Building a Mini Jira Clone frontend with:

- **Language**: TypeScript (strict mode)
- **Framework**: React 19+ (functional components)
- **Routing**: React Router v6
- **State Management**: Zustand
- **UI Library**: Ant Design (Antd)
- **Styling**: Tailwind CSS (mobile-first)
- **Authentication**: JWT-based

## React Best Practices

### Component Architecture

- Use functional components with TypeScript interfaces for props
- Use the `function` keyword for component definitions
- Implement hooks correctly (`useState`, `useEffect`, `useCallback`, `useMemo`)
- Follow the Rules of Hooks (only call hooks at the top level)
- Create custom hooks to extract reusable component logic
- Use `React.memo()` for component memoization when appropriate
- Implement `useCallback` for memoizing functions passed as props
- Use `useMemo` for expensive computations
- Avoid inline function definitions in render to prevent unnecessary re-renders
- Prefer composition over inheritance
- Use children prop and render props pattern for flexible components
- Implement `React.lazy()` and `Suspense` for code splitting
- Use refs sparingly and mainly for DOM access
- Prefer controlled components over uncontrolled components
- Implement error boundaries to catch and handle errors gracefully
- Use cleanup functions in `useEffect` to prevent memory leaks
- Use short-circuit evaluation and ternary operators for conditional rendering

### TypeScript Integration

- Define explicit interfaces for all props and state
- Use proper typing for event handlers
- Implement generic types for reusable components
- Use `as const` for immutable data structures
- Leverage TypeScript's strict mode features

## Routing

- Use React Router for navigation.
- Pages: `/login`, `/register`, `/dashboard`
- Protect routes that require authentication.
- Redirect unauthenticated users to `/login`.

## State Management with Zustand

### Store Organization

- Create separate stores for different domains:
  - `authStore`: User authentication, login state, user profile
  - `taskStore`: Task CRUD operations, filtering, sorting
  - `uiStore`: Modal states, loading states, notifications
- Use shallow comparison for store selectors to optimize re-renders
- Implement actions as methods within stores
- Use immer for complex state updates when needed
- Use devtools plugin for store

## UI and Styling Guidelines

### Ant Design Usage

- Use Antd components for complex UI elements:
  - `Form` with validation for login/register/task creation
  - `Table` or `List` for task display with sorting and filtering
  - `Modal` for task creation and editing
  - `Button`, `Input`, `Select`, `DatePicker` for form controls
  - `Drawer` or `Sider` for navigation
  - `Card` for task items in list view
  - `Tag` for task status and priority
  - `Notification` for user feedback

### Tailwind CSS Integration

- Use Tailwind for layout, spacing, and responsive design
- Implement mobile-first responsive design
- Use Tailwind utilities for rapid prototyping
- Combine with Antd's design tokens when possible
- Use Tailwind for custom styling that extends Antd components

### Responsive Design

- Mobile-first approach with `sm:`, `md:`, `lg:`, `xl:` breakpoints
- Ensure touch-friendly interfaces on mobile
- Implement responsive navigation (drawer on mobile, sidebar on desktop)
- Use Antd's responsive props where available

## Performance Optimization

### Code Splitting

- Use `React.lazy()` for route-based code splitting
- Wrap components in `Suspense` with appropriate fallbacks
- Implement dynamic imports for heavy components

### Memoization

- Use `React.memo()` for components that receive stable props
- Implement `useCallback` for event handlers passed to child components
- Use `useMemo` for expensive calculations
- Optimize Zustand selectors to prevent unnecessary re-renders

### Bundle Optimization

- Use tree shaking for Antd components (import specific components)
- Implement proper lazy loading for images and non-critical content
- Use Vite or similar for fast development and optimized builds

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── common/          # Generic components (Layout, Header, etc.)
|   ├── features/        # Feature-based structure
│       ├── auth/
│       ├── tasks/
│       └── projects
│   └── ui/              # Custom UI components
├── pages/               # Route components
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   └── DashboardPage.tsx
├── stores/              # Zustand stores
│   ├── authStore.ts
│   ├── taskStore.ts
│   └── uiStore.ts
├── assets/              # Assets for images
├── routes/              # App routing
├── hooks/               # Custom hooks
├── types/               # TypeScript type definitions
├── services/            # API services
├── utils/               # Utility functions
├── apis/                # Abstracted Apis
├── constants/           # App constants
└── styles/              # Global styles (if needed)
```

## API Integration

### HTTP Client

- Use modular/abstracted axios for API calls
- Implement request/response interceptors for JWT handling
- Create a service layer for API endpoints
- Handle error responses consistently
- Implement proper loading states

### Authentication Flow

- Store JWT tokens securely (httpOnly cookies preferred)
- Implement automatic token refresh (low priority)
- Handle 401/403 responses appropriately
- Clear auth state on logout

## Component Patterns

### Form Components

```typescript
interface TaskFormProps {
  task?: Task;
  onSubmit: (data: TaskFormData) => Promise<void>;
  onCancel: () => void;
}

function TaskForm({ task, onSubmit, onCancel }: TaskFormProps) {
  const [form] = Form.useForm();
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = useCallback(
    async (values: TaskFormData) => {
      setIsLoading(true);
      try {
        await onSubmit(values);
      } finally {
        setIsLoading(false);
      }
    },
    [onSubmit]
  );

  return (
    <Form form={form} onFinish={handleSubmit} layout="vertical">
      {/* Form fields */}
    </Form>
  );
}
```

### List Components

- Implement virtualization for large lists
- Use Antd's Table or List components with proper pagination
- Implement filtering and sorting capabilities
- Handle empty states gracefully

## Error Handling

### Error Boundaries

- Implement error boundaries for route-level error catching
- Create fallback UI components for error states
- Log errors appropriately for debugging

### Form Validation

- Use Antd's form validation with custom rules
- Implement client-side validation for better UX
- Display validation errors clearly
- Handle server-side validation errors

<!-- ## Testing Considerations -->

<!-- ### Component Testing
- Write unit tests using React Testing Library
- Test user interactions and component behavior
- Mock Zustand stores in tests
- Test accessibility features -->

### Integration Testing

- Test complete user flows (login, task creation, etc.)
- Mock API calls for consistent testing

### Visual Accessibility

- Maintain proper color contrast ratios
- Use focus indicators for keyboard navigation
- Implement proper text sizing and spacing
- Support reduced motion preferences

## Security Considerations

### Input Validation

- Validate all user inputs on the client side
- Sanitize data before displaying
- Prevent XSS attacks through proper escaping
- Implement CSRF protection

### Authentication Security

- Never store sensitive data in localStorage
- Implement proper session management
- Handle authentication state securely
- Validate tokens on critical operations

## Development Workflow

### Code Quality

- Use ESLint and Prettier for code formatting
- Use TypeScript strict mode
- Follow consistent naming conventions

### Environment Management

- Use environment variables for API endpoints
- Implement different configurations for dev/staging/prod
- Use proper build optimization for production

**Generate all code following these principles, ensuring production-ready quality with proper TypeScript typing, error handling, and accessibility features.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/myckhel)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/myckhel)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
