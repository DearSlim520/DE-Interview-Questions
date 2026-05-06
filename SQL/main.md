# 🧮 SQL

> SQL 面试经典题型 (Classic SQL Interview Questions)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [连续登录 N 天](#q1-连续登录-n-天) | ⭐⭐⭐ |
| 2 | [TopN 分组](#q2-topn-分组) | ⭐⭐ |
| 3 | [留存率](#q3-留存率) | ⭐⭐⭐ |
| 4 | [漏斗转化](#q4-漏斗转化) | ⭐⭐⭐ |
| 5 | [同环比](#q5-同环比) | ⭐⭐ |
| 6 | [行转列 / 列转行](#q6-行转列--列转行) | ⭐⭐ |

---

## Q1. 连续登录 N 天

> 📌 **频率**: 2025 超高频 · ★★★  
> `Consecutive Login N Days` — Window Function + Date Diff Grouping

### 🎯 核心思路

```
方法: 日期 - 行号 = 常数 (如果连续)

日期       行号    日期-行号
2025-01-01  1     2024-12-31  ← 同组 = 连续3天
2025-01-02  2     2024-12-31
2025-01-03  3     2024-12-31
2025-01-05  4     2025-01-01  ← 新组
```

### 📝 SQL 实现

```sql
WITH login_ranked AS (
  SELECT 
    user_id, 
    login_date,
    login_date - ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY login_date) AS grp
  FROM user_login
  GROUP BY user_id, login_date  -- 先去重
)
SELECT user_id, MIN(login_date) AS start_date, COUNT(*) AS consecutive_days
FROM login_ranked
GROUP BY user_id, grp
HAVING COUNT(*) >= 7;  -- 连续登录 ≥ 7 天的用户
```

### 💡 类比记忆

> 连续登录 = 拍照日历 📅 — 每天拍一张，编号从1开始。如果日期减编号得到的值一样，说明中间没断过。

### 🧠 记忆锚点

```
date - ROW_NUMBER() = 连续则为常数 → GROUP BY 这个常数 → COUNT ≥ N
```

---

## Q2. TopN 分组

> 📌 **频率**: 2025 高频 · ★★☆  
> `Top N per Group` — ROW_NUMBER vs RANK vs DENSE_RANK

### 🎯 三种排名函数对比

| 数据 | ROW_NUMBER | RANK | DENSE_RANK |
|------|-----------|------|------------|
| 100 | 1 | 1 | 1 |
| 100 | 2 | 1 | 1 |
| 90 | 3 | 3 | 2 |
| 80 | 4 | 4 | 3 |

### 📝 SQL 实现（每组 Top 3）

```sql
SELECT * FROM (
  SELECT 
    department,
    employee,
    salary,
    ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) AS rn
  FROM employees
) t
WHERE rn <= 3;
```

### 📊 选型

| 需求 | 用哪个 |
|------|--------|
| 严格取 N 条（不管并列） | `ROW_NUMBER` |
| 并列都取（可能超过 N 条） | `RANK` |
| 取前 N 名（不跳号） | `DENSE_RANK` |

### 🧠 记忆锚点

```
ROW_NUMBER = 严格序号(1,2,3,4) 不管并列
RANK = 并列同名次，跳号(1,1,3,4)
DENSE_RANK = 并列同名次，不跳号(1,1,2,3)
```

---

## Q3. 留存率

> 📌 **频率**: 2025 高频 · ★★★  
> `Retention Rate` — Day 1 / Day 7 / Day 30

### 🎯 公式

```
第N日留存率 (Day-N Retention) = 
  注册日后第N天活跃用户数 / 注册日新用户数 × 100%
```

### 📝 SQL 实现

```sql
SELECT
  a.register_date,
  COUNT(DISTINCT a.user_id) AS new_users,
  COUNT(DISTINCT b1.user_id) AS day1_retained,
  COUNT(DISTINCT b7.user_id) AS day7_retained,
  COUNT(DISTINCT b30.user_id) AS day30_retained,
  ROUND(COUNT(DISTINCT b1.user_id) * 100.0 / COUNT(DISTINCT a.user_id), 2) AS day1_rate,
  ROUND(COUNT(DISTINCT b7.user_id) * 100.0 / COUNT(DISTINCT a.user_id), 2) AS day7_rate
FROM new_users a
LEFT JOIN user_active b1 
  ON a.user_id = b1.user_id 
  AND b1.active_date = DATE_ADD(a.register_date, 1)
LEFT JOIN user_active b7 
  ON a.user_id = b7.user_id 
  AND b7.active_date = DATE_ADD(a.register_date, 7)
LEFT JOIN user_active b30 
  ON a.user_id = b30.user_id 
  AND b30.active_date = DATE_ADD(a.register_date, 30)
GROUP BY a.register_date;
```

### 💡 类比记忆

> 留存率 = 健身房续卡率 🏋️ — 1月1号办卡100人，1月2号来了40人 → 次日留存40%

### 🧠 记忆锚点

```
LEFT JOIN 活跃表 ON user_id + 日期偏移N天
留存率 = 第N天活跃 / 首日新增 × 100%
```

---

## Q4. 漏斗转化

> 📌 **频率**: 2025 · ★★☆  
> `Funnel Conversion Analysis`

### 🎯 核心思路

```
漏斗 = 多步骤有序行为，计算每步转化率 (Step-by-Step Conversion Rate)

示例: 浏览商品 → 加入购物车 → 提交订单 → 支付成功
      10000        5000          2000         1500
      100%         50%           20%          15%
```

### 📝 SQL 实现

```sql
SELECT
  COUNT(DISTINCT CASE WHEN step >= 1 THEN user_id END) AS step1_view,
  COUNT(DISTINCT CASE WHEN step >= 2 THEN user_id END) AS step2_cart,
  COUNT(DISTINCT CASE WHEN step >= 3 THEN user_id END) AS step3_order,
  COUNT(DISTINCT CASE WHEN step >= 4 THEN user_id END) AS step4_pay
FROM (
  SELECT user_id,
    MAX(CASE WHEN event = 'view' THEN 1 
             WHEN event = 'add_cart' AND view_time IS NOT NULL THEN 2
             WHEN event = 'order' AND cart_time IS NOT NULL THEN 3
             WHEN event = 'pay' AND order_time IS NOT NULL THEN 4 END) AS step
  FROM user_events
  WHERE dt = '2025-01-01'
  GROUP BY user_id
) t;
```

### ⚠️ 注意事项

- 漏斗需要有**时序要求** (Step 2 必须在 Step 1 之后)
- 需要设定**转化窗口** (如 30 分钟内完成)
- 用 `LEAD/LAG` 或 Window Function 保证顺序

### 🧠 记忆锚点

```
漏斗 = 多步骤有序行为 + 每步转化率
SQL: CASE WHEN 分步标记 → COUNT DISTINCT 每步人数
```

---

## Q5. 同环比

> 📌 **频率**: 2025 · ★★☆  
> `Year-over-Year (YoY) / Month-over-Month (MoM)` — LAG / LEAD

### 🎯 定义

| 指标 | English | 公式 |
|------|---------|------|
| 同比 | **YoY (Year-over-Year)** | (本期 - 去年同期) / 去年同期 × 100% |
| 环比 | **MoM (Month-over-Month)** | (本期 - 上期) / 上期 × 100% |

### 📝 SQL 实现

```sql
SELECT
  month,
  revenue,
  -- 环比 MoM (Month-over-Month)
  LAG(revenue, 1) OVER(ORDER BY month) AS last_month,
  ROUND((revenue - LAG(revenue, 1) OVER(ORDER BY month)) * 100.0 
        / LAG(revenue, 1) OVER(ORDER BY month), 2) AS mom_rate,
  -- 同比 YoY (Year-over-Year)
  LAG(revenue, 12) OVER(ORDER BY month) AS last_year_same_month,
  ROUND((revenue - LAG(revenue, 12) OVER(ORDER BY month)) * 100.0 
        / LAG(revenue, 12) OVER(ORDER BY month), 2) AS yoy_rate
FROM monthly_revenue;
```

### 💡 类比记忆

> - **同比 YoY** = 今年1月 vs 去年1月（同一时间点跨年对比）
> - **环比 MoM** = 这个月 vs 上个月（相邻时间对比）
> - `LAG(col, 1)` = 上个月 | `LAG(col, 12)` = 去年同月

### 🧠 记忆锚点

```
LAG(value, 1) = 上期(环比) | LAG(value, 12) = 去年同期(同比)
公式 = (本期 - 基期) / 基期 × 100%
```

---

## Q6. 行转列 / 列转行

> 📌 **频率**: 2025 · ★★☆  
> `Pivot (行转列) / Unpivot (列转行)`

### 🎯 行转列 (Pivot / Row to Column)

```
原始数据:                     转换后:
user  subject  score         user   math  english  chinese
A     math     90      →    A      90    85       92
A     english  85           B      88    90       87
A     chinese  92
B     math     88
```

```sql
-- 行转列 (CASE WHEN / PIVOT)
SELECT
  user_id,
  MAX(CASE WHEN subject = 'math' THEN score END) AS math,
  MAX(CASE WHEN subject = 'english' THEN score END) AS english,
  MAX(CASE WHEN subject = 'chinese' THEN score END) AS chinese
FROM scores
GROUP BY user_id;
```

### 🎯 列转行 (Unpivot / Column to Row)

```sql
-- 列转行 (UNION ALL / LATERAL VIEW EXPLODE)
SELECT user_id, 'math' AS subject, math AS score FROM scores_wide
UNION ALL
SELECT user_id, 'english', english FROM scores_wide
UNION ALL
SELECT user_id, 'chinese', chinese FROM scores_wide;

-- Hive: LATERAL VIEW
SELECT user_id, subject, score
FROM scores_wide
LATERAL VIEW EXPLODE(MAP('math', math, 'english', english)) t AS subject, score;
```

### 💡 类比记忆

> - **行转列 (Pivot)** = 竖着的成绩单 → 横着的成绩单（CASE WHEN + GROUP BY）
> - **列转行 (Unpivot)** = 横着的 → 竖着的（UNION ALL / EXPLODE）

### 🧠 记忆锚点

```
行转列: MAX(CASE WHEN col='x' THEN val END) + GROUP BY
列转行: UNION ALL 或 LATERAL VIEW EXPLODE
```
