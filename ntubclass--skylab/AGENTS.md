# SkyLab — Copilot Instructions

> 本檔為 GitHub Copilot CLI / Copilot Chat 等 Agent 的工作守則。
> **更詳盡的開發指引請參考 repo 根目錄的 [`CLAUDE.md`](../CLAUDE.md)**——本檔是其精簡版，兩者衝突以 `CLAUDE.md` 為準。

## 專案速覽

SkyLab 是 Proxmox VE 管理平台，提供：VM/LXC 生命週期、VNC/Terminal Console、批次配置、防火牆/NAT/Gateway 網路、AI 輔助運維、群組多租戶。

- **Backend**: FastAPI + SQLModel + PostgreSQL + Redis + Proxmox API + SSH (paramiko) — `uv` 套件管理
- **Frontend**: React 19 + TypeScript + Vite + TanStack Router/Query/Table + Tailwind 4 + shadcn/ui + i18next — `bun` 套件管理
- **Infra**: Docker Compose + Traefik

## 必要指令

### Backend
```bash
cd backend
uv sync                                 # 安裝
fastapi dev app/main.py                 # 本地 dev server
uv run ruff check --fix .               # lint + fix
uv run mypy .                           # 型別檢查
bash ./scripts/test.sh                  # 跑測試
# Alembic（在 docker container 內執行）
alembic revision --autogenerate -m "..." && alembic upgrade head
```

### Frontend
```bash
cd frontend
bun install
bun run dev                             # http://localhost:5173
bun run build                           # tsc + vite build
bun run lint                            # Biome check + fix
bash ./scripts/generate-client.sh       # 重生 OpenAPI client（後端 API 改動後必跑）
```

### 全棧
```bash
docker compose watch                    # 啟動全部含 hot reload
docker compose exec backend bash        # 進 backend container
```

## 架構分層（Backend）

```
api/routes/      ← 薄薄的 REST controller，只負責驗證+委派
api/websocket/   ← VNC、Terminal proxy
schemas/         ← Pydantic 請求/回應 schema
models/          ← SQLModel DB 表 + enum
services/        ← 業務邏輯（ai/, llm_gateway/, network/, proxmox/, resource/, scheduling/, user/, vm/）
infrastructure/  ← 外部系統整合（proxmox/, redis/, ssh/, ai/, worker/）
core/            ← config, db, security, request_context
exceptions.py    ← AppError 自訂例外（raise AppError(status, msg)）
main.py          ← FastAPI app、lifespan（Redis init + scheduler 啟停）
```

**核心原則**：`Routes → Services → Infrastructure`，**不可反向依賴**。Models 只放 DB，Schemas 只放 API I/O，**不要混用**。

## 維護者鐵則（節錄自 CLAUDE.md）

1. **不要把外部連線細節寫回 service**（Proxmox/SSH/LLM 必須在 `infrastructure/`）
2. **不要把業務規則寫回 route**（route 只是 controller）
3. **不要在多個地方複製權限判斷**（用 `core/security.py` 的依賴）
4. **不要讓單一 service 變成上帝物件**（按子領域拆檔）
5. **新增 async 時先處理高 I/O 熱點**，不要一次全 async 化
6. **重構優先保留 facade**，相容層最後才刪
7. **嚴禁 silent fallback**：捕到例外就 raise 或記 ERROR，**不可吞掉錯誤後改用預設值**繼續跑（曾因此導致 IP 設定 silently 退回 DHCP，debug 4 小時）

## 常見坑

- **Frontend client 是自動生成的**：`frontend/src/client/` 千萬不要手改。後端動 API → 跑 `generate-client.sh`
- **Models 改了一定要建 Alembic migration**（startup 會跑 `scripts/prestart.sh`）
- **WebSocket 雙向轉發**：VNC/Terminal proxy 一定要用 `asyncio.Event` 同步 disconnect，避免「receive after disconnect」error
- **SQLModel scalar select**：`session.exec(select(Model.column)).all()` 回傳 **scalar list**，不是 row list — 不要用 `row.column`
- **`.env` 永遠別 commit**，用 `.env.example` 為範本
- **Logging format**：用 `%s` 不要用 `%d`，因為很多 vmid 在傳遞時會被序列化成 str
- **環境變數**：在 root 的 `.env`，由 `core/config.py` 透過 `env_file="../.env"` 載入

## 部署/Provision 流程關鍵

- VM 請求 → scheduler (`services/scheduling/coordinator.py`) → `_provision_via_service_template` 或一般 provisioning
- 服務模板部署走 `services/network/script_deploy_service.py`（SSH 到 Proxmox node 跑 community-scripts）
- 部署去重：`_ACTIVE_BY_REQUEST` dict + Lock（避免 scheduler 重跑與 API 立即觸發雙觸發）
- 部署取消：`script_deploy_service.cancel_task(task_id)` 設 cancel event → SSH streaming loop 中斷 → except 走 rollback（destroy container + release IP）
- IP 分配：**必須**從 `services/network/ip_management_service` 拿 `cidr/gateway/dns`，不要 fallback 到 DHCP

## 程式風格

- **Backend**: Ruff + Mypy strict + prek pre-commit
- **Frontend**: Biome（lint + format）+ TypeScript strict
- **註解**：只在需要解釋意圖時才加，不要重複描述程式碼字面行為
- **i18n**：所有面向使用者的字串走 i18next，**不要硬編碼中/英文**

## Tool 使用

- 改檔案優先 `edit`，多個 edit 同 turn 並行
- 探索檔案用 `grep` + `glob`，不要直接 `view` 整個大檔
- 環境是 Windows，路徑用 `\`

---
> Source: [ntubclass/SkyLab](https://github.com/ntubclass/SkyLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
