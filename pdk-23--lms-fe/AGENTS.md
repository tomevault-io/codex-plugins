# Development Guide for a Coursera-like Learning Platform

## Project Overview

Build a platform for delivering online courses that includes:

- A catalog of courses organized by categories
- Advanced search and filtering
- Instructor profiles and reviews
- Course content (video, documents, quizzes)
- Knowledge assessment system (quizzes and coding-style challenges)
- Certificate issuance for course completion
- Shopping cart and payment

## Technology Stack

- Frontend Framework: React 19.2.0 + TypeScript
- Build Tool: Vite
- Styling: TailwindCSS 3.3.6
- UI Components: shadcn/ui + Lucide React Icons
- Form Management: React Hook Form + Zod Validation
- Routing: React Router DOM 7.0.0
- Utilities: clsx, tailwind-merge, class-variance-authority

## Project Structure

```
src/
├── components/
│   ├── ui/                 # shadcn/ui components (Button, Card, Dialog, etc.)
│   ├── common/             # Layout components (Header, Footer, Navbar)
│   └── course/             # Course-specific components (CourseCard, Filters, etc.)
├── pages/                  # Page components (Home, CourseDetail, etc.)
├── types/                  # TypeScript type definitions
├── lib/                    # Utility functions, constants
├── hooks/                  # Custom React hooks
├── App.tsx                 # Main app component with routing
└── main.tsx                # App entry point
```

## Data Management & Mock Data Strategy

### Current Architecture

Currently, all frontend data is provided by mock data stored in the `src/mocks/` folder:

```
src/
├── mocks/
│   ├── courses.ts        # Mock course data
│   ├── instructors.ts    # Mock instructor data
│   ├── categories.ts     # Mock category data
│   └── index.ts          # Export all mock data
```

### Design Pattern for Easy Migration to a Real API

To make it straightforward to switch from mock data to server API calls in the future, follow this pattern:

#### 1. Data Service Layer (`src/services/`)

Create a service layer that centralizes data fetching logic and isolates components from where data comes from:

```typescript
// src/services/courseService.ts
export async function getCourses(): Promise<Course[]> {
  // Current: import from mock
  // const { courses } = await import('../mocks/courses');
  // return courses;
  // Future: API call
  // const response = await fetch('/api/courses');
  // return response.json();
}

export async function getCourseById(id: string): Promise<Course | undefined> {
  // Current: return courses.find(c => c.id === id);
  // Future: return fetch(`/api/courses/${id}`).then(r => r.json());
}

export async function getInstructors(): Promise<Instructor[]> {
  // ...
}
```

#### 2. Custom Hooks (`src/hooks/`)

Wrap service calls in custom hooks that provide loading and error states to components:

```typescript
// src/hooks/useCourses.ts
export function useCourses() {
  const [courses, setCourses] = useState<Course[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    getCourses()
      .then(setCourses)
      .catch((err) => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  return { courses, loading, error };
}
```

#### 3. Component Usage

Components should call hooks and not import mock data directly:

```tsx
// Good: ✓
function CourseList() {
  const { courses, loading } = useCourses();
  return (
    <div>
      {courses.map((c) => (
        <CourseCard key={c.id} course={c} />
      ))}
    </div>
  );
}

// Avoid: ✗
// import { mockCourses } from '../mocks/courses';
// function CourseList() {
//   return <div>{mockCourses.map(...)}</div>;
// }
```

### Migration Checklist

When you're ready to move to a real API:

1. Create API endpoints that match the current data structure
2. Update the service layer (`src/services/*.ts`) to call the API instead of mocks
3. Add error handling and optional retry logic
4. Ensure hooks provide loading and empty states
5. Test the full data flow end-to-end
6. Keep or remove `src/mocks/` for development and testing as needed

### Best Practices

- Never import mock data directly in components — always go through the service layer
- Keep TypeScript types in `src/types/` aligned with API responses
- Centralize API configuration (create `src/config/api.ts` for base URL, headers, etc.)
- Surface errors from the service layer to components via hooks and use error boundaries where appropriate

## Design System & Colors (Inspired by Coursera)

- **Primary Blue**: #4f46e5 (Main CTA, navigation)
- **Dark Blue**: #1e1b4b (Text headings, dark mode)
- **Light Blue**: #f0f4ff (Backgrounds, hover states)

### Accent Colors

- **Secondary Purple**: #ec4899 (Highlights, featured courses)
- **Success Green**: #10b981 (Completion status, ratings)
- **Warning Orange**: #f59e0b (Limited slots, urgent)
- **Danger Red**: #ef4444 (Error states)

### Neutral Palette

- **White/Very Light**: #f9fafb (Primary backgrounds)
- **Light Gray**: #e5e7eb (Borders, subtle dividers)
- **Medium Gray**: #6b7280 (Secondary text, captions)
- **Dark Gray**: #1f2937 (Body text, content)
- **Very Dark**: #111827 (Headings, emphasis)

## Component Guidelines

### 1. Button Component

- **Variants**: primary, secondary, outline, ghost
- **Sizes**: sm, md, lg
- **States**: default, hover, active, disabled, loading
- **Usage**: CTAs, form submissions, navigation

Example:

```tsx
<Button variant="primary" size="md" className="w-full">
  Enroll Now
</Button>
```

### 2. Card Component

- Used for course listings, instructor profiles, testimonials
- Elevation shadow on hover
- Consistent padding (p-4 for mobile, p-6 for desktop)
- Radius: rounded-lg (12px)

Example:

```tsx
<Card className="hover:shadow-lg transition-shadow">
  <CardContent className="p-6">{/* content */}</CardContent>
</Card>
```

### 3. Input Component

- Consistent styling with Tailwind
- Clear focus states (ring-2 ring-primary-500)
- Error message support
- Placeholder text in neutral-400

### 4. Badge Component

- Category tags (blue background)
- Status badges (green for completed, orange for in-progress)
- Rating badges (with icons)

### 5. Typography

- **Headings**: font-bold, clear hierarchy (h1-h6)
- **Body Text**: font-normal, line-height-relaxed
- **Small Text**: text-sm, text-neutral-600 for captions

## Naming Conventions

### Components

- **UI Components**: `Button.tsx`, `Card.tsx`, `Input.tsx` (singular, PascalCase)
- **Feature Components**: `CourseCard.tsx`, `FilterSidebar.tsx` (descriptive names)
- **Layout Components**: `Header.tsx`, `Footer.tsx`, `Navbar.tsx`

### Files & Folders

- All TypeScript files: `.tsx` for React components, `.ts` for utilities
- Lowercase with hyphens for file/folder names if multi-word: `course-list.tsx`
- PascalCase for React component files: `CourseCard.tsx`

### Variables & Functions

- camelCase for variables and functions
- PrefixIconName for icon components: `StarIcon`, `PlayIcon`
- `use*` prefix for custom hooks: `useCourses`, `useSearch`

## Layout Pattern - Main Page Structure

### Header/Navigation (Fixed or Sticky)

```
Logo | Search Bar | Categories | Browse | Sign In | Enroll
```

### Hero Section

- Large gradient background (blue to purple)
- Headline text
- "Learn how AI is a busy teacher's best friend" style messaging
- CTA button
- Right side image/graphic

### Main Sections

1. **Trending Courses** - Carousel or grid of popular courses
2. **Browse All Courses** - Full course catalog with filters
3. **Course Categories** - Grid of category cards
4. **Top Instructors** - Carousel of instructor profiles
5. **Testimonials** - Learner success stories
6. **Call to Action** - "Start learning today"
7. **FAQ** - Accordion component

### Course Card Layout

```
[Course Image]
Category Badge | Rating Stars
Course Title (2 lines max)
Instructor Name
Price | Enroll Button
```

### Filter Sidebar

- Category checkboxes
- Price range slider
- Rating filter (stars)
- Level filter (Beginner, Intermediate, Advanced)
- Language filter
- Duration filter

## Color Usage in Components

### Buttons

- **Primary Action**: Primary-600 background, white text
- **Secondary Action**: Neutral-200 background, neutral-900 text
- **Ghost**: Transparent, primary-600 text

### Text

- **Headings**: neutral-900
- **Body**: neutral-700
- **Secondary**: neutral-600
- **Captions**: neutral-500

### Backgrounds

- **Card**: white (#ffffff)
- **Section**: neutral-50
- **Hover**: neutral-100
- **Accent**: primary-50 or secondary-50

### Borders

- **Default**: neutral-200
- **Focus**: primary-500
- **Error**: danger-500

## Accessibility Standards

- I18n support
- Semantic HTML (use proper heading hierarchy)
- ARIA labels for icon-only buttons
- Color contrast ratio at least 4.5:1
- Focus visible states on all interactive elements
- Keyboard navigation support
- Alt text for all images

## Performance Best Practices

- Lazy load course images
- Code split pages with React Router
- Use image optimization (consider next-gen formats)
- Memoize components that receive frequent props updates
- Use React.lazy() for non-critical routes

## State Management (for future)

- Start with React Context for global state (auth, theme)
- Consider Redux/Zustand if complexity grows
- Local state for form data

## Types Definition Pattern

```typescript
// types/course.ts
export interface Course {
  id: string;
  title: string;
  description: string;
  category: Category;
  instructor: Instructor;
  price: number;
  rating: number;
  totalRatings: number;
  students: number;
  thumbnail: string;
  duration: number; // in hours
  level: "Beginner" | "Intermediate" | "Advanced";
  tags: string[];
}

export interface Instructor {
  id: string;
  name: string;
  avatar: string;
  bio: string;
  rating: number;
  students: number;
}

export interface Category {
  id: string;
  name: string;
  icon: string;
  color: string; // tailwind color class
}
```

## Responsive Breakpoints

Follow Tailwind's default breakpoints:

- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

Mobile-first approach: Base styles for mobile, then add `md:`, `lg:` prefixes

## Icons (Lucide React)

- Import from `lucide-react`
- All icons sized: `w-4 h-4` (icon), `w-5 h-5` (button), `w-6 h-6` (heading)
- Icon color: inherit from text color or specify primary-600

Common icons:

- `Search`, `Menu`, `X`, `ChevronDown`, `ChevronRight`
- `Star`, `Heart`, `Share2`, `Play`, `Check`
- `Award`, `BookOpen`, `Users`, `Clock`, `BarChart3`

## Git Workflow

- Branch naming: `feature/course-list`, `fix/button-styling`
- Commit messages: Imperative mood, lowercase ("add course card component")
- Keep commits atomic and focused

## Performance Metrics to Monitor

- Core Web Vitals (LCP, FID, CLS)
- Time to Interactive (TTI)
- Bundle size

## Future Enhancements

- Dark mode toggle
- Video player component
- Rich text editor for course content
- Chat/messaging system for Q&A
- Progress tracking dashboard
- Certificate generation
- Payment integration
- Notification system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PDK-23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
