# GitHub Copilot Instructions for Mindtaker

## Project Overview

**Mindtaker** is a React Native mobile application designed to provide users with a clean, organized space for capturing and managing their daily thoughts. The app features AI-powered enhancements to help users expand and refine their ideas.

### Project Goal

Create an intuitive mobile app that allows users to:

- Capture thoughts and ideas quickly and efficiently
- Organize ideas by categories and dates
- Leverage AI to expand, summarize, and enhance their thoughts
- Maintain a clean, modern user interface with excellent UX

## Technical Stack

### Core Framework

- **React Native 0.79.5** with **Expo SDK 53** - Cross-platform mobile development
- **TypeScript** - Type safety and better development experience
- **Expo Router 5.1** - File-based navigation system

### Authentication & Backend

- **Supabase** - Backend-as-a-Service for authentication, database, and storage
- **Google OAuth** via `@react-native-google-signin/google-signin` - Social authentication
- **Supabase Auth** - User session management

### Styling & UI

- **NativeWind 4.1** - Tailwind CSS for React Native styling
- **React Native Calendars** - Calendar component for date-based organization
- **Expo Linear Gradient** - Gradient backgrounds and styling
- **Lottie React Native** - Smooth animations

### State Management

- **Zustand 5.0** - Lightweight state management for global app state
- **React Hooks** - Local component state management

### Navigation

- **Expo Router** - File-based routing with tabs and stack navigation
- **React Native Safe Area Context** - Proper safe area handling

## Main Features

### 1. Authentication System

- **Google OAuth Integration** - Seamless sign-in with Google accounts
- **Session Management** - Persistent authentication with Supabase
- **Auto-redirect** - Automatic navigation based on auth state

### 2. Ideas Management

- **Daily Idea Capture** - Quick text input for thoughts and ideas
- **Category Organization** - Ideas organized by predefined categories (General, Personal, Business, Health)
- **Date-based Filtering** - Calendar integration for viewing ideas by date
- **CRUD Operations** - Create, read, update, and delete ideas

### 3. Calendar Interface

- **Expandable Calendar** - Interactive calendar for date navigation
- **Date Selection** - Filter ideas by selected dates
- **Visual Indicators** - Show days with captured ideas

### 4. AI Enhancement

- **AI-powered Actions** - Expand or summarize ideas using AI
- **Content Enhancement** - Improve and refine user thoughts
- **Smart Suggestions** - AI-driven content recommendations

### 5. User Interface

- **Tab Navigation** - Clean two-tab interface (Home, Profile)
- **Modal Screens** - Stack navigation for idea creation/editing
- **Safe Area Support** - Proper handling of device notches and status bars
- **Pull-to-Refresh** - Refresh data with swipe gesture
- **Loading States** - Smooth loading indicators and skeletons

## Development Guidelines

### File Structure

```
app/
├── _layout.tsx              # Root layout with auth handling
├── auth.tsx                 # Authentication screen
├── (tabs)/                  # Tab navigation group
│   ├── _layout.tsx         # Tabs layout configuration
│   ├── (home)/             # Home tab group
│   │   └── index.tsx       # Ideas list with calendar
│   ├── profile.tsx         # User profile screen
│   └── ideas/              # Ideas stack (modals)
│       ├── new.tsx         # Create new idea
│       └── [id].tsx        # Edit existing idea
```

### Key Patterns

- Use **TypeScript** for all new code with proper type definitions
- Follow **Expo Router** conventions for navigation
- Implement **Zustand stores** for global state management
- Use **NativeWind classes** for styling with Tailwind CSS syntax
- Handle **loading states** and **error boundaries** appropriately
- Implement **Safe Area** handling for all screens

### Database Schema

- **Users** - User profiles and authentication data
- **Categories** - Predefined idea categories with enum types
- **Ideas** - User thoughts with content, category, and timestamp

### Environment Variables

```
EXPO_PUBLIC_SUPABASE_URL=your_supabase_url
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID=your_google_web_client_id
EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID=your_google_ios_client_id
```

## Code Quality Standards

- Maintain **consistent naming conventions**
- Implement **proper error handling** with user-friendly messages
- Use **React best practices** (hooks, memoization, proper dependencies)
- Follow **accessibility guidelines** for mobile apps
- Write **self-documenting code** with clear variable and function names
- Handle **edge cases** and loading states gracefully

## Common Tasks

- Adding new idea categories requires updating the `CategoryEnumType` enum
- New screens should follow the established navigation patterns
- API calls should include proper error handling and loading states
- UI components should be responsive and follow the design system
- Always test authentication flows and session management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dromediansk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
