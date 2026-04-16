# GitHub Copilot Instructions for Viavi 8800SX Project

## Project Overview
This is a Docker Compose project that processes output files from a Viavi 8800SX service monitor, parses them, and stores the data in a MySQL database. The application stores test records with full BLOB storage of original files and provides a web interface for searching and managing data.

## Technology Stack
- **Backend**: PHP 8.3.2-FPM
- **Database**: MySQL
- **Web Server**: Nginx (via Docker Compose)
- **Frontend**: Bootstrap 4.5.0, jQuery 3.5.1
- **Containerization**: Docker & Docker Compose
- **Code Quality**: PHP CS Fixer (PSR-12), YAML linting, Markdown linting

## Configuration System

### Current Architecture
The project uses a **traditional PHP configuration** approach with simple variable assignments in `config.php`:

```php
// Database configuration
$db_server = 'db';
$db_name = 'viavi';
$db_user = 'viavi';
$db_password = trim(file_get_contents($passwordFile));
$link = mysqli_connect($db_server, $db_user, $db_password, $db_name);

// Application configuration
$appname = '8800SX';
$language = 'en';
$no_of_records_per_page = 10;
$protocol = (isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] == 'on') ? 'https' : 'http';
$domain = $protocol . '://' . ($_SERVER['HTTP_HOST'] ?? 'localhost');

// Upload configuration
$upload_max_size = 5000000;
$upload_target_dir = "uploads/";
$upload_disallowed_exts = array(/* ... */);

// Translations
$translations = include($localeFile);
```

**Important**: 
- All configuration variables are defined directly in `config.php`
- No Config class or OOP approach
- Helper functions use `global` keyword to access configuration
- Database connection `$link` is created directly in config.php

## Code Style and Conventions

### PHP Guidelines
1. **Security First**: Always use prepared statements for database queries
2. **Input Validation**: Validate and sanitize all user inputs
3. **File Uploads**: 
   - Always validate file types using MIME types, not just extensions
   - Check file sizes against configured limits
   - Use the `handleFileUpload()` function in helpers.php
4. **Error Handling**: Use try-catch blocks and log errors appropriately
5. **Documentation**: Add PHPDoc comments for functions
6. **Code Formatting**: Follow PSR-12 standards (auto-formatted by GitHub Actions)

### Database Interactions
1. Always use parameterized queries (prepared statements)
2. Never concatenate user input into SQL queries
3. Use appropriate data types in the schema
4. Handle database connection errors gracefully
5. Access database via `$link` variable from config.php

### Global Variables Usage
Functions that need configuration values should use the `global` keyword:

```php
function myFunction() {
    global $link, $upload_target_dir, $translations;
    // Use the variables
}
```

### File Structure
```
/data/web/
├── nginx.conf         # Nginx configuration
└── app/
    ├── config.php     # Configuration variables only (no classes)
    ├── helpers.php    # Helper functions (uses global variables)
    ├── main.php       # File upload handler
    ├── upload.php     # Upload UI
    ├── alignments-*.php  # CRUD operations
    ├── locales/       # Internationalization files
    └── security-headers.php  # Security headers
/data/db/init/
└── init-db.sql        # Database initialization script
```

## Security Considerations

### Critical Security Rules
1. **Never hardcode credentials** - Use environment variables and file-based secrets
2. **Validate all inputs** - Both client and server-side
3. **Use prepared statements** - Always for database queries
4. **Implement rate limiting** - Prevent abuse
5. **Keep dependencies updated** - Regular security patches
6. **Use HTTPS** - In production environments
7. **Set security headers** - CSP, X-Frame-Options, etc.

### File Upload Security
- Validate MIME types server-side
- Restrict file extensions using allowlist
- Set maximum file size limits
- Scan uploaded files for malware if possible
- Store files with unique names to prevent overwrites

## Development Workflow

### Before Making Changes
1. Review existing code patterns
2. Check for similar implementations
3. Consider security implications
4. Update documentation if needed

### Testing
1. Test file uploads with various file types
2. Verify database operations don't have SQL injection vulnerabilities
3. Test with invalid inputs to ensure proper error handling
4. Check that uploaded files are processed correctly

### Docker Commands
```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Rebuild after changes
docker compose down
docker compose build --no-cache
docker compose up -d

# Access PHP container
docker compose exec web bash
```

## Common Patterns

### Database Query Pattern
```php
// Correct - Using prepared statements
$stmt = $connection->prepare("SELECT * FROM alignments WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$result = $stmt->get_result();
```

### Input Validation Pattern
```php
// Validate and sanitize input
$filename = basename($_FILES['file']['name']);
$path_info = pathinfo($filename);
$extension = strtolower($path_info['extension']);

// Check against allowlist
$allowed_extensions = ['txt'];
if (!in_array($extension, $allowed_extensions)) {
    throw new Exception("Invalid file type");
}

// Validate MIME type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);

if ($mime_type !== 'text/plain') {
    throw new Exception("Invalid file format");
}
```

### Error Handling Pattern
```php
try {
    // Database operation
    $stmt = $connection->prepare($sql);
    if (!$stmt) {
        throw new Exception("Database error: " . $connection->error);
    }
    // ... execute query
} catch (Exception $e) {
    error_log($e->getMessage());
    // Show user-friendly message
    echo "An error occurred. Please try again.";
}
```

## Internationalization (i18n)
- Use the `translate()` helper function for all user-facing text
- Add translations to appropriate locale files in `/data/web/app/locales/`
- Support languages: en, es, fr, de, it, pt, nl, ru, cn, jp, in, cz

## Environment Variables
- `DB_PASSWORD_FILE`: Path to file containing database password
- Configure in `docker-compose.yml`

## Debugging Tips
1. Check Docker logs: `docker compose logs web`
2. Check PHP errors in container: `docker compose exec web tail -f /var/log/php-errors.log`
3. Enable error reporting in development (disable in production)
4. Use `var_dump()` and `error_log()` for debugging

## Performance Considerations
1. Limit database queries in loops
2. Use appropriate indexes on database tables
3. Optimize file upload sizes (currently limited to 128MB)
4. Consider caching for frequently accessed data

## Code Review Checklist
- [ ] SQL queries use prepared statements
- [ ] User inputs are validated and sanitized
- [ ] File uploads are properly validated
- [ ] Errors are handled gracefully
- [ ] Security best practices are followed
- [ ] Code is documented
- [ ] No sensitive data is hardcoded
- [ ] Changes are backwards compatible

## GitHub Actions Workflows

The project includes automated workflows for code quality and versioning:

### Code Formatter (`code-formatter.yml`)
- **Purpose**: Automatically format PHP code using php-cs-fixer with PSR-12 standards
- **Triggers**: Manual (Actions tab) or automatically on pull requests
- **Actions**: 
  - Formats all PHP files in `data/web/app/`
  - Commits formatted code back to the branch
  - Comments on PRs when formatting is applied
- **Usage**: Actions → Code Formatter → Run workflow

### Code Quality (`code-quality.yml`)
- **Purpose**: Check code quality without making changes
- **Triggers**: Push to `main` or pull requests to `main`
- **Checks**:
  - PHP syntax validation
  - PHP code style (PSR-12)
  - YAML validation
  - Markdown linting
  - Dockerfile linting

### Semantic Versioning (`semantic-versioning.yml`)
- **Purpose**: Automatically update version numbers and create releases
- **Triggers**: When pull requests are merged to `main`
- **Actions**:
  - Reads version from `version.txt`
  - Bumps version based on PR labels (major, minor, patch)
  - Updates `version.txt`
  - Updates `CHANGELOG.md`:
    - Creates a new version section with date: `## [X.Y.Z] - YYYY-MM-DD`
    - Adds PR title and description as the changelog entry
    - Updates version comparison links at the end of the file
  - Updates `README.md` version badge
  - Creates GitHub release with PR title and description
- **PR Labels**: Apply one of these labels to control version bumping:
  - `major` - Breaking changes (X.0.0)
  - `minor` - New features (0.X.0)
  - `patch` - Bug fixes (0.0.X)
  - Default: If no label is provided, defaults to `patch` version bump
- **Important**: Write clear, descriptive PR titles and descriptions as they become the changelog and release notes

## Documentation Structure

Keep documentation minimal and focused:
- **README.md** - Project overview, installation, usage
- **CHANGELOG.md** - Version history and changes
- **SECURITY.md** - Security policy and guidelines
- **LICENSE** - MIT license
- **INSTALL.md** - Installation guide
- **.github/copilot-instructions.md** - Development guidelines (this file)

## Additional Resources
- [PHP Security Best Practices](https://www.php.net/manual/en/security.php)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [PSR-12 Coding Style](https://www.php-fig.org/psr/psr-12/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k9barry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
