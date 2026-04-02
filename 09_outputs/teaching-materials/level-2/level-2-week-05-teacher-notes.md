---
title: Level 2 週 05 — 程式架構與初版模型（教師教案）
type: teaching
level: 2
week: 5
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-2.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
---

# Level 2 週 05：程式架構與初版模型

## 本週學習目標

1. 學生能把專題的模型拆成三段：simulate_once / 實驗迴圈 / 圖表輸出
2. 學生寫出一個**可以執行**的初版 simulate_once（MVP 版本）
3. 學生產出至少一張結果圖，開始觀察初步趨勢

---

## 課堂時間分配（共 120 分鐘）

> 本週是「工作坊週」（參見 class-structure.md §特殊週次），取消概念講解，延長實作時間。

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | Proposal 回饋摘要，確認每人的方向 |
| 架構示範 | 25 分鐘 | Live coding：三段架構的完整示範 |
| 學生實作 | 65 分鐘 | 學生各自寫自己的 simulate_once |
| 回顧與預告 | 20 分鐘 | 展示初版結果 + 預告中期分享 |

---

## 各段落執行細節

### 暖身回顧（10 分鐘）

- 快速確認每人的 Proposal 是否獲得「可行」評估
- 若有學生 RQ 還在修改中，老師當場確認最新方向
- 說明今天的目標：「課程結束前，每個人都有一段可以跑的程式和一張圖」

---

### 架構示範（25 分鐘）

**Live coding：以 SIR 模型為例，展示三段架構**

```python
import numpy as np
import matplotlib.pyplot as plt

# ====== 第一段：simulate_once ======
def simulate_once(beta, gamma, N=1000, I0=10, days=160):
    """
    SIR 模型：一次模擬，回傳感染高峰人數和高峰時間
    
    假設：
    - 均質混合（每個人有相同機率接觸任何人）
    - 康復後不再感染
    
    參數：
    - beta: 接觸率（每天每個感染者能感染的易感者比例）
    - gamma: 康復率（每天康復的感染者比例）
    - N: 總人口
    - I0: 初始感染人數
    - days: 模擬天數
    
    回傳：
    - peak_I: 感染人數的最大值
    - peak_day: 高峰出現的天數
    """
    S, I, R = N - I0, I0, 0
    peak_I, peak_day = I0, 0

    for day in range(days):
        new_infections = beta * S * I / N
        new_recoveries = gamma * I

        S -= new_infections
        I += new_infections - new_recoveries
        R += new_recoveries

        if I > peak_I:
            peak_I = I
            peak_day = day

    return peak_I, peak_day

# ====== 第二段：掃描 beta ======
betas = np.arange(0.1, 0.6, 0.05)
peaks = []
peak_days = []

for b in betas:
    pi, pd = simulate_once(b, gamma=0.1)
    peaks.append(pi)
    peak_days.append(pd)

# ====== 第三段：視覺化 ======
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))

ax1.plot(betas, peaks, marker='o', color='tomato')
ax1.set_xlabel("接觸率 β")
ax1.set_ylabel("感染高峰人數")
ax1.set_title("β vs. 感染高峰人數")
ax1.grid(True)

ax2.plot(betas, peak_days, marker='s', color='steelblue')
ax2.set_xlabel("接觸率 β")
ax2.set_ylabel("高峰出現天數")
ax2.set_title("β vs. 高峰出現時間")
ax2.grid(True)

plt.tight_layout()
plt.show()
```

**示範過程的重點說明：**
- 為什麼 simulate_once 要把假設寫在 docstring？（之後寫報告時，假設已經整理好了）
- 為什麼用 `np.arange` 而不是手打 list？（之後要做大量掃描更方便）
- 為什麼畫兩張圖？（一個 RQ 可以有多個面向的輸出）

**刻意的討論問題：**
- 「這個模型沒有隨機性——跑兩次結果一樣。這樣好還是不好？」
- 「如果我的題目有隨機性，應該在哪裡加？」

---

### 學生實作（65 分鐘）

**目標：每個學生寫出自己的 simulate_once 並跑出一張圖**

**老師的巡視節奏：**
- 前 10 分鐘：讓學生自己開始，不主動打擾
- 10–50 分鐘：巡視，針對卡關的學生給提示（不直接寫 code）
- 50–65 分鐘：鼓勵已完成的學生加上實驗迴圈和圖表

**常見卡點與提示：**

| 卡點 | 提示方向 |
|------|---------|
| 不知道從哪裡開始 | 「先只寫 def simulate_once(...)，把輸入和輸出說清楚，裡面先 pass」 |
| 模型太複雜跑不動 | 「把最複雜的部分先跳過，先讓最簡單的版本跑起來」 |
| 結果看起來不合理 | 「加 print 確認每一步的中間值，找到哪裡開始錯了」 |
| 不會畫圖 | 提示用 `plt.plot(x, y)` 的最基本語法；標題和標籤可以之後加 |

---

### 回顧與預告（20 分鐘）

**每位學生展示：**
- 你的 simulate_once 輸入什麼、輸出什麼（1 句）
- 你的圖顯示了什麼趨勢（1 句）

**本週 3 個關鍵點：**
1. 三段架構：simulate_once → 迴圈 → 圖表
2. MVP 優先：先讓程式跑起來，再加複雜度
3. 程式的假設要寫在 docstring——之後寫報告會用到

**預告週 06：**
> 「下週是中期分享。每人 5 分鐘，說你的題目、目前的結果、遇到的問題。不需要完美，需要清楚。」

---

## 常見問題與處理方式

| 常見問題 | 建議回應 |
|---------|---------|
| 「我的模型有隨機性，結果每次不一樣」 | 「跑 50 次取平均，或先把隨機性暫時去掉，確認邏輯正確後再加回去」 |
| 「我今天沒辦法完成 simulate_once」 | 「先確保你有函數定義和一個測試用的呼叫，讓它不 error 就算完成了」 |
| 「圖看起來很奇怪，不知道對不對」 | 「試試用一個你知道答案的特殊值測試，例如 β=0 時，感染應該不會增加」 |
