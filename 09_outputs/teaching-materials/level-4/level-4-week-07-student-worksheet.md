---
title: Level 4 週 07 — 實驗與結果呈現（研究版）（學生講義）
type: teaching
level: 4
week: 7
audience: student
status: draft
source_files:
  - 01_curriculum/level-4.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 4 週 07 — 實驗與結果呈現（研究版）

---

## 本週主題說明

方法章說的是「你計劃怎麼做」；本週要真的做完，並把結果用研究報告的標準呈現出來。

研究版的結果呈現有兩個關鍵要求：

1. **實驗要嚴謹**：控制變因、足夠的重複次數、測試邊界情境
2. **呈現要清楚**：圖表有標題/軸標籤/單位，文字只陳述事實不解釋

---

## 核心概念 1：設計嚴謹的實驗

### 控制變因（Controlled Variables）

「每次只動一個旋鈕」是實驗設計的基本原則。

如果你想知道「參數 A 對結果的影響」，你就改變 A，同時把其他所有參數固定。如果你同時改了 A 和 B，你就無法判斷是 A 還是 B（或兩者的交互作用）造成了結果的差異。

**範例**：
- 正確做法：固定顧客到達率，只改變服務台數量
- 錯誤做法：同時改變顧客到達率和服務台數量

### 重複次數（Number of Replications）

含有隨機性的模擬，每次跑的結果不一樣。你需要跑多次，取平均（和標準差），才能得到穩定可信的估計。

**怎麼決定要跑幾次？**

最簡單的方法是「收斂測試」：

```python
import numpy as np

# 以排隊模擬為例
results = []
for n_reps in [10, 50, 100, 500, 1000]:
    means = [simulate_queue() for _ in range(n_reps)]
    print(f"n={n_reps:4d}: mean={np.mean(means):.3f}, std={np.std(means):.3f}")
```

觀察：當 n 增加到某個數後，mean 和 std 趨於穩定，表示次數夠了。

**Level 4 的基本標準**：每個情境至少 100 次（若計算時間允許，建議 500 次）。

### 邊界情境（Boundary Cases）

除了你最感興趣的中間參數值，也要測試：
- **最小值**（例如：服務台 1 個時，等待時間最長是多少？）
- **最大值**（例如：服務台 10 個時，等待時間已經可以忽略嗎？）
- **臨界點**（例如：系統從穩定到崩潰的邊界在哪裡？）

邊界情境的結果往往能為討論章提供豐富的素材。

---

## 核心概念 2：製作研究等級的圖表

### 一張合格的研究圖應該包含

| 要素 | 說明 | Python 對應 |
|------|------|-------------|
| 圖標題 | 描述圖的內容，置於圖下方（Figure X.） | 用圖說（caption）在報告中說明，`ax.set_title()` 可選 |
| 橫軸標籤 + 單位 | 例如：「Time (days)」 | `ax.set_xlabel('Time (days)', fontsize=12)` |
| 縱軸標籤 + 單位 | 例如：「Infected Individuals (n)」 | `ax.set_ylabel(...)` |
| 圖例（若有多條線） | 說明每條線代表什麼 | `ax.legend(fontsize=11)` |
| 誤差棒（若有重複） | 顯示標準差或信賴區間 | `ax.errorbar()` 或 `ax.fill_between()` |
| 適當的字體大小 | 至少 12pt（避免縮印後看不清楚） | `fontsize=12`, `tick_params(labelsize=11)` |
| 適當的線條粗細 | linewidth ≥ 2 | `ax.plot(..., linewidth=2)` |

### 加誤差棒的範例程式碼

```python
import numpy as np
import matplotlib.pyplot as plt

# 模擬數據：每個情境重複 200 次
n_replications = 200
parameter_values = [1, 2, 3, 4]  # 例如：服務台數量
means = []
stds = []

for c in parameter_values:
    results = [simulate_queue(num_servers=c) for _ in range(n_replications)]
    means.append(np.mean(results))
    stds.append(np.std(results))

# 畫圖
fig, ax = plt.subplots(figsize=(7, 5))
ax.errorbar(parameter_values, means, yerr=stds, 
            fmt='o-', linewidth=2, capsize=5, markersize=8,
            label='Mean ± 1 Std Dev')
ax.set_xlabel('Number of Servers', fontsize=12)
ax.set_ylabel('Average Waiting Time (min)', fontsize=12)
ax.legend(fontsize=11)
ax.tick_params(labelsize=11)
plt.tight_layout()
plt.savefig('figure2_waiting_time.png', dpi=150)
plt.show()
```

### 一張合格的結果表格應包含

| 要素 | 範例 |
|------|------|
| 表格標題 | Table 1. Summary of Simulation Results |
| 欄位名稱 + 單位 | R₀, Peak Infected (n), Peak Day, Duration (days) |
| 數字對齊 | 右對齊，保留 1–2 位小數 |
| 無多餘的顏色或格線 | 簡潔即可 |

---

## 核心概念 3：描述結果的標準寫法

### 結果章的唯一任務：陳述事實，不解釋

結果章（Results）只說「數據顯示什麼」，不說「為什麼是這個結果」。

解釋留到討論章（Discussion）。這個分離是學術寫作的基本規範。

### 標準句型

| 句型 | 範例 |
|------|------|
| 引用圖 | 「如圖 1 所示，...」 |
| 引用表 | 「如表 1 所示，...」 |
| 描述趨勢 | 「X 隨 Y 的增加而單調遞增 / 遞減。」 |
| 描述具體數值 | 「當 [參數] = [值] 時，[指標] 為 [數值]（[單位]）。」 |
| 描述最大/最小 | 「[指標] 在 [條件] 時達到最大值 [N]。」 |
| 描述比較 | 「與 [情境 A] 相比，[情境 B] 的 [指標] 高出 [X]%。」 |

### 範例：好的 vs 不好的結果段落

**情境**：比較不同 R₀ 值對 SIR 模型中感染峰值的影響

**不好的版本**：
> 我發現 R₀ 越大感染人數越多，這是因為傳染力越強，所以感染擴散得越快。R₀=3 的時候最嚴重，感覺很可怕。

**問題**：
- 「我發現」——口語
- 「這是因為...」——在結果章解釋原因不對
- 「感覺很可怕」——主觀，沒有量化

**好的版本**：
> 如圖 1 所示，感染人數峰值（peak infected count）隨 R₀ 的增加而單調遞增。當 R₀ 由 1.5 增加至 3.0 時，峰值感染人數由約 2,100 人增加至約 7,800 人，增幅達 271%。如表 1 所示，R₀ 的增加同時導致疫情達到峰值的時間由第 87 天提前至第 31 天。此外，在所有情境中，R₀ = 1.5 時疫情持續時間最長，約為 145 天；R₀ = 3.0 時疫情持續時間最短，約為 82 天。

---

## 練習任務

### 練習 1：收斂測試

針對你自己的模擬程式，進行收斂測試。設計一個類似下方的測試並填寫表格：

| 重複次數 | 均值 | 標準差 | 你的觀察 |
|---------|------|-------|---------|
| 10 | | | |
| 50 | | | |
| 100 | | | |
| 500 | | | |

問題：從哪個次數開始，均值和標準差趨於穩定？你決定用幾次作為正式實驗的重複次數？

---

### 練習 2：圖表自我檢查

製作你的第一張結果圖後，對照以下清單：

- [ ] 圖有 Figure X. 的標題說明
- [ ] 橫軸有標籤和單位
- [ ] 縱軸有標籤和單位
- [ ] 若有多條線，有圖例
- [ ] 若為隨機模擬，有誤差棒
- [ ] 字體至少 12pt，線條夠粗

---

### 練習 3：把「口語結果」改寫為「學術結果」

**原文（需要改寫）**：
> 從圖裡可以看出，參數 α = 0.5 的時候效果最好，大概比 α = 0.1 好了三倍左右，所以應該要用比較大的 α 值。

**你的改寫版本**：

> _______________________________________________
