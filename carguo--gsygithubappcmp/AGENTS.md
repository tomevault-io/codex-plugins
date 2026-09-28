# 项目定位（Project Identity - 最高优先级，先读这一段再动手）

> **本项目是 Compose Multiplatform，5 端共享 commonMain UI，agent 改任何 feature 默认改 commonMain。**

- rootProject 名：`GSYGithubAppCompose`
- 5 个 KMP target：`androidTarget()` / `jvm("desktop")` / `iosArm64()` / `iosSimulatorArm64()` / `macosArm64()`
- 3 个入口模块：`androidApp/`、`desktopApp/`、`iosApp/`，bundle id 统一为 `com.shuyu.gsygithubappcompose`
- 13 个 feature 模块（[feature/code](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/code)、[detail](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/detail)、[dynamic](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/dynamic)、[history](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/history)、[home](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/home)、[info](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/info)、[issue](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/issue)、[list](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/list)、[login](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/login)、[notification](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/notification)、[profile](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/profile)、[push](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/push)、[search](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/search)、[trending](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/trending)、[welcome](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature/welcome)）的 UI/ViewModel **全部位于 `commonMain`**。`androidMain` / `jvmMain` / `iosMain` / `macosMain` 只放平台胶水（`expect/actual`、Toast、SystemUi、文件路径、Native interop 等），不再承载业务 UI。
- 共享底座：[core/common](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common)、[core/ui](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/ui)、[core/network](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/network)、[core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database)、[data](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/data)、[composeShared](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/composeShared)（iOS/macOS 接入点 `MainViewController.kt`）。

## Ground Truth 版本表（与 [gradle/libs.versions.toml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/gradle/libs.versions.toml) 必须对齐）

| 组件 | 版本 |
| --- | --- |
| Compose Multiplatform | **1.10.3** |
| Kotlin | **2.3.20** |
| AGP | **9.0.0** |
| Ktor | **3.1.0** |
| Koin | **4.2.1** |
| Coil | **2.7.0** |
| jetbrains-navigation-compose | **2.9.1** |

> 改 build.gradle.kts、写新代码时一律以这张表为准。**禁止**临时升级/降级到其他版本"试试看"。

---

# 1. 默认改动落点：commonMain（不要默认 androidMain-only）

**铁律：任何 feature 的功能改动、Bug 修复、UI 调整，默认改 [commonMain](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature)，不要只改 androidMain。**

- ✅ 在 `feature/<name>/src/commonMain/kotlin/...` 改 `XxxScreen.kt` / `XxxViewModel.kt`
- ❌ 看到 androidMain 有同名文件就直接改 androidMain（90% 情况是误判，那只是一个胶水）
- ❌ 把 commonMain 已有的逻辑复制到 androidMain 再改

**只允许改 androidMain（platform-specific）的场景**：
1. Android 专属 OAuth Redirect / Deep Link / Intent / Push 通道
2. Android `actual` 实现（如 `Dispatchers.android.kt`、平台 SystemUi、Toast、Clipboard）
3. `androidMain/res/`、`AndroidManifest.xml`、`androidApp/` 入口、`proguard-rules.pro`
4. KSP 仍未支持 KMP 的模块（如 [core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database) 中的 Room）

> 若一个修改在某 target 上不可避免地依赖该平台 API（例如 `androidx.compose.foundation.*` 的 `LocalOverscrollFactory`、`android.content.Context`、`androidx.activity.*`），**必须在 PR/commit 描述里明确标注 "platform-specific (androidMain/jvmMain/iosMain/macosMain)"，并解释为什么不能下沉到 commonMain**。

## 常见 platform-specific API 速查（出现这些 import 立刻警觉）

| import / API | 仅可用于 |
| --- | --- |
| `androidx.compose.foundation.*`（Android 限定的，例如 `LocalOverscrollFactory`、`PullRefreshIndicator` 老 API） | androidMain |
| `androidx.compose.material.*`（旧 Material 1，CMP 已统一 Material 3） | 通常仅 androidMain；commonMain 用 `androidx.compose.material3.*` |
| `androidx.activity.*` / `ComponentActivity` / `BackHandler`（旧版） | androidMain；commonMain 用 `androidx.compose.ui.backhandler.BackHandler`（CMP 1.7+） |
| `android.content.Context` / `@StringRes` / `android.util.Log` / `Toast.makeText` | androidMain |
| `java.io.File` / `java.text.SimpleDateFormat` / `java.util.*` | jvmMain / androidMain；commonMain 用 `kotlinx.datetime`、`okio.Path` |
| `platform.UIKit.*` / `platform.Foundation.*` | iosMain / macosMain |
| `androidx.datastore` 直接用 Context 创建 | androidMain；commonMain 用 KMP DataStore + `expect fun preferenceDataStore(...)` |

> 若 commonMain 必须用到上述能力，标准做法是：在 commonMain 写 `expect`，在 4 个平台 sourceSet 各写一个 `actual`。**不允许**用 `Platform.isAndroid` 这种运行时分支绕过 expect/actual。

---

# 2. Source Set 矩阵：每个 source set 能用什么

> 严格按这张表写代码。任何越界（如 commonMain 出现 Android API）必须立刻通过 expect/actual 修正。

| Source Set | 可用 API | 不可用 API | 典型职责 |
| --- | --- | --- | --- |
| **commonMain** | `kotlin.*`、`kotlinx.coroutines.*`、`kotlinx.serialization.*`、`kotlinx.datetime`、Compose Multiplatform 1.10.3（`androidx.compose.runtime/foundation/material3/ui/animation`，注意：是 CMP 重新发布的版本）、`coil3`、Ktor 3.1.0 client-core、Koin 4.2.1 core+compose、`org.jetbrains.compose.resources.Res`（[core/common/src/commonMain/composeResources](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources) 下的资源）、jetbrains-navigation-compose 2.9.1 | 任何 `android.*`、`androidx.activity.*`、`java.io.File` 老 API、`platform.*`、Retrofit、Glide、Hilt | 全部 13 个 feature 的 Screen / ViewModel / UiState、Repository、Mapper、Paging、Domain Model |
| **androidMain** | commonMain 全部 + `android.*`、`androidx.activity.*`、`androidx.compose.foundation` 的 Android 专属 API、Apollo（仅 androidMain 保留 GraphQL 实现）、KSP-依赖（`core/database` 的 Room） | iOS / Foundation API | `MainActivity` / `GSYApplication` / `AppModule.kt` / actual 实现 / Android `res/` / OAuth Intent |
| **jvmMain**（Desktop） | commonMain 全部 + `java.awt.*`、`javax.swing.*`、`java.io.File`、`java.nio.*`、Compose for Desktop 入口（`androidx.compose.ui.window.*`） | Android API、iOS API | [desktopApp/src/jvmMain](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/desktopApp/src/jvmMain)：Main.kt、JvmLanguageDataStore、DesktopUserSession、StartRouteDiagnostics；Dispatchers.jvm.kt actual |
| **iosMain** | commonMain 全部 + `platform.UIKit.*`、`platform.Foundation.*`、`platform.darwin.*`、Kotlin/Native 协程 | Android API、JVM-only API | [composeShared/src/iosMain/.../MainViewController.kt](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/composeShared/src/iosMain/kotlin/com/shuyu/gsygithubappcompose/MainViewController.kt)、Dispatchers.ios.kt actual、iOS OAuth/Push/Clipboard actual |
| **macosMain** | 与 iosMain 相同（共享 Apple 平台 API），可读 macOS 文件系统目录 | Android API、JVM-only API | Dispatchers.macos.kt actual、macOS 专属差异（窗口/文件位置） |

> **特别提醒**：`androidx.compose.foundation` 在 CMP 中是被重发布的，多数 API（`Box`、`LazyColumn`、`Modifier.clickable`）在 commonMain 可用；但 **`overscroll`、`pullRefresh`（旧）、`Image` 的 `painterResource(@DrawableRes)` 等少数 API 仅在 androidMain 可用**。识别方式：在 IDE 里 import 报红 / 编译报 `Unresolved reference`，或文件路径属于 `androidx.compose.foundation:foundation-android` artifact。

---

# 3. Agent 写代码前必做的 Todo 模板

> 接到任何"修个 bug"、"加个功能"、"调下 UI"的任务，先在脑中或 chat 里走完这个清单再动手。

```
[ ] 1. 这个改动是否影响 5 端？默认是 → 改 commonMain
[ ] 2. 涉及的 Screen / ViewModel 在哪个 sourceSet？用 ripgrep 确认：
       rg -l "class XxxScreen" feature/<name>/src
       结果若同时出现在 commonMain 和 androidMain → 99% 应改 commonMain，androidMain 只是 actual
[ ] 3. 是否需要平台能力（Toast / 文件 / Intent / 时间格式 / 剪贴板）？
       是 → 在 commonMain 用现有 expect 函数；若没有，新加 expect + 4 个 actual（android/jvm/ios/macos）
       否 → 纯 commonMain 改完即可
[ ] 4. 是否需要新依赖？先看 [gradle/libs.versions.toml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/gradle/libs.versions.toml) 是否已有；
       新依赖必须支持全部 5 个 target，否则只能放 androidMain/jvmMain/iosMain/macosMain
[ ] 5. 是否要动 build.gradle.kts？默认不动，详见第 4 节"编辑约束"
[ ] 6. 文本是否多语言？是 → 加到 [core/common/src/commonMain/composeResources/values/strings.xml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources/values/strings.xml)
       与 [values-zh/strings.xml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources/values-zh/strings.xml)
[ ] 7. 验证策略：
       Android → :androidApp:assembleDebug + adb 真机回归 + [testing/uiautomator](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/uiautomator) playbook
       Desktop / macOS → :desktopApp:run + [testing/macos-desktop/scripts/e2e_all_routes.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/macos-desktop/scripts/e2e_all_routes.sh)
       iOS → 在 iosApp Xcode 项目跑 + [testing/ios/scripts/e2e_all_routes.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/ios/scripts/e2e_all_routes.sh)
[ ] 8. 提交前自检：commonMain 没有任何 android.* / platform.* / java.awt.* import
```

---

# 4. 编辑约束（Editing Constraints）

## 4.1 注释禁令
- **不允许随意删除原有注释和"看似无用"的代码**，可能是其他平台 actual 的占位或回归证据。
- **不主动加无意义注释**（"这是一个函数"、"获取数据"等噪音）。仅在为复杂逻辑、平台差异、坑位说明时才写注释。
- 不允许把中文注释翻译成英文（或反之），保留原作者风格。

## 4.2 Build script 约束
- **不要乱改 [build.gradle.kts](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/build.gradle.kts) / [settings.gradle.kts](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/settings.gradle.kts) / 子模块的 build.gradle.kts**。
- 改动场景必须是显式需求：加 target、新增模块、引入新依赖。其他改动（重排 plugins 顺序、调整 sourceSet 写法、"顺手优化"）一律禁止。
- 改 `libs.versions.toml` 必须保持 ground truth 表的核心版本；只能在 patch 级别（如 1.10.3 → 1.10.4）做必要修复，禁止跨 minor。
- **不要在 KMP 模块里加 `debugImplementation` 或不带 `project.dependencies.platform(...)` 的 `platform()`**——这是 KMP `sourceSets` DSL 不支持的语法，会编译失败。

## 4.3 命名 / 文件组织（沿用既有规则）
- 包路径：`com.shuyu.gsygithubappcompose.<layer>.<module>`，落到目录是 `com/shuyu/gsygithubappcompose/...`。**绝不允许出现 `com.shuyu.gsygithubappcompose` 这样带点的目录**。正确：`src/commonMain/kotlin/com/shuyu/gsygithubappcompose/data/repository/EventRepository.kt`。
- 共享代码统一放 `src/commonMain/kotlin/...`。Phase 2 时代留下的 `src/main/java` 已不存在，遇到任何文档/脚本仍写 `src/main/java` 视为过期，按 commonMain 落盘。
- 一个常规 feature 模块的目录：
  ```
  feature/<name>/
    build.gradle.kts
    src/commonMain/kotlin/.../<Name>Screen.kt
    src/commonMain/kotlin/.../<Name>ViewModel.kt
    src/commonMain/kotlin/.../di/<Name>Module.kt
    src/androidMain/kotlin/.../  ← 仅当存在 actual / Android-only Composable 时才需要
  ```

## 4.4 token / local.properties / 机密
- GitHub OAuth `client_id` / `client_secret` 一律写在 [local.properties](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/local.properties)（已 gitignore）；仓库内只有 [local.properties.sample](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/local.properties.sample) 作模板。**不允许把 token 写进 source 或 build.gradle.kts。**
- 用户 token、登录态读写走 [core/common](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common) 的 `DataStore<Preferences>`。Desktop 端有专属的 [JvmLanguageDataStore.kt](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/desktopApp/src/jvmMain/kotlin/com/shuyu/gsygithubappcompose/desktop/language/JvmLanguageDataStore.kt) / [DesktopUserSession.kt](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/desktopApp/src/jvmMain/kotlin/com/shuyu/gsygithubappcompose/desktop/session/DesktopUserSession.kt) 作 actual。
- 多语言文本：[core/common/src/commonMain/composeResources/values/strings.xml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources/values/strings.xml) + [values-zh/strings.xml](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources/values-zh/strings.xml)。**不允许写死 UI 字符串**。访问通过 CMP `Res.string.xxx` / `stringResource(Res.string.xxx)`。
- Android 仅用的 `mipmap` / launcher icon 仍放 [core/common/src/androidMain/res/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/androidMain/res)；跨平台的 Lottie / 图片 / 启动图放 [core/common/src/commonMain/composeResources](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common/src/commonMain/composeResources)。

## 4.5 模块职责（沿用既有约定，已对齐 KMP）
- [core/common](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/common)：DataStore、token、跨平台资源（compose-resources）、多语言。
- [core/ui](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/ui)：所有自定义控件、主题、颜色，包括 `GSYGeneralLoadState`、`GSYPullRefresh`、`GSYTopAppBar`、`GSYNavigator`、`GSYNavHost`、`GSYLoadingDialog`。下拉刷新和加载更多**统一**用 `GSYPullRefresh`，不要再引第三方。
- [core/network](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/network)：网络入口，Ktor 3.1.0 + GitHubApiService、`config/PAGE_SIZE`、`model/` 网络实体、Apollo schema 在 `commonMain/graphql/`。
- [core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database)：所有 Dao / Entity / AppDatabase，**改表必升 AppDatabase 版本号**。Room KSP 当前若仍仅作用于 androidMain，新表的 expect/actual 写法以现有模块为准。
- [data](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/data)：Repository（commonMain）+ `Dispatchers.android/jvm/ios/macos.kt` actual；所有 `toEntity` / `toModel` 集中在 `mapper/DataMappers.kt`。data 模块**不放实体 Model**，Model 在 [core/network/.../model/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/network)。
- [feature/*](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/feature)：每个模块只放 `XxxScreen` + `XxxViewModel` + `di/<Name>Module.kt`，全部在 commonMain。
- ViewModel：继承 `BaseViewModel`，UiState 继承 `BaseUiState`，Compose 中获取用 `viewModel: XxxViewModel = koinViewModel()`（`org.koin.compose.viewmodel.koinViewModel`，跨平台版本，不再用 `org.koin.androidx.compose.koinViewModel`）。
- 配 ViewModel/Repository **仅用 Koin 4.2.1 DSL**：`viewModel { ... }` / `single { ... }`。**全面禁止** Hilt / Dagger 注解（`@HiltViewModel`、`@AndroidEntryPoint`、`@Inject`、`@Module`、`@Provides`、`@Binds`、`hiltViewModel()` 等）。
- Kotlin 子类构造顺序：父类的 init 先于子类的属性初始化执行；**禁止**在传给父类的构造参数里调用依赖子类属性的方法。

## 4.6 导航
- 导入 `import com.shuyu.gsygithubappcompose.core.ui.LocalNavigator`（commonMain 已重写为 jetbrains-navigation-compose 2.9.1 的桥接）。
- `val navigator = LocalNavigator.current`，`navigator.navigate(...)` / `navigator.replace(...)`。
- 所有平台共享同一 NavHost；如某路由仅某端可用（如 OAuth 回调路由仅 Android），由对应入口模块在自己的 sourceSet 注册。

---

# 5. 验证 / 测试沉淀（5 端共用一套资产）

> 痛点：每次回归靠当场 `uiautomator dump` 找坐标 + 当场试 swipe 时长，知识不沉淀。从现在起 5 端各自维护资产、互不混淆。

| 端 | 资产目录 | 关键脚本 |
| --- | --- | --- |
| Android | [testing/uiautomator/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/uiautomator) | [device.md](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/uiautomator/device.md) / [screen_map.md](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/uiautomator/screen_map.md) / [playbook.md](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/uiautomator/playbook.md) |
| iOS | [testing/ios/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/ios) | [scripts/e2e_all_routes.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/ios/scripts/e2e_all_routes.sh)、[scripts/p00_cold_boot.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/ios/scripts/p00_cold_boot.sh) |
| macOS Desktop | [testing/macos-desktop/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/macos-desktop) | [scripts/e2e_all_routes.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/macos-desktop/scripts/e2e_all_routes.sh)、[scripts/p00_cold_boot.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/macos-desktop/scripts/p00_cold_boot.sh)、[scripts/p01_login_oauth.sh](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/macos-desktop/scripts/p01_login_oauth.sh) |
| 跨端差异 | [testing/PLATFORM_DIFF.md](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/testing/PLATFORM_DIFF.md) | 跨端表现差异（启动图位置、字号、键盘行为）必须在此登记 |

**强制规则**（适用于 3 个 testing 子目录）：
1. 新坐标 / 新 element id 当场入档到对应 `screen_map.md`，标注 `Verified at: yyyy-MM-dd`。
2. 新交互坑（如 Compose clickable 在 Android 必须用 swipe-80ms、macOS 上 Dialog 焦点行为）当场写进对应 `device.md`。
3. 新值得复用的回归路径写成 `playbook.md` 里的 P-XX 段。
4. **commonMain UI 改动必须在 ≥2 端跑过**（默认 Android + macOS Desktop，最低门槛）；commit message 贴出 ✅/❌ 矩阵。改 iOS-only / Android-only 的 actual 才允许只跑单端。
5. UI 重构后 `screen_map.md` 旧坐标先标 `(stale)` 或删除，禁止保留过期数据。
6. 跑回归优先调用对应 `e2e_all_routes.sh`，截图自动落到 [tools/screenshots/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/tools/screenshots) 与日志 [tools/logs/](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/tools/logs)。

**自检清单（每次提交前必走）**：
- [ ] commonMain 改动在 ≥2 端构建通过（默认 `:androidApp:assembleDebug` + `:desktopApp:run`）
- [ ] commonMain 没有 `android.*` / `platform.*` / `java.awt.*` import
- [ ] 新 `expect` 在 4 个 sourceSet 都有 `actual`，没有抛 `error("not provided")` 占位
- [ ] 文案走 compose-resources，未硬编码
- [ ] 未在源码 / build.gradle.kts 留 token
- [ ] 测试资产已更新（坐标 / playbook / device note）

---

# 6. 数据库变更注意（沿用 Phase 2 既定流程）
1. 升 [core/database](file:///Users/guoshuyu/workspace/android/GSYGithubAppCMP/core/database) 中 `AppDatabase` 版本号
2. 新加 `Entity`
3. 新加对应 `Dao`
4. 在 `DatabaseModule` 注册新的 Dao 提供方法

---

# 7. 关键禁忌（Don't）

- ❌ 默认改 androidMain（除明确为 platform-specific）
- ❌ commonMain 直接 `R.string.xxx` / `Toast.makeText` / `java.io.File`
- ❌ 在 KMP 模块写 `debugImplementation` 或裸 `platform(...)`
- ❌ 在源码里硬编码 GitHub `client_id` / `client_secret`
- ❌ 引入未支持 5 端的依赖却放 commonMain
- ❌ 改 build.gradle.kts 跨 minor 升级 CMP / Kotlin / AGP / Ktor / Koin
- ❌ 出现带点的目录 `com.shuyu.gsygithubappcompose/`
- ❌ 删别人的注释 / "顺手"重命名 / 顺手 reformat 整个文件
- ❌ commit message 不附 ≥2 端回归 ✅/❌ 矩阵

---
> Source: [CarGuo/GSYGithubAppCMP](https://github.com/CarGuo/GSYGithubAppCMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-28 -->
