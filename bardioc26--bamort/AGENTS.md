# Bamort Development Instructions

Bamort is a role-playing game character management system (MOAM replacement) with a Go backend and Vue.js frontend.

## Project Overview

- **Repository**: github.com:Bardioc26/bamort.git
- **Architecture**: Monorepo with separate backend/frontend in Docker containers
- **Backend**: Go 1.25 + Gin framework + GORM + MariaDB
- **Frontend**: Vue 3 + Vite + i18n
- **Development**: Docker Compose with live-reload (Air for Go, Vite HMR for Vue)

## Backend Architecture (`backend/`)

### Module Structure
Each domain module follows this pattern (e.g., `character/`, `pdfrender/`, `equipment/`):
```
module/
  handlers.go      # HTTP handlers (Gin controllers)
  routes.go        # Route registration: RegisterRoutes(r *gin.RouterGroup)
  *_test.go        # Tests with setupTestEnvironment(t)
```

**Key conventions:**
- **Entry point**: `cmd/main.go` registers all modules via `RegisterRoutes(protected)`
- **Models**: `models/` contains GORM entities (e.g., `Char`, `SkFertigkeit`, `EqWaffe`)
- **Database**: Shared `database.DB` instance, use `models.MigrateStructure(db)` for migrations
- **Configuration**: `config.Cfg` loaded from env vars (see `config/config.go`)
  - `TEMPLATES_DIR` for PDF templates (default: `./templates`)
  - `ENVIRONMENT=test|development|production`
  - `DATABASE_TYPE=mysql|sqlite`

### Testing Requirements
- **NEVER** create test files with `main()` functions
- **ALWAYS** use `_test.go` suffix
- **ALWAYS** call `setupTestEnvironment(t)` at start of each test:
  ```go
  func setupTestEnvironment(t *testing.T) {
      original := os.Getenv("ENVIRONMENT")
      os.Setenv("ENVIRONMENT", "test")
      t.Cleanup(func() { /* restore */ })
  }
  ```
- Use `testutils.SetupTestDB()` for database tests (creates SQLite test DB)
- Test character ID 18 (Fanjo Vetrani) exists in test database

### API Patterns
- Protected routes under `/api` prefix require JWT authentication
- Use `respondWithError(c, status, message)` for error responses
- Route registration example:
  ```go
  func RegisterRoutes(r *gin.RouterGroup) {
      group := r.Group("/module")
      group.GET("/list", ListHandler)
      group.POST("/create", CreateHandler)
  }
  ```

## Frontend Architecture (`frontend/`)

### Component Structure
- **Views**: `src/views/` - Page-level components (CharacterView, DashboardView)
- **Components**: `src/components/` - Reusable components (CharacterDetails, SkillView)
- **Utils**: `src/utils/api.js` - Axios instance with JWT interceptor
- **Locales**: `src/locales/de` and `src/locales/en` - i18n translations (JS objects, not JSON)
- **Store**: Pinia stores in `src/stores/`

### API Communication
- Use `API.get()`, `API.post()` from `utils/api.js` (auto-adds auth headers)
- Base URL: `import.meta.env.VITE_API_URL` (defaults to `http://localhost:8180`)
- Example:
  ```js
  const response = await API.get(`/api/characters/${this.id}`)
  ```

### Translation Pattern
Add to both `locales/de` and `locales/en`:
```js
export default {
  export: {
    selectTemplate: 'Vorlage wählen',  // DE
    exportPDF: 'PDF Export'
  }
}
```

## Docker Development Workflow

### Verify Containers
```bash
docker ps | grep bamort-backend-dev  # Must be running before testing
```

### Container Names & Ports
- `bamort-backend-dev`: Go API at http://localhost:8180 (Air live-reload)
- `bamort-frontend-dev`: Vue at http://localhost:5173 (Vite HMR)
- `bamort-mariadb-dev`: MariaDB at localhost:3306
- `bamort-phpmyadmin-dev`: http://localhost:8082

### Rebuild After Dockerfile Changes
```bash
cd /data/dev/bamort
docker-compose -f docker/docker-compose.dev.yml build bamort-backend-dev
docker-compose -f docker/docker-compose.dev.yml up -d bamort-backend-dev
```

### View Logs
```bash
docker logs bamort-backend-dev --tail=50
docker logs bamort-frontend-dev --tail=20
```

## Testing Commands

### Backend Tests
```bash
cd /data/dev/bamort/backend
go test -v ./pdfrender/ -run TestExportCharacterToPDF
go test -v ./character/
```

### Frontend
- HMR auto-reloads on file save
- Check browser console and `docker logs bamort-frontend-dev`

## PDF Rendering Module (`pdfrender/`)

- Uses `chromedp` for HTML→PDF (requires Chromium in Docker)
- Templates in `templates/Default_A4_Quer/` (page1_stats.html, page2_play.html, etc.)
- Continuation pages auto-generated for overflow (page1.2_stats.html pattern)
- Test with character ID 18: generates 4 pages + continuations if needed
- Template capacity defined in HTML comments: `<!-- MaxItems: 58 -->`

## Common Patterns

### Error Handling (Backend)
```go
if err != nil {
    c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
    return
}
```

### Modal Dialogs (Frontend)
```vue
<div v-if="showDialog" class="modal-overlay" @click.self="showDialog = false">
  <div class="modal-content"><!-- content --></div>
</div>
```

### Popup Blocker Workaround
Open window **synchronously** before async calls:
```js
const newWindow = window.open('', '_blank')
// ... then await API call ...
newWindow.location.href = url
```

## Critical Rules

1. **NEVER** write example/demo code - only production code
2. **NEVER** create test files with `main()` - use `_test.go`
3. **ALWAYS** check `docker ps` before assuming containers are running
4. **ALWAYS** use TDD: write failing test first, then implement
5. **ALWAYS** use KISS principle: simplest solution that works
6. **ALWAYS** add translations to both DE and EN locales
7. **ALWAYS** use global CSS definition to ensure consistent style 

## File-Specific Instructions

Load additional instructions for specific file types:
- Go files: See `.github/instructions/go.instructions.md`
- Vue files: See `.github/instructions/vue.instructions.md`
- js files: See `.github/instructions/js.instructions.md`
- css files: See `.github/instructions/css.instructions.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bardioc26) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
