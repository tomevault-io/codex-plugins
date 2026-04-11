# Portfolio Site - AI Coding Instructions

## Project Overview
This is a customizable React-based portfolio website built with Parcel bundler. The site is designed for educational purposes and GitHub Codespaces deployment. Users can personalize their portfolio by updating a single `siteProps` object in [App.jsx](../src/App.jsx).

## Architecture & Data Flow

### Single Source of Truth
- **[App.jsx](../src/App.jsx)** contains `siteProps` object with all personal information (name, email, social links, colors)
- Props cascade down to child components (`Header`, `Home`, `About`, `Portfolio`, `Education`, `Footer`)
- **Key principle**: Changes to `siteProps` automatically reflect across all sections without duplicating data

### Component Structure
```
src/
├── App.jsx              # Defines siteProps, renders main layout
├── Components/
│   ├── Home.jsx         # Hero section with background image
│   ├── About.jsx        # Skills & description (static content)
│   ├── Portfolio.jsx    # Project list array rendering
│   ├── Education.jsx    # Education background
│   ├── Header.jsx       # Navigation header
│   └── Footer.jsx       # Social links (prop-driven)
├── styles.css           # Global styles (imported into App.jsx)
└── images/              # Project assets & social icons
```

### Section Rendering Pattern
Each component follows this structure:
1. **Configuration data** defined as constants at component top (e.g., `projectList`, `skillsList`, `description`)
2. **Functional component** that accepts props
3. **PropTypes validation** for type safety
4. Static rendering with hardcoded JSX structure

## Key Customization Points

### Adding Personal Data
Update `siteProps` object in [App.jsx](../src/App.jsx):
- Basic fields: `name`, `title`, `email`
- Social handles: `gitHub`, `linkedIn`, `twitter`, `instagram`, `youTube`, `medium`
- Colors: `primaryColor`, `secondaryColor` (CSS hex values)

### Styling Sections
- Global styles in [styles.css](../src/styles.css) use `@import` for Google Fonts
- Component-specific inline styles common (see [Home.jsx](../src/Components/Home.jsx) positioning)
- Color scheme defined as hex strings in [App.jsx](../src/App.jsx), passed to Footer

### Adding Project/Content
- [Portfolio.jsx](../src/Components/Portfolio.jsx): Add objects to `projectList` array (title, description, url)
- [About.jsx](../src/Components/About.jsx): Update `skillsList` array for skills display
- [Education.jsx](../src/Components/Education.jsx): Modify education data structures

## Build & Deployment

### Development
```bash
npm start                    # Runs Parcel dev server on port 1234
npm run build               # Production build to dist/
npm run build-gh            # Build with GitHub Pages public URL
npm run deploy              # Deploy to GitHub Pages (via gh-pages)
```

### Deployment Workflow
- **GitHub Pages**: Customized URL in [package.json](../package.json) homepage field
- `predeploy` script automatically runs `build-gh` before `deploy`
- Images located in `/src/images/` must be imported in JSX (webpack/Parcel requirement)

## Code Quality & Conventions

### ESLint & Prettier
- **.eslintrc**: ESLint rules enforced (includes React & a11y plugins)
- **.prettierrc**: Code formatting settings
- Both run on save in Codespaces (pre-configured extensions)

### React Patterns
- **Functional components only** - no class components
- **PropTypes validation** used for type checking (not TypeScript)
- **Inline styles** preferred for component-specific positioning/spacing
- **Image imports**: Always import images into JSX, not direct src paths (Parcel requirement)

### Naming Conventions
- Components: PascalCase with `.jsx` extension
- Variables/constants: camelCase
- CSS classes: kebab-case with semantic names (`.min-height`, `.background`)

## Important Notes
- **No external API calls** - portfolio is entirely static/client-side
- **PropTypes library** already installed but redundant with Parcel bundling
- **Translations**: Separate README files in [translations/](../translations/) for i18n reference only (not implemented in components)
- **Image optimization**: Keep images in images/ directory; update alt text descriptions for accessibility

## Common Tasks
- **Update name/title**: Modify `siteProps` in [App.jsx](../src/App.jsx)
- **Change colors**: Update `primaryColor`/`secondaryColor` in [App.jsx](../src/App.jsx)
- **Add social link**: Add field to `siteProps`, pass to Footer, add corresponding icon import in [Footer.jsx](../src/Components/Footer.jsx)
- **Add project**: Add object to `projectList` in [Portfolio.jsx](../src/Components/Portfolio.jsx)
- **Add skill**: Add string to `skillsList` in [About.jsx](../src/Components/About.jsx)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NilayGanvit)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/NilayGanvit)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
