# AI 交互强制指令

## 🔴 强制规则（最高优先级）

**每次会话开始时，或当用户发送任何消息时，必须立即执行以下操作：**

1. 先搜索名为 interactive_feedback 的 MCP 工具（使用 tool_search 或直接调用）
2. 调用 `mcp_mcp-ai-foreve_interactive_feedback` 工具，参数如下：
   - `project_directory`: 当前工作目录
   - `summary`: 简要说明当前状态和等待用户指令
   - `timeout`: 10000（默认10000秒，约2.8小时）
3. 收到用户反馈后，执行用户请求的任务
4. 任务完成后，**再次调用** `mcp_mcp-ai-foreve_interactive_feedback` 工具汇报结果并等待下一条指令，并且同时在原来的聊天框中回复用户，告知已完成任务并等待新指令
5. 重复步骤3-4，形成无限循环，永不停止

**关键要求：**
- 永远不要在没有调用 interactive_feedback 的情况下结束对话
- 每次任务完成后都必须重新启动 interactive_feedback 等待新指令
- 如果 interactive_feedback 超时，立即重新调用它
- summary 参数中应包含：已完成的操作摘要 + "请在交互窗口输入下一条指令"
- 如果找不到这个工具，先使用 tool_search_tool_regex 搜索 "interactive_feedback" 加载它

## 启动指令

当用户输入以下任何内容时，立即启动上述无限交互循环：
- "启动交互" / "开始交互" / "启动AI" / "开始"
- "和AI交互" / "AI交互" / "交互"
- "启动无限工具" / "无限交互"
- "打开交互" / "开始工作"
- 或任何包含"交互"、"启动"关键字的指令
- 或任何消息（收到任何消息都应该触发交互循环）

## 重要提醒

你必须使用 MCP 工具 mcp_mcp-ai-foreve_interactive_feedback 与用户交互。
不要只在 VS Code 聊天中回复，必须通过 interactive_feedback 工具发送回复。
这是一个无限循环：接收指令 → 执行 → 通过 interactive_feedback 汇报 → 等待新指令。

---
> Source: [CNTianQi233/blog.github.io](https://github.com/CNTianQi233/blog.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
