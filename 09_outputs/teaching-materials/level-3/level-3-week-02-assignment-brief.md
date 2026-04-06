---
title: Level 3 週 02 — NumPy 陣列與向量化（作業單）
type: teaching
level: 3
week: 2
audience: student
status: draft
source_files:
  - 01_curriculum/level-3.md
  - 02_teaching/teaching-principles.md
  - 02_teaching/class-structure.md
  - 02_teaching/one-on-one-framework.md
---

# Level 3 週 02 作業單
## 作業名稱：NumPy 向量化改寫

---

## 作業目標

把你舊專題中一段「核心的數值計算」從 for 迴圈版本改寫成 NumPy 向量化版本，並提交計時比較與結果驗證。

這是 Level 3 最重要的練習之一：改寫的過程會讓你真正理解向量化的原理，而不只是「知道 NumPy 比較快」。

---

## 繳交清單

| # | 項目 | 格式 | 必/選 |
|---|------|------|-------|
| 1 | 改寫程式碼 | `.py` 或 `.ipynb` | 必交 |
| 2 | 執行結果截圖 | `.png` 或直接在 notebook 中顯示 | 必交 |
| 3 | 改寫說明（見要求 4） | 在程式碼頂端或 notebook 第一格，用 Markdown 或注釋 | 必交 |

**繳交方式**：上傳至課程 Google Drive 資料夾  
**繳交期限**：Week 3 上課前 24 小時

---

## 具體要求

### 要求 1：保留原始版本與 NumPy 版本

你的程式碼中必須同時包含兩個版本：

```python
# ── 原始版本（for 迴圈） ──
def compute_v1(data):
    result = []
    for x in data:
        result.append(...)  # 你的原始計算
    return result

# ── NumPy 版本 ──
def compute_v2(data):
    arr = np.array(data)
    return ...  # 向量化計算
```

### 要求 2：提供計時比較

用 `time.perf_counter` 分別計時兩個版本，並印出：
- 各版本執行時間（毫秒）
- 加速倍率（v1 時間 / v2 時間）

測試資料量至少要讓計時有意義（建議至少 10,000 個元素，以看出差異）。

### 要求 3：驗證結果一致性

使用以下方式之一驗證兩個版本結果相同：

```python
import numpy.testing as npt
npt.assert_allclose(result_v1, result_v2, rtol=1e-8)
print("✓ 結果一致")
```

或：
```python
import numpy as np
print(f"結果一致：{np.allclose(result_v1, result_v2)}")
```

如果結果不完全一致，說明原因（例如：浮點計算順序差異、累積誤差）。

### 要求 4：在程式碼頂端加上 3 行說明

```python
# 改寫說明
# 原始計算：[用一句話描述你改寫的是什麼計算]
# 改寫方式：[說明你用了哪些 NumPy 函數或操作]
# 加速結果：[例如「加速 87x，從 230ms 降到 2.6ms」]
```

### 要求 5：程式必須完整可執行

不要只繳交片段程式碼。你的 `.py` 或 `.ipynb` 必須從頭到尾執行不出錯，包含：
- `import` 語句
- 測試資料的生成
- 兩個版本的定義與呼叫
- 計時輸出
- 結果驗證

---

## 範例：符合要求的完整作業

```python
"""
Level 3 Week 02 作業
改寫說明：
原始計算：計算 N 個粒子的動能（E = 0.5 * m * v^2）
改寫方式：用 NumPy 的 array 操作取代 for 迴圈，改用 np.array 向量化
加速結果：加速 74x，從 185ms 降到 2.5ms（N=100,000）
"""
import numpy as np
import numpy.testing as npt
import time

N = 100_000
m = 1.5  # 質量

# 生成隨機速度（模擬真實資料）
velocities = list(np.random.normal(5.0, 1.0, N))

# ── 原始版本（for 迴圈） ──
def kinetic_energy_v1(velocities, mass):
    energies = []
    for v in velocities:
        energies.append(0.5 * mass * v ** 2)
    return energies

# ── NumPy 版本 ──
def kinetic_energy_v2(velocities, mass):
    v_arr = np.array(velocities)
    return 0.5 * mass * v_arr ** 2

# 計時
t0 = time.perf_counter()
r1 = kinetic_energy_v1(velocities, m)
t1 = time.perf_counter()
r2 = kinetic_energy_v2(velocities, m)
t2 = time.perf_counter()

print(f"for 迴圈版本：{(t1-t0)*1000:.1f} ms")
print(f"NumPy 版本：  {(t2-t1)*1000:.1f} ms")
print(f"加速倍率：    {(t1-t0)/(t2-t1):.0f}x")

# 驗證
npt.assert_allclose(r1, r2, rtol=1e-10)
print("✓ 兩版本結果完全一致")

# 附加：計算總動能
total_KE = np.array(r2).sum()
print(f"總動能：{total_KE:.2f} J")
```

---

## 自我檢查清單

繳交前確認：

- [ ] 程式碼從頭到尾可以執行，沒有報錯
- [ ] 同時保留 for 迴圈版本與 NumPy 版本
- [ ] 計時結果顯示出加速效果（如果 NumPy 版本沒有更快，請在說明中說明原因）
- [ ] 結果驗證通過（`assert_allclose` 或 `allclose` 為 True）
- [ ] 程式碼頂端有 3 行說明（原始計算、改寫方式、加速結果）
- [ ] 測試資料量足夠（至少 10,000 個元素）
- [ ] 檔案已上傳，格式為 `.py` 或 `.ipynb`

---

## 如果你遇到困難

**「我的 for 迴圈好像沒辦法向量化」**：很可能你的計算有累積依賴（每步依賴上一步的結果）。這種情況下，部分向量化也算——把資料生成和非依賴性的計算改成 NumPy，依賴性的部分先保留。

**「兩個版本的結果不一樣」**：先印出兩個結果的前幾個值，看看差異有多大。如果差異在 `1e-12` 以下，通常是正常的浮點誤差。如果差異很大，代表你的 NumPy 版本有邏輯錯誤。

**「NumPy 版本反而更慢」**：如果你的資料量很小（例如 100 個），NumPy 的啟動開銷可能比 for 迴圈還大。把資料量加大到 100,000 再測試。
