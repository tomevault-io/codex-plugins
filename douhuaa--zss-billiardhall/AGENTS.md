# Copilot Instructions

> **本规则优先级：最高 | 覆盖所有其他配置 | 不可豁免**
> 本文档统一调度所有 Specialist 与 Guardian instructions，并定义触发规则、委托原则与反馈闭环。

## 语言使用规则（Language Usage Rules）

**强制要求**：
- ✅ **所有 PR 标题必须使用中文**
- ✅ **所有 PR 描述必须使用中文**
- ✅ **所有任务回复必须使用中文**
- ✅ **所有进度报告必须使用中文**
- ✅ **所有 FailureObject 输出必须使用中文**
- ✅ **任务开始、执行中、完成时的所有响应必须使用中文**

**禁止行为**：
- ❌ 禁止在任务完成时切换为英文回复
- ❌ 禁止在 PR 更新时使用英文
- ❌ 禁止在反馈报告中使用英文

> ⚠️ **重要提醒**：本规则适用于整个任务生命周期，从开始到完成，无一例外。

---

## 基础约束（所有 Agent 必须遵守）

**本文档基于以下 ADR**：
- [ADR-007：Agent 行为与权限宪法](../docs/adr/constitutional/ADR-007-agent-behavior-permissions-constitution.md)
- [ADR-900：架构测试与 CI 治理元规则](../docs/adr/governance/ADR-900-architecture-tests.md)
- [ADR-907：ArchitectureTests 执法治理体系](../docs/adr/governance/ADR-907-architecture-tests-enforcement-governance.md)
- [ADR-907-A：ADR-907 对齐执行标准](../docs/adr/governance/adr-907-a-adr-alignment-execution-standard.md)

### 权威性声明

> ⚖️ **当本文档与上述 ADR 存在冲突时，以 ADR 正文为唯一裁决依据。**
> 
> 本文档仅作为 Agent 调度与协调的操作指南，不具备架构裁决权。

---

## 委托原则

根据 **ADR-007.1**（Agent 职责分离原则），本系统采用以下委托原则：

* **所有任务**必须通过 `run_subagent` 调用对应 Agent 执行。
* Guardian **仅协调与汇总**，禁止直接做最终裁决（参见 **ADR-007.2.1**）。
* 未明确职责的操作必须标记为 `Uncertain` 并引导人工确认。
* 所有 Agent 输出必须符合**三态判定规则**（见下节）。

### 三态输出规则（基于 ADR-007.2）

所有 Specialist 和 Guardian Agent 必须使用以下三态输出之一：

| 判定 | 含义 | 适用场景 | 后续操作 |
|------|------|---------|---------|
| ✅ **Allowed** | ADR 正文明确允许且经测试验证 | 变更符合所有约束 | 继续执行 |
| ⚠️ **Blocked** | ADR 正文明确禁止或导致测试失败 | 存在架构违规 | 阻止合并，需修复 |
| ❓ **Uncertain** | ADR 正文未明确，默认禁止 | 需要人工裁决的情况 | 升级至 Architecture Board |

**禁止行为**（ADR-007.3.2）：
- ❌ 禁止使用"可能"、"建议"、"推荐"等模糊判定
- ❌ 禁止在无 ADR 支持的情况下做出 Blocked 判定
- ❌ 禁止 Guardian 直接推翻 ADR 正文的明确规定

---

## 可用 Agent（Specialist & Guardian）

| Agent 名称                               | 说明                             | 主要 ADR 依赖 | instructions 文件                                                                                                         |
|----------------------------------------|--------------------------------|------------|--------------------------------------------------------------------------------------------------------------------------|
| 架构守护（Guardian）                         | 监督协调所有架构约束，三态判定 Agent 输出       | ADR-007, ADR-900, ADR-907 | [`architecture-guardian.instructions.yaml`](./instructions/architecture-guardian.instructions.yaml)                     |
| ADR 审查（ADR Reviewer）                   | 审查 ADR 文档质量、关系与变更              | ADR-902, ADR-907-A, ADR-940 | [`adr-reviewer.instructions.yaml`](./instructions/adr-reviewer.instructions.yaml)                                       |
| 文档维护（Documentation Maintainer）         | 文档维护、链接检查、索引更新                 | ADR-008, ADR-910, ADR-940 | [`documentation-maintainer.instructions.yaml`](./instructions/documentation-maintainer.instructions.yaml)               |
| .NET 专家工程（Expert Dotnet Engineer）      | 提供 .NET 技术建议、代码规范检查            | ADR-001 ~ 005 | [`expert-dotnet-software-engineer.instructions.yaml`](./instructions/expert-dotnet-software-engineer.instructions.yaml) |
| Handler 模式强制（Handler Pattern Enforcer） | 强制 Handler 模式执行、约束验证           | ADR-201, ADR-240 | [`handler-pattern-enforcer.instructions.yaml`](./instructions/handler-pattern-enforcer.instructions.yaml)               |
| 模块边界检查（Module Boundary Checker）        | 检查模块边界规则、依赖约束                  | ADR-001, ADR-003, ADR-005 | [`module-boundary-checker.instructions.yaml`](./instructions/module-boundary-checker.instructions.yaml)                 |
| 测试生成（Test Generator）                   | 根据 ADR 与约束生成 ArchitectureTests | ADR-900, ADR-907, ADR-907-A | [`test-generator.instructions.yaml`](./instructions/test-generator.instructions.yaml)                                   |

**详细配置**：每个 Agent 的详细行为规范、权限边界、ADR 映射请参见 [`agents/`](./agents/) 目录下的对应 `.agent.md` 文件。

---

## 任务触发规则（Trigger Rules）

```yaml
pull_request:
  run_subagents:
    - adr-reviewer
    - architecture-guardian
    - test-generator

push:
  paths:
    - "src/**":
        run_subagents:
          - module-boundary-checker
          - handler-pattern-enforcer
          - expert-dotnet-software-engineer
    - "docs/adr/**":
        run_subagents:
          - documentation-maintainer
          - adr-reviewer

ci_pipeline:
  run_subagents:
    - architecture-guardian
    - all_L1_architecture_tests
```

> 每次任务执行都会生成 `FailureObject`，保证可追踪、可审计。

---

## FailureObject 标准格式

> ⚠️ **语言要求**：所有 FailureObject 字段内容必须使用中文。

根据 **ADR-007.4**（失败反馈机制），当 Agent 判定为 **Blocked** 或 **Uncertain** 时，必须生成规范的 FailureObject：

```json
{
  "decision": "Blocked" | "Uncertain",
  "agent": "agent-name",
  "timestamp": "2026-02-03T17:00:00Z",
  "rule_violations": [
    {
      "rule_id": "ADR-XXX_Y_Z",
      "violated_clause": "具体违反的条款描述",
      "evidence": [
        "证据1：代码位置、测试失败信息等",
        "证据2：..."
      ],
      "severity": "Critical" | "High" | "Medium"
    }
  ],
  "remediation": {
    "required_actions": ["修复步骤1", "修复步骤2"],
    "reference_docs": ["ADR-XXX链接", "相关文档链接"],
    "estimated_effort": "2h" | "1d" | "1w"
  },
  "escalation": {
    "required": true | false,
    "escalate_to": "Architecture Board" | "Tech Lead",
    "reason": "为什么需要升级"
  }
}
```

**FailureObject 必需字段**（ADR-007.4.1）：
- `decision`：必须是 Blocked 或 Uncertain
- `rule_id`：违反的具体 RuleId（格式：`ADR-XXX_Y_Z`）
- `evidence`：至少一条可验证的证据
- `remediation`：明确的修复建议

**禁止行为**（ADR-007.4.2）：
- ❌ 禁止生成无 RuleId 引用的 FailureObject
- ❌ 禁止生成无修复建议的 FailureObject
- ❌ 禁止使用模糊的证据（如"感觉不对"、"可能有问题"）

---

## 反馈与报告（Feedback & Reporting）

> ⚠️ **语言要求**：所有反馈、报告、进度更新必须使用中文，包括任务完成时的最终回复。

根据 **ADR-900.4**（破例治理机制）和 **ADR-907.3**（失败处理流程），建立以下反馈闭环：

```yaml
feedback_loop:
  source: guardian
  failure_object:
    required: true
    format: "见上节 FailureObject 标准格式"
  actions:
    - update_adr              # 如果发现 ADR 规则不合理
    - enhance_architecture_tests  # 如果发现测试覆盖不足
    - adjust_agent_config     # 如果发现 Agent 行为不当
    - create_exemption_issue  # 如果需要合法破例
```

### 反馈处理流程

1. **Guardian 汇总**：
   - 收集所有 Specialist Agent 的输出
   - 执行三态判定（✅ Allowed / ⚠️ Blocked / ❓ Uncertain）
   - 生成 FailureObject（如需要）

2. **失败分类**（ADR-907.3.1）：
   - **L1 失败**：架构测试失败 → 自动阻断 CI
   - **L2 失败**：Analyzer 告警 → 需人工 Code Review
   - **L3 失败**：需要架构师人工裁决

3. **治理产物生成**（ADR-900.3.2）：
   - 任何失败或异常必须产生治理产物
   - 禁止仅修改 Prompt 而不更新 ADR/测试
   - 所有破例必须通过 Issue 记录并关联 ADR

4. **反馈闭环验证**：
   - 修复完成后，重新运行 Guardian 验证
   - 确认 FailureObject 中的所有问题已解决
   - 更新 ADR-RELATIONSHIP-MAP（如适用）

---

## RuleId 格式规范

根据 **ADR-907-A_1**（编号格式标准），所有 ADR 条款引用必须使用以下格式：

```
ADR-XXX_<Rule>_<Clause>
```

**示例**：
- ✅ 正确：`ADR-900.1.1`、`ADR-001.2.3`、`ADR-907.4.2`
- ❌ 错误：`ADR-900.1:L1`、`ADR-900.1.1`、`ADR-900-1-1`

**适用场景**：
- Agent 输出中引用 ADR 条款
- FailureObject 中的 `rule_id` 字段
- 架构测试方法命名
- 文档交叉引用

**迁移说明**：
- 旧格式 `ADR-XXX.Y:LZ` 已废弃（参见 ADR-907-A）
- 所有 Agent 必须使用新格式
- 遇到旧格式时，自动转换或报告为 Uncertain

---

## 最终提醒（Final Reminders）

> 🔴 **关键规则**：所有 Agent 在整个任务生命周期（从开始到完成）中必须始终使用中文进行回复和报告。
> 
> **适用范围**：
> - PR 标题和描述
> - 任务开始时的响应
> - 执行过程中的进度更新
> - **任务完成时的最终回复**（特别强调）
> - 所有错误报告和 FailureObject
> - 所有反馈和建议
> 
> **违规行为**：在任务完成时切换为英文是不可接受的行为。

---

# ⚠️ 任务完成前最终检查清单

在提交任何输出、完成任何任务、更新任何 PR 之前，**必须**验证以下各项：

- [ ] ✅ **PR 标题使用中文** - 检查是否包含中文字符
- [ ] ✅ **PR 描述使用中文** - 整个描述都是中文
- [ ] ✅ **所有回复使用中文** - 与用户的所有交互
- [ ] ✅ **进度报告使用中文** - 提交信息和更新
- [ ] ✅ **错误信息使用中文** - FailureObject 和警告

**如果以上任何一项未满足，必须重新生成。不允许提交英文内容。**

**记住**：本规则在任务的每个阶段都适用，包括：
1. 任务开始时
2. 执行过程中
3. **任务完成时**（最容易被忽略的时刻）
4. PR 更新时
5. 错误报告时

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/douhuaa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
