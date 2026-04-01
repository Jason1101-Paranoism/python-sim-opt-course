---
title: Level 1 週 7 教師教案 — 參數與敏感度
type: teaching
level: "1"
week: 7
audience: teacher
status: draft
---

# Level 1 週 7：參數與敏感度
## 教師教案（Teacher Notes）

---

### 本週學習目標

1. 學生能說明「參數」是什麼，以及它和「狀態變數」的差別
2. 學生能設計一個「一次改一個參數」的實驗，並整理結果
3. 學生能畫出「參數 vs 輸出指標」的比較圖

---

### 課堂時間分配（120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 展示上週圖表，討論「如果改一個數字，圖會怎樣？」 |
| 概念講解 | 25 分鐘 | 參數 vs 狀態變數、控制變因的重要性、敏感度直覺 |
| 範例示範 | 30 分鐘 | Live coding：系統性參數掃描 + 結果折線圖 |
| 學生實作 | 35 分鐘 | 學生對自己的專題做參數掃描實驗 |
| 回顧與預告 | 20 分鐘 | 總結、作業說明、預告小組專題 |

---

### 概念講解（25 分鐘）

**參數 vs 狀態變數（10 分鐘）**：

| | 參數（Parameter） | 狀態變數（State Variable） |
|--|-------------------|--------------------------|
| 是什麼 | 模型的設定，你來決定 | 模型執行時會變化的量 |
| 例子 | 感染率 beta、服務率、人口上限 K | 感染人數 I、顧客等待時間、人口數 |
| 在程式裡 | 函數的輸入（參數） | 迴圈裡不斷更新的變數 |

**控制變因的概念（10 分鐘）**：

> 「一次只改一個參數，其他都保持不變。這樣你才能知道是『哪個參數』造成了結果的變化。」

這就是科學實驗的控制變因原則。在模擬裡很容易做到——只需要用迴圈。

**敏感度的直覺（5 分鐘）**：

> 「有些參數改一點點，結果就差很多（高敏感度）；有些改了很多，結果幾乎不變（低敏感度）。找出最敏感的參數，就是最值得關注的設計問題。」

---

### 範例示範（30 分鐘）

**排隊系統的服務率敏感度分析**：

```python
import random
import matplotlib.pyplot as plt

def simulate_queue(n_customers, arrival_rate, service_rate):
    wait_times = []
    current_time = 0
    server_free_at = 0
    for _ in range(n_customers):
        current_time += random.expovariate(arrival_rate)
        wait = max(0, server_free_at - current_time)
        wait_times.append(wait)
        server_free_at = max(current_time, server_free_at) + random.expovariate(service_rate)
    return wait_times

# 參數掃描：固定 arrival_rate，改變 service_rate
arrival_rate = 1/3
service_rates = [1/10, 1/7, 1/5, 1/4, 1/3.5, 1/3.1]  # 從慢到快（接近臨界）
avg_waits = []

for sr in service_rates:
    wt = simulate_queue(300, arrival_rate, sr)
    avg_waits.append(sum(wt) / len(wt))

# 畫結果
avg_service_times = [1/sr for sr in service_rates]   # 轉換成「平均服務時間（分鐘）」

plt.figure(figsize=(8, 5))
plt.plot(avg_service_times, avg_waits, marker='o', color='steelblue', linewidth=2)
plt.xlabel("平均服務時間（分鐘/人）")
plt.ylabel("平均等待時間（分鐘）")
plt.title("服務速度對平均等待時間的影響\n（到達率固定：每 3 分鐘 1 人）")
plt.grid(True, alpha=0.3)
plt.axvline(x=3, color='red', linestyle='--', label='臨界點（服務時間 = 到達間隔）')
plt.legend()
plt.show()

# 列出結果表格
print(f"{'平均服務時間':>12} {'平均等待時間':>12}")
for st, aw in zip(avg_service_times, avg_waits):
    print(f"{st:>12.1f} {aw:>12.2f}")
```

**教學重點**：
- 服務時間接近 3 分鐘（= 到達間隔）時，等待時間急遽上升
- 這就是「高敏感度區域」

---

### 本週 3 個關鍵點

1. 參數是你設定的；狀態是模型自己算出來的
2. 一次只改一個參數（控制變因）
3. 找到「高敏感度」的參數，就是最值得研究的問題
