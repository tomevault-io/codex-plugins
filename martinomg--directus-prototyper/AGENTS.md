# Copilot instructions

This file contains instructions for using GitHub Copilot effectively in this repository.



## General Guidelines

- Use pnpm for package management
- Use typescript always
- Never delete files when they get corrupted - instead use `git checkout -- path/to/file` to restore them from git
- Always use `pnpm use 20.19.2` before using any npm or pnpm command
- Before confirming any changes, run `pnpm run build` to ensure everything builds correctly

## Vue files

I always use the `<script setup>` syntax in Vue files. When writing Vue components, and with the order
1. `<script setup>`
2. `<template>`
3. `<style>`

## Directus extensions file structure

Directus extensions have the following file structure:

- `./extensions/<name>/package.json`: Contains the extension metadata and dependencies. I always use bundle types, so they do a lot of stuff at the same time.
- `./extensions/<name>/hooks/<filename>.hooks.ts`: contains custom directus hooks.
- `./extensions/<name>/enxpoints/<filename>.enxpoints.ts`: contains custom directus enxpoints.
- `./extensions/<name>/services/<filename>.service.ts`: contains custom directus services.
- `./extensions/<name>/views/<filename>.vue`: contains custom directus views.
- `./extensions/<name>/views/components/<filename>.vue`: contains custom directus view components.
- `./extensions/<name>/views/composables/<filename>.ts`: contains custom directus view component composables.
- `./extensions/<name>/views/utils/<filename>.ts`: contains custom directus view component utils.

## Preferred directus context usage

I always prefer to use the first argument as an object and the second to inject the context.

On hooks: for filters, actions and schedule directus provides the context this way
```js
export default ({ filter, action, schedule }: any, context: any) => {
    // I prefer to always give the context as a second argument
    someFunction({...someArgs}, context);
}
```

On enxpoints: for directus enxpoints, I prefer to use the context as the second argument
```ts
export default (router: any, context: any) => {
    // I prefer to always give the context as a second argument
    router.get('/some-endpoint', (req, res) => {
        someFunction({...someArgs}, context);
    });

}
```

## Adding packages to extensions

```bash
cd path/to/extension && pnpm add <package-name>

# Or update the package.json directly
cd path/to/extension && pnpm install
```     

## Updating extensions

```bash
cd path/to/extension && pnpm build
```

## Event handling

Try most of the time to use the `mitt` library for event handling in Directus extensions. It provides a simple and efficient way to handle events without the overhead of a full-fledged event bus.

## V-model usage

When using v-model in Vue components, I prefer to use the defineModel function from Vue 3's Composition API. This allows for better type inference and reactivity.

## Icons and Styling

### Iconify Icons

I use Iconify for icons in Directus extensions. Install it in each extension that needs icons:

```bash
cd extensions/<extension-name> && pnpm add @iconify/vue
```

Usage in Vue components:
```vue
<script setup lang="ts">
import { Icon } from '@iconify/vue';
</script>

<template>
  <h2 class="flex items-center gap-2">
    <Icon icon="mdi:code-braces" class="text-2xl" />
    Title with Icon
  </h2>
</template>
```

Popular icon sets:
- `material-symbols:` - Material Symbols, which is the default directus icon set, so it's always better to use this. Also the preferred variant is -outline-rounded
- `svg-spinners:` - SVG Spinners for loading indicators
- `logos:` - Logos for brand icons colored
- `simple-icons:` - Simple Icons for popular brands black and white
- `cib:` - CoreUI Brands for black and white icons
- `token:` - Web3 Token Icons black and white
- `skill-icons:` - Skill Icons for technology logos colored
- `circle-flags:` - Circle Flags for country flags colored
- `flagpack:` - Flagpack for country flags colored 4:3 aspect ratio
- `mdi:` - Material Design Icons for a wide range of icons black and white

### Tailwind CSS

Extensions can use Tailwind CSS classes alongside SCSS. The typical pattern is:

```vue
<template>
  <div class="p-10 max-w-screen-2xl mx-auto">
    <div class="text-center mb-10 p-8 bg-gray-50 rounded-lg border">
      <!-- Tailwind classes work directly in templates -->
    </div>
  </div>
</template>

<style lang="scss" scoped>
/* Mix SCSS with Tailwind @apply directives when needed */
.custom-component {
  @apply rounded-lg p-6 text-white;
  
  /* Pure SCSS for complex styling */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  
  &:hover {
    @apply transform scale-105 transition-transform duration-200;
  }
}
</style>
```

## Utility Scripts

The `bin/` directory contains several utility scripts for managing Directus configuration and data:

### bin/sync-backup.sh

Synchronizes Directus configuration from the development instance to local backup files.

```bash
./bin/sync-backup.sh
```

This script:
1. Switches to pnpm version 20.19.2
2. Runs `npx directus-sync pull` with the local backup configuration
3. Pulls the latest configuration from the development Directus instance into `directus-config/snapshot/`

### bin/list-collections.sh

Lists all available collections and their field counts from the local Directus configuration.

```bash
./bin/list-collections.sh
```

Output example:
```
Available collections:

  posts (8 fields)
  directus_sync_id_map (3 fields)
  users (12 fields)

Usage: ./bin/get-collections-fields.sh <collection1,collection2,...>
```

### bin/get-collections-fields.sh

Extracts field configuration data for specified collections and outputs a JSON object.

```bash
./bin/get-collections-fields.sh posts,directus_sync_id_map
```

This script:
1. Takes comma-separated collection names as input
2. Reads all JSON field files for each collection from `directus-config/snapshot/fields/{collection}/`
3. Builds a nested JSON object with the structure:

```json
{
  "posts": {
    "field1": { /* field configuration */ },
    "field2": { /* field configuration */ }
  },
  "directus_sync_id_map": {
    "field1": { /* field configuration */ }
  }
}
```

Use `./bin/list-collections.sh` first to see available collections before running this command.

### bin/get-docker-console-snapshot.sh

Gets a comprehensive snapshot of the Directus Docker container status and logs.

```bash
./bin/get-docker-console-snapshot.sh [OPTIONS]
```

Options:
- `-l, --lines NUMBER` - Number of log lines to show (default: 100)
- `-h, --help` - Show help message

Examples:
```bash
# Show default 100 lines
./bin/get-docker-console-snapshot.sh

# Show last 50 lines
./bin/get-docker-console-snapshot.sh --lines 50

# Show last 200 lines (short form)
./bin/get-docker-console-snapshot.sh -l 200
```

This script:
1. Reads the `DOCKER_DIRECTUS_CONTAINER_NAME` from the `.env` file
2. Checks if the container is running
3. Shows recent logs (configurable number of lines)
4. Displays container status and port mappings
5. Shows current CPU/Memory usage stats

Use this command to quickly diagnose what's happening with the Directus application in Docker.

### bin/get-extension-console-snapshot.sh

Gets a comprehensive snapshot of an extension's build status and information.

```bash
./bin/get-extension-console-snapshot.sh <extension-name>
```

Example:
```bash
./bin/get-extension-console-snapshot.sh prototyper
```

This script:
1. Validates the extension exists in the `extensions/` directory
2. Switches to pnpm version 20.19.2
3. Runs `pnpm build` and captures the output
4. Shows package.json details (name, version, main, scripts)
5. Displays console log files (log/console.log or log/console.txt)
6. Lists build artifacts in `dist/` directory
7. Shows recent file changes

Use this command to quickly diagnose extension build issues and get a complete picture of the extension's current state.

### bin/build-extension.sh

Builds a specific Directus extension with proper environment setup.

```bash
./bin/build-extension.sh <extension-name>
```

Example:
```bash
./bin/build-extension.sh prototyper
```

This script:
1. Validates the extension exists in the `extensions/` directory
2. Switches to pnpm version 20.19.2
3. Runs `pnpm build` in the extension directory
4. Shows build success/failure status
5. Lists build artifacts in `dist/` directory
6. Returns the build exit code

Use this command to quickly build extensions with the correct environment setup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martinomg)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/martinomg)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
