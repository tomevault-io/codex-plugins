# Agent & Contributor Rules

Canonical operating rules for automated agents and human contributors.
When in doubt, check here first.

---

## Docs authority

| Location | Canonical for |
|---|---|
| `docs/` | Repo-wide truths (architecture, rules, runtimes, external intake) |
| `<dir>/README.md` | That folder's local how-to only |
| `ARCHITECTURE.md`, `LEGACY.md`, etc. (root) | Reference depth — accurate but not the single source of truth for rules |

Do not treat root-level reference docs as authoritative over `docs/`.

---

## Runtimes

Two runtimes share this repo. Understand which one you are touching before making changes.

| Name | Entry file | Status | Role |
|---|---|---|---|
| **SPA** | `spa.html` | Primary (beta) | Vanilla-JS single-page app; hash routing; canvas-heavy |
| **Legacy MPA** | `index.html` → `legacy/pages/home.html` | Stable entrypoint | Frozen multi-page app; current production default |

Rules:
- The SPA is the primary development target. New features go in the SPA.
- The legacy MPA is frozen. Do not add features to it; bug fixes only.
- `index.html` is a redirect shim — do not add logic there.
- SPA files live under `js/spa/`. Legacy JS lives under `legacy/js/`. Do not mix them.

---

## Vocabulary (use consistently)

### Runtime / navigation
| Term | Meaning | Example |
|---|---|---|
| **runtime** | Code executing in the browser; either SPA or legacy | "This change touches the SPA runtime" |
| **entrypoint** | The file a browser opens first | `spa.html`, `legacy/pages/home.html` |
| **route** | URL state; canonical description of sections and items | `js/spa/routes.js` |
| **view** | Top-level SPA screen; one per section | `js/spa/views/*View.js` |
| **section** | Named group of items on the horizontal navigation axis | `home`, `social`, `music` |
| **item** | A single navigable entry inside a section (vertical axis) | `spotify`, `tiktok` |

### Architecture
| Term | Meaning | Example |
|---|---|---|
| **engine** | Self-contained animation or behaviour system | `transitionEngine.js`, `particle-clusters.js` |
| **manager** | Shared app-level coordinator; one instance; global API | `overlayManager.js`, `router.js` |
| **overlay** | Layered UI rendered above all views | managed by `overlayManager.js` |
| **component** | Reusable UI fragment (not a full view) | — |
| **util** | Pure helper with no side-effects | `importantWords.js` |

### Code status
| Term | Meaning |
|---|---|
| **legacy** | Frozen/superseded code (the `/legacy` dir); *not* just "old" |
| **demo** | Showcase or prototype; not used in production | `particlecarousel.demo.js` |
| **WIP** | Incomplete feature that should not be presented as finished |

> **Legacy disambiguation**: "legacy" always refers to the frozen MPA under `/legacy`.
> Use "old", "previous", or "deprecated" for other outdated code to avoid confusion.

---

## View lifecycle contract (SPA)

A view module registers itself on `window.__SPA_Views[sectionId]` and should
implement these methods. All are optional, but cleanup is mandatory when
`onActivate` adds listeners or starts loops.

| Method | Called by | Responsibility |
|---|---|---|
| `mount(itemId, containerEl)` | router | Build DOM once; cache element refs; create engines |
| `onActivate(itemId, viewEl)` | router | Start rAF loops; add per-view event listeners |
| `onDeactivate(itemId)` | router | **Stop rAF loops; remove per-view listeners** |
| `getTransitionCanvas(itemId)` | router | Return a canvas the transition engine can sample |

Rules:
- `onDeactivate` **must** undo everything `onActivate` did. Memory leaks are bugs.
- `mount` is called once per session; do not re-mount the same view.
- Views do **not** read `routes.js` directly; they receive `itemId` from the router.

---

## External / third-party code

All third-party packs enter the repo through `external/`. See [`../external/README.md`](../external/README.md).

Rules:
- Do **not** place vendor archives (`.tgz`, `.zip`, `.tar.gz`) in `js/spa/` or `legacy/`.
- Do **not** commit `node_modules/`.
- After vetting a pack, internalize it into `assets/` (static files) or `js/spa/` (runtime modules).

---

## What not to change (without explicit discussion)

- `index.html` entry-point redirect
- `js/spa/routes.js` section/item names (other views depend on exact IDs)
- `legacy/` code beyond isolated bug fixes
- `data/parsed-stats.json` by hand (it is written by automation)
- GitHub Actions workflows (not in scope for doc-only PRs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/indrolend) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
