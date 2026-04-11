# Copilot Instructions for Sternenhaus (smartyparty)

## Project Overview
- **Sternenhaus** is a turn-based online multiplayer economic simulation set in a sci-fi universe, inspired by Dune and Fugger. The game leverages AI for emergent storytelling, dynamic NPCs, and a living world.
- **Core gameplay** revolves around reputation and influence, with players rising from independent traders to major houses through strategic decisions and interactions.

## Architecture & Tech Stack
- **Frontend:** React 18 + Next.js 14 (App Router)
- **Backend:** Firebase Cloud Functions (Gen 2)
- **Database:** Cloud Firestore
- **Auth:** Firebase Authentication
- **Hosting:** Firebase Hosting
- **AI/LLM:** Google Gemini 1.5 Flash API
- **Realtime:** Firestore listeners
- See `docs/architecture/firebase-techstack.md` for diagrams, schema, and deployment details.

## Developer Workflows
- **Sprint planning and progress:** Tracked in `docs/sprints/` and subfolders. Each sprint has a plan and story details.
- **Initial setup:**
  - Next.js app with TypeScript and Tailwind
  - Firebase SDK integration
  - Deploy via Firebase Hosting
- **Database:**
  - Collections: `users`, `games` (see `docs/architecture/firebase-techstack.md`)
  - Security rules and TypeScript interfaces are required for all Firestore access.
- **Services:**
  - Use service modules (e.g., `userService`, `gameService`) for Firestore operations.
- **Testing:**
  - No explicit test framework documented; follow story details in `docs/sprints/sprint-1/story-details.md` for code examples and acceptance criteria.
- **CI/CD:**
  - Deployment is managed via Firebase Hosting. See sprint plans for CI/CD notes.

## Patterns & Conventions
- **Emergent AI:** NPCs, events, and storylines are generated and managed by AI (Gemini API). All character and event logic should be designed for extensibility and dynamic response.
- **Game logic:** Reputation and influence are tracked as primary metrics. All major actions should update these values and trigger appropriate AI/NPC reactions.
- **Information visibility:** Actions are public, semi-public, or secret. Ensure game events are tagged for visibility and trigger correct rumor/scandal logic.
- **Spionage/Agents:** Spies have dynamic personalities and loyalties. Use AI-generated dialogue and relationship tracking for all agent interactions.
- **Modular services:** All Firestore and business logic should be encapsulated in service modules for maintainability.

## Key Files & Directories
- `README.md`: Vision, gameplay, and core mechanics
- `docs/architecture/`: Tech stack, system diagrams, database schema
- `docs/sprints/`: Sprint plans, story details, progress tracking
- `docs/architecture/firebase-techstack.md`: Main architecture reference

## External Integrations
- **Firebase:** All backend, database, and auth operations
- **Google Gemini API:** For AI-driven game logic and NPCs

## Example Patterns
- When adding a new game action, update both reputation and influence, and ensure the event is processed by the AI for NPC/world reaction.
- All Firestore access should use TypeScript interfaces and be routed through service modules.
- For new sprints, add planning and story details in `docs/sprints/` and link them in the sprint README.

---

For questions or missing conventions, review the architecture docs and sprint plans, or ask for clarification. Please suggest improvements if any section is unclear or incomplete.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/denfu80)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/denfu80)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
