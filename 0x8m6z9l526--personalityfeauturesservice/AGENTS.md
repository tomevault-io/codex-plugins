# PersonalityFeauturesService Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-23

## Active Technologies
- Go 1.25.3 (within constitution minimum 1.22+) + net/http, chi, go-playground/validator, OpenTelemetry SDK, Prometheus client, `github.com/golang-jwt/jwt/v5` for token validation (001-define-customization-ux)
- PostgreSQL 14+ with `users` & `user_customizations`, optional Redis cache keyed by username/version (001-define-customization-ux)
- Go 1.25.3 (per repo toolchain; satisfies ≥1.22 mandate) + chi router, net/http, go-playground/validator/v10, OpenTelemetry SDK, Prometheus client, `github.com/golang-jwt/jwt/v5`, standard `mime/multipart` for uploads (no new helper dependency). (001-define-customization-ux)
- PostgreSQL 14+ (user_customizations table), Redis (optional username cache); media binaries live in the shared object-storage gateway (S3-compatible) with URLs persisted in Postgres. (001-define-customization-ux)
- Go 1.25.3 (repository toolchain; satisfies ≥1.22 requirement) + chi router, net/http, go-playground/validator/v10, OpenTelemetry SDK, Prometheus client, `segmentio/kafka-go` (new) (001-define-customization-ux)
- PostgreSQL 14+ with new `iam_user_registered_inbox` table (event_id PK, user_id FK to `users`, adapted payload JSONB); optional Redis cache unchanged (001-define-customization-ux)

- Go 1.22+ (call out and justify any deviation) + net/http, chi or echo, go-playground/validator, OpenTelemetry SDK, Prometheus client (001-define-customization-ux)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Go 1.22+ (call out and justify any deviation)

## Code Style

Go 1.22+ (call out and justify any deviation): Follow standard conventions

## Recent Changes
- 001-define-customization-ux: Added Go 1.25.3 (repository toolchain; satisfies ≥1.22 requirement) + chi router, net/http, go-playground/validator/v10, OpenTelemetry SDK, Prometheus client, `segmentio/kafka-go` (new)
- 001-define-customization-ux: Added Go 1.25.3 (per repo toolchain; satisfies ≥1.22 mandate) + chi router, net/http, go-playground/validator/v10, OpenTelemetry SDK, Prometheus client, `github.com/golang-jwt/jwt/v5`, standard `mime/multipart` for uploads (no new helper dependency).
- 001-define-customization-ux: Added Go 1.25.3 (within constitution minimum 1.22+) + net/http, chi, go-playground/validator, OpenTelemetry SDK, Prometheus client, `github.com/golang-jwt/jwt/v5` for token validation


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0x8m6z9l526) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
