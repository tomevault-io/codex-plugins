# PowerShell Repository — Copilot Instructions

## Overview

This repository is a collection of reusable PowerShell scripts organized by technology area (Dataverse, EntraID, Files, PowerShell, SharePoint, etc.). Scripts follow a consistent structure and, when they call authenticated APIs, are split into two versions: a **base script** and a **WithAuth wrapper**.

---

## Project Structure

```
<TechnologyArea>/
    ScriptName.ps1              # Base script (accepts AccessToken or handles its own auth)
    ScriptNameWithAuth.ps1      # Auth wrapper that acquires a token then calls the base script
EntraID/
    GetAccessTokenDeviceCode.ps1  # Shared device-code-flow auth helper
```

- Group scripts into folders by technology/service (e.g., `Dataverse/`, `SharePoint/`, `Files/`, `PowerShell/`).
- Each folder should contain related scripts that target that technology.

---

## Script Conventions

### Comment-Based Help

Every script **must** start with a `<# ... #>` comment-based help block containing at minimum:

| Section | Required | Notes |
|---|---|---|
| `.SYNOPSIS` | Yes | One-line summary of what the script does. |
| `.DESCRIPTION` | Yes | Detailed explanation of behavior, parameters, and any prerequisites. |
| `.PARAMETER` | Yes | One entry per parameter — describe purpose, valid values, and defaults. |
| `.EXAMPLE` | Yes | At least one realistic usage example with explanation text. |
| `.AUTHOR` | Optional | `Rick Wilson` when included. |
| `.NOTES` | Optional | Prerequisites, module installs, security notes, version info. |

### Parameters

- Define parameters using a `param( )` block immediately after the help comment.
- Use `[Parameter(Mandatory = $true)]` for required parameters.
- Use `[ValidateSet()]` for parameters with a fixed set of values (e.g., `Environment`).
- Provide sensible defaults where appropriate (e.g., `$Environment = "Public"`).
- Use strong typing (`[string]`, `[string[]]`, `[switch]`, `[bool]`).
- For Dataverse/API scripts, the base script **always** takes `[string]$OrganizationUrl` and `[string]$AccessToken` as parameters.

### Code Style

- Use descriptive variable names in PascalCase (e.g., `$AccessToken`, `$OrganizationUrl`).
- Use `Write-Host` with `-ForegroundColor` for user-facing status messages (Cyan for info, Green for success, Yellow for warnings, Red for errors).
- Use `Write-Error` for fatal errors and `Write-Warning` for non-fatal issues.
- Use `Write-Output` or `return` for pipeline-friendly output.
- Extract reusable logic into local functions within the script when needed.
- Use hashtable splatting (`@params`) for calls with many parameters.

---

## Authentication Pattern — Two-Script Approach

When a script calls an authenticated API (Dataverse Web API, Microsoft Graph, etc.), create **two scripts**:

### 1. Base Script (`ScriptName.ps1`)

- Accepts `$AccessToken` as a parameter — it does **not** handle authentication itself.
- Contains all the business logic (API calls, data processing, output).
- Can be called standalone by anyone who already has a token (e.g., from another auth flow, a pipeline, or a service principal).
- Sets up HTTP headers using the provided token:

```powershell
$headers = @{
    "Authorization"    = "Bearer $AccessToken"
    "Content-Type"     = "application/json"
    "OData-MaxVersion" = "4.0"
    "OData-Version"    = "4.0"
}
```

- Uses `Invoke-RestMethod` for API calls.

### 2. Auth Wrapper Script (`ScriptNameWithAuth.ps1`)

- Handles token acquisition, then delegates to the base script.
- Accepts the **same parameters** as the base script **except** `$AccessToken`, and **adds** these auth parameters:

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `$TenantId` | `[string]` | Yes | — | Azure AD tenant ID |
| `$ClientId` | `[string]` | Yes | — | App registration client ID |
| `$Environment` | `[string]` | No | `"Public"` | Azure cloud: `Public`, `GCC`, `GCCH`, `DoD` |

- Acquires a token by calling the shared auth helper:

```powershell
$accessToken = & ..\EntraID\GetAccessTokenDeviceCode.ps1 `
    -TenantId $TenantId `
    -ClientId $ClientId `
    -Scope "$OrganizationUrl/user_impersonation" `
    -Environment $Environment
```

- Then calls the base script, passing the token and all other parameters:

```powershell
$response = & .\ScriptName.ps1 -OrganizationUrl $OrganizationUrl -AccessToken $accessToken -OtherParam $OtherParam
Write-Output $response
```

- For scripts with many parameters, use splatting to forward them cleanly:

```powershell
$scriptParams = @{
    OrganizationUrl = $OrganizationUrl
    AccessToken     = $accessToken
    OutputFormat    = $OutputFormat
}
# conditionally add optional params
if ($Tables) { $scriptParams.Tables = $Tables }

$mainScript = Join-Path $scriptDir "ScriptName.ps1"
$results = & $mainScript @scriptParams
return $results
```

- Validates the token was acquired before proceeding:

```powershell
if (-not $accessToken) {
    Write-Error "Failed to acquire access token."
    exit 1
}
```

### Path Resolution for Auth Script

Two patterns are used to reference the auth helper from a WithAuth script:

- **Relative path** (simpler scripts): `& ..\EntraID\GetAccessTokenDeviceCode.ps1`
- **`$PSScriptRoot` / `Split-Path`** (more robust):

```powershell
$scriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
$authScript = Join-Path $scriptDir "..\EntraID\GetAccessTokenDeviceCode.ps1"
$accessToken = & $authScript -TenantId $TenantId -ClientId $ClientId -Scope "$OrganizationUrl/user_impersonation" -Environment $Environment
```

Prefer the `Split-Path` approach for new scripts as it is more reliable when the working directory differs from the script location.

---

## Shared Auth Helper — `EntraID/GetAccessTokenDeviceCode.ps1`

This is the **single shared authentication script** used by all WithAuth wrappers. It implements the **OAuth 2.0 Device Code Flow**.

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `$TenantId` | `[string]` | — | Azure AD tenant ID |
| `$ClientId` | `[string]` | — | App registration client/application ID |
| `$Scope` | `[string]` | `"https://your-org.crm.dynamics.com/.default"` | OAuth scope for the target resource |
| `$Environment` | `[string]` | `"Public"` | Azure cloud environment (`Public`, `GCC`, `GCCH`, `DoD`) |

### How It Works

1. Selects the correct login endpoint based on `$Environment`:
   - **Public**: `https://login.microsoftonline.com`
   - **GCC / GCCH / DoD**: `https://login.microsoftonline.us`
2. Requests a device code from `/$TenantId/oauth2/v2.0/devicecode`.
3. Prompts the user to visit the verification URL and enter the code.
4. Polls `/$TenantId/oauth2/v2.0/token` until the user completes sign-in.
5. Returns **only the access token string** via `Write-Output`.

### Scope Conventions

- **Dataverse**: `"$OrganizationUrl/user_impersonation"` (e.g., `https://your-org.crm.dynamics.com/user_impersonation`)
- **Microsoft Graph**: `"https://graph.microsoft.com/.default"`
- Other APIs: Use the appropriate resource URI with `/user_impersonation` or `/.default`.

---

## When to Create Auth vs. No-Auth Versions

| Scenario | What to Create |
|---|---|
| Script calls an API requiring a Bearer token | Create **both** `ScriptName.ps1` (base) and `ScriptNameWithAuth.ps1` (wrapper) |
| Script uses PowerShell modules with built-in auth (e.g., `Connect-AzureAD`, `Add-PowerAppsAccount`) | Create a **single script** — the module handles its own interactive auth prompts |
| Script is purely local (file operations, XML processing, etc.) | Create a **single script** — no auth needed |

---

## Dataverse API Conventions

When calling the Dataverse Web API:

- Use API version `v9.2`: `$OrganizationUrl/api/data/v9.2/`
- Always include these headers:

```powershell
$headers = @{
    "Authorization"    = "Bearer $AccessToken"
    "Content-Type"     = "application/json"
    "OData-MaxVersion" = "4.0"
    "OData-Version"    = "4.0"
}
```

- For queries that need annotations: add `"Prefer" = "odata.include-annotations=*"` and `"Accept" = "application/json"`.
- Use `Invoke-RestMethod` (not `Invoke-WebRequest`) for API calls.
- Convert payloads to JSON with `ConvertTo-Json -Depth 10` to handle nested objects.
- Remove trailing slashes from URLs: `$OrganizationUrl = $OrganizationUrl.TrimEnd('/')`.

---

## Template: New Base Script (API)

```powershell
<#
.SYNOPSIS
    Brief description of what the script does.

.DESCRIPTION
    Detailed description of the script's behavior and purpose.

.PARAMETER OrganizationUrl
    The URL of the Dataverse organization (e.g., https://your-org.crm.dynamics.com).

.PARAMETER AccessToken
    The access token for authenticating with the Dataverse Web API.

.PARAMETER YourParam
    Description of additional parameter.

.EXAMPLE
    .\ScriptName.ps1 -OrganizationUrl "https://your-org.crm.dynamics.com" -AccessToken $token -YourParam "value"

    Description of what this example does.
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$OrganizationUrl,

    [Parameter(Mandatory = $true)]
    [string]$AccessToken,

    [Parameter(Mandatory = $true)]
    [string]$YourParam
)

# Remove trailing slash from URL if present
$OrganizationUrl = $OrganizationUrl.TrimEnd('/')

# Set up headers for API calls
$headers = @{
    "Authorization"    = "Bearer $AccessToken"
    "Content-Type"     = "application/json"
    "OData-MaxVersion" = "4.0"
    "OData-Version"    = "4.0"
}

# API endpoint
$apiUrl = "$OrganizationUrl/api/data/v9.2/YourEndpoint"

# Make the API call
$response = Invoke-RestMethod -Method Get -Uri $apiUrl -Headers $headers

# Output the response
$response
```

---

## Template: New WithAuth Wrapper Script

```powershell
<#
.SYNOPSIS
    Acquires an access token and calls ScriptName to perform the operation.

.DESCRIPTION
    This script calls the GetAccessTokenDeviceCode script to acquire an access token
    and then calls the ScriptName script to perform the operation.

.PARAMETER TenantId
    The Azure AD tenant ID.

.PARAMETER ClientId
    The client ID (application ID) of your registered Azure AD app.

.PARAMETER Environment
    The Azure environment. Valid values are "Public", "GCC", "GCCH", "DoD". Default value is "Public".

.PARAMETER OrganizationUrl
    The URL of the Dataverse organization.

.PARAMETER YourParam
    Description of additional parameter.

.EXAMPLE
    .\ScriptNameWithAuth.ps1 -TenantId "YOUR_TENANT_ID" -ClientId "YOUR_CLIENT_ID" -OrganizationUrl "https://your-org.crm.dynamics.com" -YourParam "value"

    Description of what this example does.
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$TenantId,

    [Parameter(Mandatory = $true)]
    [string]$ClientId,

    [Parameter(Mandatory = $false)]
    [ValidateSet("Public", "GCC", "GCCH", "DoD")]
    [string]$Environment = "Public",

    [Parameter(Mandatory = $true)]
    [string]$OrganizationUrl,

    [Parameter(Mandatory = $true)]
    [string]$YourParam
)

# Get the access token using device code flow
Write-Host "Acquiring access token..." -ForegroundColor Cyan
$scriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
$authScript = Join-Path $scriptDir "..\EntraID\GetAccessTokenDeviceCode.ps1"
$accessToken = & $authScript -TenantId $TenantId -ClientId $ClientId -Scope "$OrganizationUrl/user_impersonation" -Environment $Environment

if (-not $accessToken) {
    Write-Error "Failed to acquire access token."
    exit 1
}

Write-Host "Access token acquired successfully." -ForegroundColor Green

# Call the base script
$mainScript = Join-Path $scriptDir "ScriptName.ps1"
$response = & $mainScript -OrganizationUrl $OrganizationUrl -AccessToken $accessToken -YourParam $YourParam

# Output the response
Write-Output $response
```

---

## Template: New Standalone Script (No Auth)

```powershell
<#
.SYNOPSIS
    Brief description.

.DESCRIPTION
    Detailed description.

.PARAMETER YourParam
    Description.

.EXAMPLE
    .\ScriptName.ps1 -YourParam "value"

    Description of what this example does.

.NOTES
    Author: Rick Wilson
    Date  : YYYY-MM-DD
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$YourParam
)

# Script logic here
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rwilson504) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
