
## Coding paradigm

- TypeScript. Infer type as much, annotate when necessary
- When import types, prefix them with `type` keyboard. We have `verbatimModuleSyntax` enabled
- Functional core with RxJS
- Effectives are handled on the top layer in main.ts
- Functions over classes, but use class when internal state is complex
- Pure functions are in /src/lib/\*.ts
- Views are templated with lit-html, stored in /src/views/\*.ts

## Workflow

- Do NOT run `npm run dev` for user.
- Do NOT run `npm run build` for user.
- If user provided new information or changed requirements, you must update .github/copilot-instructions.md and .github/instructions/\*.instructions.md to stay up to date.

## Coding style

- Compact and efficient
- Use empty lines judiciously to separate logical blocks
- Minimum comments, only when necessary

## Styling

- Use latest CSS features, including variables, grid, flexbox
- Nesting is ok
- Component specific styles should be in the {component-name}.css file, next to {component-name}.ts
- Component CSS should nest all internal styles under a root class, e.g. `.component-name { ... }`
- The component ts file should import its own css file
- Global styles are in /src/main.css

## File organization

/index.html contains the app root
/src/main.ts is the main view and logic of the app.
/src/lib/ contains lower level functions
/src/components/ contains reactive UI components using the createComponent pattern
/src/views/ contains legacy UI components that are being migrated to /src/components/

## main.ts

- Top down organization, high level first, low level later
- Get any static global dom reference first
- Create shared state using BehaviorSubject observables
- Instantiate components and pass shared state as props
- Render components to their respective DOM containers

## RxJS patterns in main.ts

- Shared state is hoisted to main.ts and passed down to components
- Components handle their own internal state and effects
- No manual subscriptions needed in main.ts as components manage their own reactive behavior

## Components (src/components/)

- Use the createComponent utility for reactive components
- Follow the pattern: props -> internal state -> actions -> effects -> template
- Components receive shared state as Observable props and return reactive templates
- Effects are handled internally using RxJS merge with ignoreElements()

## views/\*.ts (legacy)

- These are being migrated to components/
- They should be pure functions that take data and return lit-html templates
  - Input: any observables it depends on
  - Output: lit-html template, and observables if it created new ones for other components to consume
- Avoid querying DOM (they are not type checkable). Instead pass in observables as parameters

## lib/\*.ts

- They should generally be observables, operators, or pure functions

---
> Source: [chuanqisun/courseek](https://github.com/chuanqisun/courseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
