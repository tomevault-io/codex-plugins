# Application Development Guidelines

## Overview
- This application is a cross-platform React Native application built with Expo.
- The application is designed to run on smartphones.
- The UI framework should use **Gluestack** and **NativeWindUI**.
- The programming language for the application is **TypeScript**.

## Project Structure Setup
- Follow this recommended project structure:
  ```
  dive_timers/
  ├── src/
  │   ├── assets/            # Images, fonts, and other static assets
  │   ├── components/        # Reusable UI components
  │   │   ├── common/        # Shared/generic components
  │   │   └── features/      # Feature-specific components
  │   ├── constants/         # App-wide constants
  │   ├── features/          # Feature-specific code
  │   │   └── [feature]/
  │   │       ├── components/
  │   │       ├── screens/
  │   │       ├── hooks/
  │   │       └── types.ts
  │   ├── hooks/             # Custom React hooks
  │   ├── models/            # Data models/types
  │   ├── navigation/        # Navigation configuration
  │   ├── screens/           # Screen components
  │   ├── services/          # API services, external integrations
  │   ├── store/             # State management
  │   ├── theme/             # UI theme definitions
  │   ├── types/             # TypeScript type definitions
  │   └── utils/             # Utility functions
  ├── app.json              # Expo configuration
  ├── App.tsx               # Root component
  ├── babel.config.js       # Babel configuration
  ├── package.json          # Dependencies and scripts
  ├── tsconfig.json         # TypeScript configuration
  └── README.md             # Project documentation
  ```

## Environment Setup
- Required Development Tools:
  - Node.js (v18 or newer)
  - npm (v9 or newer) or Yarn (v1.22 or newer)
  - Expo CLI
  - VSCode with recommended extensions:
    - ESLint
    - Prettier
    - React Native Tools
    - TypeScript React code snippets

- Environment Configuration:
  - Use `.env` files for environment variables with `react-native-dotenv`
  - Create separate environments:
    - `.env.development` - Development environment variables
    - `.env.staging` - Staging environment variables
    - `.env.production` - Production environment variables
  - Define environment variable types in a `env.d.ts` file

- Dependencies Management:
  - Required Core Dependencies:
    - expo: "~49.0.0"
    - react: "18.2.0"
    - react-native: "0.72.x"
    - @gluestack-ui/themed: "^0.1.x"
    - nativewind: "^2.x.x"
    - axios: "^1.x.x"
    - react-navigation/native: "^6.x.x"

- Development Workflow:
  - Start the development server: `expo start` or `npm start`
  - Run on iOS: `npm run ios`
  - Run on Android: `npm run android`
  - Run tests: `npm test`
  - Lint code: `npm run lint`
  - Format code: `npm run format`

## TypeScript Development Best Practices
- When developing in TypeScript, adhere to the following guidelines:

### Strong Typing
- Use strong typing everywhere.
- Define interfaces and types for:
    - Function inputs and outputs.
    - Components.
    - API responses.
- Avoid using `any` unless absolutely necessary.

### Code Organization
- Organize code with a modular folder structure.
- Group files by feature or domain (e.g., `models/`, `services/`, `controllers/`, `types/`, etc.).
- Use `index.ts` files to re-export modules cleanly.

### Value Constraints
- Use enums and literal types for clear value constraints.
- Replace magic strings/numbers with enums or string union types.

### API Integration
- Define API request/response types.
- Use interfaces or OpenAPI-generated types to ensure type safety with backends.
- Use **Axios** with typed request/response models.

### Reusability
- Leverage generics for reusability:
    - Use generics in utility functions, services, or data models to keep code DRY and flexible.

### TypeScript Configuration
- Configure TypeScript for strictness:
    - Enable `"strict": true` in `tsconfig.json`.

### Code Quality
- Use **ESLint** and **Prettier** for consistent code:
    - Add the `@typescript-eslint` plugin to catch TypeScript-specific issues.
    - Format code automatically with Prettier for readability.

### Testing
- Write type-safe tests:
    - Use testing frameworks like **Jest** or **Vitest** with TypeScript support.
    - Create typed mocks and ensure inputs/outputs are covered.
- Ensure comprehensive test coverage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chetbackiewicz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
