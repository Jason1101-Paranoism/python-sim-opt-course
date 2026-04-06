---
title: Level 3 週 07 — 專題實作：模型與數值方法（學生講義）
type: teaching
level: 3
week: 7
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 07：專題實作 — 模型與數值方法

## 本週主題

這週開始動手寫你的高級專題程式。核心任務是：**把數值方法整合進你的專題，讓程式能跑起來、輸出一張有意義的圖**。

本週特別關注兩件事：
1. **整合 SciPy 工具**：用 `solve_ivp`（ODE）或 `quad`（數值積分）處理你模型的核心計算
2. **結果合理性驗證**：收斂測試 + 邊界案例，確保你的程式沒有悄悄算錯

---

## 核心概念一：什麼是收斂？為什麼重要？

數值方法不是解析解，而是用計算機一步一步「逼近」真實答案。步長越小、精度越高，但計算越慢。

### 收斂測試的做法

用不同的步長（或精度設定）跑同一個問題，看關鍵指標是否趨近同一個值。

```python
import numpy as np
from scipy.integrate import solve_ivp

def simple_ode(t, y):
    """簡單衰減：dy/dt = -0.5 * y"""
    return [-0.5 * y[0]]

# 解析解：y(t) = y0 * exp(-0.5 * t)
# 用這個來驗證數值解的準確度

y0 = [10.0]
t_span = (0, 10)
t_eval = np.array([10.0])  # 只看 t=10 的值

# 解析解的真實值
true_value = 10.0 * np.exp(-0.5 * 10)
print(f"解析解（t=10）：{true_value:.6f}")

# 用不同步長測試收斂
print("\n收斂測試：")
for max_step in [2.0, 1.0, 0.5, 0.1, 0.01]:
    sol = solve_ivp(simple_ode, t_span, y0, t_eval=t_eval, max_step=max_step)
    numerical = sol.y[0, -1]
    error = abs(numerical - true_value)
    print(f"  max_step={max_step:.3f} → 數值解={numerical:.6f}，誤差={error:.2e}")
```

執行後你應該看到：步長越小，誤差越小，最終趨近於解析解。

---

## 核心概念二：結果合理性的三種驗證方法

### 方法一：邊界案例測試

把一個參數設到極端值（0 或非常大），驗證結果是否符合直覺。

```python
# 以 SIR 模型為例
def sir_model(t, y, beta, gamma, N):
    """SIR 傳染病模型"""
    S, I, R = y
    dS_dt = -beta * S * I / N
    dI_dt =  beta * S * I / N - gamma * I
    dR_dt =  gamma * I
    return [dS_dt, dI_dt, dR_dt]

N = 10000
y0 = [9990, 10, 0]
t_span = (0, 100)

# 邊界案例 1：接觸率 = 0（沒有傳播）
# 預期：感染者只減少，不增加
sol_no_spread = solve_ivp(
    sir_model, t_span, y0,
    args=(0.0, 0.1, N),
    dense_output=True
)
t_check = np.linspace(0, 100, 200)
y_check = sol_no_spread.sol(t_check)
print(f"β=0 時，感染者最大值：{y_check[1].max():.2f}")
print(f"  → 應等於初始值 {y0[1]}（感染者只減不增）")

# 邊界案例 2：康復率 = 0（沒有康復）
# 預期：康復者永遠為 0，感染者不斷增加直到全部感染
sol_no_recovery = solve_ivp(
    sir_model, t_span, y0,
    args=(0.3, 0.0, N),
    dense_output=True
)
y_no_rec = sol_no_recovery.sol(t_check)
print(f"\nγ=0 時，康復者最大值：{y_no_rec[2].max():.2f}")
print(f"  → 應為 0.00（沒有康復）")
```

---

### 方法二：與簡化版本對照

先用 for 迴圈做差分（歐拉法），再用 solve_ivp，比較兩者在相同步長下的結果。

```python
# 歐拉法手動實作
def euler_sir(beta, gamma, N, y0, T, dt):
    """用歐拉法（差分）求解 SIR 模型"""
    S, I, R = y0
    t = 0
    history_I = [I]
    
    while t < T:
        dS = -beta * S * I / N
        dI =  beta * S * I / N - gamma * I
        dR =  gamma * I
        S += dS * dt
        I += dI * dt
        R += dR * dt
        t += dt
        history_I.append(I)
    
    return max(history_I)

# 比較兩種方法（步長 dt = 0.5 天）
beta, gamma = 0.3, 0.1
dt = 0.5

# 歐拉法
peak_euler = euler_sir(beta, gamma, N, y0, 100, dt)
print(f"歐拉法（dt={dt}）：感染高峰 = {peak_euler:.2f}")

# solve_ivp
sol_scipy = solve_ivp(
    sir_model, (0, 100), y0,
    args=(beta, gamma, N),
    dense_output=True,
    max_step=dt
)
t_plot = np.linspace(0, 100, 500)
y_plot = sol_scipy.sol(t_plot)
peak_scipy = y_plot[1].max()
print(f"solve_ivp（max_step={dt}）：感染高峰 = {peak_scipy:.2f}")
print(f"兩者差距：{abs(peak_euler - peak_scipy):.2f}")
```

---

## 完整範例程式：SIR 模型整合實作

以下是一個完整可執行的專題程式框架，你可以根據自己的題目修改。

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import solve_ivp

# ================================================================
# 第一步：定義你的 ODE 模型
# ================================================================
def sir_model(t, y, beta, gamma, N):
    """
    SIR 傳染病模型
    y = [S, I, R]：易感者、感染者、康復者
    beta：接觸率，gamma：康復率，N：總人口
    """
    S, I, R = y
    dS_dt = -beta * S * I / N
    dI_dt =  beta * S * I / N - gamma * I
    dR_dt =  gamma * I
    return [dS_dt, dI_dt, dR_dt]

# ================================================================
# 第二步：設定參數
# ================================================================
N = 10000          # 總人口
y0 = [9990, 10, 0] # 初始條件：S0=9990, I0=10, R0=0
beta = 0.3         # 接觸率
gamma = 0.1        # 康復率
t_span = (0, 160)  # 模擬 160 天

# ================================================================
# 第三步：數值求解
# ================================================================
sol = solve_ivp(
    fun=sir_model,
    t_span=t_span,
    y0=y0,
    args=(beta, gamma, N),
    dense_output=True,  # 啟用：可以在任意時間點查詢
    max_step=0.5        # 最大步長（天）
)

# 在均勻時間點上取值（用於畫圖）
t_eval = np.linspace(t_span[0], t_span[1], 500)
y_eval = sol.sol(t_eval)  # shape: (3, 500)

# ================================================================
# 第四步：畫圖
# ================================================================
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 左圖：S、I、R 的時間演化
axes[0].plot(t_eval, y_eval[0], label='S（易感者）', color='steelblue')
axes[0].plot(t_eval, y_eval[1], label='I（感染者）', color='crimson')
axes[0].plot(t_eval, y_eval[2], label='R（康復者）', color='seagreen')
axes[0].set_xlabel('時間（天）')
axes[0].set_ylabel('人數')
axes[0].set_title('SIR 模型：時間演化')
axes[0].legend()

# 右圖：收斂測試（不同 max_step 下的感染高峰）
steps = [5.0, 2.0, 1.0, 0.5, 0.1, 0.05]
peaks = []
for ms in steps:
    s = solve_ivp(sir_model, t_span, y0, args=(beta, gamma, N),
                  dense_output=True, max_step=ms)
    y_s = s.sol(t_eval)
    peaks.append(y_s[1].max())

axes[1].semilogx(steps, peaks, 'o-', color='darkorange')
axes[1].invert_xaxis()  # 左側為小步長（精確），右側為大步長（粗糙）
axes[1].set_xlabel('max_step（天，對數軸）')
axes[1].set_ylabel('感染高峰人數')
axes[1].set_title('收斂測試：步長 vs 感染高峰')
axes[1].axhline(peaks[-1], color='gray', linestyle='--', label='參考值（最小步長）')
axes[1].legend()

plt.suptitle(f'SIR 模型分析（β={beta}, γ={gamma}, N={N}）', fontsize=13)
plt.tight_layout()
plt.savefig('sir_week07.png', dpi=150)
plt.show()

# ================================================================
# 第五步：印出關鍵數字
# ================================================================
peak_I = y_eval[1].max()
peak_t = t_eval[y_eval[1].argmax()]
total_infected = y_eval[2][-1]  # 最終康復者 ≈ 總感染人數

print(f"感染高峰：{peak_I:.0f} 人（第 {peak_t:.1f} 天）")
print(f"最終感染總人數（累計）：{total_infected:.0f} 人")
print(f"最終未感染人數（S）：{y_eval[0][-1]:.0f} 人")

# 基本再生數 R0（理論值）
R0_theory = beta / gamma
print(f"\n理論 R0 = β/γ = {beta}/{gamma} = {R0_theory:.2f}")
if R0_theory > 1:
    print("R0 > 1：疫情會爆發")
else:
    print("R0 ≤ 1：疫情不會大規模傳播")
```

---

## 練習題

### 練習 1：收斂測試

執行上面的收斂測試程式碼，回答以下問題：

a) 當 `max_step = 5.0` 和 `max_step = 0.05` 時，感染高峰分別是多少人？差距是多少？

b) 從哪個 `max_step` 開始，結果變化小於 1 人？這代表什麼？

c) 你認為這個問題使用 `max_step = 1.0` 是否足夠？理由是什麼？

---

### 練習 2：套用到你的專題

根據你的「高級專題計畫書」，完成以下程式框架：

```python
# 把下面的 ??? 替換成你的專題內容

def my_model(t, y, ???):
    """
    [你的模型名稱]
    y = [你的狀態變數]
    """
    # 寫出你的微分方程
    d???_dt = ???
    return [d???_dt]

# 初始條件
y0 = [???]
t_span = (0, ???)
params = (???, ???)

# 呼叫 solve_ivp
sol = solve_ivp(my_model, t_span, y0, args=params, dense_output=True)

# 畫出結果
# ...（自行完成）
```

---

### 練習 3：邊界案例設計

為你的專題設計至少兩個邊界案例。填入下表：

| 邊界案例 | 設定方式 | 預期結果 | 實際結果 | 是否符合 |
|----------|----------|----------|----------|----------|
| 案例 1 | | | | |
| 案例 2 | | | | |

---

## 本週目標

課堂結束時，你應該手上有：
- 一個可以執行的 `.py` 檔
- 至少一張有意義的圖（不一定要漂亮，但要能執行）
- 至少一個邊界案例的測試結果
