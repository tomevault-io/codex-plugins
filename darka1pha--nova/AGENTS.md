Repository: nova (Next.js project generator)

Build / Dev / Test / Lint commands

All root scripts are package-manager-agnostic: they never recurse into a
specific package manager's binary (e.g. no `npm run x --workspace=y`
embedded inside a script body), so `npm run <script>`, `pnpm <script>`,
`yarn <script>`, and `bun run <script>` all behave identically. Pick
whichever package manager you have installed. pnpm additionally requires
`pnpm-workspace.yaml` at the repo root (already present) since pnpm does
not read the `"workspaces"` field in package.json the way npm/yarn/bun do.

- Root scripts (run from repo root, with any package manager):
  - `<pm> install`
  - `<pm> run build` # compiles @nova/core (tsc) then bundles the CLI (tsup) - no workspace-specific flags involved
  - `<pm> run dev` # runs CLI against source via tsx (fast iteration)
  - `<pm> run typecheck` # tsc --noEmit for @nova/core, then tsc --noEmit for the CLI
  - `<pm> run verify:manifest-sync` # regression guard: buildPackageJson() must match FEATURE_CONTRIBUTIONS per feature
  - `<pm> run start` # runs compiled CLI (node ./bin/nova.js)

  Examples: `npm install && npm run build`, `pnpm install && pnpm build`,
  `yarn install && yarn build`, `bun install && bun run build`.

- Building/typechecking just the `@nova/core` workspace package directly
  (bypasses any package-manager workspace filtering entirely):
  - `tsc -p packages/core/tsconfig.json` (build)
  - `tsc --noEmit -p packages/core/tsconfig.json` (typecheck)

- Smoke test / generator validation:
  - `node scripts/smoke-test.mjs`

- Manifest sync check (dependency-drift regression guard):
  - `tsx scripts/verify-package-manifest-sync.ts`

- Inspecting plugins from the CLI:
  - `nova plugins` / `nova plugins <feature>` - backed by src/generator/pluginInfo.ts.

High-level architecture (big picture)

- Purpose: CLI generator that copies a base Next.js template and overlays optional add-on folders to produce a ready project.
- Core areas:
  - src/index.ts : CLI entrypoint; dispatches `nova [name]`, `nova add`, and `nova plugins`
  - src/generator/index.ts : high-level generator orchestration (builds an operation plan, executes it, runs config patchers, writes package.json). Moved here from src/generator.ts so it lives alongside its supporting modules in src/generator/.
  - src/generator/ : generator internals - index.ts (orchestration), context.ts, logger.ts, errors.ts, hooks.ts, operations.ts, pluginMetadata.ts, pluginInfo.ts, validators.ts, verifyManifestSync.ts, and patchers/ (next.config/providers/middleware)
  - src/featureContributions.ts : single source of truth for what each feature contributes to package.json (dependencies/devDependencies/scripts); consumed by packageManifest.ts (full generation), featurePackageAdditions.ts (`nova add`, a thin re-export), and pluginInfo.ts (`nova plugins`)
  - templates/base/ : base Next.js App Router project (complete app layout, providers, docs)
  - templates/addons/ : optional feature overlays (prisma, vitest, cypress, msw, sentry, ui libraries, etc.). Add-ons are plain folders that overwrite base files when applied.
  - packages/core/ : framework-agnostic shared utilities (fs/template copying, pm commands, prompts handling, logging). Built and then bundled into the CLI.
  - bin/nova.js : published CLI entrypoint (imports ../dist/index.js, built via tsup's `index` entry)
  - scripts/smoke-test.mjs : end-to-end smoke test; imports { generateProject } from "../dist/generator.js" (tsup's `generator` entry, mapped from src/generator/index.ts - filename stays "generator.js" despite the source move, see tsup.config.ts)
  - scripts/verify-package-manifest-sync.ts : regression guard asserting buildPackageJson() output matches FEATURE_CONTRIBUTIONS per-feature
- Workflow summary: CLI prompts -> generateProject (src/generator/index.ts) validates the plugin selection -> builds an operation plan (copy base, copy selected addons, copy UI overlay) -> executes the plan (rolling back the target dir on failure) -> packageManifest builds package.json from feature set + FEATURE_CONTRIBUTIONS -> config patchers (next.config/providers/middleware) run -> optional git init / install

Key conventions and repository-specific notes

- Add-on overlay model: any folder under templates/addons with matching path names will be copied on top of templates/base. Overlay files win (they replace base files) — there's no merge logic.
- UI overlays: templates/ui/\* (mui, chakra, or default shadcn primitives) are overlaid last to wire providers and example components.
- Config patching: next.config.mjs, the provider tree, and middleware.ts are patched via ordered declarative contributions in src/generator/patchers/\*.ts (feature flag -> transform), not ad hoc string-replace in src/generator/index.ts.
- Plugin metadata: src/generator/pluginMetadata.ts holds per-plugin name/description/requires/conflicts/supportedUI, checked by src/generator/validators.ts before any files are written, and surfaced to users via `nova plugins` (src/generator/pluginInfo.ts).
- Package manifest generation: a feature's dependencies/devDependencies/scripts are declared exactly once, in src/featureContributions.ts. src/packageManifest.ts (full generation), src/featurePackageAdditions.ts (`nova add`), and src/generator/pluginInfo.ts (`nova plugins`) all consume it directly - do not add feature-specific deps/scripts anywhere else. `<pm> run verify:manifest-sync` guards against regressions here.
- @nova/core is a real workspace package (packages/core). Run install at repo root before build/dev so editors and tsc path mappings (tsconfig) resolve it. pnpm users need pnpm-workspace.yaml (present at repo root) in addition to package.json's "workspaces" field.
- Package-manager independence: no root script recurses into a specific package manager's binary or uses a package-manager-specific flag (like npm's `--workspace=`). Every script invokes tsc/tsup/tsx directly via `-p <path-to-tsconfig>`, so the exact same script body works correctly no matter which of npm/pnpm/yarn/bun ran it.
- tsup entries are named explicitly ({ index: "src/index.ts", generator: "src/generator/index.ts" }) rather than passed as a bare array, specifically because both files are named index.ts - tsup derives output basenames from entry names when given an object, avoiding an index.js/index.js collision. If you add another "index.ts"-named entry point, extend this same object rather than the array form.
- Fast iteration: use `<pm> run dev` (root) to run CLI against source via tsx so changes to src/ are picked up without rebuilding.
- Typechecking/build: `build` compiles `@nova/core` directly via `tsc -p packages/core/tsconfig.json`, then runs `tsup` to produce the bundled CLI; `prepublishOnly` chains typecheck (core + root) + verify:manifest-sync + build.
- Smoke tests: scripts/smoke-test.mjs exercises many add-ons and validates presence of key files — useful reference for expected outputs.

Files to check when changing generator behavior

- src/generator/index.ts
- src/generator/patchers/\*.ts (config patching contributions)
- src/generator/pluginMetadata.ts, src/generator/pluginInfo.ts, and src/generator/validators.ts
- src/featureContributions.ts (single source for feature -> package.json contributions)
- templates/base/** and templates/addons/**
- packages/core/src/fs.ts and src/pmCommands.ts (copying and package-manager resolution for _generated projects_ - not to be confused with the repo's own root scripts)
- scripts/smoke-test.mjs (for generation expectations)
- tsup.config.ts (if adding a new top-level entry point)

AI-assistant guidance

- When proposing changes to generated project structure, update templates/ and src/featureContributions.ts together.
- If a feature adds dependencies/scripts, add them only to src/featureContributions.ts - packageManifest.ts, nova add, and `nova plugins` all pick it up automatically. Do not hand-edit featurePackageAdditions.ts (it's a re-export).
- If a plugin has real cross-plugin constraints (requires another plugin, conflicts with one, or only supports certain UI libraries), declare them in src/generator/pluginMetadata.ts so validation and `nova plugins` both reflect it.
- When adding or editing a root package.json script, never call back into a package manager (`npm run ...`, `pnpm ...`, `yarn ...`, `bun run ...`) from inside another script's body - call the underlying tool (tsc/tsup/tsx/etc.) directly instead.
- References to the generator module use "./generator/index.js" (or "../generator/index.js" from deeper files) - not "./generator.js" - since the move to src/generator/index.ts.
- Prefer small, surgical edits: add-ons intentionally overwrite base files; be explicit when adding conflict-prone files.
- Use smoke-test.mjs locally to validate changes to generation behavior.

MCP Servers

- Would you like to configure any MCP servers relevant to the project (e.g., Playwright or Storybook)? If yes, specify which one to configure.

Summary
Created .github/copilot-instructions.md capturing build/test commands, architecture overview, and repo-specific conventions. Want any additions (e.g., more commands, CI notes, or deeper mapping of add-ons to scripts)?

---
> Source: [darka1pha/nova](https://github.com/darka1pha/nova) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-03 -->
