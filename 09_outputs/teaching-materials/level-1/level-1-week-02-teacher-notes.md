---
title: Level 1 週 2 教師教案 — 機率遊戲：彩票與生日悖論
type: teaching
level: "1"
week: 2
audience: teacher
status: draft
source_files:
  - 01_curriculum/level-1.md
  - 02_teaching/class-structure.md
---

# Level 1 週 2：機率遊戲 — 彩票與生日悖論
## 教師教案（Teacher Notes）

---

### 本週學習目標

1. 學生能說明「機率」與「相對頻率」的差異
2. 學生能寫出彩票或生日悖論的模擬程式
3. 學生能用圖表呈現「模擬次數 vs 穩定性」的關係

---

### 課堂時間分配（120 分鐘）

| 段落 | 時間 | 內容 |
|------|------|------|
| 暖身回顧 | 10 分鐘 | 回顧上週作業、引入機率概念 |
| 概念講解 | 25 分鐘 | 理論機率 vs 相對頻率、兩個遊戲情境介紹 |
| 範例示範 | 30 分鐘 | Live coding：彩票模擬（主線）+ 生日悖論（進階） |
| 學生實作 | 35 分鐘 | 學生選一個做，完成模擬並觀察結果 |
| 回顧與預告 | 20 分鐘 | 總結、作業說明、預告排隊系統 |

---

### 暖身回顧（10 分鐘）

1. 請 2–3 位學生分享上週骰子程式的截圖（1 分鐘/人）
2. 提問：「N 很小的時候，結果為什麼不像我們預期的那樣均勻？」
   → 引出「一次結果 ≠ 真實機率」的概念
3. 過渡：「今天我們來玩更有趣的機率遊戲——彩票和生日悖論」

---

### 概念講解（25 分鐘）

**理論機率 vs 相對頻率（10 分鐘）**

| | 理論機率 | 相對頻率 |
|--|---------|---------|
| 怎麼得到 | 數學計算 | 實驗觀察 |
| 例子 | 擲骰子 P(6) = 1/6 | 跑 1000 次，6 出現了 172 次 → 172/1000 |
| 什麼時候一樣 | 實驗次數 → 無限大時趨近 | |

> 「模擬做的事就是：用電腦快速跑很多次，讓相對頻率趨近於理論機率。」

**彩票情境（7 分鐘）**

- 台灣大樂透：從 1–49 選 6 個號碼，全中才是頭獎
- 頭獎機率：C(49,6) 的倒數，約 1/13,983,816
- 問學生：「如果你每週買一張，要買多少年才可能中一次頭獎？」

**生日悖論情境（8 分鐘）**

- 問：「一個班 23 個人，有兩個人生日同一天的機率有多高？」
- 學生通常猜很低（23/365 ≈ 6%）
- 答案：超過 50%！（這是悖論）
- 不需要講數學，今天用模擬來「驗證」這件事

---

### 範例示範（30 分鐘）

**主線：彩票模擬（20 分鐘）**

```python
import random

def buy_lottery_ticket():
    """買一張大樂透，隨機選 6 個 1–49 的號碼"""
    return sorted(random.sample(range(1, 50), 6))

def check_win(ticket, winning_numbers):
    """檢查是否中頭獎"""
    return ticket == winning_numbers

# 模擬
winning = sorted(random.sample(range(1, 50), 6))
print(f"本期開獎號碼：{winning}")

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

> 「`random.sample` 是什麼？它和 `randint` 有什麼不同？」（讓學生猜）

**進階選項：生日悖論（10 分鐘，對進度快的學生）**

```python
import random

def simulate_birthday_match(n_people):
    """模擬 n 個人裡有沒有兩人同生日"""
    birthdays = [random.randint(1, 365) for _ in range(n_people)]
    return len(birthdays) != len(set(birthdays))  # 有重複就是 True

# 針對不同人數，計算配對機率
trials = 10000
for n in [10, 20, 23, 30, 40, 50]:
    matches = sum(simulate_birthday_match(n) for _ in range(trials))
    print(f"{n} 人：配對機率 = {matches/trials:.2%}")
```

---

### 學生實作（35 分鐘）

**讓學生選一個做**（彩票 or 生日悖論）：

彩票組任務：
1. 完成彩票模擬，觀察 100、1000、100000 次的模擬中獎率
2. 算算：「如果每週買 1 張，平均要買幾年才能中一次？」（程式計算或手算）

生日悖論組任務：
1. 完成各人數的配對機率表
2. 畫折線圖：x 軸是人數（1–60），y 軸是配對機率

**老師巡視重點**：
- 確認 `random.sample` vs `random.randint` 的用法理解
- 對完成快的學生：「你能讓彩票程式輸出『平均要買幾張才中獎』嗎？」

---

### 回顧與預告（20 分鐘）

**本週 3 個關鍵點**：
1. 理論機率是數學算的；相對頻率是實驗觀察到的——模擬讓兩者靠近
2. 彩票的中獎率低到幾乎無法直接模擬出來（要跑幾千萬次）
3. 生日悖論告訴我們：直覺在機率上很容易出錯，模擬可以幫忙驗證

**常見問題處理**：

| 問題 | 處理方式 |
|------|----------|
| `random.sample` 和 `random.choices` 的差異 | sample 不放回、choices 放回；彩票要用 sample |
| 跑 100000 次太慢 | 說明這是正常的；之後 Level 3 會學 NumPy 讓它快很多 |
| 生日悖論的圖怎麼畫 | 提示用 `plt.plot(x_list, y_list)` |
