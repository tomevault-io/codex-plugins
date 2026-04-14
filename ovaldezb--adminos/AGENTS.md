# GitHub Copilot Instructions for AdminOS

## Project Overview
AdminOS is a modern, glassy-style multi-tenant dashboard application built with Angular and styled using Tailwind CSS and DaisyUI.

The app provides tools for condominium and operations management — enabling administrators to manage users, payments, tickets, reports, and communications efficiently.

---

## Core Features & Modules
AdminOS includes the following core modules:
- **Authentication & Roles:** Login, register, password recovery, role-based access control.
- **Dashboard Overview:** KPIs, charts, recent activity.
- **Users & Roles Management:** CRUD operations with modals and tables.
- **Tickets & Tasks:** Create, assign, and resolve incidents or work orders.
- **Reports:** Data visualization, filtering, and export (Excel/PDF).
- **Notifications:** Toasts, alerts, in-app updates.
- **Settings:** Tenant configuration, profile management, and preferences.

> **Note:** Component, service, and model names must always be written in **English**, but the **HTML content (labels, text, titles, etc.) must be in Spanish**.

---

## Technology Stack

### Core Framework
- **Angular (v20+)** — Using standalone components.
- **TypeScript** — Strongly typed development.

### Styling & UI
- **Tailwind CSS v4** — Utility-first CSS framework.
- **DaisyUI v5** — Tailwind CSS component library.
- **Remix Icons** — Icon library (ri-icon-name-fill or ri-icon-name-line).

---

## Design System - PREMIUM & MODERN

### Enhanced Color Palette (Ultra High Contrast)

**Primary Colors (Modern Sky Blue):**
- Primary: #0284c7 (Sky Blue 600) - Main actions, buttons, CTAs
- Primary Dark: #0369a1 (Sky Blue 700) - Hover states, active states
- Primary Light: #e0f2fe (Sky Blue 100) - Backgrounds

**Secondary Colors (Rich Violet):**
- Secondary: #7c3aed (Violet 600) - Secondary actions, accents
- Secondary Dark: #6d28d9 (Violet 700) - Hover states
- Secondary Light: #ede9fe (Violet 100) - Backgrounds

**Semantic Colors (High Saturation):**
- Success: #059669 (Emerald 600) - Completed, active status
- Warning: #d97706 (Amber 600) - Pending, caution, alerts
- Error: #dc2626 (Red 600) - Errors, rejected, critical
- Info: #2563eb (Blue 600) - Information, details, hints

**Neutral Colors (Professional High Contrast):**
- Text Primary: #171717 (Neutral 900) - Main text, headings [ALWAYS in modals]
- Text Secondary: #404040 (Neutral 700) - Secondary text
- Text Tertiary: #737373 (Neutral 500) - Tertiary text, placeholders
- Border: #d4d4d4 (Neutral 300) - Borders, dividers
- Background: #fafafa (Neutral 50) - Page backgrounds
- Surface: #ffffff (White) - Cards, modals, elevated surfaces

### Icon Sizing Guidelines
- **Small icons:** `text-sm` (0.875rem / 14px) — Inline with text, badges, helper icons
- **Regular icons:** `text-base` (1rem / 16px) — Buttons, list items, default use (most common)
- **Medium icons:** `text-lg` (1.125rem / 18px) — Card headers, secondary emphasis, section headers
- **Large icons:** `text-xl` (1.25rem / 20px) — Page headers, primary section titles
- **Extra large:** `text-2xl` (1.5rem / 24px) — Dashboard stats, hero sections, KPI cards
- **Jumbo:** `text-5xl` (3rem / 48px) — Empty states, large visual indicators

**Icon Color Guidelines:**
- Always use semantic colors matching the context: `text-primary`, `text-success`, `text-error`, `text-warning`
- Use `text-neutral-600` for secondary/helper icons
- Use `text-neutral-400` for disabled/inactive icons
- Icons in white backgrounds should use `text-primary` or semantic colors (NEVER gray on light backgrounds for main icons)

### Design Principles - Premium & Modern

#### Visual Hierarchy & Contrast
- **Text on White/Light:** ALWAYS use `text-neutral-900` for headings, `text-neutral-800` for body text
- **Text on Colored Backgrounds:** Use `text-white` or semantic text colors with 4.5:1 minimum contrast ratio
- **Labels & Form Text:** Use `text-neutral-900` with font-weight 600 (bold) for visibility
- **Modal Text:** ALL text in modals must be `text-neutral-900` - NO light text on white backgrounds
- **Contrast Requirement:** Always test with WCAG AA compliance (4.5:1 for normal text, 3:1 for large text)

#### Glassy Effect (Modern & Attractive)
- **Use `.glass` class for:** Cards, containers, modals (applies proper backdrop blur and opacity)
- **Glassy Backgrounds:** `background: rgba(255, 255, 255, 0.98)` with `backdrop-filter: blur(20px)`
- **Glassy Borders:** Use `border: 1px solid rgba(255, 255, 255, 0.3)` for subtle definition
- **Modal Backgrounds:** Use pure white `#ffffff` with strong shadow for maximum visibility
- **Modal Borders:** Use subtle primary color border `rgba(2, 132, 199, 0.15)` for elegance

#### Spacing & Layout
- **Card Padding:** `p-6` for standard cards, `p-4` for compact areas, `p-8` for spacious layouts
- **Gap Between Elements:** Use `gap-3` for compact, `gap-4` for normal, `gap-6` for spacious layouts
- **Border Radius:** `rounded-2xl` (1rem) for cards/buttons, `rounded-xl` (0.75rem) for secondary elements
- **Section Margins:** `mb-6` between major sections for visual breathing room

#### Shadows & Depth
- **Cards:** `shadow-lg` — 0 10px 15px -3px rgba(0,0,0,0.1)
- **Elevated (Modals):** `shadow-xl` — 0 20px 25px -5px rgba(0,0,0,0.1)
- **Interactive (Hover):** `shadow-2xl` — 0 25px 50px -12px rgba(0,0,0,0.25)
- **Subtle:** `shadow-sm` — 0 1px 2px 0px rgba(0,0,0,0.05)

#### Button & Interactive Elements
- **Style:** Use gradient backgrounds for primary/secondary buttons
- **Primary Button:** `btn-primary` with gradient `#0284c7` → `#0369a1`
- **Hover State:** Lift up 2px, add colored shadow, use lighter gradient variant
- **Active State:** Return to baseline, use darker gradient
- **Font Weight:** Always `font-weight: 600` (bold) for button text
- **Transition:** `transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1)` for smooth animations

#### Badges & Labels
- **Style:** Use gradient backgrounds with semi-transparent fill
- **Badge Color:** Match semantic intent: primary (info), secondary (actions), success, error, warning
- **Borders:** Use colored borders with opacity for definition
- **Font Weight:** `font-weight: 600` (bold) for readability
- **Padding:** `py-1 px-3` for standard, adjust as needed

#### Alerts & Notifications
- **Background:** Use gradient with low opacity of the semantic color
- **Border:** Use 1.5px border in the semantic color
- **Text Color:** Use darker shade of the semantic color for text
- **Icon Color:** Match the semantic color
- **Example Error Alert:**
  - Background: `linear-gradient(135deg, rgba(220, 38, 38, 0.08) 0%, rgba(220, 38, 38, 0.04) 100%)`
  - Border: `1.5px solid #dc2626`
  - Text: `#7f1d1d` (dark red)

---

## Code Style Guidelines

### Angular Components
```typescript
// Standalone Angular Component Example
import { Component, signal } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './user-list.component.html',
  styleUrl: './user-list.component.css'
})
export class UserListComponent {
  protected readonly users = signal<User[]>([]);
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ovaldezb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
