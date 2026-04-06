---
title: Level 3 週 08 — 專題實作：最佳化與敏感度分析（教師教案）
type: teaching
level: 3
week: 8
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 08：專題實作 — 最佳化與敏感度分析（教師教案）

## 本週學習目標

1. 學生能把「我想要最大化/最小化 X」的問題轉化為可以被 `scipy.optimize.minimize` 呼叫的目標函數
2. 學生能執行最佳化，並用圖表或數字說明「最佳解是什麼、為什麼合理」
3. 學生能做簡單的單參數敏感度分析（固定其他條件，改變一個參數，觀察最佳解如何變化）
4. 學生能解釋最佳化結果的限制（初始點的影響、局部解 vs 全局解）

---

## 課堂時間分配（正常週結構，120 分鐘）

| 段落 | 時間 | 具體內容 |
|------|------|----------|
| 暖身回顧 | 10 分鐘 | 確認上週程式完成狀況；每人說「我的程式能算出什麼數字」 |
| 概念講解 | 25 分鐘 | 目標函數設計；minimize 呼叫方式；局部 vs 全局最佳解 |
| 範例示範（Live coding） | 30 分鐘 | 示範 SIR 模型的接種策略最佳化 + 敏感度分析 |
| 學生實作 | 35 分鐘 | 學生把最佳化加進自己的專題程式 |
| 回顧與預告 | 20 分鐘 | 總結三個關鍵點、說明作業、預告下週技術寫作 |

---

## 各段落執行細節

### 暖身回顧（10 分鐘）

**提問序列**：
1. 「上週的程式能跑了嗎？你的模型輸出的是什麼數字？」（讓每人說一句）
2. 「有沒有人做了收斂測試？結果怎樣？」（找 1–2 人分享）
3. 「你的程式現在能不能計算一個單一數字（如感染高峰人數、總成本）？」
   - 若可以 → 今天直接加最佳化
   - 若不行 → 今天的實作時間先補這個，最佳化在 1-on-1 或下週補

**老師注意**：本週目標是「把最佳化加進來」，但若上週程式還沒跑起來，不要強行推進。
程式先跑起來的優先級高於最佳化。

---

### 概念講解（25 分鐘）

#### 一、目標函數設計（10 分鐘）

**生活例子引入**（3 分鐘）：

> 「你去超市買東西，預算 500 元，想買到最多的蛋白質。你要最大化什麼？（蛋白質克數）限制是什麼？（總價 ≤ 500）。這就是最佳化。」

**從問題到程式**（7 分鐘）：

「`scipy.optimize.minimize` 只能做最小化。若你想最大化 f(x)，就最小化 -f(x)。」

**目標函數設計步驟**：

| 步驟 | 說明 |
|------|------|
| 1. 定義決策變數 | 「我要找的最佳值是什麼？（一個數字？多個數字？）」 |
| 2. 定義目標函數 | 「我要最大化/最小化什麼？能否寫成 f(決策變數) ？」 |
| 3. 定義限制條件 | 「決策變數有哪些限制？（範圍？不等式？等式？）」 |
| 4. 設定初始猜測 | 「從哪個起始點開始搜尋？」 |

**白板展示**（不同問題類型的目標函數格式）：

```
# 最小化問題
minimize f(x)         → minimize(f, x0)

# 最大化問題（轉換為最小化）
maximize f(x)         → minimize(lambda x: -f(x), x0)

# 帶邊界條件
minimize f(x)         → minimize(f, x0, bounds=[(a1, b1), (a2, b2)])
subject to a ≤ x ≤ b

# 帶不等式限制
minimize f(x)         → minimize(f, x0, constraints={'type': 'ineq', 'fun': g})
subject to g(x) ≥ 0
```

---

#### 二、局部最佳解 vs 全局最佳解（8 分鐘）

**視覺化**（在白板畫出一個有多個波谷的函數曲線）：

「`minimize` 從你給的初始點出發，找最近的山谷。如果函數有多個山谷，它不保證找到最深的那個。」

**實際影響**：
- 初始點 x0 不同 → 可能得到不同的「最佳解」
- 解決方法：從不同初始點跑多次，看結果是否一致

**問答**（3 分鐘）：
- 「若你跑最佳化 10 次，每次 x0 不同，得到的結果都一樣，這代表什麼？」
  → 很可能是全局最佳解（但不保證）
- 「若每次結果不同，你怎麼辦？」
  → 取最小的那個；或換用全局最佳化方法（如 `differential_evolution`）

---

#### 三、敏感度分析（7 分鐘）

「最佳化找到了一個答案，但這個答案對參數的變化有多敏感？這就是敏感度分析。」

**最簡單的做法（單參數掃描）**：

```
固定其他所有參數，改變一個參數 p 從 p_min 到 p_max
每個 p 的值都跑一次最佳化，記錄最佳解 x*
畫出 p vs x* 的折線圖
```

「如果 p 稍微變一點，x* 就大幅改變，代表最佳解對這個參數很敏感（脆弱）。」

---

### 範例示範（Live coding，30 分鐘）

**示範主題**：SIR 模型 — 找最佳接種比例（最小化感染高峰）+ 敏感度分析

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import solve_ivp
from scipy.optimize import minimize, minimize_scalar

# ================================================================
# 模型定義（沿用上週，加入接種比例 v）
# ================================================================
def sir_with_vaccination(t, y, beta, gamma, N):
    """SIR 模型（接種已在初始條件中處理）"""
    S, I, R = y
    dS_dt = -beta * S * I / N
    dI_dt =  beta * S * I / N - gamma * I
    dR_dt =  gamma * I
    return [dS_dt, dI_dt, dR_dt]

def peak_infected(v, beta=0.3, gamma=0.1, N=10000, I0=10):
    """
    計算給定接種比例 v 時的感染高峰人數。
    
    參數：
        v    : 接種比例，範圍 [0, 1]（0 = 沒有接種，1 = 全部接種）
        beta : 接觸率
        gamma: 康復率
        N    : 總人口
        I0   : 初始感染者
    
    回傳：
        感染高峰人數（整數）
    """
    # 接種者直接進入 R（已免疫）
    vaccinated = v * N
    S0 = N - I0 - vaccinated
    R0_count = vaccinated
    
    # 若接種後沒有易感者，感染不會傳播
    if S0 <= 0:
        return I0
    
    y0 = [S0, I0, R0_count]
    sol = solve_ivp(
        sir_with_vaccination,
        (0, 200),
        y0,
        args=(beta, gamma, N),
        dense_output=True,
        max_step=1.0
    )
    t_eval = np.linspace(0, 200, 1000)
    y_eval = sol.sol(t_eval)
    return y_eval[1].max()  # 感染高峰

# ================================================================
# 第一步：視覺化目標函數（在最佳化前先看）
# ================================================================
v_range = np.linspace(0, 0.95, 50)
peaks = [peak_infected(v) for v in v_range]

plt.figure(figsize=(8, 4))
plt.plot(v_range * 100, peaks, 'b-o', markersize=3)
plt.xlabel('接種比例（%）')
plt.ylabel('感染高峰人數')
plt.title('接種比例 vs 感染高峰')
plt.axhline(y=min(peaks), color='red', linestyle='--', alpha=0.5, label=f'最小值：{min(peaks):.0f} 人')
plt.legend()
plt.tight_layout()
plt.show()

# ================================================================
# 第二步：用 minimize_scalar 找最佳接種比例
# ================================================================
# minimize_scalar 是專為一維問題設計的，比 minimize 更精確
result = minimize_scalar(
    peak_infected,
    bounds=(0.0, 0.99),
    method='bounded'     # bounded 方法：搜尋範圍限制在 [0, 0.99]
)

print("=" * 50)
print("最佳化結果：")
print(f"  最佳接種比例：{result.x * 100:.2f}%")
print(f"  對應感染高峰：{result.fun:.0f} 人")
print(f"  最佳化是否成功：{result.success}")
print(f"  目標函數評估次數：{result.nfev}")

# ================================================================
# 第三步：敏感度分析 — 不同接觸率下的最佳接種比例
# ================================================================
beta_values = np.linspace(0.1, 0.6, 10)
optimal_v_list = []
min_peaks = []

for beta in beta_values:
    # 對每個 beta，重新找最佳 v
    res = minimize_scalar(
        lambda v: peak_infected(v, beta=beta),
        bounds=(0.0, 0.99),
        method='bounded'
    )
    optimal_v_list.append(res.x)
    min_peaks.append(res.fun)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(beta_values, [v * 100 for v in optimal_v_list], 'g-o')
axes[0].set_xlabel('接觸率 β')
axes[0].set_ylabel('最佳接種比例（%）')
axes[0].set_title('敏感度分析：β vs 最佳接種比例')

axes[1].plot(beta_values, min_peaks, 'r-o')
axes[1].set_xlabel('接觸率 β')
axes[1].set_ylabel('最小感染高峰人數')
axes[1].set_title('敏感度分析：β vs 最小可達感染高峰')

plt.suptitle('SIR 模型：接種策略敏感度分析', fontsize=13)
plt.tight_layout()
plt.savefig('sir_sensitivity.png', dpi=150)
plt.show()

print("\n敏感度分析摘要：")
print(f"  β 最小（{beta_values[0]:.1f}）時，最佳接種比例：{optimal_v_list[0]*100:.1f}%")
print(f"  β 最大（{beta_values[-1]:.1f}）時，最佳接種比例：{optimal_v_list[-1]*100:.1f}%")
print("  結論：接觸率越高，需要更高比例的接種才能控制疫情")
```

**示範過程的刻意錯誤**：
- 把目標函數寫成最大化（沒有加負號），讓學生觀察 minimize 的結果不對
- 問學生：「minimize 找到的 v = 0（不接種），這合理嗎？哪裡出錯了？」

---

### 學生實作（35 分鐘）

**任務說明**：
「把今天學的最佳化加進你的專題程式。第一步：把你的模型輸出轉換成一個可以最小化的目標函數。」

**巡視重點**：
- 學生是否能定義出目標函數（能被 minimize 呼叫的函數）
- 目標函數是否回傳一個純量數字（不是陣列）
- 最佳化結果是否合理（先做直覺檢查）

**進度快的學生的延伸題**：
「你的最佳化結果對初始點 x0 敏感嗎？試試 5 個不同的 x0，看結果是否一致。」

---

### 回顧與預告（20 分鐘）

**本週三個關鍵點**（白板寫下）：
1. 目標函數必須回傳一個純量；要最大化 f 就最小化 -f
2. `minimize` 只保證找到局部最佳解——若初始點不同，結果可能不同
3. 敏感度分析 = 固定其他，改變一個參數，畫圖看最佳解如何移動

**作業說明**：
- 在專題程式中加入完整的最佳化流程 + 至少一張敏感度分析圖
- 繳交：`.py` + 圖表 + 3–5 句話說明「最佳解是什麼、為什麼合理」

**預告下週**：「下週是技術寫作——你要把你做的事情用文字說清楚。」

---

## 常見問題與處理方式

| 問題 | 處理方式 |
|------|----------|
| `minimize` 報錯：目標函數回傳陣列 | 確認目標函數最後回傳的是純量（一個數字），不是 list 或 array |
| 最佳化結果在邊界（如 x=0 或 x=1） | 可能是邊界條件就是最佳解（完全合理）；但也可能目標函數方向反了 |
| `minimize` 找不到合理的解（x 很奇怪） | 檢查目標函數是否有 bug；先手動試幾個 x 值，確認趨勢對了再跑最佳化 |
| 學生問「局部解有什麼用？」 | 說明：工程上很多問題局部解就夠用；若想找全局解，可用 `scipy.optimize.differential_evolution` |
| 學生的問題難以定義目標函數 | 讓學生先回答「我最終希望最大化/最小化什麼？」如果回答不出來，代表問題還不清楚 |
