---
title: Level 3 週 05 — 最佳化入門（教師教案）
type: teaching
level: 3
week: 5
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 05 教師教案
## 主題：最佳化入門

---

## 1. 本週學習目標

1. 學生能正確說出「目標函數」、「決策變數」、「限制條件」三個概念的定義和對應關係
2. 學生能用 `scipy.optimize.minimize_scalar` 完成一維最佳化，並解讀最佳解的意義
3. 學生能用 `scipy.optimize.minimize` 完成多維最佳化（至少 2 個決策變數），理解初始點的重要性
4. 學生能完成一個「找最大／最小值」的小程式，並能用語言說明最佳解在問題背景下的意義

---

## 2. 課堂時間分配表（120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 從 ODE 參數實驗引出「最佳化」的問題意識 |
| 概念講解 | 25 分鐘 | 目標函數、決策變數、限制條件；最佳化 vs 模擬 |
| 範例示範（Live coding） | 30 分鐘 | `minimize_scalar` 一維最佳化 + `minimize` 多維最佳化 |
| 學生實作 | 35 分鐘 | 學生完成一個自選的最佳化程式 |
| 回顧與預告 | 20 分鐘 | 討論常見陷阱（局部最小值），預告 Week 6 選題 |

---

## 3. 各段落執行細節

### 3.1 暖身回顧（10 分鐘）

**引入問題**（接續 Week 4 的 ODE 內容）：
> 「上週你的 ODE 模型，感染高峰在某個 β 值時最高，某個 β 值時較低。如果我想找到『讓感染高峰最低的那個 β』，你怎麼做？」

讓學生說說：
- 直覺做法：試一排 β 值，找最小的（窮舉搜尋）
- 問題：搜尋範圍大、精度不夠
- 今天要學的：SciPy 的最佳化工具，比窮舉快且精確

**另一個日常例子**：
> 「你開網路商店，想定一個讓利潤最大的價格。太低，利潤少；太高，沒人買。怎麼找最佳價格？」

這就是一個典型的一維最佳化問題。

---

### 3.2 概念講解（25 分鐘）

**概念一：三個核心概念（15 分鐘）**

在白板上畫出三個框：

```
┌─────────────────────────────────────────────────┐
│  目標函數（Objective Function）                  │
│  你想讓什麼「最小」或「最大」？                  │
│  例：f(x) = 成本、誤差、感染人數                │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│  決策變數（Decision Variable）                   │
│  你可以調整哪些東西？                            │
│  例：x = 價格、劑量、投資比例                    │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│  限制條件（Constraints）（可有可無）             │
│  有哪些不能違反的規則？                          │
│  例：x ≥ 0、總和 = 1、不能超過預算              │
└─────────────────────────────────────────────────┘
```

用三個實際例子讓學生練習對應：

**例 A**：你在種植作物，施肥量越多收成越高，但超過某個量反而有害。你想找最佳施肥量。
- 目標函數：收成（最大化）
- 決策變數：施肥量 x
- 限制條件：x ≥ 0，x ≤ 最大施肥量

**例 B**：你要選一個函數的最低點：`f(x) = (x - 3)^2 + 2`
- 目標函數：f(x)（最小化）
- 決策變數：x
- 限制條件：無

**例 C**：你在分配廣告預算到兩個平台，預算固定 100 萬，想最大化總觸及率。
- 目標函數：觸及率（最大化）
- 決策變數：x₁（平台 A 的預算）、x₂（平台 B 的預算）
- 限制條件：x₁ + x₂ = 100，x₁ ≥ 0，x₂ ≥ 0

**概念二：最佳化 vs 模擬（10 分鐘）**

| | 模擬（ODE） | 最佳化 |
|-|------------|--------|
| 問題 | 「這個系統接下來怎麼走？」 | 「哪個設定讓結果最好？」 |
| 輸入 | 初始條件、參數 | 目標函數、初始猜測 |
| 輸出 | 時間序列 | 最佳決策變數值 |
| 工具 | `odeint` | `minimize_scalar`, `minimize` |

**強調**：這兩者可以結合——先用 ODE 模擬系統，再用最佳化找最好的模型參數。這就是 Level 3 期末專題的核心流程。

---

### 3.3 範例示範（Live Coding）（30 分鐘）

**第一段（15 分鐘）：一維最佳化 `minimize_scalar`**

```python
import numpy as np
from scipy.optimize import minimize_scalar
import matplotlib.pyplot as plt

# ── 問題一：找函數最小值 ───────────────────────────────
def f(x):
    """有雜訊的目標函數：(x-2)^2 + sin(5x)"""
    return (x - 2)**2 + np.sin(5 * x)

# 視覺化函數
x = np.linspace(-1, 5, 300)
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(x, f(x), color="steelblue")
axes[0].set_xlabel("x")
axes[0].set_ylabel("f(x)")
axes[0].set_title("目標函數")

# 找最小值（無限制）
result = minimize_scalar(f, bounds=(-1, 5), method='bounded')
print(f"最小值在 x = {result.x:.6f}")
print(f"最小值 f(x) = {result.fun:.6f}")
print(f"最佳化成功：{result.success}")

axes[0].scatter([result.x], [result.fun], color="red", s=100, zorder=5,
               label=f"最小值 ({result.x:.2f}, {result.fun:.2f})")
axes[0].legend()

# ── 問題二：最佳定價問題 ──────────────────────────────
# 需求函數：Q(p) = 100 - 2p（價格越高，需求越低）
# 成本：每單位成本 c = 10
# 利潤：Π(p) = (p - c) × Q(p) = (p - 10)(100 - 2p)

c = 10  # 單位成本

def neg_profit(p):
    """負利潤（因為 minimize 是最小化，我們取負值來最大化利潤）"""
    Q = 100 - 2 * p
    if Q < 0:
        return 0  # 需求不能為負
    profit = (p - c) * Q
    return -profit  # 取負號

result2 = minimize_scalar(neg_profit, bounds=(c, 50), method='bounded')
p_opt = result2.x
Q_opt = 100 - 2 * p_opt
profit_opt = (p_opt - c) * Q_opt

print(f"\n最佳定價：p* = {p_opt:.2f}")
print(f"最佳需求量：Q* = {Q_opt:.2f}")
print(f"最大利潤：Π* = {profit_opt:.2f}")

# 視覺化利潤函數
p = np.linspace(c, 50, 300)
profit = np.maximum(0, (p - c) * (100 - 2 * p))
axes[1].plot(p, profit, color="coral", label="利潤函數")
axes[1].axvline(p_opt, color="green", linestyle="--",
               label=f"最佳價格 p*={p_opt:.1f}")
axes[1].set_xlabel("價格 p")
axes[1].set_ylabel("利潤 Π")
axes[1].set_title("最佳定價問題")
axes[1].legend()

plt.tight_layout()
plt.show()
```

**引導學生觀察**：
- 最大化利潤 → 取負號最小化
- `bounds` 限制搜尋範圍，避免搜到無意義的負價格
- `result.success` 要檢查，確認最佳化成功

**第二段（15 分鐘）：多維最佳化 `minimize`**

```python
import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# ── Rosenbrock 函數（測試用的標準函數） ───────────────
# f(x, y) = (1-x)^2 + 100(y-x^2)^2
# 最小值在 (x, y) = (1, 1)，f = 0

def rosenbrock(xy):
    """多維函數：接受 array xy = [x, y]"""
    x, y = xy
    return (1 - x)**2 + 100 * (y - x**2)**2

# 初始猜測點
x0 = [-1.0, 0.5]

result = minimize(rosenbrock, x0, method='Nelder-Mead')
print(f"最小值位置：x={result.x[0]:.6f}, y={result.x[1]:.6f}")
print(f"最小值：f = {result.fun:.2e}")
print(f"疊代次數：{result.nit}")

# ── 廣告預算分配問題 ──────────────────────────────────
# 兩個廣告平台 A, B，預算 = 100
# 觸及率函數（邊際遞減）：
#   reach_A(x) = 50 * sqrt(x)
#   reach_B(x) = 80 * sqrt(x)
# 目標：最大化總觸及率，限制：x_A + x_B = 100，x_A, x_B >= 0

def neg_total_reach(budgets):
    x_A, x_B = budgets
    reach_A = 50 * np.sqrt(max(x_A, 0))
    reach_B = 80 * np.sqrt(max(x_B, 0))
    return -(reach_A + reach_B)

# 限制條件：x_A + x_B = 100
constraints = [{'type': 'eq', 'fun': lambda b: b[0] + b[1] - 100}]

# 決策變數的範圍：0 <= x_A, x_B <= 100
bounds = [(0, 100), (0, 100)]

# 初始猜測：平均分配
x0 = [50, 50]

result = minimize(neg_total_reach, x0, method='SLSQP',
                  constraints=constraints, bounds=bounds)

xA_opt, xB_opt = result.x
print(f"\n廣告預算分配：A = {xA_opt:.2f}，B = {xB_opt:.2f}")
print(f"總觸及率 = {-result.fun:.2f}")
print(f"限制條件驗證：{xA_opt + xB_opt:.2f} = 100")
```

**引導學生觀察**：
- 多維最佳化的決策變數是一個 array `xy = [x, y]`
- 限制條件用字典格式傳入：`type` 為 `'eq'`（等式）或 `'ineq'`（不等式）
- `bounds` 限制每個變數的範圍

---

### 3.4 學生實作（35 分鐘）

學生完成 worksheet 的練習題：
- 練習題 1：一維最佳化（利潤或成本問題）
- 練習題 2：加限制條件的資源分配問題
- 練習題 3（挑戰）：用最佳化找 ODE 模型的最佳參數

**教師巡視重點**：
- 目標函數定義是否正確（最大化問題要取負號）
- 初始猜測是否合理（過於偏離解，可能收斂失敗）
- `result.success` 是否為 True（如果 False，需要調整初始點或方法）

---

### 3.5 回顧與預告（20 分鐘）

**討論常見陷阱（10 分鐘）**：
- **局部最小值問題**：展示一個多峰函數，說明 `minimize` 從不同初始點可能找到不同的解
- **梯度資訊**：說明 `Nelder-Mead` 不需要梯度（適合複雜或不可微的目標函數），但收斂較慢；`L-BFGS-B` 利用梯度，速度快但需要函數可微
- **最佳化結果要驗證**：永遠不要只看 `result.x`，要代入目標函數確認確實是最優的

**預告 Week 6（10 分鐘）**：
- 下週：高級專題選題與設計
- 你在 Week 1–5 學到了所有 Level 3 的核心工具，現在是時候把它們整合到一個真正的專題裡
- 課前準備：思考你的期末專題方向（新題目或升級舊題目？哪些工具會用到？）

---

## 4. 關鍵程式碼範例（完整可執行）

```python
"""
Level 3 Week 05 — 最佳化完整示範：一維 + 多維 + 結合 ODE
執行環境：Python 3.8+，需安裝 scipy, numpy, matplotlib
"""
import numpy as np
from scipy.optimize import minimize_scalar, minimize
from scipy.integrate import odeint
import matplotlib.pyplot as plt

print("=" * 55)
print("Part 1：一維最佳化——庫存成本最小化")
print("=" * 55)

# 模型：EOQ（經濟訂貨量）
# 每年需求量 D，每次訂貨固定成本 K，持有成本率 h，單位成本 c
D = 1000  # 年需求量
K = 50    # 每次訂貨成本
h = 0.2   # 持有成本率
c = 10    # 單位成本

def total_cost(Q):
    """年度庫存總成本（訂貨成本 + 持有成本）"""
    if Q <= 0:
        return 1e10  # 防止 Q<=0 的情況
    ordering_cost = (D / Q) * K
    holding_cost = (Q / 2) * h * c
    return ordering_cost + holding_cost

result = minimize_scalar(total_cost, bounds=(1, 500), method='bounded')
Q_opt = result.x
print(f"最佳訂貨量（EOQ）：Q* = {Q_opt:.2f} 單位")
print(f"最小年度成本：{result.fun:.2f}")

# EOQ 解析解：Q* = sqrt(2DK / (hc))
Q_analytical = np.sqrt(2 * D * K / (h * c))
print(f"解析解 EOQ = {Q_analytical:.2f}（驗證）")

# 視覺化
Q_range = np.linspace(10, 300, 300)
costs = [total_cost(q) for q in Q_range]

plt.figure(figsize=(8, 4))
plt.plot(Q_range, costs, color="steelblue", label="總成本")
plt.axvline(Q_opt, color="red", linestyle="--", label=f"最佳訂貨量 Q*={Q_opt:.0f}")
plt.xlabel("訂貨量 Q")
plt.ylabel("年度總成本")
plt.title("EOQ 庫存模型")
plt.legend()
plt.tight_layout()
plt.show()

print("\n" + "=" * 55)
print("Part 2：多維最佳化——結合 ODE 模型")
print("=" * 55)

# 問題：找 SIR 模型的最佳參數，讓模型和觀測數據最吻合
# （參數擬合問題）

# 模擬「觀測資料」（用已知 β=0.3, γ=0.1 生成，加入雜訊）
N = 1000
t_obs = np.linspace(0, 100, 50)

def sir(y, t, beta, gamma):
    S, I, R = y
    return [-beta*S*I/N, beta*S*I/N - gamma*I, gamma*I]

# 真實參數
true_sol = odeint(sir, [N-5, 5, 0], t_obs, args=(0.3, 0.1))
I_observed = true_sol[:, 1] + np.random.normal(0, 5, len(t_obs))  # 加雜訊
I_observed = np.maximum(I_observed, 0)  # 不能為負

def objective(params):
    """目標函數：模型預測 vs 觀測資料的 MSE"""
    beta, gamma = params
    if beta <= 0 or gamma <= 0:
        return 1e10
    try:
        sol = odeint(sir, [N-5, 5, 0], t_obs, args=(beta, gamma))
        I_pred = sol[:, 1]
        mse = np.mean((I_pred - I_observed)**2)
        return mse
    except Exception:
        return 1e10

# 最佳化：找讓 MSE 最小的 β 和 γ
x0 = [0.2, 0.15]  # 初始猜測
result = minimize(objective, x0, method='Nelder-Mead',
                  options={'maxiter': 5000, 'xatol': 1e-6})

beta_fit, gamma_fit = result.x
print(f"擬合參數：β = {beta_fit:.4f}（真實值 0.3）")
print(f"擬合參數：γ = {gamma_fit:.4f}（真實值 0.1）")
print(f"MSE = {result.fun:.2f}")

# 視覺化擬合結果
t_fine = np.linspace(0, 100, 300)
sol_fit = odeint(sir, [N-5, 5, 0], t_fine, args=(beta_fit, gamma_fit))

plt.figure(figsize=(8, 4))
plt.scatter(t_obs, I_observed, s=15, color="gray", label="觀測資料", zorder=5)
plt.plot(t_fine, sol_fit[:, 1], color="coral",
         label=f"擬合曲線（β={beta_fit:.3f}, γ={gamma_fit:.3f}）")
plt.xlabel("天")
plt.ylabel("感染人數")
plt.title("SIR 模型參數擬合")
plt.legend()
plt.tight_layout()
plt.show()
```

---

## 5. 常見問題與處理方式

| 問題 | 可能原因 | 建議處理 |
|------|----------|----------|
| `minimize` 的結果 `success = False` | 初始點偏離解太遠、或步數不夠 | 嘗試不同的初始猜測值；或增加 `options={'maxiter': 10000}`；也可以先用 `minimize_scalar` 確認大概範圍 |
| 想最大化，但 `minimize` 找到了最小值 | 忘記取負號 | 明確說明：`minimize` 只做最小化，最大化要在目標函數前加負號，最後結果記得換回來 |
| 多維最佳化中決策變數的格式弄錯 | 函數接收的是 array，但學生以為是多個參數 | 確認函數簽名：`def f(x): x_1, x_2 = x[0], x[1]` 而不是 `def f(x1, x2)` |
| 目標函數在某些點的值是 `nan` 或 `inf` | 計算中出現除以零或 log(0) 等 | 在目標函數中加入防護：`if Q <= 0: return 1e10` |
| 學生問「有沒有全域最佳化的方法？」 | 想找全局最優解 | 介紹 `scipy.optimize.differential_evolution`（隨機搜索），或說明多次不同初始點的策略；Level 4 會深入討論 |
