# JobFinder AI Onboarding

## Architecture Snapshot
- Spring Boot 3.5 + Java 25 monolith with layered packages under `src/main/java/com/example/JobFinder` (controller → service → repository → model) and Thymeleaf views in `src/main/resources/templates`.
- REST-ish controllers (`controller/*.java`) mostly return server-rendered HTML; they rely on services to supply template-friendly `Map` payloads instead of exposing JPA entities directly (see `JobService#jobToMap`).
- Data persistence uses MySQL with JPA repositories plus heavy JPQL queries (`repository/JobRepository.java`, `ApplicationRepository.java`) to fetch employers, categories, and candidates in one go to avoid N+1 issues.
- Authentication & role routing live in `config/SecurityConfig.java` + `RoleBasedAuthenticationSuccessHandler`; `/jobs/**`, assets, and `/auth/**` stay public, while `/admin/**`, `/employer/**`, `/candidate/**` require ROLE_* gates.
- Domain entities in `model/` mirror the SQL schema files (`schema.sql`, `database_jobs_schema.sql`) and rely on bidirectional links (e.g., `Job ↔ Employer`, `User ↔ Candidate/Employer`).

## Runbook & Tooling
- Local DB: import `schema.sql` then `database_jobs_schema.sql` (jobs module) into a MySQL/XAMPP `jobfinder` database; seed roles/users via `data.sql` when needed.
- App config: adjust credentials in `src/main/resources/application.properties`; `spring.sql.init.mode=always` ensures `data.sql` replays when tables miss required rows.
- Build/test commands (Windows-friendly): `.\mvnw.cmd clean package -DskipTests`, `.\mvnw.cmd spring-boot:run`, and `.\mvnw.cmd test` for unit suites.
- Static assets live under `src/main/resources/static/assets` (Bootstrap bundle, vendor libs) and are directly referenced by Thymeleaf templates; keep naming consistent when adding new bundles.
- Logging is already verbose for security/web (`logging.level.org.springframework.security=DEBUG`). Trim per package before merging new noisy logs.

## Feature Conventions
- Jobs module: follow `JOBS_MODULE_README.md` for accepted endpoints (`/jobs`, `/jobs/hot`, `/jobs/{id}`, saved toggle). Controller logic should delegate to `JobService` filters so pagination, saved-job badges, and employment stats stay in sync.
- Applications: `ApplicationController` expects candidate context from `Authentication`; reuse `getCandidateFromAuth()` helper and persist cover letters/attachments via `ApplicationRepository` methods.
- User onboarding: `AuthController` + `UserService` only allow self-registration for roles defined in `UserService.SELF_REGISTERABLE_ROLES`; add role IDs via `findSelfRegisterRoles()` instead of hardcoding.
- Admin CRUD: `docs/USER_MANAGEMENT_GUIDE.md` documents `/admin/users/**` flows and avatar handling; replicate the flash-message + modal confirmation pattern spelled out there when extending admin tools.
- File uploads: avatars land in `src/main/resources/static/uploads/avatars`, CVs in `uploads/cv`; enforce size/type checks similar to `ApplicationController.handleCvUpload()` and clean up old files when replacing assets.
- Templates: frontend pages sit under `templates/frontend/**` (jobs, auth, applications) using Bootstrap 5; admin pages reuse NiceAdmin layouts in `templates/layout` and `templates/admin`. Update both HTML and CSS (`static/assets/style.css`) when changing UI contracts.

## Working Safely
- Whenever you introduce a new public route, mirror its access rule in `SecurityConfig` to prevent accidental lockouts.
- Prefer extending existing JPQL queries rather than layering in-memory filters—pagination relies on DB-level filtering.
- When exposing data to Thymeleaf, stick to the map structure returned by services (ids, `companyName`, `postedAgo`, etc.) so templates such as `templates/frontend/jobs/index.html` continue to render without refactors.
- Keep seed scripts (`data.sql`, `database_jobs_schema.sql`) and entities synchronized; migrations outside these files will desync local onboarding scripts.
- Reuse existing flash-message keys (`flashType`, `flashMessage`, `application_flash`) so UX feedback stays consistent across controllers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khacduc204)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/khacduc204)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
