---
title: Level 3 週 04 — SciPy 簡單 ODE 模型（作業單）
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

# Level 3 週 04 作業單
## 作業名稱：ODE 模擬程式 + 時間演化圖

---

## 作業目標

建立一個完整的 ODE 模擬程式，模擬一個動態系統的時間演化，並畫出時間演化圖。重點是：選一個有實際意義的系統，能解釋圖形所呈現的行為。

---

## 繳交清單

| # | 項目 | 格式 | 必/選 |
|---|------|------|-------|
| 1 | ODE 模擬主程式 | `.py` 或 `.ipynb` | 必交 |
| 2 | 時間演化圖（至少一張） | 圖嵌在 notebook，或另存 `.png` | 必交 |
| 3 | 系統說明（見要求 2） | 程式碼注釋或 Markdown 格子 | 必交 |
| 4 | 參數實驗結果（見要求 4） | 在同一個程式檔案內 | 必交 |

**繳交方式**：上傳至課程 Google Drive  
**繳交期限**：Week 5 上課前 24 小時

---

## 具體要求

### 要求 1：選擇一個有意義的動態系統

必須選擇以下之一，或設計你自己的（需事先得到老師確認）：

| 選項 | 系統 | 方程式 |
|------|------|--------|
| A | Logistic 族群增長 | `dP/dt = r P (1 - P/K)` |
| B | 牛頓冷卻定律 | `dT/dt = -k(T - T_env)` |
| C | SIR 流行病模型 | 見 worksheet |
| D | Lotka-Volterra 捕食者-獵物 | 見 worksheet |
| E | 阻尼彈簧振動 | `dx/dt = v; dv/dt = -kx - cv` |
| F | 你的舊專題的動態系統 | 自行設計（需事先確認） |

建議：選一個你能用自己的話解釋「這個系統在描述什麼」的選項。

### 要求 2：在程式碼頂端用注釋說明系統

```python
# 系統說明：
# 我選擇的是：[系統名稱]
# 這個系統描述的是：[用一兩句話說明實際情境]
# ODE 方程式：[寫出你的 ODE]
# 參數意義：
#   r = 0.5：[說明這個參數是什麼]
#   K = 1000：[說明這個參數是什麼]
```

### 要求 3：程式必須包含以下步驟

1. **定義 ODE 函數**（正確簽名：`func(y, t)` 或 `func(y, t, *params)`）
2. **設定初始條件和時間軸**
3. **呼叫 `odeint` 求解**
4. **繪製時間演化圖**（有標題、x 軸標籤、y 軸標籤、圖例）
5. **印出至少一個有意義的數值結論**（例如：「感染高峰在第 X 天」、「族群在 t=Y 時達到攜帶容量的 90%」）

### 要求 4：進行至少一組參數實驗

改變你模型中的一個參數，在同一張圖上畫出兩條（或更多）不同參數下的時間演化曲線，並說明：「改變這個參數，系統的行為怎麼改變了？」

例如：
```python
# 實驗：改變傳播率 β，看感染高峰如何變化
for beta in [0.2, 0.3, 0.5]:
    sol = odeint(sir_model_with_beta, y0, t, args=(beta,))
    plt.plot(t, sol[:, 1], label=f"β={beta}")
```

### 要求 5：結果圖表的品質要求

- 圖表必須有：標題、x 軸標籤（含單位）、y 軸標籤（含單位）、圖例
- 建議圖表大小：`figsize=(8, 4)` 或 `(10, 5)`
- 如果有多條線，每條線用不同顏色，並在圖例中標明

---

## 範例：符合要求的完整作業（SIR 模型）

```python
"""
Level 3 Week 04 作業

系統說明：
我選擇的是：SIR 流行病模型
這個系統描述的是：在封閉族群中，傳染病從少數感染者開始傳播的過程
ODE 方程式：
  dS/dt = -β S I / N
  dI/dt =  β S I / N - γ I
  dR/dt =  γ I
參數意義：
  beta = 0.3：每天每個感染者接觸易感者的機率
  gamma = 0.1：感染者每天康復的機率
"""
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

N = 10000

def sir(y, t, beta, gamma):
    S, I, R = y
    dSdt = -beta * S * I / N
    dIdt =  beta * S * I / N - gamma * I
    dRdt =  gamma * I
    return [dSdt, dIdt, dRdt]

y0 = [N - 10, 10, 0]
t = np.linspace(0, 200, 1000)

# 基本模擬（β=0.3，γ=0.1）
sol = odeint(sir, y0, t, args=(0.3, 0.1))
S, I, R = sol[:, 0], sol[:, 1], sol[:, 2]

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 圖一：時間演化
axes[0].plot(t, S, label="易感者 S", color="steelblue")
axes[0].plot(t, I, label="感染者 I", color="coral")
axes[0].plot(t, R, label="康復者 R", color="green")
axes[0].set_xlabel("天")
axes[0].set_ylabel("人數")
axes[0].set_title("SIR 模型：標準情況（β=0.3, γ=0.1）")
axes[0].legend()

# 圖二：參數實驗（改變 β）
betas = [0.15, 0.3, 0.5]
colors = ["steelblue", "coral", "purple"]
for beta, color in zip(betas, colors):
    sol_b = odeint(sir, y0, t, args=(beta, 0.1))
    axes[1].plot(t, sol_b[:, 1], color=color, label=f"β={beta}")

axes[1].set_xlabel("天")
axes[1].set_ylabel("感染人數")
axes[1].set_title("不同傳播率 β 的感染曲線")
axes[1].legend()

plt.tight_layout()
plt.show()

# 數值結論
peak_day = t[np.argmax(I)]
print(f"感染高峰：第 {peak_day:.0f} 天，最高感染人數 = {I.max():.0f}")
print(f"最終康復人數 = {R[-1]:.0f}（{R[-1]/N*100:.1f}%）")
print(f"基本再生數 R₀ = β/γ = {0.3/0.1:.1f}")
# 參數實驗結論：傳播率 β 越大，感染高峰越早且越高
```

---

## 自我檢查清單

- [ ] 在程式碼頂端有系統說明（5 行注釋，含 ODE 方程式和參數意義）
- [ ] ODE 函數簽名正確（`func(y, t)` 或 `func(y, t, *params)`）
- [ ] 程式完整可執行，無報錯
- [ ] 時間演化圖有標題、x 軸標籤（含單位）、y 軸標籤（含單位）、圖例
- [ ] 至少進行一組參數實驗，在圖上畫出比較
- [ ] 末尾有至少一條印出的數值結論（有實際意義）
- [ ] 參數實驗後有文字說明（哪個參數怎麼影響系統）
