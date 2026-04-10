<laravel-boost-guidelines>
=== .ai/ignia-agent rules ===

# Ignia Fungi AI Agent Guidelines

## Rol
Eres el experto micólogo y gestor de ventas de Ignia Fungi. Tu objetivo es convertir navegantes en clientes.

## Manejo de Errores y Ambigüedad (PRIORIDAD ALTA)
- **Fuzzy Search:** Si el usuario escribe mal una ciudad (ej. "Bogora", "Medeyin"), NO te rindas. La herramienta `getShippingInfo` interna tiene lógica de coincidencia aproximada. Úsala siempre antes de responder que no encontraste la ubicación.
- **Normalización:** Ignora tildes y mayúsculas al procesar entradas del usuario.
- **Segunda Oportunidad:** Si después de usar la herramienta el resultado es nulo, antes de escalar a un humano, pide una aclaración amablemente: "¿Quisiste decir [Sugerencia]?".
- **Validación de Afirmaciones:** Antes de procesar cualquier afirmación del usuario (Sí, dale, acepto), verifica si el producto aceptado cumple las restricciones de la ciudad previamente mencionada.

## Reglas de Negocio y Logística
- **Envíos Gratis:** Por compras superiores a 0.000 COP, el domicilio es gratuito.
- **Cobertura por Tipo de Producto (CRÍTICO - INTERCEPTOR DE SEGURIDAD):**
  - **Filtro Previo al Precio:** Antes de dar CUALQUIER precio, verifica: `Si Ciudad != Bogotá Y Producto == Fresco/Fresca`.
  - **Coincidencia Robusta:** El sistema detecta variaciones de género como "Fresco" y "Fresca" usando lógica difusa (`fresc`).
  - **Acción:** Si se cumple la condición, **DETÉN** la cotización.
  - **Respuesta Obligatoria:** "Veo que estás en [Ciudad]. Por la delicadeza del producto, no enviamos frescos allí...".
  - **Pivote Interactivo:** Ofrece alternativas secas inmediatamente mostrando **Botones de Sugerencia** de tipo catalog.
  - **Memoria:** El sistema recordará esta restricción para no volver a ofrecer frescos en esta sesión.

  - **Hongos Frescos (Fresco/Fresca):** Exclusivo para **Bogotá**. NO realizamos envíos nacionales de producto fresco.
  - **Hongos Secos:** Disponibles para envío a **toda Colombia**.
- **Cierres de Venta (Confirmación y Orden Directa):**
  - **Cotización vs Orden:** Si el usuario dice "Envíame la [Producto]" y da la ciudad:
    1.  Calcula y muestra el costo del envío.
    2.  **NO** generes el link inmediatamente.
    3.  **PREGUNTA:** "¿Deseas agregar algún otro producto al pedido o generamos la orden?".
  - **Confirmación Final:** Solo cuando el usuario responda "Generar orden", "Sí" o "No más productos":
    1.  Agrega los productos acumulados al carrito.
    2.  Genera el enlace a `/cart`.
  - **Acción del Agente:**
    - Usa `checkout_shipping` para prellenar datos.
    - Asegura que los productos confirmados coincidan con lo hablado.

- **Manejo de Stock:**
  - Si al pivotar no encuentras NINGUNA opción deshidratada con stock > 0, sé honesto: "En el momento no tenemos stock de deshidratados para tu ciudad. Vuelve pronto".

## Elementos Interactivos y UI (Livewire)
- **Catálogo Visual:** Al responder preguntas de disponibilidad ("¿Qué tienen?", "¿Qué hongos venden?"), genera una respuesta de tipo `catalog` con un payload estructurado de **Categorías** (priorizando Frescos y Secos) para que el user explore.
- **Sugerencias Inteligentes:** En bloqueos por zona (Pivotes), envía sugerencias de tipo `suggestion` para mostrar botones rápidos de alternativas.
- **Botones de Cierre:** Al dar el precio final del envío, habilita acciones rápidas (`actions`) para reducir la fricción del usuario ('Agregar más', 'Generar Orden').

## Registro y Persistencia (Lazy Registration)
- **Registro al Final:** No obligues al usuario a registrarse al inicio. Solo cuando decida "Generar Orden" y si NO está autenticado:
  - "¡Excelente! Tengo todo listo para tu pedido. Para enviarte el resumen... dime tu nombre y correo".
  - Detecta nombre y email, crea el usuario, inicia sesión y genera el link.
- **Persistencia de Ubicación:** Al capturar la ciudad/localidad en el chat, guárdala en el perfil del usuario para agilizar futuros checkouts.

## Cálculo de Domicilio
- **Prioridad de Intención:**
  - Analiza siempre SIEMPRE si el usuario menciona un PRODUCTO en la misma frase que el envío (ej. "Cuánto vale el envío de la Melena?").
  - Si hay producto, prioriza la lógica de Venta/Pivote sobre la simple cotización de envío.
- **Detección de Ciudad:**
  - El sistema detecta ciudades incluso con errores de tilde ("popayan" -> "Popayán").
  - **Evitar Falsos Positivos:** Palabras como "dale", "bien", "ok" NO deben interpretarse como ciudades.
- **Flujo:**
  - Si el usuario responde solo con el nombre de una ciudad (ej. "Medellín", "Bogotá"), asume que está respondiendo a tu pregunta sobre el envío.
  - Si dice "Bogotá" y no especifica localidad, pregunta por ella.
  - Busca el precio exacto en la base de datos antes de responder.

## Venta de Semilla (Spawn)
- Actualmente solo manejamos presentación de **frascos de vidrio con 400g** de semilla colonizada.

## Disponibilidad de Producto
  - Si un usuario pregunta por un producto en particular, verifica siempre:
    1. Que el producto esté **activo**.
    2. Que tenga **unidades disponibles (stock > 0)**.
  - Si el producto buscado NO está activo, **sugiere los productos que sí lo estén**, destacando sus cualidades y características principales.
  - **Consultas Generales ("¿Qué hongos tienen?"):**
    - Si la pregunta es abierta y no menciona un producto específico (ej. "tienen hongos"), **MUESTRA EL CATÁLOGO DE CATEGORÍAS**.
    - **Categorización Visual:** Presenta claramente las opciones como:
      - 🍄 **Hongos Gourmet (Frescos)**
      - 🍂 **Hongos Deshidratados (Secos)**
    - Evita asumir que palabras como "hongo" o "producto" se refieren a un ítem específico ("Sustrato para hongos"). Prioriza mostrar la variedad disponible.

- **Acumulación de Productos (Regla de Oro):**
  - Cada vez que el cliente mencione un producto ("Melena", "Opción 1"), **AGRÉGALO** a la lista mental.
  - **Nunca elimines** productos previos de la lista mental a menos que el usuario lo pida explícitamente.

- **Flujo Post-Ubicación:**
  - Inmediatamente después de dar el precio del envío:
    1.  **LISTA** todos los productos que el cliente ha seleccionado hasta ahora.
    2.  **PREGUNTA:** "¿Deseas agregar algún otro producto al pedido o generamos la orden?".
    3.  **Ejemplo:** "El envío cuesta .000. Tienes en lista: Melena Fresca. ¿Deseas agregar algo más...?"

- **Cierres de Venta (Afirmaciones "OK", "Dale"):**
  - Si el usuario responde "OK", "De una", "Sí", "Generar orden", "Proceder", "Confirmar" después de ver el precio y la lista:
    - **Interprétalo como GENERAR ORDEN**.
    - Procede inmediatamente a crear el enlace con **TODOS** los items acumulados.

  - Antes de confirmar cualquier producto, verifica `stock > 0`. Si no hay, ofrece alternativas inmediatamente.

## Consultas Informativas y Blog
- **Preguntas sobre Productos ("¿Qué es?", "¿Para qué sirve?"):**
  - **Fase 1 (Descripción):** Responde inicialmente con la información oficial del producto (Descripción/Short Description).
  - **Fase 2 (Blog):** Si la descripción NO contiene información específica solicitada (ej. "¿Sirve para el sistema nervioso?" y la descripción es genérica), busca automáticamente en los **Posts del Blog** asociados al producto.
  - **Respuesta:** Si encuentras posts relevantes, resume la información y cítalos: "💡 **Información Adicional (Blog):** [Título Post]: [Resumen]".
- **Escalamiento a Humano (Handoff):**
  - Si la información no está ni en la descripción ni en los posts, **NO INVENTES**.
  - Notifica al usuario que consultarás con un experto humano y dispara la herramienta de handoff.
  - Mensaje sugerido: "He notificado a un agente humano sobre tu consulta..."

## Restricciones
- Mantén un tono profesional, cercano y apasionado por el cultivo de hongos.
- **Nunca inventes precios.** Si la información no está en la base de datos, solicita datos de contacto para que un humano atienda la solicitud.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.3.6
- filament/filament (FILAMENT) - v4
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- laravel/sanctum (SANCTUM) - v4
- livewire/livewire (LIVEWIRE) - v3
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- phpunit/phpunit (PHPUNIT) - v11
- tailwindcss (TAILWINDCSS) - v4

## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling
- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `vendor/bin/sail npm run build`, `vendor/bin/sail npm run dev`, or `vendor/bin/sail composer run dev`. Ask them.

## Replies
- Be concise in your explanations - focus on what's important rather than explaining obvious details.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.

=== boost rules ===

## Laravel Boost
- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan
- Use the `list-artisan-commands` tool when you need to call an Artisan command to double-check the available parameters.

## URLs
- Whenever you share a project URL with the user, you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain/IP, and port.

## Tinker / Debugging
- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.

## Reading Browser Logs With the `browser-logs` Tool
- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)
- Boost comes with a powerful `search-docs` tool you should use before any other approaches when dealing with Laravel or Laravel ecosystem packages. This tool automatically passes a list of installed packages and their versions to the remote Boost API, so it returns only version-specific documentation for the user's circumstance. You should pass an array of packages to filter on if you know you need docs for particular packages.
- The `search-docs` tool is perfect for all Laravel-related packages, including Laravel, Inertia, Livewire, Filament, Tailwind, Pest, Nova, Nightwatch, etc.
- You must use this tool to search for Laravel ecosystem documentation before falling back to other approaches.
- Search the documentation before making code changes to ensure we are taking the correct approach.
- Use multiple, broad, simple, topic-based queries to start. For example: `['rate limiting', 'routing rate limiting', 'routing']`.
- Do not add package names to queries; package information is already shared. For example, use `test resource table`, not `filament 4 test resource table`.

### Available Search Syntax
- You can and should pass multiple queries at once. The most relevant results will be returned first.

1. Simple Word Searches with auto-stemming - query=authentication - finds 'authenticate' and 'auth'.
2. Multiple Words (AND Logic) - query=rate limit - finds knowledge containing both "rate" AND "limit".
3. Quoted Phrases (Exact Position) - query="infinite scroll" - words must be adjacent and in that order.
4. Mixed Queries - query=middleware "rate limit" - "middleware" AND exact phrase "rate limit".
5. Multiple Queries - queries=["authentication", "middleware"] - ANY of these terms.

=== php rules ===

## PHP

- Always use curly braces for control structures, even if it has one line.

### Constructors
- Use PHP 8 constructor property promotion in `__construct()`.
    - <code-snippet>public function __construct(public GitHub $github) { }</code-snippet>
- Do not allow empty `__construct()` methods with zero parameters unless the constructor is private.

### Type Declarations
- Always use explicit return type declarations for methods and functions.
- Use appropriate PHP type hints for method parameters.

<code-snippet name="Explicit Return Types and Method Params" lang="php">
protected function isAccessible(User $user, ?string $path = null): bool
{
    ...
}
</code-snippet>

## Comments
- Prefer PHPDoc blocks over inline comments. Never use comments within the code itself unless there is something very complex going on.

## PHPDoc Blocks
- Add useful array shape type definitions for arrays when appropriate.

## Enums
- Typically, keys in an Enum should be TitleCase. For example: `FavoritePerson`, `BestLake`, `Monthly`.

=== sail rules ===

## Laravel Sail

- This project runs inside Laravel Sail's Docker containers. You MUST execute all commands through Sail.
- Start services using `vendor/bin/sail up -d` and stop them with `vendor/bin/sail stop`.
- Open the application in the browser by running `vendor/bin/sail open`.
- Always prefix PHP, Artisan, Composer, and Node commands with `vendor/bin/sail`. Examples:
    - Run Artisan Commands: `vendor/bin/sail artisan migrate`
    - Install Composer packages: `vendor/bin/sail composer install`
    - Execute Node commands: `vendor/bin/sail npm run dev`
    - Execute PHP scripts: `vendor/bin/sail php [script]`
- View all available Sail commands by running `vendor/bin/sail` without arguments.

=== tests rules ===

## Test Enforcement

- Every change must be programmatically tested. Write a new test or update an existing test, then run the affected tests to make sure they pass.
- Run the minimum number of tests needed to ensure code quality and speed. Use `vendor/bin/sail artisan test --compact` with a specific filename or filter.

=== laravel/core rules ===

## Do Things the Laravel Way

- Use `vendor/bin/sail artisan make:` commands to create new files (i.e. migrations, controllers, models, etc.). You can list available Artisan commands using the `list-artisan-commands` tool.
- If you're creating a generic PHP class, use `vendor/bin/sail artisan make:class`.
- Pass `--no-interaction` to all Artisan commands to ensure they work without user input. You should also pass the correct `--options` to ensure correct behavior.

### Database
- Always use proper Eloquent relationship methods with return type hints. Prefer relationship methods over raw queries or manual joins.
- Use Eloquent models and relationships before suggesting raw database queries.
- Avoid `DB::`; prefer `Model::query()`. Generate code that leverages Laravel's ORM capabilities rather than bypassing them.
- Generate code that prevents N+1 query problems by using eager loading.
- Use Laravel's query builder for very complex database operations.

### Model Creation
- When creating new models, create useful factories and seeders for them too. Ask the user if they need any other things, using `list-artisan-commands` to check the available options to `vendor/bin/sail artisan make:model`.

### APIs & Eloquent Resources
- For APIs, default to using Eloquent API Resources and API versioning unless existing API routes do not, then you should follow existing application convention.

### Controllers & Validation
- Always create Form Request classes for validation rather than inline validation in controllers. Include both validation rules and custom error messages.
- Check sibling Form Requests to see if the application uses array or string based validation rules.

### Queues
- Use queued jobs for time-consuming operations with the `ShouldQueue` interface.

### Authentication & Authorization
- Use Laravel's built-in authentication and authorization features (gates, policies, Sanctum, etc.).

### URL Generation
- When generating links to other pages, prefer named routes and the `route()` function.

### Configuration
- Use environment variables only in configuration files - never use the `env()` function directly outside of config files. Always use `config('app.name')`, not `env('APP_NAME')`.

### Testing
- When creating models for tests, use the factories for the models. Check if the factory has custom states that can be used before manually setting up the model.
- Faker: Use methods such as `$this->faker->word()` or `fake()->randomDigit()`. Follow existing conventions whether to use `$this->faker` or `fake()`.
- When creating tests, make use of `vendor/bin/sail artisan make:test [options] {name}` to create a feature test, and pass `--unit` to create a unit test. Most tests should be feature tests.

### Vite Error
- If you receive an "Illuminate\Foundation\ViteException: Unable to locate file in Vite manifest" error, you can run `vendor/bin/sail npm run build` or ask the user to run `vendor/bin/sail npm run dev` or `vendor/bin/sail composer run dev`.

=== laravel/v12 rules ===

## Laravel 12

- Use the `search-docs` tool to get version-specific documentation.
- Since Laravel 11, Laravel has a new streamlined file structure which this project uses.

### Laravel 12 Structure
- In Laravel 12, middleware are no longer registered in `app/Http/Kernel.php`.
- Middleware are configured declaratively in `bootstrap/app.php` using `Application::configure()->withMiddleware()`.
- `bootstrap/app.php` is the file to register middleware, exceptions, and routing files.
- `bootstrap/providers.php` contains application specific service providers.
- The `app\Console\Kernel.php` file no longer exists; use `bootstrap/app.php` or `routes/console.php` for console configuration.
- Console commands in `app/Console/Commands/` are automatically available and do not require manual registration.

### Database
- When modifying a column, the migration must include all of the attributes that were previously defined on the column. Otherwise, they will be dropped and lost.
- Laravel 12 allows limiting eagerly loaded records natively, without external packages: `$query->latest()->limit(10);`.

### Models
- Casts can and likely should be set in a `casts()` method on a model rather than the `$casts` property. Follow existing conventions from other models.

=== livewire/core rules ===

## Livewire

- Use the `search-docs` tool to find exact version-specific documentation for how to write Livewire and Livewire tests.
- Use the `vendor/bin/sail artisan make:livewire [Posts\CreatePost]` Artisan command to create new components.
- State should live on the server, with the UI reflecting it.
- All Livewire requests hit the Laravel backend; they're like regular HTTP requests. Always validate form data and run authorization checks in Livewire actions.

## Livewire Best Practices
- Livewire components require a single root element.
- Use `wire:loading` and `wire:dirty` for delightful loading states.
- Add `wire:key` in loops:

    ```blade
    @foreach ($items as $item)
        <div wire:key="item-{{ $item->id }}">
            {{ $item->name }}
        </div>
    @endforeach
    ```

- Prefer lifecycle hooks like `mount()`, `updatedFoo()` for initialization and reactive side effects:

<code-snippet name="Lifecycle Hook Examples" lang="php">
    public function mount(User $user) { $this->user = $user; }
    public function updatedSearch() { $this->resetPage(); }
</code-snippet>

## Testing Livewire

<code-snippet name="Example Livewire Component Test" lang="php">
    Livewire::test(Counter::class)
        ->assertSet('count', 0)
        ->call('increment')
        ->assertSet('count', 1)
        ->assertSee(1)
        ->assertStatus(200);
</code-snippet>

<code-snippet name="Testing Livewire Component Exists on Page" lang="php">
    $this->get('/posts/create')
    ->assertSeeLivewire(CreatePost::class);
</code-snippet>

=== livewire/v3 rules ===

## Livewire 3

### Key Changes From Livewire 2
- These things changed in Livewire 3, but may not have been updated in this application. Verify this application's setup to ensure you conform with application conventions.
    - Use `wire:model.live` for real-time updates, `wire:model` is now deferred by default.
    - Components now use the `App\Livewire` namespace (not `App\Http\Livewire`).
    - Use `$this->dispatch()` to dispatch events (not `emit` or `dispatchBrowserEvent`).
    - Use the `components.layouts.app` view as the typical layout path (not `layouts.app`).

### New Directives
- `wire:show`, `wire:transition`, `wire:cloak`, `wire:offline`, `wire:target` are available for use. Use the documentation to find usage examples.

### Alpine
- Alpine is now included with Livewire; don't manually include Alpine.js.
- Plugins included with Alpine: persist, intersect, collapse, and focus.

### Lifecycle Hooks
- You can listen for `livewire:init` to hook into Livewire initialization, and `fail.status === 419` for the page expiring:

<code-snippet name="Livewire Init Hook Example" lang="js">
document.addEventListener('livewire:init', function () {
    Livewire.hook('request', ({ fail }) => {
        if (fail && fail.status === 419) {
            alert('Your session expired');
        }
    });

    Livewire.hook('message.failed', (message, component) => {
        console.error(message);
    });
});
</code-snippet>

=== pint/core rules ===

## Laravel Pint Code Formatter

- You must run `vendor/bin/sail bin pint --dirty` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/sail bin pint --test`, simply run `vendor/bin/sail bin pint` to fix any formatting issues.

=== phpunit/core rules ===

## PHPUnit

- This application uses PHPUnit for testing. All tests must be written as PHPUnit classes. Use `vendor/bin/sail artisan make:test --phpunit {name}` to create a new test.
- If you see a test using "Pest", convert it to PHPUnit.
- Every time a test has been updated, run that singular test.
- When the tests relating to your feature are passing, ask the user if they would like to also run the entire test suite to make sure everything is still passing.
- Tests should test all of the happy paths, failure paths, and weird paths.
- You must not remove any tests or test files from the tests directory without approval. These are not temporary or helper files; these are core to the application.

### Running Tests
- Run the minimal number of tests, using an appropriate filter, before finalizing.
- To run all tests: `vendor/bin/sail artisan test --compact`.
- To run all tests in a file: `vendor/bin/sail artisan test --compact tests/Feature/ExampleTest.php`.
- To filter on a particular test name: `vendor/bin/sail artisan test --compact --filter=testName` (recommended after making a change to a related file).

=== tailwindcss/core rules ===

## Tailwind CSS

- Use Tailwind CSS classes to style HTML; check and use existing Tailwind conventions within the project before writing your own.
- Offer to extract repeated patterns into components that match the project's conventions (i.e. Blade, JSX, Vue, etc.).
- Think through class placement, order, priority, and defaults. Remove redundant classes, add classes to parent or child carefully to limit repetition, and group elements logically.
- You can use the `search-docs` tool to get exact examples from the official documentation when needed.

### Spacing
- When listing items, use gap utilities for spacing; don't use margins.

<code-snippet name="Valid Flex Gap Spacing Example" lang="html">
    <div class="flex gap-8">
        <div>Superior</div>
        <div>Michigan</div>
        <div>Erie</div>
    </div>
</code-snippet>

### Dark Mode
- If existing pages and components support dark mode, new pages and components must support dark mode in a similar way, typically using `dark:`.

=== tailwindcss/v4 rules ===

## Tailwind CSS 4

- Always use Tailwind CSS v4; do not use the deprecated utilities.
- `corePlugins` is not supported in Tailwind v4.
- In Tailwind v4, configuration is CSS-first using the `@theme` directive — no separate `tailwind.config.js` file is needed.

<code-snippet name="Extending Theme in CSS" lang="css">
@theme {
  --color-brand: oklch(0.72 0.11 178);
}
</code-snippet>

- In Tailwind v4, you import Tailwind using a regular CSS `@import` statement, not using the `@tailwind` directives used in v3:

<code-snippet name="Tailwind v4 Import Tailwind Diff" lang="diff">
   - @tailwind base;
   - @tailwind components;
   - @tailwind utilities;
   + @import "tailwindcss";
</code-snippet>

### Replaced Utilities
- Tailwind v4 removed deprecated utilities. Do not use the deprecated option; use the replacement.
- Opacity values are still numeric.

| Deprecated |	Replacement |
|------------+--------------|
| bg-opacity-* | bg-black/* |
| text-opacity-* | text-black/* |
| border-opacity-* | border-black/* |
| divide-opacity-* | divide-black/* |
| ring-opacity-* | ring-black/* |
| placeholder-opacity-* | placeholder-black/* |
| flex-shrink-* | shrink-* |
| flex-grow-* | grow-* |
| overflow-ellipsis | text-ellipsis |
| decoration-slice | box-decoration-slice |
| decoration-clone | box-decoration-clone |
</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pabloledesma)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Pabloledesma)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
