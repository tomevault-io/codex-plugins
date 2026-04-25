# CLAUDE.md

## Project Overview

This is a **pre-launch landing page** for "Autonomous AI CRM" - an AI-powered customer relationship management platform. The page is designed to collect waitlist signups for early MVP access.

## Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS v4
- **UI Components:** shadcn/ui
- **Animations:** Framer Motion
- **Icons:** Lucide React
- **Fonts:** Inter (UI/body), Calistoga (display headlines)

## Design System

### Color Palette
- **Primary Accent:** `#0052FF` (Electric Blue)
- **Accent Secondary:** `#4D7CFF` (gradient endpoint)
- **Background:** `#FAFAFA` (off-white)
- **Foreground:** `#0F172A` (deep slate)
- **Muted:** `#F1F5F9` (slate-100)
- **Card:** `#FFFFFF` (pure white)
- **Border:** `#E2E8F0` (slate-200)

### Typography
- **Display/Headlines:** Calistoga (serif) - used for H1/H2
- **Body/UI:** Inter (sans-serif) - used for all other text
- **Monospace:** JetBrains Mono - used for section labels/badges

### Key Design Patterns

#### Gradient Text
```tsx
<span className="gradient-text">Highlighted Word</span>
```
Uses `bg-gradient-to-r from-[#0052FF] to-[#4D7CFF]` with `bg-clip-text text-transparent`.

#### Section Labels
```tsx
<div className="section-label">
  <Icon className="w-4 h-4" />
  <span>Section Name</span>
</div>
```

#### Primary Button
```tsx
<Button className="gradient-btn text-white rounded-full">
  Call to Action
</Button>
```

#### Card Hover Effect
```tsx
<div className="group p-6 rounded-xl bg-card border border-border hover:border-[#0052FF]/30 transition-all hover:shadow-lg hover:shadow-[#0052FF]/5">
  {/* content */}
</div>
```

### Animation Patterns

#### Fade In Up (Framer Motion)
```tsx
const fadeInUp = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0, transition: { duration: 0.6 } }
};

<motion.div initial="initial" whileInView="animate" viewport={{ once: true }} variants={fadeInUp}>
```

#### Stagger Children
```tsx
const staggerContainer = {
  animate: { transition: { staggerChildren: 0.15 } }
};
```

## Project Structure

```
my-app/
├── src/
│   ├── app/
│   │   ├── sections/           # Page sections (10 total)
│   │   │   ├── HeroSection.tsx
│   │   │   ├── ProblemSection.tsx
│   │   │   ├── SolutionSection.tsx
│   │   │   ├── EcosystemSection.tsx
│   │   │   ├── FeaturesSection.tsx
│   │   │   ├── HowItWorksSection.tsx
│   │   │   ├── WaitlistSection.tsx
│   │   │   ├── UsersSection.tsx
│   │   │   ├── VisionSection.tsx
│   │   │   └── FinalCTASection.tsx
│   │   ├── components/         # Shared components
│   │   │   ├── Navigation.tsx
│   │   │   └── Footer.tsx
│   │   ├── globals.css         # Global styles, design tokens
│   │   ├── layout.tsx          # Root layout with fonts
│   │   └── page.tsx            # Main page composition
│   ├── components/ui/          # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── input.tsx
│   │   ├── badge.tsx
│   │   └── separator.tsx
│   └── lib/
│       └── utils.ts            # Utility functions (cn helper)
├── dist/                       # Static build output
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

## Section IDs for Navigation

- `#problem` - Problem Section
- `#solution` - Solution Section
- `#features` - Features Section
- `#how-it-works` - How It Works Section
- `#waitlist` - Waitlist/MVP Access Section

## Common Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Static export output goes to /dist
```

## Component Guidelines

### Creating New Sections

1. Create a new file in `src/app/sections/`
2. Use "use client" directive for Framer Motion animations
3. Follow the existing pattern:
   - Section label at top
   - Headline with gradient text highlight
   - Subheadline with muted-foreground color
   - Content with staggered animations
   - Section ID for navigation

### Styling Guidelines

- Use Tailwind classes exclusively
- Prefer `rounded-xl` (12px) or `rounded-2xl` (16px) for cards
- Use `gap-6` or `gap-8` for grid gaps
- Use `max-w-6xl mx-auto` for container width
- Use `py-24` for section vertical padding

### Animation Guidelines

- All sections should fade in when scrolling into view
- Use `viewport={{ once: true, margin: "-100px" }}` for scroll triggers
- Keep animations subtle and professional
- Duration: 0.5-0.7s for entrances
- Use stagger for lists/grids

## Build Configuration

The project is configured for static export:
- `output: "export"` in next.config.ts
- `distDir: "dist"` for output
- Images are unoptimized for static export

## Adding shadcn/ui Components

```bash
npx shadcn@latest add [component-name]
```

Available components: https://ui.shadcn.com/docs/components

## Important Notes

- The page is a **pre-launch landing page** focused on conversion
- All CTAs should scroll to `#waitlist` section
- Form submissions are simulated (no backend connected)
- Keep animations subtle - avoid flashy effects
- Maintain accessibility: proper contrast, focus states, semantic HTML

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SwatiSTiwari) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
