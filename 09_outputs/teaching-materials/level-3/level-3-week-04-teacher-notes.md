---
title: Level 3 週 04 — SciPy 簡單 ODE 模型（教師教案）
type: teaching
level: 3
week: 4
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 04 教師教案
## 主題：SciPy — 簡單 ODE 模型

---

## 1. 本週學習目標

1. 學生能用「現在的狀態決定了變化速率」的語言解釋 ODE 的直覺
2. 學生能把一個簡單的動態系統描述（例如：族群增長率 = r × 族群數）轉化成 Python 函數
3. 學生能用 `scipy.integrate.odeint` 解一個 1–2 維的 ODE 系統，取得時間序列解
4. 學生能繪製時間演化圖，並正確解讀圖形（穩定狀態、振盪、指數增長等）

---

## 2. 課堂時間分配表（120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 從 Week 3 積分引出「動態系統」的概念 |
| 概念講解 | 25 分鐘 | ODE 直覺、差分 vs 微分、`odeint` 的呼叫方式 |
| 範例示範（Live coding） | 30 分鐘 | Logistic 族群模型 + SIR 流行病模型 |
| 學生實作 | 35 分鐘 | 學生建立一個自選 ODE 模型並畫出時間演化圖 |
| 回顧與預告 | 20 分鐘 | 解讀圖形意義，預告 Week 5 最佳化 |

---

## 3. 各段落執行細節

### 3.1 暖身回顧（10 分鐘）

**引入問題**：
> 「上週我們用 `quad` 計算了速度函數的積分等於位移。但如果速度本身是隨時間變化的，而且變化方式和速度本身有關，你怎麼辦？」

舉例：
- 族群增長：族群越多，增加越多（`dP/dt = r × P`）
- 冷卻：溫度差越大，降溫越快（牛頓冷卻定律：`dT/dt = -k(T - T_env)`）
- 這類「變化率和當前狀態有關」的方程式，叫做微分方程（ODE）

**說明今天的方向**：「我們不需要手解微分方程。SciPy 幫我們做數值積分，我們只要告訴它『規則是什麼』（即 ODE 函數），它就幫我們追蹤系統怎麼走。」

---

### 3.2 概念講解（25 分鐘）

**概念一：差分 vs 微分（10 分鐘）**

用學生熟悉的差分方程（Level 2 風格）來類比：

**差分方程（Level 2）**：
```
P(t+1) = P(t) + r × P(t)
       = P(t) × (1 + r)
```
→ 每一步離散地計算下一步

**微分方程（Level 3）**：
```
dP/dt = r × P(t)
```
→ 描述的是「每一時刻的瞬間變化率」

關係：差分是微分的離散近似。當時間步長趨近 0，差分就變成微分。

在白板上畫：
```
    P
    │          ODE 解（連續曲線）
    │       ╱
    │    ╱
    │ ╱  ← 差分解（折線近似）
    └─────────── t
```

**概念二：如何用 `odeint`（10 分鐘）**

`odeint` 需要三個東西：

1. **ODE 函數** `dydt = func(y, t)`：告訴 SciPy「在時間 t，狀態是 y，那麼變化率是多少？」
2. **初始條件** `y0`：t=0 時的狀態
3. **時間點** `t`：你想知道系統在哪些時間點的狀態

```python
from scipy.integrate import odeint
import numpy as np

def ode_func(y, t):
    dydt = ...  # 寫出變化率的公式
    return dydt

y0 = 初始條件
t = np.linspace(0, 終止時間, 時間點數)
solution = odeint(ode_func, y0, t)
```

強調：`y` 可以是一個數（1D ODE）或一個 array（多維 ODE）。

**概念三：`odeint` 做了什麼？（5 分鐘）**

用最簡單的語言解釋：
- SciPy 用的是 Runge-Kutta 數值積分法（不需要深入解釋）
- 直覺：從初始狀態出發，根據 ODE 函數計算「下一小步怎麼走」，一步步推進
- 和差分方程很像，但步長更小，方法更精確

---

### 3.3 範例示範（Live Coding）（30 分鐘）

**第一段（15 分鐘）：Logistic 族群模型**

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ── Logistic 族群模型 ──────────────────────────────────
# dP/dt = r × P × (1 - P/K)
# r：自然增長率
# K：攜帶容量（環境最大族群數）
# 直覺：族群小時快速增長，接近 K 時增長趨緩

r = 0.5   # 增長率
K = 1000  # 攜帶容量
P0 = 50   # 初始族群

def logistic(P, t):
    """Logistic 增長方程式"""
    dPdt = r * P * (1 - P / K)
    return dPdt

# 解 ODE
t = np.linspace(0, 30, 300)   # 30 個時間單位，300 個時間點
P = odeint(logistic, P0, t)

# 繪圖
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t, P, color="steelblue", label=f"P₀={P0}, r={r}, K={K}")
ax.axhline(K, color="red", linestyle="--", label=f"攜帶容量 K={K}")
ax.set_xlabel("時間")
ax.set_ylabel("族群數量")
ax.set_title("Logistic 族群增長")
ax.legend()
plt.tight_layout()
plt.show()

# 觀察：族群達到 K/2 的時間
# 用上週學的 brentq 找到！
from scipy.optimize import brentq
# 插值：找 P 第一次超過 K/2 的時間
t_half_idx = np.argmax(P[:, 0] > K / 2)
print(f"族群達到 K/2 = {K/2} 大約在 t = {t[t_half_idx]:.2f}")
```

**引導學生觀察**：
- S 形曲線（Sigmoid）：開始指數增長，接近 K 時趨緩
- K 是漸近線，族群最終接近但不超過 K
- 試試改變 r 和 K 的值，看圖形如何變

**第二段（15 分鐘）：SIR 流行病模型（2D ODE）**

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ── SIR 流行病模型 ─────────────────────────────────────
# S：易感染者（Susceptible）
# I：感染者（Infected）
# R：康復者（Recovered）
#
# dS/dt = -β × S × I / N
# dI/dt =  β × S × I / N - γ × I
# dR/dt =  γ × I
#
# β：傳播率；γ：康復率

N = 10000    # 總人口
beta = 0.3   # 傳播率
gamma = 0.1  # 康復率

S0 = N - 10   # 初始易感者
I0 = 10       # 初始感染者
R0_init = 0   # 初始康復者

def sir_model(y, t, beta, gamma, N):
    """SIR 模型的微分方程"""
    S, I, R = y
    dSdt = -beta * S * I / N
    dIdt =  beta * S * I / N - gamma * I
    dRdt =  gamma * I
    return [dSdt, dIdt, dRdt]

# 初始條件（多維 ODE：y 是 array）
y0 = [S0, I0, R0_init]
t = np.linspace(0, 200, 1000)

# 解 ODE（額外參數用 args 傳入）
solution = odeint(sir_model, y0, t, args=(beta, gamma, N))
S, I, R = solution[:, 0], solution[:, 1], solution[:, 2]

# 繪圖
fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(t, S, label="易感者 S", color="steelblue")
ax.plot(t, I, label="感染者 I", color="coral")
ax.plot(t, R, label="康復者 R", color="green")
ax.set_xlabel("天")
ax.set_ylabel("人數")
ax.set_title(f"SIR 流行病模型（β={beta}, γ={gamma}, N={N}）")
ax.legend()
plt.tight_layout()
plt.show()

# 找感染高峰
peak_day = t[np.argmax(I)]
peak_infected = I.max()
print(f"感染高峰：第 {peak_day:.0f} 天，感染人數 = {peak_infected:.0f}")
print(f"基本再生數 R₀ = β/γ = {beta/gamma:.2f}")
```

**引導學生觀察**：
- SIR 模型是 2D ODE：y 是包含 [S, I, R] 的 array
- `solution[:, 0]` 是 S 的時間序列，`solution[:, 1]` 是 I，以此類推
- 試試把 β 增加（更強的病毒）或 γ 增加（更快康復），看高峰如何變化

---

### 3.4 學生實作（35 分鐘）

學生完成 worksheet 的實作：
- 選一個 ODE 模型（簡單族群模型、牛頓冷卻定律，或自己設計）
- 建立 ODE 函數
- 用 `odeint` 求解並畫出時間演化圖

**教師巡視重點**：
- ODE 函數的簽名是否正確（`func(y, t)`，而不是 `func(t, y)`）
- 多維 ODE 中，記得從 y 拆解出各個分量
- 圖表有沒有標籤和標題

**常見技術問題**：
- `odeint` 和 `solve_ivp` 的差別：`odeint` 的函數簽名是 `func(y, t)`，`solve_ivp` 是 `func(t, y)`。本課程使用 `odeint`

---

### 3.5 回顧與預告（20 分鐘）

**統整討論（10 分鐘）**：
- 「你的 ODE 模型，最後系統到哪裡去了？穩定了、振盪、還是持續增長？」
- 讓 2–3 位學生說說他們的模型觀察
- 強調：時間演化圖是「診斷工具」，它告訴你系統長期行為

**預告 Week 5（10 分鐘）**：
- 下週：最佳化——當你不只想模擬「系統如何走」，而是要找「讓結果最好的參數」
- 引入問題：「你的 ODE 模型裡，有沒有想知道『什麼 β 值讓高峰最低』或『什麼 r 值讓產出最大』？這就是最佳化。」

---

## 4. 關鍵程式碼範例（完整可執行）

```python
"""
Level 3 Week 04 — SciPy ODE 完整示範：牛頓冷卻定律 + Logistic 增長
執行環境：Python 3.8+，需安裝 scipy, numpy, matplotlib
"""
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ── 牛頓冷卻定律 ──────────────────────────────────────
# dT/dt = -k × (T - T_env)
# 直覺：溫差越大，降溫越快

k = 0.1          # 冷卻係數
T_env = 20.0     # 環境溫度（°C）
T0 = 90.0        # 初始溫度（°C）

def newton_cooling(T, t):
    dTdt = -k * (T - T_env)
    return dTdt

t = np.linspace(0, 60, 300)   # 60 分鐘
T = odeint(newton_cooling, T0, t)

# 解析解：T(t) = T_env + (T0 - T_env) * exp(-k*t)
T_analytical = T_env + (T0 - T_env) * np.exp(-k * t)

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot(t, T, label="數值解（odeint）", color="coral", linewidth=2)
axes[0].plot(t, T_analytical, label="解析解", color="steelblue",
             linestyle="--", linewidth=1.5)
axes[0].axhline(T_env, color="gray", linestyle=":", label=f"環境溫度 {T_env}°C")
axes[0].set_xlabel("時間（分鐘）")
axes[0].set_ylabel("溫度（°C）")
axes[0].set_title("牛頓冷卻定律")
axes[0].legend()

# 驗證數值解與解析解的差距
max_error = np.abs(T[:, 0] - T_analytical).max()
print(f"數值解與解析解最大誤差：{max_error:.2e}")

# ── Logistic 增長（不同初始條件） ────────────────────
r, K = 0.4, 1000

def logistic(P, t):
    return r * P * (1 - P / K)

initial_conditions = [10, 100, 500, 900]
colors = ["steelblue", "coral", "green", "purple"]

for P0, color in zip(initial_conditions, colors):
    P = odeint(logistic, P0, t)
    axes[1].plot(t, P, color=color, label=f"P₀={P0}")

axes[1].axhline(K, color="black", linestyle="--", label=f"K={K}")
axes[1].set_xlabel("時間")
axes[1].set_ylabel("族群數量")
axes[1].set_title("Logistic 增長（不同初始條件）")
axes[1].legend()

plt.tight_layout()
plt.show()
```

---

## 5. 常見問題與處理方式

| 問題 | 可能原因 | 建議處理 |
|------|----------|----------|
| `odeint` 回傳的 shape 搞不清楚 | 1D ODE 回傳 `(n_timepoints, 1)`，多維回傳 `(n_timepoints, n_vars)` | `print(solution.shape)` 後，用 `solution[:, 0]` 取第一個變量；或用 `solution.flatten()` 處理 1D 結果 |
| ODE 函數簽名弄錯（`func(t, y)` 而非 `func(y, t)`） | 混淆了 `odeint` 和 `solve_ivp` 的 API | 明確說明：`odeint` 用 `func(y, t)`，記憶法「y 先 t 後」 |
| 解的值在短時間內爆炸（變成 `inf` 或 `nan`） | ODE 不穩定（例如增長率太大）或積分步長太小 | 減小時間範圍，或在 ODE 函數中加入 `print(y, t)` 找到問題發生的時間點 |
| 多維 ODE 中，弄不清楚 y 的各個分量 | 沒有在函數中明確拆解 | 在函數開頭加 `S, I, R = y`（或類似），增加可讀性 |
| 問「為什麼不直接用差分方程？」 | 有合理疑問 | 解釋：差分方程需要選合適的時間步長，否則不穩定；`odeint` 會自動選最佳步長，更精確且不容易出錯 |
