# Copilot Instructions - Thrive WP (WordPress + NestJS Hybrid)

## Architecture Overview
This is a **hybrid WordPress + NestJS application** running in Docker containers. WordPress serves as the frontend/CMS, while NestJS provides modern backend API capabilities. Communication between services happens via HTTP calls within the Docker network.

## Infra - Important!
This entire app must be able to be deployed to a single VPS. This means all services must be accessible within the VPS without relying on public IPs or external DNS.

### Key Services & Ports
- **WordPress**: `localhost:8080` (PHP/Apache container)
- **NestJS API**: `localhost:3000` (Node.js container) 
- **MariaDB**: Internal port 3306 (database container)
- **Network**: All services communicate on `wordpress_net` bridge network

### File Upload Configuration
File uploads require proper configuration at multiple layers. See `docs/wordpress-upload-system.md` for full details.

**Key files that configure upload limits:**
- `Dockerfile` - PHP settings (`upload_max_filesize`, `post_max_size`, `memory_limit`)
- `nginx/nginx.conf/default.conf` - Nginx settings (`client_max_body_size`)

**Current limits:** 50MB for uploads, 512MB PHP memory for image processing.

**Frontend image resizing:** The frontend (`apps/wordpress/themes/custom-theme/lib/upload-utils.ts`) automatically resizes images before upload to reduce file size.

## Critical Patterns

### WordPress-NestJS Bridge Communication
- WordPress calls NestJS via `http://nestjs:3000/` (Docker internal hostname)
- External API access via `http://localhost:8080/api/` (through Nginx proxy)
- NestJS endpoints follow REST conventions in `nestjs/src/app.controller.ts`
- **Important**: Most API endpoints require authentication via JWT token in `thrive_sess` cookie
- Admin-only endpoints use `AdminGuard` which requires user to have 'admin' role
- Test bridge with shortcode: `[test_nodejs_bridge]`
### Typing Standards
- `any` types are strictly forbidden.
- All API calls from the browser MUST be made with the shared thriveClient
- All API types MUST be declared in the shared package - this should cover most types used in - the client side code. There should be minimal type declarations in the client side apps. Use API types where possible.

### Database Table Names
- User table: `user` (singular, not `users`)
- Admin table: `admin`
- Teacher table: `teacher`
- Classes table: `classes`
- WordPress tables: `wp_*` prefix

### Docker Development Workflow
```bash
# Start entire environment
docker-compose --env-file .env.local up --build

# Rebuild specific service
docker-compose build wordpress  # or nestjs
docker-compose up -d wordpress

# View logs
docker-compose logs -f nestjs
docker-compose logs -f wordpress
```

### Plugin Development Pattern
- WordPress plugins mounted as volumes: `./wordpress/plugins/nodejs-bridge:/var/www/html/wp-content/plugins/nodejs-bridge`
- Live reload: Changes to PHP files reflect immediately (no container rebuild)
- Debug logs accessible in `_logs_wp/` directory
- Plugin structure follows WordPress conventions with main file + `includes/` directory

### NestJS Development Pattern  
- Standard NestJS structure in `nestjs/` directory
- Hot reload available via `pnpm run start:dev` (run inside container)
- Add new endpoints in `app.controller.ts` for WordPress integration
- Use Docker internal hostnames for service-to-service communication

## Environment Configuration
- Copy `.env.example` to `.env.local` before first run
- WordPress debug enabled by default in docker-compose.yml
- Site URLs configured via environment variables (not WordPress admin)

## Testing Integration Points
1. **WordPress Installation**: Visit `localhost:8080`, complete setup
2. **API Health Check**: Visit `localhost:3000` (should show "Hello World!")  
3. **Bridge Test**: Activate NodeJS Bridge plugin, use `[test_nodejs_bridge]` shortcode
4. **Verify Communication**: Check container logs for HTTP requests between services

## File Organization
- `wordpress/plugins/`: WordPress-specific code (PHP)
- `nestjs/src/`: NestJS API code (TypeScript)
- `docker-compose.yml`: Service orchestration (primary config)
- `GEMINI.md`: Architecture decisions and requirements (keep updated)
- `docs/thrive-modal-architecture.md`: Frontend React modal for blocks (authoring, triggers, runtime). Refer here for modal details.

## Service Types System

The application uses a centralized ServiceType system for consistent class/service type definitions:

### ServiceType Enum (`nestjs/src/common/types/class-types.ts`)
- **PRIVATE**: One-to-one individual sessions
- **GROUP**: Group classes with enrollment limits
- **COURSE**: Multi-session programs with structured curriculum

### ServiceKey Enum (for Stripe integration)
- **PRIVATE**: Maps to Stripe products for private sessions
- **GROUP_CLASS**: Maps to Stripe products for group classes
- **COURSE_CLASS**: Maps to Stripe products for courses

### Usage
```typescript
import { ServiceType, ServiceKey, serviceTypeToServiceKey } from '../common/types/class-types.js';

// Convert between types
const serviceKey = serviceTypeToServiceKey(ServiceType.PRIVATE); // Returns ServiceKey.PRIVATE_CLASS

// Runtime validation with Zod
import { ServiceTypeSchema } from '../common/types/class-types.js';
const result = ServiceTypeSchema.safeParse('PRIVATE');
```

### Consistency Rules
- All class types must use the ServiceType enum values
- Frontend types must match: `"PRIVATE" | "GROUP" | "COURSE"`
- Database stores enum values as strings
- Stripe product mappings use ServiceKey values
- All references must be consistent across NestJS, WordPress, and web components

## DTO Validation with nestjs-zod

The application uses `nestjs-zod` for streamlined DTO validation:

### Global Setup
```typescript
// src/app.module.ts
import { ZodValidationPipe } from 'nestjs-zod';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ZodValidationPipe,
    },
  ],
})
export class AppModule {}
```

### Zod DTOs
```typescript
export const CreatePaymentIntentSchema = z.object({
  start: z.string().datetime(),
  end: z.string().datetime(),
  teacher: z.number().int().positive(),
  serviceType: z.nativeEnum(ServiceType),
  notes: z.string().optional(),
});

export type CreatePaymentIntentDto = z.infer<typeof CreatePaymentIntentSchema>;
```

### Controller Usage
```typescript
@Post()
createPaymentIntent(
  @Body(new ZodValidationPipe(CreatePaymentIntentSchema))
  dto: CreatePaymentIntentDto,
): Promise<Response>
```

## Local WP Database
Credentials:
"MYSQL_HOST": "localhost",
"MYSQL_USER": "wordpress",
"MYSQL_PASSWORD": "wordpress",
"MYSQL_DATABASE": "wordpress"

You are able to query the database using the CLI like so: docker-compose exec db mysql -u wordpress -p wordpress wordpress.

Remember that page content is stored in the wp_posts table and user data is stored in the user table.

You will be required to update the WP site by making changes to the database. You should open the page at http://localhost:8080/ in your browser to check that the changes have been applied.

The page displayed at http://localhost:8080/ is the homepage of the WordPress site. This is the record with ID 5 in the wp_posts table.

# Makefile

Useful commands for running, stopping, migrating can all be found in `Makefile`.

# Typing !important
Use strict typing as much as possible across all apps.
- Avoid using 'any' type
- Use specific types for function parameters and return values
- Leverage TypeScript interfaces and types for complex data structures
- When you need to write javascript files, use JSDoc annotations to provide type information
- PHPStan has been set up for all PHP files, use it to analyze and improve your code quality

## Added: Unified Domain & Reverse Proxy Auth (Dev Enhancement)

An Nginx `web` service now fronts WordPress and NestJS to provide a single origin (http://localhost:8080) and inject authenticated user context into WordPress at render time.

Flow:
1. User authenticates via Google OAuth (NestJS `/auth/google`).
2. NestJS issues a short‑lived signed session cookie (`thrive_sess`, HS256, 30m) and redirects back to WordPress base URL.
3. Every page request hits Nginx → internal subrequest to `/auth/introspect` on NestJS.
4. Introspect validates the cookie and returns identity via `X-Auth-*` headers.
5. Nginx forwards headers to WordPress; plugin creates/loads corresponding WP user during `init` for template conditionals.

Config Vars:
- `SESSION_SECRET` (NestJS) – MUST override in production.
- `SESSION_COOKIE_NAME` (optional) – default `thrive_sess`.
- `WP_BASE_URL` – base URL used for post-login redirect.

Security Notes:
- Dev secret is insecure; replace before deploy.
- Cookie: HttpOnly, SameSite=Lax (adjust to `Strict` if compatible), `Secure` automatically when `NODE_ENV=production`.
- Revocation currently time-based; consider adding a denylist or session store for immediate logout if needed.

Next Potential Improvements:
- Refresh endpoint & silent renewal JS.
- Role synchronization.
- Optional Redis-backed session for instant revocation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davecook88)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/davecook88)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
