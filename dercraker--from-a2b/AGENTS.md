---
description: EXPLAIN how to create a component
globs: *.tsx
---
## Context

* Creating a component must ALWAYS follow this rules.

## Rules

- You always use `export function` without "default".
- You always use an object "props" as the first argument of your component, and add type directly in the object.

## Example

```tsx
type MyComponentProps = { 
  prop1: string; 
  prop2: number;
  prop3: number;
}

export const MyComponent = ({prop1, prop2, prop3}: MyComponentProps) => {
  return <div>{prop1}</div>;
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dercraker)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Dercraker)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
