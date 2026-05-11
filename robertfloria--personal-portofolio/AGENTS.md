# Copilot Instructions - Personal Portfolio Monorepo

## Project Overview

This is a **personal portfolio monorepo** built with modern web technologies. The application showcases professional experience, projects, skills, and provides contact functionality with CV download capabilities.

## Architecture

### Monorepo Structure (npm Workspaces)

```
personal-portofolio/
├── apps/
│   ├── web/          # Next.js 16 frontend (React 19)
│   └── api/          # NestJS backend
├── packages/
│   ├── shared-types/ # Shared TypeScript interfaces/types
│   └── shared-utils/ # Shared utility functions
└── package.json      # Root workspace configuration
```

### Technology Stack

| Layer                | Technology                                       |
| -------------------- | ------------------------------------------------ |
| **Frontend**         | Next.js 16, React 19, TypeScript, Tailwind CSS 4 |
| **Backend**          | NestJS 11, TypeScript                            |
| **State Management** | React Query (TanStack Query), React Context      |
| **Forms**            | React Hook Form + Zod validation                 |
| **Styling**          | Tailwind CSS, class-variance-authority (CVA)     |
| **Animations**       | Framer Motion, Lottie                            |
| **HTTP Client**      | Axios (API), Fetch (Services)                    |
| **Testing**          | Jest, React Testing Library                      |
| **Icons**            | Lucide React                                     |

---

## Frontend Architecture (`apps/web`)

### Directory Structure

```
apps/web/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout with providers
│   ├── page.tsx            # Home page entry
│   ├── globals.css         # Global styles & CSS variables
│   └── api/                # Next.js API routes (proxy to NestJS)
│       ├── cv/route.ts     # CV download proxy
│       └── email/route.ts  # Email sending proxy
├── components/
│   ├── common/             # Reusable UI components
│   └── layout/             # Layout components (Navbar, Footer, Providers)
├── features/
│   └── home/               # Feature-based organization
│       ├── index.tsx       # Feature entry point
│       └── sections/       # Page sections
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities, constants, HTTP client
├── services/               # API service layer
├── store/
│   └── contexts/           # React Context providers
├── types/                  # TypeScript type definitions
└── public/                 # Static assets (images, lottie)
```

### Component Patterns

#### 1. Compound Component Pattern

Used for complex UI components like `Card` and `Section`:

```tsx
// Usage example
<Card.Root variant="glass" hover="glow" animated>
  <Card.Header>
    <Card.Title>Title</Card.Title>
    <Card.Description>Description</Card.Description>
  </Card.Header>
  <Card.Content>Content</Card.Content>
  <Card.Footer>Footer</Card.Footer>
</Card.Root>

<Section.Root id="about">
  <Section.Header title="About" subtitle="Subtitle" highlightText="Me" />
  <Section.Content>Content</Section.Content>
</Section.Root>
```

#### 2. CVA (Class Variance Authority) for Variants

All UI components use CVA for consistent variant styling:

```tsx
const buttonVariants = cva('base-classes', {
  variants: {
    variant: { primary: '...', secondary: '...', outline: '...', ghost: '...', link: '...' },
    size: { sm: '...', md: '...', lg: '...', icon: '...' },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});
```

#### 3. Feature-Based Organization

Each section follows this structure:

```
sections/hero-section/
├── index.tsx           # Main section component
├── index.test.tsx      # Tests
├── components/         # Section-specific components
│   ├── index.ts        # Re-exports
│   ├── hero-text.tsx
│   └── profile-image.tsx
└── lib/
    └── data.ts         # Section-specific data
```

### State Management

#### React Query for Server State

```tsx
// Mutations with automatic notifications
const mutation = useMutationWithNotification({
  mutationFn: async (data) => await service.action(data),
});
```

#### React Context for UI State

- **UIContext**: Theme, sidebar, mobile menu state
- **NotificationContext**: Global toast notifications

### Custom Hooks

| Hook                          | Purpose                                      |
| ----------------------------- | -------------------------------------------- |
| `useMutationWithNotification` | React Query mutation with auto notifications |
| `useQueryWithNotification`    | React Query query with auto notifications    |
| `useSendEmail`                | Email sending mutation                       |
| `useCvDownload`               | CV download functionality                    |
| `useIntersectionObserver`     | Viewport visibility detection                |
| `useLockBodyScroll`           | Scroll locking for modals                    |
| `useGroupBy`                  | Array grouping utility                       |

### Service Layer Pattern

Services abstract API calls:

```tsx
// services/email.service.ts
export const emailService = {
  sendEmail: async (data: SendEmailDto): Promise<EmailResponse> => {
    const response = await fetch('/api/email', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  },
};
```

### API Routes (Proxy Pattern)

Next.js API routes proxy requests to the NestJS backend:

```tsx
// app/api/email/route.ts
export async function POST(req: Request) {
  const data = await req.json();
  const response = await apiClient.post('/email/send', data);
  return NextResponse.json(response);
}
```

---

## Backend Architecture (`apps/api`)

### Directory Structure

```
apps/api/src/
├── main.ts              # Application entry point
├── app.module.ts        # Root module
├── app.controller.ts    # Health check endpoint
├── app.service.ts       # App service
├── common/
│   └── guards/
│       └── api-key.guard.ts  # API key authentication
├── cv/
│   ├── cv.module.ts
│   ├── cv.controller.ts # GET /api/cv/pdf
│   └── cv.service.ts    # S3 integration
└── email/
    ├── email.module.ts
    ├── email.controller.ts  # POST /api/email/send
    ├── email.service.ts     # Brevo API integration
    └── dto/
        └── send-email.dto.ts
```

### NestJS Patterns

#### 1. Module Organization

Each feature is a self-contained module:

```typescript
@Module({
  imports: [HttpModule],
  controllers: [EmailController],
  providers: [EmailService],
})
export class EmailModule {}
```

#### 2. Guards for Security

API key guard protects all endpoints:

```typescript
@UseGuards(ApiKeyGuard)
@Controller('email')
export class EmailController {}
```

#### 3. DTOs with Validation

Class-validator decorators for request validation:

```typescript
export class SendEmailDto {
  @IsNotEmpty()
  @IsString()
  @Length(3, 50)
  name: string;

  @IsNotEmpty()
  @IsEmail()
  from: string;
  // ...
}
```

#### 4. Global Configuration

- **CORS**: Configured via `CORS_ORIGIN` env var
- **Validation Pipe**: Whitelist, transform, forbidNonWhitelisted
- **Rate Limiting**: ThrottlerGuard (5 requests/minute)
- **API Prefix**: All routes prefixed with `/api`

### External Integrations

| Service    | Purpose        | Configuration                                                                                |
| ---------- | -------------- | -------------------------------------------------------------------------------------------- |
| **AWS S3** | CV PDF storage | `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_BUCKET`, `AWS_S3_CV_KEY` |
| **Brevo**  | Email sending  | `BREVO_API_KEY`, `EMAIL_USER`, `RECIPIENT_EMAIL`                                             |

---

## Shared Packages

### `@portfolio/shared-types`

Shared TypeScript interfaces:

```typescript
export interface SendEmailDto {
  name: string;
  from: string;
  subject: string;
  message: string;
}

export interface EmailResponse {
  message: string;
}

export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  message?: string;
  error?: string;
}
```

### `@portfolio/shared-utils`

Shared utility functions:

```typescript
export function validateEmail(email: string): boolean;
export function validateLength(value: string, min: number, max: number): boolean;
export function validateContactForm(data: ContactData): ValidationResult;
export const devConsole; // Dev-only console logging
```

---

## Implementation Guidelines

### Adding a New Feature Section

1. Create section folder: `features/home/sections/[section-name]/`
2. Create structure:
   ```
   [section-name]/
   ├── index.tsx           # Main component
   ├── index.test.tsx      # Tests
   ├── components/
   │   ├── index.ts        # Re-exports
   │   └── [component].tsx
   └── lib/
       └── data.ts         # Static data
   ```
3. Export from `features/home/sections/index.tsx`
4. Add to `features/home/index.tsx`

### Adding a New Common Component

1. Create in `components/common/[component].tsx`
2. Use CVA for variants when applicable
3. Add JSDoc documentation
4. Export from `components/common/index.ts`
5. Create test file `[component].test.tsx`

### Adding a New API Endpoint

#### Backend (NestJS):

1. Create/update module in `apps/api/src/[feature]/`
2. Create DTO with validation decorators
3. Add controller with `@UseGuards(ApiKeyGuard)`
4. Implement service logic
5. Register module in `app.module.ts`

#### Frontend Proxy:

1. Create Next.js API route in `apps/web/app/api/[feature]/route.ts`
2. Use `apiClient` from `lib/http-client.ts`

#### Frontend Service:

1. Create service in `services/[feature].service.ts`
2. Export from `services/index.ts`
3. Create custom hook in `hooks/use-[feature].ts`

### Adding a New Custom Hook

1. Create in `hooks/use-[name].ts`
2. Add JSDoc documentation
3. Export from `hooks/index.ts`
4. For mutations, use `useMutationWithNotification` wrapper

---

## Styling Guidelines

### CSS Variables (Tailwind CSS 4)

Theme colors are defined as CSS variables in `globals.css`:

```css
:root {
  --primary: 217 91% 60%;
  --accent: 280 87% 65%;
  --background: 220 14% 96%;
  --foreground: 220 9% 10%;
  --card: 0 0% 100%;
  --border: 220 9% 85%;
  /* ... */
}

.dark {
  --background: 230 12% 7%;
  --foreground: 220 9% 95%;
  /* ... */
}
```

### Utility Classes

- Use Tailwind's spacing scale: `gap-component`, `p-section`, `p-card`
- Use `cn()` utility for conditional classes
- Prefer semantic variants over direct color classes

### Animation Constants

Use `ANIMATION_DURATIONS` from `lib/constants.ts`:

```typescript
const ANIMATION_DURATIONS = {
  FAST: 0.2,
  NORMAL: 0.3,
  SLOW: 0.6,
  NOTIFICATION: 5000,
  ERROR_NOTIFICATION: 7000,
};
```

---

## Testing

### Frontend Tests

- Jest + React Testing Library
- Mocks in `__mocks__/` for framer-motion, lottie-react, next-themes
- Test files co-located: `[component].test.tsx`

### Backend Tests

- Jest for unit tests
- Supertest for e2e tests
- Test files: `*.spec.ts`

---

## Environment Variables

### Backend (`apps/api/.env`)

```env
PORT=4000
CORS_ORIGIN=http://localhost:3000
API_SECRET=your-api-secret

# AWS S3
AWS_REGION=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_S3_BUCKET=
AWS_S3_CV_KEY=cv.pdf

# Brevo Email
BREVO_API_KEY=
EMAIL_USER=
RECIPIENT_EMAIL=
```

### Frontend (`apps/web/.env.local`)

```env
NEST_API_URL=http://localhost:4000/api
API_SECRET=your-api-secret
```

---

## Scripts Reference

```bash
# Development
npm run dev           # Run both web and api
npm run dev:web       # Run web only
npm run dev:api       # Run api only

# Build
npm run build         # Build all workspaces
npm run build:web     # Build web only
npm run build:api     # Build api only

# Quality
npm run lint          # Lint all workspaces
npm run lint:fix      # Lint and fix
npm run format        # Format with Prettier
npm run type-check    # TypeScript checks

# Testing
npm run test          # Run all tests
```

---

## Key Conventions

1. **TypeScript**: Strict mode, explicit return types for public APIs
2. **Imports**: Use `@/` alias for web app imports
3. **Components**: Prefer functional components with `forwardRef` when needed
4. **Documentation**: JSDoc comments for all exported functions/components
5. **Error Handling**: Use notification context for user feedback
6. **Validation**: Zod on frontend, class-validator on backend
7. **State**: React Query for server state, Context for UI state
8. **Styling**: Tailwind CSS with CVA for component variants

---
> Source: [robertfloria/personal-portofolio](https://github.com/robertfloria/personal-portofolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-10 -->
