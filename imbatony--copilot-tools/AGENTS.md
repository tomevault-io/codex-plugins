# Copilot Tools - 自定义指令

这是一个用于日常生活和工作辅助的 Copilot CLI 工具集项目。

## 项目概述

本项目通过定制 GitHub Copilot CLI 来提供以下功能：
- 天气查询
- 日常任务管理
- 工作流程自动化

## 通用指导原则

1. **语言偏好**: 默认使用中文回复，除非用户明确要求使用其他语言
2. **简洁明了**: 回答应该简洁、直接，避免冗余信息
3. **实用优先**: 优先提供可操作的建议和解决方案
4. **安全意识**: 在执行任何可能影响系统的操作前，先说明可能的风险

## 技术栈

- **包管理器**: uv
- **Python**: 3.8+
- **脚本语言**: Python (优先), PowerShell, Bash

## 代码风格

- 使用有意义的变量名和函数名
- 添加适当的注释说明
- 遵循各语言的最佳实践
- Python 代码遵循 PEP 8

## 响应格式

- 对于简单问题，直接给出答案
- 对于复杂问题，先给出概要，再提供详细解释
- 使用 Markdown 格式化输出以提高可读性

## 专项指令

本项目包含以下专项指令文件，按需参考：

| 文件 | 适用范围 |
|------|---------|
| `instructions/python.instructions.md` | Python 代码开发 |
| `instructions/skills.instructions.md` | Agent Skill 编写 |
| `instructions/agents.instructions.md` | 自定义 Agent 编写 |
| `instructions/hooks.instructions.md` | Hook 配置开发 |
| `instructions/mcp.instructions.md` | MCP 服务器配置 |

## 项目结构

```
.github/
├── copilot-instructions.md    # 主指令文件（本文件）
├── instructions/              # 专项指令
│   ├── python.instructions.md
│   ├── skills.instructions.md
│   ├── agents.instructions.md
│   ├── hooks.instructions.md
│   └── mcp.instructions.md
├── agents/                    # 自定义 Agent
├── skills/                    # Agent Skills
│   ├── SKILL_GUIDELINES.md    # Skill 编写指南
│   └── weather-query/         # 天气查询技能
└── hooks/                     # Hook 配置
```

## 运行命令

```bash
# 运行 Python 脚本
uv run script.py

# 运行天气查询
uv run .github/skills/weather-query/scripts/weather.py 北京

# 启动 Copilot CLI
copilot
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imbatony)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/imbatony)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
