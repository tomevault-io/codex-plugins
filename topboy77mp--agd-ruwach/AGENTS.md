<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# RUWACH-TECH Sarl - Instructions de Développement

## Vue d'ensemble du projet
Ce projet est un site web vitrine professionnel pour RUWACH-TECH Sarl, une entreprise togolaise spécialisée dans les services technologiques.

## Architecture technique
- **Frontend**: HTML5, Tailwind CSS, JavaScript vanilla
- **Backend**: Python Flask
- **Design**: Moderne, inspiré Tesla/Vercel avec effets glass
- **Couleurs**: Bleu nuit africain (hsl(220, 100%, 20%)), bleu électrique (hsl(200, 100%, 50%))
- **Typographie**: Poppins

## Services de l'entreprise
1. **Télécommunications**: Réseaux, VoIP, fibre optique
2. **Surveillance**: Caméras IP, systèmes d'alarme, monitoring 24/7
3. **Panneaux Solaires**: Installation photovoltaïque, énergies renouvelables
4. **Plomberie & Formations**: Services de plomberie et formations techniques

## Structure du projet
```
/
├── app.py                 # Application Flask principale
├── requirements.txt       # Dépendances Python
├── .env.example          # Variables d'environnement exemples
├── templates/            # Templates HTML
│   ├── base.html         # Template de base
│   ├── index.html        # Page d'accueil
│   ├── about.html        # Page à propos
│   ├── services.html     # Page services
│   ├── projects.html     # Page projets/galerie
│   └── contact.html      # Page contact avec formulaire
└── static/              # Assets statiques (si nécessaires)
```

## Bonnes pratiques de développement
- **Responsive Design**: Toujours privilégier le mobile-first
- **Performance**: Optimiser les images et animations
- **SEO**: Utiliser les balises meta appropriées
- **Accessibilité**: Respecter les standards WCAG
- **Code**: Maintenir un code propre et commenté

## Fonctionnalités spécifiques
- Formulaire de contact avec envoi d'emails via Flask-Mail
- Animations modernes avec Tailwind CSS
- Support WhatsApp intégré
- Design avec effets glass et gradients
- Navigation responsive avec menu mobile

## Configuration de développement
1. Créer un fichier `.env` basé sur `.env.example`
2. Configurer les variables d'environnement pour Flask-Mail
3. Lancer avec `python app.py` en mode développement

## Contact et informations
- **Entreprise**: RUWACH-TECH Sarl
- **Localisation**: Lomé, Togo
- **Téléphone**: +228 90 12 34 56
- **Email**: contact@ruwach-tech.tg

## Instructions pour Copilot
- Respecter l'identité visuelle (couleurs bleu nuit/bleu électrique)
- Maintenir la cohérence du design moderne et professionnel
- Optimiser pour les performances et le SEO
- Garder le code simple et maintenable
- Privilégier les solutions vanilla JS quand possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Topboy77Mp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
