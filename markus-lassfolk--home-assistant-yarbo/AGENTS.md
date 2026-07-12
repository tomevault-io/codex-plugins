# GitHub Copilot Configuration

## Review Scope
Copilot is configured for **code review only** on this repository.
No automated branch creation or PR generation.

## Context
This is a Home Assistant HACS custom integration for Yarbo robot mowers.
Key patterns:
- Async/await throughout (HA convention)
- Type annotations on all functions
- DataUpdateCoordinator pattern (push-based, no polling interval)
- Entity unique IDs: `{robot_sn}_{entity_key}`
- Never import paho-mqtt directly — use python-yarbo library

---
> Source: [markus-lassfolk/home-assistant-yarbo](https://github.com/markus-lassfolk/home-assistant-yarbo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-12 -->
