# Yantr UI Rules

When editing frontend code in this repo, follow these rules.

## Visual System

- Use the existing CSS tokens from `src/assets/main.css`: `--bg-body`, `--text-primary`, `--text-secondary`, `--surface`, and `--surface-muted`.
- Prefer flat surfaces. Do not introduce gradients, glass morphism, `backdrop-filter`, or `backdrop-blur`.
- Do not add borders, rings, or section/card outlines unless a component explicitly requires one. Use `smooth-shadow` or `smooth-shadow-lg` for elevation.

## Interaction

- Every interactive element must have a visible hover state.
- Buttons should include a leading Lucide icon unless space is critically constrained.
- Use existing motion tokens and utilities. Keep animations subtle and purposeful.

## Layout

- Design mobile-first.
- Do not introduce horizontal scrolling.
- Keep tap targets at least 44x44 px.
- Stack multi-column layouts on small screens.

## Apps Directory

- When working in `apps/`, follow `apps/apps.md` as the source of truth.
- For app definitions, keep the required `compose.yml` and `info.json` structure aligned with `apps/apps.md`.
- After adding or editing files in `apps/`, run `check.js` and fix any app-format issues it reports.

## Ports And Cards

- Prefer compact, scannable card or list layouts over dense tables when presenting small sets of app data.
- Keep the primary action obvious and close to the data it acts on.
- For dashboard cards, prefer a minimal identity-card style: quiet header, one strong focal line, at most one or two lightweight detail rows, and a clear bottom action.
- Do not use nested section backgrounds, pill backgrounds, borders, rings, or divider lines to create hierarchy inside dashboard cards.
- Create emphasis with spacing, typography, icon placement, and subtle hover motion instead of extra fills or outlines.

## Consistency

- Use Lucide as the only icon set.
- Preserve the existing typography and theme direction.
- If a new UI choice conflicts with these rules, follow these rules.

---
> Source: [besoeasy/yantr](https://github.com/besoeasy/yantr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-21 -->
