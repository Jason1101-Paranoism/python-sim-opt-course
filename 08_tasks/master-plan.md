---
title: Master Build Plan
type: admin
audience: internal
status: draft
updated: 2026-04-02
---

# 課程建立總計劃

## 現況快照

**已完成**：Phase 0（21 個）✅ + Phase 1A（4 個）✅ + Phase 1B（2 個）✅ + Phase 1C（6 個）✅ + Phase 1D（6 個）✅ + Phase 1E（1 個）✅ + Phase 1F（6 個）✅ + Phase 2（4 個）✅ + Phase 3A（40 個）✅ + Phase 3B（40 個）✅ = 130 個
**待完成**：週次教材（Level 3–4）× 80、對外文件 × 10

> 進度追蹤規則：每次執行前後必須更新 `backlog.md` 與本文件。詳見 `CLAUDE.md § Progress Tracking Workflow`。

---

## 總體分期

| 階段 | 名稱 | 產出 | 目的 |
|------|------|------|------|
| Phase 0 | 基礎規格 ✅ | 21 個 md | Claude Code 可以動工的最小集合 |
| Phase 1 | 策略文件補完 ✅ | ~20 個 md | 讓所有政策、流程、行銷說法有依據 |
| Phase 2 | 模板層 ✅ | 4 個 md | 讓教材生成有統一格式可套用 |
| Phase 3A | Level 1 週次教材 ✅ | 40 個 md | 第一級完整可授課 |
| Phase 3B | Level 2 週次教材 ✅ | 40 個 md | 第二級完整可授課 |
| Phase 3C | Level 3 週次教材 | 40 個 md | 第三級完整可授課 |
| Phase 3D | Level 4 週次教材 | 40 個 md | 第四級完整可授課 |
| Phase 4 | 對外文件 | ~15 個 md | 招生用、合作提案用 |

---

## Phase 1：策略文件補完

### 1A — 教學政策（`02_teaching/`）

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `class-structure.md` | 每次 group 課 2 小時的固定結構（暖身/概念/示範/實作/回顧） | ★★★ ✅ |
| `assignment-policy.md` | 每週作業量、遲交規則、段考週調整 | ★★ |
| `ai-usage-policy.md` | 學生可如何使用 AI、哪些行為算違規 | ★★ |
| `academic-integrity-policy.md` | 不代寫、不代做、引用規則 | ★★ |

### 1B — 課綱補充（`01_curriculum/`）

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `progression-rules.md` | 誰可直升、誰要先測驗、邊界案例 | ★★★ ✅ |
| `assessment-rubrics.md` | 各級評分規準（程式/模型/分析/報告/展示） | ★★★ ✅ |

### 1C — 營運政策（`03_operations/`）✅

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `scheduling-policy.md` | 開課時間、排課原則、假日與段考週 | ★★ ✅ |
| `attendance-makeup-policy.md` | 請假、補課、1-on-1 重排、錄影補發 | ★★ ✅ |
| `placement-and-promotion.md` | 課前評估、升級、轉班、降級機制 | ★★ ✅ |
| `refund-transfer-policy.md` | 退費、轉班、停修、保留名額 | ★★ ✅ |
| `parent-communication-policy.md` | 何時回報家長、用什麼形式、發什麼內容 | ★ ✅ |
| `crisis-backup-plan.md` | 你請假、報名不足、實體改線上等應變 | ★ ✅ |

### 1D — 行銷補充（`05_marketing/`）✅

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `faq.md` | Python 程度、作業量、是否影響課業、銜接 AP/IB 等 | ★★★ ✅ |
| `product-pages.md` | 每一級課程頁區塊（對象/收穫/內容/成果/報名條件） | ★★ ✅ |
| `parent-personas.md` | 三類家長的痛點與購買理由 | ★★ ✅ |
| `cram-school-pitch.md` | 補習班合作提案大綱與話術 | ★★ ✅ |
| `school-partner-pitch.md` | 學校合作提案大綱 | ★ ✅ |
| `social-copy-bank.md` | 招生貼文、DM 標題、EDM 標題 | ★ ✅ |

### 1E — 行銷商業補充（`04_business/`）

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `sales-objections.md` | 家長與補習班最常見反對意見與標準回覆 | ★★ ✅ |

### 1F — 行政補充（`06_admin/`）✅

| 檔案 | 內容摘要 | 優先度 |
|------|----------|--------|
| `intake-form-spec.md` | 報名表欄位規格 | ★★ ✅ |
| `placement-test-spec.md` | 程度測驗題型、評分、分流標準 | ★★ ✅ |
| `onboarding-checklist.md` | 學生錄取後的完成清單（環境安裝、同意書等） | ★★ ✅ |
| `student-progress-report-spec.md` | 期中／期末回饋報告格式 | ★ ✅ |
| `completion-certificate-spec.md` | 結業證書格式與欄位 | ★ ✅ |
| `consent-and-ip-policy.md` | 作品使用同意、隱私與宣傳授權 | ★ ✅ |

---

## Phase 2：模板層（`07_assets/templates/`）

| 檔案 | 用途 | 依賴 |
|------|------|------|
| `proposal-template.md` | Level 2 專題計畫書 | level-2.md |
| `report-template.md` | Level 2–4 報告架構 | assessment-rubrics.md |
| `literature-matrix-template.md` | Level 4 文獻分析矩陣 | level-4.md |
| `slide-template.md` | 各級簡報版型說明 | curriculum-map.md |

---

## Phase 3：週次教材（每週 4 份文件）

每週產出：
1. **教師教案**（Teacher Notes）：課堂時間分配、常見問題、程式碼範例
2. **學生講義**（Student Worksheet）：概念說明、範例程式、練習題
3. **作業單**（Assignment Brief）：作業目標、繳交規格、自我檢查清單
4. **1-on-1 討論指南**（1-on-1 Guide）：本週個別討論問題、里程碑

### Phase 3A — Level 1（10 週 × 4 份 = 40 份）

輸出路徑：`09_outputs/teaching-materials/level-1/`

| 週 | 主題 | 狀態 |
|----|------|------|
| 01 | 什麼是模擬？骰子實驗 | 待建 |
| 02 | 機率遊戲：彩票與生日悖論 | 待建 |
| 03 | 排隊系統：等待時間模擬 | 待建 |
| 04 | 成長與衰退：差分模型 | 待建 |
| 05 | 小專題 1：個人 mini project | 待建 |
| 06 | 視覺化：用圖說故事 | 待建 |
| 07 | 參數與敏感度 | 待建 |
| 08 | 小組專題工作坊 | 待建 |
| 09 | 小報告寫作練習 | 待建 |
| 10 | 展示與反思 | 待建 |

### Phase 3B — Level 2（10 週 × 4 份 = 40 份）✅

輸出路徑：`09_outputs/teaching-materials/level-2/`

| 週 | 主題 | 狀態 |
|----|------|------|
| 01 | 課綱與範例專題拆解 | 完成 ✅ |
| 02 | 模型與模擬共通示範 | 完成 ✅ |
| 03 | 題目收斂與 RQ 寫法 | 完成 ✅ |
| 04 | 專題計畫書（Proposal） | 完成 ✅ |
| 05 | 程式架構與初版模型 | 完成 ✅ |
| 06 | 中期分享 | 完成 ✅ |
| 07 | 參數掃描與（簡單）最佳化 | 完成 ✅ |
| 08 | 報告架構與寫作 | 完成 ✅ |
| 09 | 簡報與口頭問答 | 完成 ✅ |
| 10 | 期末展示與收尾 | 完成 ✅ |

### Phase 3C — Level 3（10 週 × 4 份 = 40 份）

輸出路徑：`09_outputs/teaching-materials/level-3/`

| 週 | 主題 | 狀態 |
|----|------|------|
| 01 | 目標與舊專題回顧 | 待建 |
| 02 | NumPy：陣列與向量化 | 待建 |
| 03 | SciPy：基本數值方法 | 待建 |
| 04 | SciPy：簡單 ODE 模型 | 待建 |
| 05 | 最佳化入門 | 待建 |
| 06 | 高級專題選題與方法設計 | 待建 |
| 07 | 專題實作：模型與數值方法 | 待建 |
| 08 | 專題實作：最佳化與敏感度分析 | 待建 |
| 09 | 技術寫作與簡報 | 待建 |
| 10 | 作品集展示與整理 | 待建 |

### Phase 3D — Level 4（10 週 × 4 份 = 40 份）

輸出路徑：`09_outputs/teaching-materials/level-4/`

| 週 | 主題 | 狀態 |
|----|------|------|
| 01 | 研究型專題 vs 一般 project | 待建 |
| 02 | 文獻搜尋策略 | 待建 |
| 03 | 文獻閱讀與分析矩陣 | 待建 |
| 04 | 文獻回顧寫作 | 待建 |
| 05 | 研究問題與 gap 精煉 | 待建 |
| 06 | 方法章設計與寫作 | 待建 |
| 07 | 實驗與結果呈現（研究版） | 待建 |
| 08 | 討論、限制與未來工作 | 待建 |
| 09 | 全文整合與學術寫作細節 | 待建 |
| 10 | 研究簡報與口試模擬 | 待建 |

---

## Phase 4：對外文件（`09_outputs/`）

### 招生文件（`09_outputs/marketing/`）
- [ ] 系列總覽一頁紙
- [ ] Level 1–4 各一份課程簡章（共 4 份）
- [ ] FAQ（對家長版）

### 合作提案文件（`09_outputs/partner-docs/`）
- [ ] 補習班合作 Pitch Deck（中文）
- [ ] 學校合作提案（中文）
- [ ] 定價 + 分潤試算表（含人數 × 折扣矩陣）
- [ ] 國際版 Upwork/LinkedIn 課程描述（英文）

---

## 建議執行順序

```
Phase 0 ✅
    ↓
Phase 1A + 1B（class-structure、rubrics、progression-rules）← 教材生成的前提
    ↓
Phase 2（模板）← 週次教材的格式基礎
    ↓
Phase 3A（Level 1 週次教材）← 最先開課，優先完成
    ↓
Phase 1C + 1D + 1E + 1F（政策/行銷/行政）← 可平行推進
    ↓
Phase 3B → 3C → 3D（按開課順序）
    ↓
Phase 4（對外文件）← 最後整合輸出
```

---

## 工作量估算

| 階段 | 檔案數 | 建議方式 |
|------|--------|----------|
| Phase 1 | ~20 | Claude Code 逐份生成，每份有明確規格 |
| Phase 2 | 4 | Claude Code 生成，需參照 rubrics |
| Phase 3（全部） | 160 | Claude Code 批次生成，按 `weekly-lesson-template.md` |
| Phase 4 | ~15 | Claude Code 生成，需參照 messaging-house |
| **合計** | **~200** | |

---

## Phase 0 完成清單

✅ `CLAUDE.md`
✅ `00_project/project-overview.md`
✅ `00_project/brand-positioning.md`
✅ `00_project/target-audience.md`
✅ `00_project/glossary.md`
✅ `01_curriculum/curriculum-map.md`
✅ `01_curriculum/level-1.md`
✅ `01_curriculum/level-2.md`
✅ `01_curriculum/level-3.md`
✅ `01_curriculum/level-4.md`
✅ `02_teaching/teaching-principles.md`
✅ `02_teaching/one-on-one-framework.md`
✅ `04_business/pricing.md`
✅ `04_business/discounts.md`
✅ `04_business/revenue-share.md`
✅ `04_business/partner-model.md`
✅ `05_marketing/messaging-house.md`
✅ `06_admin/preparation-checklist.md`
✅ `07_assets/templates/weekly-lesson-template.md`
✅ `08_tasks/backlog.md`
✅ `08_tasks/master-plan.md`

---

## 下一步建議

立即執行（Phase 1A + 1B，教材生成前提）：
1. `02_teaching/class-structure.md`
2. `01_curriculum/assessment-rubrics.md`
3. `01_curriculum/progression-rules.md`

完成這三份後，即可請 Claude Code 批次生成 Level 1 全部 10 週教材。
