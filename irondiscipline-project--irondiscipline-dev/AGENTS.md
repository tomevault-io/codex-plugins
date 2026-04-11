# IronDiscipline-dev – Copilot Instructions

A Folia-exclusive Minecraft plugin (Paper API 1.20.4) for military/prison RP servers. Rank data is stored in the plugin's own H2/MySQL database — LuckPerms is **not** used.

## Build & Test

```bash
# Build (produces shaded JAR in target/)
mvn clean package

# Run all tests
mvn test

# Run a single test class
mvn -Dtest=RankManagerTest test

# Run a single test method
mvn -Dtest=RankManagerTest#testPromote test
```

## Architecture

### Package layout

| Package | Purpose |
|---|---|
| `xyz.irondiscipline` | Main plugin class (`IronDiscipline.java`) – wires everything together |
| `command/` | One class per command; registered in `IronDiscipline#registerCommands()` |
| `listener/` | Bukkit event listeners (e.g., `AsyncPlayerPreLoginListener` loads rank cache) |
| `manager/` | All business logic; managers are fields on `IronDiscipline` and accessed via getters |
| `model/` | Pure data: `Rank` (enum), `JailRecord`, `KillLog`, `RadioChannel` |
| `util/` | `TaskScheduler` (Folia wrapper), `RankUtil`, `InventoryUtil`, `TabNametagUtil` |

### Manager responsibilities

- **`ConfigManager`** – reads `config.yml`; resolves i18n messages from `locales/messages_<locale>.yml`
- **`StorageManager`** – H2/MySQL async DB for jail records, kill logs, and warnings
- **`RankStorageManager`** – same DB connection, separate table (`player_ranks`) for rank data
- **`RankManager`** – in-memory `ConcurrentHashMap` cache over `RankStorageManager`; cache is populated during `AsyncPlayerPreLoginEvent` and cleared on quit
- **`DiscordManager`** – JDA 5 bot; optional, disabled when `discord.bot_token` is empty

### Startup flow

`onEnable` → init DB → `initializeManagers()` → `registerListeners()` → `registerCommands()` → start Discord bot

The single `Connection` created in `IronDiscipline` is shared between `StorageManager` and `RankStorageManager`.

## Key Conventions

### Folia scheduling — never use `Bukkit.getScheduler()`

All scheduling goes through `plugin.getTaskScheduler()` (wraps MorePaperLib):

```java
// Async work
plugin.getTaskScheduler().runAsync(() -> { ... });

// Player/entity-bound (required for anything touching a Player)
plugin.getTaskScheduler().runEntity(player, () -> { ... });

// Region/location-bound
plugin.getTaskScheduler().runRegion(location, () -> { ... });

// Repeating global task
plugin.getTaskScheduler().runGlobalTimer(runnable, delayTicks, periodTicks);
```

### Database operations — always use async variants

All DB methods have `*Async` counterparts returning `CompletableFuture`. The synchronous versions are `@Deprecated` and warn when called on the main thread.

```java
// Good
storageManager.isJailedAsync(uuid).thenAccept(jailed -> { ... });

// Bad (deprecated)
storageManager.isJailed(uuid);
```

For upserts, H2 uses `MERGE INTO ... KEY (player_id)` and MySQL uses `INSERT ... ON DUPLICATE KEY UPDATE`. Both code paths must be maintained when editing SQL in `StorageManager` or `RankStorageManager`.

### Messages and i18n

- `configManager.getMessage("key")` → prefix + colorized message (for player feedback)
- `configManager.getMessage("key", "%placeholder%", value)` → with replacements (pairs of placeholder/value)
- `configManager.getRawMessage("key")` → colorized, no prefix (for logger output)
- Locale files live in `src/main/resources/locales/messages_<locale>.yml`; default locale is `ja_JP`
- In-game color codes use `&` prefix (translated via `ChatColor.translateAlternateColorCodes`)

### Rank system

`Rank` is an enum ordered by `weight` (10 = PRIVATE … 100 = COMMANDER). Ranks are compared with `rank.isHigherThan(other)` / `rank.isLowerThan(other)`, never by ordinal or weight directly. `Rank.fromId(String)` and `Rank.fromWeight(int)` are the safe constructors (both fall back to `PRIVATE` on unknown input).

### H2 driver shading

The H2 driver is relocated to `xyz.irondiscipline.lib.h2` by the shade plugin. Driver loading always tries the relocated name first, then falls back to `org.h2.Driver`:

```java
try { Class.forName("xyz.irondiscipline.lib.h2.Driver"); }
catch (ClassNotFoundException e) { Class.forName("org.h2.Driver"); }
```

### Tab/nametag updates

After any rank or division change, call `TabNametagUtil.updatePlayer(player, rank, divisionDisplay)` inside a `runGlobal` block. Scoreboard operations (`registerNewTeam` etc.) require the global region thread in Folia — using `runEntity` causes `UnsupportedOperationException`.

## Addon System

### インストール方法（3モード）

アドオンはゲーム内コマンド `/iron addon install` でインストールします。

```
# 公認アドオン（irondiscipline.xyz のレジストリに登録済み）
/iron addon install <id>

# GitHub Release から最新 JAR を取得
/iron addon install <owner/repo>

# 直接 URL を指定（HTTPS のみ、上限 50 MB）
/iron addon install https://example.com/myaddon.jar
```

### その他のコマンド

| コマンド | エイリアス | 説明 |
|---|---|---|
| `/iron addon list` | `ls` | インストール済みアドオン一覧 |
| `/iron addon certified` | `cert`, `av` | 公認アドオン一覧（レジストリキャッシュを表示） |
| `/iron addon remove <id>` | `rm`, `uninstall` | アドオンをアンインストール（JAR 削除） |
| `/iron addon refresh` | — | 公認レジストリキャッシュを強制更新（TTL: 1時間） |

### アドオン配布の前提条件

配布する JAR には **`irdi-addon.yml`** をルートに含める必要があります。含まれていない場合はインストール時に拒否されます。

```yaml
# irdi-addon.yml の必須フィールド
id: myaddon
name: My Addon
version: 1.0.0
description: 説明文
author: yourname
api-version: "2.0"
github: owner/repo   # オプション
```

### 開発時の API 依存関係

アドオン開発者はまず API モジュールをローカルにインストールします。

```bash
# リポジトリルートで実行
mvn install -pl api
```

その後 `pom.xml` に追加：

```xml
<dependency>
  <groupId>xyz.irondiscipline</groupId>
  <artifactId>IronDiscipline-API</artifactId>
  <version>2.0.0-dev</version>
  <scope>provided</scope>
</dependency>
```

詳細仕様は `docs/ADDON_DEVELOPMENT.md` を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IronDiscipline-Project)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/IronDiscipline-Project)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
