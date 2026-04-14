# Sistem Informasi Pupuk & Bibit Subsidi - AI Coding Guide

## Project Overview

Laravel 12 application for managing subsidized fertilizer and seed products. Dual authentication system: standard users (Laravel Auth) + hardcoded admin (session-based). Features include product CRUD with multi-image uploads, user registration, and admin dashboard.

## Critical Architecture Patterns

### 1. Dual Authentication System

**Standard Users:** Laravel's built-in authentication
- Controller: `AuthController` 
- Uses `Auth::attempt()` and `auth` middleware
- Database-backed via `users` table

**Admin Access:** Session-based hardcoded credentials
- Controller: `AdminController`
- Credentials: `admin` / `admin123` (constants in controller)
- Custom middleware: `admin.auth` (checks `session('admin_logged_in')`)
- Routes prefixed with `/admin`

**Never mix these auth systems!** Admin routes use `admin.auth` middleware, user routes use `auth` middleware.

### 2. Product Multi-Image Architecture

Products support 1-5 images using a **relational pattern** (not array field):

```php
// produk table: id_produk (integer PK, NOT unsigned)
// product_images table: product_id references produk.id_produk
Product::images()       // hasMany ProductImage, ordered by 'order'
Product::primaryImage() // hasOne ProductImage where is_primary = true
```

**Key conventions:**
- First uploaded image is always `is_primary = true`
- Field `Product.gambar` stores primary image path (backward compatibility)
- Images stored in `public/images/products/` as `{timestamp}_{uniqid}_{index}.{ext}`
- Delete operations cascade via foreign key + manual file deletion
- **Always wrap image operations in DB transactions** (see `ProductController::store()`)

### 3. Auto-Fill Business Logic

When `tipe_produk = 'bibit'`, category **must** be set to "Organik":

```php
if ($validated['tipe_produk'] === 'bibit') {
    $validated['kategori'] = 'Organik'; // Enforced in controller
}
```

JavaScript auto-fills and makes field readonly on frontend, but server always enforces this rule.

### 4. Price Validation Pattern

Business rule enforced in controller (not database constraint):

```php
if ($validated['harga_subsidi'] >= $validated['harga_normal']) {
    return back()->withInput()->withErrors([...]);
}
```

Always check subsidy price < normal price before saving products.

## Database Schema Quirks

1. **Non-standard primary key:** `produk` table uses `id_produk` (integer), NOT `id` (bigIncrements)
   - Affects foreign keys: `product_images.product_id` is integer, not unsignedBigInteger
   - Model must set `protected $primaryKey = 'id_produk'`

2. **Session driver:** Database-backed (not file) - requires `sessions` table migration

3. **Expected database name:** `sistem_informasi_pupukdanbibit` (see `.env.example`)

## Essential Workflows

### Development Server

```powershell
php artisan serve  # http://127.0.0.1:8000
npm run dev        # Vite for Tailwind CSS
```

**Or use composer script (runs all services concurrently):**
```powershell
composer run dev  # Starts server + queue + logs + vite
```

### Migrations

```powershell
php artisan migrate         # Run migrations
php artisan migrate:fresh   # Drop all tables and re-migrate (caution!)
```

### Testing Admin Login

1. Navigate to `/admin/login`
2. Use `admin` / `admin123`
3. Test logout clears session: `session()->forget(['admin_logged_in', ...])`

### Adding Products

- Form validation messages are in **Bahasa Indonesia**
- Image upload limit: 1-5 images, max 2MB each, formats: jpeg/png/jpg/gif
- Client-side validation in JavaScript must match server-side rules
- Preview images before upload (see `products/create.blade.php`)

## View Organization

```
resources/views/
├── auth/           # login, register, admin-login
├── admin/          # dashboard (protected by admin.auth)
├── products/       # index, create, edit (CRUD views)
├── layouts/        # Shared layouts
├── dashboard.blade.php  # User dashboard
└── pupukdanbibit.blade.php # Public product listing
```

**Note:** `poducts/` directory is a typo - use `products/`

## Route Patterns

```php
// Public
Route::get('/', 'home')
Route::get('/pupuk-bibit', 'product listing')

// User auth (Laravel Auth)
Route::post('/login')
Route::get('/dashboard')->middleware('auth')

// Admin (Session-based)
Route::prefix('admin')->group(...)
  ->middleware('admin.auth')  // Custom middleware

// Products (resourceful)
Route::resource('products', ProductController::class)
```

## File Upload Conventions

- **Destination:** `public/images/products/`
- **Naming:** `{timestamp}_{uniqid}_{index}.{extension}`
- **Always check file exists before unlinking:**
  ```php
  if (file_exists(public_path($path))) {
      unlink(public_path($path));
  }
  ```
- **Transaction pattern:** Wrap uploads + DB saves in `DB::transaction()`

## Error Messages

All validation messages are in **Bahasa Indonesia**. When adding new validations, follow this pattern:

```php
$request->validate([...], [
    'field.required' => 'Field wajib diisi',
    'field.numeric' => 'Field harus berupa angka',
]);
```

## Common Pitfalls

1. **Don't use `id` for products** - it's `id_produk` everywhere
2. **Admin vs User auth confusion** - check which middleware is needed
3. **Foreign key type mismatch** - `product_id` is integer, not unsignedBigInteger
4. **Forgetting DB transactions** - wrap file uploads + DB operations together
5. **Category auto-fill** - must enforce "Organik" server-side for bibit
6. **Image cleanup on failure** - rollback DB but files may remain if not in transaction handler

## Testing Scenarios

1. **Upload edge case:** Select 6 images (should fail validation)
2. **Price validation:** Try subsidy price > normal price
3. **Admin session:** Logout and try accessing `/admin/dashboard` directly
4. **Cascade delete:** Delete product and verify all images removed from filesystem
5. **Auto-fill:** Change product type to 'bibit' and verify category becomes readonly "Organik"

## Documentation References

- `README_LOGIN_ADMIN.md` - Admin auth system details
- `README_PRODUK.md` - Single product CRUD with auto-fill
- `README_MULTIPLE_UPLOAD.md` - Multi-image upload architecture
- `README_USAGE.md` - Empty (placeholder)

## Tech Stack

- **Framework:** Laravel 12 (PHP 8.2+)
- **Frontend:** Blade templates + Tailwind CSS 4 + Vite
- **Database:** MySQL (Laragon default)
- **Session:** Database driver (not file-based)
- **Assets:** Font Awesome for icons (check views for CDN links)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OloanNainggolan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
