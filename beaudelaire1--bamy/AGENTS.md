<!-- Fichier rédigé en français pour guider un agent IA travaillant sur ce dépôt Xeros -->
# Instructions rapides pour les agents IA (Xeros)

Ce dépôt est une application Django « monolithique » modulaire (plusieurs apps découplées). Ces consignes permettent d'être immédiatement productif.

- **Vue d'ensemble** : projet Django avec apps principales `core`, `catalog`, `cart`, `orders`, `userauths`, `payments`, `integrations`, `marketing`, `loyalty`, `reviews`, `returns`, `crm`. Le point d'entrée est `manage.py`.
- **Architecture clé** : séparation par app (single responsibility). Les apps communiquent essentiellement par imports ORM, clés étrangères et sessions (panier).

- **Points importants à connaître**:
  - `xeros_project/settings.py` : utilise `django-environ` si installé, sinon un fallback. Variables sensibles attendues dans `local_settings.py` ou `.env`.
  - `AUTH_USER_MODEL = "userauths.User"` — utilisateur personnalisé.
  - REST API est conditionnelle : définir `ENABLE_REST_API=True` et installer `djangorestframework` pour activer l'app `api`.
  - Cart : la classe session-based est `cart.Cart` (`cart/cart.py`). Le panier est stocké dans `request.session[CART_SESSION_ID]` (clé configurée dans `settings.py`).
  - Payments : clés publiques exposées via `payments.context_processors.payment_public_keys` (variables `PAYPAL_CLIENT_ID`, `STRIPE_PUBLISHABLE_KEY`). Les intégrations serveur se trouvent sous `payments/paypal.py` et `payments/stripe_api.py`.
  - Asynchrone : Celery est configuré (`xeros_project/celery.py`). Démarrer un worker :

    ```bash
    celery -A xeros_project worker -l info
    ```

- **Commandes et workflows dev courants** :
  - Créer venv & installer : `python -m venv venv` puis `pip install -r requirements.txt`.
  - Migrer DB : `python manage.py migrate` (SQLite par défaut en dev, production -> PostgreSQL).
  - Lancer serveur local : `python manage.py runserver`.
  - Collectstatic : `python manage.py collectstatic`.
  - Créer superuser : `python manage.py createsuperuser`.

- **Scripts utiles** :
  - `audit_xeros.py` (à la racine) analyse statiquement l'arborescence, les URLs nommées et settings. Exécuter depuis la racine :

    ```bash
    python audit_xeros.py
    # produit audit_report.md et audit_findings.txt
    ```

- **Conventions de code et patterns observés** :
  - Panier tolérant : `Cart.add()` accepte `product` ou `product_id`, `qty` ou `quantity`, et applique `MIN_QTY` et condition `order_in_packs`.
  - Templates utilisent des context processors globaux : `core.context_processors.brand`, `cart.context_processors.cart`, `payments.context_processors.payment_public_keys`.
  - Administration enrichie par `jazzmin` (branding dans `settings.py`).

- **Quand modifier quoi** (exemples précis) :
  - Modifier la logique métier du panier → `cart/cart.py` (préserver format session `{product_id: {"qty":..., "unit_price": "..."}}`).
  - Ajouter une route API → mettre `ENABLE_REST_API=True`, installer `djangorestframework`, ajouter les vues dans `api/` et déclarer `urls` dans `xeros_project/urls.py`.
  - Intégrer un nouveau provider de paiement → ajouter module sous `payments/`, exposer clés publiques via `payments.context_processors`, et lire secrets via `local_settings.py`.

- **Tests & qualité** :
  - Des tests existent dans des dossiers `*/tests.py` et `*/tests/` (ex : `catalog/tests`, `orders/tests`). Utiliser `pytest` ou `python manage.py test` selon l'environnement.
  - Linter/formatter suggérés : `flake8`, `black` (non rigides ici mais observés dans la doc technique).

- **Sécurité / déploiement important** :
  - `settings.py` active des protections fortes quand `DEBUG=False` (HSTS, cookies sécurisés, etc.).
  - Les clés Stripe/PayPal attendues : `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`.

- **Prompts recommandés pour l'agent** (exemples) :
  - "Modifier `cart.Cart.add` pour appliquer un `max_qty` par produit ; mets à jour la session et écris un test unitaire dans `cart/tests.py`."  
  - "Ajouter une route nommée `checkout_success` dans `orders/urls.py` et créer le template `orders/checkout_success.html`."  

Si un point important manque ou si vous voulez que j'ajoute des exemples de PR/commit messages ou checks CI spécifiques, dites-moi lesquels et j'itère.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/beaudelaire1)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/beaudelaire1)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
