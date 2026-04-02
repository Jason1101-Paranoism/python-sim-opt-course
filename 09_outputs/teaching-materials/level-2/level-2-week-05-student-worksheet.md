---
title: Level 2 週 05 — 程式架構與初版模型（學生講義）
type: teaching
level: 2
week: 5
audience: student
status: draft
source_files:
  - 01_curriculum/level-2.md
---

# Level 2 週 05：程式架構與初版模型

## 一、三段架構回顧

```
第一段：simulate_once(parameters)
    → 模型核心：跑一次，回傳你要的輸出指標

第二段：實驗迴圈
    → 掃描參數範圍，多次呼叫 simulate_once
    → 把結果存成 list

第三段：視覺化
    → 用圖表把結果說清楚
```

**今天的目標**：讓第一段跑起來，然後加上第二、三段，產出至少一張圖。

---

## 二、simulate_once 的寫作規範

好的 simulate_once 長這樣：

```python
def simulate_once(param1, param2, ...):
    """
    說明這個函數在模擬什麼（1 句話）
    
    假設：
    - 假設 1
    - 假設 2
    
    參數：
    - param1: 說明這個參數是什麼
    - param2: 說明這個參數是什麼
    
    回傳：
    - 說明回傳值是什麼（數字或 tuple）
    """
    # 初始化
    ...
    
    # 模擬主體
    for step in range(n_steps):
        ...
    
    return result
```

**為什麼要寫 docstring？**
- 之後寫報告的「方法」段落，假設已經整理好了
- 如果兩週後你忘了這個函數在做什麼，docstring 會告訴你

---

## 三、SIR 模型完整範例（今天的示範）

```python
import numpy as np
import matplotlib.pyplot as plt

def simulate_once(beta, gamma, N=1000, I0=10, days=160):
    """
    SIR 疾病傳播模型：確定性版本（無隨機性）
    
    假設：
    - 均質混合（每個人有相同機率接觸任何人）
    - 康復後不再感染（無再感染）
    - 不考慮出生/死亡
    
    參數：
    - beta: 接觸率（每天每感染者可感染的比例）
    - gamma: 康復率（每天康復的感染者比例）
    - N: 總人口數
    - I0: 初始感染人數
    - days: 模擬天數
    
    回傳：
    - (peak_I, peak_day): 感染高峰人數 和 高峰出現天數
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

# 掃描 beta
betas = np.arange(0.1, 0.6, 0.05)
peaks, peak_days = [], []

for b in betas:
    pi, pd = simulate_once(b, gamma=0.1)
    peaks.append(pi)
    peak_days.append(pd)

# 畫圖
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

---

## 四、你的 MVP simulate_once

**現在輪到你了。**

先只寫 simulate_once，讓它可以執行：

```python
def simulate_once(???):
    """
    [你的模型說明]
    
    假設：
    - [你的假設 1]
    - [你的假設 2]
    
    參數：
    - [你的參數說明]
    
    回傳：
    - [你的輸出指標]
    """
    # 初始化
    
    # 模擬主體
    
    # 回傳結果
    pass
```

**MVP 的原則：**
- 先讓它不 error，再讓它正確，再讓它完整
- 如果某個部分太複雜，先用簡化版代替（例如：先用固定值代替隨機數）
- 跑出任何一個數字就算成功，之後再優化

---

## 五、Debug 備忘錄

| 錯誤類型 | 可能原因 | 解法 |
|---------|---------|------|
| `TypeError` / `NameError` | 變數名稱打錯 / 未定義 | 逐行檢查，加 `print` 確認 |
| 結果是 `nan` 或 `inf` | 除以 0，或數值溢出 | 確認分母不為 0；縮小數值範圍 |
| 圖是空的 | 忘了呼叫 `plt.show()` | 加 `plt.show()` |
| 迴圈跑完但結果都一樣 | 結果沒有隨參數改變 | 確認 simulate_once 裡真的用到了參數 |
