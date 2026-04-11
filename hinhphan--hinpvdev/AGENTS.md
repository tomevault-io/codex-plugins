# HinPV Portfolio & Blog - AI Coding Agent Instructions

## Project Overview
Laravel 12 portfolio and blog application with server-side rendering using Blade templates. Content is stored as Markdown in the database with pre-rendered HTML. Uses Tailwind CSS 4 and AnimeJS for styling and animations.

## Architecture & Key Patterns

### Content Management Philosophy
- **Markdown Storage**: Blog posts stored as Markdown in `posts.content` (longText field)
- **Pre-rendered HTML**: Content is processed with Shikijs (syntax highlighting) and KaTeX (math formulas) before database storage
- **SEO-First**: Separate `seo_metas` table with one-to-one relationship to posts for meta tags, OG images, canonical URLs
- **File Management**: `files` table tracks all assets (thumbnails, OG images) with path, mime, size, and disk location

### Data Model Relationships
```
posts (slug, title, content, excerpt, thumbnail_id, status, published_at)
  ├─→ files (thumbnail via thumbnail_id foreign key)
  ├─→ seo_metas (one-to-one via post_id)
  └─→ tags (many-to-many via post_tags pivot table)
```

**Status Constants**: Posts use `Post::STATUS['DRAFT']` (0) and `Post::STATUS['PUBLISHED']` (1)

### Component Architecture
- **Class-based Components**: All Blade components have PHP classes in `app/View/Components/` with PascalCase directory structure (e.g., `Layouts/Master.php` → `<x-layouts.master>`)
- **Component Organization**: Nested by category: `Buttons/`, `Cards/`, `Inputs/`, `Labels/`, `Layouts/`, `Misc/`, `Parts/`, `Posts/`, `Sections/`
- **Master Layout**: `resources/views/components/layouts/master.blade.php` with slots for `title` and main content, includes header/footer components

### Routing Pattern
Routes use closure-based rendering (`return view('posts.index')`) instead of controller actions for public pages. Admin routes use controllers and `auth` middleware.

```php
// Public pages: direct view rendering
Route::get('/posts', fn() => view('posts.index'));

// Admin: controller + middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
});
```

### Authentication
- Simple email/password auth with "remember me" functionality
- Login handled via `LoginRequest` form request validation
- Session regeneration on login/logout for security
- Protected routes redirect to `/dashboard` on success

## Development Workflow

### Starting Development
```powershell
# Single command starts server, queue worker, and Vite dev server
composer dev
# Equivalent to: php artisan serve & queue:listen & npm run dev
```

### Testing
```powershell
# Run Pest tests (PHPUnit wrapper)
composer test
# Equivalent to: php artisan config:clear && php artisan test
```

### Database Operations
```powershell
php artisan migrate              # Run migrations
php artisan migrate:fresh        # Drop all tables and re-migrate
php artisan db:seed              # Run seeders (currently empty)
```

### Building Assets
```powershell
npm run build    # Production build with Vite + Tailwind CSS 4
npm run dev      # Development with hot reload
```

## Project-Specific Conventions

### Models
- Use `HasFactory` trait with typed factory annotations: `/** @use HasFactory<\Database\Factories\PostFactory> */`
- Status constants defined as public class constants (see `Post::STATUS`)
- Models currently lack relationship methods - add `belongsTo()`, `hasMany()`, `belongsToMany()` as needed

### Policies
- Policies exist for Post, PostTag, SeoMeta, Tag but all methods return `false` (authorization not implemented)
- When implementing: policies follow standard Laravel Gate naming (`view`, `create`, `update`, `delete`, etc.)

### Frontend
- **Tailwind CSS 4**: Uses new `@tailwindcss/vite` plugin, not PostCSS
- **Vite Config**: Inputs are `resources/css/app.css` and `resources/js/app.js`
- **AnimeJS**: For animations (not currently visible in codebase, likely in views)
- **Axios**: Configured globally on `window.axios` with X-Requested-With header

### Migrations
- Relationships added in separate migration: `2025_10_10_160303_add_relationship.php`
- Foreign keys use `onDelete('cascade')` for posts/tags, `onDelete('set null')` for optional images

## Future Considerations (from NOTE.md)
- **Editor**: Considering ToastUI Editor for post creation
- **Image Storage**: Plan to use Cloudinary (bandwidth costs - optimize + lazy loading)
- **Content Processing**: Shikijs for code preview, rehype-katex/remark-math/katex for LaTeX math

## Key Files to Reference
- `routes/web.php` - All route definitions with closure pattern
- `app/Models/Post.php` - Core content model with status constants
- `database/migrations/2025_10_10_160303_add_relationship.php` - Foreign key relationships
- `resources/views/components/layouts/master.blade.php` - Base layout template
- `composer.json` scripts section - Custom dev and test commands

## What's NOT Here Yet
- Eloquent relationships not defined on models (pivot methods, belongsTo, etc.)
- Seeders are empty stubs
- Factories have no definitions
- Policy authorization logic not implemented
- Admin CRUD functionality (controllers have empty methods)

## Luồng xử lý code và tự động test, đánh giá các màn hình như sau

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hinhphan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/hinhphan)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
