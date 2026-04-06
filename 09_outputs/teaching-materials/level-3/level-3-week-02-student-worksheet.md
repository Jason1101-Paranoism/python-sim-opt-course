---
title: Level 3 週 02 — NumPy 陣列與向量化（學生講義）
type: teaching
level: 3
week: 2
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 02 學生講義
## 主題：NumPy — 陣列與向量化

---

## 本週你要學什麼

NumPy 是科學運算的核心工具。它讓你可以用極少的程式碼處理大量數值計算，速度比 Python for 迴圈快 10–100 倍甚至更多。

本週結束後，你要能做到：
1. 建立、操作 NumPy array
2. 理解向量化計算的意思
3. 把自己專題裡一段 for 迴圈改寫成 NumPy 版本

---

## 核心概念一：NumPy array 和 Python list 有什麼不同？

Python list 可以放任意型別的資料：
```python
my_list = [1, "hello", 3.14, True]  # 完全合法
```

NumPy array 要求所有元素型別相同：
```python
import numpy as np
my_array = np.array([1.0, 2.0, 3.0, 4.0])  # 全部都是 float64
```

**為什麼這讓 NumPy 更快？**

Python list 的每個元素都是一個 Python 物件，存在記憶體不同位置。NumPy array 的元素連續排列，型別固定，CPU 可以用最快的方式批次處理。

打個比方：
- Python list 是你一個一個把商品放進購物車
- NumPy array 是倉庫機器一次把整排商品掃進去

---

## 核心概念二：建立 NumPy Array

```python
import numpy as np

# 方法一：從 Python list 建立
a = np.array([1, 2, 3, 4, 5])
print(a)         # [1 2 3 4 5]
print(a.dtype)   # int64
print(a.shape)   # (5,)

# 方法二：arange（類似 range，但回傳 array）
b = np.arange(0, 10, 2)   # 從 0 到 10，步距 2
print(b)                   # [0 2 4 6 8]

# 方法三：linspace（均勻分割）
c = np.linspace(0, 1, 5)   # 從 0 到 1，共 5 個點
print(c)                    # [0.   0.25  0.5  0.75  1.  ]

# 方法四：zeros / ones
d = np.zeros((3, 4))        # 3 列 4 欄，全 0
e = np.ones(10)             # 長度 10，全 1

# 方法五：隨機數
f = np.random.normal(0, 1, size=1000)   # 標準常態，1000 個數
g = np.random.randint(0, 6, size=100)   # 0–5 的整數，100 個
```

---

## 核心概念三：索引、切片與形狀

```python
import numpy as np

# ── 1D array 的索引 ──
a = np.array([10, 20, 30, 40, 50])
print(a[0])     # 10（第一個）
print(a[-1])    # 50（最後一個）
print(a[1:4])   # [20 30 40]（第 2 到第 4）
print(a[::2])   # [10 30 50]（每隔一個）

# ── 2D array 的索引 ──
m = np.arange(12).reshape(3, 4)  # 3 列 4 欄
print(m)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

print(m[0, :])    # 第 0 列：[0 1 2 3]
print(m[:, 2])    # 第 2 欄：[2 6 10]
print(m[1:, 2:])  # 右下角 2×2 子矩陣

# ── 形狀操作 ──
print(m.shape)          # (3, 4)
print(m.T.shape)        # (4, 3)，轉置
flat = m.flatten()      # 展平成 1D
print(flat.shape)       # (12,)
```

---

## 核心概念四：向量化運算

向量化的意思是：對整個 array 做計算，不需要寫 for 迴圈。

```python
import numpy as np

x = np.array([1.0, 2.0, 3.0, 4.0])

# 基本數學（自動對每個元素）
print(x + 10)      # [11. 12. 13. 14.]
print(x * 2)       # [2.  4.  6.  8.]
print(x ** 2)      # [1.  4.  9. 16.]
print(np.sqrt(x))  # [1.  1.41 1.73 2. ]
print(np.sin(x))   # 對每個元素計算 sin

# 兩個 array 之間的運算
y = np.array([10.0, 20.0, 30.0, 40.0])
print(x + y)   # [11. 22. 33. 44.]
print(x * y)   # [10. 40. 90. 160.]  （element-wise）

# 統計
print(x.sum())    # 10.0
print(x.mean())   # 2.5
print(x.std())    # 標準差
print(x.max())    # 4.0
print(x.argmax()) # 3（最大值的索引）
```

### 向量化 vs for 迴圈的效率比較

```python
import numpy as np
import time

N = 1_000_000  # 100 萬次計算

# 方法一：for 迴圈
t0 = time.perf_counter()
result_loop = []
for i in range(N):
    result_loop.append(i * 0.001 + 1.5)
t1 = time.perf_counter()

# 方法二：NumPy 向量化
x = np.arange(N) * 0.001
result_numpy = x + 1.5
t2 = time.perf_counter()

print(f"for 迴圈：{(t1-t0)*1000:.0f} ms")
print(f"NumPy：   {(t2-t1)*1000:.0f} ms")
print(f"加速倍率：{(t1-t0)/(t2-t1):.0f}x")
```

執行後，你應該看到 NumPy 大約快 **50–150 倍**（依電腦而異）。

---

## 本週實作題

### 練習題 1：建立 array 與基本計算

完成以下程式碼，讓它能正確執行：

```python
import numpy as np

# (a) 建立一個從 1 到 100 的整數 array
a = # 你的程式碼

# (b) 計算 a 中所有「偶數位置」元素的平均值（索引 0, 2, 4, ...）
even_mean = # 你的程式碼
print(f"偶數位置平均：{even_mean}")

# (c) 建立一個 5×5 的矩陣，對角線為 1 到 5，其餘為 0
# 提示：查一下 np.diag()
matrix = # 你的程式碼
print("矩陣：\n", matrix)

# (d) 計算 0 到 2π 之間 100 個點的 sin(x) * cos(x)，存成 array
x = np.linspace(0, 2 * np.pi, 100)
y = # 你的程式碼
print(f"y 的最大值：{y.max():.4f}，最小值：{y.min():.4f}")
```

---

### 練習題 2：向量化改寫

下面是一段「計算每個位置的位能」的 for 迴圈程式。請用 NumPy 向量化改寫，並驗證結果一致：

```python
import numpy as np
import time

# ── 舊版（for 迴圈） ──
def potential_energy_v1(positions, spring_constant=1.0):
    """計算彈簧位能：E = 0.5 * k * x^2"""
    energies = []
    for x in positions:
        energies.append(0.5 * spring_constant * x ** 2)
    return energies

# 測試資料
positions = [0.1 * i for i in range(10000)]

t0 = time.perf_counter()
result_v1 = potential_energy_v1(positions)
t1 = time.perf_counter()
print(f"for 迴圈版本：{(t1-t0)*1000:.2f} ms")

# ── 你的任務：用 NumPy 改寫 ──
def potential_energy_v2(positions, spring_constant=1.0):
    """同上，NumPy 版本"""
    positions_array = np.array(positions)
    # 請在這裡寫 NumPy 版本的計算
    energies = # 你的程式碼
    return energies

t2 = time.perf_counter()
result_v2 = potential_energy_v2(positions)
t3 = time.perf_counter()
print(f"NumPy 版本：{(t3-t2)*1000:.2f} ms")

# 驗證結果一致
import numpy.testing as npt
npt.assert_allclose(result_v1, result_v2, rtol=1e-10)
print("✓ 結果一致！")
print(f"加速倍率：{(t1-t0)/(t3-t2):.0f}x")
```

---

### 練習題 3（挑戰）：改寫你的舊專題

把你舊專題中「含有 for 迴圈的核心計算」改寫成 NumPy 版本：

**要求**：
1. 原始版本（for 迴圈）與 NumPy 版本都要保留
2. 用 `time.perf_counter` 計時，報告加速倍率
3. 用 `np.allclose(result_v1, result_v2)` 確認結果一致
4. 如果結果不完全一致，說明為什麼（例如浮點精度）

```python
# 在這裡貼上你的改寫程式碼


```

---

## 本週成果

本週結束後，你應該能繳出：
- [ ] 練習題 1–2 的完整程式碼，並且可以執行
- [ ] 你自己舊專題的 NumPy 改寫版本（練習題 3）
- [ ] 改寫後的計時比較結果（加速幾倍）

如果你在改寫過程中遇到「這段迴圈好像沒辦法向量化」，記下來帶到 1-on-1 討論——很可能是有累積依賴（cumulative dependency），我們有別的方法處理。
