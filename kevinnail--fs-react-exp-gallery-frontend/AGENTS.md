# Copilot Instructions for Stress Less Glass

Welcome to the Stress Less Glass codebase! This document provides essential guidelines for AI coding agents to be productive in this project. It covers the architecture, workflows, conventions, and integration points specific to this repository.

## Project Overview

Stress Less Glass is a full-stack portfolio application showcasing glass artwork. The frontend is built with React, while the backend uses Express. Key features include:

- Dynamic image gallery with AWS S3 integration
- Secure admin authentication
- Order management and analytics

## Architecture

- **Frontend**: React application located in the `src/` directory. Major components include:
  - `MainGallery`: Displays the gallery of artwork.
  - `Admin`: Handles admin-specific features like image uploads and order management.
  - `GalleryPostCard` and `MainGalleryPostCard`: Reusable components for displaying artwork.
- **Backend**: RESTful API (not included in this repo) for data management.
- **Database**: PostgreSQL for data persistence.
- **Cloud Storage**: AWS S3 for image storage and delivery.

## Developer Workflows

### Build and Run

- Install dependencies: `npm install`
- Start development server: `npm start`
- Build for production: `npm run build`

### Testing

- Run tests: `npm test`
- Test files are located alongside components, e.g., `App.test.js`.

### Debugging

- Use React Developer Tools for inspecting component state.
- Check the browser console for API errors.

## Project-Specific Conventions

- **Component Structure**: Each component has its own folder under `src/components/` with separate files for logic (`.js`) and styles (`.css`).
- **State Management**: Local state is managed with React hooks. Global state (if needed) is managed using `stores/`.
- **Image Handling**: Images are uploaded to AWS S3. Use the `services/` directory for API calls related to image management.

## Integration Points

- **AWS S3**: Used for image uploads and retrieval. Ensure proper configuration in the backend.
- **PostgreSQL**: Database schema includes tables for users, orders, and gallery posts.
- **Authentication**: JWT-based authentication for admin routes.

## Key Files and Directories

- `src/components/`: Contains all React components.
- `src/services/`: API service files for interacting with the backend.
- `src/stores/`: State management files.
- `public/`: Static assets and the `index.html` file.
- `build/`: Production build output.

## Examples

### Adding a New Component

1. Create a new folder under `src/components/`.
2. Add a `.js` file for logic and a `.css` file for styles.
3. Export the component and import it where needed.

### Making an API Call

1. Use the `services/` directory to add a new API function.
2. Example:

   ```javascript
   import axios from 'axios';

   export const fetchGalleryPosts = async () => {
     const response = await axios.get('/api/gallery-posts');
     return response.data;
   };
   ```

### Debugging Image Uploads

- Check the `services/` directory for the `uploadImage` function.
- Verify AWS S3 credentials and bucket configuration.

---

For additional details, refer to the `README.md` or contact the project maintainer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinnail)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kevinnail)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
