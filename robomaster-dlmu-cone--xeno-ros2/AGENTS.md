# XenoCar ROS2 项目 - Copilot 编码助手说明

## 项目概述

XenoCar ROS2 Package 是一个用于大连民族大学 C·ONE 战队 2024 年工程机器人的 ROS2 项目。该项目基于 ROS2 Humble/Jazzy，运行在 Ubuntu >= 22.04 的妙算2-G平台上，实现机械臂的运动规划和控制。

### 仓库信息
- **项目类型**: ROS2 工作空间 (ament_cmake)
- **主要语言**: C++17/20, Python 3
- **框架**: ROS2 Humble/Jazzy, MoveIt2, ros2_control
- **目标平台**: Ubuntu 22.04+ (ARM64/x86_64)
- **仓库大小**: 约 5 个包，主要为中小型项目

## 构建说明

### 环境准备

**始终按以下步骤顺序执行环境配置：**

1. **安装 ROS2**（版本选择基于 Ubuntu 版本）:
   ```bash
   # Ubuntu 24.04
   sudo apt install ros-jazzy-desktop
   # Ubuntu 22.04  
   sudo apt install ros-humble-desktop
   ```

2. **安装 rosdep**（必需的依赖管理工具）:
   ```bash
   sudo apt install python3-rosdep2
   # 需要按官方文档完成 rosdep init 和 update
   ```

3. **在项目根目录安装依赖**（必须在构建前执行）:
   ```bash
   cd /path/to/xeno-ros2
   rosdep install --from-paths src --ignore-src -r -y
   ```

### 构建步骤

**始终在工作空间根目录执行构建：**

```bash
# 基本构建（用于实机运行）
colcon build

# 仿真模式构建（用于开发测试）
colcon build --cmake-args -DXENO_CONTROL_SIMULATE=ON
```

**构建后必须配置环境：**
```bash
source install/setup.bash  # 或 setup.zsh 或 setup.sh
```

### 测试和验证

**仿真模式测试（推荐用于开发）：**

1. **启动 ros2_control 节点**（带图形界面）:
   ```bash
   ros2 launch xeno_control moveit_control.gui.launch.py
   ```

2. **在新终端启动 MoveIt 运动规划**:
   ```bash
   ros2 launch xeno_moveit demo.launch.py
   ```

3. **测试简单移动节点**:
   ```bash
   ros2 run xeno_simple_move xeno_simple_move
   ```

4. **键盘控制测试**:
   ```bash
   # 终端1: 启动键盘控制
   ros2 launch xeno_control keyboard_control.gui.launch.py
   # 终端2: 运行键盘节点
   ros2 run xeno_keyboard xeno_keyboard_node
   ```

**实机运行（无图形界面）：**
```bash
# 终端1
ros2 launch xeno_control moveit_control.launch.py
# 终端2  
ros2 launch xeno_moveit move_group.launch.py
```

### 构建验证要点

- **依赖检查**: 运行 `rosdep check --from-paths src --ignore-src` 验证依赖完整性
- **编译选项**: 使用 `XENO_CONTROL_SIMULATE=ON` 进行仿真开发，实机部署时去掉该选项
- **环境变量**: 每次新终端都必须 source 环境设置
- **超时设置**: 构建时间通常 2-5 分钟，大型更改可能需要 10 分钟

## 代码库布局

### 包架构

```
src/
├── xeno_control/          # 硬件接口和控制器
│   ├── include/xeno_control/
│   │   └── xeno_hardware_interface.hpp
│   ├── src/
│   │   ├── xeno_hardware_interface.cpp
│   │   └── joints/        # 机械臂和升降机构类
│   ├── config/            # 控制器配置文件
│   ├── description/       # URDF和ros2_control描述
│   └── bringup/          # 启动文件
├── xeno_moveit/          # MoveIt2 配置
│   ├── config/           # 运动学、限制和控制器配置
│   └── launch/           # MoveIt启动文件
├── xeno_keyboard/        # 键盘控制接口
├── xeno_simple_move/     # 运动规划示例
└── xeno_urdf/           # 机器人描述文件
```

### 关键配置文件

- **控制器配置**: `src/xeno_control/config/controllers.yaml` - 定义7个关节的控制器
- **MoveIt配置**: `src/xeno_moveit/config/` - 运动学、限制和规划配置
- **硬件描述**: `src/xeno_control/description/urdf/xeno.urdf.xacro` - 机器人模型
- **CMake设置**: 支持 `XENO_CONTROL_SIMULATE` 编译选项用于仿真/实机切换

### 代码依赖

- **外部依赖**: OneMotor 驱动库（自动从 GitHub 获取）
- **ROS2 依赖**: hardware_interface, moveit, pluginlib, controller_manager
- **构建系统**: ament_cmake (C++包), setuptools (Python包)

### 重要源文件详情

**核心硬件接口 (src/xeno_control/)**:
- `src/xeno_hardware_interface.cpp` (103行) - 主硬件接口实现
- `include/xeno_control/xeno_hardware_interface.hpp` (51行) - 硬件接口定义
- `src/joints/Lift.cpp` & `src/joints/Arm.cpp` - 升降和机械臂控制类
- `xeno_control.xml` - pluginlib 插件描述文件

**关键启动文件**:
- `bringup/moveit_control.launch.py` - ros2_control 启动器
- `bringup/moveit_control.gui.launch.py` - 带GUI的控制启动器
- `bringup/keyboard_control.gui.launch.py` - 键盘控制启动器

**配置文件结构**:
- `config/controllers.yaml` - 定义 xeno_group_controller (关节1-6) 和 end_group_controller (关节7)
- `config/keyboard_controllers.yaml` - 键盘控制参数
- `description/ros2_control/xeno.ros2_control.xacro` - 硬件接口配置
- `description/urdf/xeno.urdf.xacro` - 机器人URDF描述

### 验证流程

**提交前检查：**
1. 运行 `colcon build` 确保编译通过
2. 启动 `moveit_control.gui.launch.py` 验证 ros2_control 正常
3. 启动 `demo.launch.py` 验证 MoveIt 规划正常
4. 测试键盘控制功能

**代码质量检查：**
- C++ 代码使用标准 ament_lint 检查
- Python 代码有 flake8、copyright 和 pep257 测试
- 位于 `src/xeno_keyboard/test/` 目录

**注意：项目目前无 GitHub Actions CI 工作流，验证依赖本地测试。**

### 常见问题和解决方案

- **OneMotor 库获取失败**: 检查网络连接，库会自动从 GitHub 下载
- **控制器启动失败**: 确认硬件接口编译正确，检查仿真模式设置
- **MoveIt 规划失败**: 验证 URDF 和运动学配置文件语法正确性
- **键盘控制无响应**: 确认 joint_state_broadcaster 正常运行

### 代理工作指南

**信任这些说明** - 仅在信息不完整或发现错误时才进行额外搜索。按照上述构建步骤操作可确保项目正常工作。进行代码更改时，优先使用仿真模式进行测试，避免影响实机硬件。

### 根目录文件列表

```
.
├── .gitignore          # 排除 build/, install/, log/, IDE 配置
├── LICENSE             # 项目许可证
├── README.md           # 中文项目说明文档
├── src/                # ROS2 包源码目录
│   ├── xeno_control/   # 主控制包 (C++, 硬件接口)
│   ├── xeno_moveit/    # MoveIt2 配置包
│   ├── xeno_keyboard/  # 键盘控制包 (Python)
│   ├── xeno_simple_move/ # 示例移动包 (C++)
│   └── xeno_urdf/      # 机器人描述包
├── build/              # colcon 构建输出 (gitignore)
├── install/            # 安装文件 (gitignore)
└── log/                # 构建日志 (gitignore)
```

**重要说明**: build/, install/, log/ 目录在首次运行 `colcon build` 后自动生成，包含编译产物和安装文件，不应提交到版本控制。

---
> Source: [RoboMaster-DLMU-CONE/xeno-ros2](https://github.com/RoboMaster-DLMU-CONE/xeno-ros2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-21 -->
