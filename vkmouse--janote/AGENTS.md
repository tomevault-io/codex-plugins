# JaNote — Copilot Instructions

JaNote 是一個**離線優先 (Offline-first) 個人記帳管理 PWA**，部署於 Cloudflare Pages + Workers + D1。前端採用 Vue 3 / TypeScript，本地資料庫使用 Dexie (IndexedDB)，後端以 Cloudflare Pages Functions (TypeScript) 實作，認證透過 Cloudflare Access (JWT)。

---

## 目錄結構

```
/
├── .devcontainer/          # VS Code DevContainer 設定
├── .github/
│   └── copilot-instructions.md
├── functions/              # 後端：Cloudflare Pages Functions
│   ├── _middleware.ts      # 全域 JWT 認證中介層
│   ├── types.ts / types.d.ts
│   ├── api/
│   │   ├── sync.ts         # POST /api/sync — 核心同步 API
│   │   ├── health.ts       # GET  /api/health
│   │   └── initdb.ts       # POST /api/initdb — D1 schema 初始化
│   ├── repositories/       # D1 資料庫查詢層（server-side）
│   │   └── ...
│   ├── services/           # 業務邏輯層（server-side）
│   │   └── ...
│   └── utils/
│       └── validators.ts
├── src/                    # 前端：Vue 3 應用程式
│   ├── main.ts
│   ├── App.vue
│   ├── types.ts            # 前端型別定義
│   ├── router/
│   │   └── index.ts        # Vue Router 路由設定
│   ├── stores/             # Pinia 狀態管理
│   │   └── ...
│   ├── views/              # 頁面元件
│   │   └── ...
│   ├── components/         # 可重用 UI 元件
│   ├── db/
│   │   ├── index.ts        # Dexie 資料庫實例（IndexedDB）
│   │   ├── tables.ts       # Dexie schema 定義與版本遷移
│   │   └── repositories/   # Dexie 查詢層（client-side）
│   │       └── ...
│   ├── utils/
│   │   ├── serviceWorkerUpdate.ts  # PWA 更新邏輯
│   │   ├── recurrence.ts
│   │   ├── database.ts
│   │   ├── icons.ts
│   │   └── categoryIcons.ts
│   └── assets/
├── server/
│   └── index.ts            # Cloudflare Worker 入口（wrangler main）
├── public/                 # 靜態資源（PWA icons 等）
├── docker-compose.yml
├── wrangler.jsonc          # Cloudflare Workers / Pages 設定
├── vite.config.ts
└── SYNC.md                 # 同步功能詳細技術文件
```

---

## 技術棧

| 層級       | 技術                                          |
| ---------- | --------------------------------------------- |
| 前端框架   | Vue 3.5、TypeScript 5                         |
| 狀態管理   | Pinia 3                                       |
| 路由       | Vue Router 5                                  |
| 本地資料庫 | Dexie 4（IndexedDB ORM）                      |
| PWA        | vite-plugin-pwa（Workbox，manual SW control） |
| 建置工具   | Vite 7                                        |
| 後端平台   | Cloudflare Pages Functions（TypeScript）      |
| 雲端資料庫 | Cloudflare D1（SQLite）                       |
| 認證       | Cloudflare Access（JWT via `jose`）           |
| 部署       | `wrangler deploy`（Cloudflare Pages）         |

---

## 本機開發

### 方式一：Docker Compose

```bash
docker compose up      # 啟動容器（port 5173 Vite + 8888 Wrangler）
```

容器內會執行 `npm run dev:full`，同時啟動 Vite dev server (5173) 與 Wrangler dev proxy (8888)。  
已啟用 `CHOKIDAR_USEPOLLING=1` 解決 Docker volume 檔案監聽問題。

### 方式二：VS Code DevContainer

開啟 VS Code，在 `.devcontainer/` 設定下重開容器（Reopen in Container），post-create 會自動執行：

```bash
npm ci && npm install -g wait-on
```

接著手動啟動：

```bash
npm run dev:full   # 完整開發模式（Vite + Wrangler proxy）
npm run dev        # 僅 Vite（不含 Wrangler Functions）
```

### 常用指令

```bash
npm run dev          # Vite dev server（前端只）
npm run dev:full     # Vite + Wrangler proxy（完整 stack）
npm run build        # type-check + vite build
npm run type-check   # vue-tsc 型別檢查
npm run deploy       # build + wrangler deploy（部署到 Cloudflare）
npm run cf-typegen   # 重新產生 Cloudflare binding 型別
```

> **D1 binding：** `wrangler.jsonc` 中 D1 資料庫 binding 名稱為 `DB`，本機開發使用 Wrangler 自動建立的本地 SQLite。

---

## 同步功能架構

> 完整技術規格見 [SYNC.md](../SYNC.md)。以下為 Copilot 快速參考。

### 核心概念

- **Offline-first**：所有資料變更先寫入本地 Dexie，再非同步推送至伺服器。
- **Server as Source of Truth**：衝突時以伺服器版本為準。
- **Optimistic Locking**：每筆實體有 `version` 欄位；推送時帶 `base_version`，伺服器若已有更新版本則回傳 `SKIPPED`。
- **Mutation ID（UUID）**：每次本地寫入產生唯一 ID，確保 API 冪等性。
- **Cursor**：`sync_events` 表的自增 ID，客戶端記錄 `last_cursor` 拉取增量更新。

### 資料流

```
UI 呼叫 service.create/update/delete
  → 1. 寫入本地 Dexie entity table（樂觀更新）
  → 2. 寫入 sync_queue（含 snapshot_before, base_version, mutation_id）

syncStore.performSync()
  → 3. 讀取 sync_meta.last_cursor
  → 4. POST /api/sync { push_commands, last_cursor }

後端 functions/api/sync.ts
  → 5. 驗證 Cloudflare Access JWT
  → 6. Push Phase：冪等性 + 版本衝突檢查 → 寫入 D1 + sync_events
  → 7. Pull Phase：撈出 last_cursor 之後的 sync_events（排除本次 mutation）
  → 回傳 { push_results, pull_events, new_cursor }

前端套用回應
  → 8. Pull events：強制覆蓋本地 entity，清除同 entity_id 的佇列項目
  → 9. Push results：OK → 更新 version；SKIPPED → 移除佇列；ERROR → snapshot_before rollback
  → 10. 存入 new_cursor 到 sync_meta
```

### 實體類型

| EntityType | 說明                             |
| ---------- | -------------------------------- |
| `CAT`      | 類別（Category）                 |
| `TXN`      | 交易（Transaction）              |
| `BGT`      | 預算（Budget）                   |
| `SHR`      | 使用者分享（UserShare）          |
| `RTXN`     | 週期交易（RecurringTransaction） |
| `RBGT`     | 週期預算（RecurringBudget）      |

**處理順序**：`CAT > TXN/BGT/RTXN/RBGT > SHR`（確保外鍵約束正確）

### 關鍵檔案

- **前端同步狀態機**：[src/stores/syncStore.ts](../src/stores/syncStore.ts)
- **後端同步 API**：[functions/api/sync.ts](../functions/api/sync.ts)
- **本地 DB schema**：[src/db/tables.ts](../src/db/tables.ts)、[src/db/index.ts](../src/db/index.ts)
- **Dexie repositories**：`src/db/repositories/`
- **D1 repositories**：`functions/repositories/`

### 注意事項

- `sync_queue` 產生的 `snapshot_before` 欄位儲存 **修改前的完整快照** JSON，rollback 時直接還原。
- 軟刪除：所有 entity 使用 `is_deleted = 1` 旗標，不實際刪除資料列。
- `syncStore` 統一管理同步鎖（避免並發呼叫），透過 `syncStatusStore` 對 UI 暴露進度。

---

## PWA 版本更新

> 相關邏輯集中於 [src/utils/serviceWorkerUpdate.ts](../src/utils/serviceWorkerUpdate.ts)。

- PWA 使用 **manual Service Worker 控制**（`skipWaiting: false`）。
- 前端啟動時透過 `vite-plugin-pwa` 提供的 `useRegisterSW` 偵測新 SW 進入 **waiting** 狀態。
- 確認有待更新的 SW 後，透過 `postMessage({ type: 'SKIP_WAITING' })` 觸發 `skipWaiting`。
- 5 秒未啟動則強制 `window.location.reload()`。
- SW controller 切換（`controllerchange` 事件）後自動 reload 頁面。
- Dev 模式下 PWA 強制啟用，方便本機測試更新流程。

---

## 認證

- 所有 `/api/*` 請求由 `functions/_middleware.ts` 攔截，驗證 `Cf-Access-Jwt-Assertion` header 的 JWT。
- JWT 使用 Cloudflare Access JWKS（`TEAM_DOMAIN`）驗證，需設定環境變數 `POLICY_AUD`、`TEAM_DOMAIN`。
- 本機開發若未配置 Cloudflare Access，middleware 回退至 `demo@example.com`。

---

## 架構慣例

- **Repository pattern**：DB 查詢封裝在 `repositories/`，業務邏輯在 `services/`，頁面邏輯在 Pinia `stores/`。
- **前後端各有一套 repositories**：`src/db/repositories/`（Dexie / IndexedDB）和 `functions/repositories/`（D1 / SQLite），命名相同但實作不同。
- **型別定義**：前端共用型別在 `src/types.ts`；後端在 `functions/types.ts`、`functions/types.d.ts`。
- **Dexie schema 升版**：修改 `src/db/tables.ts`，新增 `version().stores()` migration，不修改舊版本。
- **Entity 型別**：交易分類為 `EXPENSE | INCOME`（`EntryType`）。

### 前端資料流

```
views / components
       ↓  呼叫 action / 讀取 computed
    stores（Pinia）
       ↓  呼叫 repository 方法
  repositories（src/db/repositories/）
       ↓  透過 Dexie ORM 存取
      db（IndexedDB）
```

> **重要**：views 與 components **不得**直接 import `src/db/repositories/` 或 `src/db/index.ts`。所有資料存取一律透過對應的 Pinia store。

### Icon 使用規範

- **唯一出口**：所有 SVG icon 以 `?raw` 方式 import 後，**統一在 `src/utils/icons.ts` 匯出**。
- **禁止直接 import**：components 與 views **不得**直接 `import xxx from "...assets/icons/..."?raw`，一律從 `src/utils/icons.ts` 引入。
- **新增 icon**：在 `src/utils/icons.ts` 加入 import 與 export，再於需要的元件中從該檔案引入。

### 備註語言規範

撰寫程式碼備註（comments）時，使用**繁體中文**描述。變數名稱、關鍵字、技術術語（如 `type`、`interface`、`async/await`）除外。

### 修改完成標準

每次修改前端程式碼後，**必須執行 `npm run type-check` 並確認通過**，才算修改完成。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vkmouse)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vkmouse)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
