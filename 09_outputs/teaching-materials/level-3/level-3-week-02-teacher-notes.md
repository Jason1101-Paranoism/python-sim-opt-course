---
title: Level 3 週 02 — NumPy 陣列與向量化（教師教案）
type: teaching
level: 3
week: 2
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 02 教師教案
## 主題：NumPy — 陣列與向量化

---

## 1. 本週學習目標

1. 學生能說出 NumPy array 和 Python list 的核心差異（型別一致性、記憶體連續性）
2. 學生能執行 NumPy 陣列的基本操作：建立、索引、切片、形狀轉換、基本數學
3. 學生能用計時方式（`time.perf_counter` 或 `timeit`）比較 for 迴圈與向量化的效率差異
4. 學生能把自己舊專題中一段含 for 迴圈的計算改寫成 NumPy 版本，並驗證結果一致性

---

## 2. 課堂時間分配表（120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 提問舊專題升級方向，引出「為什麼 for 迴圈慢」 |
| 概念講解 | 25 分鐘 | NumPy array 概念、向量化原理、記憶體直覺 |
| 範例示範（Live coding） | 30 分鐘 | 建立 array、基本操作、計時比較、改寫示範 |
| 學生實作 | 35 分鐘 | 學生把自己的程式片段改寫成 NumPy |
| 回顧與預告 | 20 分鐘 | 回顧常見錯誤、預告 Week 3 SciPy |

---

## 3. 各段落執行細節

### 3.1 暖身回顧（10 分鐘）

**引入問題**：
> 「有沒有人跑過一個程式，等了超過 10 秒才出結果？你當時覺得是為什麼慢？」

讓 2–3 位學生說說看，再引導出：
- Python for 迴圈的「慢」是因為每次迭代都要進行型別檢查、記憶體分配等開銷
- NumPy 的向量化是把計算交給底層 C/Fortran 程式執行，繞過 Python 的開銷

**接著問**：「上週你找到了哪段程式想用 NumPy 改寫？今天我們就要動手。」

---

### 3.2 概念講解（25 分鐘）

**概念一：NumPy array vs Python list（10 分鐘）**

在白板上畫出記憶體結構差異：

```
Python list：
[ ptr→int, ptr→float, ptr→str, ptr→int ]
  ↓           ↓          ↓         ↓
[記憶體A]  [記憶體B]  [記憶體C]  [記憶體D]
（分散在各處，每個元素有自己的型別資訊）

NumPy array：
[  1.0  |  2.0  |  3.0  |  4.0  ]
（連續記憶體，型別固定，無額外開銷）
```

說明三個關鍵差異：
1. **型別一致**：array 裡的元素型別必須相同（int32、float64 等）
2. **記憶體連續**：可以利用 CPU 快取，大幅加速
3. **廣播（Broadcasting）**：`a * 2` 可以直接把整個 array 乘以 2，不需要 for 迴圈

**概念二：向量化的直覺（10 分鐘）**

舉生活化例子：
- 舊方法（for 迴圈）：你去超市，一個一個拿商品放進籃子
- 向量化：你一次把整排商品全掃進去

用 NumPy 的「通用函數（ufunc）」概念說明：
- `np.sin(array)`、`np.exp(array)` 等函數都是向量化的
- 它們對 array 的每個元素操作，但速度接近 C 語言

**概念三：形狀（shape）與維度（5 分鐘）**

```
1D array：shape = (5,)        → 長度為 5 的向量
2D array：shape = (3, 4)      → 3 列 4 欄的矩陣
3D array：shape = (2, 3, 4)   → 2 個 3×4 矩陣
```

用 `array.shape`、`array.ndim`、`array.reshape()` 展示

---

### 3.3 範例示範（Live Coding）（30 分鐘）

**第一段（10 分鐘）：NumPy 基本操作**

```python
import numpy as np

# 建立 array 的幾種方式
a = np.array([1, 2, 3, 4, 5])          # 從 list 建立
b = np.arange(0, 10, 2)                # 0, 2, 4, 6, 8
c = np.linspace(0, 1, 5)               # 0, 0.25, 0.5, 0.75, 1.0
d = np.zeros((3, 4))                   # 3×4 全零矩陣
e = np.ones((2, 3))                    # 2×3 全一矩陣
f = np.random.normal(0, 1, size=1000)  # 標準常態分布 1000 個數

# 索引與切片
print(a[0])       # 第一個元素：1
print(a[-1])      # 最後一個元素：5
print(a[1:4])     # 第 2 到第 4 個元素：[2, 3, 4]
print(a[::2])     # 每隔一個：[1, 3, 5]

# 形狀操作
m = np.arange(12).reshape(3, 4)   # 建立 3×4 矩陣
print(m.shape)     # (3, 4)
print(m.T.shape)   # (4, 3)，轉置

# 基本數學（向量化）
x = np.array([1.0, 2.0, 3.0])
print(x * 2)         # [2, 4, 6]
print(x ** 2)        # [1, 4, 9]
print(np.sqrt(x))    # [1, 1.414, 1.732]
print(x.sum())       # 6.0
print(x.mean())      # 2.0
```

**第二段（10 分鐘）：計時比較**

```python
import numpy as np
import time

N = 1_000_000

# ── for 迴圈版本 ──
def compute_v1(n):
    result = []
    for i in range(n):
        result.append(i * 0.001 + np.sin(i * 0.001))
    return result

# ── NumPy 版本 ──
def compute_v2(n):
    x = np.arange(n) * 0.001
    return x + np.sin(x)

# 計時
t0 = time.perf_counter()
r1 = compute_v1(N)
t1 = time.perf_counter()
r2 = compute_v2(N)
t2 = time.perf_counter()

print(f"for 迴圈版本：{(t1 - t0)*1000:.1f} ms")
print(f"NumPy 版本：  {(t2 - t1)*1000:.1f} ms")
print(f"加速倍率：    約 {(t1 - t0) / (t2 - t1):.0f}x")

# 驗證結果一致
import numpy.testing as npt
npt.assert_allclose(r1, r2, rtol=1e-10)
print("✓ 兩版本結果一致")
```

**第三段（10 分鐘）：把真實的模擬改寫成 NumPy**

用一個模擬「丟硬幣累積」的例子，示範完整改寫流程：

```python
import numpy as np
import matplotlib.pyplot as plt

# ── 舊版：模擬 1000 次丟硬幣，追蹤累積正面次數 ──
def coin_flip_v1(trials):
    heads_count = 0
    record = []
    for _ in range(trials):
        flip = 1 if np.random.random() < 0.5 else 0
        heads_count += flip
        record.append(heads_count)
    return record

# ── 新版：NumPy 向量化 ──
def coin_flip_v2(trials):
    flips = np.random.randint(0, 2, size=trials)  # 一次生成所有結果
    return np.cumsum(flips)                         # 累積加總

trials = 10_000
record_v1 = coin_flip_v1(trials)
record_v2 = coin_flip_v2(trials)

# 驗證：最後的累積值應該接近（但因隨機，不會完全相同）
print(f"舊版最終正面次數：{record_v1[-1]}")
print(f"新版最終正面次數：{record_v2[-1]}")

# 視覺化
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(record_v2, label="累積正面次數（NumPy）", color="steelblue")
ax.axhline(trials / 2, color="red", linestyle="--", label="期望值（5000）")
ax.set_xlabel("丟硬幣次數")
ax.set_ylabel("累積正面次數")
ax.set_title("1000 次丟硬幣模擬")
ax.legend()
plt.tight_layout()
plt.show()
```

---

### 3.4 學生實作（35 分鐘）

引導學生完成 worksheet 的實作題：
- 拿出自己舊專題中「含 for 迴圈的那段計算」
- 嘗試改寫成 NumPy 版本
- 用 `time.perf_counter` 計時，計算加速倍率
- 用 `np.allclose` 或 `npt.assert_allclose` 驗證結果一致

教師巡視時優先處理：
- 學生不知道怎麼對應 for 迴圈與 NumPy 操作的（需要個別指導）
- 改寫後結果不一致的（通常是浮點精度問題，教 `rtol` 參數）
- 不知道從哪裡開始的（讓他先寫 `np.array([他的資料])` 確認形狀）

---

### 3.5 回顧與預告（20 分鐘）

**常見錯誤回顧（10 分鐘）**：
- 用 `+` 合併兩個 array 的形狀不匹配（廣播規則）
- 想要的是 element-wise 乘法，卻用了矩陣乘法（`*` vs `@`）
- 改寫後忘記驗證結果

**預告 Week 3（10 分鐘）**：
- 下週：SciPy——當你需要「解方程式」或「計算面積」時
- 預習建議：思考你的專題裡有沒有「某個等式要等於零才對」或「某個量是面積/總量」的計算

---

## 4. 關鍵程式碼範例（完整可執行）

```python
"""
Level 3 Week 02 — NumPy 向量化完整示範
執行環境：Python 3.8+，需安裝 numpy, matplotlib
"""
import numpy as np
import numpy.testing as npt
import time
import matplotlib.pyplot as plt

# ── 1. 建立各種 array ──────────────────────────────────
a1 = np.arange(10)                     # [0,1,2,...,9]
a2 = np.linspace(0, np.pi, 100)        # 0 到 π 的 100 個點
a3 = np.random.normal(0, 1, (3, 3))    # 3×3 標準常態隨機矩陣

print("a1:", a1)
print("a2 shape:", a2.shape)
print("a3:\n", a3)

# ── 2. 向量化數學 ──────────────────────────────────────
x = np.linspace(0, 2 * np.pi, 1000)
y = np.sin(x) + 0.5 * np.sin(2 * x)   # 一行計算 1000 個點

# ── 3. 計時比較（模擬計算） ───────────────────────────
N = 500_000

def v1(n):
    result = []
    for i in range(n):
        result.append(np.sin(i / 1000.0))
    return result

def v2(n):
    x = np.arange(n) / 1000.0
    return np.sin(x)

t0 = time.perf_counter()
r1 = v1(N)
t1 = time.perf_counter()
r2 = v2(N)
t2 = time.perf_counter()

print(f"\nfor 迴圈：{(t1-t0)*1000:.1f} ms")
print(f"NumPy：   {(t2-t1)*1000:.1f} ms")
print(f"加速：    {(t1-t0)/(t2-t1):.0f}x")

npt.assert_allclose(r1, r2, rtol=1e-12)
print("✓ 結果一致")

# ── 4. 視覺化 ─────────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot(x, y, color="steelblue")
axes[0].set_title("sin(x) + 0.5·sin(2x)")
axes[0].set_xlabel("x")
axes[0].set_ylabel("y")

axes[1].hist(np.random.normal(0, 1, 10000), bins=50, color="coral", edgecolor="white")
axes[1].set_title("標準常態分布（10000 個樣本）")
axes[1].set_xlabel("值")
axes[1].set_ylabel("頻率")

plt.tight_layout()
plt.show()
```

---

## 5. 常見問題與處理方式

| 問題 | 可能原因 | 建議處理 |
|------|----------|----------|
| `ValueError: operands could not be broadcast together` | 兩個 array 的形狀不相容 | 先印出兩個 array 的 `.shape`，找出哪裡不符合廣播規則；常見是 `(5,)` vs `(5,1)` |
| 改寫後結果差很多 | for 迴圈中有累積依賴（每步依賴上一步結果） | 不是所有 for 迴圈都能直接向量化；有累積依賴的用 `np.cumsum` 或考慮保留迴圈 |
| `assert_allclose` 報錯但結果看起來一樣 | 浮點精度問題 | 調整 `rtol=1e-6` 或 `atol=1e-8`，並向學生解釋浮點數的限制 |
| 學生的 for 迴圈太複雜，不知道怎麼改 | 邏輯依賴性高 | 鼓勵先改寫其中一部分（例如只把資料生成向量化），不一定要整段改 |
| 裝不到 NumPy / 版本太舊 | 環境問題 | `pip install --upgrade numpy`；建議用 conda 或 venv 統一環境 |
