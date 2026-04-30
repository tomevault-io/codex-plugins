# Amrutbaa.com - AI Agent Instructions

## Project Overview
**Amrut Baa** is a production e-commerce site for a 90+ year old Chilly Garlic Chutney from Kathiyawad. The site features a **weekly batch-based ordering system** (not on-demand) with Razorpay payment integration and Cloudflare Workers backend.

## Architecture & Data Flow
- **Frontend**: Single-page HTML (`index.html`) with embedded CSS/JS—no build process
- **Payment Flow**: Modal form → Cloudflare Worker (`worker.js`) → Razorpay API → Payment verification → Success modal
- **Batch Logic**: Weekly deadline is Sunday 9:00 PM (calculated client-side in `main.js` via `getNextSundayDeadline()`)
- **Deployment**: Static files on Cloudflare Pages with Worker routes for payment endpoints

## Critical Patterns

### Weekly Batch System (Non-Negotiable)
- Deadline: **Sunday 9:00 PM** (calculated dynamically per timezone)
- Countdown states in `main.js`: Normal → Urgent (24h) → Critical (3h) → Closed
- Messaging changes based on remaining time (see `displayCountdown()` function)
- Modal footer displays: "Monday: sourced & prepared | Tuesday: dispatched | Wed-Fri: delivered"
- Never expose "slots remaining"—focus on time-based urgency

### Payment Integration (Razorpay)
- `assets/js/modal-component.js` handles modal + form + payment flow
- `worker.js` creates Razorpay orders and verifies signatures
- **Key IDs must be injected**: Set `RAZORPAY_KEY_ID` in worker env vars AND update frontend key in modal-component.js (line ~638)
- Order amounts: Trial ₹199 | Family ₹449 | Duo ₹398 | Quarterly ₹849
- Free shipping on orders ₹399+; ₹59 otherwise

### Color & Design System (CSS Variables in main.css)
```css
--maroon: #6B1C23;           /* Brand primary */
--golden: #C9A961;           /* Accent for urgency/CTAs */
--cream: #FAF7F2;           /* Background */
--charcoal: #1A1A1A;        /* Text */
```
- Urgency states: Add `.urgent` or `.critical` class to trigger color/animation changes
- Animations: `fadeInUp`, `slideInDown`, `glow`, `pulse`, `shimmer` (all CSS)

## Key File Purposes
- **index.html** (2556 lines): Full page with hero, story, product, countdown, dual-form, FAQs, policies
- **main.js** (690 lines): Countdown timer, scroll animations, form validation, navbar effects
- **modal-component.js** (825 lines): Self-contained modal with Razorpay integration
- **main.css** (2887 lines): All styles, animations, responsive breakpoints
- **worker.js** (153 lines): Cloudflare Worker for order creation/verification

## Form Architecture (Two-Step in Modal)
1. **Step 1**: Phone number only (quick commitment)
2. **Step 2**: Name, email, address, quantity, package selection
- Summary box shows pricing dynamically based on selected package
- Validation: Phone (10 digits), email (RFC5322), required fields
- On success: Modal shows order details + batch timeline + next batch countdown

## Common Tasks for Agents

### Update Batch Dates
Search `index.html` for "Next Preparation Date" or "Monday:" timeline marker. Countdown logic auto-calculates from Sunday deadline.

### Change Product Pricing
In `modal-component.js`, update `catalog` object (lines ~175–180). Shipping logic: free if price ≥ ₹399.

### Modify Urgency Messaging
Adjust strings in `main.js` `displayCountdown()` function (lines ~60–90). Update CSS for `.urgent` / `.critical` states in main.css.

### Add Form Validation
Validation logic: `modal-component.js` lines ~400–500. Regex patterns: phone = `/^[0-9]{10}$/`, email = standard RFC5322.

### Test Payment Flow (Dev)
Use Razorpay test mode card: `4111 1111 1111 1111`, any future expiry, any CVV. Worker logs go to Cloudflare dashboard.

## Deployment Notes
- **Local**: `python3 -m http.server 8000` → `http://localhost:8000`
- **Production**: `wrangler pages deploy . --project-name=amrutbaa-com`
- **Secrets**: Set `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` in Cloudflare dashboard (Settings → Variables)
- **Custom Domain**: Add via Pages → Custom domains after first deploy

## Conventions This Project Uses
- **No frameworks**: Pure vanilla JS—no React, no build tooling
- **Inline styles in HTML**: Critical CSS embedded in `<head>` for speed
- **CSS variables for theming**: All colors/spacing defined in `:root`
- **Semantic HTML**: `<section>`, `<article>`, `<header>`, `<footer>` for accessibility
- **Date/time**: All times assume IST (Sunday 9 PM hardcoded); consider timezone if expanding to other regions

## References
- Weekly batch logic: [WEEKLY_BATCH_SYSTEM.md](../WEEKLY_BATCH_SYSTEM.md)
- Payment setup: [RAZORPAY_SETUP.md](../RAZORPAY_SETUP.md)
- Deployment: [DEPLOYMENT.md](../DEPLOYMENT.md)
- Quick operations: [QUICK_REFERENCE.md](../QUICK_REFERENCE.md)

---
> Source: [prinkitpatel/amrutbaa.com](https://github.com/prinkitpatel/amrutbaa.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
