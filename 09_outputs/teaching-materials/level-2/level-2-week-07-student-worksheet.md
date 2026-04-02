---
title: Level 2 週 07 — 參數掃描與簡單最佳化（學生講義）
type: teaching
level: 2
week: 7
audience: student
status: draft
source_files:
  - 01_curriculum/level-2.md
---

# Level 2 週 07：參數掃描與（簡單）最佳化

## 一、實驗設計三原則

### 原則 1：每次只改一個參數

你有兩個參數 A 和 B。如果同時改，你不知道輸出的變化是由 A 還是 B 造成的。

**正確方式**：
1. 固定 B，掃描 A 的範圍
2. 固定 A，掃描 B 的範圍
3. 如果要研究 A 和 B 的「交互作用」，用雙參數 heatmap

### 原則 2：參數範圍要有意義

- 太窄：看不出趨勢
- 太寬：模型可能不適用
- 建議：先跑 5–10 個值看趨勢，再決定要不要加密

### 原則 3：有隨機性就多跑幾次

| 模型類型 | 建議 |
|---------|------|
| 確定性（無隨機性）| 每個參數值跑 1 次即可 |
| 有隨機性 | 每個參數值跑 30–50 次，取平均 |

---

## 二、單參數掃描範例

```python
import numpy as np
import matplotlib.pyplot as plt

# simulate_once 已定義好
betas = np.linspace(0.1, 0.5, 20)
results = []

for b in betas:
    peak_I, _ = simulate_once(b, gamma=0.1)
    results.append(peak_I)

plt.plot(betas, results, marker='o', color='tomato')
plt.xlabel("接觸率 β")
plt.ylabel("感染高峰人數")
plt.title("β 對感染高峰的影響（γ = 0.1 固定）")
plt.grid(True)
plt.show()
```

---

## 三、雙參數 Heatmap（進階）

```python
betas = np.linspace(0.1, 0.5, 10)
gammas = np.linspace(0.05, 0.3, 10)

peak_matrix = np.zeros((len(gammas), len(betas)))
for i, g in enumerate(gammas):
    for j, b in enumerate(betas):
        peak_matrix[i, j] = simulate_once(b, g)[0]

plt.imshow(peak_matrix, origin='lower',
           extent=[betas[0], betas[-1], gammas[0], gammas[-1]],
           aspect='auto', cmap='hot_r')
plt.colorbar(label="感染高峰人數")
plt.xlabel("接觸率 β")
plt.ylabel("康復率 γ")
plt.title("β 和 γ 的交互影響")
plt.show()
```

**讀 Heatmap 的方法：**
- 顏色越深（或越亮，依色板）= 數值越大
- 橫軸和縱軸對應你掃描的兩個參數
- 找出「哪個角落最深」= 哪種參數組合讓輸出最大

---

## 四、暴力搜尋最佳參數（選做）

```python
# 找讓高峰最晚出現的 β
best_beta = None
latest_day = -1

for b in np.linspace(0.1, 0.5, 100):
    _, peak_day = simulate_once(b, gamma=0.1)
    if peak_day > latest_day:
        latest_day = peak_day
        best_beta = b

print(f"最佳 β = {best_beta:.3f}，對應高峰天數 = {latest_day}")
```

**注意**：暴力搜尋只在你定義的範圍內找最佳值，範圍外的可能更好。

---

## 五、你的實驗設計表

填寫你的專題實驗計畫：

| 參數 | 掃描範圍 | 掃描值數量 | 其他參數固定在 |
|------|---------|----------|--------------|
| [參數 1] | [最小值] ~ [最大值] | [N 個] | [其他參數值] |
| [參數 2] | [最小值] ~ [最大值] | [N 個] | [其他參數值] |

**我的模型有隨機性嗎？**
- [ ] 有 → 每個參數值要跑 ___ 次取平均
- [ ] 沒有 → 每個參數值跑 1 次

---

## 六、分析結果的思考框架

跑完實驗，看著你的圖，問自己：

1. **趨勢是什麼？**（增加？減少？先增後減？）
2. **為什麼是這個趨勢？**（連回你的模型邏輯）
3. **有沒有讓你意外的地方？**（如果有，為什麼？是模型的問題還是現象本身？）
4. **你的 RQ 現在能回答了嗎？**（你的圖有沒有給出一個答案？）

這四個問題的答案，就是報告「結果與討論」段落的內容。
