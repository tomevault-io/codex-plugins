
---

Ant Design + React Component Structure Rule for Cursor AI

You are a frontend engineer building user interfaces with React, TypeScript, SCSS Modules, and Ant Design (antd). This rule ensures that all UI components are implemented with a consistent, maintainable, and scalable file structure. Cursor AI should strictly follow this convention when scaffolding, refactoring, or generating component code.

Folder & File Structure Convention

When implementing a new feature, always create the following folder layout:

src/
├── components/
│ └── <FeatureName>/
│ ├── <ComponentName>.tsx
│ ├── <ComponentName>.module.scss
│ ├── modal/
│ │ ├── <ModalName>.tsx
│ │ └── <ModalName>.module.scss
│ ├── hooks/
│ │ └── use<Component>.ts
│ ├── types.ts
│ ├── utils.ts
│ ├── constants.ts
│ ├── scss/
│ │ ├── \_variables.scss
│ │ └── \_mixins.scss
│ └── index.ts

Mandatory Conventions

Component Code:

Use functional components with React.FC<Props> or fully typed function declaration.

Style components using .module.scss only. Do not use plain .scss or inline styles.

All stateful logic should be abstracted to hooks/.

Use classnames for conditional classNames.

Component must follow single-responsibility principle.

Modal Components:

Must be stored inside the modal/ subfolder of the feature.

File names must follow <ModalName>.tsx and <ModalName>.module.scss convention.

Modal logic should be decoupled and reusable.

Styling:

Place design tokens and common SCSS logic inside scss/\_variables.scss and scss/\_mixins.scss.

Avoid using hardcoded colors or fonts.

Support Files:

Use types.ts for component-specific type declarations.

Use utils.ts for helpers and formatter functions.

Use constants.ts for enums or default values used in the component.

Export default components via index.ts for easy import.

Implementation Practices

Use Ant Design components by default: <Button />, <Input />, <Form />, <Table />, etc.

Implement validations using Ant Design Form validation or Yup where applicable.

Handle empty states, loading states, and error states with proper UI.

Component should have unit tests using Jest and/or React Testing Library.

For data-fetching UI, include loading skeletons using Skeleton from antd.

Responsive layout must be ensured via Grid or media queries in SCSS.

Commit Message Guidelines

Follow Conventional Commits:

feat: New component or feature

fix: Bug fixes

refactor: Refactor structure or logic

test: Add/modify test cases

style: Style/SCSS only changes

chore: Minor maintenance (e.g. folder rearrangement)

Example (UserProfile Feature)

src/components/UserProfile/
├── UserProfile.tsx
├── UserProfile.module.scss
├── modal/
│ └── EditUserModal.tsx
├── modal/
│ └── EditUserModal.module.scss
├── hooks/
│ └── useUserProfile.ts
├── types.ts
├── utils.ts
├── constants.ts
├── scss/
│ ├── \_variables.scss
│ └── \_mixins.scss
└── index.ts

Final Notes

This rule ensures scalable and organized UI development.

Cursor AI must use this structure even for single-component features.

Consistency is more important than complexity — prioritize clarity, readability, and modularity.

Reuse existing components or hooks wherever possible.

Always update index.ts for easy import access.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duybui2608bku) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
