# MyLaboAccess - AI Coding Assistant Instructions

## Project Overview
MyLaboAccess is a **Flutter + PHP/MySQL** laboratory equipment management system with role-based access (Guest/User/Admin). Users request equipment loans; admins manage inventory, users, and approve/reject requests.

## Architecture & Data Flow

### Three-Tier Client-Server
```
Flutter App (Dart) ←→ PHP REST API ←→ MySQL Database
```

**Critical Pattern**: All API calls flow through `ApiService` (singleton pattern) in [lib/services/api_service.dart](lib/services/api_service.dart). **Never** make direct HTTP calls from pages—always add methods to `ApiService`.

### Key Components
- **Frontend**: [lib/pages/](lib/pages/) - UI pages with `StatefulWidget` pattern
- **Backend**: [server-samples/mylabo_api/](server-samples/mylabo_api/) - PHP endpoints (no framework)
- **Data Models**: [lib/models/user.dart](lib/models/user.dart) - Simple Dart classes
- **API Layer**: [lib/services/api_service.dart](lib/services/api_service.dart) - Centralized HTTP client

## Critical Configuration

### API URL Switching
[lib/config.dart](lib/config.dart) controls backend connectivity:
```dart
const String apiBaseUrl = 'http://localhost/mylabo_api';    // Web/Windows
// const String apiBaseUrl = 'http://10.0.2.2/mylabo_api';  // Android Emulator
```

**Why**: Android emulator uses `10.0.2.2` to reach host machine's localhost. Physical devices require LAN IP (`192.168.x.x`).

### API Fallback Strategy
`ApiService._postWithFallback()` tries multiple URL variants (`127.0.0.1`, `localhost`, `10.0.2.2`) to handle different environments. When adding endpoints, use this pattern.

## Developer Workflows

### Running the App
```bash
# Prerequisites: Laragon/XAMPP running with PHP + MySQL
flutter pub get                    # Install dependencies
flutter run -d chrome              # Web (preferred for testing)
flutter run -d android             # Android emulator/device
flutter run -d windows             # Windows desktop
```

### Web Deployment (GitHub Pages)
```bash
flutter build web --release --web-renderer html --output=docs
# Commit docs/ folder → Enable GitHub Pages from /docs in Settings
```

### Database Setup
1. Import schema: `mysql -u root -p mylaboipi < server-samples/mylabo_api/setup_borrow_requests.sql`
2. Verify DB credentials in **all** PHP files: `$dbHost`, `$dbName`, `$dbUser`, `$dbPass`

## Project-Specific Conventions

### Backend PHP Pattern
Each endpoint follows this structure (no framework):
```php
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');  // Required for Flutter Web CORS

$input = json_decode(file_get_contents('php://input'), true);
// Validation...
// PDO prepared statements (no raw SQL)
echo json_encode(['success' => true|false, 'message' => '...', 'data' => []]);
```

### State Management (Flutter)
- **No external state library** (no Provider/Bloc/Riverpod)
- Uses vanilla `StatefulWidget` with `setState()`
- Admin panel auto-refreshes via `Timer.periodic()` every 5 seconds ([admin_panel.dart#L35](lib/pages/admin_panel.dart#L35))

### Role-Based Navigation
Routing logic in [main.dart](lib/main.dart):
- `/` → Login page (entry point)
- `/home` → User dashboard (equipment browsing + borrow history)
- `/admin` → Admin panel (4 tabs: Users, Equipment, Borrow Requests, Reports)
- Navigation via `Navigator.pushNamed(context, '/route')`

### User Roles
Defined in [models/user.dart](lib/models/user.dart):
```dart
enum UserRole { invite, utilisateur, admin }
```
Backend stores as string (`'invite'`, `'utilisateur'`, `'admin'`) in `users.role` column.

## Common Patterns

### API Call Pattern
```dart
// In ApiService
static Future<Map<String, dynamic>> methodName(params) async {
  return await _postWithFallback('/endpoint.php', {'key': value});
}

// In page widget
final result = await ApiService.methodName(params);
if (result['success'] == true) {
  setState(() { /* update UI */ });
}
```

### Admin Panel Refresh Loop
[admin_panel.dart](lib/pages/admin_panel.dart) uses `Timer` to poll for new borrow requests:
```dart
_refreshTimer = Timer.periodic(const Duration(seconds: 5), (_) {
  _loadBorrowRequests();  // Fetches from get_all_borrow_requests.php
});
```

### Password Security
- **Frontend**: Plain text sent over HTTP (not production-ready)
- **Backend**: Uses `password_hash()` and `password_verify()` (bcrypt)
- All DB inserts use PDO prepared statements to prevent SQL injection

## Database Schema

### Key Tables
- `users`: `id`, `email`, `nom`, `role`, `password_hash`, `created_at`
- `equipment`: `id`, `name`, `quantity` (total), `available` (current), `description`
- `borrow_requests`: `id`, `user_email`, `equipment_name`, `quantity`, `status` (`'En attente'`/`'Approuvé'`/`'Rejeté'`)
- `reports`: `id`, `user_email`, `equipment_name`, `description`, `created_at`

### Data Integrity
- **No foreign keys** in current schema (uses `user_email` strings, not user IDs)
- Equipment availability calculated dynamically: `available = quantity - borrowed_count`

## Testing & Debugging

### No Automated Tests
- [test/widget_test.dart](test/widget_test.dart) is Flutter default boilerplate (not actively used)
- Testing is manual via `flutter run`

### Common Issues
1. **"Cannot connect to server"**: Check Laragon/Apache is running, verify `apiBaseUrl` in [config.dart](lib/config.dart)
2. **CORS errors (Web)**: Ensure all PHP files have `Access-Control-Allow-Origin: *` header
3. **Empty data lists**: Check PHP endpoint returns `{'success': true, 'data': [...]}`
4. **Red squiggles in VS Code**: Run `flutter pub get` + restart Dart analysis server

## When Adding Features

### New API Endpoint
1. Create PHP file in [server-samples/mylabo_api/](server-samples/mylabo_api/) (copy existing for template)
2. Add method to `ApiService` using `_postWithFallback()` or GET variant
3. Call from page widget, handle `result['success']` response

### New Page
1. Create in [lib/pages/](lib/pages/) as `StatefulWidget`
2. Register route in [main.dart](lib/main.dart) `routes` map
3. Navigate via `Navigator.pushNamed(context, '/route')`

### Modifying UI
- Theme colors defined in [main.dart](lib/main.dart) `ThemeData` (red accent on white)
- Use `ElevatedButton`, `TextField`, `Card` widgets consistently
- Tab navigation via `TabController` (see [main.dart#L69](lib/main.dart#L69) and [admin_panel.dart](lib/pages/admin_panel.dart))

## Dependencies
- **http** (`^1.1.0`): Only external package (for API calls)
- **cupertino_icons** (`^1.0.8`): iOS-style icons
- **flutter_launcher_icons** (dev): App icon generator

## Out of Scope
- ❌ No offline mode / local storage
- ❌ No push notifications
- ❌ No JWT authentication (sessions are stateless)
- ❌ No HTTPS in dev environment (use Laragon's SSL for production)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redadiouri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
