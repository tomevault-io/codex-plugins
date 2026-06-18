
# 技能描述

此技能用于高并发异步扫描本地 Codex 认证文件的存活状态，并可一键清理失效（401）凭证。基于 `asyncio` + `aiohttp`，默认 200 路协程并发。

## 前置依赖

```bash
pip install aiohttp
```

## 工作流程

```bash
# 常规扫描（只看不删）
python3 <SKILL目录>/scanner.py --no-quarantine

# 扫描 + 删除 401 死号
python3 <SKILL目录>/scanner.py --no-quarantine --delete-401 --yes

# JSON 机读模式
python3 <SKILL目录>/scanner.py --output-json --no-quarantine
```

## 清理规则

只清理明确失效的认证文件。

### 不清理（网络/瞬时错误）
- `network error`
- `timeout`
- `parse error`

### 可清理（明确失效）
- HTTP 401 Unauthorized
- `invalid auth`
- `revoked`

## 执行纪律

> [!IMPORTANT]
> - 必须使用 **Python 3** 环境运行。
> - 汇报时，提取输出底部 "Scan Summary" 中的统计数字，以友好的 Markdown 摘要呈现。
> - 涉及 `--delete-401` 等破坏性操作前，必须确认用户有明确的删除意图（如「删掉」「清理」「扬了」）。只说「看看」「扫一下」的情况下**只扫描不删除**。

---
> Source: [paradoxie/cpa-codex-auth-sweep](https://github.com/paradoxie/cpa-codex-auth-sweep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-18 -->
