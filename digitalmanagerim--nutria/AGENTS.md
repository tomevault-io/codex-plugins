# CLAUDE.md — NutrIA Project

## What is this project?

NutrIA is a body recomposition coaching PWA (Progressive Web App) with AI and gamification (Duolingo-style). The mascot is "Nuri", an otter that acts as a personal nutritionist and trainer. The name is a triple wordplay: Nutrición + IA + Nutria.

**Full specifications**: See `docs/NutrIA_Especificaciones_Proyecto.md` for complete details.

---

## Tech Stack

- **Frontend**: React 18 + Vite + Tailwind CSS + Framer Motion + React Router
- **Backend**: Node.js + Express
- **Database**: PostgreSQL with Prisma ORM
- **AI**: OpenAI GPT-4o (vision + text) — all responses use Nuri's personality
- **Auth**: JWT (access + refresh tokens) with bcrypt
- **PWA**: Service Worker + Web App Manifest
- **Language**: TypeScript (both client and server)

---

## Project Structure

```
nutria/
├── client/                     # React PWA (Vite)
│   ├── public/
│   │   ├── manifest.json
│   │   ├── sw.js
│   │   ├── icons/
│   │   └── nuri/               # Nuri SVG illustrations
│   ├── src/
│   │   ├── components/
│   │   │   ├── common/         # Button, Card, Modal, ProgressBar, Input, etc.
│   │   │   ├── nuri/           # NuriAvatar, NuriBubble, NuriReaction, NuriFAB
│   │   │   ├── chat/           # ChatWindow, ChatBubble, ChatInput, ActionButton
│   │   │   ├── onboarding/     # Step1Register, Step2Basics, Step3Measurements, etc.
│   │   │   ├── nutrition/      # MealLog, DailyTracker, WeeklyMenu
│   │   │   ├── training/       # WorkoutPlan, ActiveSession, ExerciseCard
│   │   │   ├── weight/         # WeightEntry, WeightChart
│   │   │   ├── gamification/   # XPBar, LevelBadge, StreakCounter, AchievementCard
│   │   │   └── stats/          # Charts, WeeklySummary
│   │   ├── pages/              # Splash, Login, Register, Onboarding, Dashboard, etc.
│   │   ├── hooks/              # useAuth, useUser, useChat, useGamification
│   │   ├── context/            # AuthContext, UserContext, ChatContext
│   │   ├── services/           # api.ts (axios instance), auth.ts, chat.ts, food.ts, etc.
│   │   ├── utils/              # formatters, validators, constants
│   │   ├── assets/             # Images, sounds
│   │   └── styles/             # Global styles, Tailwind config
│   ├── index.html
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── package.json
│
├── server/                     # Express API
│   ├── src/
│   │   ├── routes/             # auth.ts, onboarding.ts, food.ts, weight.ts, chat.ts, etc.
│   │   ├── controllers/        # Same structure as routes
│   │   ├── middleware/         # auth.ts (JWT verify), upload.ts (multer), rateLimiter.ts
│   │   ├── services/
│   │   │   ├── ai.ts           # OpenAI integration, all prompts
│   │   │   ├── chat.ts         # Context builder, memory management, streaming
│   │   │   └── gamification.ts # XP calculation, level up, streak tracking
│   │   ├── prompts/            # System prompt templates for each AI feature
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── migrations/
│   │   ├── seeds/              # achievements.ts, initial data
│   │   └── index.ts            # Express app entry
│   ├── tsconfig.json
│   └── package.json
│
├── docs/
│   └── NutrIA_Especificaciones_Proyecto.md
├── docker-compose.yml
├── .env.example
├── CLAUDE.md                   # This file
└── README.md
```

---

## Design System

### Colors
```
Primary (turquoise):   #00B4D8  — main actions, water/otter theme
Secondary (green):     #58CC02  — progress, success
Accent (orange):       #FF9600  — streaks, fire, XP, energy
Alert (coral red):     #FF4B4B  — streak danger, deficit
Background:            #FFFFFF / #F0F9FF (light blue tint)
Text:                  #2D3748
Gold (achievements):   #FFC800
Nuri brown:            #8B6914  — warm accents
```

### Typography
- Font: **Nunito** (Google Fonts) — Bold for titles, Regular for body, SemiBold for data
- Nuri's messages: Nunito Medium Italic

### UI Principles
- Mobile-first, max-width 480px centered on desktop
- Border radius: 16px on cards
- Min touch target: 48px
- Framer Motion for all transitions and micro-animations
- Nuri speech bubbles: comic-style with tail pointing to Nuri avatar
- Bottom nav with 5 tabs + floating Nuri FAB for chat
- Duolingo-inspired: colorful, playful, rewarding, addictive

---

## Nuri — The Otter Mascot

Nuri is the soul of the app. She's a European otter who works as a sports nutritionist and personal trainer.

**Personality**: Friendly, direct, motivational, funny. Makes otter/fish references. Never condescending. Celebrates wins enthusiastically, points out improvements kindly.

**Visual states** (SVG illustrations, cartoon flat style):
- Normal (standing, smiling, hands on hips)
- Chef (chef hat, ladle) — food registration
- Fitness (headband, dumbbells) — training
- Scientist (lab coat, glasses) — analytics, weight, stats
- Celebrating (jumping, confetti) — achievements
- Fire (flames around) — active streak 3+ days
- Sleeping (on rock, pizza nearby) — 2+ days inactive
- Worried (biting nails) — streak at risk
- Thinking (chin on paw) — AI loading
- Sad (puppy eyes, thumbs up) — streak lost
- Explorer (backpack, map) — onboarding
- Medal (podium, medal) — challenge completed

**For Phase 1**: Generate placeholder SVG illustrations for at least: normal, chef, fitness, scientist. Use simple, consistent cartoon style. Nuri is brown (#8B6914), round face, small ears, friendly eyes, stands upright.

---

## Current Phase: PHASE 1 — MVP

### What to build (in order):

#### 1. Project Setup
- Initialize monorepo with `client/` and `server/`
- Client: Vite + React 18 + TypeScript + Tailwind + Framer Motion + React Router
- Server: Express + TypeScript + Prisma + PostgreSQL
- Docker Compose: postgres service + volumes
- PWA manifest + basic service worker
- Environment variables (.env.example)

#### 2. Database Schema (Prisma)
```prisma
model User {
  id                  String   @id @default(uuid())
  name                String
  email               String   @unique
  passwordHash        String
  sex                 String?
  age                 Int?
  heightCm            Float?
  hasSmartScale       Boolean  @default(false)
  activityLevel       String?
  sleepHours          Float?
  stressLevel         Int?
  workType            String?
  supplements         String?
  xp                  Int      @default(0)
  level               Int      @default(1)
  onboardingCompleted Boolean  @default(false)
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt

  measurements    UserMeasurement[]
  weightEntries   WeightEntry[]
  bloodTests      BloodTest[]
  foodLogs        FoodLog[]
  aiEvaluations   AiEvaluation[]
  chatMessages    ChatMessage[]
  chatSummaries   ChatSummary[]
  achievements    UserAchievement[]
  streaks         Streak[]
}

model UserMeasurement {
  id        String   @id @default(uuid())
  userId    String
  chestCm   Float?
  waistCm   Float?
  hipCm     Float?
  armCm     Float?
  thighCm   Float?
  measuredAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
}

model WeightEntry {
  id              String   @id @default(uuid())
  userId          String
  weightKg        Float
  bodyFatPct      Float?
  muscleMassKg    Float?
  waterPct        Float?
  visceralFat     Float?
  basalMetabolism Float?
  boneMassKg      Float?
  source          String   @default("manual") // manual | smart_scale_photo
  imagePath       String?
  recordedAt      DateTime @default(now())
  user            User     @relation(fields: [userId], references: [id])
}

model BloodTest {
  id            String   @id @default(uuid())
  userId        String
  imagePath     String
  extractedData Json?
  testDate      DateTime?
  uploadedAt    DateTime @default(now())
  user          User     @relation(fields: [userId], references: [id])
}

model FoodLog {
  id            String   @id @default(uuid())
  userId        String
  imagePath     String?
  mealName      String?
  mealType      String   // breakfast | lunch | dinner | snack
  location      String?
  aiAnalysis    Json?    // { items: [], calories: N, protein: N, carbs: N, fat: N }
  userAdjusted  Boolean  @default(false)
  adjustedData  Json?
  loggedAt      DateTime @default(now())
  user          User     @relation(fields: [userId], references: [id])
}

model AiEvaluation {
  id        String   @id @default(uuid())
  userId    String
  type      String   // initial | weekly | monthly
  content   Json
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
}

model ChatMessage {
  id            String   @id @default(uuid())
  userId        String
  role          String   // user | assistant
  content       String
  imagePath     String?
  actionButtons Json?
  isProactive   Boolean  @default(false)
  isRead        Boolean  @default(true)
  createdAt     DateTime @default(now())
  user          User     @relation(fields: [userId], references: [id])
}

model ChatSummary {
  id                      String   @id @default(uuid())
  userId                  String
  summary                 String
  messagesSummarizedUntil DateTime
  createdAt               DateTime @default(now())
  updatedAt               DateTime @updatedAt
  user                    User     @relation(fields: [userId], references: [id])
}

model Achievement {
  id          String   @id @default(uuid())
  code        String   @unique
  name        String
  description String
  icon        String
  xpReward    Int
  category    String   // nutrition | training | consistency | milestone
  users       UserAchievement[]
}

model UserAchievement {
  userId        String
  achievementId String
  unlockedAt    DateTime @default(now())
  user          User        @relation(fields: [userId], references: [id])
  achievement   Achievement @relation(fields: [achievementId], references: [id])
  @@id([userId, achievementId])
}

model Streak {
  id              String   @id @default(uuid())
  userId          String
  type            String   // food_log | workout | weight | complete_day
  currentCount    Int      @default(0)
  bestCount       Int      @default(0)
  lastActiveDate  DateTime?
  freezeAvailable Boolean  @default(true)
  freezeUsedThisWeek Boolean @default(false)
  user            User     @relation(fields: [userId], references: [id])
}
```

#### 3. Auth System
- `POST /api/auth/register` — name, email, password → hash password, create user, return JWT
- `POST /api/auth/login` — email, password → verify, return JWT + refresh token
- `GET /api/auth/me` — JWT middleware → return user profile
- `POST /api/auth/refresh` — refresh token → new access token
- JWT middleware that protects all other routes
- Passwords hashed with bcrypt (12 rounds)
- Access token: 15min expiry. Refresh token: 7 days.

#### 4. Onboarding Flow (7 steps)
All endpoints require auth:
- `PUT /api/onboarding/basics` — sex, age, height, weight (creates first WeightEntry), hasSmartScale
- `POST /api/onboarding/smart-scale` — upload image → call GPT-4o vision to extract metrics → save to WeightEntry
- `PUT /api/onboarding/measurements` — chest, waist, hip, arm, thigh
- `PUT /api/onboarding/lifestyle` — activityLevel, sleepHours, stressLevel, workType
- `POST /api/onboarding/blood-test` — upload image/PDF → call GPT-4o vision to extract values → save
- `PUT /api/onboarding/supplements` — supplements text
- `POST /api/onboarding/evaluate` — gather ALL user data → call GPT-4o with Nuri personality → generate initial evaluation → save to AiEvaluation → mark onboardingCompleted = true

Frontend: wizard-style stepper, one step per screen, skip buttons on optional steps. Nuri appears with contextual messages at each step (use the messages from the specs). Progress bar at top.

#### 5. Dashboard (Home)
- Shows after onboarding is complete
- Nuri greeting based on time of day (morning/afternoon/evening/night)
- Today's caloric progress bar (consumed vs target from evaluation)
- Macros summary (protein, carbs, fat) — circular progress or bars
- List of today's food logs with thumbnail
- XP bar and current level
- Current streak with fire icon
- Quick action buttons: "Register meal", "Log weight"

#### 6. Food Registration with AI
- `POST /api/food/log` — multipart: image + optional mealName, mealType, location
- Backend: upload image → send to GPT-4o vision with prompt to identify food and estimate calories/macros → save to FoodLog → calculate XP (+10) → check/update streaks → return analysis + Nuri comment
- Frontend: camera/gallery picker → loading with Nuri Chef thinking → show results card (items identified, kcal, macros) → user can adjust → confirm
- The AI prompt for food analysis:
```
You are Nuri, an otter nutritionist. Analyze this food photo.
Return ONLY valid JSON:
{
  "items": ["item1", "item2"],
  "calories": number,
  "protein": number,
  "carbs": number,
  "fat": number,
  "comment": "A short comment in Spanish with Nuri's personality (max 100 chars)"
}
```

#### 7. Weight Registration
- `POST /api/weight/log` — manual: weightKg OR upload smart scale photo
- Manual: simple number input → save WeightEntry → +15 XP
- Frontend: clean input with Nuri Scientist, save button, show trend vs last entry

#### 8. Chat with Nuri
- `POST /api/chat/message` — { content: string, image?: file }
- Backend flow:
  1. Save user message to ChatMessage
  2. Build context: user profile + today's food logs + latest weight + streaks + last 20 chat messages
  3. Call GPT-4o with streaming (SSE) using Nuri's chat system prompt + context
  4. Stream response tokens to client via SSE
  5. Save complete assistant message to ChatMessage
- `GET /api/chat/history?page=1&limit=20` — paginated, newest first
- Frontend: messaging UI with bubbles, Nuri avatar on left, user on right. Text input + send button + image attach. Streaming display (tokens appear in real-time). Accessible via FAB (Nuri face) floating on all screens, opens as bottom sheet/modal overlay.
- The FAB is visible on every page except during onboarding.

#### 9. Basic Gamification
- XP system: track in user.xp, calculate level from XP thresholds
- Level names: Cría de Nutria (1-5), Nutria Nadadora (6-10), Nutria Cazadora (11-20), Nutria Alfa (21-35), Nutria Legendaria (36-50), Nutria Inmortal (50+)
- XP awards: food log +10, weight log +15, workout +25, complete day +50 bonus
- Streaks: update on each relevant action, check if lastActiveDate was yesterday
- Show XP bar, level badge, streak counter on dashboard
- Basic achievements: seed the initial achievements listed in specs, check conditions after each action

#### 10. PWA Setup
- manifest.json with app name "NutrIA", theme color #00B4D8, icons
- Basic service worker for offline shell caching
- Install prompt on first visit

### Phase 1 Endpoints Summary:
```
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/me
POST   /api/auth/refresh

PUT    /api/onboarding/basics
POST   /api/onboarding/smart-scale
PUT    /api/onboarding/measurements
PUT    /api/onboarding/lifestyle
POST   /api/onboarding/blood-test
PUT    /api/onboarding/supplements
POST   /api/onboarding/evaluate

POST   /api/food/log
PUT    /api/food/log/:id
GET    /api/food/daily/:date

POST   /api/weight/log
GET    /api/weight/history

POST   /api/chat/message          (SSE streaming response)
GET    /api/chat/history

GET    /api/gamification/status
GET    /api/gamification/achievements
```

---

## Commands

```bash
# Setup
cd client && npm install
cd server && npm install

# Database
cd server && npx prisma migrate dev
cd server && npx prisma db seed

# Development
cd client && npm run dev          # Vite dev server on :5173
cd server && npm run dev          # Express with nodemon on :3000

# Docker (postgres only for dev)
docker compose up -d postgres

# Build
cd client && npm run build        # Output to client/dist
```

---

## Important Rules

1. **All AI responses use Nuri's personality** — never generic. She's an otter, she makes fish jokes, she's motivating and direct.
2. **Mobile-first always** — design for 375px width first, then scale up.
3. **TypeScript strict mode** — no `any` types unless absolutely necessary.
4. **All API responses** follow format: `{ success: boolean, data?: any, error?: string }`
5. **Image uploads** use multer, stored in `./uploads/{userId}/`, max 10MB.
6. **OpenAI calls** always include error handling and fallback messages from Nuri.
7. **XP and streaks** are calculated server-side, never trust the client.
8. **JWT** is sent as Bearer token in Authorization header.
9. **Prisma** is the only way to interact with the database. No raw SQL.
10. **Environment variables** are never committed. Use .env.example as template.
11. **Spanish** is the default language for all UI text and Nuri's messages.
12. **Framer Motion** for all animations — page transitions, card appearances, progress bars, celebrations.
13. **Chat streaming** uses Server-Sent Events (SSE), not WebSockets.
14. **Rate limiting** on AI endpoints: max 60 requests/minute per user.

---

## OpenAI Integration Notes

- Model: `gpt-4o` for all calls (vision + text)
- For food photo analysis: send image as base64 in the user message content array
- For smart scale / blood test reading: same approach, different prompt
- For chat: use streaming with `stream: true`, pipe chunks as SSE to client
- For evaluation: non-streaming, wait for full response, parse JSON
- Always wrap in try/catch, return Nuri-flavored error: "¡Ups! Se me ha ido la onda. Inténtalo de nuevo, ¿vale?"
- Keep prompts in separate files under `server/src/prompts/` for maintainability

---

## Phases Overview (for context, only build Phase 1 now)

- **Phase 1 (current)**: Auth, Onboarding, Food log with AI, Weight log, Chat with Nuri, Dashboard, Basic XP/levels, PWA
- **Phase 2**: Nutrition module (menus, shopping list), Training module (plans, active session), Smart scale photo reading, Blood test reading, Progress charts, Advanced chat (images, action buttons, memory)
- **Phase 3**: Full gamification (streaks with freeze, 30+ achievements, weekly/monthly challenges, animations, sounds, proactive Nuri messages)
- **Phase 4**: Push notifications, n8n automation, performance optimization, final polish

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DigitalManagerIM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
