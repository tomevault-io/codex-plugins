# Copilot Instructions — cornyhorse-debian-python

## Project overview
This repository builds and publishes **PGO+LTO optimized CPython** Docker
base images to GitHub Container Registry (`ghcr.io/cornyhorse/python-base`).
Images are built for **Python 3.12, 3.13, and 3.14** on **linux/amd64** and **linux/arm64**.
The images are consumed by the **trombbone** project as:

```dockerfile
FROM ghcr.io/cornyhorse/python-base:3.14
```

## Repository layout

```
Dockerfile                     # Two-stage build: builder → runtime
.dockerignore
.github/
  workflows/
    nightly.yml                # Weekly (amd64) + monthly (amd64+arm64) + push + manual
    release.yml                # Tag-triggered release build (v3.12.*, v3.13.*, v3.14.*)
README.md
```

## Image conventions
- Python is installed to **`/opt/python`**; `/opt/python/bin` is on `PATH`.
- pip, setuptools, and wheel are pre-installed.
- Environment: `PYTHONUNBUFFERED=1`, `PYTHONDONTWRITEBYTECODE=1`, `LANG=C.UTF-8`.
- Base OS: Debian Bookworm (slim).
- No compiler toolchain in the final image — only runtime shared libraries.
- Multi-arch: `linux/amd64` and `linux/arm64` (arm64 via QEMU on GitHub Actions).

## Dockerfile build args
| Arg              | Default   | Purpose                        |
|------------------|-----------|--------------------------------|
| PYTHON_VERSION   | 3.14.2    | CPython release to compile     |
| DEBIAN_VERSION   | bookworm  | Debian suite for base image    |

## Workflow design
- **nightly.yml** (misnamed — actually weekly/monthly): runs weekly on Sunday
  at 04:00 UTC (amd64 only) and monthly on the 1st (amd64 + arm64).
  Also on push to `main` and `workflow_dispatch` (with platform choice).
  Uses a matrix to build 3.12, 3.13, and 3.14 in parallel.
  Detects the latest CPython patch for each minor from python.org FTP;
  skips versions already published on scheduled runs.
- **release.yml**: triggered by pushing a tag like `v3.14.2`, `v3.13.3`, or `v3.12.9`.
  Always builds and pushes for both amd64 and arm64.

Both workflows:
- Use `docker/build-push-action` with `linux/amd64,linux/arm64` platforms.
- Use QEMU via `docker/setup-qemu-action` for arm64 cross-compilation.
- Authenticate to ghcr.io with `GITHUB_TOKEN`.
- Use GitHub Actions cache (`type=gha`, scoped per minor version).
- Tag the image as `:<minor>`, `:<specific-version>`, and `:latest` (3.14 only).
- Add OCI `org.opencontainers.image.*` labels.

## Coding guidelines
- Keep the Dockerfile layer-efficient; combine `apt-get` commands, remove caches.
- In workflows, pin third-party actions to a major version tag (e.g., `@v4`).
- Use `${{ github.repository_owner }}` in image references to remain fork-friendly.
- When editing the Dockerfile, preserve the two-stage pattern and `/opt/python` path.
- All shell in workflows should use `set -euo pipefail`.
- Prefer `curl` over `wget` in workflows for consistency with the Dockerfile.
- When adding a new Python minor version, update the matrix in nightly.yml and the
  tag patterns in release.yml.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cornyhorse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
