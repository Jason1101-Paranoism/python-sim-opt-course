---
title: Tech Stack & Delivery Architecture
type: strategy
audience: internal
status: draft
last_updated: 2026-04-01
---

# 技術架構與呈現規劃

## 設計原則

- 補習班合作方與家長：**不需要帳號、開連結就能看**
- 學生上課：**熟悉的工具、上傳作業不需要學新東西**
- 老師（自己）：**一個地方管所有檔案，不重複維護**

---

## 三層架構

```
┌─────────────────────────────────────────┐
│  Layer 1：GitHub（版控主倉庫）           │
│  所有 .md 教材的唯一真相來源             │
│  Claude Code 的工作區                   │
└──────────────┬──────────────────────────┘
               │ 自動部署（GitHub Actions）
               ▼
┌─────────────────────────────────────────┐
│  Layer 2：MkDocs + GitHub Pages         │
│  對外展示網站（補習班、家長、招生用）     │
│  課程介紹 / 定價 / 路徑圖 / 教師簡介    │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  Layer 3：Google Classroom              │
│  每班一個，課程管理 + 學生交作業         │
│  老師批改 / 家長觀察 / 進度追蹤          │
└─────────────────────────────────────────┘
```

---

## 各層說明

### Layer 1：GitHub（版控主倉庫）

**用途**：所有 `.md` 課程檔案的版本控制與儲存

**規格**：
- 私有 repo（`python-sim-opt-course`）
- 每次更新教材就 commit
- 不需要對外公開

**誰用**：老師本人 + Claude Code

---

### Layer 2：MkDocs + GitHub Pages（對外展示）

**用途**：給補習班合作方、家長、潛在學生看的課程網站

**包含內容**：
- 課程四級路徑圖
- 每一級的課程介紹頁（從 `05_marketing/product-pages.md` 生成）
- 定價與優惠說明（從 `04_business/pricing.md` 生成）
- 合作模式簡介（從 `04_business/partner-model.md` 生成）
- FAQ（從 `05_marketing/faq.md` 生成）
- 教師簡介

**技術規格**：
- 工具：[MkDocs Material](https://squidfunk.github.io/mkdocs-material/)（最美觀的 MkDocs 主題）
- 部署：GitHub Pages（免費）
- 自動部署：GitHub Actions（每次 push 自動更新網站）
- 設定檔：`mkdocs.yml`（放在 repo 根目錄）

**網址格式**：`https://[你的GitHub帳號].github.io/python-sim-opt-course/`

**補習班看到的體驗**：
> 收到一個連結 → 直接在瀏覽器打開 → 看到專業課程介紹 → 不需要登入

---

### Layer 3：Google Classroom（每班課程管理）

**用途**：實際開課後的課程管理、作業發布、學生繳交、老師回饋

**每一個 Google Classroom 對應**：一個班級（例：Level 1 第一期）

**功能對應**：

| 需求 | Google Classroom 怎麼做 |
|------|------------------------|
| 發布每週課程說明 | 發布「作業」，貼上 assignment-brief 內容 |
| 學生繳交 `.py` 檔案 | 直接在 Classroom 附件上傳 |
| 學生繳交截圖 | 上傳圖片或 Google Doc |
| 老師留回饋 | 在學生作業上直接留言或評分 |
| 家長追蹤進度 | 加入「監護人」角色，自動收 email 摘要 |
| 宣布事項（段考週、補課） | 「串流」發布公告 |
| 1-on-1 預約 | 發 Google Meet 連結或 Google Calendar 邀請 |

**費用**：免費（學生用 Gmail 帳號即可加入）

---

## 執行步驟

### 步驟 1：初始化 Git Repo（15 分鐘）

```bash
cd /c/Users/LL/Documents/course/python-sim-opt-course
git init
git add .
git commit -m "Initial commit: course materials Phase 0–3A"
```

建立 `.gitignore`：
```
__pycache__/
*.pyc
.DS_Store
site/          # MkDocs 本地 build 輸出，不需要 commit
```

推上 GitHub（建立新 repo 後）：
```bash
git remote add origin https://github.com/[帳號]/python-sim-opt-course.git
git push -u origin main
```

---

### 步驟 2：安裝 MkDocs 並建立設定（30 分鐘）

```bash
pip install mkdocs-material
```

建立 `mkdocs.yml`（根目錄）：
```yaml
site_name: Python Simulation & Optimization Pathway
site_description: 從模擬入門到 Research Capstone 的完整學習路徑
site_author: [你的名字]

theme:
  name: material
  language: zh-TW
  palette:
    primary: indigo
    accent: blue
  features:
    - navigation.tabs
    - navigation.sections
    - search.suggest

nav:
  - 課程總覽: index.md
  - 課程介紹:
    - Level 1：模擬入門: courses/level-1.md
    - Level 2：AP/IB 專題: courses/level-2.md
    - Level 3：數值方法與優化: courses/level-3.md
    - Level 4：Research Capstone: courses/level-4.md
  - 定價與優惠: pricing.md
  - 合作方案: partner.md
  - 常見問題: faq.md

docs_dir: docs/          # 對外網站的 md 放這裡（從 05_marketing 生成）
```

本地預覽：
```bash
mkdocs serve
# 開瀏覽器看 http://localhost:8000
```

---

### 步驟 3：設定 GitHub Actions 自動部署（15 分鐘）

建立 `.github/workflows/deploy.yml`：
```yaml
name: Deploy MkDocs
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

每次 `git push`，網站自動更新。

---

### 步驟 4：建立第一個 Google Classroom（20 分鐘）

1. 前往 classroom.google.com
2. 建立課程：「Level 1 模擬入門 — 第一期」
3. 設定班級代碼（給學生加入用）
4. 匯入第一週作業（從 `level-1-week-01-assignment-brief.md` 複製）
5. 建立「監護人邀請」選項（讓家長加入）

---

## 各工具的維護分工

| 工作 | 在哪裡做 |
|------|---------|
| 新增或修改教材 | GitHub（用 Claude Code） |
| 更新對外網站 | 自動（push 即更新） |
| 發布每週作業 | Google Classroom |
| 看學生繳交狀況 | Google Classroom |
| 給學生個別回饋 | Google Classroom |
| 對外招生連結 | 分享 GitHub Pages 網址 |
| 補習班合作提案 | 分享 GitHub Pages 網址 |

---

## 未來可選擴充

| 需求 | 工具 |
|------|------|
| 線上收費 / 報名表 | Google Forms + 綠界或 stripe |
| 更精緻的課程網站 | 升級為 Docusaurus 或自訂 HTML/CSS |
| 學生作品集展示 | GitHub Pages 子頁面 |
| 多老師協作 | GitHub Org + Google Classroom 共同教師 |
| 行銷 email | Mailchimp（免費方案到 1000 人） |
