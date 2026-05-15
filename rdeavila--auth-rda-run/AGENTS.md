# Copilot Instructions for auth.rda.run

## Project Overview
This is a containerized authentication service using [pocket-id](https://github.com/pocket-id/pocket-id) with Cloudflare R2 storage for data persistence. The service runs in Alpine Linux with automatic R2 synchronization.

## Architecture
- **Main Service**: pocket-id binary (downloaded at build time)
- **Storage**: Cloudflare R2 bucket with bidirectional sync
- **Data Flow**: Local `/app/data` ↔ R2 bucket via rclone
- **Sync Strategy**: Initial pull on startup, push every 5 minutes via cron

## Key Components

### Container Setup ([Dockerfile](Dockerfile))
- Base: Alpine 3.22 with rclone, tini, curl
- Service binary: Downloaded from GitHub releases
- Entrypoint: Custom script handling R2 setup and sync

### R2 Integration ([entrypoint.sh](entrypoint.sh))
Required environment variables:
- `R2_ACCOUNT_ID`: Cloudflare account ID
- `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`: R2 credentials  
- `R2_BUCKET`: Target bucket name

The entrypoint:
1. Validates R2 environment variables
2. Configures rclone with R2 credentials
3. Syncs R2 → local on startup
4. Sets up cron job for local → R2 sync every 5 minutes
5. Launches pocket-id service

## Development Workflow

### Build and Run
```bash
podman build -t auth .
podman run -it --rm --name auth \
  -e R2_ACCOUNT_ID=your_account_id \
  -e R2_ACCESS_KEY_ID=your_key \
  -e R2_SECRET_ACCESS_KEY=your_secret \
  -e R2_BUCKET=bucket_name \
  -e APP_URL=https://pocket-id.domain.tld \
  -e MAXMIND_LICENSE_KE=your_maxmind_api_key \
  -e TRUST_PROXY=true \
  -e PORT=1411 \
  -e HOST=0.0.0.0 \
  -p 1411:1411 \
  auth
```

### Debugging
- Container logs show R2 sync operations
- Health check validates R2 connectivity
- Manual sync: `docker exec auth /usr/local/bin/r2sync-push`

## Conventions
- All scripts use `set -eu` for strict error handling
- R2 credentials map to AWS-compatible environment variables
- Background processes (crond) run with explicit logging levels
- Tini handles proper signal forwarding to pocket-id

---
> Source: [rdeavila/auth.rda.run](https://github.com/rdeavila/auth.rda.run) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-15 -->
