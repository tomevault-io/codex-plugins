# Copilot Instructions — JX3 茗伊插件集 (MY Plugin Collection)

## 项目概述

这是剑网3（JX3）游戏的 Lua 插件集合，包含多个独立插件模块。插件需要同时兼容**重制版（remake/HD）**和**缘起（classic/origin）**两个游戏版本。

## 文件编码

本项目存在两种编码，按文件类型区分：

**GBK/GB2312 编码（简体中文相关）：**
- 所有 `.lua` 源文件
- `zhcn.jx3dat` 简体中文语言/数据文件
- `package.ini`、`info.ini` 等配置文件

**UTF-8 编码（繁体中文相关）：**
- `zhtw.jx3dat` 繁体中文语言/数据文件
- `package.ini.zh_TW`、`info.ini.zh_TW` 等繁体配置文件

修改文件时必须使用二进制模式读写，根据文件类型选择正确的编码：
- 推荐使用 Python 的 `open(path, 'rb')` / `open(path, 'wb')` 配合对应编码的 `.encode()` / `.decode()`。
- 不要使用默认文本编辑工具直接写入中文，可能导致编码损坏。

## 项目结构

```
MY_!Base/          — 基础库，所有插件依赖此模块，提供公共 API（通过全局变量 MY 即 X 访问）
MY_Cataclysm/      — 团队面板
MY_TeamTools/       — 团队工具
MY_LifeBar/         — 血条
MY_Chat/            — 聊天增强
...                 — 其他插件模块
!src-dist/          — 构建、发布、语言转换等工具脚本
tools/              — 开发工具
```

## 基础库 (MY_!Base)

`MY_!Base` 是核心基础库，所有插件通过 `local X = MY` 引用。基础库提供：

- **类型检查**：`X.IsFunction()`, `X.IsTable()`, `X.IsNil()`, `X.IsEmpty()` 等
- **游戏 API 封装**：`X.GetKungfuName()`, `X.GetTeamMemberInfo()` 等
- **版本兼容**：封装不同游戏版本的 API 差异
- **环境信息**：`X.ENVIRONMENT` 表和 `X.IS_REMAKE`, `X.IS_CLASSIC` 标志
- **事件系统**：`X.RegisterEvent()` 等
- **常量定义**：`X.CONSTANT` 表

关键源文件路径：
- `MY_!Base/src/lib/Base.lua` — 版本检测与环境初始化
- `MY_!Base/src/lib/BaseLua.lua` — 基础 Lua 工具函数
- `MY_!Base/src/lib/Game.Skill.lua` — 技能/心法相关兼容封装
- `MY_!Base/src/lib/Game.Team.lua` — 团队相关封装
- `MY_!Base/src/lib/Environment.lua` — 功能限制/特性开关系统
- `MY_!Base/src/lib/Constant.lua` — 常量定义

## 游戏版本兼容

游戏有两个主要分支，部分全局 API 仅在其中一个版本存在：

| 环境变量 | 重制版 (remake/HD) | 缘起 (classic/origin) |
|---|---|---|
| `GAME_BRANCH` | `'remake'` | `'intl'` |
| `GAME_API_BRANCH` | `'remake'` | `'classic'` |
| `GAME_LANG` | `'zhcn'` | `'zhtw'` |
| `CODE_PAGE` | GBK (936) | UTF-8 (65001) |
| `X.IS_REMAKE` | `true` | `false` |
| `X.IS_CLASSIC` | `false` | `true` |

### 兼容函数编写规范

当遇到某个版本不存在的全局 API 时，**不要在调用处直接做 if 判断**，而是在基础库中添加兼容封装函数，然后在调用处使用封装后的版本。

**标准模式**（在 `MY_!Base/src/lib/` 相关文件中添加）：

```lua
-- 方式一：简单存在性检查
function X.SomeFunction(arg)
    if SomeGlobalAPI then
        return SomeGlobalAPI(arg)
    end
    return fallbackValue
end

-- 方式二：X.IsFunction 检查（用于更复杂的场景）
function X.SomeFunction(arg)
    if X.IsFunction(SomeGlobalAPI) then
        return SomeGlobalAPI(arg)
    elseif X.IsFunction(OlderGlobalAPI) then
        return OlderGlobalAPI(arg)
    end
    return fallbackValue
end

-- 方式三：pcall 检测（用于方法存在性不确定的场景）
do local bNewAPI
function X.SomeFunction(KObject)
    if X.IsNil(bNewAPI) then
        bNewAPI = pcall(function()
            if not KObject.NewMethod then assert(false) end
        end)
    end
    if bNewAPI then
        return KObject.NewMethod()
    else
        return KObject.OldMethod()
    end
end
end
```

## 语言文件 (i18n)

每个插件的 `lang/` 目录下有 `zhcn.jx3dat`（简体中文）和 `zhtw.jx3dat`（繁体中文）语言文件。

### 重要规则

- `zhcn.jx3dat`（GBK 编码）和 `zhtw.jx3dat`（UTF-8 编码）均需手动维护。
- 带有 `zhtw` 字样的数据文件是 **UTF-8** 编码，不是 GBK。
- 修改 `zhcn` 简体语言文件后，需运行 `!src-dist/clang.py` 来更新对应的繁体语言文件。
- **不要直接修改 `zhtw.jx3dat` 繁体语言文件**，由 `clang.py` 从简体转换生成。
- 同理，`package.ini.zh_TW` 和 `info.ini.zh_TW` 也由工具生成，不要手动修改。

## 编码规范

### 代码风格

- `if ... then` 必须换行，**禁止单行** `if ... then ... end`。
- `return` 后**禁止接 `else`**，直接在后续写逻辑即可。

```lua
-- ✅ 正确
if not player then
    return
end
DoSomething()

-- ❌ 错误：if then 单行
if not player then return end

-- ❌ 错误：return 后接 else
if not player then
    return
else
    DoSomething()
end
```

### 变量命名 — 匈牙利命名法

变量名使用**类型前缀 + 帕斯卡命名**（Hungarian Notation），前缀表示类型：

| 前缀 | 类型 | 示例 |
|------|------|------|
| `dw` | DWORD (无符号整数) | `dwPlayerID`, `dwKungfuID`, `dwForceID` |
| `n` | number (整数) | `nTime`, `nCount`, `nLevel` |
| `f` | float (浮点数) | `fScale`, `fPercent` |
| `sz` | string (字符串) | `szName`, `szGlobalID`, `szFilePath` |
| `b` | boolean | `bEnable`, `bVisible`, `bNewAPI` |
| `t` | table | `tKungfu`, `tScore`, `tLine` |
| `a` | array (数组型 table) | `aTeam`, `aSkillID`, `aKungfuList` |
| `fn` | function | `fnAction`, `fnCallback` |
| `h` | handle (KGUI 界面元素) | `hWndEdit`, `hWndButton`, `hTextPlayerName` |
| `k` | kernel object (SO3Client 游戏对象) | `kPlayer`, `kTarget`, `kSkill` |

**`h` (handle)** — KGUI 导出的界面 Lua userdata 封装，命名格式为 `h` + 组件类型 + 描述：
- `hWndEdit` — 编辑框窗口
- `hWndButton` — 按钮窗口
- `hTextPlayerName` — 玩家名文本元素
- `hItemDataPlayer` — 玩家数据项

**`k` (kernel object)** — SO3Client（游戏引擎）导出的 C++ 对象，代表游戏世界中的实体：
- `kPlayer` — 玩家对象，可访问 `kPlayer.szName`, `kPlayer.dwID` 等属性
- `kTarget` — 目标对象
- `kSkill` — 技能对象

## 提交规范

- Commit message 使用中文，格式为 `type(模块名): 描述`，模块名用中文。
- `type` 必须是以下之一：`feat`、`fix`、`docs`、`style`、`refactor`、`perf`、`test`、`chore`、`build`、`release`。
- 示例：
  ```
  feat(基础库): 增加 X.GetHDKungfuID 兼容函数
  feat(聊天助手): 新增基于玩家名称过滤的逻辑
  fix(团队工具): 修复缘起版本调用 GetHDKungfuID 报错
  refactor(基础库): 重构技能兼容函数
  ```
- 兼容函数和实际调用处的修改应分开提交：
  1. 第一个提交：在基础库中添加兼容函数
  2. 第二个提交：在实际使用处调用兼容函数

---
> Source: [tinymins/JX3MY](https://github.com/tinymins/JX3MY) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
