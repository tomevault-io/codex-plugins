# Pangolin Gateway Controller - AI Agent Instructions

## Project Architecture

This is a **Kubernetes Gateway API controller** written in Go that bridges Kubernetes Gateway/HTTPRoute resources with the Pangolin platform's access management API.

### Core Components (5-tier reconciliation model)

1. **Pangolin Client** (`pkg/pangolin/client.go`): REST API wrapper for Pangolin Integration API
   - Manages Sites and Resources via Integration API (self-hosted Pangolin)
   - Uses Bearer token authentication, requires `PANGOLIN_API_KEY` and `PANGOLIN_ORG_ID`
   - Key methods: `CreateSite()`, `ListSites()`, `DeleteSite()`, `PickSiteDefaults()`, `ListResources()`, `CreateResource()`
   - **CRITICAL**: `CreateSite()` does NOT return credentials - must preserve from `PickSiteDefaults()`
   - Response format: All API responses wrapped in `{"data": {...}}` structure

2. **Gateway Reconciler** (`pkg/controller/gateway_controller.go`):
   - Watches `Gateway` resources with `gatewayClassName: pangolin`
   - Creates Pangolin **newt-type Site** named `pgc-{gateway-name}` (not namespace-based)
   - Uses `PickSiteDefaults()` API to get subnet allocation, exit node, and credentials
   - Stores site ID in Gateway label: `gateway.pangolin.net/site-id`
   - Uses finalizers (`gateway.pangolin.net/finalizer`) for cleanup - **DELETES SITE ON GATEWAY DELETION**
   - Creates Secret with newt credentials for VPN authentication
   - Periodic reconciliation every 5 minutes to verify site still exists
   - Auto-recreates site if deleted from Pangolin
   - **Controller name**: `gateway-site` (unique identifier to avoid conflicts with Newt controller)

3. **Newt Reconciler** (`pkg/controller/newt_controller.go`):
   - Watches Gateway resources and automatically deploys newt VPN instances
   - Reads credentials from Secret: `{gateway-name}-newt-cred`
   - Deploys newt Deployment with WireGuard container (configurable via `NEWT_IMAGE`, default: fosrl/newt:1.10.0)
   - Creates ClusterIP Service for WireGuard ports (51820, 51821 UDP)
   - All resources use OwnerReferences to Gateway for automatic cleanup
   - **Newt endpoint auto-derived**: If API is `api.example.com`, newt connects to `pangolin.example.com`
   - **Controller name**: `gateway-newt` (unique identifier)

4. **HTTPRoute Reconciler** (`pkg/controller/httproute_controller.go`):
   - Watches `HTTPRoute` resources
   - **One Pangolin resource per unique hostname** (shared across multiple HTTPRoutes)
   - Resources named by full hostname (e.g., "test.dev0ps.me") not namespace-indexed
   - **Hostname filtering**: Skips hostnames that don't match any Pangolin domain (prevents invalid resources like "test.example.com.dev0ps.me")
   - **Path-based target ownership**: Each HTTPRoute manages only targets matching its rule paths (prevents deletion loops)
   - Creates targets pointing to Service ClusterIP addresses (`PUT /resource/{resourceId}/target`)
   - **One target per HTTPRoute rule** (not per backend) with routing properties: path, pathMatchType, priority
   - Periodic reconciliation every 5 minutes to verify resource still exists
   - Auto-recreates resource if deleted from Pangolin
   - SSO configuration: Controlled via annotation `gateway.pangolin.net/disable-sso=true` (default: false/enabled)

5. **GRPCRoute Reconciler** (`pkg/controller/grpcroute_controller.go`):
   - Watches `GRPCRoute` resources for TCP/UDP services
   - Creates Pangolin resources with `http:false` and `protocol:"tcp"|"udp"`
   - Uses annotation `gateway.pangolin.net/protocol` to specify tcp or udp (default: tcp)
   - Same target/resource management as HTTPRoute
   - Periodic reconciliation every 5 minutes
   - Auto-recreates resource if deleted from Pangolin

### Data Flow Pattern

```
K8s Gateway → Pangolin Newt Site (via PickSiteDefaults API)
               ↓
          Site credentials (NEWT_ID, NEWT_SECRET, subnet, exitNode)
               ↓
          Secret: {gateway-name}-newt-cred (PANGOLIN_ENDPOINT, NEWT_ID, NEWT_SECRET)
               ↓
          Newt Deployment (WireGuard VPN) + ClusterIP Service
               ↓
          VPN tunnel established to Pangolin exit node
```

**Endpoint Configuration:**
- `PANGOLIN_BASE_URL` (e.g., `https://api.example.com/v1`) - Integration API for controller
- `NEWT_ENDPOINT` (auto-derived to `https://pangolin.example.com`) - VPN authentication endpoint
- Logic: Strip `api.` prefix, add `pangolin.` prefix, remove `/v1` path

## Development Workflow

### Essential Commands
```bash
# Run locally (disables leader election for dev)
export PANGOLIN_API_KEY=xxx PANGOLIN_ORG_ID=xxx
export PANGOLIN_BASE_URL=https://api.example.com/v1
export ENABLE_LEADER_ELECTION=false LOG_LEVEL=debug
make run

# Build & test
make build          # Builds to bin/controller
make test           # Run unit tests
make docker-build   # Build container image (IMG=name:tag)

# Deploy to kind cluster
make install        # Install Gateway API CRDs
kind load docker-image pangolin-gateway-controller:latest
make deploy IMG=pangolin-gateway-controller:latest
```

### Testing Pattern (Kind + Self-Hosted Pangolin)
1. Start kind cluster: `kind create cluster`
2. Deploy controller: `make docker-build && kind load docker-image ... && make deploy`
3. Deploy nginx test app: `kubectl apply -f deploy-nginx-app.yaml`
4. Create Gateway: `kubectl apply -f examples/gateway.yaml`
5. Check newt VPN logs: `kubectl logs -l app.kubernetes.io/name=newt`
   - Success: "Tunnel connection to server established successfully!"
   - Common errors: 401 (bad credentials), 405 (wrong endpoint)
6. Verify site online: `curl -H "Authorization: Bearer $PANGO_KEY" https://api.example.com/v1/org/home/sites | jq '.data.sites[] | {name, online}'`

## Project-Specific Conventions

### Gateway API Version Requirements
- **Gateway API v1.4.1** (updated from v1.0.0)
- GRPCRoute now in `v1` (was `v1alpha2` in earlier versions)
- HTTPRoute and Gateway remain in `v1`
- **CRITICAL**: When updating Gateway API versions, check if alpha resources moved to stable
- Import paths: Use `gatewayv1 "sigs.k8s.io/gateway-api/apis/v1"` for all v1 resources

### Controller Name Uniqueness
- **Multiple controllers watching same resource type MUST have unique names**
- Gateway controller: `.Named("gateway-site")` - manages Pangolin site creation
- Newt controller: `.Named("gateway-newt")` - manages VPN deployment
- Prevents controller-runtime conflicts when multiple reconcilers watch Gateway resources
- Pattern: `ctrl.NewControllerManagedBy(mgr).For(&Resource{}).Named("unique-name").Complete(r)`

### Resource Idempotency Pattern
```go
// HTTPRoute controller checks for existing resources BEFORE creating
func (r *HTTPRouteReconciler) findExistingResourceBySubdomain(ctx, hostname, log) (string, error) {
    // 1. List all resources via ListResources()
    // 2. Match by resource name field (full hostname like "test.dev0ps.me")
    // 3. Return resourceID if found, empty string if not
}

// In reconcileHTTPRoute:
resourceID, _ := r.findExistingResourceBySubdomain(ctx, hostname, log)
if resourceID == "" {
    // Create new resource using hostname as name
    resourceID = r.createPangolinResourceForHostname(ctx, route, gateway, hostname, hostname, log)
} else {
    // Use existing resource (shared across HTTPRoutes)
}
```
- **Why**: One Pangolin resource per hostname, shared by multiple HTTPRoutes with same hostname
- **Pattern**: Check existence by resource name (full hostname), not K8s labels
- Resources no longer stored in HTTPRoute labels (they're shared)

### Site Naming & Deletion
- Sites named `pgc-{gateway-name}` (not namespace-based)
- Finalizer ensures site deletion: `DeleteSite(siteID)` called before Gateway removal
- Site ID stored in label, parsed with `strconv.Atoi()`

### Credential Preservation (CRITICAL BUG FIX)
```go
// WRONG - CreateSite doesn't return credentials
createdSite, _ := client.CreateSite(ctx, site)
secret.Data["NEWT_ID"] = createdSite.NewtID // Empty!

// CORRECT - Preserve from PickSiteDefaults
defaults, _ := client.PickSiteDefaults(ctx)
site := &Site{NewtID: defaults.NewtID, Secret: defaults.NewtSecret}
createdSite, _ := client.CreateSite(ctx, site)
createdSite.NewtID = site.NewtID  // Preserve!
createdSite.Secret = site.Secret
```

### Endpoint Auto-Derivation
```go
// In config.ApplyDefaults()
if c.Controller.NewtEndpoint == "" {
    parsed, _ := url.Parse(c.Pangolin.BaseURL)
    host := parsed.Hostname()
    host = strings.TrimPrefix(host, "api.") // Remove api. prefix
    c.Controller.NewtEndpoint = fmt.Sprintf("%s://pangolin.%s", parsed.Scheme, host)
}
```

### Status Updates Pattern
- Update Gateway status with conditions: `Programmed`, `Accepted`
- HTTPRoute status updates per parent: `RouteParentStatus` with `ControllerName: pangol.in/gateway-controller`
- Always set `ObservedGeneration` to detect stale status
- **IMPORTANT**: Use `metav1.Condition`, NOT `gatewayv1.Condition` (Gateway API v1.0.0+)

### Reconciliation Idempotency
- Check if Pangolin resource exists before creating:
  1. Search by resource name (full hostname) via `ListResources()`
  2. If found, use existing resourceID (shared resource pattern)
  3. If not found, create new resource with hostname as name
- **Target Management (Path-based Ownership)**:
  - Each HTTPRoute only manages targets matching its rule paths
  - Orphaned target cleanup: Only delete targets whose path matches current HTTPRoute's paths
  - Prevents deletion loops when multiple HTTPRoutes share same resource
  - Example: HTTPRoute A manages `/api`, HTTPRoute B manages `/extra` on same resource
- **Hostname Filtering**:
  - Skip hostnames that don't match any Pangolin domain
  - Logs: "Skipping hostname that doesn't match any Pangolin domain"
  - Prevents invalid resources like "test.example.com.dev0ps.me"
  - Status: Set to False with "NoMatchingDomains" when all hostnames filtered
- Finalizers ensure Pangolin resources cleaned up before K8s resource deletion
- For resource deletion: Only delete resource when all targets removed (shared by multiple HTTPRoutes)

### Error Handling Strategy
- Transient errors: requeue with `ctrl.Result{RequeueAfter: 30 * time.Second}`
- Permanent errors: update status condition with error message, don't requeue
- API errors: log but continue, will retry on next reconciliation

### Logging Strategy
- **Info level** (default): Reconciliation events, errors, major state changes
  - "Reconciling HTTPRoute", "Successfully reconciled HTTPRoute"
  - "Created Pangolin resource", "Deleting HTTPRoute targets"
  - "Skipping hostname that doesn't match any Pangolin domain"
  - Error conditions and failures
- **V(1) Debug level**: Verbose details for troubleshooting
  - "Using existing Pangolin resource", "Target already exists with correct configuration"
  - "Checking domains for hostname", "Extracted subdomain with dot"
  - "Found existing resource with matching hostname"
  - "Skipping target with non-matching path (from different HTTPRoute)"
  - Domain matching details, target drift detection
- Set via `LOG_LEVEL=debug` environment variable or `--zap-log-level=1` flag
- Use `log.V(1).Info()` for debug-level logs to reduce noise in production

## Configuration & Secrets

- **Environment Variables** (preferred for deployments):
  ```bash
  PANGOLIN_API_KEY, PANGOLIN_ORG_ID (required)
  PANGOLIN_BASE_URL=https://api.pangolin.net/v1 (Integration API)
  NEWT_ENDPOINT=https://api.pangolin.net (optional, auto-derived from BASE_URL)
  NEWT_IMAGE=docker.io/fosrl/newt:1.10.0
  GATEWAY_CLASS_NAME=pangolin
  WATCH_NAMESPACE="" (empty = all namespaces)
  LOG_LEVEL=info|debug
  ```

- **Config File** (for local dev): `--config config/config.example.yaml`
- Credentials stored in Secret: `pangolin-api-credentials` in `pangolin-system` namespace
- Newt credentials per Gateway: `{gateway-name}-newt-cred` in same namespace

## Integration Points

### Kubernetes APIs
- Uses controller-runtime v0.17.0 with Watch/Cache pattern
- Gateway API v1.4.1 (GRPCRoute in `v1`, HTTPRoute/Gateway in `v1`)
- **CRITICAL**: Use `metav1.Condition` for status, NOT `gatewayv1.Condition`
- RBAC requires: Gateway API resources, Secrets, Deployments, Services (full CRUD)
- Leader election via `coordination.k8s.io/leases` in pangolin-system namespace

### API Reference Sources
- **`references/newt/`** - newt VPN client source code (Go). Use this to verify websocket/auth endpoints and token request/response types.
- **`references/pangolin/server/routers/`** - Pangolin server router source (TypeScript). **Authoritative source for all API endpoints**, request schemas, and response structures.
  - `integration.ts` — all Integration API routes (Bearer token)
  - `external.ts` — all UI/external API routes (session cookie)
  - `resource/`, `target/`, `site/`, `newt/` — handler implementations with Zod schemas

### Pangolin Integration API Endpoints
All confirmed from `references/pangolin/server/routers/integration.ts`. Bearer token auth, prefix `/v1`.

**Site endpoints:**
- `GET /org/{orgId}/pick-site-defaults` - Auto-allocate subnet, exit node, credentials
- `PUT /org/{orgId}/site` - Create site (doesn't return credentials!)
- `GET /org/{orgId}/sites` - List sites (note: plural) — response: `{"data":{"sites":[...]}}`
- `GET /org/{orgId}/site/{niceId}` - Get site by niceId
- `GET /site/{siteId}` - Get site by numeric ID
- `POST /site/{siteId}` - Update site
- `DELETE /site/{siteId}` - Delete site (no org in path)

**Resource endpoints:**
- `GET /org/{orgId}/resources` - List all resources — response: `{"data":{"resources":[...],"pagination":{...}}}`
  - Each resource includes embedded `targets[]` array with `{targetId, ip, port, enabled, healthStatus}`
- `PUT /org/{orgId}/resource` - Create resource — response: `{"data": Resource}`
- `PUT /org/{orgId}/site/{siteId}/resource` - Create resource scoped to site
- `GET /resource/{resourceId}` - Get resource by numeric ID
- `POST /resource/{resourceId}` - Update resource fields (sso, skipToIdpId, name, subdomain, ssl, blockAccess, domainId, enabled, stickySession, maintenanceModeEnabled, maintenanceModeType, maintenanceTitle, maintenanceMessage, maintenanceEstimatedTime, postAuthPath)
- `DELETE /resource/{resourceId}` - Delete resource
- `POST /resource/{resourceId}/roles` - Set allowed roles (empty array = no role restrictions)

**Site-Resource endpoints** (different from Resource — links a resource to a site):
- `PUT /org/{orgId}/site-resource` - Create site-resource
- `GET /org/{orgId}/site-resources` - List all site-resources for org
- `GET /org/{orgId}/site/{siteId}/resources` - List resources for a site
- `GET /site-resource/{siteResourceId}` - Get site-resource
- `POST /site-resource/{siteResourceId}` - Update site-resource
- `DELETE /site-resource/{siteResourceId}` - Delete site-resource
- `POST /site-resource/{siteResourceId}/roles` - Set roles on site-resource

**Target endpoints:**
- `PUT /resource/{resourceId}/target` - Create target — schema: `{siteId:int, ip:string, port:int, enabled:bool, path?, pathMatchType?, rewritePath?, rewritePathType?, priority?, hcEnabled?, ...}`
- `GET /resource/{resourceId}/targets` - List targets — response: `{"data":{"targets":[...]}}`
- `GET /target/{targetId}` - Get target
- `POST /target/{targetId}` - Update target
- `DELETE /target/{targetId}` - Delete target

**Resource routing rules:**
- `PUT /resource/{resourceId}/rule` - Create routing rule
- `GET /resource/{resourceId}/rules` - List rules
- `POST /resource/{resourceId}/rule/{ruleId}` - Update rule
- `DELETE /resource/{resourceId}/rule/{ruleId}` - Delete rule

**Domain/other:**
- `GET /org/{orgId}/domains` - List domains — response: `{"data":{"domains":[...]}}`

All endpoints: JSON payloads, `{"data": {...}}` response wrapper, `Authorization: Bearer <key>` header.

### Pangolin External (UI) API Endpoints
Served at `pangolin.example.com/api/v1` (session cookie auth). **Not accessible via Integration API key.**
- `GET /api/v1/server-info` - Returns `{version, supporterStatusValid, build, enterpriseLicenseValid, enterpriseLicenseType}` — requires session cookie
- `POST /api/v1/auth/newt/get-token` - Newt VPN authentication; returns `{data:{token, serverVersion}}` — **no auth required!** This is how `GetServerVersion()` works.
- All normal UI actions (auth, user management, etc.)

**Integration API Limitations:**
- **TWO SEPARATE APIS**: Integration API (`api.example.com/v1`) vs UI API (`pangolin.example.com/api/v1`)
  - Integration API: Bearer token auth, used by controller
  - UI API: Session cookie auth, used by web interface
  - Different capabilities and endpoints!
- **No version endpoint on Integration API** — use `GetServerVersion()` (calls newt auth token endpoint) to detect server version
- ❌ `PATCH /resource/{resourceId}` - Not supported in Integration API (only in UI API)
- ❌ `GET /v1/version` or similar - Does not exist on Integration API
- **Resource schema** (createResource): `{name:string, http:boolean, protocol:"tcp"|"udp", domainId:string, subdomain?:string, stickySession?:boolean, postAuthPath?:string}`
  - For raw (non-HTTP) resources: `{name:string, http:false, protocol:"tcp"|"udp", proxyPort:int}` (requires `allow_raw_resources` flag)
- **Pangolin API Version Compatibility**:
  - **Tested with**: Pangolin 1.16.2 (latest as of March 2026)
  - **Reference source**: `references/pangolin/` contains server source code at tag `1.16.2`
  - **Breaking change in 1.16.0**: `GET /org/{orgId}/sites` and `GET /org/{orgId}/resources` changed pagination params:
    - Old (≤1.15.x): `?limit=1000&offset=0` — default was 1000 items
    - New (1.16.0+): `?pageSize=20&page=1` — default dropped to **20 items**
    - Our client passes `?pageSize=1000&page=N` and paginates through all pages (see `listPageSize`/`listMaxPages` constants)
    - Backward-compatible: old servers ignore unknown query params and return their default
  - `verifyLimits` middleware on mutating endpoints (SaaS builds only)
- **SSO Configuration** (verified in Pangolin 1.15.4):
  - **Working in Integration API**: `POST /resource/{resourceId}` with `{"sso":false,"skipToIdpId":null}`
  - Controller implementation (v14+): Directly disables SSO via POST endpoint
  - Use annotation `gateway.pangolin.net/disable-sso=true` to disable SSO on resource creation
- **Target schema**: `{siteId:int, ip:string, port:int, enabled:boolean, ...health check fields}`
  - Use ClusterIP or Pod IP, not DNS names (newt may not resolve cluster DNS)
- Client code includes wrapped response handling (`{"data": {...}}`) for all endpoints

## Common Pitfalls

1. **Missing GatewayClass filter**: Always check `gateway.Spec.GatewayClassName == r.ControllerClass`
2. **Credential loss**: CreateSite doesn't return credentials - preserve from PickSiteDefaults response
3. **Wrong endpoint for newt**: Newt connects to `pangolin.example.com`, not `api.example.com/v1`
4. **Finalizer deadlock**: Remove finalizer AFTER cleaning up Pangolin resources, not before
5. **Service FQDN**: Always use cluster DNS format: `{service}.{namespace}.svc.cluster.local`
6. **RBAC permissions**: Need Secrets, Deployments, Services - not just Gateway API resources
7. **Newt container capabilities**: Requires `NET_ADMIN` for WireGuard tunnel creation
8. **Resource existence**: Always check `ListResources()` before creating to avoid 409 conflicts
9. **Controller name conflicts**: Multiple controllers watching same resource need unique names via `.Named()`
10. **Port mismatches**: Container images have default ports (nginx:80, not 8080) - must match service targetPort
11. **Shared resource deletion loops**: When multiple HTTPRoutes share a resource, only delete targets matching current HTTPRoute's paths
12. **Invalid hostnames**: Filter hostnames that don't match Pangolin domains to prevent resources like "test.example.com.dev0ps.me"

## Key Files Reference
- Entry point: `cmd/controller/main.go` (controller-runtime setup with all 3 reconcilers)
- Constants: `pkg/controller/gateway_controller.go:23-30` (finalizer names, labels)
- API types: `pkg/pangolin/client.go:36-140` (Site, SiteDefaults structs)
- Config logic: `pkg/config/config.go:145-165` (newt endpoint auto-derivation)
- Newt deployment: `pkg/controller/newt_controller.go:180-280` (Secret, Deployment, Service builders)
- RBAC: `config/rbac.yaml` (includes Secrets, Deployments, Services permissions)

## Debugging Checklist

1. **Newt VPN won't connect?**
   - Check Secret has all 3 values: `kubectl get secret {gateway}-newt-cred -o yaml`
   - Check PANGOLIN_ENDPOINT value: Should be `https://pangolin.example.com`, not `https://api.example.com/v1`
   - Check newt logs: `kubectl logs -l app.kubernetes.io/name=newt --tail=50`
   - Common errors: 401 (bad credentials), 405 (wrong endpoint), 404 (DNS issue)

2. **Site not created?**
   - Check controller logs: `kubectl logs -n pangolin-system -l app=pangolin-gateway-controller`
   - Verify API credentials: `echo $PANGOLIN_API_KEY | cut -c1-20` (should not be empty)
   - Test API manually: `curl -H "Authorization: Bearer $KEY" https://api.example.com/v1/org/home/sites`

3. **Gateway stuck in non-Programmed state?**
   - Check Gateway status: `kubectl describe gateway {name}`
   - Look for condition messages with error details
   - Verify GatewayClass exists: `kubectl get gatewayclass pangolin`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dxas90)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dxas90)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
