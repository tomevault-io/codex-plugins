# CLAUDE.md — Banora Chiropractic Website

## Project Overview

This is the website for **Banora Chiropractic**, a chiropractic clinic located at 2/44 Greenway Drive, Tweed Heads South NSW 2486. The site is built with Astro, Tailwind CSS, and deployed on Vercel via GitHub.

The website's primary goal is to **convert local searchers into booked patients** while educating the community about chiropractic care.

A second clinic — Palm Beach Chiropractic & Remedial (1/28 Palm Beach Ave, Palm Beach QLD 4221) — will be added later. Build with this in mind: keep clinic-specific data in config files so the architecture supports multi-clinic expansion without refactoring.

---

## Personas

When working on this project, adopt the relevant persona(s) for the task:

### SEO Expert
You are a local SEO specialist focused on healthcare practices in the Tweed Heads / Gold Coast region. Every page must be optimised for search. Think in terms of search intent, keyword placement, internal linking, schema markup, and Core Web Vitals. Local SEO is the #1 growth driver for this clinic.

### Humanised Copywriter
You write warm, approachable, patient-focused copy. Avoid clinical jargon unless explaining a condition. Use "you" language. Write as if speaking to someone who's in pain and looking for help — be reassuring, credible, and clear. Every page should feel like a conversation, not a textbook.

### Chiropractic Content Specialist
You understand chiropractic care deeply — phases of care (relief, corrective, wellness), common conditions (sciatica, back pain, neck pain, headaches, postural issues), and evidence-based approaches. All content must be AHPRA-compliant: no guaranteed outcomes, no misleading claims, no testimonials that imply guaranteed results.

### Mobile-First Developer
You build for phones first, tablets second, desktop third. Navigation must be thumb-friendly. Buttons must be large enough to tap. Text must be readable without zooming. Page speed is non-negotiable — target 90+ Lighthouse scores across all metrics.

---

## Brand Guidelines

### Colours

```
--color-navy:       #1B3A5C   (primary — headers, footer, nav)
--color-mid-blue:   #2C5F8A   (secondary — buttons, links, accents)
--color-light-blue: #5B9EC9   (tertiary — hover states, highlights)
--color-light-grey: #F5F5F5   (backgrounds, alternating sections)
--color-white:      #FFFFFF   (primary background)
--color-dark-text:  #1A1A1A   (body text)
```

### Typography
- **Headings**: Use a clean, professional sans-serif (e.g., Montserrat, Raleway, or similar)
- **Body**: Use a highly readable sans-serif (e.g., Open Sans, Source Sans Pro, or similar)
- **Minimum body size**: 16px on mobile, 18px on desktop
- **Line height**: 1.6 for body text

### Tone of Voice
- Warm and professional, never clinical or cold
- Use "we" for the clinic, "you" for the patient
- Active voice, short sentences, simple words
- Reassuring without making promises or guarantees (AHPRA compliance)

### Visual Style
- Clean, spacious layouts with generous whitespace
- Rounded corners on cards and buttons (8px)
- Subtle shadows for depth, never harsh drop shadows
- Photography: warm, natural, showing real people (not stock photos where possible)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Astro |
| Styling | Tailwind CSS |
| Deployment | Vercel |
| Version Control | GitHub |
| Forms | Formspree (endpoint: mpqjeego) |
| Booking | IconPractice widget |
| Analytics | Google Analytics 4 |
| Maps | Google Maps Embed API |
| Chatbot | Custom Claude-powered widget |

### Why Astro
Astro ships zero JavaScript by default. Pages are pre-rendered as static HTML at build time, which means lightning-fast load times and perfect SEO out of the box. JavaScript is only added where interactivity is genuinely needed (mobile menu, chatbot, forms). This is ideal for a content-driven clinic website.

### Booking URL
```
https://www.iconpractice.com/ob/7138/banorachiropractic/245386/2
```

### Chatbot Deployment
Existing Netlify deployment: `thriving-conkies-08c1ca.netlify.app`
Widget script: `banora-chatbot-widget.js`

---

## Site Architecture

```
/ (Home)
├── /about
│   └── Meet Dr James Shipway (BSc DC) & Dr Paul Cater (B.Sc.Chiro. & M.Chiro)
├── /services
│   ├── /services/chiropractic-adjustments
│   ├── /services/posture-correction
│   ├── /services/sports-chiropractic
│   ├── /services/pregnancy-chiropractic
│   ├── /services/paediatric-chiropractic
│   └── (additional service pages as needed)
├── /conditions
│   ├── /conditions/back-pain
│   ├── /conditions/neck-pain
│   ├── /conditions/sciatica
│   ├── /conditions/headaches-migraines
│   ├── /conditions/shoulder-pain
│   └── (additional condition pages as needed)
├── /areas
│   ├── /areas/tweed-heads-south
│   ├── /areas/tweed-heads
│   ├── /areas/banora-point
│   ├── /areas/coolangatta
│   ├── /areas/palm-beach
│   ├── /areas/currumbin
│   ├── /areas/elanora
│   ├── /areas/bilinga
│   ├── /areas/tugun
│   ├── /areas/kirra
│   ├── /areas/burleigh-heads
│   ├── /areas/terranora
│   ├── /areas/chinderah
│   └── /areas/kingscliff
├── /patient-resources
│   └── Links to patient hub, exercise guides, posture information
└── /contact
    └── Contact form, map, phone, booking widget
```

---

## Project Structure

```
banora-website/
├── CLAUDE.md                    (this file)
├── .skills/                     (Claude Code skills)
│   ├── seo/SKILL.md
│   ├── copywriting/SKILL.md
│   ├── components/SKILL.md
│   ├── performance/SKILL.md
│   ├── brand/SKILL.md
│   └── human-voice/SKILL.md
├── src/
│   ├── layouts/
│   │   └── BaseLayout.astro     (root layout — head, nav, footer, analytics)
│   ├── pages/
│   │   ├── index.astro          (homepage)
│   │   ├── about.astro
│   │   ├── services/
│   │   │   └── [slug].astro     (dynamic service pages)
│   │   ├── conditions/
│   │   │   └── [slug].astro     (dynamic condition pages)
│   │   ├── areas/
│   │   │   └── [slug].astro     (dynamic location pages)
│   │   ├── patient-resources.astro
│   │   └── contact.astro
│   ├── components/
│   │   ├── layout/              (Header.astro, Footer.astro, Navigation.astro, MobileMenu.astro)
│   │   ├── ui/                  (Button.astro, Card.astro, Section.astro, Badge.astro)
│   │   ├── forms/               (ContactForm.astro, BookingCTA.astro)
│   │   ├── sections/            (Hero.astro, ServiceGrid.astro, CTABanner.astro, FAQAccordion.astro)
│   │   └── integrations/        (ChatbotWidget.astro, BookingWidget.astro, GoogleMap.astro)
│   ├── data/
│   │   ├── clinic.ts            (clinic info — name, address, phone, hours, practitioners)
│   │   ├── services.ts          (service definitions for dynamic routes)
│   │   ├── conditions.ts        (condition definitions for dynamic routes)
│   │   └── areas.ts             (suburb data for location pages)
│   ├── utils/
│   │   └── helpers.ts           (shared utilities)
│   └── styles/
│       └── global.css           (Tailwind base + custom CSS variables)
├── public/
│   ├── images/
│   └── fonts/
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
└── package.json
```

---

## Astro Fundamentals

### How Astro Components Work

Astro components (`.astro` files) have two parts separated by `---` fences:

```astro
---
// Component Script (runs at build time — server-side only)
// Import components, fetch data, define props
import Button from '../components/ui/Button.astro';

interface Props {
  title: string;
  subtitle?: string;
}

const { title, subtitle } = Astro.props;
---

<!-- Component Template (HTML output) -->
<section>
  <h1>{title}</h1>
  {subtitle && <p>{subtitle}</p>}
  <Button text="Book Now" href="/contact" />
</section>
```

The script section runs at build time and is never sent to the browser. The template section becomes static HTML. This is why Astro is so fast — no framework JavaScript ships to the client by default.

### Props
Pass data to components via props, just like HTML attributes:
```astro
<Hero title="Welcome to Banora Chiropractic" ctaText="Book a Visit" />
```

### Slots
Use `<slot />` for flexible content injection:
```astro
<!-- Section.astro -->
<section class="py-16">
  <slot />
</section>

<!-- Usage -->
<Section>
  <h2>Our Services</h2>
  <p>Content goes here</p>
</Section>
```

### Client-Side JavaScript (Islands)
Most of the site needs zero JavaScript. When you DO need interactivity, use a `<script>` tag in the component:

```astro
<button id="menu-toggle">Menu</button>
<nav id="mobile-menu" class="hidden">...</nav>

<script>
  document.getElementById('menu-toggle')?.addEventListener('click', () => {
    document.getElementById('mobile-menu')?.classList.toggle('hidden');
  });
</script>
```

Prefer vanilla JS `<script>` tags for simple interactions. Only use framework islands (React/Preact) if the interactivity is genuinely complex.

### Layouts
The base layout wraps every page and handles `<head>`, schema, analytics:

```astro
---
// src/layouts/BaseLayout.astro
import Header from '../components/layout/Header.astro';
import Footer from '../components/layout/Footer.astro';

interface Props {
  title: string;
  description: string;
  ogImage?: string;
  canonicalUrl?: string;
  schema?: object;
}

const { title, description, ogImage, canonicalUrl, schema } = Astro.props;
---

<!DOCTYPE html>
<html lang="en-AU">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{title}</title>
  <meta name="description" content={description} />
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  {ogImage && <meta property="og:image" content={ogImage} />}
  {canonicalUrl && <link rel="canonical" href={canonicalUrl} />}
  {schema && <script type="application/ld+json" set:html={JSON.stringify(schema)} />}
</head>
<body>
  <Header />
  <main>
    <slot />
  </main>
  <Footer />
</body>
</html>
```

---

## Data-Driven Pages

Services, conditions, and location pages are generated from data files using Astro's `getStaticPaths()`. This means:
- **One template** per page type (service, condition, area)
- **One data file** per page type with all content
- Adding a new service/condition/suburb = adding an entry to the data file, not building a new page

### Dynamic Route Pattern

```astro
---
// src/pages/services/[slug].astro
import BaseLayout from '../../layouts/BaseLayout.astro';
import { services } from '../../data/services';

export function getStaticPaths() {
  return services.map((service) => ({
    params: { slug: service.slug },
    props: { service },
  }));
}

const { service } = Astro.props;
---

<BaseLayout title={service.metaTitle} description={service.metaDescription}>
  <h1>{service.title}</h1>
  <!-- page content using service data -->
</BaseLayout>
```

Each data entry includes: slug, title, meta title, meta description, hero content, body content, schema markup data, and internal links.

---

## SEO Requirements (Every Page)

1. Unique `<title>` tag following pattern: `{Page Title} | Banora Chiropractic | Tweed Heads South`
2. Unique `<meta name="description">` (150-160 characters, includes location + service keywords)
3. Open Graph tags (og:title, og:description, og:image, og:url)
4. Canonical URL
5. Semantic HTML: one `<h1>` per page, logical heading hierarchy
6. `alt` text on every image
7. LocalBusiness schema (JSON-LD) on every page
8. Breadcrumb schema on inner pages
9. Internal links to related services/conditions/areas
10. NAP consistency: always use "2/44 Greenway Drive, Tweed Heads South NSW 2486" and "(07) 5599 2322"

---

## AHPRA Compliance Rules

All content must comply with AHPRA advertising guidelines for healthcare:

- **Never** guarantee outcomes or use words like "cure", "fix", "guarantee", "best"
- **Never** use before/after comparisons that imply guaranteed results
- **Never** include testimonials that claim specific health outcomes
- **Always** use phrases like "may help", "can assist", "aims to", "designed to support"
- **Always** recommend patients consult with the practitioner for individual advice
- Social proof must focus on experience and service quality, not clinical outcomes

---

## Component Guidelines

- All components are Astro components (`.astro` files) unless interactivity requires JavaScript
- Use Tailwind utility classes for styling — no inline styles
- Components accept content via props and slots — no hardcoded text in component files
- Every interactive element (buttons, links, forms) must be accessible (ARIA labels, keyboard navigation, focus states)
- Use Astro's `<Image>` component from `astro:assets` for automatic image optimisation, or native `<img>` with `loading="lazy"`, `width`, and `height` attributes
- External links use `rel="noopener noreferrer"` and `target="_blank"`
- Phone numbers use `<a href="tel:+61755992322">` for click-to-call
- Booking CTAs link to IconPractice URL or trigger booking widget
- Keep JavaScript to a minimum — use `<script>` tags for simple interactivity, not framework components

---

## Integrations

### IconPractice Booking Widget
Embed as an iframe or link to:
`https://www.iconpractice.com/ob/7138/banorachiropractic/245386/2`
Place a prominent "Book Now" button in the header and throughout key pages.

### Formspree Contact Form
POST to `https://formspree.io/f/mpqjeego`
Fields: Name, Email, Phone, Message
Include honeypot spam protection.
Use a standard HTML `<form>` with `method="POST"` and `action="https://formspree.io/f/mpqjeego"` — no JavaScript needed.

### AI Chatbot Widget
Load the chatbot widget script from the Netlify deployment. Position: bottom-right corner, floating button that expands to chat window. Load with `defer` attribute so it doesn't block page rendering:

```html
<script defer src="https://thriving-conkies-08c1ca.netlify.app/banora-chatbot-widget.js"></script>
```

### Google Analytics 4
Load with `async` in the `<head>` via BaseLayout:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_TRACKING_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_TRACKING_ID');
</script>
```

Track: page views, booking button clicks, form submissions, phone number clicks.

### Google Maps
Embed on contact page showing clinic location at 2/44 Greenway Drive, Tweed Heads South NSW 2486.

---

## Performance Targets

- Lighthouse Performance: 90+
- Lighthouse Accessibility: 95+
- Lighthouse Best Practices: 95+
- Lighthouse SEO: 100
- First Contentful Paint: < 1.5s
- Largest Contentful Paint: < 2.5s
- Cumulative Layout Shift: < 0.1
- Total page weight: < 500KB per page (excluding images)

Astro's zero-JS-by-default approach gives us a massive head start on these targets. Don't ruin it by adding unnecessary client-side JavaScript.

---

## Git Workflow

- `main` branch = production (auto-deploys to Vercel)
- `dev` branch = work in progress
- Feature branches for individual pages or components: `feature/homepage`, `feature/service-pages`, etc.
- Commit messages: descriptive and prefixed (e.g., `feat: add homepage hero section`, `fix: mobile nav z-index`, `seo: add schema to service pages`)

---

## Multi-Clinic Expansion (Future)

All clinic-specific data lives in `src/data/clinic.ts`. When adding Palm Beach Chiropractic:

- Add a second clinic object to the config
- Create a clinic selector or subdomain/path strategy
- Reuse all components and templates
- Only content and branding differences change

Do not hardcode clinic details anywhere outside the data files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hellobanora)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/hellobanora)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
