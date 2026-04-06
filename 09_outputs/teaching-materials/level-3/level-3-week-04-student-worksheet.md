---
title: Level 3 週 04 — SciPy 簡單 ODE 模型（學生講義）
type: teaching
level: 3
week: 4
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 04 學生講義
## 主題：SciPy — 簡單 ODE 模型

---

## 本週你要學什麼

到目前為止，你用 NumPy 加快了計算，用 SciPy 找到了方程的根和積分。本週要學的是 SciPy 最強大的功能之一：**求解微分方程（ODE）**。

ODE 讓你能模擬「**隨時間演變的系統**」——族群增長、疾病傳播、溫度冷卻、任何你能寫出「變化率」的現象。

本週結束後，你要能做到：
1. 把一個動態系統的描述轉化成 ODE 函數
2. 用 `scipy.integrate.odeint` 求解並取得時間序列
3. 畫出時間演化圖並解讀系統的長期行為

---

## 核心概念一：什麼是 ODE？

ODE（Ordinary Differential Equation，常微分方程）描述的是：
> **現在的狀態，決定了下一刻如何改變。**

### 一個具體例子：族群增長

假設一個族群，每個個體每單位時間都會生出 r 個後代。那麼：
```
dP/dt = r × P
```
這就是一個 ODE。`dP/dt` 是族群的瞬間變化率，它等於 `r × P`。

和差分方程的比較：

| | 差分方程（Level 2）| 微分方程（Level 3）|
|-|-------------------|--------------------|
| 描述 | `P(t+1) = P(t) × (1+r)` | `dP/dt = r × P` |
| 時間 | 離散（每步跳一格） | 連續（每一時刻） |
| 計算 | 手動 for 迴圈 | SciPy odeint 自動處理 |

ODE 描述的是更「真實」的物理過程，而差分是它的近似。

---

## 核心概念二：使用 `odeint` 的步驟

`odeint` 需要你提供三個東西：

### 步驟 1：定義 ODE 函數

函數簽名固定為 `func(y, t)`：
- `y`：系統當前狀態（可以是一個數，或多個數組成的 array）
- `t`：當前時間（很多模型的方程式不直接用到 t，但仍然要寫在簽名裡）
- 回傳值：`dydt`，也就是狀態的變化率

```python
def my_ode(y, t):
    dydt = ...   # 寫出你的方程式
    return dydt
```

### 步驟 2：設定初始條件

```python
y0 = 50.0   # 初始族群數量
# 或多維：y0 = [S0, I0, R0]
```

### 步驟 3：設定時間軸

```python
import numpy as np
t = np.linspace(0, 50, 500)   # 從 t=0 到 t=50，共 500 個時間點
```

### 步驟 4：呼叫 `odeint`

```python
from scipy.integrate import odeint
solution = odeint(my_ode, y0, t)
```

---

## 核心概念三：完整範例——牛頓冷卻定律

**問題**：一杯 90°C 的咖啡，在 20°C 的房間裡冷卻。溫度怎麼隨時間變化？

**ODE 描述**：溫差越大，降溫越快。
```
dT/dt = -k × (T - T_env)
```
其中 k 是冷卻係數，T_env 是環境溫度。

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ── 參數設定 ──────────────────────────────────────────
k = 0.1          # 冷卻係數
T_env = 20.0     # 環境溫度（°C）
T0 = 90.0        # 初始溫度（°C）

# ── 步驟 1：定義 ODE 函數 ──────────────────────────────
def cooling(T, t):
    """牛頓冷卻定律：dT/dt = -k × (T - T_env)"""
    dTdt = -k * (T - T_env)
    return dTdt

# ── 步驟 2–3：初始條件和時間軸 ────────────────────────
y0 = T0
t = np.linspace(0, 60, 300)   # 60 分鐘，300 個時間點

# ── 步驟 4：求解 ODE ───────────────────────────────────
T = odeint(cooling, y0, t)

# ── 取出結果並繪圖 ─────────────────────────────────────
T_values = T[:, 0]   # odeint 回傳 (n, 1) 的 array，取第一欄

plt.figure(figsize=(8, 4))
plt.plot(t, T_values, color="coral", label="溫度（數值解）")
plt.axhline(T_env, color="gray", linestyle="--", label=f"環境溫度 {T_env}°C")
plt.xlabel("時間（分鐘）")
plt.ylabel("溫度（°C）")
plt.title("咖啡冷卻：牛頓冷卻定律")
plt.legend()
plt.show()

# ── 驗證：和解析解比較 ─────────────────────────────────
T_analytical = T_env + (T0 - T_env) * np.exp(-k * t)
max_error = np.max(np.abs(T_values - T_analytical))
print(f"數值解與解析解最大誤差：{max_error:.2e}")
# 誤差應該非常小（< 1e-6）
```

---

## 核心概念四：多維 ODE——SIR 流行病模型

當系統有多個狀態量（例如族群中的易感者、感染者、康復者），`y` 就是一個 array。

**SIR 模型**：
```
dS/dt = -β × S × I / N
dI/dt =  β × S × I / N - γ × I
dR/dt =  γ × I
```

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# 參數
N = 10000    # 總人口
beta = 0.3   # 傳播率
gamma = 0.1  # 康復率

def sir_model(y, t):
    """SIR 流行病模型"""
    S, I, R = y          # 從 y 中拆解出各個分量
    dSdt = -beta * S * I / N
    dIdt =  beta * S * I / N - gamma * I
    dRdt =  gamma * I
    return [dSdt, dIdt, dRdt]   # 回傳所有分量的變化率

# 初始條件（S, I, R）
y0 = [N - 10, 10, 0]
t = np.linspace(0, 200, 1000)

# 求解
solution = odeint(sir_model, y0, t)
S = solution[:, 0]   # 第 0 欄是 S
I = solution[:, 1]   # 第 1 欄是 I
R = solution[:, 2]   # 第 2 欄是 R

# 繪圖
plt.figure(figsize=(10, 4))
plt.plot(t, S, label="易感者 S", color="steelblue")
plt.plot(t, I, label="感染者 I", color="coral")
plt.plot(t, R, label="康復者 R", color="green")
plt.xlabel("天")
plt.ylabel("人數")
plt.title(f"SIR 模型（β={beta}, γ={gamma}）")
plt.legend()
plt.show()

# 找感染高峰
peak_day = t[np.argmax(I)]
print(f"感染高峰：第 {peak_day:.0f} 天，感染人數 = {I.max():.0f}")
print(f"基本再生數 R₀ = β/γ = {beta/gamma:.2f}")
```

---

## 本週實作題

### 練習題 1：Logistic 族群增長

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# Logistic 方程式：dP/dt = r × P × (1 - P/K)
# r：增長率；K：攜帶容量

r = 0.5
K = 1000

def logistic(P, t):
    # 請填寫 dP/dt 的表達式
    dPdt = # 你的程式碼
    return dPdt

# (a) 設定初始族群 P0 = 10，解 ODE 到 t = 40
t = np.linspace(0, 40, 400)
P0 = 10
P = odeint(logistic, P0, t)

# (b) 畫出時間演化圖，在圖上標記攜帶容量 K
# 你的程式碼：

# (c) 族群何時達到 K/2？（提示：用 np.argmax 找第一次超過的索引）
# 你的程式碼：

# (d) 把 P0 改成 1200（超過 K），看系統怎麼變化。說明你的觀察。
```

---

### 練習題 2：牛頓冷卻定律的參數探索

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

T_env = 25.0
T0 = 80.0

def cooling(T, t, k):
    """冷卻方程，k 是參數"""
    return -k * (T - T_env)

t = np.linspace(0, 60, 300)

# (a) 在同一張圖上，畫出 k = 0.05, 0.1, 0.2 三種情況的冷卻曲線
# 提示：odeint 可以用 args=(k,) 傳入額外參數
# 例如：sol = odeint(cooling, T0, t, args=(0.1,))

# 你的程式碼：

# (b) 哪個 k 值讓溫度在 30 分鐘後最接近環境溫度？
# 你的分析：
```

---

### 練習題 3（挑戰）：設計你自己的 ODE 模型

選擇以下任一個，或設計你自己的：

**選項 A：捕食者-獵物模型（Lotka-Volterra）**
```
dR/dt = a × R - b × R × F     # R：兔子（獵物），a：增長率，b：被捕食率
dF/dt = c × R × F - d × F     # F：狐狸（捕食者），c：增長率，d：死亡率
```
試試 a=1, b=0.1, c=0.075, d=1.5，初始 R=10, F=5

**選項 B：彈簧振動（阻尼系統）**
```
dx/dt = v
dv/dt = -k × x - c × v   # x：位移，v：速度，k：彈簧常數，c：阻尼
```
試試 k=1, c=0.1，初始 x=1, v=0

**選項 C：你自己的模型**
用你的舊專題裡的動態系統，寫成 ODE 形式

---

## 本週成果

- [ ] 完成練習題 1（Logistic 族群增長 + 問題 a/b/c）
- [ ] 完成練習題 2（牛頓冷卻定律 + 三條曲線）
- [ ] 嘗試練習題 3（選填，但建議嘗試 Lotka-Volterra）
- [ ] 時間演化圖有標題、x 軸（時間）、y 軸（狀態量）、圖例
