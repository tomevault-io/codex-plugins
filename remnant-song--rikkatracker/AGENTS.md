## Git 提交消息生成规范

## 1. 强制格式规范（Commit Message 结构）
必须严格遵循 **Conventional Commits 1.0.0** 格式：
```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```
- 第一行（标题行）长度 ≤ 72 字符。
- 正文每行 ≤ 72 字符。
- 类型（type）、范围（scope）、主题（subject）均为小写（subject 首字母允许小写）。

## 2. 类型（type）强制枚举
AI 只能从以下列表中选择 **且仅一个** 类型：
- `feat` – 新功能（与语义化版本 MINOR 对应）
- `fix` – 缺陷修复（与语义化版本 PATCH 对应）
- `docs` – 仅文档变更
- `style` – 代码风格（不影响代码含义，如空格、缩进、分号等）
- `refactor` – 重构（既非新功能也非修复）
- `perf` – 性能优化
- `test` – 增加或修正测试
- `build` – 构建系统或外部依赖变更（如 gradle, npm, makefile）
- `ci` – CI 配置或脚本变更（如 GitHub Actions, Jenkins）
- `chore` – 其他不修改 src 或 test 的变更（如工具配置、辅助脚本）
- `revert` – 回退提交（必须附带 `Revert: <被回退的提交哈希>` 在标题或脚注）

**禁止使用**：`update`、`change`、`modify` 等模糊类型。

## 3. 范围（scope）约束
- 范围 **可选**，但若提供，必须为单个名词短语（如 `api`, `parser`, `ui`, `config`, `auth`）。
- 范围不得包含空格或特殊符号（仅允许小写字母、数字、连字符）。
- 若变更涉及多个模块，可省略范围或使用 `*`（但推荐省略）。

## 4. 主题（subject）约束
- 必须以动词开头（一般现在时），如 `add`, `fix`, `update`, `remove`。
- 不得以句号（`.`）结尾。
- 必须使用英文（除非项目明确定义使用中文，但此规则强制英文）。
- 应清晰描述“做了什么”，而非“为什么做”。

## 5. 正文（body）约束（若存在）
- 必须使用完整句子，解释 **变更的背景、动机** 以及 **与之前行为的差异**。
- 每行不超过 72 字符。
- 不得包含无意义填充词（如 `just`, `simply`）。
- 若变更涉及性能、兼容性、破坏性变更，必须在此明确说明。

## 6. 脚注（footer）约束（若存在）
- 用于引用 Issue、PR、JIRA 等，格式：
  - `Closes #123`
  - `Refs #456, #789`
  - `BREAKING CHANGE: <描述>`（大写固定）
- 破坏性变更（BREAKING CHANGE）必须放在脚注，且后跟冒号和空格，描述不少于 10 个字符。

## 7. 自动生成时的强制检查点（AI 内部校验）
AI 在输出最终提交信息前，必须逐项验证以下条件，任一不满足则拒绝生成，并提示具体违反项：

- [ ] 标题行长度 ≤ 72。
- [ ] 类型在枚举列表中。
- [ ] 范围（若有）符合命名规则。
- [ ] 主题首字符为小写字母（除非是数字开头）。
- [ ] 主题不以句号结尾。
- [ ] 正文（若有）每行 ≤ 72，且非空行分隔正确。
- [ ] 脚注（若有）格式匹配标准（`Closes|Refs|BREAKING CHANGE`）。
- [ ] 整个信息不包含 Emoji 或非 ASCII 装饰符号（除非项目明确允许）。
- [ ] 不得包含占位符（如 `TODO`, `XXX`, `placeholder`）作为实际内容。

## 8. 长度与截断规则
- 若变更文件列表过大（> 20 个），AI 必须简要归纳变更范围（如 `multiple files`），而非穷举。
- 若主题超出长度限制，AI 应自动压缩（如删除冗余动词、缩写公认术语）。

## 9. 多提交场景策略（交互式或批量）
- 若 AI 被要求生成多个提交信息，每个信息必须 **独立满足上述所有规则**，且每个提交对应一个逻辑单元（single responsibility）。
- 不得生成“混合型”提交（如 `feat` + `fix` 同时发生），应拆分为两个提交。

## 10. 语言与术语一致性
- 提交信息全文（标题、正文、脚注）使用英文。
- 技术术语保持与代码库一致（如 `handler`, `middleware`, `dto`）。

## 11. 禁止行为
- **禁止** 生成情绪化或主观性描述（如 `improve a little`）。
- **禁止** 复制粘贴命令行输出或错误日志作为提交信息。
- **禁止** 使用 `-m` 多段式简写拼接正文（必须保留换行结构）。

---
> Source: [remnant-song/RikkaTracker](https://github.com/remnant-song/RikkaTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-22 -->
