---
title: Level 3 週 01 — 目標與舊專題回顧（教師教案）
type: teaching
level: 3
week: 1
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 01 教師教案
## 主題：目標與舊專題回顧

---

## 1. 本週學習目標

1. 每位學生能清楚描述自己在 Level 2（或舊專題）中所建立的模型與程式架構
2. 學生能辨識舊專題在技術上的局限性（例如：for 迴圈效率低、缺乏數值方法、無最佳化流程）
3. 學生能用具體語言說出「升級方向」：要引入哪些工具或方法
4. 全班對 Level 3 的工具（NumPy、SciPy）與核心主題有初步認識

---

## 2. 課堂時間分配表（120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 每人用 90 秒口頭介紹舊專題（題目、方法、成果） |
| 概念講解 | 25 分鐘 | Level 3 總覽：為何需要數值方法與最佳化、工具地圖 |
| 範例示範（Live coding） | 30 分鐘 | 展示一個「升級前 vs 升級後」對比程式（簡單模擬） |
| 學生實作 | 35 分鐘 | 學生自行填寫「升級計畫表」並在小組中說明 |
| 回顧與預告 | 20 分鐘 | 統整共同痛點、預告 Week 2 NumPy 主題 |

---

## 3. 各段落執行細節

### 3.1 暖身回顧（10 分鐘）

**引入問題**（板書或投影）：
> 「你在 Level 2 的專題，最複雜的計算是哪一段程式？它跑起來快嗎？結果你確定是對的嗎？」

- 每位學生輪流用 90 秒介紹自己的舊專題，要求說清楚：
  1. 問題是什麼
  2. 怎麼模擬或計算的（幾句話說清楚方法）
  3. 有沒有覺得「這段程式可以更好」的地方
- 教師記下每位學生的技術弱點，後續 1-on-1 使用

### 3.2 概念講解（25 分鐘）

**講解重點一：Level 3 工具地圖（10 分鐘）**

在白板畫出以下結構：

```
問題類型            工具
──────────────────────────────
大量重複計算    →  NumPy 向量化
解方程 / 積分   →  SciPy optimize / integrate
動態系統模擬    →  SciPy odeint
找最佳解        →  SciPy minimize
```

強調：「這些工具都不是為了考試，是為了讓你的專題程式跑得更快、結果更可信、報告更有說服力。」

**講解重點二：什麼叫「技術升級」（10 分鐘）**

舉兩個對比範例：

**例 A：效率升級**
```python
# 升級前（Level 2 風格）
results = []
for i in range(10000):
    results.append(i * 2.5 + 1.3)

# 升級後（Level 3 風格）
import numpy as np
i = np.arange(10000)
results = i * 2.5 + 1.3  # 快 100 倍以上
```

**例 B：方法升級**
```python
# 升級前：手動用迴圈找零點（不精確）
for x in range(-100, 100):
    if abs(x**2 - 5) < 0.1:
        print(x)

# 升級後：用數值求根（精確）
from scipy.optimize import brentq
root = brentq(lambda x: x**2 - 5, 0, 5)
print(root)  # 2.2360679...
```

**講解重點三：升級的意義不只是「換工具」（5 分鐘）**

說明 Level 3 的核心是讓專題從「可以跑」升級到「有方法論」：
- 報告中能寫出「使用 SciPy odeint 解 Lotka-Volterra 方程」而不只是「用 Python 算」
- 能討論數值方法的限制與誤差

### 3.3 範例示範（Live Coding）（30 分鐘）

**示範情境**：用一個「Level 2 風格的費波那契模擬」升級成 Level 3 風格

目的是讓學生看到「同一個問題，升級前後的程式差在哪裡」，而不是教新語法（語法下週才教）。

```python
# ========== 升級前（Level 2 風格） ==========
import time

def simulate_population_v1(generations, growth_rate):
    """舊版：用 for 迴圈模擬族群增長"""
    population = [100]  # 初始族群
    for _ in range(generations - 1):
        next_pop = population[-1] * growth_rate
        population.append(next_pop)
    return population

start = time.time()
pop_v1 = simulate_population_v1(10000, 1.02)
print(f"舊版執行時間：{time.time() - start:.4f} 秒")
print(f"第 10000 代族群：{pop_v1[-1]:.2f}")


# ========== 升級後（Level 3 NumPy 風格） ==========
import numpy as np

def simulate_population_v2(generations, growth_rate):
    """新版：用 NumPy 向量化計算族群增長"""
    t = np.arange(generations)
    population = 100 * (growth_rate ** t)  # 向量化，一行搞定
    return population

start = time.time()
pop_v2 = simulate_population_v2(10000, 1.02)
print(f"新版執行時間：{time.time() - start:.4f} 秒")
print(f"第 10000 代族群：{pop_v2[-1]:.2f}")

# 驗證兩者結果一致
print(f"結果差異：{abs(pop_v1[-1] - pop_v2[-1]):.6f}")
```

**Live coding 提示**：
- 先讓學生猜「哪個快？快多少？」再執行，製造驚喜感
- 強調「結果驗證」的重要性：升級後要確認答案沒有改變
- 如有時間，再展示一個加上 Matplotlib 圖表的版本

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 4))
plt.subplot(1, 2, 1)
plt.plot(pop_v1[:50])
plt.title("舊版（前 50 代）")
plt.xlabel("世代")
plt.ylabel("族群數量")

plt.subplot(1, 2, 2)
plt.plot(pop_v2[:50], color='orange')
plt.title("新版（前 50 代）")
plt.xlabel("世代")
plt.ylabel("族群數量")

plt.tight_layout()
plt.show()
```

### 3.4 學生實作（35 分鐘）

學生獨立完成「升級計畫表」（見 student-worksheet），要求：
1. 用文字描述舊專題的核心計算邏輯
2. 找出 1–2 個可以用 Level 3 工具升級的地方
3. 嘗試用 2–3 行 Python 的偽代碼（pseudo-code）寫出升級後的樣子

最後 10 分鐘讓 2–3 位學生上台分享，教師給即時回饋。

### 3.5 回顧與預告（20 分鐘）

**統整（10 分鐘）**：
- 列出班上學生共同的痛點（通常是：for 迴圈太慢、結果驗證不夠、沒有最佳化）
- 確認每位學生都有「至少一個明確的升級方向」

**預告 Week 2（10 分鐘）**：
- 下週主題：NumPy——為什麼 array 比 list 快這麼多？
- 請學生課前準備：把舊專題裡「最核心的一段計算」找出來，下週要動手改寫
- 課前預習建議：NumPy 官方入門 10 分鐘教學（https://numpy.org/doc/stable/user/quickstart.html）

---

## 4. 關鍵程式碼範例（完整可執行）

```python
"""
Level 3 Week 01 — 升級前後對比示範
執行環境：Python 3.8+，需安裝 numpy, matplotlib
"""
import time
import numpy as np
import matplotlib.pyplot as plt

# ── 升級前：for 迴圈版本 ──────────────────────────────────
def simulate_v1(n, r):
    """模擬 n 代族群增長，初始 100，增長率 r"""
    pop = [100.0]
    for _ in range(n - 1):
        pop.append(pop[-1] * r)
    return pop

# ── 升級後：NumPy 向量化版本 ──────────────────────────────
def simulate_v2(n, r):
    """同上，NumPy 版本"""
    t = np.arange(n)
    return 100.0 * (r ** t)

# ── 計時比較 ──────────────────────────────────────────────
N = 100_000

t0 = time.perf_counter()
result_v1 = simulate_v1(N, 1.001)
t1 = time.perf_counter()
result_v2 = simulate_v2(N, 1.001)
t2 = time.perf_counter()

print(f"for 迴圈版本：{(t1 - t0)*1000:.2f} ms")
print(f"NumPy 版本：  {(t2 - t1)*1000:.2f} ms")
print(f"加速倍率：    {(t1 - t0) / (t2 - t1):.1f}x")
print(f"結果誤差：    {abs(result_v1[-1] - result_v2[-1]):.2e}")

# ── 視覺化 ────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(result_v2[:200], label="族群增長（NumPy）", color="steelblue")
ax.set_xlabel("世代")
ax.set_ylabel("族群數量")
ax.set_title("Level 3 Week 01：升級後模擬結果")
ax.legend()
plt.tight_layout()
plt.show()
```

---

## 5. 常見問題與處理方式

| 問題 | 可能原因 | 建議處理 |
|------|----------|----------|
| 學生說「我的舊專題沒有可以升級的地方」 | 對 Level 3 工具不熟悉，無法對應 | 引導問：「你的計算有沒有用到 for 迴圈？有沒有要解方程？有沒有要找最佳參數？」幾乎所有專題都有可升級之處 |
| 學生的舊專題太簡單（只有幾十行） | Level 2 完成度不足 | 不要當場批評，記下來 1-on-1 討論，考慮先從課程內建的範例題目出發 |
| 學生找不到舊專題程式碼 | 沒帶筆電或程式碼遺失 | 要求下次上課前上傳到 Google Drive 或 GitHub；本週先用紙筆描述邏輯 |
| 多位學生同時問「NumPy 要怎麼裝？」 | 環境設定問題 | 統一花 5 分鐘帶全班執行 `pip install numpy scipy matplotlib`，確認環境就緒 |
| 學生對「升級」沒有動機感 | 覺得舊版本已經夠用 | 展示大計算量下 for 迴圈 vs NumPy 的時間差，通常 100x 加速就能引發興趣 |
