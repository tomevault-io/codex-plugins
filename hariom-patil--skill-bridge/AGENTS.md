# Copilot Instructions (SkillBridge)

## Project overview
- Angular 17 app using standalone components (no NgModules). Example: `standalone: true` with `imports: [CommonModule, FormsModule]` in components like Internship Finder.
- The homepage composes feature sections and opens tools in a modal overlay. The modal is controlled by `activeTool` in [src/app/app.component.ts](../src/app/app.component.ts) and rendered via `ngSwitch` in [src/app/app.component.html](../src/app/app.component.html).
- Feature cards emit tool selections via an Output event. Example: `FeaturesSectionComponent` emits `toolSelected`, handled by `openTool()` in the app root.

## UI/UX patterns
- Modal layout lives in [src/app/app.component.html](../src/app/app.component.html) with backdrop and close button styling in [src/app/app.component.scss](../src/app/app.component.scss).
- Individual tools (e.g., Internship Finder) render inside the modal; each tool owns its own HTML/SCSS under [src/app/components](../src/app/components).
- Forms are template-driven with `[(ngModel)]`. See [src/app/components/internship-finder/internship-finder.component.html](../src/app/components/internship-finder/internship-finder.component.html).

## Key directories
- Components: [src/app/components](../src/app/components)
- Services: [src/app/services](../src/app/services)
- Interceptors: [src/app/interceptors](../src/app/interceptors)
- Assets: [src/assets](../src/assets)
- Docs: [md](../md)

## Dev workflows
- Dev server: `ng serve`
- Build: `ng build`
- Tests: `ng test`
(From [md/README.md](../md/README.md))

## Conventions to follow
- Prefer standalone components and local SCSS (component-level styles).
- Keep modal flow consistent: trigger via `openTool()` and close via `closeTool()` in the app root.
- Use `ngModel` bindings for form inputs and keep form state in the component class.
- Keep UI changes within the component’s HTML/SCSS unless the change affects the global modal shell.

## Examples to reference
- Modal shell and tool routing: [src/app/app.component.html](../src/app/app.component.html)
- Tool selection flow: [src/app/components/features-section/features-section.component.ts](../src/app/components/features-section/features-section.component.ts)
- Internship Finder UI/form: [src/app/components/internship-finder/internship-finder.component.html](../src/app/components/internship-finder/internship-finder.component.html)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hariom-paTil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
