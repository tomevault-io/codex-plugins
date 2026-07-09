📋 Silent Circle — Copilot / AI Agent Instructions

This document explains the essential structure, conventions, and development workflows in this repository. Use it to make safe, productive edits, especially when working with AI (Gemini) integration and Firestore.

---

## 1) Big picture
- Next.js app (App Router) in `app/` with client components under `components/`.
- Firebase (Firestore + Auth) is initialized in `lib/firebase.js` and used for `posts` collection.
- A server API at `app/api/moderate/route.js` performs AI-based moderation using the `@google/generative-ai` SDK and a REST fallback.
- UI flow: `page.tsx` loads the main views. `NewPostForm.js` sends the draft to `/api/moderate` for validation -> if safe, saves to Firestore -> `Timeline.js` listens with `onSnapshot` to show posts.

## 2) How to run and debug locally
- Install and run dev server:
```bash
npm install
npm run dev
```
- Build and production preview:
```bash
npm run build
npm run start
```
- Linting:
```bash
npm run lint
```

## 3) Important environment variables
- Firebase client config (public): set in `.env.local` as `NEXT_PUBLIC_FIREBASE_*` keys used in `lib/firebase.js`.
- Gemini / Generative API (server):
  - Prefer `GEMINI_API_KEY` (server-only) for production in `.env.local`.
  - A `NEXT_PUBLIC_GEMINI_API_KEY` is allowed as a dev fallback but avoid shipping it in public repos.
  - Optional: `GENERATIVE_MODEL_CANDIDATES` (comma-separated model names) to control fallback models in `app/api/moderate/route.js`.

Example `.env.local` (local dev):
```
GEMINI_API_KEY=sk-... (server-only)
NEXT_PUBLIC_FIREBASE_API_KEY=...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=...
NEXT_PUBLIC_FIREBASE_PROJECT_ID=...
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=...
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=...
NEXT_PUBLIC_FIREBASE_APP_ID=...
GENERATIVE_MODEL_CANDIDATES=gemini-1.5,gemini-1.5-pro
```

## 4) Code patterns & conventions to follow
- Client vs server: Files inside `app/` are server components by default unless they include `"use client"` at the top. `page.tsx` and the components use `"use client"` when they rely on browser APIs (hooks, localStorage, DOM events).
- UI styles: The project uses CSS Modules (e.g., `components/NewPostForm.module.css` and `Timeline.module.css`) and `app/globals.css` (Tailwind imported).
- Firebase: `lib/firebase.js` initializes Firebase and calls `signInAnonymously()` in the browser — this is expected by components that save posts.
- Firestore collection: `posts` (document fields: `text`, `claps`, `createdAt`, `uid`). Keep field names stable if migrating or adding downstream features.
- LocalStorage: settings and like-tracking are stored client-side (`silent-circle-settings`, `likedPostIds`). Preserve these keys when changing the feature.

## 5) Moderation API specifics (critical)
- File: `app/api/moderate/route.js` — This is the gatekeeper for posting content.
- The route expects a POST JSON body: `{ text: string }` and returns JSON: `{ isSafe: boolean, reason: string, matchedWords: string[], usedModel?: string }`.
- The route:
  - Uses `GEMINI_API_KEY` (or `NEXT_PUBLIC_GEMINI_API_KEY` fallback).
  - Queries `listModels` via SDK or a REST endpoint to construct fallback candidates.
  - Attempts each candidate model via `genAI.getGenerativeModel().generateContent(prompt)` and expects strict JSON output.
  - Logs masked API keys and other debug info; do not print full keys or write them into the repo.

When editing this route:
- Keep the JSON response shape stable — frontend relies on boolean `isSafe`.
- If you change the prompt or format, update `NewPostForm.js` to parse the API response correctly.
- Avoid exposing the API key to the browser — prefer `GEMINI_API_KEY` (server-only) and keep the key in `.env.local`.

## 6) Suggested edits and examples
- Add a new moderation check (e.g., flagged content categories): modify the prompt in `app/api/moderate/route.js` and add a new `categories` array to the JSON response.
  Example: update the prompt string and `jsonResponse` fields then update `NewPostForm.js` to check `result.categories`.
- Add pagination to timeline: replace `query(collection(db, 'posts'), orderBy('createdAt', 'desc'))` with `limit(n)` and/or `startAfter` logic in `Timeline.js`.
- Add tests or simulate responses: since there are no unit tests, add jest + msw for mocking `fetch('/api/moderate')` responses.

## 7) Avoid common mistakes
- Do not store secret keys in the repo. For local testing you can set `NEXT_PUBLIC_GEMINI_API_KEY` but prefer `GEMINI_API_KEY` for server-only env.
- Do not remove the moderation step in `NewPostForm.js` — the app relies on the `/api/moderate` gate to block content server side.
- When editing Firestore field names, migrate all read/write sites (`NewPostForm.js` and `Timeline.js`).

---

If anything in this file is unclear or incomplete, tell me which sections you'd like expanded or examples added (e.g., sample `.env.local` values, more endpoints to mock, or a suggested PR checklist). ✅

---
> Source: [kin4k0/Silent-Circle](https://github.com/kin4k0/Silent-Circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-09 -->
