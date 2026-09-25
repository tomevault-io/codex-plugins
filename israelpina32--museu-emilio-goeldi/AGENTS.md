# Copilot Instructions for AI Agents

## Project Overview
This is a Next.js-based web application designed as an interactive children's game for the Museu Emilio Goeldi. The app features realistic 3D GLTF models of prehistoric animals, with a focus on accessibility, visual appeal, and educational value for young audiences.

## Architecture & Key Components
- **Framework**: Next.js (React-based, file-based routing in `src/pages/`)
- **3D Models**: GLTF format, referenced in the UI (see `public/` for assets)
- **Styling**: Global styles in `src/styles/globals.css`
- **Entry Points**: Main app setup in `src/pages/_app.tsx` and document structure in `src/pages/_document.tsx`
- **API**: Example endpoint in `src/pages/api/hello.ts` (expandable for backend logic)

## Developer Workflows
- **Install dependencies**: `npm install`
- **Start dev server**: `npm run dev` (default port: 5173)
- **Access app**: [http://localhost:5173](http://localhost:5173)
- **No custom test/build scripts**: Use standard Next.js/Node.js workflows unless otherwise documented.

## Project-Specific Patterns
- **3D Model Integration**: GLTF models are loaded and rendered in the UI. Follow existing patterns for adding new models (see usage in main page/component).
- **Educational Content**: All UI and content should be child-friendly, visually engaging, and promote learning.
- **Responsive Design**: Ensure compatibility with desktops and tablets.

## Conventions & Best Practices
- **File Organization**: Place new pages in `src/pages/`, assets in `public/`, and global styles in `src/styles/`.
- **Component Structure**: Use functional React components. Co-locate related files when possible.
- **Localization/Accessibility**: Prioritize clear language and accessible UI elements for children.

## Integration Points
- **3D Assets**: Add new GLTF models to `public/` and reference them in components.
- **API Extensions**: Add new endpoints under `src/pages/api/` as needed.

## References
- See `README.md` for project background, setup, and feature overview.
- Key files: `src/pages/index.tsx`, `src/pages/_app.tsx`, `src/pages/_document.tsx`, `public/` (assets), `src/styles/globals.css`

---
For any unclear conventions or missing documentation, consult the project maintainer or open an issue for clarification.

---
> Source: [IsraelPina32/museu-emilio-goeldi](https://github.com/IsraelPina32/museu-emilio-goeldi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-25 -->
