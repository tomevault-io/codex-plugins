# The Dojo - Copilot Instructions

## Project Overview: Training Management Platform

The system follows a hierarchical structure with role-based access control, focusing on managing trainees, training subjects (courses), and assigned tasks (assignments/projects).

## Technology Stack (STRICT REQUIREMENTS)

### Core Technologies ONLY

- **React 19+** with **TypeScript** (strict mode)
- **TanStack Router** - File-based routing only
- **Native Fetch API** - HTTP requests
- **React Context API** - State management via Provider Pattern
- **CSS Modules or inline styles** - No external CSS-in-JS libraries

### Prohibited External Libraries

- ❌ No state management libraries (Redux, Zustand, MobX, etc.)
- ❌ No UI component libraries (Material-UI, Chakra, Ant Design, etc.)
- ❌ No toast/notification libraries
- ❌ No CSS-in-JS libraries (styled-components, emotion, etc.)
- ❌ No utility libraries (lodash, moment, date-fns, etc.)
- ❌ No form libraries (Formik, React Hook Form, etc.)

### TypeScript Requirements

- Strict mode enabled in `tsconfig.json`
- All components must have proper TypeScript interfaces
- API response types must be defined
- No `any` types allowed - use `unknown` if necessary
- All props interfaces must be explicitly defined

## Core System Architecture

### Backend Integration

- **RESTful API** with JSON responses (see Postman collection for endpoints `Resources/postman/`)
- **HTTP Methods**: GET, POST, PUT, DELETE
- **Error Handling**: Standardized error responses with status codes
- **Authentication**: Bearer token-based (JWT)
- **Data Storage**: Relational database
- **Security**: CORS-enabled with encrypted authentication

### File Structure

- create a clear, feature-based file structure below: is a guide but only create folders that are necessary for the project

```
src/
├── routes/           # TanStack Router file-based routes
├── components/       # Reusable UI components
├── providers/        # Context providers
├── hooks/           # Custom hooks
├── api/             # API service layer
├── types/           # TypeScript type definitions
├── utils/           # Utility functions
└── styles/          # CSS modules
```

## Provider Pattern Implementation

### Core Provider Structure

```typescript
// providers/AppProvider.tsx
interface AppContextType {
  state: AppState;
  actions: AppActions;
}

interface AppState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

interface AppActions {
  setUser: (user: User) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
}

const AppContext = createContext<AppContextType | undefined>(undefined);

export const AppProvider: React.FC<{children: React.ReactNode}> = ({children}) => {
  const [state, setState] = useState<AppState>(initialState);

  const actions = useMemo<AppActions>(() => ({
    setUser: (user) => setState(prev => ({...prev, user})),
    setLoading: (loading) => setState(prev => ({...prev, loading})),
    setError: (error) => setState(prev => ({...prev, error}))
  }), []);

  return (
    <AppContext.Provider value={{state, actions}}>
      {children}
    </AppContext.Provider>
  );
};

export const useApp = (): AppContextType => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
};
```

### Custom Toast Provider

```typescript
// providers/ToastProvider.tsx
interface Toast {
  id: string
  message: string
  type: 'success' | 'error' | 'warning' | 'info'
  duration?: number
}

interface ToastContextType {
  toasts: Toast[]
  addToast: (toast: Omit<Toast, 'id'>) => void
  removeToast: (id: string) => void
}

export const ToastProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  // Implementation here
}

export const useToast = (): ToastContextType => {
  // Hook implementation
}
```

## Key System Components

### 1. User Management System

- **Multi-role architecture**: Admin and Trainee roles
- **Approval workflow**: Trainees can be approved, rejected, or kept pending
- **Profile management**: Basic user profiles (reference Postman collection for fields)
- **Account lifecycle**: Full CRUD operations for user accounts

### 2. Subject/Course Management

- **Curriculum structure**: Subjects represent training modules/courses
- **Content organization**: Each subject has detailed descriptions and metadata
- **Active/inactive status**: Subjects can be enabled/disabled
- **Audit trail**: Tracks who created/modified subjects

### 3. Task/Assignment Management

- **Structured assignments**: Tasks are linked to specific subjects
- **Detailed requirements**: Each task includes description, requirements, and scoring
- **Time management**: Due dates and deadlines
- **Scoring system**: Maximum scores for evaluation
- **Status tracking**: Tasks can be active/inactive

## Data Relationships & Types

```typescript
// types/index.ts
interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'trainee'
  status: 'approved' | 'pending' | 'rejected'
  avatar?: string
  createdAt: string
  updatedAt: string
}

interface Subject {
  id: string
  name: string
  description: string
  isActive: boolean
  createdBy: string
  createdAt: string
  updatedAt: string
}

interface Task {
  id: string
  subjectId: string
  title: string
  description: string
  requirements: string
  dueDate: string
  maxScore: number
  isActive: boolean
  createdBy: string
  createdAt: string
  updatedAt: string
}
```

## API Interaction Patterns

### Service Layer Structure

```typescript
// api/base.ts
const API_BASE_URL = process.env.REACT_APP_API_BASE_URL

interface ApiResponse<T> {
  data: T
  message?: string
  success: boolean
}

class ApiService {
  private async request<T>(
    endpoint: string,
    options: RequestInit = {},
  ): Promise<ApiResponse<T>> {
    // Implementation with error handling, auth headers, etc.
  }

  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: 'GET' })
  }

  async post<T>(endpoint: string, data: unknown): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    })
  }
}

export const apiService = new ApiService()
```

### Error Handling Pattern

```typescript
// hooks/useApiCall.ts
interface UseApiCallState<T> {
  data: T | null
  loading: boolean
  error: string | null
}

export const useApiCall = <T>() => {
  const [state, setState] = useState<UseApiCallState<T>>({
    data: null,
    loading: false,
    error: null,
  })

  const { addToast } = useToast()

  const execute = useCallback(
    async (apiCall: () => Promise<ApiResponse<T>>) => {
      setState((prev) => ({ ...prev, loading: true, error: null }))

      try {
        const response = await apiCall()
        if (response.success) {
          setState({ data: response.data, loading: false, error: null })
        } else {
          throw new Error(response.message || 'API call failed')
        }
      } catch (error) {
        const errorMessage =
          error instanceof Error ? error.message : 'Unknown error'
        setState({ data: null, loading: false, error: errorMessage })
        addToast({ message: errorMessage, type: 'error' })
      }
    },
    [addToast],
  )

  return { ...state, execute }
}
```

## Business Logic & Workflows

### Admin Workflow:

1. **Onboard trainees**: Register users and set roles
2. **Review applications**: Approve/reject pending trainees
3. **Manage curriculum**: Create subjects for different training modules
4. **Assign work**: Create tasks under subjects with deadlines
5. **Monitor progress**: View trainee lists and task assignments

### Trainee Workflow:

1. **Access assigned tasks**
2. **Submit completed work**
3. **Track progress and scores**
4. **View training curriculum**

## Routing & Navigation

### TanStack Router Structure

- Use file-based routing in `/src/routes/`
- Feature-based route organization
- Protected routes for admin-only access
- Type-safe route definitions

```typescript
// routes/admin/subjects.tsx
export const Route = createFileRoute('/admin/subjects')({
  component: SubjectsPage,
  beforeLoad: ({ context }) => {
    if (context.user?.role !== 'admin') {
      throw redirect({ to: '/unauthorized' })
    }
  },
})
```

## State Management Strategy

### Provider Hierarchy

```typescript
// App.tsx
function App() {
  return (
    <AppProvider>
      <ToastProvider>
        <AuthProvider>
          <Router />
        </AuthProvider>
      </ToastProvider>
    </AppProvider>
  );
}
```

### State Management Rules

- **Local state**: Use `useState` for ephemeral UI state (modals, form inputs, temporary data)
- **Lifted state**: When multiple sibling components share data, lift to nearest common ancestor
- **Context providers**: Use Provider Pattern for cross-component state (user auth, theme, toast notifications)
- **Avoid prop drilling**: Use context when passing props through more than 2 levels

## Design System Implementation

### Core Design Approach

- Treat visual design as objective discipline, not subjective preference
- Use specific vocabulary (large/small, primary/secondary, heavy/light)
- Apply fundamental principles: scale, hierarchy, balance, contrast, Gestalt principles
- Design to follow mobile-first approach (responsive design, fluid layouts) this is suggested but not mandatory
- Use pixels for design precision,
- rem for scalable layouts, consistent scaling, typography system
- em for component based scalability, buttons, spacing
- % or Max/Min with PX for fluid layouts, containers, grids
- vh/vw for full screen sections, hero areas basically for responsive headlines, hero text such.
- Line height: 1.4-1.6 for readability
- Font sizes: 14px base, 16px for body text, 20px+ for headings
- Spacing: 8px base unit, multiples for margins/padding

### Visual Hierarchy & Scale

- **Create clear information flow**: Arrange elements from most to least important
- **Use size strategically**: Make primary elements larger for focus
- **Label importance levels**: Classify as primary, secondary, tertiary

### Layout & Spacing

- **Align all elements**: Position along consistent vertical/horizontal lines
- **Group related content**: Place associated elements closer together
- **Utilize white space**: Allow content to breathe, avoid cramped layouts

### Styling Implementation

```typescript
// components/Button/Button.module.css
.button {
  /* Base styles */
}

.primary {
  /* Primary button styles */
}

.secondary {
  /* Secondary button styles */
}

// components/Button/Button.tsx
import styles from './Button.module.css';

interface ButtonProps {
  variant: 'primary' | 'secondary';
  children: React.ReactNode;
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({ variant, children, onClick }) => {
  return (
    <button
      className={`${styles.button} ${styles[variant]}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

## Development Guidelines

### Component Conventions

- **Composition over booleans**: Prefer composite components over boolean props
- **Provider Pattern**: Use for sharing state and actions across component tree
- **Custom hooks**: Extract reusable logic into custom hooks
- **Type safety**: Every component must have TypeScript interfaces

### Code Quality Standards

- **No `any` types**: Use proper TypeScript typing
- **Error boundaries**: Implement for graceful error handling
- **Loading states**: Show appropriate loading UI during async operations
- **Validation**: Client-side validation before API calls
- **Accessibility**: Proper ARIA labels and keyboard navigation

### Performance Considerations

- **Memoization**: Use `useMemo` and `useCallback` for expensive operations
- **Provider optimization**: Memoize context values to prevent unnecessary re-renders
- **Component splitting**: Lazy load routes and heavy components
- **State updates**: Batch related state updates when possible

## Authentication & Security

### JWT Implementation

```typescript
// providers/AuthProvider.tsx
interface AuthContextType {
  user: User | null
  login: (credentials: LoginCredentials) => Promise<void>
  logout: () => void
  isAuthenticated: boolean
}

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  // JWT token management
  // User authentication state
  // Automatic token refresh
}
```

### Protected Routes

- Implement route guards based on user roles
- Redirect unauthorized users appropriately
- Handle token expiration gracefully

## Error Handling & User Feedback

### Custom Error Boundary

```typescript
// components/ErrorBoundary.tsx
interface ErrorBoundaryProps {
  children: React.ReactNode
  fallback?: React.ComponentType<{ error: Error }>
}

export class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  // Error boundary implementation
}
```

### Toast Notification System

- Custom implementation using Provider Pattern
- Multiple toast types (success, error, warning, info)
- Auto-dismiss functionality
- Queue management for multiple toasts

## Key Features Implementation

### Scalable Pagination

```typescript
// hooks/usePagination.ts
interface UsePaginationParams {
  totalItems: number
  itemsPerPage: number
  currentPage: number
}

export const usePagination = (params: UsePaginationParams) => {
  // Pagination logic
}
```

### Filtering & Search

- When filters/search/sort are applied encode the state in the URL query params and persist across navigations
- Debounce search input to minimize API calls

```typescript
// hooks/useFilter.ts
export const useFilter = <T>(data: T[], filterFn: (item: T) => boolean) => {
  // Filter logic
}
```

### Audit Logging

- Track all CRUD operations with timestamps
- User attribution for all changes
- Soft delete implementation where appropriate

## Testing Strategy

### Component Testing

- Test component rendering with different props
- Test user interactions and state changes
- Test error states and edge cases

### Provider Testing

- Test context providers in isolation
- Test hook behavior with mocked providers
- Test provider composition

### API Testing

- Mock API responses for testing
- Test error handling scenarios
- Test loading states

## Deployment Considerations

### Environment Variables

```typescript
// types/env.d.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      REACT_APP_API_BASE_URL: string
      REACT_APP_JWT_SECRET: string
    }
  }
}
```

### Build Optimization

- Tree shaking for unused code
- Code splitting at route level
- Asset optimization and caching

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OscarNgombo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
