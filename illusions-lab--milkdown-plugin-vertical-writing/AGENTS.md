# Copilot instructions

## Milkdown compatibility updates

- This package supports Milkdown 7.x with a public peer dependency floor of `^7.5.1`.
- Milkdown `7.5.1` through `7.5.7` were not published; use `7.5.8` for the minimum-version compatibility run without changing the declared peer floor.
- Do not widen the peer dependency range or add Milkdown 8 support. A major update requires a separate human review.
- Keep fixes narrowly scoped to the Dependabot Milkdown update. Do not change release workflows, repository security settings, or dependency-update policy.
- Preserve the vertical-writing DOM contract: Milkdown's resolved root receives `milkdown-vertical-writing`, `data-writing-mode`, optional `data-line-length`, and `--milkdown-vertical-writing-line-length`. Hot switching must preserve the ProseMirror DOM and document content. Teardown must restore the root's prior class, attributes, and custom property.
- Preserve logical scroll progress across writing-mode changes. Vertical wheel input may map to the horizontal block axis, but a nested vertical scroller that can consume the gesture must retain it.
- Update README and the English, Japanese, and Traditional Chinese documentation when a user-visible API or behavior changes.

Before approving a Milkdown update, run:

```text
npm run check
npm run build
npm run test:package
npm run docs:build
npm run test:browser -- --project=chromium
```

Also confirm that every required GitHub check for the current PR head commit succeeded. Before pushing any new commit, remove any previous approval marker from the PR body. Only after the final commit and its complete checks pass, add exactly one marker to the PR body using the current full 40-character head SHA:

```text
<!-- copilot-compatibility-approved: HEAD_SHA -->
```

Never copy an approval marker from an older commit and never approve based on partial or pending checks.

---
> Source: [illusions-lab/milkdown-plugin-vertical-writing](https://github.com/illusions-lab/milkdown-plugin-vertical-writing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-03 -->
