
# 50 — Dostępność (WCAG 2.2)

**Wymagane:**
- Kontrast, focus-visible, skip-links, pełna obsługa klawiaturą.
- Łącza i przyciski mają czytelne nazwy dostępności (`aria-label` lub tekst).
- Interakcje asynchroniczne → `aria-busy`, `aria-live` gdzie stosowne.

**Testy**
- **jest-axe/axe-core** dla komponentów.
- **Playwright**: snapshoty a11y i nawigacja klawiaturą.

**Snippety**: `snippets/testing-a11y.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Piotr-Pod)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Piotr-Pod)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
