# Fava Budget Freedom - AI Agent Guide

## 项目概览

这是一个 **Fava 扩展插件**，为 Beancount 账本提供增强的预算管理功能。该插件集成到 Fava Web 界面，通过解析自定义 Beancount 指令实现灵活的预算跟踪。

### 核心架构

**入口点**: `fava_budget_freedom:BudgetFreedom` (在 `pyproject.toml` 中注册为 Fava 插件)

**数据流**:

1. Beancount 文件 → 包含 `custom "budget"` 指令
2. `BudgetParser` → 解析预算定义
3. `UsageCalculator` → 计算实际支出（全部条目用于历史数据，过滤条目用于报表）
4. `BudgetCalculator` → 处理滚存计算和有效预算
5. `BudgetFreedom.__init__.py` → 生成报表数据
6. `templates/BudgetFreedom.html` → Jinja2 模板渲染

### 关键组件说明

**`__init__.py:BudgetFreedom`** - 主扩展类

- 继承自 `FavaExtensionBase`
- `generate_budget_report()`: 核心方法，协调所有计算并生成报表
- 使用两阶段预算调整：计算总预算 → 减去直接子级预算（避免重复计算）

**`budget.py:BudgetParser`** - 预算定义解析器

- 解析 `custom "budget" "Pattern" "Period" "Amount Currency" ["rollover"]` 指令
- 返回按日期排序的预算定义字典 `{pattern: [budget_defs]}`

**`usage.py:UsageCalculator`** - 支出计算

- `calculate_usage_for_period()`: 用于滚存计算（使用所有历史条目）
- `calculate_all_usages()`: 用于报表（使用过滤条目）
- **特异性规则**: 交易仅计入最具体（最长）的匹配模式

**`calculator.py:BudgetCalculator`** - 预算计算逻辑

- 处理 `rollover` 功能：累积年初至报告期开始的所有月份余额/超支
- 支持多周期预算（`monthly`, `weekly`, `quarterly`, `yearly`）
- 仅月度预算支持滚存

**`utils.py`** - 工具函数

- `matches_pattern()`: 通配符匹配逻辑（支持 `*` 和精确匹配）
- `is_subset()`: 判断模式的子集关系（用于层级预算调整）

## 开发规范

### Beancount 集成模式

**自定义指令格式**:

```beancount
YYYY-MM-DD custom "budget" "Pattern" "Period" "Amount Currency" ["rollover"]
```

**通配符规则**:

- `Expenses:Food` → 精确匹配或匹配子账户 (`Expenses:Food:*`)
- `Expenses:*` → 通配符匹配所有子账户
- 使用 `fnmatch.fnmatch()` 进行模式匹配

### 预算调整逻辑

在 `__init__.py:_generate_report_rows()` 中实现两阶段计算：

1. **Gross Budget**: 模式匹配的所有交易总额
2. **Adjusted Budget**: `gross_budget - sum(direct_children_budgets)`
   - 仅减去**直接子级**预算（非嵌套子级）
   - 避免父级模式重复计入子级预算的支出

### 滚存计算机制

仅适用于 `period="monthly"` 且 `rollover=True` 的预算：

- **年初至报告期开始**: 累积每月 `(预算 - 实际)` 余额
- **报告期**: 添加该期间的月度预算总和
- 超支（负余额）会减少后续月份的可用额度

### Fava 上下文获取

```python
try:
    entries, date_range = g.filtered.entries, g.filtered.date_range
except:
    entries, date_range = self.ledger.all_entries, None
```

- 优先使用 Fava 的过滤上下文 (`g.filtered`)
- 回退到完整账本数据

## 本地开发工作流

### 启动开发服务器

```bash
# 设置 PYTHONPATH 以加载本地插件
PYTHONPATH=. fava example.beancount
```

### 测试数据

使用 `example.beancount` 进行测试，包含：

- 多年份预算定义（2024, 2025, 2026）
- 月度/年度预算示例
- 滚存与非滚存预算对比
- 通配符模式测试（`Expenses:*`, `Expenses:Food`）

### 调试技巧

- 查看 `print()` 输出：插件加载时会在控制台显示
- 检查 Fava 日志：`fava example.beancount` 的终端输出
- 验证预算定义：确保日期早于报告期结束日期

## 常见修改场景

### 添加新预算周期类型

1. 在 `calculator.py:_calculate_period_budget()` 中添加周期逻辑
2. 更新 `budget.py` 的解析验证（可选）

### 调整进度条颜色逻辑

编辑 `templates/BudgetFreedom.html` 中的 Jinja2 模板：

- 主逻辑在 `{% if row.time_percent is not none %}` 分支
- 颜色规则：绿色（低于时间进度）→ 黄色（+10%）→ 红色（超支）

### 修改账户链接行为

模板中 `onclick="window.open(this.href); return false;"` 控制打开新窗口，可改为 `href="{{ url_for('account', ...) }}"` 直接跳转

## 架构决策记录

**为什么使用两个 UsageCalculator 实例？**

- `usage_calculator_all`: 使用全部历史条目计算滚存（需要年初至今的完整数据）
- `usage_calculator_report`: 使用过滤条目计算报表行（遵循 Fava 的时间过滤）

**为什么减去直接子级预算？**

- 父级模式（如 `Expenses:*`）会匹配所有子级交易
- 子级预算（如 `Expenses:Food`）已单独计算
- 不减去会导致交易被重复计入，高估总支出

**为什么仅月度预算支持滚存？**

- 月度预算具有明确的周期边界（每月 1 日）
- 其他周期（周/季/年）的滚存逻辑复杂且使用场景少

**预算定义的年度有效性**

- `monthly` 和 `yearly` 预算仅在**定义年份**有效
- 示例：2025 年定义的预算不会自动应用到 2026 年
- 在 `__init__.py:_generate_report_rows()` 中通过 `latest_budget['date'].year != report_start.year` 过滤
- 其他周期类型（`weekly`, `quarterly`）可跨年使用（更宽松的策略）
- **原因**：避免旧预算意外应用到新年度，确保预算配置的明确性

---
> Source: [Leon2xiaowu/fava_budget_freedom](https://github.com/Leon2xiaowu/fava_budget_freedom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-25 -->
