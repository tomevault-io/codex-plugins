# 📜 Guide de Développement - Association Mama Esther

Ce fichier contient les directives, les choix architecturaux et les informations contextuelles essentiels pour maintenir la cohérence du projet lors des interactions avec Gemini CLI.

## 1. Directives de Développement
- **Internationalisation (i18n) :** Le site est bilingue (Français/Anglais). Toute nouvelle chaîne de caractères doit être ajoutée dans `src/locales/fr/translation.json` et `src/locales/en/translation.json`, puis appelée via le hook `useTranslation`.
- **Structure Sémantique :** Veiller à ce que la balise `<main>` soit unique par page (généralement dans le composant de page racine). Les composants réutilisables doivent utiliser des balises `<section>`, `<article>` ou `<div>`.
- **Séparation des Styles :** Privilégier les fichiers CSS dédiés dans `src/styles/` ou à côté du composant, plutôt que les styles en ligne (inline styles).
- **Composants de Structure :** Utiliser systématiquement le composant `<Divider />` pour séparer les grandes sections visuelles sur la page d'accueil et les pages de contenu.

## 2. Architecture et Choix Techniques
- **Frontend :** React (Vite) avec une organisation par dossiers :
    - `src/components/` : Composants UI réutilisables.
    - `src/pages/` : Composants de page complets.
    - `src/data/` : Fichiers de données statiques (ex: `newsletters.js`).
- **Backend :** API Node.js/Express située dans le dossier `/backend-newsletter`.
- **Gestion des Newsletters :** Les données sont centralisées dans `src/data/newsletters.js`. Le composant `NewsletterCarousel` est le moyen privilégié pour afficher les dernières publications.

## 3. Contexte du Projet et Environnement
- **API Backend :** L'application communique avec un backend Express tournant par défaut sur le port **5000** (utilisé notamment par `DonationCounter`).
- **Commandes Utiles :**
    - Frontend : `npm run dev` (Vite)
    - Backend : Situé dans `/backend-newsletter`, utilise généralement `npm start` ou `nodemon`.
- **Assets :** Les images et PDF sont stockés dans `/public/assets/`. Les chemins dans le code doivent commencer par `/assets/`.

---
*Note : Ce fichier doit être mis à jour lors de changements architecturaux majeurs.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PapaG57)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/PapaG57)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
