---
title: Level 3 週 03 — SciPy 基本數值方法（學生講義）
type: teaching
level: 3
week: 3
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 03 學生講義
## 主題：SciPy — 基本數值方法（求根與數值積分）

---

## 本週你要學什麼

有些問題不是「計算出答案」，而是「找到讓某個條件成立的值」或「算出一個曲線下的面積」。這類問題就需要**數值方法**。

本週你要學的兩個工具：
- `scipy.optimize.brentq`：數值**求根**——找讓函數等於零的 x
- `scipy.integrate.quad`：數值**積分**——計算函數在某區間內的面積

---

## 核心概念一：什麼是數值求根？

**問題**：解方程式 `cos(x) = x`，x 等於多少？

這個方程式沒有「整數解」，也沒有簡單的代數公式。你可以改寫成：

```
f(x) = cos(x) - x = 0
```

然後問：讓 f(x) = 0 的 x 是多少？這就叫求根（root finding）。

**直覺：二分法**

如果你知道 `f(0) > 0`，`f(2) < 0`，那麼在 `[0, 2]` 之間**一定存在**一個零點。

二分法的做法：
1. 取中點 `m = 1`
2. 算 `f(1)`，如果 > 0，把左界換成 1；如果 < 0，把右界換成 1
3. 重複，區間越縮越小，直到夠精確

`brentq` 是二分法的加速版，保證精確收斂。

---

## 核心概念二：使用 `scipy.optimize.brentq`

### 基本語法

```python
from scipy.optimize import brentq

def f(x):
    return 你的函數

root = brentq(f, a, b)  # a 和 b 是區間，要求 f(a) 和 f(b) 異號
```

### 完整範例：找 cos(x) = x 的解

```python
import numpy as np
from scipy.optimize import brentq
import matplotlib.pyplot as plt

# 定義函數（把方程式移項，讓右邊等於 0）
def f(x):
    return np.cos(x) - x   # 我們要找讓這個 = 0 的 x

# 第一步：畫圖，確認零點大概在哪
x = np.linspace(-1, 3, 300)
plt.figure(figsize=(7, 4))
plt.plot(x, f(x), label="cos(x) - x", color="steelblue")
plt.axhline(0, color="gray", linestyle="--", label="y = 0")
plt.xlabel("x")
plt.ylabel("f(x)")
plt.title("找 cos(x) = x 的解")
plt.legend()
plt.show()

# 從圖可以看到，零點在 x = 0 和 x = 1 之間
# 確認 f(0) 和 f(1) 異號
print(f"f(0) = {f(0):.4f}")   # 正的
print(f"f(1) = {f(1):.4f}")   # 負的

# 第二步：用 brentq 求根
root = brentq(f, 0, 1)
print(f"\n解：x = {root:.10f}")

# 第三步：驗證
print(f"cos({root:.6f}) = {np.cos(root):.10f}")
print(f"差值：{abs(np.cos(root) - root):.2e}")  # 幾乎是 0
```

### 重要：兩個常見錯誤

**錯誤 1：區間包含多個零點**
```python
# 如果 f 在 [0, 10] 之間有多個零點，brentq 只會找到一個
# 記得先畫圖確認你要找的零點在哪個區間
```

**錯誤 2：區間兩端同號**
```python
# 這樣會報錯：
root = brentq(f, 0, 0.5)   # 如果 f(0) 和 f(0.5) 都是正的

# 正確做法：先確認
print(f(0), f(0.5))   # 確認它們異號再用 brentq
```

---

## 核心概念三：什麼是數值積分？

**問題**：`sin(x)` 從 `x = 0` 到 `x = π` 圍成的面積是多少？

解析解告訴我們答案是 2（微積分基本定理）。但如果函數很複雜，或者是從資料來的，就無法用公式算。

**數值積分的直覺（黎曼和）**：

把面積切成許多小長方形，每個長方形的寬度很小，高度等於函數值，加起來就是近似面積。切得越細，誤差越小。

`scipy.integrate.quad` 使用更聰明的方法（自適應 Gauss-Kronrod），比黎曼和精確得多，通常誤差 < 10⁻⁸。

---

## 核心概念四：使用 `scipy.integrate.quad`

### 基本語法

```python
from scipy.integrate import quad

def f(x):
    return 你要積分的函數

result, error = quad(f, a, b)  # 計算 f 從 a 到 b 的定積分
```

注意：`quad` 回傳**兩個值**：結果和誤差估計。

### 完整範例：計算物體位移

```python
import numpy as np
from scipy.integrate import quad
import matplotlib.pyplot as plt

# 速度函數 v(t) = 3t² - 2t + 1
# 位移 = ∫₀^5 v(t) dt
# 解析解：[t³ - t² + t]₀^5 = 125 - 25 + 5 = 105

def velocity(t):
    return 3*t**2 - 2*t + 1

# 數值積分
displacement, error = quad(velocity, 0, 5)
print(f"數值積分結果：{displacement:.6f} m")
print(f"解析解：105.0 m")
print(f"數值誤差估計：{error:.2e}")

# 視覺化積分區域
t = np.linspace(0, 5, 300)
v = velocity(t)

plt.figure(figsize=(8, 4))
plt.plot(t, v, color="steelblue", label="v(t) = 3t² - 2t + 1")
plt.fill_between(t, v, alpha=0.3, color="steelblue",
                 label=f"位移 = {displacement:.1f} m")
plt.xlabel("時間 (s)")
plt.ylabel("速度 (m/s)")
plt.title("速度積分 = 位移")
plt.legend()
plt.show()
```

---

## 什麼時候用數值方法？

| 問題類型 | 是否需要數值方法 | 工具 |
|----------|-----------------|------|
| 解簡單代數方程（如 2x+3=0） | 不需要，直接解 | 不用 |
| 解複雜方程（如 cos(x)=x） | 需要 | `brentq` |
| 找供需均衡點 | 需要 | `brentq` |
| 計算 x² 從 0 到 1 的面積 | 不需要（=1/3） | 不用 |
| 計算 sin(x²) 的積分（無解析解） | 需要 | `quad` |
| 計算實驗量測的累積量 | 需要 | `quad` 或 `trapz` |

---

## 本週實作題

### 練習題 1：數值求根

```python
import numpy as np
from scipy.optimize import brentq
import matplotlib.pyplot as plt

# 問題：找方程式 x^3 - x - 1 = 0 的實數根
# 提示：先畫圖，找出根大概在哪個區間

def f(x):
    return x**3 - x - 1

# (a) 畫出 f(x) 在 [-2, 2] 的圖形
x = np.linspace(-2, 2, 300)
# 你的程式碼：畫圖

# (b) 從圖上判斷根大概在哪個區間
# 你的分析：根大約在 x = __ 和 x = __ 之間

# (c) 用 brentq 精確求根
root = # 你的程式碼：呼叫 brentq
print(f"根：x = {root:.8f}")

# (d) 驗證：代入 f(root)，結果應該非常接近 0
print(f"驗證：f({root:.4f}) = {f(root):.2e}")
```

---

### 練習題 2：數值積分

```python
import numpy as np
from scipy.integrate import quad

# 問題一：計算 e^(-x²) 從 0 到 ∞ 的積分
# 這個積分沒有簡單的解析解，但答案已知為 √π / 2 ≈ 0.8862...

def f1(x):
    return np.exp(-x**2)

# 計算積分（把「無窮」用 np.inf 表示）
result1, err1 = quad(f1, 0, np.inf)
print(f"∫₀^∞ e^(-x²) dx = {result1:.8f}")
print(f"解析答案 √π/2 = {np.sqrt(np.pi)/2:.8f}")
print(f"誤差：{abs(result1 - np.sqrt(np.pi)/2):.2e}")

# 問題二：計算機率
# 假設某種零件的壽命服從以下機率密度：
# f(t) = 0.5 * exp(-0.5*t)，t > 0（指數分布，參數 0.5）
# 問：零件在前 3 年內壞掉的機率是多少？

def lifetime_pdf(t):
    return 0.5 * np.exp(-0.5 * t)

prob, err = quad(lifetime_pdf, 0, 3)
print(f"\n前 3 年壞掉的機率：{prob:.4f}（{prob*100:.2f}%）")
# 理論答案：1 - e^(-1.5) ≈ 0.7769
```

---

### 練習題 3（進階）：應用到你的專題

根據你的舊專題，嘗試以下其中一個：

**選項 A：求根應用**
- 找你的模型中某個「等式成立」的條件
- 例如：「什麼參數值下，我的模擬結果等於某個目標值？」

**選項 B：積分應用**
- 計算你的模型中某個量的累積值
- 例如：「在整個模擬時間內，某個資源的總消耗量是多少？」

```python
# 在這裡寫你的程式


```

---

## 本週成果

- [ ] 完成練習題 1（brentq 求根 + 驗證）
- [ ] 完成練習題 2（quad 積分兩個問題）
- [ ] 能說出：什麼時候用求根、什麼時候用積分
- [ ] 嘗試練習題 3（選填，但強烈建議）
