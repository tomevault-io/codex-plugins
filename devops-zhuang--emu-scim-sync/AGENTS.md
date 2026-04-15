# emu_scim_sync 工作区说明

- 工作说明
   本项目中的所有交互，请首选中文；代码和代码注释请使用英文。
- 代码风格
   目前没有强制的代码风格工具，但请尽量保持一致性和可读性。
   
## 项目目的

这个仓库是一个概念验证项目，用于把 21V 运营的 Microsoft Entra ID 中的用户生命周期数据，通过 GitHub SCIM 2.0 REST API 同步到 GitHub Enterprise Managed Users。

当前范围刻意保持收敛：

- 同步多个 Entra 安全组中的 direct members 用户。
- 支持创建、更新、软停用、重新启用，以及受控可选硬删除。
- 支持将 Entra 组及其 direct members 成员关系同步到 GitHub Enterprise SCIM Groups。
- 可以作为本地脚本运行，也可以通过 Azure Functions 定时器运行。
- 默认启用 `DRY_RUN=true`，优先保证安全。

开始动手前，先阅读 [README.md](../README.md)，了解高层流程和环境变量。

## 技术栈

- Python 3
- Azure Functions Python worker，用于定时触发执行
- `requests`，用于调用 Graph 和 GitHub SCIM HTTP 接口
- `python-dotenv`，用于本地加载环境变量
- `pytest` 风格的轻量测试布局，当前测试位于 [tests/test_mapping.py](../tests/test_mapping.py)

依赖定义位于 [requirements.txt](../requirements.txt)。

## 常用命令

使用现有项目环境，不要自动创建新的虚拟环境。

- 安装依赖：`pip install -r requirements.txt`
- 本地执行一次同步：`python -m src.main`
- 运行测试：`pytest`

如果需要验证 Azure Functions 行为，重点查看 [src/function_app.py](../src/function_app.py)，并使用开发环境里已经配置好的本地 Functions 工作流。

## 架构概览

- [src/main.py](../src/main.py)：本地单次执行入口，负责组装配置、客户端、状态存储和同步引擎。
- [src/function_app.py](../src/function_app.py)：Azure Functions 定时器入口。
- [src/config.py](../src/config.py)：加载环境变量，并推导中国区 token 与 Graph 端点。
- [src/graph_client.py](../src/graph_client.py)：按 group displayName 解析安全组并读取 direct members 用户。
- [src/github_scim_client.py](../src/github_scim_client.py)：封装 GitHub EMU SCIM API 的读取与写入。
- [src/sync_engine.py](../src/sync_engine.py)：核心同步规则与 payload 映射逻辑所在。
- [src/state_store.py](../src/state_store.py)：把已同步用户、组、运行元数据和删除保护状态写入本地 JSON。
- [src/models.py](../src/models.py)：定义源用户和统计结果的数据模型。
- [tests/test_mapping.py](../tests/test_mapping.py)：最小映射回归测试。

## 项目约定

- 代码和代码注释保持英文。
- 非代码说明、评审、总结和文档优先使用中文。
- 改动保持最小化，并聚焦同步工作流本身。
- 除非需求确实改变数据结构，否则保持现有 dataclass 模型不变。
- 优先扩展现有 client 和 sync engine，不要轻易引入新的抽象层。
- 新增测试时，遵循当前轻量风格，优先覆盖映射逻辑和决策逻辑。

## 配置规则

- 必填环境变量校验位于 [src/config.py](../src/config.py)。
- 该项目默认面向 21V 中国区：
  - `ENTRA_TOKEN_URL` 默认指向 `login.partner.microsoftonline.cn`
  - `GRAPH_BASE_URL` 默认指向 `microsoftgraph.chinacloudapi.cn`
- GitHub SCIM 需要 classic PAT，且必须具备 `scim:enterprise` 权限，并带上非空 `User-Agent`。
- `DRY_RUN` 默认值为 `true`，不要随意改动这个默认值。

如果变更配置行为，需要同时更新 [src/config.py](../src/config.py) 和 [README.md](../README.md)。

## 状态与副作用

- 本地同步状态存储在 [state](../state) 目录下，由 `STATE_FILE` 指定具体文件。
- 这个状态文件用于判断哪些历史已同步用户已经离开当前同步范围，从而执行软停用。
- 任何涉及 `externalId`、用户名匹配规则、状态持久化语义的修改，都可能导致误停用或重复创建，必须谨慎。

## 常见陷阱

- 不要默认使用全球版 Entra 端点；除非需求明确变更，否则这里是中国区场景。
- 变更行为时不要移除 `DRY_RUN` 保护。
- 不要在未考虑 Azure Functions 文件系统特性的前提下替换状态存储机制。
- 不要在未明确提出需求时增加 group-to-team 同步；这不在当前 PoC 范围内。
- 不要把 nested groups 的成员自动纳入同步范围；当前语义只取 direct members。
- 修改 [src/sync_engine.py](../src/sync_engine.py) 中的比较逻辑时，必须同时验证更新路径和重新启用路径。

## 推荐改动模式

- 新增映射字段：如有需要先更新 [src/models.py](../src/models.py)，再在 [src/graph_client.py](../src/graph_client.py) 取值，在 [src/sync_engine.py](../src/sync_engine.py) 映射，并补充或更新测试。
- 新增 SCIM 写入行为：HTTP 细节保留在 [src/github_scim_client.py](../src/github_scim_client.py)，业务决策保留在 [src/sync_engine.py](../src/sync_engine.py)。
- 新增运行配置项：在 [src/config.py](../src/config.py) 中添加，同时更新 [README.md](../README.md)，并保持默认值安全。

## 新增功能规则建议

当准备添加新功能时，默认遵守以下规则：

- 先定义功能是否会影响 create、update、soft deprovision、reactivate 四条主路径。
- 先判断该功能属于“字段映射扩展”“同步决策扩展”“外部接口扩展”还是“运行配置扩展”，不要把多类变化混在一个提交里。
- 任何可能引起写操作变化的功能，都必须优先支持 `DRY_RUN` 观察结果。
- 任何改变用户匹配方式的功能，都必须说明是否影响 `externalId` 和 `userName` 的兼容性。
- 任何新增字段同步，都要明确数据来源、默认值、空值策略和是否需要双向兼容旧状态。
- 任何与停用、恢复、重名匹配有关的功能，都应先补测试，再改逻辑。
- 如果功能会扩大同步范围，例如从单组扩展到多组，必须先重新设计状态文件语义。
- 如果功能会引入生产化依赖，例如 Blob、Table、SQL、队列或审计日志，应把基础设施与同步逻辑分开处理。

## ASPICE 文档沉淀规范

如果当前工作采用 ASPICE 角色化协作方式，默认将阶段性工作产物沉淀在 [docs/aspice](../docs/aspice) 目录下。

- 按角色和阶段使用固定子目录，不要把不同阶段文档混放。
- 每个阶段文档都应显式写明输入、输出、风险、假设和 handoff 对象。
- 每次新增需求、设计、实现、测试或追踪文档时，优先复用该阶段目录中的约定结构。
- 文件名默认遵守统一命名规则：`YYYY-MM-DD_HHMMSS_<artifact-code>_<work-item-id>_<short-name>.md`
- `artifact-code` 推荐使用：`sysreq`、`sysarch`、`swreq`、`swarch`、`swimpl`、`swit`、`syit`、`vtrace`
- `work-item-id` 推荐使用稳定编号，例如 `REQ-0001`、`CHG-0002`、`BUG-0010`
- `short-name` 使用 ASCII kebab-case，简短表达当前工作主题
- 如果文档内容发生阶段切换，不要覆盖原文件，而是交由下一个阶段在自己的目录中生成新的输出文件
- handoff 时至少附带：上游输入清单、当前输出清单、未解决问题、风险与假设、下一角色期望动作

具体目录和约定见 [docs/aspice/README.md](../docs/aspice/README.md)。

## 完成前检查

- 行为有变化时，运行相关测试。
- 如果没有运行测试，要明确说明原因。
- 如果改动可能影响创建、更新或软停用结果，要在总结里说明操作风险。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DevOps-zhuang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
