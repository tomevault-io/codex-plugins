# Copilot Instructions for FCP SFD Portal Stub

## Project Overview

This is a **reference implementation** demonstrating how client portals integrate with the **SFD Document Upload Service**. It's a stub/example, not a production portal. The codebase showcases:

- OAuth2 authentication with AWS Cognito (via CDP API Gateway)
- Initiating upload sessions with business metadata via Object Processor API
- **Three upload patterns**: gateway-routing (recommended), frontend-redirect (multi-client), and direct (fallback)
- Browser-based file uploads (routing depends on mode - see Upload Modes section)
- Polling for upload status through Object Processor

**Critical Architecture**: This stub demonstrates **three different upload patterns**:
- **gateway-routing/frontend-redirect**: Files flow User Browser → Gateway → CDP Uploader → S3
- **direct mode**: Files flow User Browser → CDP Uploader → S3 (requires JavaScript)

The portal handles metadata and orchestration. In all modes, files **never pass through the portal backend** - they go directly from browser to uploader (via gateway or direct).

## Tech Stack Conventions

- **ESM modules only** - all `import`/`export`, no CommonJS (`require`/`module.exports`)
- **Hapi.js 21** - server framework with plugin architecture
- **Nunjucks** - templating engine with GOV.UK Frontend components
- **Webpack** - bundles client-side JS/SCSS from `src/client/`
- **Vitest** - testing framework with `vi` mocking utilities
- **Neostandard** - ESLint config (ECMAScript 2025)
- **Convict** - schema-based configuration management
- **Pino** - structured logging in ECS format (production) or pretty format (dev)

## Code Patterns

### Route Definitions

Routes export objects with `method`, `path`, and `handler` properties:

```javascript
export const metadataGet = {
  method: 'GET',
  path: '/document-upload/metadata',
  handler: (request, h) => {
    // Route logic
  }
}
```

All routes are auto-registered via [src/plugins/router.js](../src/plugins/router.js) which finds route files in `src/routes/`.

### Session State

Use `request.yar` (Hapi Yar plugin) for session management:

```javascript
request.yar.set('metadata', metadata)
const crn = request.yar.get('crn')
```

Session cookies configured in [src/plugins/session.js](../src/plugins/session.js).

### Configuration

Use [src/config/config.js](../src/config/config.js) convict schema. Access via:

```javascript
import { config } from './config/config.js'
const host = config.get('objectProcessor.host')
```

Override with environment variables (see `../.env.example`).

### Logging

Import `createLogger()` from [src/common/helpers/logging/logger.js](../src/common/helpers/logging/logger.js):

```javascript
const logger = createLogger()
logger.info({ submissionId }, 'Initiating upload')
logger.error({ error, statusUrl }, 'Status check failed')
```

Use structured logging with context objects.

### Object Processor Integration

Two main API calls in [src/common/helpers/object-processor.js](../src/common/helpers/object-processor.js):

1. **`initiateUpload(metadata)`** - POST `/api/v1/initiate` with business metadata, returns `{correlationId, uploadId, uploadUrl, statusUrl}`
2. **`getUploadStatus(statusUrl)`** - GET status URL to poll upload progress

Both automatically include OAuth2 Bearer token when Cognito enabled (`COGNITO_ENABLED=true`).

### Upload Modes

The portal supports **three upload patterns** controlled by `UPLOAD_MODE` environment variable:

#### 1. Gateway Routing Mode (Default, Recommended)
- **User access**: `http://localhost:3019` (nginx gateway)
- **Upload flow**: Browser → Gateway → CDP Uploader (standard HTML form, no JavaScript)
- **Redirect handling**: Relative redirects stay within gateway domain
- **CSP**: Simple - only `'self'` needed
- **Progressive enhancement**: ✅ Works without JavaScript
- **Best for**: CDP services, services with gateway infrastructure

#### 2. Frontend Redirect Mode (Multi-Client)
- **User access**: `http://localhost:3020` (portal directly)
- **Upload flow**: Browser → Gateway → CDP Uploader (standard HTML form, no JavaScript)
- **Redirect handling**: Gateway routes to document-upload-frontend-stub which maps client identifiers to absolute URLs
- **CSP**: Requires gateway domain in `form-action` directive
- **Progressive enhancement**: ✅ Works without JavaScript
- **Best for**: Multi-client scenarios, external services needing centralized redirect mapping
- **Key files**: `document-upload-frontend-stub/` service

#### 3. Direct Mode (Fallback Only)
- **User access**: `http://localhost:3020` (portal directly)
- **Upload flow**: Browser → CDP Uploader (JavaScript fetch with redirect interception)
- **Redirect handling**: JavaScript detects opaque redirect and manually navigates
- **CSP**: Requires uploader domain in `form-action` and `connect-src`
- **Progressive enhancement**: ❌ Requires JavaScript - fails without client-side code
- **Best for**: Situations where gateway infrastructure is not available (use as last resort)
- **Implementation**: [src/client/javascript/document-upload.js](../src/client/javascript/document-upload.js)

**Key Code Pattern** (all modes):
In [src/routes/document-upload.js](../src/routes/document-upload.js), the upload URL is set based on mode:

```javascript
const uploadMode = config.get('uploadMode')
let uploadUrl = result.uploadUrl

if (uploadMode === 'gateway-routing' || uploadMode === 'frontend-redirect') {
  // Route uploads through gateway for both modes
  const gatewayUrl = config.get('gatewayUrl')
  uploadUrl = `${gatewayUrl}/upload-and-scan/${result.uploadId}`
}
// In direct mode, use uploadUrl from Object Processor as-is
```

**Redirect Prefix** (frontend-redirect only):
In frontend-redirect mode, the redirect path is prefixed with `/fcp-sfd-doc-upload/{clientIdentifier}` so the gateway can route to the document-upload-frontend-stub:

```javascript
if (uploadMode === 'frontend-redirect') {
  const clientIdentifier = 'portal-stub'
  redirect = `/fcp-sfd-doc-upload/${clientIdentifier}${redirect}`
}
```

## Document Upload Stub (Local Development)

The `document-upload-stub/` directory contains a **local development stub** that replaces both Object Processor and CDP Uploader. This stub is **used by default** in `docker compose up`, eliminating external dependencies for local development.

### Stub Architecture

- **Hapi.js server** on port 3021 (`http://localhost:3021`)
- **In-memory storage** for upload sessions (no database)
- **CORS enabled** to accept browser requests from portal (port 3020)
- **No authentication** required (suitable for local dev only)

### Stub Endpoints

1. **POST `/api/v1/initiate`** - Creates upload session, returns `{correlationId, uploadId, uploadUrl, statusUrl}`
2. **GET `/api/v1/status/{correlationId}`** - Returns upload status (simulates IN_PROGRESS → SUCCESSFUL after 2s)
3. **POST `/upload-and-scan/{uploadId}`** - Accepts file uploads, returns 302 redirect

### Key Files in Stub

- `document-upload-stub/src/server.js` - Hapi server setup with CORS
- `document-upload-stub/src/routes/object-processor.js` - Object Processor API stubs
- `document-upload-stub/src/routes/uploader.js` - CDP Uploader stub
- `document-upload-stub/src/config/config.js` - Simple convict config (port, uploaderHost)

### How Sessions Work

Sessions stored in `Map()` keyed by `correlationId`. The uploader route uses `getSessionByUploadId()` helper to find sessions and `updateSessionStatus()` to mark them SUCCESSFUL after timeout.

### CSP Configuration

Portal's CSP ([src/plugins/content-security-policy.js](../src/plugins/content-security-policy.js)) is **automatically configured based on `UPLOAD_MODE`**:

- **gateway-routing**: Only `'self'` needed (same-origin uploads)
- **frontend-redirect**: Needs gateway domain in `form-action` via `ADDITIONAL_UPLOAD_DOMAINS` (cross-origin form POST)
- **direct**: Needs uploader domain in both `form-action` and `connect-src` via `ADDITIONAL_UPLOAD_DOMAINS`

In [compose.yml](../compose.yml), `ADDITIONAL_UPLOAD_DOMAINS` defaults to `http://localhost:3019,http://localhost:3021` to support all modes in local development.

The CSP plugin reads `config.get('uploadMode')` and dynamically adjusts `formAction` and `connectSrc` directives accordingly.

## Development Workflow

### Local Development

```bash
# Gateway routing mode (DEFAULT - recommended)
docker compose up
# Visit http://localhost:3019 (nginx gateway)

# Frontend redirect mode (multi-client scenarios)
UPLOAD_MODE=frontend-redirect docker compose up
# Visit http://localhost:3020 (portal directly)

# Direct mode (fallback, requires JavaScript)
UPLOAD_MODE=direct docker compose up
# Visit http://localhost:3020 (portal directly)

# Or run directly without Docker (requires Node 24.12.0+)
npm install
npm run dev  # Starts webpack watch + nodemon
```

**Services and Ports**:
- **3019**: nginx gateway (gateway-routing user access, all modes upload endpoint)
- **3020**: Portal application (frontend-redirect/direct user access)
- **3021**: Document upload stub (Object Processor + CDP Uploader APIs)
- **3022**: Document upload frontend stub (redirect mapping for frontend-redirect mode)

**Default behavior**: `UPLOAD_MODE` defaults to `gateway-routing` and `OBJECT_PROCESSOR_HOST` defaults to `http://document-upload-stub:3021` in `compose.yml`, so the portal uses the local stub automatically.

**To use real Object Processor**: Override with `OBJECT_PROCESSOR_HOST=http://fcp-sfd-object-processor:3004` in `.env`.

### Testing

```bash
# Run unit tests with coverage (in Docker)
npm run docker:test

# Watch mode for TDD
npm run docker:test:watch

# Lint
npm run lint
npm run lint:fix
```

Tests use Vitest with `vi.mock()` for dependency injection. See test files in `test/unit/` for patterns.

### Cognito Authentication

Toggle with `COGNITO_ENABLED=false` (default for local dev). When disabled, no tokens sent and Object Processor must have `AUTH_ENABLED=false`.

For production: set `COGNITO_ENABLED=true` and configure `COGNITO_DOMAIN`, `COGNITO_CLIENT_ID`, `COGNITO_CLIENT_SECRET`.

## Key Directories

- **`src/routes/`** - Route handlers (auto-registered by router plugin)
- **`src/plugins/`** - Hapi plugins (CSP, headers, router, session)
- **`src/common/helpers/`** - Shared utilities (Cognito, Object Processor client, logging)
- **`src/config/`** - Convict config schema and Nunjucks setup
- **`src/client/`** - Frontend JavaScript/SCSS (bundled by Webpack)
- **`src/views/`** - Nunjucks templates
- **`test/unit/`** - Vitest unit tests mirroring `src/` structure
- **`document-upload-stub/`** - Local development stub (replaces Object Processor + CDP Uploader)
- **`document-upload-frontend-stub/`** - Redirect mapper service (maps client identifiers to absolute URLs for frontend-redirect mode)
- **`nginx/`** - Gateway configuration (routes `/upload-and-scan/` to uploader, `/fcp-sfd-doc-upload/` to frontend stub)

## Integration Points

**Default (Local Development)**:
1. **Document Upload Stub** - Included in this repo at `document-upload-stub/`, used by default in `docker compose up`
   - Combines Object Processor and CDP Uploader functionality
   - Runs on `http://localhost:3021`
   - No authentication required
   - In-memory session storage
   - Returns relative redirects (resolved by browser to originating domain)

2. **Document Upload Frontend Stub** - Included in this repo at `document-upload-frontend-stub/`, used in frontend-redirect mode
   - Maps client identifiers to absolute redirect URLs
   - Runs on `http://localhost:3022`
   - Accessed via gateway at `/fcp-sfd-doc-upload/{clientIdentifier}/*`
   - Client mapping: `{'portal-stub': 'http://localhost:3020'}`

3. **NGINX Gateway** - Included in this repo at `nginx/`, runs on `http://localhost:3019`
   - Routes `/upload-and-scan/*` to document-upload-stub (uploader)
   - Routes `/fcp-sfd-doc-upload/*` to document-upload-frontend-stub
   - Routes all other requests to portal application
   - Required for gateway-routing and frontend-redirect modes

**Optional (Production/Testing)**:
1. **Object Processor** - Backend API for SFD Document Upload Service at `OBJECT_PROCESSOR_HOST`
2. **CDP Uploader** - File upload service (browser connects via gateway or directly depending on mode)
3. **AWS Cognito** - OAuth2 token provider via CDP API Gateway (required when using real Object Processor)
4. **Document Upload Frontend** - Production redirect mapping service (for frontend-redirect mode in production)

## Common Tasks

**Add a new route**: Create handler in `src/routes/`, export objects with `{method, path, handler}`. Router plugin auto-registers.

**Add config value**: Update `src/config/config.js` convict schema, add to `../.env.example`.

**Mock external APIs in tests**: Use `vi.mock()` at top of test file (see existing tests for patterns).

**Update GOV.UK styles**: Modify SCSS in `src/client/stylesheets/`, Webpack rebuilds on save in dev mode.

**Add Hapi plugin**: Register in `src/server.js` `server.register([...])` array.

---
> Source: [DEFRA/fcp-sfd-portal-stub](https://github.com/DEFRA/fcp-sfd-portal-stub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
