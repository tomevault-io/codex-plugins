# Instructions Copilot pour Portfolio 3D

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

Ce projet est un portfolio 3D interactif utilisant:

## Technologies principales
- **React 18** avec TypeScript pour l'interface utilisateur
- **Three.js** avec React Three Fiber pour la 3D
- **Tailwind CSS** pour le styling
- **Vite** comme bundler

## Architecture du projet
- `src/components/` - Composants React/Three.js
- `src/types/` - Définitions TypeScript
- `src/data/` - Données des projets
- L'île 3D est au centre avec des bâtiments représentant les projets
- Navigation circulaire infinie autour de l'île
- Tooltips au survol et popups détaillés au clic

## Conventions de code
- Utiliser des composants fonctionnels avec hooks
- Préférer les animations Three.js natives pour les performances
- Maintenir la cohérence visuelle avec le thème "île paradisiaque"
- Optimiser les performances 3D (géométries réutilisables, matériaux partagés)

## Spécificités 3D
- Système de coordonnées: Y vers le haut
- Éclairage atmosphérique avec plusieurs sources
- Ombres activées pour le réalisme
- Camera avec contrôles OrbitControls limités

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Clem2tem) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
