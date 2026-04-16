# 🤖 GitHub Copilot Instructions – Synapse App

These are project-specific development rules and setup steps for **Synapse**, our internal chat application built using **React Native** and **Copilot Agents**.

---

## 🧩 1. Project Overview

**Project Name:** Synapse  
**Goal:** Internal chat platform  
**Tech Stack:** React Native + Node.js (backend) + PostgreSQL.  
**Focus Areas:**  
- Reusable components  
- Modular architecture  
- Consistent theming  
- Type-safe and maintainable code  
- Industry-standard naming and structure  

---

## ⚙️ 2. Environment Setup

**Prerequisites:**
- Node.js ≥ 18  
- npm or yarn  
- React Native CLI or Expo CLI  
- Android Studio / Xcode  
- GitHub Copilot Agents enabled  


---

#### 2. `.github/DESIGN_SYSTEM.md`
> **Purpose:** Makes Copilot’s UI generations consistent — spacing, fonts, radii, shadows, etc.

```markdown
# 🎨 Synapse Design System

Copilot Agents must follow these UI and styling rules to ensure a consistent look across components.

## 🧱 Base Style Tokens

| Property | Light | Dark |
|-----------|--------|------|
| Background | #FFFFFF | #000000 |
| Text Primary | #000000 | #FFFFFF |
| Accent | #007AFF | #0A84FF |
| Card | #F7F7F7 | #1C1C1E |

## 🔤 Typography
| Type | Font Size | Weight |
|------|------------|--------|
| Heading | 24px | 700 |
| Subheading | 18px | 600 |
| Body | 16px | 400 |
| Caption | 14px | 400 |

## 📏 Layout Spacing
Use multiples of **8px** for padding and margins:

## 🧩 Border Radius
Small: 8px
Medium: 12px
Large: 20px
Button: 50px (rounded)


## 🌫️ Shadows (Elevation)
Light mode: rgba(0,0,0,0.1)
Dark mode: rgba(255,255,255,0.08)

perl
Copy code

## 🧠 Copilot Rules
- Always use theme colors (no hardcoded hex values)
- Always apply `borderRadius` and consistent padding
- Avoid inline styles; use a `styles.js` file
- Include responsiveness via `Dimensions` or `useWindowDimensions`

# 🤖 Copilot Agent Guide – Synapse

## 🧠 Coding Style
- Always use **functional components** with hooks  
- Use `useTheme()` from `/src/theme/`  
- Keep styles in `/styles.js`  
- Use ES modules (`import/export`)  
- Type-check with PropTypes or TypeScript interfaces  

## 🧩 Component Generation Rules
When creating new components:
- Create a folder per component (`/Button/`, `/ChatItem/`)
- Include `index.js` and `styles.js`
- Respect theme and spacing guidelines from DESIGN_SYSTEM.md

## 🎨 UI Behavior
- Respect dark/light theme values
- Use icons from `react-native-vector-icons`
- Use consistent elevation and padding
- Animate transitions via `react-native-reanimated` or `Animated`

## 🧭 Navigation Rules
- Use `@react-navigation` for tab and stack navigation
- Define routes in `/navigation/RootNavigator.js`
- Use descriptive screen names: `LoginScreen`, `ChatScreen`, etc.

## 📡 Data and Context
- Access user/session data via `/context/AuthContext.js`
- Access chat/contact data via `/context/ChatContext.js`
- Avoid prop drilling — use context hooks instead

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kavatideepak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
