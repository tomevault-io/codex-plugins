# Copilot instructions for this repo (CrediExpress)

This is a small React + TypeScript app built with Vite and Tailwind. It implements a 3‑step pre‑qualification wizard with a results screen. Use these notes to quickly contribute safely and consistently.

## Build, run, structure
- Run locally: `npm run dev` (Vite). Build: `npm run build`. Preview prod build: `npm run preview`.
- No routing library; the wizard is controlled by `step` state in `src/App.tsx` and animated via Framer Motion.
- Global form state lives in `FormProvider` (`src/context/FormContext.tsx`) and persists to `sessionStorage` under `crediexpress:data`.
- UI styling: Tailwind + a few utility classes in `src/index.css` (e.g., `card`, `btn-primary`, `error`). Colors and radii defined in `tailwind.config.cjs`.
- Assets are in `public/` and referenced as `/...` paths (Vite serves them statically).

## Key modules and data flow
- Wizard pages: `src/pages/Step1Identity.tsx` → `Step2Vehicle.tsx` → `Step3Review.tsx` → `Result.tsx`. Navigation and splash handled in `src/App.tsx`.
- Context shape (`FormContext.tsx`):
  - `Applicant { idNumber, fullName, maritalStatus: "single"|"married", spouseId? }`
  - `Loan { vehicleAmount: number, downPaymentPct: number (0–1), termMonths: number }`
  - Persisted default set in `defaultData`. Update context via `setData`, which also persists.
- Validation: Zod schemas in `src/utils/validators.ts` used through `react-hook-form` resolvers. Example: `identitySchema` also enforces spouse ID when married.
- Services:
  - `src/services/api.ts`: mocked async `submitPrequalification(FormData)`; returns `{status: "approved"|"review"|"denied"}` based on financed amount and term.
  - `src/services/calculator.ts`: `monthlyPayment(amount, months, monthlyRate=0.02)`; beware `downPaymentPct` is stored as 0.xx, UI sliders use percent.
- Components:
  - `Input.tsx`: thin wrapper around `<input>` integrated with react-hook-form and shows `FieldError`.
  - `Stepper.tsx`: animated progress bar; pass `current` and `total`.
  - `HeroLayout.tsx`: two‑column hero + form container; imports `index.css` and uses Framer Motion for subtle motion.

## Conventions and patterns
- Keep all form source of truth in `FormContext`; pages read/update via `useFormData()`. Always update the full `FormData` object when calling `setData` so persistence stays consistent.
- Persisted key is `crediexpress:data`. If you change `FormData`, migrate defaults and reads accordingly to avoid JSON parse or undefined field bugs.
- Validation lives in `utils/validators.ts`; when adding fields, extend the Zod schema and wire it via `zodResolver` in the page. Show errors using `Input`’s `error` prop.
- Maintain the percent vs fraction contract: UI sliders may show percent (e.g., 20) but context stores fraction (e.g., 0.2). Convert explicitly when saving/reading.
- Animations wrap page transitions with `AnimatePresence` and `motion.div` in `App.tsx`. If adding steps, copy the transition pattern for consistency.
- Text/UI copy is Spanish; keep tone consistent in new UI.

## Common changes and how to do them
- Add an Identity field (e.g., phone):
  1) Extend `Applicant` and `defaultData` in `FormContext.tsx`.
  2) Extend `identitySchema` in `utils/validators.ts`.
  3) Add an `<Input>` in `pages/Step1Identity.tsx` with `register("phone")` and include it in `defaultValues` and in the `onSubmit` mapper.
  4) Show it in `pages/Step3Review.tsx` if needed.
- Adjust approval logic: edit thresholds in `services/api.ts`; keep return shape `{status}` stable.
- Change payment calc assumptions: update `monthlyRate` default in `services/calculator.ts` and reflect any explanatory copy in `Step2Vehicle.tsx`.

## Gotchas
- `downPaymentPct` is fraction in context; convert to percent only in UI. `Step2Vehicle` uses local state in percent and converts on save.
- Strict TS is enabled; prefer explicit types on context, props, and service returns.
- No test setup is present; if you add tests, keep them optional and avoid breaking the current scripts.

## File map (quick refs)
- App shell and step flow: `src/App.tsx`
- Global state/persistence: `src/context/FormContext.tsx`
- Pages: `src/pages/*`
- Validation: `src/utils/validators.ts`
- Services: `src/services/api.ts`, `src/services/calculator.ts`
- Styling: `src/index.css`, `tailwind.config.cjs`

---
> Source: [nabokov223u/Crediexpress](https://github.com/nabokov223u/Crediexpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
