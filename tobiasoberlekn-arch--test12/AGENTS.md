# Copilot Instructions

Diese Regeln gelten für alle Code-Änderungen in diesem Repository.

Zusätzliche verschärfte Teamregeln stehen in:

- `.github/copilot-instructions.strict.md`

## Zielbild

- Das Repository ist ein modulares Monorepo mit:
  - `apps/web` (Angular Frontend)
  - `apps/api` (Node.js/Fastify Backend)
  - `packages/shared-contracts` (gemeinsame API-Typen)
  - `packages/shared-utils` (framework-unabhängige Utilities)
- Änderungen sollen wiederverwendbar, klar getrennt und minimal-invasiv sein.

## Architektur- und Modulregeln

- Neue Fachlogik immer in das passende Modul legen, nicht in zentrale Sammeldateien.
- Backend:
  - Routen in `*.module.ts`
  - Businesslogik in `*.service.ts`
  - Keine Businesslogik direkt im Route-Handler.
- Frontend:
  - Feature-Code unter `apps/web/src/app/features/<feature>`
  - Infrastruktur-/Service-Code unter `apps/web/src/app/core`
  - Wiederverwendbare UI-Bausteine unter `apps/web/src/app/shared`
- Shared:
  - API-Verträge (DTOs/Interfaces) nur in `packages/shared-contracts`
  - Generische Helpers nur in `packages/shared-utils`
  - Keine Framework- oder Infrastrukturabhängigkeiten in Shared-Paketen

## Import- und Abhängigkeitsregeln

- Verwende vorhandene Path-Aliases aus `tsconfig.base.json`:
  - `@shared/contracts`
  - `@shared/utils`
- Keine „Abkürzungs-Imports“ über Modulgrenzen hinweg.
- Keine unnötigen neuen Dependencies einführen.

## Coding-Style

- TypeScript strikt halten (`strict` darf nicht umgangen werden).
- Kleine, fokussierte Funktionen und klar benannte Variablen/Methoden.
- Öffentliche APIs bewusst klein halten.
- Bestehende Datei-/Namenskonventionen beibehalten.
- Kommentare nur dort, wo sie Mehrwert bieten (Why statt nur What).

## Änderungen an Konfigurationen

Wenn `package.json`, `angular.json`, TSConfigs oder CI-Dateien geändert werden:

- Dokumentation mitpflegen in:
  - `docs/configuration-reference.md`
  - passender Bereichs-README (`apps/*/README.md`, `packages/*/README.md`)
  - ggf. `README.md` im Root

## Qualitätssicherung

Vor Abschluss von Änderungen nach Möglichkeit ausführen:

- `npm run lint`
- `npm run build`
- optional: `npm run test --workspaces --if-present`

## Pull-Request-Erwartung

- Änderungen klein und thematisch fokussiert halten.
- Keine unrelated refactors in demselben PR.
- PR-Titel im Stil von `feat: ...`, `fix: ...`, `docs: ...`, `chore: ...`.
- PR-/Issue-Templates unter `.github/` beachten.

## Bevorzugtes Verhalten für Copilot

- Erst vorhandene Struktur nutzen, dann erweitern.
- Bei Mehrdeutigkeit die einfachste modulare Lösung bevorzugen.
- Keine neuen Patterns einführen, wenn bereits ein passendes Pattern existiert.
- Bei neuen Modulen direkt eine klare Trennung von Transport, Logik und Verträgen einhalten.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tobiasoberlekn-arch)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tobiasoberlekn-arch)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
