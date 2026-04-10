
# VLM-NAV 项目架构文档

> **目的**：帮助 AI 助手快速理解项目结构和代码组织  
> **更新日期**: 2026-01-10

---

## 1. 项目概述

**VLM-NAV** 是一个将 **ETP-R1**（图结构 VLN 模型）与 **Qwen3-VL-2B**（视觉语言模型）融合的导航项目。

### 核心思想：双系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│  System 1 (ETP-R1): 快速空间感知  [冻结]                        │
│  360° RGB+Depth → CLIP/Depth Encoders → VLN-BERT DPFT           │
│  输出: Graph Node Embeddings (B, N, 768)                        │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  Projector: MLPProjector (6层)  [可训练]                        │
│  768 → LayerNorm → Linear → GELU → ResBlock ×4 → Linear → 2048 │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  System 2 (Qwen3-VL + LoRA): 慢思考推理                         │
│  Visual Tokens + Text Prompt → LLM → CoT 推理                   │
│  输出格式: <think>推理</think><answer><node_N></answer>         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 目录结构与关键文件

### 2.1 VLM 训练核心 (`vlnce_baselines_VLM/`)

| 文件 | 说明 | 阅读优先级 |
|------|------|-----------|
| `fusion_offline_trainer.py` | **Stage 1 & 2 离线训练器**（无需 Habitat） | ⭐⭐⭐ |
| `models/fusion/etp_qwen_policy.py` | 双系统融合 Policy 定义 | ⭐⭐⭐ |
| `models/fusion/projectors.py` | MLPProjector 架构定义 | ⭐⭐ |
| `models/fusion/offline_dataset.py` | 离线 CoT 数据集加载 | ⭐⭐ |
| `eval_sap_r2r.py` | R2R 离线评估（无需 Habitat） | ⭐ |
| `vlm_inference_server.py` | VLM 推理/训练服务器 (Stage 3 Server) | ⭐⭐⭐ |
| `fusion_grpo_client.py` | GRPO Client (Stage 3，Habitat 侧) | ⭐⭐⭐ |

### 2.2 ETP-R1 原始代码 (`vlnce_baselines/`)

| 文件 | 说明 | 阅读优先级 |
|------|------|-----------|
| `models/R1Policy.py` | ETP 模型定义（DPFT, 视觉编码器） | ⭐⭐ |
| `ss_trainer_ETP_R1.py` | 原始 SFT (DAgger) 训练器 | ⭐ |
| `GRPO_trainer_ETP_R1.py` | 原始 GRPO 训练器 | ⭐ |
| `extract_cot_aligned_features.py` | **COT 对齐特征提取** | ⭐⭐⭐ |
| `verify_cot_alignment.py` | 特征-COT 对齐验证 | ⭐ |

### 2.3 COT 数据生成 (`data_engine/`)

| 文件 | 说明 | 阅读优先级 |
|------|------|-----------|
| `generate_cot_v2.py` | COT 数据生成主脚本，调用 VLM API | ⭐⭐ |
| `utils_v2.py` | 工具类（ConnectivityGraph, ImageLoader） | ⭐ |
| `config/cot_config.yaml` | COT 生成配置 | ⭐ |

### 2.4 训练脚本 (`run_r2r_30_VLM/`)

| 文件 | 说明 |
|------|------|
| `run_stage1_offline.bash` | Stage 1 Alignment 训练 |
| `run_stage2_offline.bash` | Stage 2 SFT 训练 |
| `run_eval_sap_r2r.bash` | R2R 评估 |
| `etp_qwen_fusion.yaml` | 主配置文件 |

---

## 3. 三阶段训练流程

| 阶段 | 目标 | 可训练参数 | 命令 |
|------|------|-----------|------|
| **Stage 1** | 对齐视觉特征到 LLM 空间 | Projector + lm_head | `bash run_r2r_30_VLM/run_stage1_offline.bash` |
| **Stage 2** | 学习 CoT 推理格式 | Projector + LoRA | `bash run_r2r_30_VLM/run_stage2_offline.bash` |
| **Stage 3** | 闭环强化学习 (GRPO) | Projector + LoRA | Client-Server 架构已完成 |

### Checkpoint 流转

```
ETP-R1 GRPO Checkpoint (iter270.pth)
    ↓ [特征提取: extract_cot_aligned_features.py]
预提取 DPFT 特征 (data/cot_aligned_features/R2R_train/)
    ↓ [Stage 1 训练]
Stage 1 Checkpoint (ckpt.stage1.final.pth)
    ↓ [Stage 2 训练: 加载 pretrained_projector]
Stage 2 Checkpoint (ckpt.stage2.final.pth)
    ↓ [评估]
```

---

## 4. 数据格式

### 4.1 COT 数据 (`R2R_train_cot.jsonl`)

```json
{
  "id": "561_1",                    // {path_id}_{step}
  "scan": "82sE5b5pLXE",
  "viewpoint": "8ce00b6b...",       // 当前位置
  "gt_next": "ad3a2912...",         // 正确的下一个位置
  "instruction": "...",
  "candidates": ["<node_0>", "<node_1>"],  // 候选标签
  "gt_label": "<node_0>",           // 正确答案
  "cot": "<think>...</think><answer><node_0></answer>"  // VLM 推理
}
```

### 4.2 特征文件 (`{path_id}/{step}.pt`)

```python
{
  "gmap_embeds": Tensor(N+1, 768),  # [STOP, cand_0, cand_1, ...]
  "candidate_vp_ids": [None, vp_0, vp_1, ...],
  "gt_idx": int,  # 正确答案索引 (0=STOP, 1=<node_0>, 2=<node_1>)
  "cur_vp": str,
  "num_candidates": int
}
```

### 4.3 索引对应关系

| COT 标签 | 特征 `gt_idx` | 说明 |
|----------|--------------|------|
| STOP | 0 | 导航结束 |
| `<node_0>` | 1 | 第一个候选点 |
| `<node_1>` | 2 | 第二个候选点 |

---

## 5. 关键配置

### LoRA 配置

```yaml
lora_r: 16
lora_alpha: 32
lora_dropout: 0.05
target_modules: ["q_proj", "k_proj", "v_proj", "o_proj"]
```

### Projector 配置

```yaml
PROJECTOR_DIM: 768        # ETP 输出维度
LLM_HIDDEN_DIM: 2048      # Qwen 输入维度
```

---

## 6. 常用命令

```bash
# 激活环境
conda activate train

# Stage 1 训练
bash run_r2r_30_VLM/run_stage1_offline.bash

# Stage 2 训练
bash run_r2r_30_VLM/run_stage2_offline.bash

# 评估
bash run_r2r_30_VLM/run_eval_sap_r2r.bash val_unseen

# 验证特征对齐
python vlnce_baselines/verify_cot_alignment.py \
    --feature_dir data/cot_aligned_features/R2R_train \
    --cot_data pretrain_src/datasets/R2R/cot_generated_v2/R2R_train_cot.jsonl
```

---

## 7. 快速理解路径

**如果你要理解整个项目，按以下顺序阅读代码：**

1. **架构理解**：`Improved_COT.md` → 理解双系统融合思想
2. **训练入口**：`vlnce_baselines_VLM/fusion_offline_trainer.py` → 训练流程
3. **数据流**：`vlnce_baselines_VLM/models/fusion/offline_dataset.py` → 数据加载
4. **模型核心**：`vlnce_baselines_VLM/models/fusion/projectors.py` → Projector 架构
5. **原始 ETP**：`vlnce_baselines/models/R1Policy.py` → ETP 模型结构

---

## 8. 当前状态

- [x] 架构设计
- [x] COT 数据生成 (`data_engine/`)
- [x] 特征提取 (`extract_cot_aligned_features.py`)
- [x] 特征对齐验证 (100% 通过)
- [x] Stage 1 训练代码
- [x] Stage 2 训练代码
- [x] Stage 3 GRPO 代码 (Client-Server 架构)
- [x] 评估代码

---

## 9. 最新工程分析 (Updates)

### 9.1 双环境运行机制 (Dual-Env Architecture)
由于 Habitat (System 1) 依赖 Python 3.6/3.7，而 Qwen3-VL (System 2) 依赖 Python 3.10+，项目运行在两个隔离环境：
- **Client (etpnav)**: `fusion_grpo_client.py` - Habitat 交互、ETP 特征提取、动作执行
- **Server (train)**: `vlm_inference_server.py` - Qwen3-VL 推理、Projector、GRPO 梯度更新

### 9.2 Stage 3 GRPO 架构 ✅ (已完成)
- **Client**: `fusion_grpo_client.py` 实现 `GRPOVLMClient` (TCP 通信) 和 `GRPOEnvInteractor` (Habitat 集成)
- **Server**: `vlm_inference_server.py` 支持 `--mode train` 进行 GRPO 训练，`--mode eval` 进行纯推理
- **API**: `sample_group` (采样), `train_step` (梯度更新), `inference` (评估)
- **关键修复**: Prompt 一致性问题已解决，训练与采样使用相同的 `_build_prompt` 逻辑

### 9.3 代码一致性 ✅
- **Projector 定义**: 已统一引用 `vlnce_baselines_VLM.models.fusion.projectors.MLPProjector`
- **Stage 3 挑战已解决**: GRPO 已拆分为 Client-Server 架构，Habitat 端无需加载 LLM

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zjuer-dm)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/zjuer-dm)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
