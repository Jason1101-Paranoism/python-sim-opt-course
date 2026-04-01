---
title: Level 1 週 4 教師教案 — 成長與衰退：差分模型
type: teaching
level: "1"
week: 4
audience: teacher
status: draft
---

# Level 1 週 4：成長與衰退 — 差分模型
## 教師教案（Teacher Notes）

---

### 本週學習目標

1. 學生能用「下一步 = 這一步 + 變化量」的思維建立差分模型
2. 學生能寫出人口成長或簡化疫情（SIR 概念）的模擬程式
3. 學生能畫出時間序列圖，並比較不同參數下的曲線

---

### 課堂時間分配（120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 回顧排隊系統，引入「隨時間變化」的想法 |
| 概念講解 | 25 分鐘 | 差分思維 + 人口模型 + 簡化 SIR |
| 範例示範 | 30 分鐘 | Live coding：人口成長（指數/邏輯斯蒂）+ 簡化感染模型 |
| 學生實作 | 35 分鐘 | 選一個做，修改參數觀察曲線變化 |
| 回顧與預告 | 20 分鐘 | 總結、作業說明、預告週 5 個人專題 |

---

### 概念講解（25 分鐘）

**差分思維（10 分鐘）**：

> 「你不需要知道整個函數，只需要知道每一步的變化規則。」

```
下一步的值 = 這一步的值 + 變化量
x[t+1] = x[t] + f(x[t])
```

例子：
- 人口每年增加 2%：`pop[t+1] = pop[t] + 0.02 * pop[t]`
- 銀行存款每月加利息：`balance[t+1] = balance[t] * (1 + r)`

**人口成長兩種版本（10 分鐘）**：

| 模型 | 公式 | 特性 |
|------|------|------|
| 指數成長 | `x[t+1] = x[t] * (1 + r)` | 無限成長，不現實 |
| 邏輯斯蒂（有上限）| `x[t+1] = x[t] + r*x[t]*(1 - x[t]/K)` | 趨近上限 K 後停止成長 |

**簡化 SIR 感染模型（5 分鐘）**：

```
S（易感者）→ I（感染者）→ R（康復者）

每一步：
新感染人數 = beta * S * I / N
新康復人數 = gamma * I
```

---

### 範例示範（30 分鐘）

```python
import matplotlib.pyplot as plt

# 版本 1：指數成長人口
def simulate_exponential_growth(initial_pop, growth_rate, steps):
    population = [initial_pop]
    for t in range(steps):
        next_pop = population[-1] * (1 + growth_rate)
        population.append(next_pop)
    return population

# 版本 2：邏輯斯蒂成長（有上限）
def simulate_logistic_growth(initial_pop, growth_rate, carrying_capacity, steps):
    pop = [initial_pop]
    K = carrying_capacity
    for t in range(steps):
        current = pop[-1]
        next_pop = current + growth_rate * current * (1 - current / K)
        pop.append(next_pop)
    return pop

# 版本 3：簡化 SIR
def simulate_sir(N, I0, beta, gamma, steps):
    S, I, R = [N - I0], [I0], [0]
    for t in range(steps):
        new_infected = beta * S[-1] * I[-1] / N
        new_recovered = gamma * I[-1]
        S.append(S[-1] - new_infected)
        I.append(I[-1] + new_infected - new_recovered)
        R.append(R[-1] + new_recovered)
    return S, I, R

# 畫圖
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 人口對比
pop_exp = simulate_exponential_growth(1000, 0.05, 50)
pop_log = simulate_logistic_growth(1000, 0.1, 10000, 50)
axes[0].plot(pop_exp, label="指數成長")
axes[0].plot(pop_log, label="邏輯斯蒂成長")
axes[0].set_xlabel("時間步驟")
axes[0].set_ylabel("人口")
axes[0].set_title("人口成長模型比較")
axes[0].legend()

# SIR
S, I, R = simulate_sir(N=10000, I0=10, beta=0.3, gamma=0.05, steps=100)
axes[1].plot(S, label="易感者 S")
axes[1].plot(I, label="感染者 I")
axes[1].plot(R, label="康復者 R")
axes[1].set_xlabel("時間步驟（天）")
axes[1].set_ylabel("人數")
axes[1].set_title("簡化 SIR 模型")
axes[1].legend()

plt.tight_layout()
plt.show()
```

**關鍵問答**：
- 「`beta` 大一點，感染曲線會怎麼變？」（讓學生預測再改）
- 「如果 `gamma` 很大（康復很快），會怎樣？」

---

### 本週 3 個關鍵點

1. 差分思維：不需要微分方程，只需要「每一步的變化規則」
2. 指數成長無限增長；加入「上限」就得到邏輯斯蒂成長
3. SIR 模型三條曲線的意義：S 下降、I 先升後降、R 持續上升
