## Design Context

### Users
Experienced trade contractors — electricians, plumbers, HVAC techs, security integrators. Solo or small crew. Five or more years in the trade. Active QuickBooks users who already know their prices. They use this app on a phone, standing on a job site, at 7am, before the client shows up or right after the job is done. They are not browsing. They are executing. Every second the app takes is a second they're standing in a parking lot.

### Emotional Goal
**"I handled it."** Not relief. Not "in control." The specific feeling is: I did the thing, it's done, I can drive away. The app should feel like crossing something off a mental list before you've even thought about it.

### Brand Personality
**Earned. Crisp. Unsparing.**
- Earned: feels built by someone who's been on a job site. Nothing decorative that doesn't pull its weight.
- Crisp: every number, status, and tap target is exactly where expected. Dense but never cluttered.
- Unsparing: tells you the truth. Bad margin, overdue invoice — here's what's real.

### Typography
- **Headings / amounts / metrics:** Barlow Semi Condensed (500, 600, 700) — specification-sheet character, industrial precision
- **Body / labels / navigation:** Rethink Sans (400, 500, 600) — architectural, precise, not warm
- **Not Outfit** — replaced; it's overused in AI-generated SaaS
- All financial figures: `font-variant-numeric: tabular-nums lnum`

### Color
- Accent: `oklch(40% 0.17 262)` (~`#1B3A8C`) — heavier and more desaturated than the previous Tailwind blue-700
- Dark theme only — ergonomically correct for outdoor/job-site use
- Neutrals tinted subtly toward brand hue (chroma 0.005–0.008)
- No purple, teal, or gradient accents

### Design Principles
1. **Utility over decoration** — every visual element must have a functional purpose
2. **Numbers are the product** — tabular numerals, unambiguous hierarchy, never reflow
3. **State over story** — what's pending, what's done, what needs action — instantly
4. **Confidence without chrome** — no glows, no decorative side stripes, no gradient text
5. **Earned density** — appropriate density is respectful; white space is structural, not decorative

### Hard Rules
- No colored left-border (or right-border) stripes on cards, list items, or callouts
- No gradient text (`background-clip: text`)
- No modal dialogs unless there is genuinely no alternative
- Numbers in tables are left-aligned, never centered
- Status chips: 11px, all-caps, border-style (not filled), 4px radius
- No glowing dark mode aesthetic — no neon accents, no box-shadow glow effects

### References
- Yes: Stripe Dashboard, Linear, Vercel, Knipex pliers (physical)
- No: Jobber, ServiceTitan, FinTech startup visual language, teal/purple SaaS palettes

---
> Source: [tyciccarello08-gif/Tradeflow](https://github.com/tyciccarello08-gif/Tradeflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-22 -->
