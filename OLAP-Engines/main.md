# 🚀 OLAP Engines

> ClickHouse / Doris / StarRocks 分析型引擎

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Doris Rollup vs 物化视图](#q1-doris-rollup-vs-物化视图) | ⭐⭐ |
| 2 | [ClickHouse MergeTree merge 策略 + Too many parts](#q2-clickhouse-mergetree-merge-策略) | ⭐⭐ |
| 3 | StarRocks 存算分离架构 | - |
| 4 | ClickHouse 分布式表 vs 本地表写入策略 | - |
| 5 | Doris Colocate Join 原理 | - |

---

## Q1. Doris Rollup vs 物化视图

> 📌 **频率**: 2025 · ★☆☆

### 🎯 核心要点

| 对比维度 | Rollup | 物化视图 (Async MV) |
|----------|--------|---------------------|
| 数据来源 | 单表 | 多表 JOIN / 复杂表达式 |
| 列要求 | 必须包含基础表部分列前缀作为 Key | 无限制 |
| 刷新方式 | **同步**（写入时自动更新） | **异步**（定时/手动刷新） |
| 查询改写 | 优化器自动命中 | 优化器自动改写 |
| 适用场景 | 简单聚合上卷 | 跨表预计算、复杂 ETL |

### 💡 类比记忆

> - **Rollup** = Excel 里的"分类汇总" — 在原表上加几行 SUM
> - **MV** = 建了张新视图表 — 可以从多张表 JOIN 出来

### 📝 示例

```sql
-- 基础表
sales(date, city, product, amount)

-- Rollup: 同表预聚合
ALTER TABLE sales ADD ROLLUP sales_by_city(date, city, SUM(amount));

-- 查询自动命中 Rollup，扫描量 1/100
SELECT date, city, SUM(amount) FROM sales GROUP BY date, city;
```

### 🧠 记忆锚点

```
Rollup = 同表预聚合 ｜ MV = 跨表新视图
```

---

## Q2. ClickHouse MergeTree merge 策略

> 📌 **频率**: 2024-2025 · ★★☆

### 🎯 核心要点

```
写入流程:
  INSERT → 立即生成一个 part (小文件夹)
                ↓
  后台异步 merge (类 LSM 大小分层)
                ↓
  小 part → 中 part → 大 part
```

**Merge 触发条件**：part 数量 / 大小 / 年龄

**⚠️ 风险**：频繁小批写入 → part 暴增 → merge 跟不上 → `Too many parts (300)` 报错

### 💡 类比记忆

> 好比快递站：每来一件就开一个箱子，后台工人定时把小箱合成大箱。
> 如果 1 秒来 100 件 → 工人合并速度跟不上 → 站台堆爆 💥

### ✅ 最佳实践

| 策略 | 说明 |
|------|------|
| 客户端攒批 | ≥ 1000 行 / 每秒一次 |
| Buffer 表 | 中间缓冲，自动 flush |
| Kafka 引擎 | 用 Kafka Engine 做写入缓冲 |
| 调大阈值 | `parts_to_throw_insert` 临时兜底 |

### 🧠 记忆锚点

```
LSM 分层合并 ｜ 写要攒批
```
