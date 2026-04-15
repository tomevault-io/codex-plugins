# AI Virtual Try-On - Copilot Instructions

## Project Overview
WordPress plugin providing AI-powered virtual try-on using Google Gemini 2.5 Flash Image API. Dual-mode architecture: WooCommerce modal integration (primary) and legacy shortcode for custom pages. Version 2.6.1+.

## Architecture Patterns

### Three-Tier Security Model
**CRITICAL**: Frontend NEVER communicates directly with Gemini API. All requests proxy through `avto-ajax-handler.php`:
```
[User Browser] → [WordPress AJAX] → [Gemini API]
```
- API key stored ONLY in `wp-config.php` via `AVTO_GEMINI_API_KEY` constant
- All API calls require nonce verification (`check_ajax_referer`)
- AJAX handler validates, sanitizes, and proxies requests

### Dual-Mode Operation
1. **WooCommerce Mode** (preferred): Modal overlay on product pages
   - Triggered by `avto-wc-tryon-trigger` button (see `avto-woocommerce.php`)
   - Uses product gallery images automatically
   - Data flow: `avto-frontend.js` → `avto_get_product_images_ajax()` → `avto_handle_generate_image_request()`

2. **Shortcode Mode** (legacy): `[ai_virtual_tryon]` on any page
   - Manual clothing catalog management via admin settings
   - See `avto-shortcode.php` for UI rendering

**Code Insight**: Check `$is_wc_mode` vs `$is_shortcode_mode` in `avto-ajax-handler.php:45-51` to understand dual handling.

### File Structure Logic
```
/includes/
├── avto-ajax-handler.php    # Single endpoint for both modes (line 44 mode detection)
├── avto-woocommerce.php      # Product page integration + modal HTML
├── avto-shortcode.php        # Legacy UI for custom pages
├── avto-admin.php            # Settings page with 5+ tabs
└── avto-my-account.php       # Try-On History (CPT + WC My Account endpoint)

/assets/
├── js/avto-frontend.js       # Dual-mode client logic (WC modal + shortcode)
└── css/avto-frontend.css     # Unified styling for both modes
```

### Custom Post Type: Try-On History
**CPT**: `avto_tryon_session` tracks user's generated images (since v2.3.0)
- Registered in `ai-virtual-try-on.php:224` (`avto_register_tryon_session_cpt()`)
- Created after successful generation in `avto-ajax-handler.php:306+`
- Displayed in WooCommerce My Account via custom endpoint: `try-on-history`
- **Note**: Requires `flush_rewrite_rules()` on activation (see `avto_activate()`)

## Development Workflows

### Testing Changes
```bash
# No build step required - direct PHP/JS/CSS edits
# Clear WordPress cache after changes:
wp cache flush

# Test both modes:
# 1. WooCommerce: Visit any product page with try-on button
# 2. Shortcode: Add [ai_virtual_tryon] to test page

# Check browser console for JS errors (F12)
# Enable debug mode: Settings → Virtual Try-On → Advanced → Debug Mode
```

### Adding New Settings
Follow WordPress Settings API pattern in `avto-admin.php`:
```php
register_setting( 'avto_settings_group', 'avto_new_setting', array(
    'type'              => 'string',
    'sanitize_callback' => 'sanitize_text_field',
    'default'           => 'default_value',
) );
```
**Always** use sanitization callbacks - never trust user input.

### Hook System
Use extensibility hooks (see `HOOKS-REFERENCE.md`):
- **Before API**: `avto_before_api_call` - validation, analytics
- **After Success**: `avto_after_generation_success` - notifications, tracking
- **Filters**: `avto_gemini_prompt` - customize AI prompt per product/user

Example:
```php
add_filter('avto_gemini_prompt', function($prompt, $user_img, $clothing_img, $product_id) {
    if ($product_id === 123) {
        return "Special prompt for product 123...";
    }
    return $prompt;
}, 10, 4);
```

## Critical Conventions

### WooCommerce Integration Toggle
**Check first**: `avto_wc_integration_enabled` option must be `true` (see `avto-woocommerce.php:72`)
- Hook location: `avto_wc_display_hook` (default: `woocommerce_single_product_summary`)
- Hook priority: `avto_wc_hook_priority` (default: `35`)
- Target categories: `avto_wc_target_categories` (empty = all products)

### Image Validation Pattern
**Two-stage validation** in `avto-ajax-handler.php:115-136`:
1. Client MIME type check (quick, unreliable)
2. `finfo_file()` content check (reliable, prevents AVIF/unsupported formats)

Allowed formats: JPEG, PNG, WebP, HEIC, HEIF (NOT AVIF - Gemini doesn't support)

### Rate Limiting
Dual rate limiting system (since v2.4.0):
- **Per-user**: `avto_check_rate_limit()` - default 10 requests/60 seconds
- **Global**: `avto_check_global_rate_limit()` - site-wide protection
- Uses WordPress transients for tracking

### Media Library Integration
All generated images saved via `media_handle_upload()` (line 160+):
```php
$attachment_id = media_handle_upload( 'user_image', 0 );
```
Never use direct file uploads - always integrate with WP Media Library for metadata, optimization, and backup compatibility.

## Common Pitfalls

### ❌ Don't hardcode plugin URL
```php
// BAD
$url = '/wp-content/plugins/ai-virtual-try-on/assets/...';

// GOOD
$url = AVTO_PLUGIN_URL . 'assets/...';
```

### ❌ Don't bypass nonce checks
All AJAX handlers must start with:
```php
check_ajax_referer( 'avto-generate-image-nonce', 'nonce' );
```

### ❌ Don't forget mode detection
When modifying `avto-ajax-handler.php`, handle both WooCommerce and shortcode modes:
```php
$is_wc_mode = ( $product_id > 0 && $clothing_image_id > 0 );
$is_shortcode_mode = ( ! empty( $clothing_id ) && ! empty( $clothing_file ) );
```

### ❌ Don't skip flush_rewrite_rules()
After adding/modifying CPTs or endpoints, flush rules in activation hook (see `avto_activate():102`).

## Quick Reference

### Key Constants
- `AVTO_VERSION` - Plugin version (increment on release)
- `AVTO_PLUGIN_DIR` - Absolute filesystem path
- `AVTO_PLUGIN_URL` - Public URL (for assets)
- `AVTO_GEMINI_API_KEY` - API key (must be in `wp-config.php`)

### Admin Settings Location
WordPress Admin → Settings → Virtual Try-On
- 5 tabs: General, WooCommerce, Clothing Items, UI Customization, Advanced

### Frontend Localization
Check `avto_enqueue_frontend_assets()` in `ai-virtual-try-on.php:270+`:
```php
wp_localize_script( 'avto-frontend-js', 'avtoData', array(
    'ajaxUrl'         => admin_url( 'admin-ajax.php' ),
    'generateNonce'   => wp_create_nonce( 'avto-generate-image-nonce' ),
    'productNonce'    => wp_create_nonce( 'avto-product-images-nonce' ),
    'maxFileSize'     => get_option( 'avto_max_file_size', 5 ) * 1024 * 1024,
) );
```

### Debugging
Enable debug mode: `update_option('avto_debug_mode', true)` or via admin panel.
Shows detailed Gemini API errors in AJAX responses.

## Version History Notes
- **v2.6.x**: Full-resolution lightbox with zoom controls
- **v2.4.x**: Global rate limiting feature
- **v2.3.x**: Try-On History CPT + My Account endpoint
- **v2.0+**: WooCommerce modal integration introduced
- **Pre-2.0**: Shortcode-only mode

## External Dependencies
- **Google Gemini 2.5 Flash Image API**: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent`
- **WooCommerce**: Optional, but required for modal mode (5.0+ tested to 9.0)
- **WordPress Media Library**: Required for image storage/management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opj161) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
