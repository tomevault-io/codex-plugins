# Launch to Lead — Copilot Workspace Instructions

## Who We Are

Launch to Lead Engineering (L²) is a premium 1-on-1 engineering career coaching business run by Mansour Manci. We help engineers land top-tier roles through resume rebuilds, interview prep, and mock interviews. Our primary product is the 21-Day Engineering Career Accelerator ($997, 3×90-minute sessions). We are NOT an agency, NOT a group program, NOT a recruiter — we are an engineer coaching engineers.

## Brand Context Reference

**Always read `BRAND_CONTEXT.md` in the project root before doing any significant work.** It contains the complete brand bible: colors, design system, programs, pricing, avatars, funnel architecture, ad strategy, named methods, copywriting patterns, testimonials, and file structure. Reference it for any details not covered in these instructions.

## Voice & Tone

- **Brand voice:** Direct, confident, engineering-appropriate. No hype, no salesy language, no exclamation marks in professional content. Speak like a senior engineer giving career advice to a junior engineer they care about.
- **Mansour's voice (1st person content):** Conversational, honest about struggles, anti-corporate, values efficiency over fluff. "I'm not HR. I'm not a recruiter. I'm an engineer."
- **Audit/analysis voice:** Helpful friend who happens to be an expert. Factual, direct, never inflammatory or condescending. Frame problems as opportunities.
- **Banned words in audits:** disaster, filler, dead, invisible, kills/killer, pure filler, BS, thin, red flag, zero value.
- **CTA language:** "See If You're a Fit" (primary), "FREE Resume Guide" (secondary), "Career Launch Call" (booking). Never "Book a Call" or "Schedule a Demo."

## Design System — "Emerald Flow"

| Token | Hex | Usage |
|-------|-----|-------|
| `brand-primary` | `#064e3b` (Emerald 900) | Primary buttons, headers, footer, logo |
| `brand-secondary` | `#14b8a6` (Teal 500) | Hover states, accents, check icons |
| `brand-accent` | `#a3e635` (Lime 400) | Highlights, badges, CTA alt |
| `brand-light` | `#ecfdf5` (Emerald 50) | Section backgrounds |
| `brand-surface` | `#ffffff` | Default surface |

- **Font:** Plus Jakarta Sans (Google Fonts, weights 300-800)
- **Scoring font:** Space Grotesk (for numbers in audits)
- **Icons:** Lucide via CDN (`<i data-lucide="icon-name">`)
- **CSS:** Tailwind CSS via CDN + `tailwind-config.js` — never write custom CSS unless absolutely necessary
- **Shadows:** `soft`, `card`, `fresh` (all emerald-tinted, defined in tailwind-config.js)

## Tech Constraints

- **No build tools.** Pure HTML + Tailwind CDN + vanilla JS. No npm, no bundlers, no frameworks.
- **Hosting:** GitHub Pages, branch `main`, domain `launchtolead.io`.
- **Analytics on every page:** GA4 (`G-L8V47TR52C`), Meta Pixel (`3141490496240983`), LinkedIn Insight Tag (`8832100`). Always include the internal traffic exclusion cookie check.
- **Shared navigation:** `<div id="navigation-container"></div><script src="navigation.js"></script>` — never hardcode nav.
- **Scarcity badge on every page:** "1-on-1 Live Coaching — Only 10 Spots / Month" with pulsing red dot.

## Key Business Rules

- **10 spots/month** capacity constraint — always enforce in copy and UI.
- **$997** (or 3×$349) for Engineering Career Accelerator.
- **21-day, 3-session** program structure. Don't invent additional sessions.
- **Guarantee:** "I will keep working with you until you do." Not a money-back guarantee.
- **ROI framing:** ~$7,000/month lost while job searching (BLS data).
- **Impact Bullet Builder (IBB):** The 4-part formula: Accomplishment + Metric + How + Why. Three tiers: ★/★★/★★★.
- **Results disclaimer** on any page with testimonials: "Individual experiences presented here may not be typical. Your results may vary."

## File Organization

- Client files: `clients/{path}/{client_name}/`
- Protocols: `protocols/`
- Lead magnets: `lead-magnets/active/`
- Client materials: `client_materials/launch/` and `client_materials/accelerate/`
- Ads: `ads/` (HTML creative files)
- Proposals: `proposals/`
- Always use lowercase-with-hyphens for new filenames. Underscores for client names only.

## Working Patterns

- When creating HTML pages, follow the template in `BRAND_CONTEXT.md` Section 21.
- When editing existing pages, preserve the analytics scripts, nav component, and scarcity badge.
- When writing copy, use the Hormozi Value Equation framing: maximize Dream Outcome and Perceived Likelihood, minimize Time Delay and Effort.
- When creating resume audits, follow `protocols/resume-audit-protocol.md` exactly.
- When referencing the lead magnet, it's called "Impact Bullet Builder Starter Guide" — not "free ebook" or "PDF guide."
- Dual CTA pattern everywhere: primary "See If You're a Fit" + secondary "FREE Resume Guide."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/themanci)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/themanci)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
