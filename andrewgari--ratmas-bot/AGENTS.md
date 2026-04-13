# Copilot Instructions

## Architecture Snapshot

- `src/index.ts` bootstraps the Discord client, instantiates the scoped services (`UserService`, `MessageService`, `ChannelService`, `RoleService`) and still handles the legacy prefix commands; mirror that composition when introducing new entry-points.
- `src/services/rat.service.ts` orchestrates Ratmas events, delegating validation to `rat.service.helpers.ts`, pairing/DM flows to `rat.service.operations.ts`, and persistence to `repositories/ratmas.repository.ts`.
- Prisma models live in `prisma/schema.prisma`; the shared client is exported via `src/persistence/prisma-client.ts`—never construct `PrismaClient` directly.
- Domain notes live under `docs/` (especially `ARCHITECTURE.md` and `PERSISTENCE.md`) and explain why services are narrow and data access is centralized.

## Ratmas Domain & Persistence

- `RatmasEvent` includes schedule fields (`eventStartDate`, `eventEndDate`, `purchaseDeadline`, `revealDate`, `timezone`); keep `src/types/ratmas.types.ts` in sync with schema and update enum guards (`RatmasEventStatus`, `ACTIVE_STATUSES`) when adding states.
- `RatmasRepository` is the only Prisma touch-point; reuse its mapping helpers and existing validation (`VALID_STATUS_VALUES`) when adding queries or writes.
- Pairings are replaced transactionally through `replacePairings`; never mutate pairing rows ad hoc or you’ll violate the uniqueness guarantees baked into the schema.

## Discord Integrations

- New slash behaviour sits in `src/commands/`; `ratmas-start.command.ts` registers `/ratmas start`, uses `ensureRatmasStartCommand` per guild, and gates access via `RATMAS_ROLE_ID` + admin permissions.
- Modal + channel orchestration lives in `ratmas-start.helpers.ts`, which leans on `ChannelService` for creation/permissions and posts the welcome message with an opt-out button (`RATMAS_OPT_OUT_BUTTON_ID`).
- Timezone parsing/formatting goes through `src/utils/date.utils.ts` (Luxon). Always pass the schedule through `parseRatmasSchedule` so inputs are validated and normalized to UTC before persistence.

## Services & Utilities

- Services are thin wrappers around the Discord client; prefer adding capabilities to the existing service or composing a new one rather than reviving the legacy `DiscordService`.
- `RoleService.removeRoleFromMember` is the approved way to drop the Ratmas role during opt-out interactions.
- Helper utilities (`shuffleArray`, `generateId`, schedule helpers) accept injectable dependencies—use that to keep tests deterministic.

## Workflows & Tooling

- Node 20+ with native ESM is required; always import local files with the `.js` suffix.
- Install deps with `npm install`; key scripts: `npm run build`, `npm run lint`, `npm run format:check`, `npm run prisma:generate`, `npm run prisma:migrate:dev`.
- Tests run with Jest ESM + ts-jest (`npm test` sets `NODE_OPTIONS=--experimental-vm-modules`). The command/utility tests mock `discord.js` behaviour directly—follow those patterns for new coverage.
- Regenerate Prisma client after touching the schema and ensure migrations land in `prisma/migrations/`.

## Testing Patterns

- `__tests__/commands/` covers the `/ratmas start` modal flow and helper utilities—use these as templates for mocking Discord interactions without hitting the real API.
- `__tests__/utils/date.utils.test.ts` demonstrates Luxon-based assertions; prefer asserting on formatted strings rather than raw Date objects when validating user-facing copy.
- `__tests__/index.test.ts` shows how to boot the app while intercepting `process.exit` and spoofing the Discord client.

## Repo Hygiene

- Follow `CLAUDE.md`: keep temporary planning in `/temp/`, avoid new helpers at repo root, and separate infra vs gameplay logic changes.
- Documentation and operational notes belong under `docs/`; keep PRs aligned with the existing structure and conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrewgari)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/andrewgari)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
