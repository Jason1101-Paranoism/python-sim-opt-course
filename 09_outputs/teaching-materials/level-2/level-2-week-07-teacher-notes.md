---
title: Level 2 週 07 — 參數掃描與簡單最佳化（教師教案）
type: teaching
level: 2
week: 7
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-2.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
---

# Level 2 週 07：參數掃描與（簡單）最佳化

## 本週學習目標

1. 學生能為自己的專題設計系統性的實驗表（要改哪些參數、範圍、次數）
2. 學生能跑出多組實驗結果並整理成表格與圖
3. 進階學生：用暴力搜尋找出讓輸出指標最大或最小的參數組合

---

## 課堂時間分配（共 120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 中期回饋修改狀況 |
| 概念講解 | 25 分鐘 | 實驗設計：控制變因、掃描策略 |
| 範例示範 | 30 分鐘 | Live coding：單參數掃描 → 雙參數 heatmap → 暴力搜尋最佳 |
| 學生實作 | 35 分鐘 | 學生為自己的專題設計並跑實驗 |
| 回顧與預告 | 20 分鐘 | 結果整理 + 預告報告寫作 |

---

## 各段落執行細節

### 暖身回顧（10 分鐘）

- 「中期後你修改了什麼？有沒有遇到新的問題？」（每人 1–2 句）
- 確認：本週結束後，學生應有「完整的實驗結果」可以進報告

---

### 概念講解（25 分鐘）

**實驗設計三原則：**

1. **每次只改一個參數（控制變因）**
   - 若同時改 β 和 γ，你不知道結果的變化是由哪個造成的
   - 除非你要研究「兩個參數的交互作用」，才做雙參數實驗

2. **參數範圍要有意義**
   - 太小：看不出效果（例如 β 只從 0.10 到 0.11）
   - 太大：超出模型合理範圍（例如 β = 10，感染率超過 100%）
   - 原則：找到「結果明顯改變」的範圍

3. **重複次數：有隨機性就要多跑幾次**
   - 確定性模型（如差分方程）：跑 1 次就夠
   - 隨機模型（如 random.expovariate）：每個參數值至少跑 30–50 次取平均

---

### 範例示範（30 分鐘）

**Live coding：三個層次**

**層次 1：單參數掃描（基礎）**
```python
import numpy as np
import matplotlib.pyplot as plt

# 假設 simulate_once 已定義好
betas = np.linspace(0.1, 0.5, 20)  # 20 個均勻分布的值
results = [simulate_once(b, gamma=0.1)[0] for b in betas]  # 取高峰人數

plt.plot(betas, results, marker='o')
plt.xlabel("接觸率 β")
plt.ylabel("感染高峰人數")
plt.title("單參數掃描：β 對感染高峰的影響")
plt.grid(True)
plt.show()
```

**層次 2：雙參數 heatmap（進階）**
```python
betas = np.linspace(0.1, 0.5, 10)
gammas = np.linspace(0.05, 0.3, 10)

# 建立結果矩陣
peak_matrix = np.zeros((len(gammas), len(betas)))
for i, g in enumerate(gammas):
    for j, b in enumerate(betas):
        peak_matrix[i, j] = simulate_once(b, g)[0]

plt.figure(figsize=(8, 6))
plt.imshow(peak_matrix, origin='lower',
           extent=[betas[0], betas[-1], gammas[0], gammas[-1]],
           aspect='auto', cmap='hot_r')
plt.colorbar(label="感染高峰人數")
plt.xlabel("接觸率 β")
plt.ylabel("康復率 γ")
plt.title("β 與 γ 對感染高峰的交互影響（Heatmap）")
plt.show()
```

**層次 3：暴力搜尋最佳參數（選做）**
```python
# 假設我們要找讓高峰出現最晚的 β（最佳化：延遲高峰）
best_beta = None
latest_peak_day = -1

for b in np.linspace(0.1, 0.5, 50):
    _, peak_day = simulate_once(b, gamma=0.1)
    if peak_day > latest_peak_day:
        latest_peak_day = peak_day
        best_beta = b

print(f"讓高峰最晚出現的 β = {best_beta:.3f}，高峰天數 = {latest_peak_day}")
```

**說明暴力搜尋的限制：**
- 只能在你定義的範圍內找
- 計算量隨參數數量指數增長（兩個參數就是 50×50 = 2500 次）
- Level 3 會學更有效率的最佳化方法

---

### 學生實作（35 分鐘）

**任務：完成你自己專題的完整實驗**

1. 確定要掃描的參數和範圍
2. 跑完所有實驗，整理成圖
3. 進階學生：試著做雙參數掃描或暴力搜尋

**老師巡視提示：**
- 若參數範圍太窄：「把範圍拉大試試，看趨勢是不是更明顯」
- 若圖的趨勢「平坦」（沒有變化）：「這個參數可能對你的輸出影響很小，考慮換一個更敏感的參數」

---

### 回顧與預告（20 分鐘）

**本週 3 個關鍵點：**
1. 每次只改一個參數，否則說不清楚是什麼造成了結果的變化
2. 結果要有圖，圖要有標題和軸標籤
3. 「為什麼是這個趨勢」和「有沒有意外」——這是報告分析段落要說的

**預告週 08：**
> 「下週開始寫報告。報告不是把圖貼上去，而是用文字說清楚『我研究了什麼、怎麼研究、發現了什麼、這代表什麼』。我們會逐段看寫法。」

---

## 常見問題與處理方式

| 問題 | 建議回應 |
|------|---------|
| 「我的結果每次跑都不一樣，怎麼辦？」 | 「每個參數值跑 50 次，取平均值作為該參數的結果。」 |
| 「掃描太多參數，程式跑很久」 | 「先用少量值（5–10 個）做草圖，確認方向正確再加密。」 |
| 「我的參數範圍不知道要設多大」 | 「先從『你認為有意義的』最小和最大值開始，觀察圖的形狀再調整。」 |
