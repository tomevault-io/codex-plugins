# Copilot Instructions

These instructions guide AI assistants (GitHub Copilot, ChatGPT, etc.) when generating code for this repository.

---

# Core Rules

AI must always:

* Follow existing project architecture
* Use TypeScript strictly
* Avoid `any`
* Prefer existing utilities/hooks
* Do not import Node's `fs` or `path` modules; use repository utilities instead
* Follow folder structure
* Use Tailwind CSS for styling
* Keep components small and composable
* Maintain accessibility and localization
* During build time, if any locale file is not found, highlight in build screen or put them in logs folder, skip the page and throw 404 page. Build should not stop or hang to find missed locale files. Same applied across all pages.

* When the assistant triggers a full build (for example running `npm run build`) from the chat or CI runner, the assistant must wait for the build process to complete before making further repository edits or starting another build. Do not run concurrent builds or edit files while a build is in progress — wait for the build result and act on errors/failures only after the build finishes.

---

# SEO and Indexing Preservation

AI must **NEVER** modify, remove, or disrupt Google indexing and IndexNow integration:

### Google Indexing Protection
- Never alter `public/sitemap.xml` generation or content
- Never modify `public/robots.txt`
- Never remove or change SEO meta tags (title, description, canonical)
- Never disrupt structured data (JSON-LD) output
- Never change Open Graph or Twitter meta tags

### IndexNow Protection
- Never modify `lib/constants.ts` INDEXNOW_KEY
- Never remove `public/{key}.txt` verification files
- Never alter `scripts/notify-indexnow.js`
- Never change IndexNow notification in build scripts
- Never disrupt IndexNow API integration

### Build SEO Protection
- Never modify sitemap generation scripts
- Never alter SEO audit scripts
- Never change build-time SEO file handling

**If any task conflicts with SEO preservation, halt immediately and seek clarification.**

---

# Folder Structure

```
app/
  components/
  hooks/
  context/
  feature-folders/
lib/
types/
scripts/
public/
  locales/
  images/
```

Rules:

* Components → `app/components/`
* Hooks → `app/hooks/`
* Utilities → `lib/`
* Types → `types/`
* Pages → `app/<route>/page.tsx`

---

# Naming Conventions

| Item       | Convention       |
| ---------- | ---------------- |
| Components | PascalCase       |
| Functions  | camelCase        |
| Variables  | camelCase        |
| Constants  | UPPER_SNAKE_CASE |
| Folders    | kebab-case       |
| Types      | PascalCase       |

Examples

```
AboutClient.tsx
useLocale.ts
PAGE_SIZE
userProfile
```

---

# Component Pattern

All React components must follow this structure.

```tsx
"use client";

import { useLocale } from "@app/context/locale-context";
import PageLayout from "@components/common/PageLayout";

export default function ExampleClient() {
  const { t } = useLocale();

  return (
    <PageLayout>
      {t("example.title")}
    </PageLayout>
  );
}
```

Rules

* Use functional components
* Always type props
* Use `"use client"` for client components
* Avoid class components
* Keep JSX readable

---

# Page Pattern

Each route contains

```
app/<route>/page.tsx
```

Example

```tsx
import { createGenerateMetadata } from "@lib/pageUtils";
import ExampleClient from "./exampleclient";

export const generateMetadata = createGenerateMetadata("example");

export default function Page() {
  return <ExampleClient />;
}
```

---

# State Management

Use:

* `useState` for simple state
* `useReducer` for complex local logic
* `React Context` for shared state

Avoid introducing Redux unless already used.

---

# Internationalization

All UI text must come from locale files.

Never hardcode UI text.

Example

```tsx
const { t } = useLocale();
<h3>{t("about.title")}</h3>
```

---

# Styling

Use Tailwind CSS only.

Avoid inline styles.

Correct example

```tsx
className="text-[#5b2d12] rounded-2xl shadow-lg"
```

---

# Accessibility

UI must include:

* semantic HTML
* ARIA labels
* keyboard navigation
* accessible contrast

Example

```tsx
<button aria-label={t("donate_button_label")}>
  {t("donate")}
</button>
```

---

# Testing

Use

* Jest
* @testing-library/react

Example

```tsx
import { render, screen } from "@testing-library/react";
import Component from "../Component";

test("renders text", () => {
  render(<Component />);
  expect(screen.getByText("Example")).toBeInTheDocument();
});
```

---

# Code Quality

Before committing ensure:

```
npm run lint
```

Runs:

* lint

Also ensure formatting and tests pass if the project provides scripts for them


## Build Script Policy

* Do NOT modify `.render-build.sh` or the `scripts` section of `package.json` without explicit approval from repository maintainers.
* Changes to deployment or CI scripts require clear justification and maintainer approval.

---

# Build and Locale Handling


During build time:

* If any locale file is not found, log it in the logs folder
* Skip the page and when user accesses the page, throw a 404 page for missing locales
* Apply the strict static export solution to all pages:

**Strict Static Export Solution:**

```ts
export async function generateStaticParams() {
  const data = await getAllStories(); // your source
  return data
    .filter(item => item?.id && item?.localeExists) // filter invalid
    .map(item => ({ id: item.id }));
}
```

* Ensure output: 'export' is used for static generation
* Filter out invalid items and those missing locales
* This solution must be applied to all pages and OpenSpec artifacts

---

# SEO and Static Export Requirements

* Generate `sitemap.xml` automatically and include all static and dynamic routes.
* Generate `robots.txt` and allow all indexable pages while blocking only unnecessary paths.
* Add canonical URLs for every page using absolute URLs in metadata.
* Use `generateMetadata()` on every page with title, description, keywords, Open Graph, and Twitter metadata.
* Add structured data (JSON-LD) where appropriate using `Article`, `WebPage`, or `Breadcrumb` schemas.
* Implement `generateStaticParams()` for all dynamic routes so every page exists at build time.
* Use clean static URLs and avoid query-based navigation for core content.
* Ensure critical content is server-rendered and indexable without client-only hydration.
* Avoid server-only features or runtime-only pages that break static export.

---

# Repository Build Script Policy

* Do NOT modify `.render-build.sh` or the `scripts` section of `package.json` without explicit approval from repository maintainers.
* These scripts are critical for deployment and CI/CD stability.
* Any change to build/deploy scripts must include a clear justification and maintainer approval.

---

# Repository-specific Conventions

These repository conventions must be applied when generating code or OpenSpec artifacts for this project:

* **Tech stack:** Next.js, TypeScript (strict), React, Tailwind CSS.
* **Folder layout:** `app/` (pages and client components), `app/components/`, `app/hooks/`, `lib/`, `types/`, `public/locales/` for translations.
* **Localization:** All UI text must come from locale files and use the `useLocale` context hook; never hardcode UI strings.
* **Client components:** Use `"use client"` at top of client components; always type props.
* **Utilities:** Prefer existing utilities in `lib/` and hooks in `app/hooks/` over adding new helpers.
* **No direct Node fs/path:** Do not import Node's `fs` or `path`; use repository utilities such as `lib/storage.ts` if available.
* **Testing & CI:** Run `npm run lint` (linter) before committing changes and ensure formatting/tests pass if the project provides scripts for them.
* **Task sizing:** Break implementation tasks into small, testable chunks (max ~2 hours per task).

Follow these conventions in addition to the generic rules above.

---

# Page JSON Rendering

When generating or updating page components, render the page's JSON locale/context object into the UI for discovery and content completeness. However, exclude the following keys from direct rendering: `meta`, `openGraph`, and `schema` — these are for metadata only and should be used to populate page metadata or structured-data blocks, not displayed as page content.

Guidelines:

* Use the locale namespace or the page's `index.json` as the source of truth for content.
* Render any string, paragraphs, arrays, or structured content from the JSON except the excluded keys above.
* Use existing helpers (`useLocaleSection`, `getLocaleNamespaceObject`, `loadLocaleData`) to load data.
* Sanitize or format long text (e.g., split on `\n\n` into paragraphs) before rendering.
* Use `meta`, `openGraph`, and `schema` only to populate `generateMetadata`, Open Graph tags, and JSON-LD respectively.

---
> Source: [vulchivijay/sanatana](https://github.com/vulchivijay/sanatana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
