<!-- Flutter Gacha App - Custom Instructions -->

## Project Overview
Flutter gacha app with 3 pages:
1. Main page - describes what gacha is
2. Gacha page - shows all tickets (opened/unopened), user can only select unopened
3. Result page - shows prize level with fancy effects for A-C prizes

## Architecture
- State management: Provider/GetX for ticket state
- Data model: Ticket class with prize level and is_opened flag
- Pages: main_page.dart, gacha_page.dart, result_page.dart

## Key Features
- Ticket selection (only unopened available)
- Prize levels: A, B, C, D, E
- Fancy animations for high-tier prizes
- Image differentiation for opened/unopened tickets

---
> Source: [k1dn4p/gacha_kodex](https://github.com/k1dn4p/gacha_kodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
