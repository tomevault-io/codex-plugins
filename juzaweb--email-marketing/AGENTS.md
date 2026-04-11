# Email Marketing Module - AI Coding Instructions

## Architecture Overview

This is a **Juzaweb modular plugin** for email marketing campaigns with Laravel Queue-based bulk sending, automation triggers, and comprehensive tracking. Key architectural patterns:

- **Module Structure**: Uses Juzaweb's modular architecture with `module.json` and specific namespace `Juzaweb\Modules\EmailMarketing`
- **Batch Processing**: Campaign sending uses Laravel's `Bus::batch()` for progress tracking and cancellation
- **Registry Pattern**: Automation triggers use singleton registry instead of hardcoded enums

## Key Components & Data Flow

### Campaign Execution Flow
1. `CampaignService::execute()` chunks subscribers by configurable batch size
2. Creates `CampaignBatch` records for progress tracking 
3. Dispatches `SendCampaignJob` instances using `Bus::batch()`
4. Each job sends via `DynamicCampaignMail` and records `CampaignTracking`

### Mail Content Processing
- `MailContentProcessor::transform()` adds tracking pixels, wraps links with click tracking, and appends unsubscribe footer
- Links become tracking URLs: `route('track.click', ['url' => encrypt($originalUrl), 'cid' => $campaign->id, 'sid' => $subscriber->id])`
- Tracking pixel: `route('track.open', [$campaign->id, $subscriber->id])`

### Automation System
- `AutomationTriggerRegistry` (singleton) manages extensible trigger types
- Service provider registers default triggers: `user_registered`, `user_birthday`, `member_registered`
- Use `AutomationTrigger::register($key, $config)` to add custom triggers
- Triggers support delay configuration and conditions

## Database Schema Patterns

- **UUIDs**: All primary keys use `uuid()` instead of auto-increment
- **Connection**: All migrations specify `protected $connection = 'mysql'`
- **Enums**: Status fields use PHP 8.1 backed enums (e.g., `CampaignStatusEnum::DRAFT`)
- **Pivot Tables**: Many-to-many relations use descriptive names like `email_segment_campaign`

## Development Conventions

### Model Patterns
```php
// Always extend Juzaweb base model with traits
class Campaign extends Model
{
    use HasAPI, Networkable; // Standard for all models
    
    protected $casts = [
        'status' => CampaignStatusEnum::class, // Use enum casting
        'automation_conditions' => 'array',   // JSON arrays
        'scheduled_at' => 'datetime',
    ];
}
```

### Service Layer
- Extend `BaseService` for common functionality
- Use `$this->transaction()` for database operations
- Return `$this->result($success, $data, $message)` format

### Queue Jobs
- Implement `ShouldQueue` with `Batchable` trait for campaign sending
- Always check `$this->batch()->cancelled()` for user-initiated stops
- Handle `TransportExceptionInterface` specifically for SMTP failures

### Controllers
- Extend `AdminController` from Juzaweb Admin module
- Use `Breadcrumb::add()` for navigation
- DataTable pattern: `SomeDataTable $dataTable` injection with `render()` method

## Configuration

Key configs in `config/email-marketing.php`:
- `batch_size`: Jobs per batch (default: 100)
- `default_delay_minutes`: Automation delays
- `max_retry_attempts`: Queue retry limits

## Testing & Debugging

### Queue Monitoring
```php
// Check batch progress
$batch = Bus::findBatch($batchId);
$batch->progress(); // Returns percentage
$batch->cancelled(); // Check if stopped
```

### Tracking Verification
- Open tracking: Check `CampaignTracking` records with `type = 'opened'`
- Click tracking: Links redirect through `TrackingController`
- Unsubscribe: Uses signed routes for security

## Routes & Permissions

- Admin routes: `Route::admin('email-marketing/campaigns', CampaignController::class)`
- Permissions follow pattern: `email-marketing.{resource}.{action}`
- Tracking routes: Public web routes for pixel/click tracking

## Extension Points

### Adding Custom Automation Triggers
```php
// In your module's service provider
$registry = app(AutomationTriggerRegistryInterface::class);
$registry->register('custom_trigger', [
    'label' => 'Custom Event',
    'description' => 'Triggers when...',
    'delay_support' => true,
]);
```

### Custom Mail Content Processing
Extend `MailContentProcessor` or create custom transformer and inject in `DynamicCampaignMail`.

## Critical Dependencies

- Laravel Queue system for batch processing
- Juzaweb Admin module for base models/controllers  
- Laravel Mail system with configurable mail drivers
- Database supports JSON column type for conditions/metadata

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juzaweb)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/juzaweb)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
