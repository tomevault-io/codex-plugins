Always respond in Brazilian Portuguese.

Page Isolation Rule (Mandatory)

This project contains multiple HTML pages/screens (e.g. birth, marriage, death),
and they must not break each other due to side effects of changes.

Therefore:

Each HTML page must have distinct and isolated code, including:

its own HTML structure,

page-specific classes/ids when needed,

page-specific JS initialization,

page-specific CSS when applicable.

Any change requested for a specific page (e.g. “birth”) must be applied only to that page, without impacting others.

Do NOT automatically reuse or modify the same HTML/CSS/JS blocks across pages if that may cause cascading breakage.

The only exception is when I explicitly request that a change be applied to all pages.

When reuse is needed, it must be done only via:

neutral shared modules (e.g. src/shared/*),

reusable utility functions,

explicitly imported components/partials,
never by copy-pasting page code.

Before applying any change, clearly determine:

which page is the target (birth/marriage/death),

whether the change is local (single page) or global (all pages).

API Boundary Rule (Mandatory)

Whenever implementing or modifying business logic that:

generates outputs (print, export, save),

processes domain data (validators, mappers),

or may later be moved to a backend service,

you MUST:

Create a clear API boundary (function or interface) that abstracts the operation.

Keep the current implementation local, but design it as if it could be called by an HTTP API in the future.

Do NOT couple UI, DOM, or storage details inside this boundary.

Prefer pure functions or service-like modules with explicit inputs/outputs.

Document the boundary with a short JSDoc explaining how it could be exposed or consumed by an API.

Do NOT implement actual HTTP calls unless explicitly requested.

Domain Reuse Rule (Mandatory)

This project implements multiple “2nd copy” (2ª via) flows (e.g. birth, marriage, death),
which belong to the SAME domain family.

Therefore:

Any logic that is common across 2nd copy flows MUST be implemented once and reused.
Examples include (but are not limited to):

name validation and normalization

CPF validation

date and time validation/formatting

registry number (matrícula) generation and verification

generic text normalization and escaping

shared print utilities

Act-specific modules (birth, marriage, death) MUST:

reuse shared validators, mappers, and utilities whenever possible,

only implement truly act-specific rules and fields.

Before creating new logic inside an act-specific folder, ALWAYS check whether:

a similar rule already exists for another act,

the rule can be generalized and moved to a shared/domain module.

Shared domain logic MUST live in a neutral location (e.g. src/shared/*),
never duplicated across acts.

If a rule is shared today but may diverge in the future,
implement it as a configurable or parameterized function instead of duplicating it.

The goal is to treat all 2nd copy flows as variations of the same domain,
not as independent implementations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edjan11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
