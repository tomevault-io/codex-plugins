# WeTech Frontend - Copilot Instructions

## Architecture Overview

**WeTech-FE** is a React-based e-learning platform frontend with admin and user dashboards. The app uses **Context API** for global state management and **axios** for API communication with a Spring Boot backend.

### Core Data Flows

- **Authentication**: Token stored in `sessionStorage`, user data in `localStorage`. `AuthContext` manages login state and syncs with `/api/auth/get-info` endpoint.
- **Cart Management**: `CartContext` fetches cart items from `/cart/get-item` and maintains count.
- **Notifications**: `NotificationContext` provides toast-like notifications with auto-dismiss. All API errors trigger notifications via axios interceptor.
- **Real-time Payments**: `usePaymentSocket` hook connects to WebSocket `/wss` endpoint and subscribes to `/topic/payment/{userId}`.

## Project Structure & Patterns

### Context Providers (wrap entire app in `App.js`)

- `AuthProvider` - User authentication, retrieves from `/api/auth/get-info`
- `CartProvider` - Shopping cart state
- `NotificationProvider` - Global toast notifications
- Providers are stacked; `CartProvider` wraps routes inside `NotificationProvider`

### API Communication

- **authAxios** (in [src/services/axios-instance.js](src/services/axios-instance.js)): Adds `Authorization: Bearer {token}` to all requests. Intercepts 401/403 errors, clears token, redirects to `/login`
- **publicAxios**: For unauthenticated endpoints
- **Notification Integration**: Authentication errors trigger notifications via `showAuthErrorNotification()` (set in [App.js](App.js#L42))

### Component Organization

- **[src/components](src/components/)**: 40+ reusable UI components (CourseCard, NavBar, ChatWidget, etc.) with CSS modules
- **[src/pages/User](src/pages/User/)**: User workflows (Login, Home, DetailCourse, CartPage, Profile, etc.)
- **[src/pages/Admin](src/pages/Admin/)**: Admin dashboards (DashBoard, ListCourse, Transactions, etc.)
- **[src/context](src/context/)**: Four context files - Auth, Cart, DetailCourse, Notification
- **[src/hooks](src/hooks/)**: Custom hooks (`useNotification`, `useAuth`, `useCart`)

### Styling & Assets

- **CSS Modules**: Each component has `.module.css` (e.g., `CourseCard.jsx` + `CourseCard.module.css`)
- **Lottie Animations**: JSON files in [src/assets](src/assets/) (login-animation.json, purchase.json, etc.)
- **Icons/Images**: Imported in components (avatar_user.png, etc.)

## Critical Workflows

### Building & Deployment

```bash
npm start           # Dev server on http://localhost:3000
npm run build       # Production build (with --no-eslint flag)
npm run format      # Prettier formatting
npm test            # Run tests
```

### Environment Setup

- **REACT_APP_BACKEND_URL**: Base URL for axios instances (set in `.env`)
- Built with **Node 20 Alpine** and deployed via **Nginx** (see [Dockerfile](Dockerfile))

## Key Conventions & Gotchas

### Notification Pattern

- Always use `useNotification()` hook for user feedback (not console.log)
- Example: `const { showSuccess, showError } = useNotification(); showSuccess("Added to cart")`
- Notification helper auto-initializes in `App.js` via `setNotificationHelper()`

### Token & Auth Flow

- Token is **sessionStorage** (cleared on logout), user data in **localStorage** (persistent across refreshes)
- `AuthContext.fetchUser()` runs on app load to restore session if token exists
- On auth error (401/403), axios interceptor automatically logs out and redirects to login page

### Cart Updates

- After adding/removing courses, manually call `fetchCartCount()` from `useCart()` to refresh the count
- Cart items endpoint `/cart/get-item` filters out null items (see [CartContext.js](src/context/CartContext.js#L19))

### WebSocket Subscriptions

- Used only for payment notifications in `RegisterPayment` page
- Client connects to `/wss` and subscribes to `/topic/payment/{userId}`
- Always cleanup connection in component unmount

### Form & API Patterns

- Forms submit to backend API and trigger notifications on success/error
- Example: LoginForm → `authAxios.post("/api/auth/login")` → update AuthContext → redirect
- Course management: CourseForm submits to backend; CourseContent displays enrolled courses

## External Integrations

- **Google OAuth**: Google login via `@react-oauth/google` (`GoogleLoginButton` component)
- **Excel Export**: Use `xlsx` package for downloading data (e.g., in Admin transaction reports)
- **WebSocket**: `@stomp/stompjs` + `sockjs-client` for real-time payment updates

## Common Development Tasks

**Adding a new page**: Create folder in [src/pages/User](src/pages/User/) (or Admin), add component + route in [App.js](App.js)

**Adding a notification**: Import and call `useNotification()` hook; trigger from API response handlers

**Fetching user data**: Use `useAuth()` hook to access current user from AuthContext

**Updating cart after purchase**: Call `cartContext.fetchCartCount()` after successful payment

**Styling components**: Create `.module.css` alongside component file and import as `import styles from "./Component.module.css"`

---

**Last Updated**: 2026-01-28  
**Backend Base URL**: Configured via `REACT_APP_BACKEND_URL` environment variable

---
> Source: [envidi3107/wetech-fe](https://github.com/envidi3107/wetech-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
