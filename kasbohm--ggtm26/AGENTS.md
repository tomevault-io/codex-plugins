# Copilot Instructions for Gåpings Mallorca Codebase

## Overview
This project is designed to facilitate route planning for a cycling trip in Mallorca, utilizing GPX files to manage routes and display them on a map. The application is structured to support both elite and recreational cyclists, allowing users to select and customize their routes.

## Architecture
- **Main Components**: The application consists of HTML files for structure (`index.html`, `mobile.html`) and CSS for styling embedded within the HTML files.
- **Data Flow**: Routes are loaded from GPX files located in the `routes/` directory. The application processes these files to generate route options for users.
- **Service Boundaries**: The main service boundaries are defined by the user interface (UI) components in `index.html` and `mobile.html`.

## Developer Workflows
- **Building and Running**: Open `index.html` in a web browser to run the application. Ensure that all GPX files are present in the `routes/` directory.
- **Testing**: There are no formal tests defined in this codebase. Manual testing is required to ensure functionality.
- **Debugging**: Use browser developer tools to inspect elements and debug JavaScript. Console logs are used for error reporting and debugging.

## Project-Specific Conventions
- **File Naming**: GPX files follow a specific naming convention: `GGTM26_D{dayNum}_{type}_{description}.gpx`. The `{type}` can be `A` for elite routes and `B` for recreational routes.
- **JavaScript Functions**: Key functions include `autoMakeElite()` and `autoMakeMosjonist()`, which automate route selection based on user preferences.

## Integration Points
- **External Dependencies**: The project uses Leaflet.js for map rendering, included via CDN in the HTML files. Ensure internet access for proper functionality.
- **Cross-Component Communication**: The UI components communicate with the JavaScript logic through event handlers defined in the HTML files, such as `onclick` for buttons and `onchange` for checkboxes.

## Key Files and Directories
- **HTML Files**: 
  - `index.html`: Main entry point for desktop users.
  - `mobile.html`: Entry point for mobile users, redirects from `index.html` based on user agent.

- **Routes Directory**: 
  - `routes/`: Contains all GPX files used by the application.

## Conclusion
This document serves as a guide for AI coding agents to understand the structure and functionality of the Gåpings Mallorca codebase. For further assistance, refer to the README.md for additional context on project setup and usage.

---
> Source: [kasbohm/ggtm26](https://github.com/kasbohm/ggtm26) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
