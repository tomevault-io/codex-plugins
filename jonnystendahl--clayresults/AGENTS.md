# ClayResults AI Guidance

This repository is a Laravel 12 application for clay shooting clubs, memberships, and training results.

Core stack:
- PHP 8.2+
- Laravel 12
- MySQL for app runtime
- Blade templates for UI
- Bootstrap 5 for styling and components
- Vite for frontend assets

Default development rules:
- Follow Laravel conventions before inventing custom structure.
- Keep controllers thin and use Form Requests for non-trivial validation.
- Prefer Eloquent relationships, route model binding, and named routes.
- Use Blade and Bootstrap 5. Do not introduce React, Vue, Livewire, or Alpine unless explicitly requested.
- For any task that changes files, start from `main` and create a new branch before editing. Do not work directly on `main`.
- Use branch names in the format `<type>/<short-kebab-summary>`, where `<type>` usually matches the conventional commit scope, for example `feat/club-renewal-reminders`, `fix/public-club-links`, `docs/release-workflow`, or `chore/dependency-updates`.
- Use conventional commits when creating git commits, for example `feat: add club home page` or `docs: update setup notes`.
- Preserve the existing visual direction and avoid replacing Bootstrap with another UI approach.
- Protect all user-owned data with authorization or ownership checks.
- Prefer feature tests for user-facing behavior, especially auth and multi-user boundaries.
- Fix only the requested scope unless a nearby issue directly blocks the task.

Application-specific rules:
- Training results belong to individual authenticated users.
- Users must only be able to view and modify their own results.
- Users can belong to one or more clubs.
- A user with multiple clubs should have one valid `main_club_id` that points to a club they still belong to.
- The home page is club-first: guests see general club/application information, and authenticated users land on their main club page.
- Club switching must only allow a user to switch to clubs they are actually a member of.
- Club memberships track club-specific role, paid status, joined date, last paid date, and membership end date.
- Use clear Swedish/English clay shooting discipline names already established in the app unless asked to change them.
- Keep forms server-rendered and simple to maintain.
- If database changes are needed, create migrations instead of editing schema manually.

When adding a feature, usually update:
- routes
- controller
- request validation
- model or relationship
- Blade view
- feature tests

For club-related features, usually also review:
- home page behavior
- navigation and club switcher behavior
- main club synchronization rules
- admin club membership flows

When reviewing or editing code, prioritize:
- authorization correctness
- validation correctness
- Laravel convention alignment
- regression risk in routes and views
- main club consistency after membership changes
- test coverage for important flows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonnystendahl)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jonnystendahl)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
