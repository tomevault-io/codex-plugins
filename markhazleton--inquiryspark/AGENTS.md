# InquirySpark AI Coding Agent Instructions

## Project Overview
InquirySpark is a .NET 10 survey/inquiry management system with three main applications:
- **InquirySpark.WebApi** - RESTful API with Swagger documentation
- **InquirySpark.Admin** - MVC admin interface with Bootstrap 5 + DataTables
- **InquirySpark.Web** - Razor Pages public-facing application

Core business logic resides in **InquirySpark.Common** (shared models/SDK) and **InquirySpark.Repository** (EF Core + services).

## Architecture Patterns

### Response Wrapper Pattern
All service methods return `BaseResponse<T>` or `BaseResponseCollection<T>` for consistent error handling:

```csharp
// Single item
public async Task<BaseResponse<SurveyItem>> GetSurveyBySurveyId(int surveyId)
{
    return await DbContextHelper.ExecuteAsync<SurveyItem>(async () =>
    {
        return await _context.Surveys
            .Where(w => w.SurveyId == surveyId)
            .Include(i => i.QuestionGroups)
            .Select(s => SurveyServices_Mappers.Create(s))
            .FirstOrDefaultAsync();
    });
}
```

**Key Points:**
- `DbContextHelper.ExecuteAsync` wraps EF Core operations with exception handling
- Returns `IsSuccessful` boolean and `Errors` collection
- `Data` property contains the result (null if failed)
- Use `ExecuteCollectionAsync` for List<T> results

### Dependency Injection Pattern
Services follow constructor injection with primary constructors:

```csharp
public class SurveyService(InquirySparkContext context, ILogger<SurveyService> logger) : ISurveyService
{
    private readonly InquirySparkContext _context = context;
    private readonly ILogger<SurveyService> _logger = logger;
}
```

Register services in Program.cs:
```csharp
builder.Services.AddDbContext<InquirySparkContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddTransient<ISurveyService, SurveyService>();
```

### EF Core Context Pattern
`InquirySparkContext` contains 40+ DbSets including:
- Entity tables (Application, Survey, Question, Company)
- Lookup tables (LuApplicationType, LuSurveyType, LuQuestionType)
- View mappings (VwApplication, VwQuestionLibrary, VwSurveyResponseDetail)

**Views are heavily used for complex queries** - check for existing views before writing complex joins.

## Database Conventions

### Naming Patterns
- Primary Keys: `{Entity}Id` (e.g., `SurveyId`, `QuestionId`)
- Foreign Keys: `{Entity}Id` (e.g., `SurveyTypeId`)
- Boolean flags: `{Name}Fl` (e.g., `ActiveFl`, `CommentFl`)
- Descriptions: `{Name}Ds` (e.g., `QuestionDs`, `SurveyDs`)
- Names: `{Name}Nm` (e.g., `SurveyNm`, `SurveyShortNm`)
- Codes: `{Name}Cd` (e.g., `ApplicationCd`)
- Dates: `{Name}Dt` (e.g., `ModifiedDt`, `StartDt`)
- Audit fields: `ModifiedDt`, `ModifiedId` (present on most tables)

### Connection Strings
- **WebApi/Repository**: `InquirySparkContext` (SQL Server)
- **Admin**: `InquirySparkConnection` (SQLite for demo) + `ControlSparkUserContextConnection` (Identity)
- **Web**: `DefaultConnection` (SQL Server with Identity)

## Admin UI Conventions (InquirySpark.Admin)

### Bootstrap 5 + DataTables Standard
**All CRUD index views follow this pattern:**

```html
<div class="card border-0 shadow-sm">
    <div class="card-header bg-primary text-white d-flex justify-content-between align-items-center">
        <h2 class="mb-0"><i class="bi bi-{icon}"></i> {Title}</h2>
        <a asp-action="Create" class="btn btn-light btn-sm">
            <i class="bi bi-plus-circle"></i> Create New
        </a>
    </div>
    <div class="card-body p-0">
        <div class="table-responsive">
            <table class="table table-striped table-hover align-middle mb-0">
                <thead class="table-light">
                    <tr>
                        <th><i class="bi bi-{icon}"></i> Column Name</th>
                        <th class="no-sort">Actions</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach (var item in Model)
                    {
                        <tr>
                            <td>@item.Property</td>
                            <td>
                                <div class="btn-group btn-group-sm" role="group">
                                    <a asp-action="Details" asp-route-id="@item.Id" class="btn btn-outline-info" title="View Details">
                                        <i class="bi bi-eye"></i>
                                    </a>
                                    <a asp-action="Edit" asp-route-id="@item.Id" class="btn btn-outline-primary" title="Edit">
                                        <i class="bi bi-pencil"></i>
                                    </a>
                                    <a asp-action="Delete" asp-route-id="@item.Id" class="btn btn-outline-danger" title="Delete">
                                        <i class="bi bi-trash"></i>
                                    </a>
                                </div>
                            </td>
                        </tr>
                    }
                </tbody>
            </table>
        </div>
    </div>
    <div class="card-footer text-muted d-flex justify-content-between align-items-center">
        <span><i class="bi bi-info-circle"></i> Total: @Model.Count() records</span>
        <span><i class="bi bi-lightning"></i> DataTables enabled</span>
    </div>
</div>
```

**Key Requirements:**
- Use Bootstrap utility classes (no inline styles, no custom CSS)
- Use Bootstrap Icons (bi-*) for visual indicators
- DataTables auto-initializes on `.table` class (configured in site.js)
- Add `class="no-sort"` to action columns
- Card layout with header/body/footer structure
- See [BOOTSTRAP5-TABLE-TEMPLATE.md](InquirySpark.Admin/BOOTSTRAP5-TABLE-TEMPLATE.md) for complete template

### NPM Build System (100% CDN-Free)
Frontend dependencies are managed via npm, NOT LibMan or CDN:

```bash
# Install dependencies
npm install

# Build (copies to wwwroot/lib)
npm run build

# Automatically runs on dotnet build
```

**Libraries Included:**
- jQuery 3.7.1, Bootstrap 5.3.8 (JS only, CSS from Bootswatch)
- DataTables 2.3.5 with Bootstrap 5 integration
- DataTables Extensions: Buttons, Responsive, Select, SearchPanes
- Bootstrap Icons 1.13.1
- jQuery Validation (for ASP.NET Core forms)
- JSZip & PDFMake (for DataTables export)

**Critical:** Bootstrap CSS comes from WebSpark.Bootswatch theme system, NOT npm. Only JavaScript is local.

### DataTables Configuration
Configured in [wwwroot/js/site.js](InquirySpark.Admin/wwwroot/js/site.js):
- Auto-initializes all `.table` elements
- Default: 25 rows, sorting, searching, pagination
- Add `data-datatable="false"` to disable
- Add `.datatable-export` class for Excel/PDF/CSV export buttons
- Add `class="no-sort"` to `<th>` to disable column sorting
- State saving enabled (24-hour persistence)

### Bootswatch Theme Integration
Dynamic theme switching via WebSpark.Bootswatch package:
- 28+ themes with light/dark modes
- CSS served from embedded resources (no CDN)
- JavaScript from local npm package
- Theme persists in cookies
- See [CDN-FREE-IMPLEMENTATION.md](InquirySpark.Admin/CDN-FREE-IMPLEMENTATION.md)

## Global Usings

### InquirySpark.Admin
```csharp
global using Microsoft.AspNetCore.Http;
global using Microsoft.AspNetCore.Mvc;
global using System.Globalization;
global using WebSpark.Bootswatch;
global using WebSpark.HttpClientUtility;
```

### InquirySpark.Repository
Check existing global usings before adding namespace imports.

## Controller Patterns

### Admin Controllers
Inherit from `BaseController` which provides `_logger`:

```csharp
public class SurveysController(ILogger<SurveysController> logger) : BaseController(logger)
{
    // Use _logger for logging
}
```

### API Controllers
Use primary constructors with ApiController attribute:

```csharp
[ApiController]
[Route("api/[controller]")]
public class SurveyController(ISurveyService service, ILogger<SurveyController> logger) : ControllerBase
{
    protected readonly ISurveyService _service = service;
    protected readonly ILogger<SurveyController> _logger = logger;
    
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        return await ApiResponseHelper.ExecuteAsync(() => _service.GetSurveyCollection(), _logger);
    }
}
```

`ApiResponseHelper` converts `BaseResponse<T>` to appropriate IActionResult (200/400/404/500).

## Build & Test Commands

### Building
```powershell
# Build entire solution
dotnet build InquirySpark.sln

# Build specific project
dotnet build InquirySpark.Admin/InquirySpark.Admin.csproj

# Admin project automatically runs npm build
```

### Running
```powershell
# API (default: https://localhost:5001)
dotnet run --project InquirySpark.WebApi

# Admin (default: https://localhost:7001)
dotnet run --project InquirySpark.Admin

# Web (default: https://localhost:5002)
dotnet run --project InquirySpark.Web
```

### Testing
```powershell
# Run all tests
dotnet test

# Run specific test project
dotnet test InquirySpark.Common.Tests
```

## Common Pitfalls

1. **Don't add Bootstrap CSS to npm** - Themes come from Bootswatch, only JS is local
2. **Don't use inline styles** - Use Bootstrap utility classes exclusively
3. **Don't forget `DbContextHelper.ExecuteAsync`** - Repository methods must wrap EF operations
4. **Don't query entities directly in controllers** - Use services and response wrappers
5. **Check for existing views** - Database has many pre-built views for complex queries
6. **Don't bypass DataTables** - All admin tables should auto-initialize unless explicitly disabled
7. **Use global usings** - Check GlobalUsing.cs before adding repetitive using statements

## XML Documentation
All public APIs require XML doc comments (enabled in Common, Repository, WebApi projects):

```csharp
/// <summary>
/// Gets survey by unique identifier
/// </summary>
/// <param name="surveyId">The survey identifier</param>
/// <returns>Survey item with question groups and members</returns>
public async Task<BaseResponse<SurveyItem>> GetSurveyBySurveyId(int surveyId)
```

## Documentation File Organization

**CRITICAL: All Copilot-generated .md files MUST follow this structure:**

- **Root README.md ONLY**: The only .md file allowed outside `/docs` is `README.md` in the project root
- **All other documentation**: MUST be placed in `/docs/copilot/session-{date}/` folders
  - Format: `/docs/copilot/session-YYYY-MM-DD/` (e.g., `/docs/copilot/session-2025-12-04/`)
  - Session folders group related documentation by date
  - No .md files in project folders (InquirySpark.Admin, InquirySpark.Repository, etc.)

**Examples:**
- ✅ `/docs/copilot/session-2025-12-04/BOOTSTRAP5-TABLE-TEMPLATE.md`
- ✅ `/docs/copilot/session-2025-12-04/API-DESIGN.md`
- ✅ `/README.md`
- ❌ `InquirySpark.Admin/BOOTSTRAP5-MODERNIZATION.md`
- ❌ `InquirySpark.Repository/MIGRATION-GUIDE.md`

## References
- [Bootstrap 5 Table Template](docs/copilot/session-2025-12-04/BOOTSTRAP5-TABLE-TEMPLATE.md)
- [Bootstrap 5 Modernization](docs/copilot/session-2025-12-04/BOOTSTRAP5-MODERNIZATION.md)
- [NPM Build Process](docs/copilot/session-2025-12-04/NPM-BUILD.md)
- [CDN-Free Implementation](docs/copilot/session-2025-12-04/CDN-FREE-IMPLEMENTATION.md)
- [DataTables Reference](docs/copilot/session-2025-12-04/DATATABLES-REFERENCE.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markhazleton)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/markhazleton)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
