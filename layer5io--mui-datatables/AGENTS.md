# Copilot instructions for `mui-datatables`

## Build, test, and lint commands

- Use Node.js 20+ (`package.json` engines).
- `npm install`
- `npm run dev` starts the webpack example playground at `http://localhost:5050/` using `examples/Router/index.js`.
- `npm run build` runs Prettier across `src/`, `docs/`, and `test/`, then bundles `src/index.js` to `dist/index.js` with Rollup.
- `npm test` runs the Mocha + Enzyme suite in `test/**/*.test.js` through `test/babel-register.js`.
- Run one test file with `npx mocha --require ./test/babel-register.js --extensions js,jsx test/MUIDataTable.test.js`.
- Run one test case with `npx mocha --require ./test/babel-register.js --extensions js,jsx test/MUIDataTable.test.js --grep "should render a table"`.
- `npm run coverage` runs the nyc coverage suite.
- `npm run lint` is the defined lint command and only targets `src/`, but the current ESLint config does not parse the class field syntax used throughout the source. If linting is part of your task, inspect `eslint.config.js` first instead of assuming the failures come from your change.
- If you touch docs, use `npm run docs:dev`, `npm run docs:build`, and `npm run docs:start`.
- `test/babel-register.js` still references `@babel/plugin-proposal-class-properties` and `@babel/plugin-proposal-object-rest-spread`. On a fresh install, test failures for missing Babel plugins usually come from that bootstrap file before they come from the code under test.

## High-level architecture

- `src/MUIDataTable.js` is the stateful orchestrator. It merges default options with incoming props, validates option requirements, restores persisted state from `options.storageKey`, normalizes columns/data, and owns filtering, searching, sorting, pagination, selection, expansion, and callback dispatch.
- The internal data model is not the raw `props.data`. `setTableData()` builds `state.data` as `{ index, data }` rows and `state.displayData` as `{ data, dataIndex }` rows. Most behavior changes should be traced through `buildColumns()`, `transformData()`, `computeDisplayRow()`, and `getDisplayData()`.
- Rendering is compositional. `render()` wires `TableToolbar`, `TableFilterList`, `TableResize`, `TableHead`, `TableBody`, and `TableFooter`, and those same building blocks are part of the public override surface via the `components` prop.
- `src/index.js` re-exports the default table, the core subcomponents, and the debounce search plugin. Changes in `src/components/*` can affect both the default UI and downstream consumers that import those pieces directly.
- Column drag/drop is split between `src/components/TableHead*.js` and `src/hooks/useColumnDrop.js`, then activated in `MUIDataTable.render()` through `DndProvider`.
- There are two separate developer surfaces: `npm run dev` serves the interactive examples from `examples/`, while `docs/` is a separate Next.js app used by the `docs:*` scripts.

## Key conventions

- Column definitions are normalized immediately. The public API accepts either strings or objects, but internal code assumes a full column object with defaults filled in by `buildColumns()`.
- Column visibility is string-based, not boolean-based. Preserve values like `'true'`, `'false'`, and `'excluded'`; rendering logic checks for the exact string `'true'`.
- Data rows can be arrays or objects. Object rows are resolved by column name, and nested object access only works when `options.enableNestedDataAccess` is set to the separator string.
- `customBodyRender` affects more than presentation. Filtering, searching, and CSV export read either the returned string or a React element's `props.value`, so expose a `value` prop when rendered JSX should still behave like a raw data value.
- `customBodyRenderLite` is the lightweight render path; the richer metadata/update path is centered on `customBodyRender`.
- `serverSide: true` is callback-driven and must be paired with `onTableChange`. In that mode, the component skips its normal client-side filtering/search/sort/pagination pipeline.
- Persisted state is opt-in via `options.storageKey`, and the saved snapshot intentionally excludes `selectedRows`, `data`, and `displayData`.
- Styling uses `tss-react/mui` (`withStyles`, `makeStyles`) rather than MUI `sx` patterns. Follow the existing styling approach in `src/components/*` unless you are intentionally migrating a surface.
- Localization follows the `textLabels` shape in `src/textLabels.js` and the README localization examples. When changing labels or adding copy, keep that nested object shape aligned.
- Tests are written with Mocha + Enzyme + jsdom (`test/setup-mocha-env.js`), and the existing suite leans heavily on `shallow()`/`mount()` against internal component structure.
- Prefer the non-deprecated extension points already used by the codebase: `customFilterListOptions.render/update` over `customFilterListRender`, and `options.sortOrder` over per-column `sortDirection`.

---
> Source: [layer5io/mui-datatables](https://github.com/layer5io/mui-datatables) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
