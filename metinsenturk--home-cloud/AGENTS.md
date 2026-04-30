# Project Context: The Mini-Cloud (Traefik + Docker Compose)

You are an expert DevOps engineer assisting with a modular self-hosted infrastructure. 
The project uses a "Base + App" folder strategy to allow launching subsets of services.

## Architecture Principles
- **Reverse Proxy:** Traefik v3 (lives in `/apps/traefik`).
- **Networking:** All public-facing containers must connect to the external bridge network `home_network`.
- **Modularity:** Each application must live in its own subdirectory under `/apps`.
- **Routing:** Use Subdomains (e.g., `appname.${DOMAIN}`) via Traefik labels, NOT subpaths.
- **Service Discovery:** Traefik watches the Docker socket. Labels are the source of truth for routing.
- **Documentation:** Every service MUST include clear comments explaining WHY a setting is used.
- **Workflow:** ALWAYS provide a high-level plan or "Action List" before providing code.

## Infrastructure & Shared Services Pattern
- **Service Naming (Dashes):** Use `infra-` prefix with dashes (e.g., e.g., `infra-postgres`, `infra-mssql`). This acts as the internal DNS hostname.
- **Folder Naming (App Name Only):** Infrastructure app folders should use only the app name, with no `infra_` prefix/suffix pattern (e.g., `postgres`, `mssql`, `redis`).
- **Container Naming (Underscores):** Use `infra_` prefix with underscores (e.g., `infra_postgres`, `infra_mssql`).
- **Persistence & Networking (Underscores):** Strictly follow these patterns:
    - Network: `home_infra_<name>_network`
    - Volume: `home_infra_<name>_data`
- **Connectivity:** - Always attach to `home_network` for cross-app discovery.
    - Set `traefik.enable=false` (Infrastructure is accessed via Service Name, not URLs).
- **Local Environment Variables:** Each app must define its own environment variables in its local `.env` file. If a variable's value should come from a global variable, reference it in the local `.env` file (e.g., `APP_TZ=${TZ}`), not directly in the compose file.

## Service Lifecycle & Discovery
- **New Service Protocol:** Every new application is treated as a modular "plug-in" to the existing Mini-Cloud. It must be research-validated (images, ports, entrypoints) before code generation.
- **Task-Specific Execution:** Detailed scaffolding of new apps is handled via the `/add-app` prompt. Always refer to `.github/prompts/add-app.prompt.md` for the multi-step research and confirmation workflow.
- **Binding Rule:** Services must never listen on `127.0.0.1` inside the container; they must bind to `0.0.0.0` to ensure the Traefik proxy on the `home_network` can reach them.
- **Connectivity Check:** If a service has multiple networks, the Traefik routing label MUST explicitly specify `traefik.docker.network=home_network` to prevent 504 Gateway Timeouts.

## Standard Docker Compose Pattern
When generating a new app in `/apps/<name>/docker-compose.yml`, always follow this template:

1. **External Networking:** Connect the entry-point container to `home_network`.
2. **Internal Networking:** Use a private bridge for internal service-to-service talk if needed.
3. **Labels:** Add Traefik labels to the entry-point container for routing:
   - `traefik.enable=true`
   - `traefik.http.routers.<name>.rule=Host(`<name>.${DOMAIN}`)`
   - `traefik.http.services.<name>.loadbalancer.server.port=<port>`
4. **Multi-Network Routing:** If a container is connected to more than one network, you **MUST** explicitly tell Traefik which network to use for routing to avoid 504 errors.
   - **Label:** `- "traefik.docker.network=home_network"`
5. Avoid using version key in `docker-compose.yml` (use the latest syntax).
6. Avoid hardcoding ports in the compose file; rely on Traefik for routing.
7. **Ordering:** Follow the strict key order: `image`, `container_name`, `restart`, `networks`, `volumes`, `labels`, `environment`, `logging`, `healthcheck`, `depends_on`, `env_file`.
8. **Clean Interpolation:** Use `${VARIABLE}` without hardcoded fallbacks. 
    - **Local Variables Only:** Compose files must only reference variables defined in the app's local `.env` file. Never reference global variables directly in compose files.
    - **Global Variable Mapping:** If an app needs a global variable (like `TZ` or `DOMAIN`), define it in the app's local `.env` file by referencing the global variable (e.g., `MY_APP_TZ=${TZ}`).
    - **Documentation:** In the app's `.env.example` file, clearly document which variables are set from global variables with comments (e.g., `MY_APP_TZ=${TZ}  # Set from global TZ variable`).
9. **Network Attachment:** Public services must use `home_network`. Internal communication must use `home_<appname>_network`.
10. **Explicit Naming**: To prevent Docker Compose from adding folder-name prefixes to resources, always use the `name:` attribute for local networks and volumes. See example below.
```yaml
networks:
  home_myapp_network:
    name: home_myapp_network
    driver: bridge
```
11. **Logging:** Always include log-rotation (max-size: 10m) for every service. Json logging is preferred for better integration with Dozzle.
12. **Healthchecks:** Always include a healthcheck for each service to allow Traefik to detect unhealthy containers and avoid routing to them. Use the `interval`, `timeout`, `retries`, and `start_period` options to fine-tune the healthcheck behavior.
13. **Documentation:** Include comments in the `docker-compose.yml` explaining the purpose of each setting, especially for non-obvious configurations. Comments about global architectural decisions (`home_network`, Traefik labels, Global environment variables, etc.) should be included in the root README rather than individual compose files to avoid redundancy.

## Healthcheck Standards
- **Native Tools First:** Always prefer built-in binary utilities over `curl` or `wget`.
    - **PostgreSQL:** Use `pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB`.
    - **MSSQL:** Use `/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P $$SA_PASSWORD -Q "SELECT 1" -C`.
    - **Redis:** Use `redis-cli ping`.
    - **MySQL/MariaDB:** Use `mysqladmin ping -h localhost -u $$MYSQL_USER -p$$MYSQL_PASSWORD`.
- **Fallback to Shell Built-ins:** If no native tool or `curl` is available, use TCP socket checks:
    - `test: ["CMD-SHELL", "exec 3<>/dev/tcp/127.0.0.1/<port> || exit 1"]`
- **Configuration:** Always include `start_period` to allow for initialization (especially for databases).

# Naming Conventions

- **Folder Names:** Use lowercase letters and underscores. If the app name is multiple words, use underscores (e.g., `myapp`, `my_app`).
- **Infra Folder Names:** Use the plain app name only (e.g., `postgres`, `mssql`, `redis`). Do not use an `infra_` prefix/suffix pattern for folder names.
- **Service Names:** Use lowercase letters and hyphens (e.g., `myapp`). If multiple services are needed, use `<appname>-<purpose>` (e.g., `myapp-db`).
- **Container Names:** Follow the pattern `<appname>` (e.g., `myapp`). If multiple containers are needed, use `<appname>_<purpose>` (e.g., `myapp_db`).
- **Network Names:** Use `home_<appname>_network` for internal networks (e.g., `home_myapp_network`).
- **Volume Names:** Use `home_<appname>_data` for persistent storage (e.g., `home_myapp_data`). If multiple volumes are needed, use `home_<appname>_<purpose>` (e.g., `home_myapp_db_data`).
- **Env Variables:** Use uppercase letters with underscores (e.g., `MYAPP_ENV_VAR`). For global variables, use a clear prefix (e.g., `HOME_CLOUD_` for global credentials, `INFRA_` for infrastructure secrets).

# Coding Style

- When defining networks, define `home_network` first, then internal networks.
- When defining volumes, define `home_<appname>_data` first, then any additional volumes.

## Makefile Integration
* **App-Specific Targets:** Every new app must have its own `up-` and `down-` targets.
* **No Top-Level Variables:** Do not define app paths as variables at the top. Use the direct path within the command.
* **The "Double-Env" Pattern:** All app targets must explicitly load the root `.env` first, then the local `.env`, meaning the app-specific values will override globals.
* **Standard Format:**
```makefile
.PHONY: up-appname
up-appname: create-network
	docker compose --env-file .env --env-file apps/appname/.env -f apps/appname/docker-compose.yml up -d

.PHONY: down-appname
down-appname:
	docker compose --env-file .env --env-file apps/appname/.env -f apps/appname/docker-compose.yml down

```
* **Orchestration:** Add new apps to the `up-all` and `down-all` aggregate targets.

## Port Registry (PORTS.md)
- **Purpose:** A centralized log of all host-accessible ports to prevent conflicts.
- **Central Registry:** All ports exposed to the host MUST be documented in `/PORTS.md`.
- **Content:** Includes the host port, the service name, the internal container port, and a brief description.
- **Update Rule:** When adding a new host-exposed port, append a single row to the existing table in `/PORTS.md`. Do not rewrite the file or add extra commentary.
- **Structure:** 
   - `# Host Ports`: Brief description of the file's purpose.
   - `## Ports Table`: A Markdown table with columns: `Host Port`, `Service Name`, `Internal Port`, `Description`.
   - `## Notes`: A list of specific operational notes referencing the table entries.
- **Mapping Style:** Prefer consistent mapping (e.g., `5432:5432`) unless a conflict exists on the host.
- **Conflict Prevention:** Before assigning a host port to a new service, check `/PORTS.md` to ensure it is not already in use.

## Environment Variable Interpolation
- **Host-Level ($):** Use a single dollar sign for values that Docker Compose must resolve from `.env` files at startup (e.g., image tags, volume paths, or `environment:` definitions).
- **Container-Level ($$):** Use a double dollar sign for variables that must be evaluated by the container's shell at runtime. 
    - **MANDATORY:** Always use `$$` in `healthcheck` commands (e.g., `$$POSTGRES_USER`) to ensure the container uses its own internal environment.

## Global Environment Dictionary
- **Standardization:** All global variables are defined in the root `.env` and documented in the root `.env.example`.
- **Use of Global Variables:** Apps must define their own variables in their local `.env` file. To use a global variable's value, reference it in the local `.env` file (e.g., `APP_TIMEZONE=${TZ}`, `APP_DOMAIN=${DOMAIN}`).
- **Inheritance:** Apps access global variables via the Makefile's "Double-Env" pattern, which loads the root `.env` first, then the app's `.env`. This allows app variables to reference and override globals.
- **Variables:** The full list is defined in the root `.env.example`. Some key variables include:
    - `DOMAIN`, `TZ`, `PUID`, `PGID`
    - `HOME_CLOUD_EMAIL`, `HOME_CLOUD_PASSWORD`
    - `INFRA_POSTGRES_PASSWORD`, `OPENAI_API_KEY`, etc.
- **SMTP Variables:** When an app requires mail settings, map its local variables to the HOME_CLOUD_SMTP_* suite.
    - Note: Always check if the app requires a "STARTTLS" vs "SSL/TLS" setting and match it to HOME_CLOUD_SMTP_SECURE.

## Secret Management
- **Global Secrets:** Store in root `.env` for shared services (e.g., `INFRA_POSTGRES_PASSWORD`).
- **App Secrets:** Store in `apps/<name>/.env` for unique, service-specific credentials.
- **Reference Pattern:** Compose files should use variable substitution (e.g., `${SECRET_VAR}`) rather than hardcoded strings.
- **Priority:** The Makefile loads root `.env` then app `.env`. App-local values ALWAYS take priority over globals.
- **Usage:** Services must use global secrets defined in the root `.env` for any variables that are shared across multiple apps (e.g., database passwords). App-specific secrets should only be defined in the local `.env` if they are unique to that app.

## Tech Stack Preferences
- **Logging:** Centralized via Dozzle (`logs.localhost`).
- **Updates:** Managed by What's Up Docker (WUD) (no manual pulls).
- **Environment:** Use `${TZ}` and `${DOMAIN}` variables.
- **Constraints:** Always include log-rotation (max-size: 10m) for every service.

---
> Source: [metinsenturk/home-cloud](https://github.com/metinsenturk/home-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
