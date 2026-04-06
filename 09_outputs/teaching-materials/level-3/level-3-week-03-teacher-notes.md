---
title: Level 3 週 03 — SciPy 基本數值方法（教師教案）
type: teaching
level: 3
week: 3
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 03 教師教案
## 主題：SciPy — 基本數值方法（求根與數值積分）

---

## 1. 本週學習目標

1. 學生能說出「數值求根」的直覺意義，並能用 `scipy.optimize.brentq` 找到函數的零點
2. 學生能說出「數值積分」的直覺意義，並能用 `scipy.integrate.quad` 計算定積分
3. 學生能判斷「什麼情況下用數值方法比解析解更實際」
4. 學生能完成一個結合求根或積分的小型程式，並能解釋計算結果的物理或問題意義

---

## 2. 課堂時間分配表（120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 回顧 NumPy 改寫，引出「NumPy 能算，但有些問題需要『解』」 |
| 概念講解 | 25 分鐘 | 數值求根直覺（二分法）、數值積分直覺（黎曼和）、SciPy 是什麼 |
| 範例示範（Live coding） | 30 分鐘 | `brentq` 求根示範 + `quad` 積分示範 |
| 學生實作 | 35 分鐘 | 學生完成一個求根或積分小程式 |
| 回顧與預告 | 20 分鐘 | 討論數值誤差，預告 Week 4 ODE |

---

## 3. 各段落執行細節

### 3.1 暖身回顧（10 分鐘）

**引入問題**：
> 「NumPy 很強，但如果我問你：`x^3 - 2x + 1 = 0`，x 等於多少？NumPy 能幫你嗎？」

引導討論：
- NumPy 能幫你「算出」某個 x 值代入的結果
- 但「找出讓等式成立的 x」是另一種問題：這就是**求根**
- 再舉一例：「一條曲線下面的面積是多少？」→ 這是**積分**

說明今天的主題：SciPy 的 `optimize` 和 `integrate` 就是為了解決這兩類問題。

---

### 3.2 概念講解（25 分鐘）

**概念一：數值求根的直覺（10 分鐘）**

**用二分法（Bisection method）解釋**：

在白板上畫一條函數曲線（例如 `f(x) = x^2 - 2`），問：
> 「你怎麼找到讓 f(x) = 0 的 x？」

手動演示二分法流程：
1. 找一個讓 f(a) < 0 的 a，和讓 f(b) > 0 的 b
2. 計算中點 m = (a+b)/2
3. 如果 f(m) < 0，把 a 換成 m；如果 f(m) > 0，把 b 換成 m
4. 重複，直到夠精確

```
f(a) < 0 ─────────┬───────── f(b) > 0
                   │
           f(m) 是正還是負？
```

說明 `brentq`（Brent's method）是二分法的改進版，保證收斂且速度更快，是 SciPy 的預設求根方法。

**強調**：「你不需要知道 Brent's method 的細節。你需要知道的是：給它一個『f(a) 和 f(b) 異號的區間』，它就能找到零點。」

**概念二：數值積分的直覺（10 分鐘）**

在白板畫一條曲線，問：「這條曲線下面的面積怎麼算？」

解析：
- 如果函數有解析解（例如 `x^2`），可以用微積分公式
- 如果函數是從資料來的，或沒有簡單的反導函數，就需要數值積分
- **黎曼和**：把面積切成許多小長方形，加起來

示意圖（用 ASCII 或白板）：
```
    │  ██
    │ ████
    │██████
    └──────────
```

說明 `scipy.integrate.quad` 使用自適應積分（adaptive integration），比黎曼和精確得多，誤差通常在 `1e-8` 以內。

**概念三：什麼情況下用數值方法？（5 分鐘）**

| 情況 | 用解析解 | 用數值方法 |
|------|----------|------------|
| 簡單多項式、指數函數 | 通常可以 | 也可以 |
| 複雜複合函數 | 困難或不可能 | 推薦 |
| 從實驗資料估算 | 不適用 | 必須 |
| 結果要「說得出精度」 | 取決於方法 | `quad` 會告訴你誤差上界 |

---

### 3.3 範例示範（Live Coding）（30 分鐘）

**第一段（15 分鐘）：數值求根 `brentq`**

```python
import numpy as np
from scipy.optimize import brentq
import matplotlib.pyplot as plt

# ── 問題：找 cos(x) = x 的解 ──────────────────────────
# 等價於找 f(x) = cos(x) - x = 0 的零點

def f(x):
    return np.cos(x) - x

# 畫圖，先確認大概位置
x = np.linspace(-1, 3, 300)
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, f(x), label="cos(x) - x", color="steelblue")
ax.axhline(0, color="gray", linestyle="--")
ax.set_xlabel("x")
ax.set_ylabel("f(x)")
ax.set_title("找 cos(x) = x 的解")
ax.legend()
plt.tight_layout()
plt.show()

# 從圖可以看到，解在 x = 0 和 x = 1 之間
# 確認 f(0) 和 f(1) 異號
print(f"f(0) = {f(0):.4f}")   # 應為正
print(f"f(1) = {f(1):.4f}")   # 應為負

# 用 brentq 求解
root = brentq(f, 0, 1)
print(f"\ncos(x) = x 的解：x = {root:.10f}")
print(f"驗證：cos({root:.4f}) = {np.cos(root):.10f}")
print(f"差值：{abs(np.cos(root) - root):.2e}")  # 應該非常接近 0
```

**引導學生觀察**：
- 為什麼要先畫圖？（確認零點大概在哪，選合適的區間）
- 如果區間給錯（f(a) 和 f(b) 同號），`brentq` 會報錯
- `root` 的精確度達到 `1e-10` 級別

**第二段（15 分鐘）：數值積分 `quad`**

```python
import numpy as np
from scipy.integrate import quad
import matplotlib.pyplot as plt

# ── 問題一：計算 sin(x) 從 0 到 π 的面積 ─────────────
# 解析解：[-cos(x)] 從 0 到 π = -cos(π) + cos(0) = 1 + 1 = 2

def integrand(x):
    return np.sin(x)

result, error = quad(integrand, 0, np.pi)
print(f"∫₀^π sin(x) dx = {result:.10f}")
print(f"解析解 = 2.0")
print(f"數值誤差估計：{error:.2e}")

# ── 問題二：計算標準常態分布的 CDF ────────────────────
# 計算 P(-1 < X < 1)，也就是「一個標準差內的機率」
# 解析解：≈ 0.6827

def normal_pdf(x):
    """標準常態分布的機率密度函數"""
    return (1 / np.sqrt(2 * np.pi)) * np.exp(-0.5 * x**2)

prob, err = quad(normal_pdf, -1, 1)
print(f"\nP(-1 < X < 1) = {prob:.6f}")
print(f"（常識值：約 68.27%，你的結果：{prob*100:.2f}%）")

# ── 視覺化積分區域 ──────────────────────────────────
x = np.linspace(-3, 3, 300)
y = np.vectorize(normal_pdf)(x)

x_fill = np.linspace(-1, 1, 100)
y_fill = np.vectorize(normal_pdf)(x_fill)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, y, color="steelblue", label="標準常態分布")
ax.fill_between(x_fill, y_fill, alpha=0.4, color="steelblue", label=f"P(-1<X<1) ≈ {prob:.4f}")
ax.set_xlabel("x")
ax.set_ylabel("概率密度")
ax.set_title("標準常態分布：-1 到 1 的積分")
ax.legend()
plt.tight_layout()
plt.show()
```

**引導學生觀察**：
- `quad` 回傳兩個值：結果和誤差估計
- 誤差估計通常非常小（`1e-10` 量級）
- `np.vectorize` 是讓自定義函數能接受 array 輸入的小技巧

---

### 3.4 學生實作（35 分鐘）

學生完成 worksheet 的練習題：
- 練習題 1：用 `brentq` 找 `x^3 - x - 1 = 0` 的實數根
- 練習題 2：用 `quad` 計算一個自選函數的積分
- 練習題 3（進階）：把上述方法用在自己的專題問題上

教師巡視重點：
- 學生在設定 `brentq` 的區間時，有沒有先畫圖確認？（好習慣）
- `quad` 呼叫後，學生有沒有用解析解驗證？
- 有學生嘗試對複雜函數積分時，如果函數有奇點，`quad` 可能需要加 `points` 參數

---

### 3.5 回顧與預告（20 分鐘）

**統整討論（10 分鐘）**：
- 「數值方法的精確度如何？」→ `quad` 通常誤差 < `1e-8`，`brentq` 達到機器精度
- 「數值方法有沒有限制？」→ `brentq` 需要事先知道根的大概位置（給區間）；`quad` 對不連續函數可能失效
- 強調：「用數值方法，你就有責任驗證結果是否合理」

**預告 Week 4（10 分鐘）**：
- 下週：ODE——當你的問題是「系統如何隨時間變化」
- 引入問題：「如果一個族群每一刻的增長率都和當前族群數有關，你怎麼算它的未來值？」
- 這就是微分方程（ODE），用 `scipy.integrate.odeint` 解

---

## 4. 關鍵程式碼範例（完整可執行）

```python
"""
Level 3 Week 03 — SciPy 求根與積分完整示範
執行環境：Python 3.8+，需安裝 scipy, numpy, matplotlib
"""
import numpy as np
from scipy.optimize import brentq
from scipy.integrate import quad
import matplotlib.pyplot as plt

print("=" * 50)
print("Part 1：數值求根")
print("=" * 50)

# 問題：找供需均衡價格
# 供給函數：S(p) = 2p - 10
# 需求函數：D(p) = 100 - p
# 均衡條件：S(p) = D(p) → 2p - 10 = 100 - p → 3p = 110

def excess_demand(p):
    """超額需求函數：正值代表需求>供給，負值代表供給>需求"""
    demand = 100 - p
    supply = 2 * p - 10
    return demand - supply

# 找均衡價格（excess_demand = 0 的點）
p_eq = brentq(excess_demand, 10, 100)
q_eq = 100 - p_eq
print(f"均衡價格：{p_eq:.4f}")
print(f"均衡數量：{q_eq:.4f}")
print(f"驗證：供給={2*p_eq-10:.2f}，需求={100-p_eq:.2f}")

# 視覺化
p = np.linspace(5, 80, 200)
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot(p, 100 - p, label="需求 D(p)", color="steelblue")
axes[0].plot(p, 2*p - 10, label="供給 S(p)", color="coral")
axes[0].axvline(p_eq, color="green", linestyle="--", label=f"均衡價格 p*={p_eq:.1f}")
axes[0].set_xlabel("價格")
axes[0].set_ylabel("數量")
axes[0].set_title("供需均衡")
axes[0].legend()
axes[0].set_ylim(0, 120)

print("\n" + "=" * 50)
print("Part 2：數值積分")
print("=" * 50)

# 問題：計算物體從 t=0 到 t=5 的位移
# 速度函數：v(t) = 3t^2 - 2t + 1
# 解析解：∫₀^5 (3t²-2t+1) dt = [t³-t²+t]₀^5 = 125 - 25 + 5 = 105

def velocity(t):
    return 3*t**2 - 2*t + 1

displacement, err = quad(velocity, 0, 5)
print(f"位移（數值積分）：{displacement:.6f} m")
print(f"解析解：105.0 m")
print(f"誤差估計：{err:.2e}")

# 視覺化積分區域
t = np.linspace(0, 5, 300)
v = velocity(t)

axes[1].plot(t, v, color="steelblue", label="v(t) = 3t² - 2t + 1")
axes[1].fill_between(t, v, alpha=0.3, color="steelblue", label=f"位移 = {displacement:.1f} m")
axes[1].set_xlabel("時間 (s)")
axes[1].set_ylabel("速度 (m/s)")
axes[1].set_title("速度函數積分 = 位移")
axes[1].legend()

plt.tight_layout()
plt.show()
```

---

## 5. 常見問題與處理方式

| 問題 | 可能原因 | 建議處理 |
|------|----------|----------|
| `brentq` 報 `ValueError: f(a) and f(b) must have different signs` | 給的區間內 f(a) 和 f(b) 同號，或區間不包含零點 | 先 `print(f(a), f(b))` 確認符號；讓學生先畫圖，從圖上確認零點大概位置 |
| `quad` 回傳的誤差很大（>1e-4） | 被積函數在積分區間內有奇點或劇烈振動 | 嘗試縮小積分區間、使用 `limit` 參數增加分段數，或改用 `quad(..., points=[奇點位置])` |
| 學生問「為什麼 `brentq` 比 `fsolve` 好」 | 想知道多種求根方法的差別 | 說明 `brentq` 保證收斂（只要給對區間），`fsolve` 是牛頓法基礎，需要初始猜測值，不保證全局收斂 |
| `quad` 積分值和手算差很大 | 被積函數寫錯，或積分區間設錯 | 先在積分區間中取幾個點，手動計算函數值是否合理；確認上下限的單位一致 |
| 不知道「什麼時候用求根、什麼時候用積分」 | 概念未清楚 | 求根：找「讓某個量等於某個值」的未知數；積分：算「某個量在一段範圍內的累積總量」 |
