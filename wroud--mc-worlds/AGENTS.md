# MC-Worlds Agent Instructions

## Project Overview
Fabric Minecraft mod that allows server admins to create/manage multiple worlds via in-game commands.
- **Mod ID:** `mc-worlds`
- **Version info:** See `gradle.properties` and `versions/latest/gradle.properties`
- **Published on:** Modrinth and CurseForge (see `build.gradle.kts` publishMods block)

## Architecture

### Core Flow
```
McWorldMod (ModInitializer) → McWorld (facade) → WorldsManager (orchestrator)
  → WorldHandle (world ref) → CustomServerLevel (extended ServerLevel)
                            → WorldsLevelData (config/persistence)
```

### Key Responsibilities
| Class | Responsibility |
|-------|---------------|
| `McWorldMod` | Mod entry point, event registration |
| `McWorld` | Per-server facade for world management |
| `WorldsManager` | Load/create/delete/unload worlds |
| `WorldHandle` | Data holder: id + level + levelData |
| `CustomServerLevel` | Extends ServerLevel, state machine (Init→Active→Stopping→Stopped), independent WeatherData |
| `WorldsLevelData` | World config (seed, provider, spawn, lazy flag), persisted via Codec |
| `WorldsCreator` | Factory for new worlds (preset/dimension/seed resolution) |
| `WorldsData` | SavedData container for all worlds, Codec-serialized |
| `DefaultServerLevelProvider` | Registered as `mc-worlds:default`, creates CustomServerLevel |
| `WorldsRegistries` | Fabric registry for `ServerLevelProvider` implementations |

## Key Files (read these first for any task)

- **Entry point:** `src/main/java/dev/wroud/mc/worlds/McWorldMod.java`
- **Core manager:** `src/main/java/dev/wroud/mc/worlds/manager/WorldsManager.java`
- **World level:** `src/main/java/dev/wroud/mc/worlds/server/level/CustomServerLevel.java`
- **World data:** `src/main/java/dev/wroud/mc/worlds/manager/level/data/WorldsLevelData.java`
- **Commands:** `src/main/java/dev/wroud/mc/worlds/command/` (CreateCommand, DeleteCommand, TeleportCommand, SettingsCommand, WorldsCommands)
- **Mixin config:** `src/main/resources/worlds.mixins.json` (server), `src/client/resources/worlds.client.mixins.json` (client)
- **Build config:** `build.gradle.kts`, `gradle.properties`, `versions/latest/gradle.properties`
- **Mod metadata:** `src/main/resources/fabric.mod.json`

## Build & Run

```bash
# Build the mod JAR
./gradlew build

# Run Minecraft client with mod (dev environment)
./gradlew runClient

# Run Minecraft server with mod (dev environment)
./gradlew runServer

# Run data generation
./gradlew runDatagen

# Check build only (no JAR)
./gradlew assemble
```

Build output: `build/libs/worlds-<version>.jar`

## Mixin Packages & Their Purpose

Mixins use ASM bytecode injection (via Mixin framework) to patch closed-source Minecraft classes at runtime.

| Package | Purpose |
|---------|---------|
| `mixin/fixes/` | Vanilla mechanics fixes for custom dimensions (portals, maps, entities) |
| `mixin/filefix/` | Data format migration (hooks into MC's DataFixers system) |
| `mixin/` (root) | Core patches: server init, weather, wandering trader spawner |
| `client/mixin/` | Client-side patches (packet handling, dimension context) |

**When adding a new mixin:**
1. Create the class in the appropriate package under `src/main/java/dev/wroud/mc/worlds/mixin/`
2. Register it in `src/main/resources/worlds.mixins.json` under the `"mixins"` array (or client equivalent)
3. Use `@Mixin`, `@Inject`, `@Redirect`, or `@ModifyVariable` annotations

## Patterns & Conventions

### Codec Serialization
World data uses DFU Codec API for serialization/deserialization:
```java
public static final Codec<WorldsLevelData> CODEC = RecordCodecBuilder.create(instance -> ...);
```

### State Machine (CustomServerLevel)
World lifecycle: `InitializationLevelState → ActivationLevelState → ActiveLevelState → StoppingLevelState → StoppedLevelState`
- Auto-unload after 1200 ticks (60s) of no players

### Registry Pattern
Custom registries via Fabric API:
```java
// Register custom level provider
WorldsRegistries.SERVER_LEVEL_PROVIDER.register(id, myProvider)
```

### Command Pattern
Commands use Brigadier (MC's command library):
- All commands require `ServerLevel.LEVEL_ADMINS` permission
- Commands are registered in `WorldsCommands.java`; each command in its own class

### Translation/Localization
All user-facing strings use translation keys:
```java
Component.translatable("command.worlds.create.creating", worldId)
```
Keys defined in `src/main/resources/assets/mc-worlds/lang/en_us.json`

## Common Tasks

### Adding a New Command
1. Create `src/main/java/dev/wroud/mc/worlds/command/MyCommand.java`
2. Register it in `WorldsCommands.java`
3. Add translation keys to `src/main/resources/assets/mc-worlds/lang/en_us.json`
4. Add corresponding datagen entries under `src/datagen/`

### Adding a New Mixin
1. Create class in `src/main/java/dev/wroud/mc/worlds/mixin/`
2. Add class name to `src/main/resources/worlds.mixins.json` under `"mixins"` array
3. Annotate with `@Mixin(TargetClass.class)`

### Adding World Persistent Data
1. Add field to `WorldsLevelData`
2. Add to its `CODEC` definition
3. Add getters/setters
4. Access via `CustomServerLevel.getWorldData()` which returns `WorldsLevelData`

### Updating Minecraft/Mod Version
- Version variables: `versions/latest/gradle.properties` (minecraft_version, java_version) and `gradle.properties` (mod_version, loader_version, fabric_version)
- Update `build.gradle.kts` if plugin/API versions changed
- Fix any broken mixin targets (Minecraft internals change between versions)
- Run `./gradlew build` to verify

## Minecraft Source Code

Fabric Loom decompiles and remaps Minecraft into named source JARs stored in the Gradle cache. These are the actual Minecraft classes this mod patches via mixins.

**Mappings:** Mojang official mappings (Mojmap) — class names like `ServerLevel`, `MinecraftServer`, `WeatherData` are Mojmap names. The explicit mappings line in `build.gradle.kts` is commented out; Loom uses default/embedded Mojmap behavior.

### Source JAR location

```
.gradle/loom-cache/minecraftMaven/net/minecraft/
├── minecraft-common-<hash>/<version>/
│   ├── minecraft-common-<hash>-<version>-sources.jar   ← server + shared source
│   └── minecraft-common-<hash>-<version>.jar           ← compiled bytecode
└── minecraft-clientOnly-<hash>/<version>/
    ├── minecraft-clientOnly-<hash>-<version>-sources.jar  ← client-only source
    └── minecraft-clientOnly-<hash>-<version>.jar
```

The `<hash>` and `<version>` values are derived from the configured `minecraft_version`.

### How to access Minecraft source in IDE

Loom auto-configures the IDE to attach these sources to the `minecraft` dependency. In IntelliJ or VS Code with the Java extension:
- **Ctrl+Click** (or **Cmd+Click**) any Minecraft class/method to jump to its decompiled source
- The IDE opens files directly from inside the `-sources.jar`

### Regenerating sources

```bash
# Regenerate decompiled sources (run after upgrading MC version)
./gradlew genSources
```

### Reading source manually

The sources JARs are standard ZIP archives. To extract and browse:
```bash
# Extract server sources to a temp directory
unzip .gradle/loom-cache/minecraftMaven/net/minecraft/minecraft-common-*/*/minecraft-common-*-sources.jar -d /tmp/mc-src
```

Relevant packages for this mod:
- `net.minecraft.server.level` — `ServerLevel`, `ServerPlayer` (core targets)
- `net.minecraft.server` — `MinecraftServer`
- `net.minecraft.commands` — Brigadier command classes
- `net.minecraft.world.level` — `LevelData`, `WeatherData`, `SavedData`
- `net.minecraft.world.level.storage` — level storage and data fixers

## No Test Suite
There are no automated tests. Manual testing via `./gradlew runServer` is the standard QA approach.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wroud)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Wroud)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
