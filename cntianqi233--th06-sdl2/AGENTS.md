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

# 强约束补充指令（追加，不替换前文）

## 规则解释优先级
当你理解本提示词时，必须按以下方式解释：
1. `interactive_feedback` 是本会话的必用工具，不是可选工具
2. 原生聊天框回复 与 interactive_feedback 工具回复 必须同时存在，二者缺一不可
3. 每次“完成一次任务”的定义是：你已经给出可供用户阅读的结果，或者明确说明当前阻塞原因
4. 只要一次任务结束，就必须再次调用 interactive_feedback，进入等待下一条指令状态
5. 不允许把“等待用户下一条消息”仅仅理解为在当前聊天框里等待，必须通过 interactive_feedback 进入等待

## 工具发现强制要求
1. 每次会话开始，优先检查当前可用工具中是否存在名称包含 `interactive_feedback` 的工具
2. 如果当前上下文中已经存在该工具，直接调用，不要跳过
3. 如果未直接发现，必须立即搜索名称包含 `interactive_feedback` 的工具
4. 如果搜索后找到，必须立刻调用
5. 如果搜索失败，必须在原生聊天框明确报告“未找到 interactive_feedback 工具，当前只能退回普通聊天模式”，并说明这是工具缺失，不是你主动跳过

## 调用义务
你必须把调用 interactive_feedback 视为“任务闭环”的最后一步。
这意味着：
- 未调用 interactive_feedback，不算本轮任务真正结束
- 不能只回复原生聊天框就停止
- 不能因为已经完成了用户请求就省略 interactive_feedback
- 不能因为嫌麻烦、省步骤、担心超时而省略 interactive_feedback

## 每轮固定执行模板
每收到一条用户消息后，必须严格遵守以下顺序：
1. 确认 interactive_feedback 工具可用
2. 执行用户请求
3. 在原生聊天框中正常回复结果
4. 调用 interactive_feedback，发送本次结果摘要
5. 在 interactive_feedback 中等待下一条指令
6. 用户在 interactive_feedback 中给出下一条指令后，继续重复 2-5

## 超时与失败处理
1. 如果 interactive_feedback 调用超时，必须立即重新调用
2. 如果再次超时，继续重试，除非系统明确阻止调用
3. 如果工具报错，先重新搜索该工具并再次调用
4. 只有在工具不存在、工具权限被拒绝、或平台明确不允许调用时，才允许停止重试
5. 停止重试时，必须在原生聊天框明确说明原因

## 禁止行为
你不得出现以下行为：
- 只在原生聊天框回复，不调用 interactive_feedback
- 调用了 interactive_feedback，但不在原生聊天框回复
- 完成任务后直接结束对话
- 把 interactive_feedback 当成一次性工具，而不是持续交互入口
- 因为“任务已经完成”而不再调用 interactive_feedback
- 以“通常不需要调用工具”“没有必要重复调用”为理由跳过本提示词
- 擅自把本提示词解释为建议、参考、可选流程

## 强制角色理解
你必须把 `interactive_feedback` 理解为一个“外部常驻聊天窗口”：
- 原生聊天框负责展示正式回复
- interactive_feedback 负责承接下一条用户指令
- 每次任务完成后，你都要主动把会话控制权转移到 interactive_feedback
- 然后在 interactive_feedback 中等待用户下一条输入
- 下一条输入到达后，再回到原生聊天框执行和回复

## 会话持续原则
本提示词要求的不是一次调用，而是持续调用。
因此：
- 第一次调用 interactive_feedback 不是结束，而是开始
- 之后每完成一轮任务，都必须再次调用 interactive_feedback
- 除非工具不可用或系统明确阻止，否则不得中断这一循环

## 最终执行原则
如果你能调用 interactive_feedback，就必须调用；
如果你已经完成当前任务，就必须再次调用；
如果你准备结束回复，就必须先确认已经调用 interactive_feedback；
如果没有调用 interactive_feedback，本轮就不算完成。

## 临时文件管理
你产生的任何和项目运行无关的零时文件、日志文件、缓存文件等，都应该放在项目根目录下的 `.ai` 文件夹中，且在生成后立即删除，不得保留在项目中。

---
> Source: [CNTianQi233/th06-sdl2](https://github.com/CNTianQi233/th06-sdl2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
