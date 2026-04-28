When declaring functions on the top-level of a module,
declare their return types. This will help future AI
assistants understand the function's purpose.

```ts
const myFunc = (): string => {
  return "hello";
};
```

One exception to this is components which return JSX.
No need to declare the return type of a component,
as it is always JSX.

```tsx
const MyComponent = () => {
  return <div>Hello</div>;
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaloLebrin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
