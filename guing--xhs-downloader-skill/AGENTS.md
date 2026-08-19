# GitHub Copilot - XHS-Downloader Skill

## 技能说明

这是一个用于下载小红书（Xiaohongshu/Red）内容的 GitHub Copilot 技能。

## 触发条件

当用户发送以下内容时自动触发：
- 小红书分享链接（如：`https://www.xiaohongshu.com/discovery/item/xxxxx`）
- `https://xhslink.com/xxxxx` 短链接
- 命令：`@workspace 下载小红书`

## 功能特性

- ✅ 自动检测小红书链接
- 📥 下载图片和视频内容
- 📝 生成详细的作品信息 Markdown 文件
- 💾 记录作品数据到 SQLite 数据库
- 📁 每个作品保存在独立文件夹中

## 使用方法

### 1. 首次设置

运行以下命令进行初始化：

```bash
# 克隆 XHS-Downloader 项目
git clone -b develop https://github.com/JoeanAmier/XHS-Downloader.git

# 进入项目目录
cd XHS-Downloader

# 创建虚拟环境
uv venv

# 安装依赖
uv sync
```

### 2. 下载内容

**方式一：直接发送链接**
```
请帮我下载这个小红书内容：https://www.xiaohongshu.com/discovery/item/xxxxx
```

**方式二：使用 @workspace**
```
@workspace 下载小红书 https://www.xiaohongshu.com/discovery/item/xxxxx
```

**方式三：在终端中运行**
```bash
uv run python download_manager.py "https://www.xiaohongshu.com/discovery/item/xxxxx"
```

## 工作流程

1. 🔍 检测到小红书链接
2. 🔧 检查并初始化环境（如需要）
3. 📥 解析链接并下载内容
4. 💾 保存文件到独立文件夹
5. 📝 生成作品信息 Markdown 文件
6. 🗄️ 记录数据到数据库
7. ✅ 返回下载结果和文件路径

## 输出结构

```
XHS-Downloader/Volume/Download/
├── 作品信息.md                    # 所有作品的汇总信息
├── ExploreData.db                 # SQLite 数据库
├── 2023-03-27_作者_标题/          # 单个作品文件夹
│   ├── 2023-03-27_作者_标题_1.jpeg
│   ├── 2023-03-27_作者_标题_2.jpeg
│   └── ...
└── 2026-01-23_作者_视频/          # 视频作品
    └── 2026-01-23_作者_视频.mp4
```

## 作品信息文件内容

每次下载后会在 `作品信息.md` 中追加以下信息：
- 作品标题和作者
- 发布时间
- 作品类型（图文/视频）
- 互动数据（点赞、评论、分享、收藏）
- 作品描述和标签
- 下载链接
- 本地文件列表

## 配置说明

主要配置在 `download_manager.py` 中：

```python
params = {
    "folder_mode": True,      # 每个作品独立文件夹
    "record_data": True,      # 记录到数据库
    "work_path": "./Volume",  # 下载根目录
}
```

## 依赖要求

- Python 3.10+
- Git
- [uv](https://github.com/astral-sh/uv) - Python 包管理器

## 注意事项

⚠️ **重要提醒**：
- 本工具仅供学习和研究使用
- 请遵守相关法律法规和小红书平台使用条款
- 下载内容仅供个人学习使用
- 不得用于商业用途或侵犯他人权益

## 故障排除

### 问题：项目未初始化
**解决方案**：运行初始化命令
```bash
cd XHS-Downloader && uv venv && uv sync
```

### 问题：下载失败
**解决方案**：
1. 检查链接是否有效
2. 更新项目：`cd XHS-Downloader && git pull origin develop`
3. 重新同步依赖：`uv sync`

### 问题：找不到下载文件
**解决方案**：
- 检查下载目录：`XHS-Downloader/Volume/Download`
- 查看 `作品信息.md` 确认文件位置

## 更多信息

- 项目地址：https://github.com/JoeanAmier/XHS-Downloader
- 问题反馈：通过 GitHub Issues 提交

---

**提示**：GitHub Copilot 检测到小红书链接时会自动建议使用此技能下载内容。

---
> Source: [Guing/xhs-downloader-skill](https://github.com/Guing/xhs-downloader-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-19 -->
