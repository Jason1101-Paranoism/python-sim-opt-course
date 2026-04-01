---
title: Level 1 週 4 學生講義 — 差分模型
type: teaching
level: "1"
week: 4
audience: student
status: draft
---

# Level 1 週 4：成長與衰退 — 差分模型
## 學生講義（Student Worksheet）

---

## 核心概念：差分思維

> 不需要微分方程。只需要知道：**下一步 = 這一步 + 變化量**

```python
x[t+1] = x[t] + change(x[t])
```

這個想法可以描述很多系統：
- 人口成長
- 疫情擴散
- 存款利滾利
- 物種競爭

---

## 模型 A：人口成長

### 指數成長（無上限）

```python
def simulate_exponential_growth(initial_pop, growth_rate, steps):
    population = [initial_pop]
    for t in range(steps):
        next_pop = population[-1] * (1 + growth_rate)
        population.append(next_pop)
    return population

pop = simulate_exponential_growth(initial_pop=1000, growth_rate=0.05, steps=50)
```

### 邏輯斯蒂成長（有上限 K）

```python
def simulate_logistic_growth(initial_pop, growth_rate, carrying_capacity, steps):
    pop = [initial_pop]
    K = carrying_capacity
    for t in range(steps):
        current = pop[-1]
        next_pop = current + growth_rate * current * (1 - current / K)
        pop.append(next_pop)
    return pop

pop = simulate_logistic_growth(1000, growth_rate=0.1, carrying_capacity=10000, steps=100)
```

> 加入 `(1 - current/K)` 的意思：當人口接近上限 K 時，成長速度放慢

---

## 模型 B：簡化 SIR 感染模型

```
S（Susceptible，易感者）→ 可能被感染
I（Infected，感染者）  → 已感染，會傳染給別人
R（Recovered，康復者）→ 康復後不再感染
```

```python
def simulate_sir(N, I0, beta, gamma, steps):
    """
    N: 總人口
    I0: 初始感染人數
    beta: 感染率（每天每個感染者平均傳染幾人）
    gamma: 康復率（每天感染者中有多少比例康復）
    """
    S, I, R = [N - I0], [I0], [0]

    for t in range(steps):
        new_infected = beta * S[-1] * I[-1] / N
        new_recovered = gamma * I[-1]

        S.append(S[-1] - new_infected)
        I.append(I[-1] + new_infected - new_recovered)
        R.append(R[-1] + new_recovered)

    return S, I, R

S, I, R = simulate_sir(N=10000, I0=10, beta=0.3, gamma=0.05, steps=100)
```

---

## 試試看

### 練習 1：比較兩種人口模型
同時畫出指數成長和邏輯斯蒂成長，觀察它們在哪個時間點開始不同。

### 練習 2：調整 SIR 參數
分別試試：
- `beta=0.5`（傳染力強）vs `beta=0.1`（傳染力弱）
- `gamma=0.1`（快速康復）vs `gamma=0.01`（慢速康復）

**觀察**：哪個參數對「感染高峰的高度」影響最大？

### 練習 3（挑戰）：加入「防疫措施」
在第 30 天時，把 `beta` 砍半（代表實施口罩令）。模擬結果如何改變？

```python
# 提示：你需要在迴圈裡加一個條件判斷
for t in range(steps):
    if t == 30:
        beta = beta / 2   # 第 30 天開始防疫
    ...
```
