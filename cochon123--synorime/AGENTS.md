# Copilot instructions for Synonyme & Rime

## Vue d'ensemble du projet
- Mono-repo avec **backend NestJS** (`backend/`) et **frontend SvelteKit** (`frontend/`).
- Objectif: trouver des mots français qui **riment** et sont **synonymes**, à partir de **Lexique.org** (phonétique) et **WOLF** (synsets).
- Base PostgreSQL modélisée via Prisma (`backend/prisma/schema.prisma`) avec:
  - `Mot`: mots + phonétique + `rimeSignature` + `nbSyllabes` + `categorie` + `frequence`.
  - `Synset`: groupes de synonymes WOLF.
  - `MotSynset`: table de liaison (champ `mot` = lemme, pas de FK vers `Mot`).

## Back-end (NestJS + Prisma)
- Point d'entrée Nest: `backend/src/main.ts`, module racine `backend/src/app.module.ts` (importe `PrismaModule` et `MotsModule`).
- La logique métier principale est dans `backend/src/mots/`:
  - `mots.controller.ts`: expose l'API REST sous `/api/mots/*` (voir README pour les endpoints exacts).
  - `mots.service.ts`:
    - `trouverRimes(mot)`: récupère le `Mot` source puis tous les `Mot` partageant la même `rimeSignature`.
    - `trouverSynonymes(mot)`: passe par `MotSynset` pour trouver les lemmes puis requête `Mot` pour récupérer les infos complètes.
    - `trouverRimesSynonymes(mot)` et `trouverRimesAvecSynonymesDe(motRime, motSynonyme)`: intersection entre rimes et synonymes.
- N'utilise que Prisma Client (cf. `PrismaService`) pour la DB; respecter les types Prisma existants.
- Import de données lourdes via `backend/scripts/`:
  - `import-lexique.ts`: parse `data/Lexique383.tsv` et dérive `rimeSignature`.
  - `import-wolf.ts`: parse `data/wolf-1.0b4.xml` pour remplir `Synset` et `MotSynset`.
  - Scripts npm utiles (dans `backend/package.json`):
    - `npm run import:lexique`, `npm run import:wolf`, `npm run import:all`, `npm run seed`.

### Conventions backend
- Garder la séparation **controller (HTTP)** / **service (métier)** / **Prisma (accès DB)**.
- Lorsque vous ajoutez un nouvel endpoint:
  - Déclarez-le dans `mots.controller.ts` et implémentez la logique dans `mots.service.ts`.
  - Préférez retourner des objets "plats" de type `ResultatRecherche` (`mot`, `phonetique`, `categorie`, `nbSyllabes`) plutôt qu'exposer les entités Prisma complètes.
- Pour les synonymes, souvenez-vous que `MotSynset.mot` est simplement une **chaîne**; il faut toujours rejoindre avec `Mot` pour récupérer la phonétique/fréquence.

## Front-end (SvelteKit)
- Racine UI: `frontend/src/routes/+page.svelte` (page unique, pas de routing complexe).
- L'UI implémente trois modes:
  - **Simple**: champ unique `motSimple`, appelle `/api/mots/rime-synonyme?mot=...`.
  - **Avancé**: `motRime` + `motSynonyme`, appelle `/api/mots/rime-synonyme?motRime=...&motSynonyme=...`.
  - **Debug**: champ `motDebug`, appelle successivement `/mots/synonymes`, `/mots/rimes`, `/mots/rime-synonyme` et affiche **trois panneaux** (synonymes / rimes / intersection).
- L'URL de l'API est lue depuis `import.meta.env.VITE_API_URL` (voir `frontend/.env.example`). Ne pas hardcoder d'URL.
- Style très travaillé directement dans `+page.svelte` (pas de Tailwind). En cas d'ajout, respecter:
  - Utilisation des variables CSS déjà définies (`--color-bg`, `--color-accent`, etc.).
  - Animations existantes (`fadeInUp`, `fadeInScale`, etc.), mais éviter d'ajouter des animations trop agressives sans besoin explicite.

### Conventions frontend
- Rester dans la logique Svelte actuelle: état local avec `let` et `bind:value`, pas de store global.
- Factoriser la logique de tri via la fonction `trierResultats` existante au lieu de recréer des tris parallèles.
- Pour toute nouvelle API backend, ajouter les appels dans `+page.svelte` en gardant le même pattern:
  - `loading` / `erreur` / `rechercheFaite` + gestion des promesses avec `try/catch/finally`.

## Workflows de développement
- Démarrage local classique (hors Docker):
  - Backend:
    ```bash
    cd backend
    npm install
    npx prisma generate
    npx prisma migrate dev --name init
    npm run seed   # ou npm run import:all pour les vrais jeux de données
    npm run start  # ou nest start --watch si besoin
    ```
  - Frontend:
    ```bash
    cd frontend
    npm install
    npm run dev
    ```
- Endpoints à utiliser dans le code frontend ou pour le debug (préfixés par `/api/mots`): `stats`, `recherche`, `rimes`, `synonymes`, `rime-synonyme`.

## Ce que l'agent doit éviter
- Ne pas modifier la structure Prisma (`schema.prisma`) ou les scripts d'import sans raison forte (cela casse la compatibilité avec les données téléchargées).
- Ne pas introduire de nouveaux frameworks côté frontend (conserver Svelte + CSS natif).
- Ne pas dupliquer la logique rime/synonyme; toujours réutiliser `MotsService`.

## Fichiers clés à lire avant changements majeurs
- `README.md` (vue d'ensemble + commandes).
- Backend: `backend/src/mots/mots.service.ts`, `backend/src/mots/mots.controller.ts`, `backend/prisma/schema.prisma`.
- Frontend: `frontend/src/routes/+page.svelte`, `frontend/vite.config.ts`, `frontend/.env.example`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cochon123)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cochon123)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
