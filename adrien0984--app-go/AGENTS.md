# Instructions Copilot - GoAI Editor

## Contexte rapide
- PWA 100% client (React 18 + Vite + TypeScript), aucun backend.
- Point d’entrée UI + providers dans [src/main.tsx](src/main.tsx) (Redux + i18n + SW).
- Navigation Menu/Éditeur gérée par `App` dans [src/App.tsx](src/App.tsx).

## Architecture et flux clés
- État global via Redux Toolkit : `game`, `ui`, `settings`, `evaluations` (voir [src/store/index.ts](src/store/index.ts) et [src/store/slices](src/store/slices)).
- Plateau interactif Canvas dans `Board` [src/components/Board.tsx](src/components/Board.tsx).
  - Rendu en 7 layers via `canvasUtils` [src/utils/canvasUtils.ts](src/utils/canvasUtils.ts).
  - Pixel↔coord Go + sizing responsive via `boardUtils` [src/utils/boardUtils.ts](src/utils/boardUtils.ts).
  - Validation/alternance des coups via `GameService` [src/services/GameService.ts](src/services/GameService.ts).
- Persistance locale via `StorageService` [src/services/StorageService.ts](src/services/StorageService.ts) (IndexedDB, `ensureDB()` appelle `initDB()` si nécessaire).
- Analyse KataGo via `KataGoService` [src/services/KataGoService.ts](src/services/KataGoService.ts) (singleton, cache Map, simulation MVP).
  - Types complets dans [src/types/katago.ts](src/types/katago.ts) : `KataGoAnalysisResult` avec `policy: number[][]` (distribution NN 19×19).
  - UI dans `AnalysisPanel` [src/components/AnalysisPanel.tsx](src/components/AnalysisPanel.tsx) : winrate, score, top moves, loading, erreurs.

## Conventions spécifiques
- Aliases d’import : `@`, `@components`, `@services`, `@types`, `@utils`, `@store` (voir [tsconfig.json](tsconfig.json) et [vite.config.ts](vite.config.ts)).
- i18n : ressources FR/EN dans [src/locales](src/locales) et config dans [src/utils/i18nConfig.ts](src/utils/i18nConfig.ts) avec namespace par défaut `common`.
- Le `GameEditor` affiche un placeholder de plateau (voir [src/components/GameEditor.tsx](src/components/GameEditor.tsx)) : intégrer `Board` doit préserver les sélecteurs E2E.

## PWA / Service Worker
- VitePWA configuré dans [vite.config.ts](vite.config.ts) (cache CDN jsDelivr pour modèles).
- Service worker custom dans [public/sw.ts](public/sw.ts) et registration dans [src/main.tsx](src/main.tsx) : toute modif de cache doit rester cohérente entre les deux.

## Tests et workflows
- Scripts npm (voir [package.json](package.json)) : `dev`, `build`, `preview`, `test` (Vitest), `test:e2e` (Playwright), `lint`, `type-check`.
- E2E Playwright cible `canvas.board-canvas`, aria-labels et éléments `.status-text` / `.current-player` (voir [tests/e2e/board.spec.ts](tests/e2e/board.spec.ts)).
- Les tests E2E utilisent le port Vite 5173 (voir [vite.config.ts](vite.config.ts)) et la doc d’exécution dans [tests/e2e/README.md](tests/e2e/README.md).
## Documentation
- Documentation organisée dans [docs/](docs/) par catégories : specifications, architecture, user-stories, qa-reports, validation, guides.
- Guide de navigation : [DOCUMENTATION.md](DOCUMENTATION.md) pour trouver rapidement les documents.
- Spécifications : [docs/specifications/](docs/specifications/) - SF et ST complètes.
- Architecture : [docs/architecture/ARCHITECTURE.md](docs/architecture/ARCHITECTURE.md) - Vue d'ensemble technique.
- Index complet : [docs/INDEX.md](docs/INDEX.md) - Hub de navigation de toute la documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adrien0984) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
