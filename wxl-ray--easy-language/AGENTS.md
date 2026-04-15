## 目标

为仓库内的 AI 代码助手（Copilot / agents）提供精确、可操作的上下文，使其能立即开始修改或扩展此项目中的交易脚本。

## 代码库高层（Big picture）

- 这个仓库目前包含一个 EasyLanguage 风格的交易脚本（文件：`.tmLanguage`）和一个用 mermaid 绘制的流程图（`.mmd`）。
- `.tmLanguage` 是主要源码：实现了基于两条移动平均线（MAq / MAs）与一个名为 `HiLo[10,10]` 的二维数组来检测趋势（金叉/死叉）并绘制趋势线。
- 变更通常会直接修改 `.tmLanguage` 中的逻辑或在图中 (`.mmd`) 更新流程说明。

## 关键模式与注意点（来自代码）

- 变量与含义：
  - `MAq = Average(close, MaLen1)`（快速均线），`MAs = Average(close, MaLen2)`（慢速均线）。
  - `MApenLen` 用作窗口宽度（例如：确定最近 N 根 K 线是否全部满足条件）。
  - `HiLo[10,10]` 用于保存多次历史高/低点、状态值（`HiLo[0,0]` 表示当前状态：1 = 多头，-1 = 空头，0 = 无状态）。

- 核心流程（可直接映射到代码）：
  - 检测多头：当 `MAq[MApenLen-1]` 上穿 `MAs[MApenLen-1]` 且最近 `MApenLen` 根 K 线满足 `MAq > MAs`，且 `HiLo[0,0] != 1` 时触发。
  - 多头触发后：数组历史记录右移、计算并保存高点/低点、调用 `tl_new_bn(...)` 创建趋势线并 `tl_setcolor(..., green)`，将 `HiLo[0,0] = 1`。
  - 空头逻辑在代码中是 “半实现” 状态：`MAq 下穿 MAs` 的分支存在但未完整，请在修改时补全并参考多头分支的对称实现。

- 常见坑和值得注意的实现细节：
  - 代码中存在命名不一致的迹象（例如 `HiL0` 与 `HiLo` 的混用）。在修改时务必核对变量名并运行编译/语法检查（目标平台）。
  - `HiLo` 的索引语义很重要：`HiLo[0,1]`、`HiLo[1,0]` 等约定用于不同含义，编辑时请保持索引含义一致，避免打乱历史移位逻辑。
  - 趋势线 ID 的生成与颜色设置（`tl_new_bn` / `tl_setcolor`）必须成对正确使用，否则 UI 上会出现未定义或重复的线条ID。

## 修改/PR 建议（可作为任务 checklist）

1. 修复命名不一致（搜索 `HiL0`、`HiLo` 并统一）并在注释中说明改动。示例：将所有 `HiL0` 替换为 `HiLo`（若语义一致）。
2. 完整实现空头（死叉）分支，镜像多头分支的步骤：数组移位、低点判断、绘制趋势线、更新 `HiLo[0,0] = -1`。
3. 在关键位置添加注释，解释 `HiLo` 索引含义与 `MApenLen` 的用途。
4. 保持现有绘图调用（Plot1/Plot2、tl_new_bn、tl_setcolor）不变，除非修复明确的 bug。

## 工作流与验证（项目特定）

- 本仓库没有 CI、构建脚本或单元测试。要验证改动，必须在目标交易平台（例如 TradeStation / MultiCharts / NinjaTrader 的 EasyLanguage 编辑器）中手动：
  1. 在目标平台中新建或打开脚本，粘贴 `.tmLanguage` 的内容并编译（或保存为策略/指标）。
  2. 将指标附加到历史或实时图表，观察：两条均线（蓝/红）是否绘制，趋势线是否在触发时出现且颜色正确，且多头/空头状态切换合理。
  3. 对于逻辑性修改，使用已知历史序列回测（回测或回放）以确认触发条件的正确性。

## 例子：代码片段说明（来自 `.tmLanguage`）

- 多头触发条件（示例）:

  MAq[MApenLen - 1] cross over MAs[MApenLen - 1] and CountIf(MAq > MAs, MApenLen) = MApenLen and HiLo[0,0] <> 1

- 创建趋势线并设置颜色（示例）:

  HiLo[0,5] = tl_new_bn(HiLo[3,1],HiLo[1,1],HiLo[4,1],HiLo[2,1]);
  tl_setcolor(HiLo[0,5],green);

## 交互与约束（给 AI 助手的具体指导）

- 不要进行没有平台验证的语法或行为变更；重大逻辑变更（例如替换 MA 类型、改变窗口宽度含义）前，请在 PR 描述中说明验证计划并请求人工测试。 
- 当你发现拼写/命名错误（如 `HiL0`），先在小范围内修复并运行一次平台验证，避免一次性更改多个独立问题。
- 若需要新增示例输入或测试数据，请先向仓库维护者请求历史数据样本或截图回放日志。

## 文件参考

- 源码示例：`.tmLanguage` （主要逻辑）
- 文档/流程图：`.mmd`（mermaid 流程，展示了多头/空头分支与数组移位逻辑）

---

如果你希望我把空头分支完整补全、或把 `HiLo` 变量名统一并创建一个小的 PR，我可以直接修改 `.tmLanguage` 并提交一个草稿供你审阅；或请告诉我你希望优先解决的具体问题。 

请告知是否需要把本文件翻译成英文或扩充成更长的 agent 运行手册（包含示例回测步骤、截图模板）。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wxl-ray) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
