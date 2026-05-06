# 🚀 OLAP Engines

> ClickHouse / Doris / StarRocks 分析型引擎 (Analytical Engines)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Doris Rollup vs 物化视图](#q1-doris-rollup-vs-物化视图) | ⭐⭐ |
| 2 | [ClickHouse MergeTree merge 策略](#q2-clickhouse-mergetree-merge-策略) | ⭐⭐ |
| 3 | [StarRocks 存算分离架构](#q3-starrocks-存算分离架构) | ⭐⭐⭐ |
| 4 | [ClickHouse 分布式表 vs 本地表写入策略](#q4-clickhouse-分布式表-vs-本地表写入策略) | ⭐⭐ |
| 5 | [Doris Colocate Join 原理](#q5-doris-colocate-join-原理) | ⭐⭐⭐ |

---

## Q1. Doris Rollup vs 物化视图

> 📌 **频率**: 2025 · ★☆☆

### 🎯 核心要点

| 对比维度 | Rollup | Materialized View (Async MV) |
|----------|--------|------------------------------|
| 数据来源 | 单表 (Single Table) | 多表 JOIN / 复杂表达式 |
| 列要求 | 必须包含基础表部分列前缀作为 Key | 无限制 |
| 刷新方式 | **同步** (Synchronous) — 写入时自动更新 | **异步** (Asynchronous) — 定时/手动刷新 |
| 查询改写 | 优化器自动命中 (Auto Hit) | 优化器自动改写 (Query Rewrite) |
| 适用场景 | 简单聚合上卷 (Pre-aggregation) | 跨表预计算、复杂 ETL |

### 💡 类比记忆

> - **Rollup** = Excel 里的"分类汇总 (Subtotal)" — 在原表上加几行 SUM
> - **MV** = 建了张新视图表 (New View Table) — 可以从多张表 JOIN 出来

### 📝 示例

```sql
-- 基础表 (Base Table)
sales(date, city, product, amount)

-- Rollup: 同表预聚合 (Same-table Pre-aggregation)
ALTER TABLE sales ADD ROLLUP sales_by_city(date, city, SUM(amount));

-- 查询自动命中 Rollup，扫描量 1/100
SELECT date, city, SUM(amount) FROM sales GROUP BY date, city;
```

### 🧠 记忆锚点

```
Rollup = Single-table Pre-aggregation (同步)
MV = Multi-table View (异步 + Query Rewrite)
```

---

## Q2. ClickHouse MergeTree merge 策略

> 📌 **频率**: 2024-2025 · ★★☆

### 🎯 核心要点

```
写入流程 (Write Path):
  INSERT → 立即生成一个 Part (小文件夹, immutable)
                ↓
  后台异步 Merge (类 LSM-tree 大小分层)
                ↓
  小 Part → 中 Part → 大 Part
```

**Merge 触发条件 (Trigger Conditions)**：Part 数量 / 大小 / 年龄 (Age)

**⚠️ 风险**：频繁小批写入 (Frequent Small Inserts) → Part 暴增 → Merge 跟不上 → `Too many parts (300)` 报错

### 💡 类比记忆

> 好比快递站：每来一件就开一个箱子 (Part)，后台工人 (Background Merge Thread) 定时把小箱合成大箱。
> 如果 1 秒来 100 件 → 工人合并速度跟不上 → 站台堆爆 💥

### ✅ 最佳实践 (Best Practices)

| 策略 | 说明 |
|------|------|
| 客户端攒批 (Client-side Batching) | ≥ 1000 行 / 每秒一次 |
| Buffer 表 (Buffer Table) | 中间缓冲，自动 Flush |
| Kafka Engine | 用 Kafka Engine 做写入缓冲 |
| 调大阈值 | `parts_to_throw_insert` 临时兜底 |

### 🧠 记忆锚点

```
LSM-tree Style Merge | Write = Batch ≥1000 rows | Too many parts = 攒批不够
```

---

## Q3. StarRocks 存算分离架构

> 📌 **频率**: 2025 · ★★☆  
> `Storage-Compute Separation Architecture`

### 🎯 架构演进

```
┌─── 存算一体 (Coupled) ──────────────┐
│  FE (Frontend) + BE (Backend)        │
│  BE 同时负责存储 + 计算               │
│  ⚠️ 扩容需同时加存储和计算            │
└──────────────────────────────────────┘
          ↓ StarRocks 3.0+
┌─── 存算分离 (Decoupled) ────────────┐
│  FE (Frontend): 元数据 + 查询规划    │
│  CN (Compute Node): 无状态计算节点   │
│  S3 / HDFS: 共享存储 (Shared Storage)│
│  Cache: 本地热数据缓存 (Local Cache) │
│                                      │
│  ✅ 计算弹性伸缩 (Elastic Scaling)   │
│  ✅ 存储成本低 (S3 = 1/10 SSD)      │
│  ✅ 多集群共享数据                    │
└──────────────────────────────────────┘
```

### 📊 对比

| 维度 | 存算一体 (Coupled) | 存算分离 (Decoupled) |
|------|-------------------|---------------------|
| 扩容 | 计算+存储绑定 | 独立扩缩 (Independent Scaling) |
| 成本 | SSD 存储贵 | S3/HDFS 便宜 |
| 弹性 | 固定集群 | Serverless / Auto-scaling |
| 延迟 | 本地 SSD 快 | 需 Cache 加速 |
| 多租户 | 资源竞争 | 多 CN 集群隔离 (Multi-cluster Isolation) |

### 💡 类比记忆

> - **存算一体** = 自建机房 🏢 — 买多大服务器就有多大能力，闲置浪费
> - **存算分离** = 云上弹性 ☁️ — 存储用 S3（无限便宜），计算按需开关

### 🧠 记忆锚点

```
存算分离 = FE + Stateless CN + Shared Storage(S3) + Local Cache
好处: Independent Scaling + Low Cost + Multi-tenant Isolation
```

---

## Q4. ClickHouse 分布式表 vs 本地表写入策略

> 📌 **频率**: 2025 · ★★☆  
> `Distributed Table vs Local Table Write Strategy`

### 🎯 两种写入方式

```
方式1: 写 Distributed 表 (分布式表)
  Client → Distributed Table → 自动路由到各 Shard 的 Local Table
  ⚠️ 问题: 中间 buffer 可能丢数据 + 额外网络开销

方式2: 直接写 Local 表 ✅ 推荐
  Client → 自行路由 → 直接写对应 Shard 的 Local Table
  ✅ 无中间缓冲，数据更安全
```

### 📊 对比

| 维度 | 写 Distributed 表 | 写 Local 表 |
|------|-------------------|-------------|
| 简便性 | 简单（自动路由） | 需客户端自行路由 |
| 数据安全 | 有丢失风险（异步转发） | 安全（直接写入） |
| 性能 | 有额外网络开销 | 最优 |
| 一致性 | 可能有延迟 | 强一致 |
| 适用场景 | 开发/测试 | **生产推荐** |

### 📝 最佳实践

```
生产环境:
1. 建 Local 表 (MergeTree) on each shard
2. 建 Distributed 表 → 仅用于查询 (SELECT only)
3. 写入时: 客户端 hash(sharding_key) → 确定目标 shard → 直写 Local 表
4. 配合 Kafka Engine / Buffer Table 做攒批
```

### 💡 类比记忆

> - 写 Distributed = 寄快递给总部，总部再分发（可能丢件）📮→🏢→📦
> - 写 Local = 直接送到目的地门店（稳妥）📮→📦

### 🧠 记忆锚点

```
Distributed = 仅用于读 | Local = 用于写 | 客户端自行 Sharding
```

---

## Q5. Doris Colocate Join 原理

> 📌 **频率**: 2025 · ★★☆  
> `Colocate Join` = 本地关联 (Local Join without Shuffle)

### 🎯 核心原理

```
普通 Join (Shuffle Join):
  Table A (Shard 1,2,3) ──┐
                           ├─ Shuffle 重分布 → Join
  Table B (Shard 1,2,3) ──┘
  ⚠️ 大量网络传输 (Network I/O)

Colocate Join:
  Table A (Shard 1) + Table B (Shard 1) → Local Join on Node 1
  Table A (Shard 2) + Table B (Shard 2) → Local Join on Node 2
  Table A (Shard 3) + Table B (Shard 3) → Local Join on Node 3
  ✅ 零 Shuffle，直接本地 Join
```

### 📝 使用条件

| 条件 | 说明 |
|------|------|
| 同一 Colocate Group | 两表声明在同一组 |
| 相同 Bucket 数 | 分桶数必须一致 |
| 相同分桶列 (Bucket Column) | 按相同列 Hash 分桶 |
| 相同副本数 (Replica) | 副本分布完全一致 |

### 📝 建表示例

```sql
-- 两表使用相同 Colocate Group
CREATE TABLE orders (
  order_id BIGINT, user_id BIGINT, amount DECIMAL
) DISTRIBUTED BY HASH(user_id) BUCKETS 32
PROPERTIES ("colocate_with" = "user_group");

CREATE TABLE users (
  user_id BIGINT, name STRING, city STRING
) DISTRIBUTED BY HASH(user_id) BUCKETS 32
PROPERTIES ("colocate_with" = "user_group");

-- Join 时自动走 Colocate Join（无 Shuffle）
SELECT * FROM orders o JOIN users u ON o.user_id = u.user_id;
```

### 💡 类比记忆

> - **Shuffle Join** = 两个不同城市的公司开会 → 都飞到第三方会议室 ✈️
> - **Colocate Join** = 两个公司本来就在同一栋楼 → 直接去隔壁会议室 🚶

### 🧠 记忆锚点

```
Colocate = 同 Group + 同 Bucket 数 + 同分桶列 → Zero-Shuffle Local Join
```
