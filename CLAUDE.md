# CLAUDE.md

本專案用於建立「Python Simulation & Optimization Research Pathway」的完整開課系統。
Claude Code 的任務是依據既有規格文件，自動生成教材、招生文案、行政文件、合作提案與對外內容。
不得自行改動核心商業規格，除非使用者明確要求。

---

## Source of Truth Priority

1. `04_business/*.md` 為商業規格真相來源（定價、優惠、分潤不可自行改動）
2. `01_curriculum/*.md` 為課程內容真相來源
3. `02_teaching/*.md` 為教學方法與教案規格真相來源
4. `03_operations/*.md` 與 `06_admin/*.md` 為行政流程真相來源
5. `05_marketing/*.md` 為對外訊息真相來源

若不同檔案衝突：
- 先回報衝突
- 不得自行推翻高優先級文件

---

## File Naming Rules

- 所有檔名一律使用 `lowercase-hyphen-case`
- level 檔案固定命名：`level-1.md` / `level-2.md` / `level-3.md` / `level-4.md`
- 產出檔名需包含用途與 level，例如：
  - `level-1-week-01-slides.md`
  - `level-2-week-03-proposal-worksheet.md`
  - `level-4-literature-review-rubric.md`
- 不可建立意義不明的檔名，如 `final.md`, `new.md`, `temp.md`, `notes2.md`

---

## Output Rules

生成內容時，必須區分以下文件類型：
- `strategy`：策略與規範文件
- `template`：可重複使用模板
- `teaching`：教學內容
- `admin`：行政文件
- `marketing`：行銷文案
- `partner`：合作提案

每份輸出文件開頭需包含 YAML frontmatter：

```yaml
---
title:
type:
level:
audience:
status:
source_files:
---
```

`status` 僅可為 `draft` / `ready` / `approved`

---

## Locked Business Rules

- 每個 level 為 **10 週**
- 每週固定為 group **2 小時** + 1-on-1 **1 小時**
- 每位學生總時數為 **30 小時**
- 建議班級人數為 **4–8 人**
- Level 1–3 建議定價為 **30,000 TWD**
- Level 4 建議定價為 **34,000 TWD**
- 優惠規則以 `discounts.md` 為準
- 分潤模式以 `revenue-share.md` 為準
- 核心定位不是 APCS 語法班，而是 **project / research pathway**

---

## Writing Style Rules

- 對家長：清楚、可信、避免過度學術術語
- 對學生：具挑戰感，但不浮誇
- 對學校／補習班合作方：強調結構化、可招生、可續班、可形成產品線
- 對教材：具體、可執行、避免空泛形容詞
- 優先使用繁體中文；必要時保留英文術語並加中文說明

---

## Material Generation Workflow

在生成某一 level 教材前，必須依序讀取：
1. `01_curriculum/` 對應 level 文件
2. `02_teaching/teaching-principles.md`
3. `02_teaching/class-structure.md`
4. `02_teaching/one-on-one-framework.md`
5. `07_assets/templates/weekly-lesson-template.md`

生成時必須一次產出：
- weekly teacher notes
- student worksheet
- assignment brief
- 1-on-1 discussion guide
- deliverables checklist

---

## Marketing Workflow

生成招生文案前，必須讀取：
- `00_project/brand-positioning.md`
- `05_marketing/messaging-house.md`
- `04_business/pricing.md`
- `04_business/discounts.md`
- `05_marketing/faq.md`

不得自行創造新的優惠方案或價格。

---

## Admin Workflow

生成行政文件前，必須讀取：
- `03_operations/*.md`
- `06_admin/*.md`

必須確保所有表單欄位、流程與政策互相一致。

---

## Task Execution Rules

- 每次只執行一個 `08_tasks/` 下的任務檔
- 執行前先列出將讀取的 source files
- 執行後輸出：
  - created files
  - updated files
  - unresolved questions
- 若缺資料，不要猜，請建立 `assumptions.md` 或提出問題

---

## Progress Tracking Workflow（必須遵守）

**每次執行任何工作前**，必須依序：
1. 讀取 `08_tasks/backlog.md`，確認此任務存在且狀態為待辦
2. 讀取 `08_tasks/master-plan.md`，確認此任務所屬的 Phase 與執行順序合理
3. 確認所有 source files 已存在（不存在則先回報，不執行）

**每次執行任何工作後**，必須依序：
1. 更新 `08_tasks/backlog.md`：將完成項目從 `- [ ]` 改為 `- [x]`
2. 更新 `08_tasks/master-plan.md`：將對應週次或任務的狀態從「待建」改為「完成 ✅」，並更新「現況快照」的已完成數字
3. 回報執行摘要：已建立的檔案清單、未解決的問題

**標記規則**：
- 待辦：`- [ ] 檔案名稱`
- 完成：`- [x] 檔案名稱 ✅`
- 封鎖中（缺依賴）：`- [ ] 檔案名稱 ⚠️ 依賴：xxx`

**不得跳過 Progress Tracking**：即使是單一小檔案的任務，執行後也必須更新兩份追蹤文件。

---

## Do Not

- 不要自行改動已鎖定的定價、分潤、週數、時數
- 不要把策略文件與最終對外文案混成一份
- 不要覆蓋使用者已核准的檔案，除非明確要求
- 不要生成空泛的教材；每週教材需有具體學習目標、活動、作業與產出
- 不要省略 1-on-1 指引
