---
title: Level 2 週 02 — 模型與模擬共通示範（教師教案）
type: teaching
level: 2
week: 2
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-2.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
---

# Level 2 週 02：模型與模擬共通示範

## 本週學習目標

1. 學生能用「狀態、參數、隨機輸入、輸出指標」四個元素拆解任何模擬問題
2. 學生看懂一個完整的模擬程式三段架構（simulate_once → 實驗迴圈 → 圖表輸出）
3. 學生能把自己的題目方向畫成「模型草圖」（不寫程式）

---

## 課堂時間分配（共 120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 回顧作業：每人說一個題目方向 |
| 概念講解 | 25 分鐘 | 模型四元素 + 三段架構 |
| 範例示範 | 30 分鐘 | Live coding：完整排隊模擬（問題→程式→圖） |
| 學生實作 | 35 分鐘 | 手繪自己題目的模型草圖 |
| 回顧與預告 | 20 分鐘 | 總結 + 作業說明 + 預告 RQ 收斂 |

---

## 各段落執行細節

### 暖身回顧（10 分鐘）

請每位學生用 **1 句話** 說出他們上週帶來的最有興趣的題目方向：
- 不評論好壞，只是讓大家互相聽到
- 老師快速記錄在白板（主題分群：物理/生態/社會/遊戲/其他）

引入問題：
> 「你的題目裡，有哪些東西是『會變的』？哪些是你想觀察的結果？」
> 這個問題不需要答案，只是在熱身——今天就要學怎麼回答這個問題。

---

### 概念講解（25 分鐘）

**模型四元素：**

| 元素 | 說明 | 排隊範例 | SIR 範例 |
|------|------|---------|---------|
| **狀態（State）** | 系統在某時刻的描述 | 等待隊列長度、服務台是否忙碌 | S/I/R 各有多少人 |
| **參數（Parameter）** | 控制系統行為的數值（可調整）| 到達率、服務時間 | 接觸率 β、康復率 γ |
| **隨機輸入（Randomness）** | 不確定性的來源 | 每位顧客到達的間隔時間 | 每次接觸是否感染 |
| **輸出指標（Output）** | 我們想量化的結果 | 平均等待時間 | 感染高峰時的人數 |

板書強調：**輸出指標 = 你的 RQ 的答案**

**三段程式架構：**

```
[函數] simulate_once(parameters)
    → 跑一次模擬，回傳輸出指標

[迴圈] 多次模擬 / 改變參數
    → 重複 N 次求平均，或掃描參數範圍

[視覺化] 畫圖
    → 把結果說清楚
```

強調：為什麼要跑多次？
- 一次模擬結果有隨機性，不穩定
- 多次平均才能反映「典型行為」

---

### 範例示範（30 分鐘）

**Live coding：排隊模擬（從空白開始）**

邊打邊說明思路：

```python
import random
import matplotlib.pyplot as plt

# 第一段：simulate_once
def simulate_once(arrival_rate, service_time, n_customers=200):
    """
    模擬一次排隊過程，回傳平均等待時間
    假設：單一服務台，FIFO，服務時間固定
    """
    wait_times = []
    server_free_at = 0.0  # 服務台何時空閒

    current_time = 0.0
    for _ in range(n_customers):
        # 隨機到達間隔（指數分布模擬 Poisson 到達）
        inter_arrival = random.expovariate(arrival_rate)
        current_time += inter_arrival

        # 等待到服務台空閒
        start = max(current_time, server_free_at)
        wait_times.append(start - current_time)
        server_free_at = start + service_time

    return sum(wait_times) / len(wait_times)

# 第二段：多次模擬 + 掃描參數
rates = [0.5, 0.7, 0.9, 1.1, 1.3, 1.5]
service_time = 1.0
n_runs = 50  # 每個參數跑 50 次取平均

avg_waits = []
for rate in rates:
    runs = [simulate_once(rate, service_time) for _ in range(n_runs)]
    avg_waits.append(sum(runs) / len(runs))

# 第三段：視覺化
plt.plot(rates, avg_waits, marker='o', color='steelblue')
plt.xlabel("顧客到達率（人/分鐘）")
plt.ylabel("平均等待時間（分鐘）")
plt.title("到達率對平均等待時間的影響")
plt.grid(True)
plt.tight_layout()
plt.show()
```

**刻意留一個錯誤讓學生找：**
- 把 `server_free_at = start + service_time` 誤寫成 `server_free_at = current_time + service_time`
- 讓學生觀察結果看起來「哪裡不對」

**示範結束後問：**
- 「這個程式的 simulate_once 裡，假設了什麼？」（服務時間固定、FIFO、只有一台）
- 「如果我想讓程式更真實，最先改什麼？」（服務時間隨機化）

---

### 學生實作（35 分鐘）

**任務：為你的題目畫「模型草圖」**

學生用紙筆（不寫程式）填寫：

```
我的題目：___________________

狀態（系統裡有哪些「東西」在改變）：
→ ___________________

參數（我可以調整哪些數值）：
→ ___________________

隨機輸入（哪些東西有不確定性）：
→ ___________________

輸出指標（我最想觀察的結果是）：
→ ___________________

用一句話說：我的 simulate_once 函數，輸入什麼，回傳什麼？
→ 輸入：_______  回傳：_______
```

老師巡視提示：
- 若學生不確定什麼是「狀態」：「想想看，如果你把模擬暫停在某個時間點，你需要記錄什麼來知道系統當前是什麼情況？」
- 若學生分不清「參數」和「狀態」：「參數是你在開始前設定的，狀態是程式跑的時候會變的。」

---

### 回顧與預告（20 分鐘）

**本週 3 個關鍵點：**
1. 模型四元素：狀態、參數、隨機輸入、輸出指標
2. 三段架構：`simulate_once` → 迴圈 → 圖表
3. `simulate_once` 是你的模型核心，先把它想清楚，再寫其他部分

**預告週 03：**
> 「下週你要把自己的題目方向，轉化成一個正式的研究問題（RQ）。RQ 是你整個專題的基礎——寫不好會影響後面的每一步。我們會學怎麼寫，以及怎麼自我檢查。」

---

## 常見問題與處理方式

| 常見問題 | 建議回應 |
|---------|---------|
| 「我的題目好像沒有『隨機輸入』」 | 「有些模型是確定性的（無隨機），這也沒問題，我們下週再確認是否適合 Level 2 的深度。」 |
| 「simulate_once 要怎麼回傳多個指標？」 | 「可以回傳 tuple 或 dict，但先只做一個輸出指標，確保程式可以跑再加複雜度。」 |
| 「我想做的題目好像很難用 Python 實作」 | 「先說說你想做什麼，我們找找最簡單的近似方式。」 |
