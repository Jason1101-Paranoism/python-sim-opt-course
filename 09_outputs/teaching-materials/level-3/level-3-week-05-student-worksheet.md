---
title: Level 3 週 05 — 最佳化入門（學生講義）
type: teaching
level: 3
week: 5
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 05 學生講義
## 主題：最佳化入門

---

## 本週你要學什麼

前四週你學會了：NumPy 加速計算、SciPy 求根與積分、SciPy 解 ODE。這些都是「給定條件，計算結果」。

本週你要學的是另一種問題：「**在所有可能的選擇中，找到讓結果最好的那個。**」這就是最佳化（Optimization）。

本週結束後，你要能做到：
1. 把一個實際問題轉化成「目標函數 + 決策變數 + 限制條件」
2. 用 `scipy.optimize.minimize_scalar` 解一維最佳化
3. 用 `scipy.optimize.minimize` 解多維最佳化

---

## 核心概念一：三個關鍵名詞

在解任何最佳化問題前，必須先想清楚三件事：

### 1. 目標函數（Objective Function）

你想讓什麼「最小」或「最大」？

目標函數是一個以決策變數為輸入、以某個衡量指標為輸出的函數。

例子：
- 總成本（你想最小化）
- 總利潤（你想最大化）
- 預測誤差（你想最小化）

### 2. 決策變數（Decision Variable）

你可以「調整」哪些東西來影響目標函數？

例子：
- 商品的定價 p
- 廣告預算 x₁, x₂
- 模型的參數 β, γ

### 3. 限制條件（Constraints）

有哪些規則不能違反？

例子：
- 預算不能超過 100 萬（不等式限制：x₁ + x₂ ≤ 100）
- 比例總和必須等於 1（等式限制：x₁ + x₂ = 1）
- 數量不能為負（邊界限制：x ≥ 0）

### 實際練習：對應以下問題

**問題**：你在生產某種零件，生產量 Q 每增加一個，生產成本增加 5 元，但倉儲成本也隨之增加。設總成本為 C(Q) = 5Q + 100/Q（Q > 0）。你想找到讓總成本最低的生產量。

- 目標函數：___
- 決策變數：___
- 限制條件：___

---

## 核心概念二：一維最佳化 `minimize_scalar`

當只有**一個**決策變數時，用 `minimize_scalar`：

```python
from scipy.optimize import minimize_scalar

def objective(x):
    return ...   # 你要最小化的函數

result = minimize_scalar(objective, bounds=(a, b), method='bounded')

print(result.x)    # 最佳的 x 值
print(result.fun)  # 最小的函數值
print(result.success)  # 是否成功（應為 True）
```

### 重要：最大化 = 最小化負值

`minimize_scalar` 只能最小化。如果你想最大化，把目標函數取負號：

```python
def neg_profit(price):
    profit = (price - cost) * demand(price)
    return -profit   # 取負號！

result = minimize_scalar(neg_profit, ...)
max_profit = -result.fun   # 結果記得換回來
optimal_price = result.x
```

---

## 核心概念三：完整範例——最佳定價

**問題**：你銷售一個軟體，單位成本 = 10 元。市場需求函數為 Q(p) = 100 - 2p（價格越高，銷量越低）。求讓利潤最大的售價。

**分析**：
- 目標函數：利潤 Π(p) = (p - 10) × Q(p) = (p - 10)(100 - 2p)（最大化）
- 決策變數：售價 p
- 限制條件：p ≥ 10（不能低於成本），需求 Q ≥ 0

```python
import numpy as np
from scipy.optimize import minimize_scalar
import matplotlib.pyplot as plt

cost = 10  # 單位成本

def neg_profit(p):
    """負利潤（用來最大化利潤）"""
    Q = 100 - 2 * p
    if Q <= 0:
        return 0   # 需求不能為負
    return -(p - cost) * Q  # 取負號

# 搜尋範圍：價格在 10 到 50 之間
result = minimize_scalar(neg_profit, bounds=(10, 50), method='bounded')

p_opt = result.x
Q_opt = 100 - 2 * p_opt
profit_opt = -result.fun   # 換回正號

print(f"最佳售價：p* = {p_opt:.2f} 元")
print(f"對應需求量：Q* = {Q_opt:.2f}")
print(f"最大利潤：Π* = {profit_opt:.2f} 元")

# 驗證：解析解應為 p* = (100 + 2*cost) / 4 = 30
print(f"\n解析解：p* = {(100 + 2*cost) / 4}")

# 視覺化
p = np.linspace(10, 50, 300)
profit = np.maximum(0, (p - cost) * (100 - 2*p))

plt.figure(figsize=(8, 4))
plt.plot(p, profit, color="steelblue", label="利潤函數")
plt.axvline(p_opt, color="red", linestyle="--",
            label=f"最佳售價 p*={p_opt:.1f}")
plt.scatter([p_opt], [profit_opt], color="red", s=100, zorder=5)
plt.xlabel("售價 p")
plt.ylabel("利潤 Π")
plt.title("最佳定價問題")
plt.legend()
plt.show()
```

---

## 核心概念四：多維最佳化 `minimize`

當有**多個**決策變數時，用 `minimize`：

```python
from scipy.optimize import minimize

def objective(x):
    # x 是一個 array：x[0], x[1], ...
    return ...

x0 = [初始猜測]  # 要和決策變數數量相同
result = minimize(objective, x0, method='Nelder-Mead')

print(result.x)     # 最佳的 [x₁, x₂, ...]
print(result.fun)   # 最小的函數值
```

### 加入限制條件

```python
# 等式限制：g(x) = 0
constraints = [{'type': 'eq', 'fun': lambda x: x[0] + x[1] - 100}]

# 不等式限制：h(x) >= 0
# constraints.append({'type': 'ineq', 'fun': lambda x: x[0] - 10})

# 決策變數邊界：(min, max)
bounds = [(0, 100), (0, 100)]

result = minimize(objective, x0, method='SLSQP',
                  constraints=constraints, bounds=bounds)
```

### 範例：廣告預算分配

```python
import numpy as np
from scipy.optimize import minimize

# 問題：分配 100 萬預算到平台 A 和 B，最大化總觸及率
# reach_A(x) = 50 * sqrt(x)，reach_B(x) = 80 * sqrt(x)

def neg_total_reach(budgets):
    xA, xB = budgets
    reach = 50 * np.sqrt(max(xA, 0)) + 80 * np.sqrt(max(xB, 0))
    return -reach   # 取負號（最大化 → 最小化負值）

constraints = [{'type': 'eq', 'fun': lambda b: b[0] + b[1] - 100}]
bounds = [(0, 100), (0, 100)]
x0 = [50, 50]

result = minimize(neg_total_reach, x0, method='SLSQP',
                  constraints=constraints, bounds=bounds)

xA, xB = result.x
print(f"平台 A：{xA:.2f} 萬")
print(f"平台 B：{xB:.2f} 萬")
print(f"總觸及率：{-result.fun:.2f}")
print(f"預算合計：{xA + xB:.2f}（應為 100）")
```

---

## 本週實作題

### 練習題 1：一維最佳化——生產成本

```python
import numpy as np
from scipy.optimize import minimize_scalar
import matplotlib.pyplot as plt

# 問題：工廠每天生產 Q 個產品
# 固定成本：100 元/天
# 變動成本：2Q 元（線性增加）
# 品質罰分：50/Q 元（產量太少時，品質管理成本高）
# 總成本：C(Q) = 100 + 2Q + 50/Q

def total_cost(Q):
    if Q <= 0:
        return 1e10
    return 100 + 2*Q + 50/Q

# (a) 先畫出 C(Q) 在 Q = 1 到 50 的圖形
Q_range = np.linspace(1, 50, 300)
# 你的程式碼：畫圖

# (b) 用 minimize_scalar 找最佳生產量
result = # 你的程式碼
Q_opt = result.x
print(f"最佳生產量：Q* = {Q_opt:.2f}")
print(f"最低總成本：C* = {result.fun:.2f}")

# (c) 解析解：dC/dQ = 2 - 50/Q^2 = 0 → Q* = sqrt(25) = 5
# 驗證你的數值解和解析解是否一致
print(f"解析解：Q* = {np.sqrt(25):.2f}")
```

---

### 練習題 2：多維最佳化——資源分配

```python
import numpy as np
from scipy.optimize import minimize

# 問題：你有 60 小時，要分配到學習數學（x₁）和程式（x₂）
# 數學技能：10 * sqrt(x₁)
# 程式技能：15 * sqrt(x₂)
# 限制條件：x₁ + x₂ = 60，x₁ >= 0，x₂ >= 0

def neg_total_skill(hours):
    x1, x2 = hours
    skill = 10 * np.sqrt(max(x1, 0)) + 15 * np.sqrt(max(x2, 0))
    return -skill

# (a) 定義限制條件和邊界
constraints = # 你的程式碼
bounds = # 你的程式碼
x0 = [30, 30]

# (b) 執行最佳化
result = minimize(neg_total_skill, x0, method='SLSQP',
                  constraints=constraints, bounds=bounds)

x1_opt, x2_opt = result.x
print(f"數學時數：{x1_opt:.2f} 小時")
print(f"程式時數：{x2_opt:.2f} 小時")
print(f"最大技能總值：{-result.fun:.2f}")

# (c) 驗證：限制條件是否成立？
print(f"時數合計：{x1_opt + x2_opt:.2f}（應為 60）")

# (d) 你的最佳分配合理嗎？程式技能比數學收益高，
# 所以最佳化結果應該把更多時數分配給程式。是這樣嗎？
```

---

### 練習題 3（挑戰）：最佳化你的 ODE 模型

把上週的 ODE 模型和本週的最佳化結合：

```python
# 問題：對你的 ODE 模型，找到讓某個指標最優的參數值
# 例如：
# - SIR 模型：找讓感染高峰最低的 β（假設你能控制 β，例如透過社交距離）
# - Logistic：找讓族群在第 20 天達到最大的增長率 r
# - 冷卻定律：找讓溫度在 30 分鐘後最接近 40°C 的冷卻係數 k

# 提示：
# 1. 定義一個目標函數，內部呼叫 odeint
# 2. 把你想優化的參數當成決策變數
# 3. 用 minimize_scalar 或 minimize 求解

# 在這裡寫你的程式：


```

---

## 本週成果

- [ ] 能說出「目標函數、決策變數、限制條件」各自的定義
- [ ] 完成練習題 1（一維最佳化 + 圖表）
- [ ] 完成練習題 2（多維最佳化 + 限制條件）
- [ ] 嘗試練習題 3（選填，但強烈建議）
- [ ] `result.success` 為 True（如果不是，找出原因）

---

## 工具速查表

| 工具 | 用途 | 方法 |
|------|------|------|
| `minimize_scalar` | 一維最佳化（無限制） | `method='brent'`（預設） |
| `minimize_scalar` | 一維最佳化（有界） | `method='bounded'`，加 `bounds=(a,b)` |
| `minimize` | 多維最佳化 | `method='Nelder-Mead'`（無梯度）、`'SLSQP'`（有限制條件） |

記住：**最大化 = 最小化負值**
