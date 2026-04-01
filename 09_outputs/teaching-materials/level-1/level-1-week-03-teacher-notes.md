---
title: Level 1 週 3 教師教案 — 排隊系統：等待時間模擬
type: teaching
level: "1"
week: 3
audience: teacher
status: draft
---

# Level 1 週 3：排隊系統 — 等待時間模擬
## 教師教案（Teacher Notes）

---

### 本週學習目標

1. 學生能說明排隊系統的三個要素（到達、服務、等待）
2. 學生能寫出單櫃台排隊模擬程式
3. 學生能用程式輸出平均等待時間，並用圖觀察分佈

---

### 課堂時間分配（120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 回顧上週機率遊戲，引入「等待」的現實情境 |
| 概念講解 | 25 分鐘 | 排隊系統三要素 + M/M/1 模型直覺 |
| 範例示範 | 30 分鐘 | Live coding：單櫃台排隊模擬 |
| 學生實作 | 35 分鐘 | 學生修改參數、觀察等待時間變化 |
| 回顧與預告 | 20 分鐘 | 總結、作業說明、預告差分模型 |

---

### 概念講解（25 分鐘）

**引入（5 分鐘）**：「你去便利商店或診所，有沒有等很久的經驗？為什麼有時候快、有時候慢？」

**三要素（15 分鐘）**：

| 要素 | 說明 | 在程式裡怎麼表示 |
|------|------|-----------------|
| 到達（Arrival） | 顧客多久來一個 | 用指數分配的隨機時間間隔 |
| 服務（Service） | 服務一個人要多久 | 也用指數分配的隨機時間 |
| 等待（Waiting） | 排在前面的人沒服務完，就要等 | 用迴圈模擬時間推移 |

> 指數分配：`random.expovariate(rate)` → 平均值 = 1/rate
> 例：平均每 3 分鐘來一個人 → `random.expovariate(1/3)`

**小問答（5 分鐘）**：「如果服務速度和到達速度一樣快，會發生什麼？」→ 佇列可能無限增長

---

### 範例示範（30 分鐘）

```python
import random
import matplotlib.pyplot as plt

def simulate_queue(n_customers, arrival_rate, service_rate):
    """
    模擬單櫃台排隊系統
    arrival_rate: 每分鐘平均到達人數
    service_rate: 每分鐘平均服務人數
    回傳：每位顧客的等待時間列表
    """
    wait_times = []
    current_time = 0       # 目前時刻
    server_free_at = 0     # 服務員何時空閒

    for _ in range(n_customers):
        # 這個顧客什麼時候到達
        inter_arrival = random.expovariate(arrival_rate)
        current_time += inter_arrival

        # 等待時間 = max(0, 服務員完成時間 - 顧客到達時間)
        wait = max(0, server_free_at - current_time)
        wait_times.append(wait)

        # 服務時間
        service_time = random.expovariate(service_rate)
        server_free_at = max(current_time, server_free_at) + service_time

    return wait_times

# 執行模擬
wait_times = simulate_queue(
    n_customers=200,
    arrival_rate=1/3,   # 每 3 分鐘來一個人
    service_rate=1/2    # 每 2 分鐘服務一個人
)

print(f"平均等待時間：{sum(wait_times)/len(wait_times):.2f} 分鐘")
print(f"最長等待時間：{max(wait_times):.2f} 分鐘")

# 視覺化
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
plt.title("各顧客等待時間（時間序列）")

plt.tight_layout()
plt.show()
```

**刻意製造問題**：把 `arrival_rate=1/2`、`service_rate=1/3`（到達比服務快），讓學生看到等待時間爆炸。

---

### 學生實作（35 分鐘）

**任務**：
1. 跑出基本版模擬，輸出平均等待時間和直方圖
2. 嘗試讓到達率 ≥ 服務率，觀察結果
3. 試著改成「模擬 10 次取平均」，讓結果更穩定

**對進度快的學生**：「你能讓程式畫出『服務率從慢到快』vs 平均等待時間的折線圖嗎？」

---

### 本週 3 個關鍵點

1. 排隊系統有三個要素：到達、服務、等待
2. 當到達速度 ≥ 服務速度，佇列會無限增長（系統不穩定）
3. 指數分配是描述「隨機等待時間」的常用工具
