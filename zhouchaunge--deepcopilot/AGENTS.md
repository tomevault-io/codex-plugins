# DeepCopilot 项目级 Copilot / AI 审核指引

> 本文件会被 GitHub Copilot Code Review、Copilot Chat 以及 `.github/workflows/ai-review.yml`
> 中的 DeepSeek 审核器读取，作为审查 PR 时的项目上下文。

## 项目概述
DeepCopilot 是一个 VS Code 扩展，核心能力：
- 通过 **DeepSeek API** 提供对话式编程助手
- 在 webview 中渲染聊天 UI（`media/`、`src/webview/`）
- 通过工具调用（`src/tools/`）让模型读写文件、执行命令、抓取网页
- 提供 MCP 集成（`src/mcp.js`）和技能（skills）扩展点

## 技术栈
- Node.js + JavaScript（不是 TS）
- VS Code Extension API
- esbuild 打包（`esbuild.config.js`）
- Webview + 原生 HTML/CSS/JS（KaTeX、DOMPurify）

## 审查时必须关注的红线
1. **密钥与凭据**：禁止把 API Key、token 硬编码进任何文件；禁止在日志/异常里打印完整 key。
2. **命令执行安全**：`src/tools/exec.js`、`src/tools/shell.js` 涉及命令执行，必须确认：
   - 不接受未经检查的用户拼接命令
   - 没有 `shell: true` + 用户输入直拼
3. **文件系统**：`src/tools/file-read.js`、`file-write.js` 要确认路径在工作区内，防止 `..` 路径穿越。
4. **Webview CSP**：`src/webview/html.js` 修改时，CSP 不能放宽到 `unsafe-eval`、`unsafe-inline`（除已存在的 nonce 模式）。
5. **网络抓取**：`src/tools/web-fetch.js`、`web-search.js` 不得绕过用户授权、不得允许 `file://`、`localhost` SSRF。
6. **依赖变更**：`package.json` 增删依赖必须在 PR 描述里说明用途，拒绝引入已知废弃 / 高危包。

## 代码风格
- 使用 2 空格缩进、单引号、必要时分号
- 错误必须通过 `src/errors.js` 中的工具处理，不要 `console.log` 调试代码
- 国际化文案统一通过 `src/utils/i18n.js`
- 路径相关用 `src/utils/paths.js`，不要直接拼字符串

## 审核结论格式
请在审查最后一行给出明确结论：

> 结论：通过 / 需修改 / 拒绝

需修改 / 拒绝时必须列出具体修改项，便于贡献者跟进。

---
> Source: [ZhouChaunge/DeepCopilot](https://github.com/ZhouChaunge/DeepCopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
