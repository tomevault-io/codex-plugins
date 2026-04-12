## InKCre Core — 快速上手（给 AI 编码代理）

目标：让 AI 代理能快速理解本仓库的使命、主要结构、常用约定与可执行的开发/调试步骤。

- 项目入口：`run.py`（定义了 FastAPI `api_app`，并在模块直接运行时使用 `uvicorn.run`）。在开发时常用命令：

  - 启动（虚拟环境存在情况下）：`python -m uvicorn run:api_app --reload` 或使用 PDM：`pdm run uvicorn run:api_app --reload`
  - 关键环境变量：`DB_CONN_STRING`（用于 `app/engine.py` 中的 `create_engine`）。

- 架构总览（大致）：

  - API 层：`run.py` + `app/routes/*`（例如 `app/routes/block.py`, `app/routes/relation.py`）。
  - 业务逻辑：`app/business/*`（`block`, `relation`, `root`, `resolver`, `source`, `extension`, `sink`, `storage`）。每个子模块提供 Manager/Resolver 抽象。
  - 数据层：`app/engine.py`（使用 SQLModel/sqlalchemy，`SessionLocal()`），`app/schemas/*` 包含 SQLModel/Pydantic 定义。
  - 调度：`app/task.py` 使用 `apscheduler`；`run.py` 的 lifespan 会启动/关闭调度器。

- 关键开发约定与模式（从代码可观察到的具体约定）：

  - Session 模式：直接使用 `with SessionLocal() as db_session:` 或通过 `get_db_session()` 作为 FastAPI 依赖（见 `app/engine.py` 和 `app/business/root.py`）。
  - fetchsert 风格：Manager 类通常提供 `fetchsert`（fetch-or-insert）语义，用于幂等插入（见 `app/business/root.py` 对 `BlockManager`/`RelationManager` 的用法）。
  - Resolver 插件：所有解析器放在 `app/business/resolver/`，Resolver 子类（例如 `webpage.py` 的 `WebpageResolver`）实现 `create_brs` / `get_text` 等方法，并通过 `StorageManager.new_storage(block)` 获取内容。
  - Extension 系统：扩展由 `app/business/extension.py` 管理，项目根 `extensions/` 包含第三方/外部扩展（例如 `extensions/twitter`），扩展通常提供 API 客户端和 resolver/handler。

- 交互点与外部依赖（可在 `pyproject.toml` 中查到）：

  - 数据库：Postgres（通过 `psycopg2-binary` 和 `sqlmodel`），连接由 `DB_CONN_STRING` 环境变量提供。
  - 模型/LLM：依赖 `openai`, `litellm`（注意：具体调用点分散在业务层与 libs）。
  - 网络/抓取：`aiohttp`, `requests`, `html2text`（在 `app/business/resolver/webpage.py` 中使用 `html2text` 转换 HTML 到文本）。

- 代码改动/贡献要点（给代理的具体建议）：

  - 修改数据库模型/迁移时，请检查 `migrations/`，并保持 `sqlmodel` 模型与 Alembic 迁移一致。
  - 新增 Resolver：放在 `app/business/resolver/`，继承 `Resolver` 并实现 `create_brs`（用于把外部资源转换为 StarGraphForm）和必要的提取方法（参考 `webpage.py`）。
  - 与存储交互应通过 `app/business/storage` 的 `StorageManager.new_storage(block)` 获取存储实例，避免直接在 resolver 中做 I/O 抽象外的假设。

- 参考文件（优先阅读顺序）：
  1. `run.py` — 服务入口与 lifecycle（了解启动流程、CORS 与扩展注册）
  2. `app/engine.py` — DB session 与 `DB_CONN_STRING` 环境变量
  3. `app/business/root.py` — 核心的“星形图”插入逻辑，展示 Manager/Session 使用模式
  4. `app/business/resolver/webpage.py` — Resolver 模式的最小示例
  5. `app/business/extension.py` + `extensions/` — 扩展加载与第三方集成示例

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InKCre)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/InKCre)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
