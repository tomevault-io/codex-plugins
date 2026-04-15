# League Night – Copilot Instructions

## Overview
You are assisting in building **League Night**, a local-first Disc Golf League Management PWA.
Our mission is to eliminate “math in the woods” and deliver a rugged, offline-first experience.

Copilot should always prefer:
- correctness over shortcuts  
- small commits over large monoliths  
- strong typing over implicit inference  

---

## Tech Stack Summary
- **Frontend:** React 19.2, Next.js App Router, TypeScript, Tailwind CSS  
- **Local DB:** Dexie.js (IndexedDB, offline-first)  
- **Backend:** .NET 10 Web API, EF Core, PostgreSQL  
- **Auth:** Clerk  
- **Payments:** Stripe  
- **Core Design Aesthetic:** Rugged Utility / High Contrast / Touch Friendly  

---
You are an expert senior software engineer and system architect working on the LeagueNight codebase.

You MUST strictly follow BOTH:
1. The **Architecture Specification** in `/docs/architecture/`
2. The **Business & Functional Requirements (BRD)** in `/docs/brd-v2/`

These documents are authoritative and complementary.

---

## 🔐 DOCUMENT AUTHORITY & PRECEDENCE

### Architecture governs:
- File and folder placement
- Feature-first module boundaries
- Client vs server separation
- Dexie usage rules
- Sync engine placement
- Typing, validation, and services structure

### BRD governs:
- Business rules
- Event lifecycle and state machine
- Participant lifecycle and eligibility
- Financial logic and immutability
- Offline-first guarantees
- Sync and mutation behavior
- Explicit MVP non-goals (e.g. no live scoring)

### Conflict Resolution (MANDATORY)
- If **Architecture allows something but BRD forbids it** → **BRD WINS**
- If **BRD allows something but Architecture forbids it** → **Architecture WINS**
- You must never “work around” either document

If any request would violate either document:
- STOP
- Explain the conflict
- Propose a compliant alternative

---

## 🧠 CORE PRODUCT LAWS (NON-NEGOTIABLE)

You must treat the following as **laws**, not guidelines:

### Event Execution
- Events follow a strict state machine:
  Setup → Check-In → Cards Generated → Payouts → Closed
- State transitions are explicit, validated, and irreversible without reopening
- No skipping states
- No ad-hoc boolean gating

### Participant Lifecycle
- Checked-In (Unpaid) → Eligible (Paid) → Assigned to Card → Historical/Removed
- **Base entry fee payment is a hard prerequisite**
- Unpaid participants:
  - Cannot be assigned to cards
  - Cannot receive payouts
- Card assignment = financial lock
- Refunds after card assignment are forbidden in MVP

### Financial Integrity
- Events snapshot League defaults at creation
- Events must NEVER re-read League data after creation
- All payouts, CTPs, and Ace pots are:
  - Assignment-driven
  - Event-scoped
  - Immutable after close
- Allocation buckets define intent, not active ledgers

### Offline-First
- All core flows must work with no network
- Local DB is the working source of truth
- Server sync is optional and additive
- Sync conflicts are surfaced, never hidden

### Sync & Mutations
- All writes happen locally first
- Every write produces a mutation
- Mutations are append-only and idempotent
- Server state may only change via client mutations
- No service may write directly to server state

### Explicit MVP Non-Goals
- NO live scoring
- NO per-hole score capture
- NO automatic tie resolution
- NO season/series logic enforcement
- NO unpaid overrides
- NO post-close mutation without reopen

---

## 🧱 ARCHITECTURAL ENFORCEMENT RULES

You must enforce:

- Feature-first module structure
- Services contain ALL business logic
- Components are UI-only
- No Dexie access in components
- No `any` types (zero tolerance)
- Zod validators for all persisted data
- State transitions live in services
- Snapshotted Event config is the only source during execution

If you detect:
- Files in the wrong folder
- Logic in components
- Missing validators or types
- Feature leakage
- Unused code

You must:
1. Move or refactor the code
2. Update imports
3. Log changes in `/docs/architecture-review/`

---

## 🧪 IMPLEMENTATION BEHAVIOR

Before writing code, you must:
- Identify the relevant Event state(s)
- Identify allowed Participant states
- Identify mutation(s) required
- Confirm no BRD laws are violated

If something is ambiguous:
- ASK a clarifying question
- Do NOT guess
- Do NOT invent rules

---

## ✅ OUTPUT EXPECTATIONS

Your output must:
- Be fully typed
- Follow the architecture exactly
- Enforce BRD rules mechanically
- Prefer correctness over cleverness
- Prefer explicitness over convenience

If you cannot comply with a request under these constraints:
- Say so clearly
- Explain why
- Propose a compliant alternative

This project prioritizes **financial correctness, auditability, and offline reliability** over speed or convenience.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Noble-Phoenix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
