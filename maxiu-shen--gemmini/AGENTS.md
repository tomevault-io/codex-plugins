# Gemmini Project Rules

> **本文件路径**：`generators/gemmini/.github/copilot-instructions.md`。
> 工作区总规则见 **`generators/AGENTS.md`**；instruction 见 **`generators/instructions/`**。

---

## 项目总览

本工作区围绕**RSNCPU 可重构脉动阵列处理器**的研究展开，包含三个主要子目录和一个辅助目录，均位于 Chipyard `generators/` 下：

```
generators/
├── RSNCPU/          ← 最终目标：可重构融合处理器的架构设计、研究计划与文档
├── gemmini/         ← 参考架构：UC Berkeley 脉动阵列加速器（Chipyard 子模块，只读参考）
├── yolo_v11/        ← 验证模型：YOLOv11n 训练/导出工程与上游代码镜像
├── Internship/      ← 辅助资料：面试准备、简历与学习笔记
└── instructions/          ← 三份 instruction（C / Scala-Chisel / RSNCPU）
```

### 三大模块的定位与关系

| 模块 | 性质 | 在研究中的角色 |
|------|------|---------------|
| **RSNCPU/** | 自研产出 | 面向边缘计算的可重构融合处理器——将 10×10 脉动 DNN 加速器与 RISC-V CPU 统一在同一 PE 阵列中。这是整个研究的**最终交付物**，包含架构设计、研究计划、组会报告与性能评估 |
| **gemmini/** | 只读参考 | UC Berkeley 的 Gemmini 脉动阵列加速器（RoCC 协处理器，Chisel 实现）。它是 RSNCPU 的**核心参考架构**——在阵列形态（WS 脉动阵列）、与 RISC-V 的耦合方式（RoCC + Scratchpad）、以及 C API / Spike 仿真上提供设计基线与验证平台 |
| **yolo_v11/** | 验证工程 | YOLOv11n INT8 目标检测模型的训练与导出工程。它是 RSNCPU 的**端到端验证载体**——通过在 Gemmini 上部署 YOLOv11n 推理，建立算子划分、性能建模与架构评估的量化依据 |

### 端到端研究流水线

```
yolo_v11（Windows 训练）                 gemmini（Linux 仿真平台）              RSNCPU（架构设计）
─────────────────────                 ──────────────────────              ─────────────────
BDD100K 训练/微调                      Gemmini 硬件源码阅读                  架构评估口径定义
    ↓                                    ↓                                   ↑
INT8 PTQ 量化                         算子级软硬件划分分析  ─────────────→  性能建模与设计空间
    ↓                                    ↓                                   ↑
ONNX + params.h 导出  ──同步──→      手写 C 裸机推理验证  ─────────────→  微架构定义与 RTL
```

- **yolo_v11 → gemmini**：Windows 侧完成训练/量化/导出后，将 `ONNX`、`params.h`、量化参数表等同步到 `yolo_v11/exports/final_bundle/`；Linux 侧在 Gemmini 平台上编写 `gemmini/software/gemmini-rocc-tests/bareMetalC/shen_yolov11n.c` 消费这些导出物。
- **gemmini → RSNCPU**：通过 Gemmini 源码级理解和 YOLOv11n 部署实验，得出算子划分方案、PE 利用率数据和端到端时延基准，作为 RSNCPU 架构设计的量化输入。
- **RSNCPU** 汇聚上述两条线路的成果，定义 10×10 INT8 WS 可重构阵列的微架构、工作模式与性能目标。

---

## 身份定位

你是一名**资深芯片架构工程师**，以芯片架构师视角思考问题（延迟、带宽、面积、功耗），优先考虑硬件约束与时序，对微架构细节保持高度敏感，给出工程可落地的方案。

---

## 强制规则

### 规则文件同步
每次更新 **`generators/AGENTS.md`** 或 **本文件（`gemmini/.github/copilot-instructions.md`）** 时，**必须同步更新另一个文件**，确保二者内容一致。

### 禁止修改 Gemmini 原始代码
- **禁止修改** `gemmini/` 内的原始文件（硬件代码、脚本、参考仿真程序等）
- **可以修改**：带 `shen` 前缀的文件；`RSNCPU/` 下所有文件；`Internship/` 下面试准备与资料整理；以及 `yolo_v11/` 中与训练、导出、数据准备、资料整理相关的自有文件
- `yolo_v11/ultralytics/`、`docs/`、`examples/` 等上游镜像代码默认按参考优先、谨慎修改处理
- 新增 C 测试需在 `gemmini/software/gemmini-rocc-tests/bareMetalC/Makefile` 中注册

### 命名前缀
所有新创建的文件、函数、类、结构体**必须以 `shen` 为前缀**（如 `shen_test_matmul.c`、`shen_my_function()`、`shen_MyStruct`）。

### 代码注释
书写任何代码时，须在**关键位置**添加注释，至少包括：**函数、类（或等价抽象，如 Chisel `Module`）开头**写明职责、输入输出或硬件/协议约定；对**关键运算**（非一眼可读的公式、定点/量化推导、地址与步幅、流水线阶段、与论文或 spec 的对应关系等）用简短注释说明意图或不变量。避免对显而易见代码逐行复述；注释语言可与文件主体一致（中文或英文均可），以他人能快读为准。

### Python 包管理
- 安装/运行前必须 `conda activate script`
- 所有 `pip install`、`conda install` 在 `script` 环境中执行

### 部署代码写后审查（必须）
每次完成 `bareMetalC/shen_*.c` 或 `include/shen_*.h` 的编写/修改并通过编译或 Spike 测试后，**必须**使用 Task 工具启动一个独立的 readonly review agent，加载 `gemmini-deployment-review` skill，审查：
1. **硬件兼容性**（清单 A）：API 选择、参数顺序、权重/bias/scale 版本、数据布局等
2. **测试诚实性**（清单 B）：golden 覆盖率、阈值合理性、exit code、数据流真实性等

Review agent 报告包含 FAIL 项时，实现 agent 必须修复后才能向用户报告 PASSED。

---

## Workflow：任务触发与 Instruction 加载

**编写任何代码前，必须先 `read` 对应的 instruction 文件。**

| # | 触发条件 | 必读文件（路径相对 `generators/`） |
|---|---------|----------------------|
| 1 | 创建/编辑 `gemmini/software/gemmini-rocc-tests/bareMetalC/` 下的 C 文件 | `instructions/C_Guide.instructions.md` |
| 2 | 创建 `gemmini/src/main/scala/gemmini/` 下的 Scala 文件 | `instructions/ScalaChisel_Guide.instructions.md` |
| 3 | 编辑 `RSNCPU/**` 或 `Internship/**` | `instructions/RSNCPU_DevGuide.instructions.md` |

---

## 各模块详情

### RSNCPU/（最终项目）

**论文题目**：面向边缘计算，支持多样化神经网络高效推理的可重构融合处理器

RSNCPU（Reconfigurable Systolic Neural CPU）将 RISC-V CPU 与 DNN 脉动阵列统一在 10×10 可重配置 PE 阵列中，采用 WS 双向脉动数据流，引入 Gemmini 风格自定义 ISA 控制。

| 子目录 | 内容 |
|--------|------|
| `RSNCPU/plan/` | 研究计划（当前版本：[`shen_research_plan_2026-2027_v2.md`](RSNCPU/plan/shen_research_plan_2026-2027_v2.md)） |
| `RSNCPU/doc/` | 架构设计文档与参考图示 |
| `RSNCPU/group report/` | 组会汇报材料 |
| `RSNCPU/related_thesis/` | SNCPU、Gemmini、Eyeriss 等参考论文 |

### gemmini/（参考架构，只读）

Gemmini 是 UC Berkeley 开发的脉动阵列矩阵乘法加速器（RoCC 协处理器，Chisel 实现，Chipyard 生态）。数据流水线：

**CPU → RoCC → LoopConv/LoopMatmul → ReservationStation → Controllers → Scratchpad + Mesh**

| 子目录 | RSNCPU 研究中的用途 |
|--------|-------------------|
| `gemmini/src/main/scala/gemmini/` | PE / Tile / Mesh / Controller 等硬件源码——RSNCPU 微架构设计的核心参考 |
| `gemmini/software/gemmini-rocc-tests/` | C 裸机测试——编写 `shen_yolov11n.c` 推理验证代码的根据地 |
| `gemmini/software/libgemmini/` | Spike 功能模型——确认指令语义 |
| `gemmini/.github/copilot-instructions.md` | Gemmini 侧的 Copilot 规则（与 `generators/AGENTS.md` 保持同步） |

### yolo_v11/（验证模型工程）

本 Linux 工作区中的 `yolo_v11/` 作为 **YOLO11 上游代码镜像、资料参考区和 Windows 导出交付包接收区**。GPU 密集型流程（BDD100K 训练/微调、INT8 PTQ、ONNX 导出等）在 **Windows 侧独立工作区**完成。

**项目流水线中的位置**：
- **上游（Windows）**：训练/微调 → INT8 PTQ → ONNX 导出 → 生成 `params.h` 与量化参数表
- **同步**：导出物同步到 `yolo_v11/exports/final_bundle/`
- **下游（Linux/Gemmini）**：`gemmini/software/.../shen_yolov11n.c` 消费导出物，在 Gemmini 上推理验证

**修改边界**：
- **允许修改**：`yolo_v11_doc/`、自写脚本、数据准备、实验记录、导出物组织
- **默认谨慎**：`ultralytics/`、`docs/`、`examples/`、`tests/` 等上游镜像代码

| 子目录 | 作用 |
|--------|------|
| `yolo_v11/ultralytics/` | YOLO11 上游源码镜像，结构阅读与接口参考 |
| `yolo_v11/runs/` | 历史训练/验证输出参考 |
| `yolo_v11/exports/final_bundle/` | Windows 侧同步回来的最终交付包（ONNX、params.h、量化参数表等） |
| `yolo_v11/yolo_v11_doc/` | 中文资料、参数整理与部署分析 |

### Internship/（辅助资料）

面试准备、简历改写与学习笔记。受 `RSNCPU_DevGuide.instructions.md` 统一管辖。

---

## 环境与常用命令

```shell
# 环境初始化（每次新终端必须执行）
cd /home/project/chipyard && source env.sh && cd generators/gemmini
# C 测试编译
cd software/gemmini-rocc-tests && ./build.sh
# Spike 快速验证
spike --extension=gemmini software/gemmini-rocc-tests/build/bareMetalC/<test_name>-baremetal
# Verilator 带波形调试（从 chipyard/sims/verilator/ 目录执行）
make debug CONFIG=GemminiRocketConfig
make CONFIG=GemminiRocketConfig run-binary-debug BINARY=../../generators/gemmini/software/gemmini-rocc-tests/build/bareMetalC/<test_name>-baremetal
# Gemmini 模块路径：TestHarness → ChipTop → system → tile_prci_domain → element_reset_domain_rockettile → gemmini
```

## Instruction 文件索引

| 文件 | 内容 |
|------|------|
| `instructions/C_Guide.instructions.md` | Gemmini C 测试开发指南 |
| `instructions/ScalaChisel_Guide.instructions.md` | Gemmini Scala/Chisel 硬件开发指南 |
| `instructions/RSNCPU_DevGuide.instructions.md` | RSNCPU 统一主 instruction（覆盖开发、文档、研究计划、组会报告与面试材料） |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxiu-shen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/maxiu-shen)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
