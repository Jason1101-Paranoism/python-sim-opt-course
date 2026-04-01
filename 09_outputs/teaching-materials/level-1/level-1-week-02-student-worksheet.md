---
title: Level 1 週 2 學生講義 — 機率遊戲：彩票與生日悖論
type: teaching
level: "1"
week: 2
audience: student
status: draft
---

# Level 1 週 2：機率遊戲 — 彩票與生日悖論
## 學生講義（Student Worksheet）

---

## 核心概念

### 理論機率 vs 相對頻率

| | 理論機率 | 相對頻率 |
|--|---------|---------|
| 定義 | 用數學計算出來 | 用實驗/模擬觀察到的 |
| 例子 | P(擲到 6) = 1/6 | 跑了 1000 次，6 出現 172 次 → 172/1000 |
| 關係 | 實驗次數越多，相對頻率越趨近於理論機率 |

> **模擬的核心價值**：當數學很難計算（或根本沒有公式）時，用大量模擬代替理論推導。

---

## 選項 A：彩票模擬

### 大樂透規則
- 從 1–49 選 6 個不重複號碼
- 6 個全中 → 頭獎
- 中頭獎機率 ≈ 1 / 13,983,816（約 0.000007%）

### 完整程式碼

```python
import random

def buy_lottery_ticket():
    """隨機選 6 個 1–49 的不重複號碼（排序）"""
    return sorted(random.sample(range(1, 50), 6))

def check_win(ticket, winning_numbers):
    """比對是否完全相同"""
    return ticket == winning_numbers

# 設定本期開獎號碼
winning = sorted(random.sample(range(1, 50), 6))
print(f"本期開獎：{winning}")

# 模擬買彩票
trials = 100000
wins = 0

for _ in range(trials):
    ticket = buy_lottery_ticket()
    if check_win(ticket, winning):
        wins += 1

print(f"買了 {trials} 張，中頭獎 {wins} 次")
print(f"模擬中獎率：{wins/trials:.8f}")
print(f"理論中獎率：{1/13983816:.8f}")
```

### `random.sample` 是什麼？

```python
random.sample(range(1, 50), 6)
# 從 1–49 中，不放回地隨機取 6 個數字
# 每個數字最多出現一次（這才是正確的彩票規則）

# 對比：random.choices（放回，可重複）
# random.randint（只取一個整數）
```

### 試試看

1. 把 `trials` 改成 1000、10000、1000000，觀察模擬中獎率和理論值的差距
2. 計算：「如果每週買 1 張，平均需要買幾年才中一次頭獎？」

```python
# 提示：理論中獎率的倒數就是「平均要買幾張」
avg_tickets = 1 / (1/13983816)
avg_years = avg_tickets / 52   # 每年 52 週
print(f"平均需要買 {avg_years:.0f} 年")
```

---

## 選項 B：生日悖論

### 問題
> 一個班有 23 個人，其中有兩人生日同一天的機率是多少？

你的直覺猜測：______ %

### 完整程式碼

```python
import random
import matplotlib.pyplot as plt

def simulate_birthday_match(n_people):
    """模擬 n 個人，回傳是否有兩人同生日"""
    birthdays = [random.randint(1, 365) for _ in range(n_people)]
    # set() 會移除重複值；若長度變短，代表有重複
    return len(birthdays) != len(set(birthdays))

# 針對不同人數，計算配對機率
trials = 10000
people_counts = list(range(2, 61))   # 2 到 60 人
probabilities = []

for n in people_counts:
    matches = sum(simulate_birthday_match(n) for _ in range(trials))
    probabilities.append(matches / trials)

# 畫折線圖
plt.figure(figsize=(10, 5))
plt.plot(people_counts, probabilities, marker='o', markersize=3)
plt.axhline(y=0.5, color='red', linestyle='--', label='50% 機率線')
plt.axvline(x=23, color='orange', linestyle='--', label='23 人')
plt.xlabel("班級人數")
plt.ylabel("有兩人同生日的機率")
plt.title("生日悖論：班級人數 vs 配對機率")
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

### 結果觀察

- 23 人時，機率約是：______%
- 機率超過 50% 大約需要幾人：______人
- 這和你的直覺一樣嗎？為什麼有這個「悖論」？

_______________________________________________

---

## 本週關鍵詞

| 詞彙 | 意思 |
|------|------|
| `random.sample(pool, k)` | 從 pool 中不放回地取 k 個元素 |
| `set()` | 自動移除重複值的資料結構 |
| 相對頻率 | 實驗中某事件發生次數 / 總實驗次數 |
| 生日悖論 | 人數遠小於 365 時，同生日機率卻遠高於直覺 |
