<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

This is an Angular component library workspace. Please generate Angular components, modules, and services using best practices for reusable libraries. Use SCSS for styles and prefix all selectors with 'cmp-'.

## Coding guidelines

### setup
- setup an Angular project with Storybook. Use the latest version and configure it to have stories files next to the components, delete the sample content from Storybook in the `/stories` folder
- check if the project uses Angular libraries or nx libraries and configure Storybook accordingly

### working
- different variations of a component should be managed through inputs on the component itself. For example, `Size = sm` in Figma should convert to a property called size on the component, that also refers to a specific class in the component styles
- try to keep the component HTML clean of any functions related to styling. Applying styles should be managed through properties, corresponding functions should reside in the functional code of the component 
- use encapsulated native HTML elements where possible to avoid re-implementing accessibility features
- make sure you are following WCAG guidelines for accessible components
- component attributes should be kebab-case (e.g. `Hierarchy = Secondary Gray` should convert to hierarchy="secondary-gray")
- make sure to include hover and focus states as well
- analyze the Figma design for each component and ensure that all properties, states, and variations are implemented in the component. Pay attention that all background colors, text colors, and other styles are correctly applied 
- when composing Storybook stories, make sure to include actual samples that load the component in a proper way. For a button, you will also need to provide a template that renders content and connects component attributes. Make sure to have stories for all states of a component, e.g. Primary, Secondary, etc
- use standalone components only, make sure to include them in the correct way in Storybook
- for every stories file you create, also create an .mdx documentation file
- put stories files next to the component files, not in a separate folder
- remember to omit `tags: ['autodocs']` when creating MDX files for components, as this is conflicting
- use CSF for writing stories, do not use the legacy format or render functions if possible
- When using Angular standalone components, always explicitly import any required Angular features (such as NgClass, NgIf, NgFor, etc.) in the component's imports array. Do not assume they are globally available.
- For native HTML attributes that are not standard Angular inputs (like aria-* attributes), always use the [attr.attributeName] binding syntax (e.g., [attr.aria-disabled]) instead of [attributeName].
- Use the new Angular control flow syntax (@if, @for, etc.) instead of legacy structural directives like *ngIf and *ngFor, and ensure any required features (e.g., NgComponentOutlet) are also imported.
- Double-check that all template bindings are valid for the element or component they are used on.
- Do not use the same name for an @Output() event and a method or property in the same component. For example, avoid having both an @Output() click and a method named onClick. Always use unique names for @Output() events to prevent accidental recursion or event binding conflicts.
- When implementing state styles (hover, focus, active, disabled) for components with multiple hierarchies or variants, always scope state styles within each hierarchy/variant selector. Do not define global state styles that apply to all variants, as this can cause state color/style mixing.
- For each hierarchy (e.g., primary, secondary, tertiary), explicitly define the hover, focus, and active state styles inside the corresponding hierarchy's SCSS block. This ensures each variant has its own correct state colors and effects.
- Review and test each variant in Storybook to confirm that its interactive states (hover, focus, active, disabled) match the intended design for that hierarchy.

### Critical lessons learned from implementation challenges

#### Figma integration and component specification
- **Always fetch Figma specifications first** before implementing any component to understand the exact properties, states, and data structures required
- **Read component properties carefully** from Figma specifications and match them exactly in the Angular component - don't assume properties that aren't explicitly defined
- **Check component complexity** - if Figma shows 24 variants, understand how they break down (e.g., 6 types × 4 states = 24 variants) to implement the right combination of properties
- **Use actual Figma property names** when possible, but convert them to Angular-friendly formats (camelCase for TypeScript, kebab-case for templates)

#### Component architecture and type safety
- **Always check existing component exports** before creating stories - use the actual exported class name, not assumed names
- **Match component property types exactly** - if a component uses `string[]`, don't create stories with `{value: string, label: string}[]` objects
- **Read the component implementation** before writing stories to understand the actual input/output interface
- **Keep component types consistent** with Figma specifications - if Figma shows 'default', 'icon-leading', etc., use those exact values in TypeScript enums

#### Storybook stories best practices
- **Always verify component property types** before creating argTypes in stories - incorrect types cause TypeScript errors
- **Use simple data structures** for component options unless the component specifically expects complex objects
- **Match story names to actual component capabilities** - don't create stories for properties that don't exist on the component
- **Test stories compilation** before considering the task complete - TypeScript errors in stories prevent proper functionality

#### File naming and organization
- **Be consistent with file naming conventions** - if files are prefixed with `HELLO-WORLD-`, apply this consistently across all files
- **Check export names** before importing in other files - component class names may differ from file names
- **Update public APIs** immediately after creating components to ensure they can be imported properly
- **Maintain consistent naming** between component files, story files, and MDX files for better organization

#### Development workflow
- **Always verify component compilation** before moving to stories to catch basic TypeScript errors early
- **Check one file at a time** for errors rather than creating multiple files with potential issues
- **Use read_file to understand existing code** before making assumptions about interfaces or naming
- **Update implementation status** in external systems (like Figma) only after successful implementation and testing

#### Error prevention
- **Don't assume property names** - always verify what properties exist on the component before using them in stories
- **Read error messages carefully** - TypeScript errors often indicate exactly what's wrong (e.g., 'Property X does not exist on type Y')
- **Validate component interface** against stories interface to ensure compatibility
- **Test component integration** by checking if it compiles and exports properly before writing stories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreaswissel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
