---
title: Level 1 週 6 教師教案 — 視覺化：用圖說故事
type: teaching
level: "1"
week: 6
audience: teacher
status: draft
---

# Level 1 週 6：視覺化 — 用圖說故事
## 教師教案（Teacher Notes）

---

### 本週學習目標

1. 學生能選擇合適的圖表類型（折線圖、直方圖、散點圖）
2. 學生能為圖表加上完整的標題、軸標籤、圖例
3. 學生能為自己的小專題產出至少 2 張可說明發現的圖表

---

### 課堂時間分配（120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 展示學生的第一版專題截圖，討論「圖缺少什麼」 |
| 概念講解 | 25 分鐘 | 三種圖表的適用情境 + 「好圖 vs 壞圖」比較 |
| 範例示範 | 30 分鐘 | 把一個「裸圖」一步步改成「有說服力的圖」 |
| 學生實作 | 35 分鐘 | 學生為自己的專題重新做圖 |
| 回顧與預告 | 20 分鐘 | 每人展示 1 張最新的圖，同儕給一句回饋 |

---

### 概念講解（25 分鐘）

**三種圖表的使用情境（12 分鐘）**：

| 圖表 | 適合什麼 | 例子 |
|------|----------|------|
| 折線圖 | 隨時間變化的量 | 人口變化、SIR 曲線、等待時間序列 |
| 直方圖 | 一個數值的分佈 | 等待時間分佈、骰子結果、模擬結果統計 |
| 散點圖 | 兩個變數的關係 | 服務率 vs 平均等待時間、N vs 模擬誤差 |

**好圖 vs 壞圖（8 分鐘）**：

展示同一張圖的兩個版本：

壞圖（學生常見錯誤）：
- 沒有標題
- x/y 軸沒有標籤和單位
- 圖例寫「line1, line2」
- 字太小，無法閱讀

好圖：
- 標題說清楚「這張圖在告訴我什麼發現」
- 軸標籤有單位（分鐘、人數、次數）
- 圖例用有意義的名稱
- 配色對色盲友好（避免純紅/純綠同時使用）

**圖說的寫法（5 分鐘）**：

> 「圖說（caption）= 一句話告訴讀者他應該從這張圖得到什麼結論」

差的圖說：「圖 1：等待時間直方圖」（只說圖是什麼）
好的圖說：「圖 1：服務率 = 0.5 時，大多數顧客等待時間低於 5 分鐘，但尾巴較長」

---

### 範例示範（30 分鐘）

**從「裸圖」改成「有說服力的圖」**：

```python
import matplotlib.pyplot as plt
import numpy as np

# 生成示範數據
np.random.seed(42)
wait_times_fast = np.random.exponential(2, 200)   # 快速服務
wait_times_slow = np.random.exponential(8, 200)   # 慢速服務

# 版本 1：裸圖（先展示這個，問學生缺什麼）
plt.hist(wait_times_fast)
plt.show()

# 版本 2：完整圖
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].hist(wait_times_fast, bins=20, alpha=0.7, color='steelblue', edgecolor='black')
axes[0].set_xlabel("等待時間（分鐘）")
axes[0].set_ylabel("顧客人數")
axes[0].set_title("快速服務（平均 2 分鐘）的等待時間分佈")
axes[0].axvline(np.mean(wait_times_fast), color='red', linestyle='--',
                label=f'平均值 = {np.mean(wait_times_fast):.1f} 分鐘')
axes[0].legend()

axes[1].hist(wait_times_slow, bins=20, alpha=0.7, color='coral', edgecolor='black')
axes[1].set_xlabel("等待時間（分鐘）")
axes[1].set_ylabel("顧客人數")
axes[1].set_title("慢速服務（平均 8 分鐘）的等待時間分佈")
axes[1].axvline(np.mean(wait_times_slow), color='red', linestyle='--',
                label=f'平均值 = {np.mean(wait_times_slow):.1f} 分鐘')
axes[1].legend()

plt.suptitle("服務速率對等待時間的影響", fontsize=14)
plt.tight_layout()
plt.savefig("waiting_time_comparison.png", dpi=150)
plt.show()
```

**教學重點**：
- `alpha=0.7`：讓圖半透明，重疊時能看清
- `axvline`：加平均值參考線
- `suptitle`：整張圖的大標題
- `savefig`：存成 PNG（截圖不如直接存）

---

### 學生實作（35 分鐘）

**任務**：為自己的個人小專題重新做圖，要求：
1. 至少 2 張圖，類型不同
2. 每張圖有完整的標題、軸標籤
3. 至少一張圖有圖說（寫在程式注解或 Google Doc）

---

### 回顧（20 分鐘）

每人把最好的一張圖展示 30 秒，全班給一句回饋：「這張圖讓我看到了什麼？」

**老師重點觀察**：哪些學生的圖「說出了發現」，哪些只是「輸出了數字」。
