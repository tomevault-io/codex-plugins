# Copilot Instructions for SCALE User Action API

## Project Overview
This is an **ASP.NET Core 8 Minimal API** that provides a generic POST endpoint for Manhattan SCALE (Snapdragon) web applications. It acts as a bridge between SCALE's UI dialog boxes and custom SQL stored procedures, enabling user-driven data updates without creating separate APIs for each action.

**Key Architecture**: Single POST endpoint (`/ExecProc`) → Generic stored procedure (`usp_UserAction`) → Action-based SQL branching

## Critical Workflow

### API Request Flow
1. SCALE UI dialog sends POST to `/ExecProc?action=<ActionName>` with Windows credentials
2. API validates Windows authenticated user (no API key needed)
3. Request body: JSON object(s) with `internalID` and `changeValue` fields
4. Deserializes JSON (single object or array)
5. Builds comma-separated list of all `internalID` values (single or multiple)
6. Calls `usp_UserAction` **once** with 4 parameters: `@action`, `@internalID` (CSV), `@changeValue`, `@userName`
7. Returns standardized response: `{ MessageCode, Message, ConfirmationMessageCode, ConfirmationMessage }`

### Adding New Actions
**NO CODE CHANGES NEEDED** - only update `usp_UserAction.sql`:

1. Add `ELSE IF @action = N'YourActionName'` branch
2. **Optional Validation**: Check if operation is allowed (return error immediately if validation fails)
3. **Bulk Update**: Use `STRING_SPLIT(@internalID, ',')` to update all rows at once
4. Set `@MessageCode` and `@Message` before returning
5. Example pattern in `usp_UserAction.sql`:
   ```sql
   IF @action = N'WavePriority'
   BEGIN
       -- Validation (optional)
       IF EXISTS (SELECT 1 FROM table WHERE id IN (SELECT value FROM STRING_SPLIT(@internalID, ',')) AND invalid_condition)
       BEGIN
           SET @MessageCode = N'ERR_...'
           SELECT @MessageCode AS MessageCode, @Message AS Message
           RETURN
       END
       
       -- Bulk update (all rows at once)
       UPDATE table SET column = @changeValue WHERE id IN (SELECT CAST(value AS INT) FROM STRING_SPLIT(@internalID, ','))
   END
   ```

## Configuration Pattern

### IIS Deployment (Production)
Uses **Windows Authentication** - no secrets stored in configuration files:

1. **web.config** (in published folder, not source control):
   - Connection string uses `Integrated Security=true` (Windows Auth) - no username/password
   - `forwardWindowsAuthToken="true"` passes Windows identity to ASP.NET Core
   - Windows Authentication enabled, Anonymous Authentication disabled
   - Uses double underscore (`__`) for nested config keys (ASP.NET Core convention)

2. **appsettings.json** (source control):
   - Contains `AllowedHosts` and connection string template
   - Development/staging configuration

**Security Model**: 
- IIS Application Pool identity authenticates to SQL Server (no stored credentials)
- User's Windows identity captured for auditing
- Service account must have SQL Server permissions

### Local Development
- Uses `Properties/launchSettings.json` for HTTPS profile (default: https://localhost:7174)
- Configure connection string in User Secrets or `appsettings.Development.json` (not in repo)

## Code Conventions

### Security Patterns
- **Windows Authentication**: Validates `context.User.Identity?.Name` for authenticated user
- **Authorization**: `[Authorize]` attribute applied globally via fallback policy
- **Payload limits**: 10KB max request size enforced
- **SQL injection**: Use `SqlCommand` with `AddWithValue` - never string concatenation
- **HTTPS only**: HSTS enabled with 60-day max age
- **Auditing**: `@userName` parameter passed to all stored procedure calls

### JSON Deserialization
The API accepts **both single object and array** formats (lines 98-111):
```csharp
// Tries array first, falls back to single object
List<Dictionary<string, object>> items = JsonSerializer.Deserialize<List<...>>(raw);
// If fails, tries single object and wraps in list
```

### Error Response Format
All errors return consistent structure (see lines 70-75, 124-130):
```json
{
  "ErrorCode": "MissingParams",
  "ErrorType": 1,
  "Message": "Human-readable error",
  "AdditionalErrors": [],
  "Data": null
}
```

## Integration with SCALE

### SCALE Dialog Configuration
Use event `_webUi.insightListPaneActions.modalDialogPerformPostForSelection` with parameters:
- `POSTServiceURL=/UserAction/ExecProc?action=<ActionName>`
- `PostData_Grid_<GridName>_internalID=<fieldName>` (maps to `@internalID`)
- `PostData_Input_<EditorName>_changeValue=<value>` (maps to `@changeValue`)
- `ModalDialogName=<DialogName>`
- `UseDefaultCredentials=true` (passes Windows credentials - **REQUIRED**)

**Example**: See README.md step 8 for WavePriority button integration

## Build & Deployment

### Build Commands
```powershell
dotnet build                    # Build solution
dotnet publish -c Release       # Publish to bin\Release\net8.0\publish
```

### Deployment Checklist
1. Publish to folder on SCALE IIS server
2. Update `web.config` with actual connection string (Integrated Security)
3. Create IIS Application Pool (use service account)
4. Add as Application under SCALE site (suggested name: `UserAction`)
5. Enable Windows Authentication, disable Anonymous Authentication
6. Grant SQL Server permissions to service account
7. Recycle app pool to apply changes
8. Test with `/health` endpoint

## Database Schema Requirements

### Stored Procedure Signature
```sql
CREATE OR ALTER PROC [dbo].[usp_UserAction] (
    @action NVARCHAR(50),
    @internalID NVARCHAR(MAX),  -- CSV: "123" or "123,456,789"
    @changeValue NVARCHAR(50),
    @userName NVARCHAR(255) = NULL  -- Windows authenticated user
)
```

**Must return**: `SELECT @MessageCode AS MessageCode, @Message AS Message`

### Key Patterns

**Validation (optional)**:
```sql
IF EXISTS (SELECT 1 FROM table WHERE id IN (SELECT value FROM STRING_SPLIT(@internalID, ',')) AND invalid_condition)
BEGIN
    SET @MessageCode = N'ERR_...'
    SET @Message = N'Error description'
    SELECT @MessageCode AS MessageCode, @Message AS Message
    RETURN  -- Early return prevents update
END
```

**Bulk Update** (processes all rows at once):
```sql
UPDATE table 
SET column = @changeValue 
WHERE id IN (SELECT CAST(value AS INT) FROM STRING_SPLIT(@internalID, ','))
```

**Per-Row Processing** (when business logic requires it):
```sql
DECLARE cursor_name CURSOR FOR SELECT CAST(value AS INT) FROM STRING_SPLIT(@internalID, ',')
-- Process each ID individually
```

### Parameter Flexibility
- `@internalID` passed as NVARCHAR(MAX) with comma-separated values
- Cast in stored procedure as needed: `CAST(value AS INT)`, `CONVERT(NUMERIC(9,0), value)`
- API handles JsonElement types and converts to appropriate SQL parameters

## Dependencies
- **Microsoft.Data.SqlClient** (6.1.1): SQL Server connectivity
- **Swashbuckle.AspNetCore** (9.0.4): Swagger UI (development only)
- **.NET 8.0**: Target framework with nullable reference types enabled

## Testing
- **Health Check**: `GET /health` returns `{ status: "ok" }`
- **Manual Testing**: Use Postman/curl with `x-api-key` header and JSON body
- No unit tests in project - validate via SCALE integration testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShutterSeeker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
