---
title: Level 1 週 3 學生講義 — 排隊系統
type: teaching
level: "1"
week: 3
audience: student
status: draft
---

# Level 1 週 3：排隊系統 — 等待時間模擬
## 學生講義（Student Worksheet）

---

## 核心概念

### 排隊系統的三要素

| 要素 | 現實意思 | 程式裡的表示 |
|------|----------|-------------|
| **到達（Arrival）** | 顧客多久來一個 | `random.expovariate(arrival_rate)` |
| **服務（Service）** | 服務一個人要多久 | `random.expovariate(service_rate)` |
| **等待（Waiting）** | 排在前面的人沒服務完就等 | `max(0, server_free_at - current_time)` |

### 指數分配是什麼？

```python
import random

# 平均每 3 分鐘來一個顧客
# 但實際間隔是隨機的（有時 1 分鐘，有時 6 分鐘）
inter_arrival = random.expovariate(1/3)
```

> `expovariate(rate)` 產生的值，平均等於 `1/rate`。
> 用在排隊系統是因為現實中，到達和服務時間通常不是固定的。

### 什麼時候佇列會爆炸？

- 到達速度 < 服務速度 → 系統穩定，等待時間有限
- 到達速度 ≥ 服務速度 → 佇列無限增長，等待時間越來越長

---

## 完整程式碼

```python
import random
import matplotlib.pyplot as plt

def simulate_queue(n_customers, arrival_rate, service_rate):
    """
    模擬單櫃台排隊系統
    arrival_rate: 每分鐘平均到達人數（例：1/3 表示平均每 3 分鐘一個人）
    service_rate: 每分鐘平均服務人數（例：1/2 表示平均每 2 分鐘服務完）
    """
    wait_times = []
    current_time = 0
    server_free_at = 0

    for _ in range(n_customers):
        # 這位顧客到達
        inter_arrival = random.expovariate(arrival_rate)
        current_time += inter_arrival

        # 計算等待時間
        wait = max(0, server_free_at - current_time)
        wait_times.append(wait)

        # 服務完成時間
        service_time = random.expovariate(service_rate)
        server_free_at = max(current_time, server_free_at) + service_time

    return wait_times

# 執行
wait_times = simulate_queue(
    n_customers=200,
    arrival_rate=1/3,   # 每 3 分鐘一個人到達
    service_rate=1/2    # 每 2 分鐘服務完一個人
)

# 統計
avg_wait = sum(wait_times) / len(wait_times)
print(f"平均等待時間：{avg_wait:.2f} 分鐘")
print(f"最長等待時間：{max(wait_times):.2f} 分鐘")
print(f"沒有等待的人數：{wait_times.count(0)}/{len(wait_times)}")

# 圖表
plt.figure(figsize=(10, 4))

plt.subplot(1, 2, 1)
plt.hist(wait_times, bins=20, edgecolor='black')
plt.xlabel("等待時間（分鐘）")
plt.ylabel("人數")
plt.title("等待時間分佈")

plt.subplot(1, 2, 2)
plt.plot(wait_times)
plt.xlabel("顧客編號")
plt.ylabel("等待時間（分鐘）")
plt.title("各顧客等待時間")

plt.tight_layout()
plt.show()
```

---

## 試試看

### 練習 1：讓佇列爆炸
把 `arrival_rate=1/2`、`service_rate=1/3`（到達比服務快），觀察發生什麼事。

**你看到了什麼？**
_______________________________________________

### 練習 2：找到臨界點
讓 `arrival_rate` 和 `service_rate` 慢慢靠近，找出「等待時間開始急遽上升」的轉折點大概在哪裡。

### 練習 3（挑戰）：掃描服務率
寫一個迴圈，讓 `service_rate` 從 `1/10` 變化到 `1/1`，記錄每個服務率對應的平均等待時間，畫成折線圖。

```python
service_rates = [1/r for r in range(10, 0, -1)]
avg_waits = []

for rate in service_rates:
    wt = simulate_queue(200, arrival_rate=1/3, service_rate=rate)
    avg_waits.append(sum(wt)/len(wt))

# 畫折線圖（自己完成）
```
