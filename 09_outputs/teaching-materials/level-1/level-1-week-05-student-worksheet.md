---
title: Level 1 週 5 學生講義 — 個人小專題
type: teaching
level: "1"
week: 5
audience: student
status: draft
---

# Level 1 週 5：個人小專題構思與實作
## 學生講義（Student Worksheet）

---

## 你的模擬設計圖

在開始寫程式之前，先回答這幾個問題：

**我要模擬的是：**
_______________________________________________

**系統裡隨機的部分是：**（這裡的「隨機」讓你不能直接計算答案）
_______________________________________________

**我想觀察的輸出是：**（一個數字？一條時間序列曲線？一個機率？）
_______________________________________________

**我打算用哪個模板：**
- [ ] 模板 A：機率 / 隨機事件（`simulate_once` + 多次重複）
- [ ] 模板 B：排隊 / 事件驅動（時間軸推進）
- [ ] 模板 C：差分 / 動態系統（逐步更新狀態）

---

## 三個模板提醒

### 模板 A：機率類
```python
import random

def simulate_once(params):
    # 做一件隨機的事，回傳結果
    return result

N = 1000
results = [simulate_once(params) for _ in range(N)]
# 統計：平均、最大、成功次數
# 畫直方圖
```

### 模板 C：差分類（最常用）
```python
def simulate_system(initial_state, params, steps):
    state = [initial_state]
    for t in range(steps):
        next_state = state[-1] + change_rule(state[-1], params)
        state.append(next_state)
    return state
# 畫折線圖
```

---

## 最小可行程式（MVP）原則

> 不要一開始就寫完整的版本。先讓程式能跑、能輸出任何東西。

**第一步**：寫一個能跑的空殼
```python
def simulate_once():
    return 0   # 先回傳假值，之後再改

print(simulate_once())   # 確認能跑
```

**第二步**：加入真實邏輯（一次只加一個）

**第三步**：加入迴圈和圖表

---

## 本週你要帶走的

- [ ] 一個**可以執行**的程式（哪怕結果很粗糙）
- [ ] 至少一個輸出（數字或圖都可以）
- [ ] 能用一句話說清楚「我的程式在模擬什麼」
