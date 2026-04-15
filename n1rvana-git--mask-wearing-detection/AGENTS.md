# AI Coding Agent Instructions - 口罩检测系统

## 项目概览

这是一个**毕业设计项目**，基于YOLOv11n的口罩佩戴检测系统。项目目标是构建端到端的智能检测系统，包含模型训练、后端API服务和微信小程序前端。

**核心约束**：
- **开题报告驱动**：所有工作必须服务于`开题报告.md`中承诺的研究目标和技术指标
- **学术严谨性**：实验设计需支持论文写作，必须包含消融实验和基线对比
- **资源受限部署**：目标平台是微信小程序（Web环境），模型需轻量化（≤80MB）
- **无数据库架构**：系统采用"瞬时处理"模式，所有检测结果在内存中处理后即时返回
- **RTX 3050 Laptop GPU**：4GB显存限制，批次大小需自动优化

## 项目架构

```
核心组件流：
1. YOLOv7/YOLOv11n模型 (yolov7/, models/)
   → 训练脚本优化GPU使用 (batch_size_optimizer.py)
   → 模型轻量化/ONNX导出 (models/yolov7_model_loader.py)

2. Flask后端 (backend/)
   → 模型加载器 (utils/model_loader.py) - 单例模式，启动时加载
   → 检测API (api/detection.py) - RESTful接口
   → 无状态设计：不存储检测历史

3. 前端 (frontend/, 未来迁移到微信小程序)
   → <camera>采集 → WebSocket → Flask推理 → <canvas>渲染
```

**关键设计决策**：
- **YOLOv7官方代码集成**：`yolov7/`子模块作为训练基础，训练脚本位于`models/train_yolov7_mask_detection.py`
- **端云协同架构**：前端只负责采集和渲染，所有AI推理在后端完成
- **内存优先策略**：无数据库、无持久化存储，适用于隐私敏感场景

## 开发工作流

### 1. 模型训练与优化
```powershell
# 自动优化批次大小并训练（根据4GB显存）
python models/train_yolov7_mask_detection.py --model yolov7 --epochs 100 --batch-size -1

# 查看GPU优化建议
python batch_size_optimizer.py

# 测试训练流程（快速验证）
python simple_training_test.py
```

**重要模式**：
- 训练前必须调用`batch_size_optimizer.py`获取推荐配置
- 数据集配置文件：`data/mask_detection.yaml`（2类：no_mask, mask）
- 训练输出统一到`runs/yolov7_mask_detection/`，权重保存到`weights/`子目录

### 2. 后端服务开发
```powershell
# 启动Flask开发服务器（自动加载模型）
python backend/app.py

# 健康检查
curl http://localhost:5000/api/health

# 测试API
python tests/test_api.py
```

**API设计原则**：
- **POST /api/detect**：单张图像检测（multipart/form-data）
- **POST /api/detect_base64**：Base64编码图像检测（为小程序准备）
- **POST /api/batch_detect**：批量检测
- 所有接口返回JSON格式：`{detections: [{class, confidence, bbox}], inference_time, ...}`

### 3. 测试与验证
```powershell
# 完整测试套件
pytest tests/ -v

# 仅测试API（跳过前端）
pytest tests/test_api.py -v

# 性能测试
pytest tests/test_performance.py -v -m performance
```

**测试架构特点**：
- `conftest.py`：自动等待服务启动、创建测试图像、清理环境
- 标记系统：`@pytest.mark.api`, `@pytest.mark.frontend`, `@pytest.mark.performance`
- 性能基准：单图检测时间≤3秒（开题报告要求）

## 项目特定约定

### 数据集管理
```yaml
# data/mask_detection.yaml 标准结构
train: E:/graduation_project/datasets/mask_detection/train
val: E:/graduation_project/datasets/mask_detection/val
test: E:/graduation_project/datasets/mask_detection/test
nc: 2  # 类别数固定为2
names: [no_mask, mask]
```
- 使用**绝对路径**（Windows环境）
- 数据准备脚本：`data/prepare_dataset.py`
- 数据增强策略在训练脚本中配置（Mosaic、Mixup等）

### GPU内存优化模式
```python
# 自动配置示例（batch_size_optimizer.py）
def get_training_config(model_size='yolov5s', img_size=640):
    gpu_memory = 4.0  # RTX 3050 Laptop
    if model_size == 'yolov5n':
        return {'batch_size': 32, 'workers': 8}
    elif model_size == 'yolov5s':
        return {'batch_size': 32, 'workers': 8}
    # ... 根据显存自动调整
```
- **混合精度训练**（AMP）默认启用
- 训练前后自动清理GPU缓存（`torch.cuda.empty_cache()`）
- 批次大小设为`-1`触发自动优化

### 模型轻量化流程
1. **训练**：使用YOLOv7-tiny或YOLOv11n基线
2. **算法改进**：参考`开题报告.md`中的文献（DyHead、GAM注意力、PConv等）
3. **量化**：导出ONNX格式，目标≤80MB
4. **消融实验**：必须对比Baseline vs. Baseline+模块A vs. Baseline+模块B

## 关键文件与模块

| 文件/目录 | 用途 | 何时修改 |
|----------|------|---------|
| `开题报告.md` | **项目宪章**，定义所有目标和约束 | **只读**，所有工作需对齐此文件 |
| `models/train_yolov7_mask_detection.py` | YOLOv7训练入口 | 添加新的网络结构改进时 |
| `batch_size_optimizer.py` | GPU配置优化器 | 支持新模型或GPU硬件时 |
| `backend/utils/model_loader.py` | 模型加载单例 | 切换模型格式（.pt → .onnx）时 |
| `data/mask_detection.yaml` | 数据集配置 | 数据集路径变更时 |
| `tests/conftest.py` | pytest全局配置 | 添加新的测试fixture时 |

## 部署注意事项

### Docker部署
```powershell
# 构建镜像（基于deployment/docker/Dockerfile）
docker build -t mask-detection -f deployment/docker/Dockerfile .

# 运行容器（GPU支持）
docker run -p 5000:5000 --gpus all mask-detection
```

### 微信小程序集成（未来）
- **通信协议**：WebSocket（低延迟实时通信）
- **抽帧策略**：5-10 FPS（避免后端过载）
- **Base64 API**：使用`/api/detect_base64`接口
- **前端渲染**：`<canvas>`绘制检测框

## 常见问题解决

### 1. GPU内存不足（OOM）
```powershell
# 查看推荐批次大小
python batch_size_optimizer.py

# 使用自动优化
python models/train_yolov7_mask_detection.py --batch-size -1
```

### 2. 模型训练精度不达标
**遵循五步法**（参考`prompts/Detection.md`）：
1. 回溯开题报告：确认目标指标（mAP@0.5 ≥ 92%）
2. 数据审查：检查标注质量、类别平衡
3. 技术选型：参考文献[1]-[18]中的改进方案
4. 消融实验：设计对照组
5. 论文写作：提炼贡献点

### 3. API响应时间过长
- 检查模型推理时间：`inference_time`字段
- 优化图像预处理：`backend/utils/image_utils.py`
- 考虑ONNX Runtime加速或TensorRT

## 核心原则

1. **开题报告优先**：任何代码变更都需对齐`开题报告.md`中的研究目标
2. **实验驱动开发**：重要改进必须有消融实验支撑
3. **资源感知编程**：始终考虑4GB显存限制和80MB模型大小约束
4. **学术可复现性**：所有实验配置需在代码或文档中明确记录
5. **无状态API设计**：后端不保存检测历史，符合隐私保护需求

---

**特别提示**：本项目是毕业设计，交付物包括**可运行的系统**和**学术论文**。在提供代码建议时，需同时考虑工程实现和论文写作的需求。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/N1rvana-git) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
