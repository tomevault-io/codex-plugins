# Instructions rapides pour les agents IA

Résumé court
- Projet Laravel (PHP ^8.2, Laravel 11) avec une UI front React (dans `api_front/public/app`) et un panneau d'administration basé sur Filament v5.
- Frontend build via `laravel-mix` (voir `webpack.mix.js`), back-end standard Laravel avec commandes artisan personnalisées.
- **Langue** : Toujours répondre en français pour chaque nouvelle conversation, aussi bien dans le chat de discussion que dans tous les fichiers générés (plans d'implémentation, bilan/walkthrough, etc.).

Ce que l'agent doit savoir en premier
- Lire `composer.json` pour dépendances (Filament, Scout/Meilisearch, Sanctum, Fortify).
- Lire `webpack.mix.js` pour comprendre les alias JS (`@app`, `@components`, etc.) et la logique d'environnement (fichiers `.env.*` dans `api_front/public/app/`).
- L'admin panel est géré avec Filament — voir `app/Providers/Filament/AdminPanelProvider.php` (branche: `exchange-or-features-for-filamentphp`).

Commandes de développement importantes
- Installer backend: `composer install` (attention aux dépendances Google Drive ; README décrit contournement Windows pour le timeout).
- Installer frontend: `npm install` depuis la racine (les scripts sont dans `package.json`).
- Compiler assets (exemples):
  - développement : `npm run dev`
  - production/preprod : `npm run prod` ou `npm run pre-prod`
- Commandes artisan utiles (décrites dans `README.md`):
  - `php artisan key:generate`
  - `php artisan ide-helper:generate` (IDE helper)
  - `php artisan project:fresh_db` (réinitialise + seed — destructive)
  - `php artisan project:init_storage` (crée arborescence Google Drive)
  - `php artisan project:init` (réinitialisation DB alternative)

Conventions et patterns projet
- PSR-4 mapping personnalisé (voir `composer.json` `autoload`):
  - `App\` -> `app/`
  - `Api\` -> `api_front/api/` (API côté front)
  - `School\` -> `school/`
  - `Models\` -> `Models/` (les modèles principaux sont ici)
- Frontend React : code dans `api_front/public/app`, les builds écrivent dans `public/`.
- Variables d'environnement front end : `api_front/public/app/.env.{environment}` (chargées par `webpack.mix.js`).
- Stockage cloud : adapter Google Drive ; le projet attend un fichier `creds.json` (ou `credentialsofserviceaccount.json`) à la racine — voir `README.md`.
- Recherche : Laravel Scout + Meilisearch. Dossier `meili_data/` contient configuration pour exécution locale (Docker + Windows instructions dans README).

Points d'intégration et d'attention
- Filament : plusieurs commandes de post-install (voir scripts composer) incluent `@php artisan filament:upgrade` — attention aux migrations du panneau d'administration.
- Scripts composer post-update publient les assets et génèrent l'IDE helper (`vendor:publish`, `ide-helper:generate`).
- Webpack/Mix : compression d'images via `imagemin-webpack-plugin` — builds CI pourraient devoir exécuter avec Node 16 (README recommande Node v16.9).
- DB locale : le README montre `DB_PORT=3307` (Laragon) — ne pas présumer du port par défaut.

Tests et CI
- Pas de tests unitaires maintenus dans ce dépôt. Ne pas écrire ni compter sur des tests unitaires automatiques.
- Pour exécuter les tests existants (si présents) : `php artisan test`.
- `phpunit.xml` existe et contient des variables d'environnement utiles (`CACHE_DRIVER=array`, `QUEUE_CONNECTION=sync`) — utiliser ces réglages si vous exécutez des tests d'intégration/feature.

Exemples concrets à utiliser par l'agent
- Si vous cherchez la logique d'administration : ouvrir `app/Providers/Filament/AdminPanelProvider.php` et `app/Actions/`.
- Pour comprendre l'API front-end, inspecter `api_front/api/routes.php` et `api_front/public/app` (aliases et points d'entrée dans `webpack.mix.js`).
- Pour trouver seeders et fixtures : `database/seeders/` et `storage/` (fichiers CSV utilisés par seeders, voir README).

Ce que l'agent ne doit pas faire automatiquement
- Ne pas lancer de commandes destructives sans confirmation explicite (ex: `project:fresh_db`, `project:init`).
- Ne pas modifier les clés d'environnement sensibles ni pousser de `creds.json`/fichiers de clé.

Questions à poser au développeur (si incertain)
- Quel est l'URL local habituel (ex: `https://qleiade.test`) et la configuration Laragon souhaitée ?
- La migration vers Filament est-elle terminée ou doit-on conserver du code legacy à supprimer progressivement ?
- Y a‑t‑il un workflow CI/CD (build, tests, déploiement) à respecter pour les PRs ?

Fichiers de référence à consulter en priorité
- `README.md` (racine)
- `composer.json`, `package.json`, `webpack.mix.js`
- `phpunit.xml`
- `app/Providers/Filament/AdminPanelProvider.php`
- `api_front/public/app/.env.*` (exemples d'env front)

===

<laravel-boost-guidelines>

=== Règles fondamentales ===

# Consignes Laravel Boost

Les consignes Laravel Boost ont été spécialement élaborées par les mainteneurs de Laravel pour cette application. Il est important de les suivre scrupuleusement afin d'améliorer la satisfaction des utilisateurs lors du développement d'applications Laravel.

## Contexte fondamental

Cette application est une application Laravel. Les principaux packages et versions de son écosystème Laravel sont indiqués ci-dessous. Vous maîtrisez parfaitement ces éléments. Assurez-vous de respecter scrupuleusement ces packages et versions.

- php - 8.2.23
- filament/filament (FILAMENT) - v5
- laravel/fortify (FORTIFY) - v1
- laravel/framework (LARAVEL) - v11
- laravel/mcp (MCP) - v0
- laravel/prompts (PROMPTS) - v0
- laravel/sanctum (SANCTUM) - v4
- laravel/scout (SCOUT) - v10
- laravel/socialite (SOCIALITE) - v5
- livewire/livewire (LIVEWIRE) - v4
- laravel/sail (SAIL) - v1
- phpunit/phpunit (PHPUNIT) - v10
- eslint (ESLINT) - v7
- laravel-echo (ECHO) - v1
- react (REACT) - v17

## Conventions
- Vous devez respecter le code existant. Conventions utilisées dans cette application. Lors de la création ou de la modification d'un fichier, vérifiez que les fichiers frères respectent la structure, l'approche et la dénomination.
- Utilisez des noms descriptifs pour les variables et les méthodes. Par exemple, `isRegisteredForDiscounts`, et non `discount()`.
- Recherchez les composants existants pouvant être réutilisés avant d'en écrire un nouveau.

## Scripts de vérification
- Ne créez pas de scripts de vérification et évitez toute modification lorsque les tests couvrent déjà cette fonctionnalité et prouvent son bon fonctionnement. Les tests unitaires et fonctionnels sont prioritaires.

## Structure et architecture de l'application

- Respectez la structure de répertoires existante ; ne créez pas de nouveaux dossiers de base sans autorisation.
- Ne modifiez pas les dépendances de l'application sans autorisation.

## Regroupement du frontend

- Si l'utilisateur ne constate aucune modification du frontend dans l'interface utilisateur, il peut être nécessaire d'exécuter `npm run build`, `npm run dev` ou `composer run dev`. Demandez-lui de l'aide.

## Réponses

- Soyez concis dans vos explications : concentrez-vous sur l’essentiel plutôt que de détailler des évidences.
- **Répondre systématiquement en français.** Cela s’applique au chat ainsi qu’à tous les documents produits (fichiers Markdown de plan, de walkthrough, de tâche, etc.).

## Fichiers de documentation

- Vous ne devez créer de fichiers de documentation que si l’utilisateur vous le demande explicitement.

=== Règles Boost ===

## Laravel Boost

- Laravel Boost est un serveur MCP doté d’outils puissants conçus spécifiquement pour cette application. Utilisez-les.

## Artisan

- Utilisez l’outil `list-artisan-commands` lorsque vous devez appeler une commande Artisan afin de vérifier les paramètres disponibles.

## URL

- Lorsque vous partagez l’URL d’un projet avec l’utilisateur, vous devez utiliser l’outil `get-absolute-url` pour vous assurer d’utiliser le schéma, le domaine/l’adresse IP et le port corrects.

## Expérimentation / Débogage

- Utilisez l'outil `tinker` pour exécuter du code PHP afin de déboguer ou interroger directement les modèles Eloquent.
- Utilisez l'outil `database-query` pour lire uniquement les données de la base de données.

## Lecture des journaux du navigateur avec l'outil `browser-logs`

- L'outil `browser-logs` de Boost vous permet de lire les journaux, erreurs et exceptions de votre navigateur.
- Seuls les journaux récents sont utiles ; ignorez les anciens.

## Recherche dans la documentation (Essentiel)

- Boost intègre l'outil puissant `search-docs`, à utiliser en priorité pour la recherche dans Laravel et ses packages. Cet outil transmet automatiquement la liste des packages installés et leurs versions à l'API Boost, afin de ne renvoyer que la documentation spécifique à la version et à votre situation. Si vous recherchez la documentation d'un package en particulier, vous pouvez spécifier un tableau de packages à filtrer. L'outil `search-docs` est idéal pour tous les packages liés à Laravel, notamment Laravel, Inertia, Livewire, Filament, Tailwind, Pest, Nova, Nightwatch, etc.
- Il est indispensable d'utiliser cet outil pour rechercher la documentation de l'écosystème Laravel avant d'envisager d'autres méthodes.
- Consultez la documentation avant de modifier votre code afin de vous assurer d'adopter la bonne approche.
- Utilisez plusieurs requêtes simples, générales et thématiques pour commencer. Par exemple : `['limite de débit', 'limite de débit de routage', 'routage']`.
- N'ajoutez pas de noms de packages aux requêtes ; les informations relatives aux packages sont déjà partagées. Par exemple, utilisez `table de ressources de test`, et non `table de ressources de test Filament 4`.

### Syntaxe de recherche disponible

Vous pouvez et devez effectuer plusieurs requêtes simultanément. Les résultats les plus pertinents seront affichés en premier.

1. Recherche de mots simples avec lemmatisation automatique : `query=authentication` trouve « authenticate » et « auth ».
2. Recherche de plusieurs mots (opérateur ET) : `query=rate limit` trouve les informations contenant à la fois « rate » ET « limit ».
3. Expressions entre guillemets (position exacte) : `query="infinite scroll"` : les mots doivent être adjacents et dans cet ordre.
4. Requêtes mixtes : `query=middleware "rate limit"` : « middleware » ET l'expression exacte « rate limit ».
5. Requêtes multiples : `queries=["authentication", "middleware"]` : N'IMPORTE QUEL de ces termes.

=== Règles PHP ===

## PHP

- Utilisez toujours des accolades pour les structures de contrôle, même si elles ne comportent qu'une seule ligne.

### Constructeurs

- Utilisez la promotion des propriétés du constructeur PHP 8 dans `__construct()`.
- <code-snippet>public function __construct(public GitHub $github) { }</code-snippet>
- N'autorisez pas les méthodes `__construct()` vides (sans paramètre) sauf si le constructeur est privé.

### Déclarations de types

- Utilisez toujours des déclarations explicites de type de retour pour les méthodes et les fonctions.
- Utilisez les annotations de type PHP appropriées pour les paramètres des méthodes.

<code-snippet name="Types de retour et paramètres de méthodes explicites" lang="php">
protected function isAccessible(User $user, ?string $path = null): bool

{
...
}
</code-snippet>

## Commentaires
- Privilégiez les blocs PHPDoc aux commentaires en ligne. N'utilisez jamais de commentaires directement dans le code, sauf en cas de complexité majeure.

## Blocs PHPDoc
- Ajoutez des définitions de type utiles pour les tableaux, le cas échéant.

## Énumérations
- En général, les clés d'une énumération doivent être en casse de titre (TitleCase). Par exemple : `FavoritePerson`, `BestLake`, `Monthly`.

=== Règles Laravel/Core ===

## Adopter les bonnes pratiques Laravel

- Utilisez les commandes `php artisan make:` pour créer de nouveaux fichiers (migrations, contrôleurs, modèles, etc.). Vous pouvez lister les commandes Artisan disponibles avec l'outil `list-artisan-commands`.
- Pour créer une classe PHP générique, utilisez `php artisan make:class`.
- Ajoutez l'option `--no-interaction` à toutes les commandes Artisan pour qu'elles fonctionnent sans intervention de l'utilisateur. Spécifiez également les options `--options` appropriées pour un comportement correct.

### Base de données

- Utilisez toujours les méthodes de relation Eloquent appropriées avec des indications de type de retour. Privilégiez les méthodes de relation aux requêtes brutes ou aux jointures manuelles.
- Utilisez les modèles et les relations Eloquent avant de suggérer des requêtes de base de données brutes.
- Évitez `DB::` ; préférez `Model::query()`. Générez du code qui exploite les capacités de l'ORM de Laravel plutôt que de les contourner.
- Générez du code qui prévient les problèmes de requêtes N+1 en utilisant le chargement anticipé.
- Utilisez le générateur de requêtes de Laravel pour les opérations de base de données très complexes.

### Création de modèles

- Lors de la création de nouveaux modèles, créez également des fabriques et des seeders utiles. Demandez à l'utilisateur s'il a besoin d'autres éléments, en utilisant `list-artisan-commands` pour vérifier les options disponibles pour `php artisan make:model`.

### API et ressources Eloquent

- Pour les API, utilisez par défaut les ressources d'API Eloquent et le versionnage d'API, sauf si les routes d'API existantes ne le font pas, auquel cas vous devez suivre les conventions de l'application existante.

### Contrôleurs et validation

- Créez toujours des classes de requêtes de formulaire pour la validation plutôt que d'utiliser la validation en ligne dans les contrôleurs. Incluez à la fois les règles de validation et les messages d'erreur personnalisés.
- Vérifiez les requêtes de formulaire sœurs pour déterminer si l'application utilise des règles de validation basées sur des tableaux ou des chaînes de caractères.

### Files d'attente

- Utilisez les tâches en file d'attente pour les opérations longues avec l'interface `ShouldQueue`.

### Authentification et autorisation

- Utilisez les fonctionnalités d'authentification et d'autorisation intégrées de Laravel (gates, policies, Sanctum, etc.).

### Génération d'URL

- Lors de la génération de liens vers d'autres pages, privilégiez les routes nommées et la fonction `route()`.

### Configuration

- Utilisez les variables d'environnement uniquement dans les fichiers de configuration ; n'utilisez jamais la fonction `env()` directement en dehors des fichiers de configuration. Utilisez toujours `config('app.name')`, et non `env('APP_NAME')`.

### Tests

- Lors de la création de modèles pour les tests, utilisez les fabriques de modèles. Vérifiez si la fabrique possède des états personnalisés utilisables avant de configurer manuellement le modèle.
- Faker : Utilisez des méthodes telles que `$this->faker->word()` ou `fake()->randomDigit()`. Respectez les conventions existantes pour l'utilisation de `$this->faker` ou `fake()`.
- Lors de la création de tests, utilisez `php artisan make:test [options] {name}` pour créer un test fonctionnel et `--unit` pour créer un test unitaire. La plupart des tests devraient être des tests fonctionnels.

### Erreur Vite
- Si vous recevez l'erreur « Illuminate\Foundation\ViteException : Impossible de localiser le fichier dans le manifeste Vite », vous pouvez exécuter `npm run build` ou demander à l'utilisateur d'exécuter `npm run dev` ou `composer run dev`.

=== Règles Laravel/v11 ===

## Laravel 11

- Utilisez l'outil `search-docs` pour obtenir la documentation spécifique à la version.
- Ce projet a été mis à niveau depuis Laravel 10 sans migrer vers la nouvelle structure de fichiers simplifiée de Laravel 11.
- Ceci est **parfaitement correct** et recommandé par Laravel. Conservez la structure existante de Laravel 10. Il n'est pas nécessaire de migrer vers la structure de Laravel 11, sauf si l'utilisateur le demande explicitement.

### Structure de Laravel 10

- Les middlewares se trouvent généralement dans `app/Http/Middleware/` et les fournisseurs de services dans `app/Providers/`.
- Il n'existe pas de fichier de configuration `bootstrap/app.php` dans une structure Laravel 10 :
    - L'enregistrement des middlewares se trouve dans `app/Http/Kernel.php`
    - La gestion des exceptions se trouve dans `app/Exceptions/Handler.php`
    - L'enregistrement des commandes de la console et de la planification se trouve dans `app/Console/Kernel.php`
    - Les limites de débit sont probablement définies dans `RouteServiceProvider` ou `app/Http/Kernel.php`

### Base de données

- Lors de la modification d'une colonne, la migration doit inclure tous les attributs précédemment définis pour cette colonne. Dans le cas contraire, ils seront supprimés.
- Laravel 11 permet de limiter nativement le nombre d'enregistrements chargés immédiatement, sans package externe : `$query->latest()->limit(10);`.

### Modèles

- Les conversions de type (casts) peuvent et devraient probablement être définies dans une méthode `casts()` d'un modèle plutôt que dans la propriété `$casts`. Suivez les conventions existantes pour les autres modèles.

### Nouvelles commandes Artisan

- Listez les commandes Artisan à l'aide de l'outil MCP de Boost, s'il est disponible. Nouvelles commandes disponibles dans Laravel 11 :
    - `php artisan make:enum`
    - `php artisan make:class`
    - `php artisan make:interface`

=== Règles Livewire/Core ===

## Livewire

- Utilisez l'outil `search-docs` pour trouver la documentation spécifique à votre version concernant l'écriture de composants Livewire et de tests Livewire.
- Utilisez la commande Artisan `php artisan make:livewire [Posts\CreatePost]` pour créer de nouveaux composants.
- L'état doit être stocké sur le serveur et reflété par l'interface utilisateur.
- Toutes les requêtes Livewire accèdent au backend Laravel ; elles sont similaires aux requêtes HTTP classiques. Validez systématiquement les données des formulaires et effectuez des contrôles d'autorisation dans les actions Livewire.

## Bonnes pratiques Livewire

- Les composants Livewire nécessitent un seul élément racine.
- Utilisez `wire:loading` et `wire:dirty` pour des états de chargement agréables. - Ajoutez `wire:key` dans les boucles :

    ```blade
    @foreach ($items as $item)
        <div wire:key="item-{{ $item->id }}">
            {{ $item->name }}
        </div>
    @endforeach
    ```

- Privilégiez les hooks de cycle de vie comme `mount()` et `updatedFoo()` pour l'initialisation et les effets de bord réactifs :

<code-snippet name="Exemples de hooks de cycle de vie" lang="php">
    public function mount(User $user) { $this->user = $user; }
    public function updatedSearch() { $this->resetPage(); }
</code-snippet>

## Test de Livewire

<code-snippet name="Exemple de test du composant Livewire" lang="php">
    Livewire::test(Counter::class)
        ->assertSet('count', 0)
        ->call('increment')
        ->assertSet('count', 1)
        ->assertSee(1)
        ->assertStatus(200);
</code-snippet>

<code-snippet name="Test de présence du composant Livewire sur la page" lang="php">
    $this->get('/posts/create')->assertSeeLivewire(CreatePost::class);
</code-snippet>

=== Règles phpunit/core ===

## PHPUnit

- Cette application utilise PHPUnit pour les tests. Tous les tests doivent être écrits sous forme de classes PHPUnit. Utilisez `php artisan make:test --phpunit {nom}` pour créer un nouveau test.
- Si un test utilise « Pest », convertissez-le en PHPUnit.
- À chaque mise à jour d'un test, exécutez-le.
- Lorsque les tests relatifs à votre fonctionnalité réussissent, demandez à l'utilisateur s'il souhaite exécuter l'ensemble des tests pour vérifier que tout fonctionne correctement.
- Les tests doivent couvrir tous les cas de figure : nominaux, erreurs et comportements anormaux.
- Vous ne devez supprimer aucun test ni fichier de test du répertoire `tests` sans autorisation. Il ne s'agit pas de fichiers temporaires ou auxiliaires ; ce sont des éléments essentiels de l'application.

### Exécution des tests

- Exécutez le nombre minimal de tests, en utilisant un filtre approprié, avant de finaliser.
- Pour exécuter tous les tests : `php artisan test --compact`.
- Pour exécuter tous les tests d'un fichier : `php artisan test --compact tests/Feature/ExampleTest.php`.
- Pour filtrer sur un nom de test particulier : `php artisan test --compact --filter=nom_du_test` (recommandé après avoir modifié un fichier associé).

</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/la-cite-des-formations)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/la-cite-des-formations)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
