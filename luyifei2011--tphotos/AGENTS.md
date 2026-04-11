## 速览（高层架构）

- Flutter 前端单仓库应用（`lib/`）。核心分层：
  - `lib/api/`：HTTP 客户端适配层，封装请求、cookie 与 CSRF 处理（关键文件：`tos_client.dart`, `tos_api.dart`）。
  - `lib/api/*_api.dart`：按功能分的 API（`auth_api.dart`, `photos_api.dart`），通过 `TosClient` 发起网络请求并解析返回。
  - `lib/models/`：从 API JSON 到 Dart 对象的映射（`timeline_models.dart`, `photo_list_models.dart`）。注意字段命名与 API 返回的拼写不一致处。
  - `lib/pages/`：UI 页面（`login_page.dart`, `photos_page.dart` 等），大量使用 `FutureBuilder` 与缓存策略。`

## 关键工作流 / 运行命令

- 安装依赖：
```bash
flutter pub get
```
- 运行应用（在开发机器上或连接设备）：
```bash
flutter run
```
- 在本地开发时需要配置后端地址：默认在 `lib/main.dart` 和登录页中使用 `SharedPreferences` 的 `server` 值，登录页 UI 的默认值在 `lib/pages/login_page.dart`。可直接修改输入框或在代码中临时改写 `TosAPI` 构造参数以指向你的测试服务器。

## 项目特有约定与实现细节（不要忽略）

- TosClient（`lib/api/tos_client.dart`）负责：
  - 统一构造请求头（包括 Content-Type、User-Agent、以及当前 cookie 字段）。
  - 自动管理 `Set-Cookie` 并把 cookie 写回后续请求（内部存放于 `_cookies`）。
  - 从 cookie 中提取 CSRF token，自动添加 `X-CSRF-Token` 头。任何新增的 API 调用若需要登录态或二进制数据，优先使用 `TosClient.getBytes()` 或 `TosClient.get()` 来保留 cookie/CSRF 行为。

- 登录 RSA 流程（参考 `lib/api/auth_api.dart`）：
  1. 先调用 `GET /tos/` 以获取 cookie/CSRF。  
  2. 调用 `GET /v2/lang/tos`（`AuthAPI` 使用 `includeHeaders: true`），从 response headers 读取 `x-rsa-token`（这是 PEM 文本的 base64），解析为 RSA 公钥并用 `encrypt` 包加密密码后发 `/v2/login`。  
  3. 若新增登录相关逻辑，务必遵循上面步骤以保证 CSRF/cookie 与 RSA 流程完整。

- 二进制资源（缩略图/原图）必须通过 `PhotosAPI.thumbnailBytes()` / `PhotosAPI.originalPhotoBytes()`（底层都用 `TosClient.getBytes()`），以避免 403（服务端依赖 cookie 认证）。

- API 返回与模型不一致的典型坑：
  - `PhotoItem.fromJson` 使用 `json['timetamp']`（注意拼写），客户端显式接受该字段；不要自动改成 `timestamp`，除非同时更新所有解析处。  
  - `TimelineResponse` / `PhotoListResponse` 的 `code` 字段是布尔类型（代表成功），`data` 结构按模型定义。修改解析逻辑时请参考 `lib/models/*.dart`。

## 常见代码模式（示例指导）

- 新增后端调用：优先在 `lib/api/` 下添加方法并复用 `TosClient`：
  - JSON 请求：使用 `TosClient.post(path, json: {...})` 或 `get(path, params: ...)`。
  - 二进制：使用 `TosClient.getBytes(path)`。
  - 错误处理：`TosClient` 在非 2xx 会抛出 `APIError`，上层 API 可捕获并包装或直接向 UI 抛出。

- 页面层（`lib/pages/*`）常用模式：
  - 使用 `FutureBuilder` 处理异步数据加载；异常状态在 UI 层捕获并显示错误信息。
  - 缓存策略示例：`PhotosPage` 用 `_datePhotoCache`、`_thumbCache` 存放已加载的图片列表与缩略图，避免重复请求。

## 编辑/扩展注意事项（对 agent 的具体建议）

- 修改或新增网络路径时：
  - 在 `lib/api/*_api.dart` 中添加对应方法并写单元级的解析（或在 `models` 中添加 fromJson/toJson）。
  - 确保对需要登录的接口使用 `TosClient`，不要直接用 `http.Client()`，以免绕过 cookie/CSRF 管理。 

- 当改动模型字段（例如修正 `timetamp` 拼写）时：
  - 全库搜索该字段（`grep` / IDE 全局搜索），同步修改 `fromJson` / `toJson` 与所有使用点；否则会导致运行时 KeyError/类型错误。示例文件：`lib/models/photo_list_models.dart`、`lib/pages/photos_page.dart`。

- 资源 URL 与 baseUrl：`TosAPI.baseUrl` 由 `TosClient.baseUrl` 提供，用于构建代理路径（PhotosAPI 里常见）。若变更后端地址风格，请在 `PhotosAPI` 与 `AuthAPI` 一并调整。

## 典型调试提示

- 登录失败：检查是否已先 `GET /tos/`（cookie/CSRF），并确保 `x-rsa-token` 可在 `GET /v2/lang/tos` 的 headers 中读取。
- 403 获取图片：确认使用 `getBytes()`（保留 cookie）而非普通 `get()` 返回的字符串。检查 `TosClient._cookies` 的内容与 `Set-Cookie` 解析。

## 参考文件

- `lib/api/tos_client.dart` — 请求构造、cookie 与 CSRF 管理
- `lib/api/auth_api.dart` — RSA 登录实现示例
- `lib/api/photos_api.dart` — 缩略图/原图与分页加载（photoListAll）
- `lib/models/*.dart` — API <-> Model 映射（注意拼写陷阱）
- `lib/pages/login_page.dart`、`lib/pages/photos_page.dart` — UI 层使用模式与缓存示例
- `pubspec.yaml` — 依赖（`http`, `encrypt`, `pointycastle`, `shared_preferences`）

---

修改完成代码后，需要检查错误。
该项目不支持 Web 端。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuYifei2011)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/LuYifei2011)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
