# RAG Learning Assistant — Copilot 專案指令

> Python RAG（Retrieval-Augmented Generation）學習助手專案。

---

## 專案概覽

- **語言**：Python 3.10+
- **框架**：Gradio (Web UI) + LangChain (RAG pipeline)
- **用途**：基於文件的 RAG 問答系統，用於學習輔助

## 專案結構

```
RAG_learning_assistant/
├── app/                    # 主應用程式模組
├── app.py                  # 應用程式入口
├── rebuild_embeddings.py   # 重建向量索引
├── templates/              # 模板檔案
├── Sources/                # 原始文件（受保護，勿修改）
├── evaluate/               # 評估腳本
├── pyproject.toml          # 專案配置 & Ruff 設定
├── enviroment.yml          # Conda 環境配置
├── .pre-commit-config.yaml # Pre-commit hooks 設定
└── .github/
    ├── workflows/lint.yml  # CI/CD 流程
    └── instructions/       # Copilot 技能檔案
```

## 品質保證工具鏈

本專案使用雙層品質把關：

| 層級 | 工具 | 觸發時機 |
|------|------|----------|
| 本地 | pre-commit (Ruff + detect-secrets) | `git commit` 時 |
| 雲端 | GitHub Actions CI | push / PR 到 `main` 時 |

> 📖 詳見 `.github/instructions/` 下的各技能文件。

## 受保護的檔案與目錄

以下檔案 / 目錄**不應直接修改**：

- `Sources/` — 原始學習文件
- `datasets/` — 資料集
- `chunks/` — 分塊後的資料
- `.env` — 環境變數（機密）
- `.secrets.baseline` — detect-secrets 基線
- `package-lock.json` / `*.lock`

## 常用指令

```bash
# 啟動應用
python app.py

# 重建向量索引
python rebuild_embeddings.py

# 程式碼檢查
ruff check .
ruff format --check .

# 執行 pre-commit
pre-commit run --all-files

# 機密偵測
detect-secrets scan --baseline .secrets.baseline
```

## 開發注意事項

1. 所有 Python 程式碼須通過 Ruff lint + format 檢查
2. 提交前確保 `pre-commit run --all-files` 通過
3. 不要將機密（API key、token）寫入程式碼
4. `Sources/` 目錄為原始文件，排除在 lint 範圍外

---
> Source: [Moyuan0523/RAG_learning_assistant](https://github.com/Moyuan0523/RAG_learning_assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-31 -->
