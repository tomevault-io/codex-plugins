# ShadBuzz - Application de Chat IA Local

## Description du Projet
ShadBuzz est une application Python utilisant ttkbootstrap (thème cyborg) et PyTorch pour créer une expérience de chat IA locale similaire à Polybuzz.

## Fonctionnalités Principales
- Création et personnalisation de personnages IA (nom, personnalité, avatar)
- Chat interactif avec historique des conversations
- Réponses générées par LLM selon la personnalité du personnage
- Sauvegarde des données en JSON
- Interface futuriste avec design néon/pastel
- Mode jeu de rôle avec scénarios personnalisés
- Génération d'avatars via modèle léger (optionnel)

## Technologies Utilisées
- Python 3.8+
- ttkbootstrap (thème cyborg)
- PyTorch pour le LLM local
- JSON pour la sauvegarde des données
- PIL/Pillow pour la gestion des images

## Structure du Projet
```
shadbuzz/
├── main.py                 # Point d'entrée principal
├── requirements.txt        # Dépendances Python
├── README.md              # Documentation
├── config/
│   ├── settings.py        # Configuration de l'application
│   └── themes.py          # Thèmes et styles UI
├── ui/
│   ├── main_window.py     # Fenêtre principale
│   ├── chat_interface.py  # Interface de chat
│   ├── character_panel.py # Panneau de gestion des personnages
│   └── components/        # Composants UI réutilisables
├── models/
│   ├── character.py       # Modèle de personnage IA
│   ├── conversation.py    # Modèle de conversation
│   └── llm_handler.py     # Gestionnaire du LLM local
├── utils/
│   ├── data_manager.py    # Gestion des données JSON
│   ├── avatar_generator.py# Génération d'avatars
│   └── helpers.py         # Fonctions utilitaires
└── data/
    ├── characters/        # Données des personnages
    ├── conversations/     # Historique des conversations
    └── assets/           # Images et ressources
```

## Instructions de Développement
- Code modulaire et bien commenté
- Optimisation pour exécution locale fluide
- Interface réactive avec animations subtiles
- Gestion d'erreurs robuste
- Tests unitaires pour les composants critiques

## Style et Design
- Thème cyborg de ttkbootstrap
- Palette de couleurs néon/pastel
- Animations CSS subtiles
- Design futuriste et moderne
- Interface utilisateur intuitive

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShadCanard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
