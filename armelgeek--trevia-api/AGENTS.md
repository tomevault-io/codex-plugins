# GitHub Copilot – Instructions pour le projet API Backend

## Contexte du projet
- Projet : API backend modulaire, architecture RESTful, séparation claire des domaines (utilisateurs, produits, commandes, etc.).
- Objectif : Générer rapidement endpoints, tâches, et documentation structurée à partir de descriptions fonctionnelles ou de documentation existante.

## Architecture & Organisation (ARCI)
- **Acteur** : Développeurs backend
- **Responsable** : Lead technique / Product Owner
- **Consulté** : Équipe frontend, QA, Product Owner
- **Informé** : Parties prenantes, documentation projet

## Conventions & attentes
- Respecter la structure des prompts d’automatisation présents dans `docs/prompt-automation.md` et `docs/prompt-backend-automation.md`.
- Générer pour chaque endpoint :
  - Tâche TODO claire
  - Use case (titre + description)
  - Scénario nominal et exceptions
  - Critères d’acceptation
  - Paramètres (query, path, body)
  - Exemple de payload si pertinent
- Séparer les tâches par domaine (ex : Utilisateur, Produit, Commande…)
- Utiliser le format Markdown pour les todo-lists et la documentation générée.
- Prioriser les tâches critiques (auth, admin, sécurité).

## Bonnes pratiques
- Générer du code lisible, modulaire, typé (TypeScript recommandé)
- Respecter les conventions de nommage et la structure du projet existant
- Documenter chaque endpoint et chaque tâche générée
- Proposer des scénarios d’exception réalistes et des critères d’acceptation précis

## Utilisation des prompts
- Pour générer une todo-list backend complète à partir d’une description, utiliser le prompt de `docs/prompt-backend-automation.md`.
- Pour générer une todo-list frontend ou mixte, utiliser le prompt de `docs/prompt-automation.md`.

## Instructions techniques

### Architecture
- Utiliser une architecture modulaire et hexagonale (domain-driven design recommandé).
- Séparer clairement : domain (modèles, logique métier), application (services, use-cases), infrastructure (controllers, db, middlewares, config).
- Respecter la structure de dossiers existante : `src/domain`, `src/application`, `src/infrastructure`.
- Utiliser des interfaces pour les repositories et services, injecter les dépendances.

### Technologies & Outils
- Langage : TypeScript (strict mode recommandé)
- Framework HTTP : Express.js ou Fastify
- ORM/Query builder : Drizzle, Prisma ou TypeORM (Drizzle déjà présent)
- Validation : Zod ou Joi pour les schémas de payload
- Tests : Vitest ou Jest pour les tests unitaires et d’intégration
- Lint : ESLint, Prettier (déjà configurés)

### Conventions de code
- Typage strict partout (types/interfaces pour les modèles, DTO, etc.)
- Nommage explicite des fonctions, variables et endpoints
- Utiliser des middlewares pour l’authentification, la gestion des erreurs, la validation
- Documenter chaque endpoint (JSDoc ou commentaires)

### Gestion des erreurs & sécurité
- Centraliser la gestion des erreurs (middleware dédié)
- Toujours valider et nettoyer les entrées utilisateur
- Protéger les routes sensibles (authentification, autorisation RBAC)
- Ne jamais exposer d’informations sensibles dans les messages d’erreur

### Tests & qualité
- Écrire des tests unitaires pour la logique métier (domain/application)
- Écrire des tests d’intégration pour les endpoints principaux
- Utiliser des mocks pour les dépendances externes
- Couverture de code minimale recommandée : 80 %

### Documentation & automatisation
- Générer automatiquement la documentation des endpoints (OpenAPI/Swagger si possible)
- Utiliser les prompts d’automatisation pour générer les todo-lists et la documentation technique

### Étapes techniques pour réaliser un use case backend

1. **Analyse et rédaction**
   - Comprendre le besoin métier, rédiger le use case (titre, description, scénario nominal, exceptions, critères d’acceptation).
2. **Définition du modèle**
   - Créer ou adapter les modèles/domaines nécessaires dans `src/domain/models`.
   - Définir les types/interfaces associés.
3. **Repository & accès aux données**
   - Définir ou compléter l’interface repository dans `src/domain/repositories`.
   - Implémenter l’accès aux données dans l’infrastructure si besoin.
4. **Service / Application logic**
   - Implémenter la logique métier dans un service ou use-case dans `src/application/services` ou `src/application/use-cases`.
   - Injecter les dépendances (repository, services externes).
5. **Controller / Endpoint**
   - Créer ou compléter le controller dans `src/infrastructure/controllers`.
   - Définir la route, valider les entrées (Zod/Joi), gérer les erreurs.
6. **Sécurité & validation**
   - Ajouter les middlewares nécessaires (auth, RBAC, validation).
7. **Tests**
   - Écrire des tests unitaires pour la logique métier.
   - Écrire des tests d’intégration pour l’endpoint.
8. **Documentation**
   - Documenter le use case, l’endpoint, les paramètres et les scénarios dans le code et/ou la doc Markdown.

---

**Exemple d’instruction à Copilot** :
« Lis la description fonctionnelle du projet et génère automatiquement une todo-list backend exhaustive (format Markdown) pour l’implémentation de tous les endpoints nécessaires, avec use case, scénario nominal, exceptions, critères d’acceptation, paramètres, body et exemple de payload pour chaque tâche, en séparant les domaines. »

# Copilot Instructions – Ajout d’une nouvelle fonctionnalité (exemple : gestion des conducteurs)

## 1. Architecture du projet

- **Domain** : Modèles, types et interfaces métier (`src/domain/models`, `src/domain/types`, `src/domain/repositories`)
- **Application** : Use cases (logique métier orchestrée), services (`src/application/use-cases`, `src/application/services`)
- **Infrastructure** :
  - **Controllers** : API REST (Hono/OpenAPI) (`src/infrastructure/controllers`)
  - **Database** : Schémas, accès DB (`src/infrastructure/database/schema`, `src/infrastructure/database/db`)
  - **Config, Middlewares, Schedulers**
- **Entrée principale** : `src/app.ts`, `src/server.ts`

## 2. Étapes pour ajouter une nouvelle fonctionnalité (ex : Drivers)

1. **Définir le modèle métier**
   - Créer un fichier `driver.model.ts` dans `src/domain/models` avec le type Driver.
   - Définir les types associés dans `src/domain/types` si besoin.
2. **Définir l’interface repository**
   - Créer `driver.repository.interface.ts` dans `src/domain/repositories` pour les méthodes CRUD.
3. **Créer le schéma DB**
   - Ajouter la table `drivers` dans `src/infrastructure/database/schema/schema.ts`.
4. **Implémenter le repository**
   - Créer `driver.repository.ts` dans `src/infrastructure/repositories` pour l’accès DB.
5. **Créer les use cases**
   - Créer un dossier `src/application/use-cases/driver/`.
   - Ajouter les use cases : `create-driver.use-case.ts`, `get-driver.use-case.ts`, `list-drivers.use-case.ts`, etc.
6. **Créer le controller**
   - Créer `driver.controller.ts` dans `src/infrastructure/controllers`.
   - Définir les routes REST (GET, POST, PUT, DELETE) avec validation Zod/OpenAPI.
7. **Brancher le controller**
   - Ajouter le controller dans l’index des routes (`src/infrastructure/controllers/index.ts` ou dans l’app principale).
8. **Tests et documentation**
   - Ajouter des tests unitaires pour les use cases et le repository.
   - Documenter les endpoints dans OpenAPI/Swagger si besoin.

## 3. Convention de nommage
- Dossiers et fichiers en kebab-case.
- Types et classes en PascalCase.
- Méthodes et variables en camelCase.

## 4. Exemple de structure pour la fonctionnalité Driver

```
src/
  domain/
    models/driver.model.ts
    repositories/driver.repository.interface.ts
    types/driver.type.ts
  application/
    use-cases/driver/
      create-driver.use-case.ts
      get-driver.use-case.ts
      list-drivers.use-case.ts
      ...
  infrastructure/
    controllers/driver.controller.ts
    repositories/driver.repository.ts
    database/schema/schema.ts (ajout table drivers)
```

---

**Résumé :**
Pour toute nouvelle fonctionnalité, respecter la séparation Domain / Application / Infrastructure, créer les modèles, repository, use cases, controller, brancher les routes, et tester/documenter.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/armelgeek)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/armelgeek)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
