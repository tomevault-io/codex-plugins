# Copilot Instructions for TCC Codebase

## Overview
This is a PHP-based web application for vehicle sales, built with a modular structure. The project is organized by feature and responsibility, with clear separation between business logic, UI components, and data access.

## Architecture & Key Directories
- **Root PHP files**: Entry points for main pages (e.g., `index.php`, `compras.php`, `pagina-venda.php`).
- **controladores/**: Contains backend controllers for user actions (e.g., `cadastro.php`, `login.php`, `veiculos/anunciar-veiculo.php`).
- **estruturas/**: UI components and reusable elements (e.g., `navbar/`, `modal/`, `card-compra/`).
- **funcoes_php/**: Utility PHP functions, often for session and authentication.
- **img/**: Static assets, organized by type (carros, marcas, compras, etc.).
- **menu/**: Secondary pages and feature-specific views.

## Data Flow & Integration
- Database access is handled via `conexao_bd.php` (both root and controller versions). All SQL queries and connections should use these files for consistency.
- Controllers in `controladores/` process form submissions, session management, and CRUD operations. They are invoked via AJAX or direct form actions.
- UI components in `estruturas/` are included in main pages using PHP `include` or `require` statements.

## Developer Workflows
- **Local development**: Use XAMPP (see `c:\xampp\htdocs\sites\TCC`). Place all files in the `htdocs` directory for Apache/PHP.
- **Debugging**: Use `teste.php` for ad-hoc code and debugging. Errors are surfaced in the browser; check Apache/PHP logs for details.
- **Database**: Schema is in `fahren.sql`. Import via phpMyAdmin or MySQL Workbench.

## Project Conventions
- All user authentication and session logic is handled in `controladores/login.php`, `funcoes_php/logout.php`, and related files.
- UI is built with PHP includes for modularity. Example: `include('estruturas/navbar/navbar-default.php')`.
- AJAX endpoints are typically in `controladores/` or its subfolders.
- CSS is global (`style.css`), with some inline styles in components.
- JavaScript is in `script.js` (global behaviors).

## Patterns & Examples
- To add a new vehicle listing: Use `controladores/veiculos/anunciar-veiculo.php` for backend, and `menu/editar-anuncio.php` for frontend.
- To change user info: Use `controladores/mudar-infos.php`.
- To favorite a vehicle: Use `controladores/veiculos/favoritar-veiculo.php`.

## External Dependencies
- No package manager detected; all dependencies are local PHP, JS, and CSS files.
- Images and assets are referenced via relative paths from `img/`.

## Tips for AI Agents
- Always use existing controller and utility files for backend logic.
- Follow the directory conventions for new features (controllers in `controladores/`, UI in `estruturas/`, assets in `img/`).
- Reference the database schema in `fahren.sql` for any new data features.
- Use PHP includes for UI composition.

---
If any section is unclear or missing, please provide feedback or specify which workflows or patterns need more detail.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kersilva)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kersilva)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
