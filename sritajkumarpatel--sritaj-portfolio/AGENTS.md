# GitHub Copilot / AI Agent Instructions

Purpose: help AI agents and contributors be productive immediately when working in this repository.

Quick Summary

- Single-page React portfolio built with Vite and TailwindCSS.
- **Fully plug-and-play**: All content lives in JSON files (`src/config.json` and `src/data/*.json`). Zero hardcoded content.
- Main application: `src/App.jsx` imports JSON data and passes to data-driven components. `src/main.jsx` bootstraps the app.
- Styling: `src/index.css` contains Tailwind directives and project-specific fallback/custom CSS.
- Build/deploy: Vite (`npm run dev` / `npm run build`) and GitHub Pages via `gh-pages` (`npm run deploy`).

Quick Start (dev + build)

- Install dependencies: `npm install`
- Run dev server: `npm run dev` (uses Vite)
- Build for production: `npm run build`
- Preview production build: `npm run preview`
- Deploy to GitHub Pages: `npm run deploy` (uses `gh-pages -d dist`)

Important Files & Why

**Configuration:**

- `src/config.json` — Central hub for personal info, titles, bio, social links, theme. Edit this first to customize for a new portfolio owner.

**Data (JSON-based):**

- `src/data/aboutMe.json` — Bio sections, competencies, and philosophy. Used by `About.jsx`.
- `src/data/awards.json` — All awards with title, year, quarter, company, description. Used by `Awards.jsx`.
- `src/data/experience.json` — Work history with roles, highlights, key achievements. Used by `Experience.jsx`.
- `src/data/certifications.json` — Certifications with issuers, dates, credentials. Used by `Certifications.jsx`.
- `src/data/techStacks.json` — Technical skills and proficiencies. Used by `TechStack.jsx`.
- `src/data/projects.json` — Portfolio projects with support for featured/regular projects, modular sections (keyFeatures, technologies, quickStart), and data-driven content. Each project controls which sections display via `sections` array. Used by `Projects.jsx` → `ProjectCard.jsx` → `CardSection.jsx`.
- `src/data/mediumArticles.json` — Medium articles feed (auto-fetched by `update-medium.js` or manual). Used by `MediumArticles.jsx`.
- `src/data/education.json` — Education with degrees, institutions, periods, focus areas. Used by `Education.jsx`.

**Components:**

- `src/App.jsx` — Main orchestrator: imports all JSON files and passes them as props to components. Navigation state managed here.
- `src/components/Nav.jsx` — Navigation bar using `config.personal.name`.
- `src/components/Hero.jsx` — Hero section using `config.bio.headline`, `config.bio.subtitle`, personal links.
- `src/components/About.jsx` — About section rendering from `aboutMe.json`.
- `src/components/Awards.jsx` — Awards section (compact centered cards) rendering from `awards.json`.
- `src/components/Experience.jsx` — Experience section rendering from `experience.json`.
- `src/components/Certifications.jsx` — Certifications rendering from `certifications.json`.
- `src/components/TechStack.jsx` — Tech skills rendering from `techStacks.json`.
- `src/components/Projects.jsx` — Projects section orchestrator. Renders featured/regular projects from `projects.json` using `ProjectCard` component.
- `src/components/ProjectCard.jsx` — Reusable project card component. Renders individual projects with dynamic sections based on `sections` array from JSON.
- `src/components/CardSection.jsx` — Modular card section renderer for `keyFeatures`, `technologies`, and `quickStart`. All content data-driven from JSON.
- `src/components/MediumArticles.jsx` — Medium articles rendering from `mediumArticles.json`.
- `src/components/Education.jsx` — Education section rendering from `education.json`.
- `src/components/Footer.jsx` — Footer using `config.personal.name` and links.
- `src/main.jsx` — React bootstrap.

**Other:**

- `src/index.css` — Tailwind directives plus custom CSS (`.glass-card`, gradients, responsive tweaks).
- `vite.config.js` — Vite config; `base` is set to `/sritaj-portfolio/`. Update if repo name changes.
- `package.json` — Scripts and dependencies. Includes `gh-pages` for deployment.
- `tailwind.config.cjs` / `postcss.config.cjs` — Tailwind + PostCSS integration.
- `scripts/update-medium.js` — Auto-fetch Medium articles (optional).
- `CHECKLIST.md` — Visual and styling adjustments; useful for pixel-perfect tweaks.

Architecture & Patterns

- **Single-page app**: No routing library. Navigation is controlled by `activeSection` state in `App.jsx`.
- **Content-as-JSON**: ALL content lives in JSON files (`src/config.json` and `src/data/*.json`). ZERO hardcoded content. Components import JSON and render it.
  - `src/config.json` contains: personal info, titles, bio text, social links, theme settings.
  - `src/data/*.json` contain all section-specific content (about, awards, experience, etc.).
- **Data-driven components**: Each component receives JSON data as a prop from `App.jsx` and renders it. Examples:
  - `<About aboutMe={aboutMe} />` — passes `aboutMe.json` to About component.
  - `<Awards awards={awards} />` — passes `awards.json` to Awards component.
  - `<Experience experience={experience} />` — passes `experience.json` to Experience component.
- **UI / styling**: Tailwind utility classes used heavily. Custom CSS in `src/index.css` (`.glass-card`, `.experience-card`, gradients). Keep consistent when adding UI.
- **Icon usage**: `lucide-react` icons imported in components as needed. Use existing icons for brand consistency.
- **Deployment**: Vite's `base` option set to `/sritaj-portfolio/`. If repo is forked/renamed, update `vite.config.js` base value.

Project-specific Conventions

- **Zero hardcoded content**: DO NOT add content directly in components. Always create or update a JSON file in `src/data/` or `src/config.json`.
- **Reuse `glass-card` CSS** for new cards/panels. Implements the visual "glass" look across elements.
- **Component naming**: PascalCase for components in `src/components/`. Keep single-purpose.
- **External links**: Use `target="_blank" rel="noopener noreferrer"` for all external links.
- **⚠️ DOCUMENTATION**: Do NOT create unnecessary `.md` files unless explicitly requested by the user. Keep documentation minimal and focused on the Quick Start, architecture, and debugging tips.

## 🤖 Using Copilot to Speed Up Customization

Users can leverage Copilot/AI agents to customize this portfolio **10x faster**. Here's how:

### For Users (Forks)

**Tell Copilot what you want**, and it will update all JSON files automatically:

#### Example Prompt to Copilot:

```
"My name is Jane Developer. I'm a cloud architect with 8 years of experience.
My headline is 'Cloud Native Architect | DevOps Enthusiast'.
My bio is: 'Specialized in Kubernetes, AWS, and infrastructure automation.
Love mentoring junior engineers.'
My GitHub is janedev, LinkedIn is jane-developer, Medium is janedev.
Titles: Cloud Architect, DevOps Engineer, SRE, Tech Lead
I work at TechCorp as a Technical Architect.
Update all my portfolio data."
```

**What Copilot will do:**

1. Update `src/config.json` with personal info, headline, titles
2. Update `src/data/aboutMe.json` with bio and competencies
3. Update `src/data/experience.json` with work history
4. Update `src/data/techStacks.json` with relevant skills
5. Make sure `vite.config.js` base path is correct

**User then runs:**

```bash
npm install
npm run deploy
```

**Portfolio is live in 5 minutes.**

### For AI Agents (Data Exchange)

When working on this portfolio, AI agents should follow this data flow:

#### How Data Flows:

```
User Input/Request
    ↓
Copilot/Agent reads config.json structure
    ↓
Agent updates JSON files in src/data/ and src/config.json
    ↓
Agent validates JSON syntax
    ↓
Agent runs: npm run build (verify no errors)
    ↓
Agent confirms: "Portfolio updated and ready to deploy"
```

#### Structured Data Passing to Agents:

When asking Copilot to update portfolio data, provide info in this format:

**Personal Info:**

```json
{
  "name": "Full Name",
  "email": "email@example.com",
  "github": "github-username",
  "linkedin": "linkedin-handle",
  "medium": "medium-handle"
}
```

**Bio & Titles:**

```json
{
  "headline": "Professional headline",
  "subtitle": "Detailed bio/about text",
  "titles": ["Title 1", "Title 2", "Title 3"]
}
```

**Work Experience:**

```json
{
  "company": "Company Name",
  "roles": ["Role 1", "Role 2"],
  "highlights": ["Achievement 1", "Achievement 2"],
  "awards": ["Award 1"]
}
```

**Skills:**

```json
{
  "category": "Category Name",
  "skills": ["Skill 1", "Skill 2"]
}
```

**Projects:**

```json
{
  "title": "Project Name",
  "description": "Short description",
  "featured": true,
  "sections": ["keyFeatures", "technologies", "quickStart"],
  "keyFeatures": ["Feature 1", "Feature 2"],
  "technologies": ["Tech 1", "Tech 2"],
  "quickStart": {
    "title": "Quick Start:",
    "steps": [
      { "text": "Step 1 text", "code": "optional-code-snippet" },
      { "text": "Step 2 text" }
    ]
  },
  "github": "https://github.com/user/repo",
  "link": "https://project-url.com"
}
```

#### Agent Instructions:

When updating portfolio data:

1. **Always update JSON files in `src/data/`** — never hardcode in components
2. **Always validate JSON syntax** — use JSON validators
3. **Always update `src/config.json` first** — it's the primary source
4. **Always run `npm run build`** — verify no errors before confirming
5. **Never modify components** — only modify JSON data files
6. **Never create new .md files** — only modify existing ones if needed
7. **Always pass complete data objects** — don't assume agent understands partial data

### Example: User Asks Agent for Bulk Update

**User:** "Update my portfolio with my new job at Google as Senior SRE, my Medium handle is 'sre-jane', and my top 5 skills are: Kubernetes, AWS, Go, Terraform, CI/CD"

**Agent should:**

1. Read current `src/config.json` and `src/data/experience.json`
2. Update `src/config.json` with medium: "sre-jane"
3. Update `src/data/experience.json` to add Google job with SRE role
4. Update `src/data/techStacks.json` to include the 5 skills
5. Run `npm run build` to validate
6. Report: "✅ Portfolio updated. Your experience now shows Google SRE role, Medium handle updated, and top skills added."

This way, **any user can fork and have their entire portfolio customized in under 5 minutes** just by describing what they want to Copilot.

How to Add a New Nav Section (example)

1. Add the section key to the nav array near the top of `App.jsx`:
   `['about','experience','certifications','tech','github','articles','projects','blog']`
2. Create a JSON file in `src/data/` for your content (e.g., `src/data/blogPosts.json`).
3. Import the JSON at the top of `App.jsx`: `import blogPosts from './data/blogPosts.json'`
4. Pass data to a component: `<BlogSection blogPosts={blogPosts} />`
5. Create the component in `src/components/BlogSection.jsx` that accepts and renders the prop.
6. Add conditional render block in `App.jsx` similar to `activeSection === 'articles'`.
7. Add a nav label mapping if different from the key (like `experience` -> `IT Experience`).

**Why this approach?**

- All content is in one place (JSON files)
- Components stay clean and focused on rendering
- Easy to customize for different portfolio owners
- No hardcoded content anywhere

Small code example: Add a blog post to `blogPosts.json`

```json
{
  "title": "My New Blog Post",
  "date": "2024-01-15",
  "excerpt": "Short description",
  "link": "https://example.com/post",
  "tags": ["React", "Web Dev"]
}
```

Debugging Tips & Gotchas

- If Tailwind classes don't appear: ensure `tailwind.config.cjs` `content` includes files (it does: `./index.html`, `./src/**/*.{js,jsx,ts,tsx}`) and `npm run dev` was restarted after installing Tailwind.
- If deploying to GitHub Pages produce broken paths: check `vite.config.js` `base` value and confirm your repo’s name matches.
- If `npm run dev` fails: check Node version (use Node 18+), run `npm ci`/`npm install`, then re-run; Vite errors usually indicate mismatched dependency versions.
- No test or CI config present—expect contributors to add test runners and GitHub Actions if needed.

Extending the App

- Prefer JSON files in `src/data/` over component arrays. Keep the data-to-component flow consistent.
- Create small, single-purpose components in `src/components/` that accept data as props.
- Never hardcode content in components. If you need content, create a JSON file for it.
- If adding routing (React Router) or SSR, be explicit about SPA changes and update `vite.config.js` base and gh-pages deployment instructions.
- When adding third-party services/calls: store API keys in environment variables and describe usage in `CHECKLIST.md`.

Integration Points & External Links

- External content: `LinkedIn`, `Medium`, `GitHub` links are stored in `config.json`.
- `gh-pages` is used for deployment; the deploy script pushes the `dist` directory to GitHub Pages for hosting.

Contributions & Best Practices (project specific)

- Keep the `glass-card` styling consistent; reuse the utility class and existing CSS where possible.
- Keep `App.jsx` structure predictable: JSON imports → component render mapping. If a change is large, split into a new component under `src/components`.
- Run `npm run build` and `npm run preview` before creating a PR for visual changes.

If anything is missing or needs to be updated (for example, test commands or CI), please point me to files you'd like me to read and I'll iterate on these instructions.

---
> Source: [sritajkumarpatel/sritaj-portfolio](https://github.com/sritajkumarpatel/sritaj-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
