# Copilot / AI Agent Instructions for ExamCert ⚡️

Purpose
- Be concise and pragmatic: focus on changes that can be implemented and tested quickly.
- Preserve existing app behavior; point out inconsistencies and propose minimal, testable fixes.

Big picture
- Two components: `cert_exam_app/` (Flutter frontend) and `cert-backend/` (Node/Express + PostgreSQL).
- Frontend consumes backend JSON via `ApiService.baseUrl = http://localhost:3000`.
- Local offline storage: `lib/db/db_helper.dart` uses `sqflite` (mobile); `assets/questions.json` is a simple fallback.

Key files & responsibilities
- Frontend
  - `lib/services/api_service.dart` — HTTP client; modify here when changing API contract.
  - `lib/models/*.dart` — canonical Dart models; note mapping between DB keys and Dart fields (see below).
  - `lib/db/db_helper.dart` — local SQLite schema & access (used offline).
  - `lib/screens/*.dart` — UI and flow (e.g., `certification_list_screen.dart`, `quiz_screen.dart`).
- Backend
  - `cert-backend/index.js` — Express routes: `GET /questions`, `POST /questions`, `GET /certifications`.
  - `cert-backend/.env` expected for DB creds (see `index.js` `Pool` config).

Important patterns & gotchas
- Naming conventions: Backend DB uses snake_case (e.g., `option_a`, `correct_index`). Dart models use camelCase (`optionA`, `correctIndex`). `Question.fromMap` maps snake_case → camelCase — keep this when adding fields.
- Certification filtering is not implemented end-to-end:
  - `CertificationListScreen` passes `certificationId` to `QuizScreen`.
  - `QuizScreen` calls `ApiService.fetchQuestions(certificationId: ...)` but current `ApiService.fetchQuestions()` does NOT accept that argument — this is a known mismatch to address.
  - Two reasonable fixes: (1) add optional `certificationId` query param in `ApiService.fetchQuestions({int? certificationId})` and filter on backend (`/questions?certificationId=...`), or (2) add a dedicated endpoint like `/certifications/:id/questions` and update client.
- Local DB schema in `DBHelper._initDB()` stores `certificationId` (camelCase) while backend likely uses snake_case — when syncing local ↔ remote, normalize names consistently.

Developer workflows & commands
- Frontend (Flutter)
  - Install: `cd cert_exam_app && flutter pub get`
  - Run: `flutter run -d chrome` (web), `-d windows`, `-d android` (emulator/device)
  - Analyze: `flutter analyze`
  - Tests: `flutter test` (there's only a default widget test now)
- Backend (Node)
  - Install: `cd cert-backend && npm install`
  - Configure: create `.env` with `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_NAME` (same names used in `index.js`)
  - Run: `node index.js` or `npx nodemon index.js` (dev)
  - Health: `curl http://localhost:3000/` → should reply with running message
- Database: README mentions Dockerized PostgreSQL, but no Docker compose files are in the repo. Use a local Postgres instance or add a `docker-compose.yml` if you need reproducible DB setup.

Quick examples (where to change code)
- Add filter param to client:
  - Change `lib/services/api_service.dart` → `static Future<List<Map<String,dynamic>>> fetchQuestions({int? certificationId})` and append `?certificationId=$id` when present.
- Add backend filtering:
  - Update `cert-backend/index.js` GET `/questions` handler to check `req.query.certificationId` and `WHERE certification_id = $1` in the SQL query.

Testing tips
- Use Postman / curl to validate backend endpoints before changing the client.
- When changing model fields, update `Question.fromMap` and API decoding accordingly.
- Use Flutter DevTools and the app logs — `print()` is used in various places (see `QuizScreen._loadQuestions` catch block).

When to ask for help
- If API contract changes are large (new fields / renames), request confirmation before modifying both client and server.
- If adding DB migrations, prefer adding a small migration script and documenting schema changes in `cert-backend/README`.

If anything above is unclear or you want the instructions to cover a different level of detail (e.g., example patches for the certification filtering fix), tell me which sections to expand or edit and I will iterate. ✅

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uchennaezeilo)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/uchennaezeilo)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
