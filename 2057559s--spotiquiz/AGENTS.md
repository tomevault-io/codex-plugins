<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Spotiquiz - Spotify Data Quiz Web App

## Project Overview
A Next.js web application that allows users to import their Spotify listening history and participate in interactive music trivia based on their personal data.

## Core Features
- **Spotify Authentication**: OAuth 2.0 integration with Spotify API
- **Data Import**: Import user's top tracks, artists, and listening history
- **Quiz System**: 
  - When did you listen to this song the most?
  - How many times have you listened to this song?
  - Identify artists from songs
  - Track popularity guesses
- **Beautiful UI**: Modern, responsive design using Tailwind CSS

## Tech Stack
- **Framework**: Next.js 14+ with TypeScript
- **Styling**: Tailwind CSS
- **Authentication**: Spotify OAuth 2.0
- **Database**: Consider using Vercel KV or similar for session storage
- **State Management**: React hooks and context API

## Key Files & Structure
```
src/
  app/           - Next.js App Router pages
  components/    - Reusable React components
  lib/           - Utility functions and API helpers
  types/         - TypeScript type definitions
  contexts/      - React context for state management
```

## Development Guidelines
1. Keep components small and focused
2. Use TypeScript for type safety
3. Follow Next.js App Router conventions
4. Implement proper error handling for API calls
5. Ensure responsive design on mobile devices
6. Handle user authentication securely

## Important Integration Points
- Spotify Web API for data fetching
- Session management for user authentication
- Quiz logic and scoring system

---
> Source: [2057559s/SpotiQuiz](https://github.com/2057559s/SpotiQuiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-24 -->
