---
title: Level 2 週 02 — 模型與模擬共通示範（學生講義）
type: teaching
level: 2
week: 2
audience: student
status: draft
source_files:
  - 01_curriculum/level-2.md
---

# Level 2 週 02：模型與模擬共通示範

## 一、模型四元素

任何模擬問題，都可以用這四個元素來拆解：

| 元素 | 說明 | 排隊例子 |
|------|------|---------|
| **狀態（State）** | 系統在某個時刻的描述 | 目前隊列有幾人、服務台是否忙碌 |
| **參數（Parameter）** | 可以調整的數值，控制系統行為 | 顧客到達率、每位顧客的服務時間 |
| **隨機輸入（Randomness）** | 不確定性的來源 | 每位顧客到達的間隔時間是隨機的 |
| **輸出指標（Output Metric）** | 我們想量化的結果 | 平均等待時間 |

> 提醒：**輸出指標就是你 RQ 的答案**。設計模型之前，先想清楚你要觀察什麼。

---

## 二、三段程式架構

一個完整的模擬程式，通常分成三段：

```
第一段：simulate_once(parameters)
    → 把你的模型用程式寫出來，跑一次模擬，回傳輸出指標

第二段：多次模擬 / 參數掃描
    → 因為結果有隨機性，跑 N 次取平均
    → 或改變某個參數，觀察輸出怎麼變

第三段：視覺化
    → 畫圖，把結果說清楚
```

**為什麼要跑多次？**
模擬有隨機性——跑一次的結果可能「剛好」好或「剛好」壞。
跑 50 次或 100 次取平均，才能看到「典型的」行為。

---

## 三、排隊模擬完整範例

```python
import random
import matplotlib.pyplot as plt

# --- 第一段：simulate_once ---
def simulate_once(arrival_rate, service_time, n_customers=200):
    """
    模擬一次排隊過程。
    
    假設：
    - 只有一個服務台
    - 顧客先到先服務（FIFO）
    - 服務時間固定（不隨機）
    
    參數：
    - arrival_rate: 每分鐘平均到達幾位顧客
    - service_time: 每位顧客的服務時間（分鐘）
    
    回傳：
    - 平均等待時間（分鐘）
    """
    wait_times = []
    server_free_at = 0.0  # 服務台何時空閒

    current_time = 0.0
    for _ in range(n_customers):
        # 指數分布：模擬 Poisson 到達過程
        inter_arrival = random.expovariate(arrival_rate)
        current_time += inter_arrival

        # 顧客要等到服務台空閒才能開始
        start = max(current_time, server_free_at)
        wait_times.append(start - current_time)
        server_free_at = start + service_time

    return sum(wait_times) / len(wait_times)

# --- 第二段：掃描參數 ---
rates = [0.5, 0.7, 0.9, 1.1, 1.3, 1.5]
service_time = 1.0
n_runs = 50  # 每個參數值跑 50 次取平均

avg_waits = []
for rate in rates:
    runs = [simulate_once(rate, service_time) for _ in range(n_runs)]
    avg_waits.append(sum(runs) / len(runs))

# --- 第三段：視覺化 ---
plt.plot(rates, avg_waits, marker='o', color='steelblue')
plt.xlabel("顧客到達率（人/分鐘）")
plt.ylabel("平均等待時間（分鐘）")
plt.title("到達率對平均等待時間的影響（服務時間 = 1 分鐘）")
plt.grid(True)
plt.tight_layout()
plt.show()
```

**讀懂這段程式：**
- `simulate_once` 的輸入是什麼？回傳什麼？
- 為什麼 `arrival_rate = 1.5` 的等待時間比 `0.5` 高很多？
- 如果 `service_time` 變成 `2.0`，結果會怎樣？

---

## 四、試試看

### 練習 1：修改參數，觀察結果

把上面的程式複製到 Jupyter Notebook，試著改動以下內容，記錄你的觀察：

| 改動 | 你預測結果會怎麼變 | 實際結果 |
|------|------------------|---------|
| `service_time = 2.0`（服務變慢）| | |
| `n_runs = 5`（只跑 5 次）| | |
| `n_customers = 20`（只有 20 個顧客）| | |

### 練習 2：把你自己的題目填進模型四元素表

| 元素 | 我的題目是什麼 |
|------|--------------|
| 狀態 | |
| 參數 | |
| 隨機輸入 | |
| 輸出指標 | |

### 練習 3：用一句話描述你的 `simulate_once`

> 我的 `simulate_once` 函數，輸入 **___________**，模擬 **___________**，回傳 **___________**。
