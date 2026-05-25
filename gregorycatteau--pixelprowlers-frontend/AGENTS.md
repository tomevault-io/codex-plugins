# GitHub Copilot Instructions

## Objectif

Ce fichier guide l'utilisation de GitHub Copilot dans le cadre du projet **PixelProwlers**, en définissant des conventions, des attentes techniques et des contextes métier. Il sert à améliorer la pertinence des suggestions et à garantir la cohérence du code.

---

## Contexte du projet

PixelProwlers est une application web composée :

- d’un **frontend** en **Vue 3**, **Nuxt 4**, **Tailwind CSS** et **Vite**
- d’un **backend** en **Django 5** avec **PostgreSQL** et une architecture modulaire par environnement (`development`, `test`, `production`)

La mission principale du site est de :

- Présenter une offre de services tech (landing page, storytelling fort)
- Offrir un blog de nouvelles
- Proposer un workflow d’onboarding utilisateur avec présentation de projet
- Favoriser la conversion via un CTA optimisé (formulaire de contact ou lancement de projet)

---

## Langue

Toutes les suggestions, commentaires et noms de fonctions doivent être rédigés en **français**. Préserver les noms explicites et conformes à la logique métier.

---

## Conventions de codage

### Backend (Django)

- Respecter la séparation des fichiers `settings` (base/dev/test/prod)
- Utiliser `python-decouple` pour les variables d’environnement
- Éviter les logiques métiers dans les vues ; préférer les services
- Les tests sont placés dans des modules `tests` par application

### Frontend (Nuxt)

- Composants organisés dans `components/`, `pages/`, `layouts/`
- Utiliser Tailwind CSS pour le style ; pas de CSS pur
- Préférer les `defineComponent`/`<script setup>` pour les composants Vue
- Les appels API doivent passer par `useFetch()` ou `$fetch` avec base URL

---

## Utilisation recommandée de Copilot

- Générer des squelettes de composants Vue, serializers Django, vues génériques
- Proposer des hooks ou composables Nuxt personnalisés
- Automatiser la génération de formulaires ou de modèles
- Ne pas surcharger les fichiers existants ; préférer l’ajout incrémental

---

## Collaboration & Bonnes pratiques

- Chaque module ou composant significatif doit être documenté
- Les migrations de BDD doivent être explicites et versionnées
- Toute logique métier ajoutée doit être testée (`pytest-django`, `vitest` à venir)
- Les commits doivent être clairs et en français (`feat:`, `fix:`, `test:`, etc.)

---

## Mise à jour

Ce fichier doit évoluer avec le projet. Toute nouvelle convention ou architecture ajoutée doit être reflétée ici pour maintenir l’alignement de Copilot et des contributeurs.

---

## Astuces pour améliorer les suggestions

- Commencer les commentaires par un verbe d’action clair
- Ajouter le contexte métier ou fonctionnel dans les docstrings
- Préciser les types attendus pour les paramètres et retours de fonction

---
> Source: [gregorycatteau/pixelprowlers-frontend](https://github.com/gregorycatteau/pixelprowlers-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-24 -->
