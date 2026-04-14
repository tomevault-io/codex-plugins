# CoreVsPhim AI Coding Agent Instructions

## Project Architecture Overview

**CoreVsPhim** is a Vietnamese movie streaming platform built on **Laravel 8.x + OphimCMS + Backpack CRUD**. The architecture follows a package-extended approach where core functionality comes from vendor packages that are extended rather than modified directly.

### Key Architecture Principles
- **Vendor Extension Pattern**: Extend vendor classes in `app/` instead of modifying `vendor/` files
- **CMS-First Design**: OphimCMS provides core movie management, Backpack handles admin interfaces
- **Model Extension**: Core models like `Episode` extend vendor base classes (e.g., `Ophim\Core\Models\Episode`)
- **Theme Integration**: MotChill theme provides frontend, custom features added via dedicated controllers

## Critical Development Patterns

### 1. Model Extension Pattern
```php
// Always extend vendor models, never modify them directly
class Episode extends BaseEpisode
{
    // Add new fields to fillable
    protected $fillable = ['name', 'slug', 'is_short', 'hashtags', 'likes'];
    
    // Add custom relationships and scopes
    public function scopeShorts(Builder $query) {
        return $query->where('is_short', true);
    }
}
```

### 2. Backpack CRUD Controller Pattern
```php
// Standard CRUD setup with field validation
public function setup()
{
    CRUD::setModel(ModelName::class);
    CRUD::setRoute(config('backpack.base.route_prefix') . '/route-name');
    
    // ALWAYS validate fields before adding
    foreach (Option::getAllOptions() as $key => $field) {
        if (!isset($field['name']) || empty($field['name'])) {
            \Log::error('Field missing name key', ['key' => $key]);
            continue;
        }
        CRUD::addField($field);
    }
}
```

### 3. Migration Strategy for Feature Extensions
- New features extend existing tables (e.g., shorts added to `episodes` table)
- Use migrations like `add_shorts_fields_to_episodes_table.php`
- Create junction tables for interactions: `episode_interactions`, `short_comments`

## Essential Workflows

### Local Development Setup
```bash
# Standard Laravel setup
cp .env.example .env
composer install
php artisan key:generate
php artisan migrate

# OphimCMS specific
php artisan ophim:user          # Create admin user
php artisan storage:link
php artisan ophim:menu:generate # Generate CMS menus
```

### Docker Development (Recommended)
```bash
cd deploy/
docker-compose up -d
# Access: http://localhost:8080, phpMyAdmin: http://localhost:8081
```

### Adding New Features (Shorts Feature Example)
1. **Extend existing models** instead of creating new ones
2. **Add fields via migrations** to existing tables
3. **Create dedicated controllers** following Backpack patterns
4. **Use Blade components** for reusable frontend elements

## Database Architecture

### Core Tables
- `movies`: Main content (from OphimCMS)
- `episodes`: Video episodes (extended for shorts functionality)
- `categories`, `regions`: Taxonomy (from OphimCMS)
- `episode_interactions`: User interactions (likes, views, shares)

### Extension Pattern
```sql
-- Extend existing tables instead of creating new ones
ALTER TABLE episodes ADD COLUMN is_short BOOLEAN DEFAULT FALSE;
ALTER TABLE episodes ADD COLUMN hashtags JSON NULL;
```

## Frontend Patterns

### Blade Template Structure
```
resources/views/
├── shorts/
│   ├── layout.blade.php     # Base layout with shared CSS/JS
│   ├── index.blade.php      # Main shorts feed with infinite scroll
│   ├── show.blade.php       # Individual short view
│   └── feed.blade.php       # AJAX-loaded content component
```

### JavaScript Architecture
- **Component-based**: Each feature has its own class (e.g., `ShortsPlayer`)
- **AJAX-heavy**: Infinite scroll, interactions, comments all via AJAX
- **Progressive Enhancement**: Works without JS, enhanced with JS

## Integration Points

### OphimCMS Integration
- Core models: `vendor/hacoidev/ophim-core/src/Models/`
- Crawler: `vendor/hacoidev/ophim-crawler/src/`
- Never modify vendor files - extend in `app/Models/`

### Theme Integration
- Base theme: `vendor/ophimcms/theme-motchill/`
- Custom views in `resources/views/`
- Override theme views by matching directory structure

### External Dependencies
- **Guzzle**: HTTP client for API calls and crawling
- **Laravel Sanctum**: API authentication
- **Backpack**: Admin panel framework

## Code Quality Standards

### Error Handling
```php
// Always log with context, use appropriate levels
try {
    // Operation
} catch (Exception $e) {
    \Log::error('Operation failed', [
        'error' => $e->getMessage(),
        'context' => $relevantData
    ]);
    return response()->json(['error' => 'User-friendly message'], 500);
}
```

### Field Validation Pattern
```php
// Check field existence before processing
foreach ($options as $k => $v) {
    if (array_key_exists($k, $fields)) {
        $fields[$k]['value'] = $v;
    } else {
        \Log::warning('Field exists in DB but not in definitions', [
            'field_name' => $k, 'value' => $v
        ]);
    }
}
```

## Testing and Debugging

### Key Artisan Commands
```bash
php artisan ophim:menu:generate    # Regenerate CMS menus
php artisan optimize:clear         # Clear all caches
php artisan migrate:status         # Check migration status
php artisan tinker                 # Test models and relationships
```

### Common Debug Points
- Check `storage/logs/laravel.log` for application errors
- Verify model relationships with `php artisan tinker`
- Use `CRUD::addField()` validation for Backpack forms
- Check route definitions for proper model binding

### Database Access
```bash
# Docker setup
docker-compose exec db mysql -u root -proot_password corevsphim

# Local setup  
mysql -u user -p database_name
```

Remember: Always extend vendor packages rather than modifying them directly, use proper error logging with context, and follow the established patterns for model extensions and CRUD controllers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huynq11019) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
