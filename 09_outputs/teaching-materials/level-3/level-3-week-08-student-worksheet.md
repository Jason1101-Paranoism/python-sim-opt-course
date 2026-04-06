---
title: Level 3 週 08 — 專題實作：最佳化與敏感度分析（學生講義）
type: teaching
level: 3
week: 8
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 08：專題實作 — 最佳化與敏感度分析

## 本週主題

你的專題程式已經可以模擬和計算了。這週要讓它更進一步：**不只觀察結果，而是自動找到最好的解**。

本週兩件事：
1. **最佳化**：用 `scipy.optimize` 自動找讓目標達到最大/最小的參數值
2. **敏感度分析**：改變一個參數，觀察最佳解如何隨之變化

---

## 核心概念一：目標函數設計

### 從問題到程式的翻譯過程

「最佳化」在程式中的對應關係：

| 你說的話 | 程式中的對應 |
|----------|-------------|
| 「我想最小化感染高峰」 | 目標函數回傳 `peak_infected_count` |
| 「我想最大化效率」 | 目標函數回傳 `-efficiency`（加負號） |
| 「接種比例必須在 0% 到 100% 之間」 | `bounds=[(0.0, 1.0)]` |
| 「總費用不能超過 1000 元」 | `constraints={'type': 'ineq', 'fun': lambda x: 1000 - total_cost(x)}` |

### `scipy.optimize.minimize` 的基本格式

```python
from scipy.optimize import minimize

# 最基本的呼叫方式
result = minimize(
    fun=my_objective,    # 目標函數：輸入決策變數，輸出一個數字
    x0=[1.0, 2.0],       # 初始猜測值（決策變數的起始點）
    method='Nelder-Mead' # 最佳化方法（無需梯度的搜尋）
)

print(result.x)    # 最佳決策變數值
print(result.fun)  # 最佳目標函數值
print(result.success)  # 是否成功收斂
```

---

## 核心概念二：局部最佳解的陷阱

`minimize` 從初始點出發，找「最近的谷底」，不保證找到全局最小值。

```python
import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# 有兩個谷底的函數
def tricky_function(x):
    """這個函數在 x=-1.5 附近有局部最小值，在 x=2.0 附近有全局最小值"""
    return x[0]**4 - 4 * x[0]**3 - 2 * x[0]**2 + 12 * x[0]

# 視覺化
x_range = np.linspace(-2, 4, 300)
y_range = [tricky_function([x]) for x in x_range]

plt.figure(figsize=(8, 4))
plt.plot(x_range, y_range, 'b-')
plt.xlabel('x')
plt.ylabel('f(x)')
plt.title('有多個局部最小值的函數')
plt.grid(True)

# 從不同初始點出發，看結果
for x0_val in [-1.5, 0.0, 1.0, 3.5]:
    res = minimize(tricky_function, [x0_val], method='Nelder-Mead')
    plt.plot(res.x[0], res.fun, 'ro', markersize=8)
    print(f"初始點 x0={x0_val:.1f} → 最佳解 x={res.x[0]:.4f}，f(x)={res.fun:.4f}")

plt.show()
```

**觀察**：同一個函數，不同的 x0 會找到不同的「最佳解」。
**啟示**：最佳化前，先畫出目標函數的圖，了解它大概長什麼樣子。

---

## 完整範例程式：SIR 模型接種策略最佳化

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import solve_ivp
from scipy.optimize import minimize_scalar

# ================================================================
# 模型定義
# ================================================================
def sir_model(t, y, beta, gamma, N):
    """SIR 傳染病模型"""
    S, I, R = y
    dS_dt = -beta * S * I / N
    dI_dt =  beta * S * I / N - gamma * I
    dR_dt =  gamma * I
    return [dS_dt, dI_dt, dR_dt]

# ================================================================
# 目標函數：給定接種比例 v，回傳感染高峰人數
# ================================================================
def objective_vaccination(v, beta=0.3, gamma=0.1, N=10000, I0=10):
    """
    目標函數：計算接種比例 v 對應的感染高峰人數
    
    我們要「最小化」這個值，所以這就是我們的目標函數。
    （不需要加負號，因為我們本來就想最小化感染人數）
    """
    vaccinated = v * N            # 接種人數
    S0 = max(0, N - I0 - vaccinated)  # 易感者（不能為負）
    R0_init = vaccinated          # 接種者已免疫，視為已恢復
    
    if S0 == 0:
        return float(I0)  # 沒有易感者，高峰就是初始感染者
    
    y0 = [S0, I0, R0_init]
    sol = solve_ivp(
        sir_model, (0, 300), y0,
        args=(beta, gamma, N),
        dense_output=True, max_step=1.0
    )
    t_eval = np.linspace(0, 300, 1000)
    y_eval = sol.sol(t_eval)
    return y_eval[1].max()

# ================================================================
# 步驟 1：先視覺化目標函數
# ================================================================
v_values = np.linspace(0, 0.95, 50)
peak_values = [objective_vaccination(v) for v in v_values]

plt.figure(figsize=(8, 4))
plt.plot(v_values * 100, peak_values, 'b-', linewidth=2)
plt.xlabel('接種比例（%）')
plt.ylabel('感染高峰人數')
plt.title('目標函數視覺化：接種比例 vs 感染高峰')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('objective_function.png', dpi=150)
plt.show()
print("觀察：接種比例增加時，感染高峰如何變化？")

# ================================================================
# 步驟 2：找最佳接種比例
# ================================================================
# 使用 minimize_scalar（一維最佳化，比 minimize 更精確）
result = minimize_scalar(
    objective_vaccination,
    bounds=(0.0, 0.99),
    method='bounded'
)

print("\n" + "=" * 50)
print("最佳化結果：")
print(f"  最佳接種比例：{result.x * 100:.2f}%")
print(f"  對應感染高峰：{result.fun:.0f} 人（佔總人口 {result.fun/10000*100:.1f}%）")
print(f"  不接種時的高峰：{objective_vaccination(0):.0f} 人")
print(f"  接種效益：減少 {objective_vaccination(0) - result.fun:.0f} 人感染")

# ================================================================
# 步驟 3：敏感度分析 — 接觸率 β 對最佳接種比例的影響
# ================================================================
beta_range = np.linspace(0.15, 0.60, 15)
optimal_vaccinations = []
minimum_peaks = []

for beta in beta_range:
    res = minimize_scalar(
        lambda v: objective_vaccination(v, beta=beta),
        bounds=(0.0, 0.99),
        method='bounded'
    )
    optimal_vaccinations.append(res.x * 100)  # 轉換為百分比
    minimum_peaks.append(res.fun)

# 畫敏感度分析圖
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

ax1.plot(beta_range, optimal_vaccinations, 'g-o', linewidth=2)
ax1.set_xlabel('接觸率 β')
ax1.set_ylabel('最佳接種比例（%）')
ax1.set_title('β 對最佳接種比例的影響')
ax1.grid(True, alpha=0.3)
ax1.axvline(x=0.3, color='gray', linestyle='--', alpha=0.7, label='基準值 β=0.3')
ax1.legend()

ax2.plot(beta_range, minimum_peaks, 'r-o', linewidth=2)
ax2.set_xlabel('接觸率 β')
ax2.set_ylabel('最小可達感染高峰（人）')
ax2.set_title('β 對最佳策略下感染高峰的影響')
ax2.grid(True, alpha=0.3)
ax2.axvline(x=0.3, color='gray', linestyle='--', alpha=0.7, label='基準值 β=0.3')
ax2.legend()

plt.suptitle('SIR 模型敏感度分析（γ=0.1, N=10000）', fontsize=13)
plt.tight_layout()
plt.savefig('sensitivity_analysis.png', dpi=150)
plt.show()

# 敏感度分析摘要
print("\n敏感度分析摘要：")
print(f"  β = {beta_range[0]:.2f}：最佳接種比例 = {optimal_vaccinations[0]:.1f}%")
print(f"  β = {beta_range[-1]:.2f}：最佳接種比例 = {optimal_vaccinations[-1]:.1f}%")
print(f"  結論：接觸率每增加 0.1，最佳接種比例約增加 {(optimal_vaccinations[-1]-optimal_vaccinations[0])/(beta_range[-1]-beta_range[0])*0.1:.1f}%")
```

---

## 練習題

### 練習 1：理解最佳化結果

執行上面的程式，回答以下問題：

a) 最佳接種比例是多少？這個數字的意義是什麼（佔總人口的比例）？

b) 在最佳接種比例下，感染高峰比不接種時少了多少人？降低了多少百分比？

c) 為什麼不把接種比例設為 99%？（提示：考慮實際可行性，但也思考：數學上接種越多一定越好嗎？）

---

### 練習 2：套用到你的專題

根據你的計畫書，完成目標函數的設計：

```python
def my_objective(x, param1=???, param2=???):
    """
    我的目標函數
    
    輸入：x = [決策變數1, 決策變數2, ...]
    輸出：一個純量數字（我要最小化這個值）
    
    如果我要最大化，我回傳 -（我的指標）
    """
    # 第一步：用 x 計算你的模型
    result = ???
    
    # 第二步：從模型結果提取你的目標指標
    objective_value = ???
    
    return objective_value  # 必須是一個純量（一個數字）

# 測試目標函數
print(my_objective([???]))  # 手動輸入一個 x 值，確認有輸出
```

---

### 練習 3：敏感度分析設計

設計你的專題的敏感度分析。填入下表：

| 要改變的參數 | 範圍 | 要觀察的指標 |
|-------------|------|-------------|
| 參數 1（___） | ___ 到 ___ | 最佳解 ___ 如何變化 |
| 參數 2（___，選做） | ___ 到 ___ | 最佳目標值如何變化 |

---

## 本週目標

課堂結束時，你應該有：
- 一個可以被 `minimize` 呼叫的目標函數（能回傳純量）
- 至少一次最佳化執行結果（有 `result.x` 和 `result.fun`）
- 至少一張敏感度分析圖（一個參數的改變如何影響最佳解）
