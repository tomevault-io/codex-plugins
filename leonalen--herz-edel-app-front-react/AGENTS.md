# GitHub Copilot Instructions

## Purpose

This file provides guidelines for AI coding agents (such as GitHub Copilot) to work effectively in this project. It covers conventions, architecture, and best practices for automated code changes.

## Project Overview

- **Stack:** React + Vite + Tailwind CSS + TypeScript
- **Key Directories:**
  - `src/External/components/` (shared UI components)
  - `src/External/pages/` (main pages)
  - `src/External/secciones/` (section-specific components/pages)
  - `src/store/external/NavegacionContext.tsx` (navigation context)
  - `public/img/` (image assets)
- **SEO:** Meta tags managed via `react-helmet-async` and `index.html`
- **State:** React context (no Redux)
- **Icons:** `react-icons` (no MUI)
- **Carousels:** Custom implementation (no Swiper)

## Coding Conventions

- Use functional components and hooks.
- Prefer Tailwind CSS for styling; avoid external UI libraries.
- Organize section-specific components under their respective folders in `src/External/secciones/`.
- Optimize images for web (use compressed formats, appropriate sizes).
- Use localStorage for simple client-side persistence (e.g., visit counter).
- Clean up unused imports and dependencies.

## Automated Agent Guidelines

- When reorganizing sections/components (e.g., moving Xammy to hembras, adding Bruce as joven, updating Nash's photo), update both the component file and its usage in the relevant section/page.
- For SEO improvements, update both meta tags and visible text content.
- When adding new features, prefer lightweight, native solutions over heavy dependencies.
- Always validate changes with a production build (`npm run build`).
- Remove any unused code, assets, or dependencies after migration.

## Example Tasks

- Move a component (e.g., Xammy) from one section to another and update its references.
- Add a new joven (e.g., Bruce) by creating a component and updating the section page.
- Update an image reference (e.g., Nash's photo) in its component and ensure the asset exists.
- Improve SEO by updating meta tags and visible text in pages/components.
- Add a visit counter using localStorage in the HomePage.

## Review & Validation

- After automated changes, run `npm run build` to ensure the app compiles and works as expected.
- Check for unused imports and remove them.
- Validate that all images and assets are correctly referenced and displayed.

## Contact

For questions or manual review, contact the project maintainer.

---
> Source: [leonalen/herz_edel_app_front_react](https://github.com/leonalen/herz_edel_app_front_react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
