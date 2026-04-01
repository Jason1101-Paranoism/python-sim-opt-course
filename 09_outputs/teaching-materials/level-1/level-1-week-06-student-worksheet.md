---
title: Level 1 週 6 學生講義 — 視覺化
type: teaching
level: "1"
week: 6
audience: student
status: draft
---

# Level 1 週 6：視覺化 — 用圖說故事
## 學生講義（Student Worksheet）

---

## 選哪種圖？

| 我想展示的是… | 用這種圖 |
|-------------|---------|
| 某個數值隨時間的變化 | 折線圖 `plt.plot()` |
| 一組數值的分佈（多的、少的） | 直方圖 `plt.hist()` |
| 兩個變數之間的關係 | 散點圖 `plt.scatter()` |
| 多組數值的比較 | 折線圖（多條）或並排直方圖 |

---

## 一張好圖需要有什麼？

```python
plt.figure(figsize=(8, 5))        # 設定圖的大小

# 畫圖（折線圖為例）
plt.plot(x_data, y_data, label="我的資料", color='steelblue', linewidth=2)

# 必要元素
plt.xlabel("x 軸代表什麼（單位）")
plt.ylabel("y 軸代表什麼（單位）")
plt.title("這張圖告訴我們什麼發現")
plt.legend()                       # 如果有 label，記得加

# 可選元素
plt.grid(True, alpha=0.3)         # 淡淡的格線
plt.axhline(y=某個值, color='red', linestyle='--', label='參考線')
plt.tight_layout()                 # 自動調整間距

plt.savefig("my_plot.png", dpi=150)   # 存成圖檔（比截圖清楚）
plt.show()
```

---

## 常用技巧

### 加平均值參考線
```python
avg = sum(data) / len(data)
plt.axhline(y=avg, color='red', linestyle='--', label=f'平均值 = {avg:.2f}')
```

### 在同一張圖畫多條線
```python
plt.plot(x, y1, label='情境 A', color='blue')
plt.plot(x, y2, label='情境 B', color='orange')
plt.legend()
```

### 並排兩張圖
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
ax1.hist(data1, bins=20)
ax1.set_title("情境 A")
ax2.hist(data2, bins=20)
ax2.set_title("情境 B")
plt.tight_layout()
```

---

## 本週任務

為你的個人小專題：

1. 選出最能說明你發現的 **2–3 個問題**
2. 為每個問題選一種合適的圖表類型
3. 用上面的模板，把圖做完整（標題、軸標籤、圖例）
4. 在每張圖下面寫一句「圖說」：這張圖告訴我什麼？

**圖說範例**：
> 「當服務率從 0.2 提升到 0.5，平均等待時間從 15 分鐘降至 3 分鐘，顯示服務速度的些微提升可以大幅改善顧客體驗。」
