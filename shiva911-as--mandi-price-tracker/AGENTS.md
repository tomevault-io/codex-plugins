# Copilot Instructions for MandiPriceTracker

## Project Overview
- This repository contains two similar Java Spring Boot applications:
  - `mandi-price-tracker/` (package: `com.example.mandi_price_tracker`)
  - `web-app/` (package: `com.mandi.pricetracker`)
- Both apps track and display mandi (market) prices, using a standard Spring Boot MVC structure.
- Each app has its own `pom.xml`, `src/`, and `target/` directories.

## Key Components
- **Controllers:**
  - `controller/PriceController.java` handles HTTP requests for price data and view rendering.
- **Templates:**
  - `src/main/resources/templates/prices.html` is the main UI for displaying prices.
  - `error.html` is used for error handling.
- **Static Assets:**
  - `static/styles.css` for UI styling.
- **Entry Point:**
  - `MandiPriceTrackerApplication.java` in each app is the Spring Boot entry point.

## Build & Run
- Use Maven Wrapper scripts for builds:
  - Windows: `mvnw.cmd clean package` (from the app root)
  - Linux/macOS: `./mvnw clean package`
- Run the app:
  - `java -jar target/*.jar` (after build)
- Tests:
  - Run `mvnw.cmd test` to execute unit tests (see `MandiPriceTrackerApplicationTests.java`).

## Conventions & Patterns
- **Spring Boot MVC:**
  - Controllers in `controller/`, templates in `templates/`, static files in `static/`.
- **No database integration** is visible; price data is likely hardcoded or loaded from files (check `PriceController.java`).
- **Error handling** is via `error.html` template.
- **No custom build/test scripts**—use Maven defaults.

## Integration & Dependencies
- Managed via each module's `pom.xml`.
- No external service integrations detected.

## Recommendations for AI Agents
- When adding features, mirror the structure in both `mandi-price-tracker/` and `web-app/` unless otherwise specified.
- Follow existing package naming conventions.
- Place new controllers in `controller/`, templates in `templates/`, and static assets in `static/`.
- Reference `PriceController.java` and `prices.html` for UI/data flow patterns.

---

_If any section is unclear or missing important project-specific details, please provide feedback for further refinement._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shiva911-as) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
