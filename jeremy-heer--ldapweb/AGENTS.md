# LDAP Browser - Copilot Coding Agent Instructions

## Project Overview

**LDAP Browser** is a comprehensive Java web application for browsing, searching, and managing LDAP directories. Built with Spring Boot and Vaadin, it provides a modern interface featuring both directory management and schema exploration capabilities.

# LDAP Browser — Copilot coding agent instructions

This file summarizes repository structure, build steps, and common
change points. Lines are short and statements are concise to match
Google style conventions used by the project's Checkstyle setup.

## Project at a glance

- Project: LDAP Browser (Spring Boot + Vaadin)
- Language: Java 17+
- LDAP client: UnboundID LDAP SDK
- Build: Maven

Primary deliverable: self-contained JAR runnable on Java 17+.

## Quick build & run

Work in the repository root when running these commands.

- Clean: `mvn clean`
- Compile: `mvn compile`
- Tests: `mvn test`
- Checkstyle: `mvn checkstyle:check` (plugin uses Google checks)
- Production package: `mvn clean package -Pproduction -DskipTests`
- Dev reload: `./dev-reload.sh` or
    `mvn spring-boot:run -Dspring.profiles.active=development`

Notes:
- `mvn clean` before production builds is recommended.
- Checkstyle warnings are expected; the Maven config does not fail
    the build on violations.

## Where to change things

- LDAP operations: `src/main/java/.../service/LdapService.java`
- Server configuration: JSON files and
    `src/main/java/.../model/LdapServerConfig.java`
- UI components: `src/main/java/.../ui/components`
- Application entry: `src/main/java/.../LdapBrowserApplication.java`

## Project layout (abbreviated)

```
src/main/java/...     # application code (models, services, ui)
src/main/resources    # application.properties and profiles
src/main/bundles      # pre-built Vaadin frontend assets
test/                 # unit and UI tests
```

## Build validation checklist

1. `mvn compile` — verify compilation.
2. `mvn test` — run unit tests.
3. `mvn checkstyle:check` — run style checks.
4. `mvn clean package -Pproduction -DskipTests` — produce release JAR.

## Quality and style hints

- Follow Google Java style for new Java files.
- Keep method and class Javadoc for public APIs.
- Limit line length to 100 chars in Java files.
- Organize imports lexicographically.

## Quick troubleshooting

- Missing dependencies: try `mvn clean compile`.
- Vaadin frontend build errors: check Node toolchain and
    frontend assets in `bundles/`.

## Small change workflow

1. Implement code and tests.
2. `mvn compile` and `mvn test` locally.
3. Run `mvn checkstyle:check` to review style warnings.
4. Create a concise commit and push.

---

This document is intentionally short and formatted to keep lines
under 100 characters and to match the project's Checkstyle expectations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jeremy-Heer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
