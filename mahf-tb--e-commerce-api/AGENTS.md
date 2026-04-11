# Backend API - Instructions pour Agents IA

## Architecture Globale

Backend Node.js/Express pour une application e-commerce complète. Architecture modulaire avec séparation stricte en couches :
- **Entry Point** : `server.js` → `src/index.js` (Express app) → `src/routes/index.js` (routing)
- **Modules** : Pattern `modules/{domain}/{domain}.{routes|controller|service|validation}.js`
- **Models** : Mongoose schemas avec discriminators (`src/models/`) - Client hérite de User
- **Middlewares** : Auth JWT, validation Joi, upload Multer, rate limiting
- **Utils** : JWT, password hashing, file handling, reference generation, email service

## Structure Modulaire Stricte

Chaque module (produit, commande, auth, etc.) suit cette structure :
```
src/modules/{module}/
  ├── {module}.routes.js     # Routes Express
  ├── {module}.controller.js # Handlers req/res, validation d'accès
  ├── {module}.service.js    # Logique métier, opérations DB
  ├── {module}.validation.js # Schémas Joi
  └── {module}.helper.js     # Fonctions utilitaires (optionnel)
```

**Ne jamais** placer de logique métier dans les contrôleurs. Les contrôleurs appellent les services.

## Patterns Critiques

### Validation avec Joi
Utiliser le middleware `validate()` avec schémas Joi dans routes :
```javascript
const validate = require('../../middlewares/validate');
const { createSchema } = require('./produit.validation');
router.post('/', authMiddleware, validate(createSchema), controller.create);
```

Messages d'erreur en français avec `.messages()` dans les schémas.

### Authentification & Autorisation
- **Auth middleware** : `src/middlewares/auth.middleware.js` - extrait JWT du header `Authorization: Bearer {token}`
- Injecte `req.user_auth` avec `{ id, email, role }`
- **Roles** : `client`, `admin`, `manager`, `support`
- **authorizeRole** : `src/middlewares/authorizeRole.js` - vérifier les rôles
- Staff = `['admin', 'manager', 'support']` - patterns dans [commande.controller.js](src/modules/commande/commande.controller.js#L13)

### Gestion des Fichiers
- **Upload** : Multer configuré dans `src/middlewares/upload.js`
- Stockage local : `uploads/{avatars|logos|produits|others}/`
- Route statique : `/uploads` servie par Express
- Helper : `src/utils/file.js` pour `prefixUrlWithBase()`, `removeFile()`

### Produits avec Variantes
Modèle complexe dans [produit.model.js](src/models/produit.model.js) :
- Chaque produit a `variantes[]` (couleur-taille, etc.)
- Chaque variante : `{ variant, code, qte, prixUnitaire, statut, etatProduit, seuil }`
- Helpers dans [produit.helper.js](src/modules/produit/produit.helper.js) : `mapProduitToVariant()`, `mapProduitToRow()`

### Commandes avec Références Auto
- Références format `CMD-2025-0001` générées automatiquement
- Logique dans [src/utils/reference.js](src/utils/reference.js) : `nextReferenceFromLast()`
- Pre-save hook dans [commande.model.js](src/models/commande.model.js)
- Lignes de commande : snapshots produit/variante avec `{ produit, variantId, codeVariant, prixUnitaire, quantite }`

### OAuth Google
- Configuration dans [src/config/google.client.js](src/config/google.client.js)
- Validation token avec `google-auth-library`
- Routes dans [auth.routes.js](src/modules/auth/auth.routes.js)

### Authentification Complète
Module auth avec fonctionnalités étendues dans [auth.service.js](src/modules/auth/auth.service.js) :
- **Login/Register** : Local (email/password) + Google OAuth
- **Mot de passe oublié** : `forgotPassword()` génère token sécurisé (SHA256), expire 1h, envoie email avec lien
- **Reset password** : `resetPassword(token, newPassword)` vérifie token et change le mot de passe
- **Change password** : `changePassword()` pour utilisateurs connectés (comptes locaux uniquement)
- **Update profile** : `updateProfile()` pour modifier username, nom, prenom, telephone, adresse
- **Avatar management** : `uploadAvatar()` upload photo → `/uploads/avatars/`, `deleteAvatar()` supprime (sauf Google)
- **Pattern sécurité** : Token reset hashé en DB, ne jamais stocker en clair, validation provider avant changement password
- Documentation complète : [docs/AUTH_FEATURES.md](docs/AUTH_FEATURES.md)

### Service Email (Nodemailer)
- Configuration SMTP dans [config/env.config.js](config/env.config.js) : `SMTP_HOST`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM`
- Service centralisé : [src/utils/email.js](src/utils/email.js)
- Templates e-commerce : confirmation commande, changement statut, bienvenue, panier abandonné, alerte stock
- **Intégration automatique** : 
  - [commande.service.js](src/modules/commande/commande.service.js) → `sendCommandeConfirmation()` après création
  - [commande.service.js](src/modules/commande/commande.service.js) → `sendCommandeStatusChange()` lors changement statut
  - [commande.helper.js](src/modules/commande/commande.helper.js) → `sendLowStockAlert()` si stock ≤ seuil
  - [auth.service.js](src/modules/auth/auth.service.js) → `sendWelcomeEmail()` après inscription
- Graceful fallback : si SMTP non configuré, logs warning mais n'empêche pas l'app
- **Pattern** : try/catch autour des appels email pour éviter blocages (non-critique)

### Gestion du Stock & Popularité
- **Décrémentation stock** : Transactions Mongoose avec `session.withTransaction()` dans [commande.service.js](src/modules/commande/commande.service.js)
- Vérification disponibilité avant création commande
- **Champ popularité** : Incrémenté automatiquement lors de commandes (stratégie configurable : par quantité ou par commande)
- **Alerte stock faible** : Comparer `variant.qte` vs `variant.seuil` → envoyer email admin

### Notifications en Base
- Modèle [notification.model.js](src/models/notification.model.js) : types `commande_creee`, `commande_statut_change`, `paiement_statut_change`, etc.
- Champs : `user`, `type`, `titre`, `message`, `estLu`, `context`, `lien`
- **Pattern** : Créer notification + envoyer email en parallèle lors d'événements majeurs

### Favoris & Avis Clients
- **Favoris** : [favoris.model.js](src/models/favoris.model.js) - Liste polymorphique avec `refPath` (Produit ou Commande)
- **Avis** : [avis.model.js](src/models/avis.model.js) - Note (1-5) + commentaire par produit
- Champs agrégés sur Produit : `nombreAvis`, `noteMoyenne` (à calculer/mettre à jour)
- Routes séparées : `src/modules/avis/` (CRUD complet avec ownership validation)

### Modèle Client/User avec Discriminator
- **User** (base) : [user.model.js](src/models/user.model.js) - `email`, `password`, `role`
- **Client** (héritage) : [client.model.js](src/models/client.model.js) - ajoute `nom`, `prenom`, `adresse`, `telephone`, `commandes[]`
- Utiliser `User.discriminator('Client', clientSchema)` pour polymorphisme
- Requêtes : `Client.findById()` retourne document avec tous les champs

### Workflow Commande Complet
1. **Création** : Validation items → Vérif stock → Transaction Mongoose → Snapshot prix/produit → Décrémentation stock → Calcul totaux
2. **Référence auto** : Format `CMD-YYYY-NNNN` via pre-save hook + [reference.js](src/utils/reference.js)
3. **Statuts** : `en_attente` → `en_preparation` → `expediee` → `livree` (ou `annulee`)
4. **Paiement** : `en_attente`, `paye`, `echoue`, `remboursee`, `non_paye` - Modes : `espece`, `card`, `mvola`, `airtelMoney`, `orangeMoney`
5. **Notifications** : Envoyer email + créer notification DB à chaque changement de statut
6. **Accès** : Client voit ses commandes, staff voit tout - helper `canAccessCommande()` dans [commande.helper.js](src/modules/commande/commande.helper.js)

## Best Practices

### Validation & Sécurité
- Toujours vérifier `mongoose.Types.ObjectId.isValid()` avant requêtes DB (voir services)
- Valider ownership : un client ne modifie que ses propres ressources (voir `canAccessCommande()`)
- Populer relations (`marque`, `categorie`, `client`, `produit`) avec `.populate()` pour enrichir les réponses
- Sanitiser les fichiers uploadés : extensions autorisées, taille max (config Multer dans `upload.js`)

### Transactions & Cohérence de Données
- Utiliser transactions Mongoose pour opérations atomiques (ex: commande + décrémentation stock)
- Pattern : `const session = await mongoose.startSession(); await session.withTransaction(async () => {...})`
- Snapshots immutables : copier prix/nom produit dans lignes de commande (pas de références dynamiques)

### Performance & Scalabilité
- Indexes Mongoose : définir sur `email`, `reference`, `client`, `statut` pour requêtes fréquentes
- Pagination obligatoire : `page`, `limit` dans queries list (voir [commande.controller.js](src/modules/commande/commande.controller.js#L87))
- Agrégations MongoDB : calculer `minPrice`, `stockTotal`, `noteMoyenne` via pipelines (voir [produit.service.js](src/modules/produit/produit.service.js))

### Gestion des Erreurs
- Ne jamais exposer stack traces en production : gestionnaire global dans [errorHandler.js](config/errorHandler.js)
- Statuts HTTP cohérents : 201 (créé), 400 (validation), 401 (non auth), 403 (interdit), 404 (introuvable)
- Logger avec Winston : `logger.error()`, `logger.info()` (voir [logger.js](src/config/logger.js))

### Features E-commerce Non Implémentées
- Tests unitaires/intégration (`npm test` non configuré - TODO)
- Webhooks paiement (Stripe, PayPal) - config présente mais pas d'endpoints
- Système de coupons/réductions
- Historique de prix produits
- Multi-langue (i18n)
- CI/CD et déploiement automatisé

## Configuration & Environnement

Variables d'env centralisées dans [config/env.config.js](config/env.config.js) :
- **MongoDB** : `MONGO_URI`, `DB_NAME`
- **JWT** : `JWT_SECRET`, `JWT_EXPIRES_IN`, `SALT_ROUNDS`
- **CORS** : `CLIENT_ORIGIN` (frontend URL, par défaut `http://localhost:5173`)
- **Google OAuth** : `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- **SMTP Email** : `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM`
- **Paiement** (prévu) : `STRIPE_SECRET_KEY`, `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`
- **Storage** : `BASE_URL` pour préfixer URLs fichiers (uploads statiques)
- **Rate Limiting** : `rateLimit.windowMs`, `rateLimit.max` (config anti-DDoS)

## Commandes de Développement

```bash
npm run dev    # nodemon avec watch sur src/ et config/
npm start      # production
npm run lint   # ESLint
```

Le serveur affiche l'IP locale au démarrage via [config/ip-adress.config.js](config/ip-adress.config.js).

## Conventions de Code

- **CommonJS** : `require()`/`module.exports` (type: "commonjs" dans package.json)
- **Error handling** : Gestionnaire global dans [config/errorHandler.js](config/errorHandler.js)
- **Logging** : Winston configuré dans [src/config/logger.js](src/config/logger.js), Morgan pour HTTP logs
- **Security** : Helmet, CORS avec credentials, rate limiting (config dans env.config.js)
- **Retours JSON** : Toujours renvoyer `res.json()` avec statut approprié

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mahf-TB)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mahf-TB)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
