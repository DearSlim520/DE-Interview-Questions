# 🚀 OLAP Engines

> ClickHouse（主要）/ Snowflake / BigQuery / Redshift · Doris / StarRocks（选读）

## 题目列表

| # | 题目 | 难度 | 标签 |
|---|------|------|------|
| 1 | [ClickHouse MergeTree merge 策略](#q1-clickhouse-mergetree-merge-策略) | ⭐⭐ | 核心 |
| 2 | [ClickHouse 分布式表 vs 本地表写入策略](#q2-clickhouse-分布式表-vs-本地表写入策略) | ⭐⭐ | 核心 |
| 3 | [OLAP 引擎选型：Cloud DW 对比](#q3-olap-引擎选型cloud-dw-对比) | ⭐⭐⭐ | 核心 |
| 4 | [StarRocks 存算分离架构](#q4-starrocks-存算分离架构) | ⭐⭐⭐ | 选读 |
| 5 | [Doris Rollup vs 物化视图](#q5-doris-rollup-vs-物化视图) | ⭐⭐ | 选读 |
| 6 | [Doris Colocate Join 原理](#q6-doris-colocate-join-原理) | ⭐⭐⭐ | 选读 |

---

## Q1. ClickHouse MergeTree merge 策略

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

## Q2. ClickHouse 分布式表 vs 本地表写入策略

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

## Q3. OLAP 引擎选型：Cloud DW 对比

> 📌 **频率**: FAANG 面试系统设计必考 · ★★★  
> `Snowflake` vs `BigQuery` vs `Redshift` vs `ClickHouse`

### 🎯 核心对比

| 维度 | **Snowflake** | **BigQuery** | **Redshift** | **ClickHouse** |
|------|--------------|-------------|--------------|----------------|
| **架构** | 存算分离，Virtual Warehouse | Serverless | 存算分离（RA3）| 存算一体（可分离）|
| **计费** | 按 VW 运行时长 (Credit) | 按扫描量 ($5/TB) 或 Slot | 按节点时长 | 自建 / ClickHouse Cloud |
| **并发** | Multi-cluster 自动扩展 | Serverless，无上限 | 并发槽有限 | 单集群高并发 |
| **延迟** | 秒级（热查询快）| 秒级 | 秒到分钟 | 毫秒级（亚秒响应）|
| **生态** | 丰富（Data Marketplace）| GCP 原生，ML 集成 | AWS 原生 | 开源，自建 |
| **适合场景** | 企业级 DW，多工作负载隔离 | 大规模 ad-hoc，ML 场景 | AWS 重度用户 | 实时分析，高并发点查 |
| **学习曲线** | 低（标准 SQL）| 低 | 中 | 中（专有函数）|

### 📝 选型决策树

```
需求: 实时分析 + 高并发 + 亚秒响应
  → ClickHouse（日志分析/用户行为/监控大盘）

需求: 企业级 DW + 多团队隔离 + 标准SQL
  → Snowflake

需求: GCP 生态 + BigML + 无运维
  → BigQuery

需求: AWS 重度用户 + 现有 Redshift 投资
  → Redshift Serverless

需求: 成本敏感 + 开源 + 团队有运维能力
  → ClickHouse（自建）
```

### 📊 典型 FAANG 使用场景

```
Meta:     Presto / Spark on internal lakehouse
Google:   BigQuery (内部版 Dremel)
Amazon:   Redshift + Athena + EMR
Airbnb:   Spark + Presto on S3 (Lakehouse)
Uber:     Pinot (实时 OLAP) + Hive (批量)
Netflix:  Druid + Iceberg + Spark
LinkedIn: Pinot + Presto + Hive
```

### 📝 ClickHouse 特别说明（国内外均常见）

```
ClickHouse 优势:
  ✅ 列存 + 向量化执行引擎 → 聚合查询极快
  ✅ 亚秒级响应（10亿行 COUNT 约 0.1s）
  ✅ 高压缩率（LZ4/ZSTD，比行存省 10x 空间）
  ✅ 开源免费，ClickHouse Cloud 有托管版

ClickHouse 劣势:
  ❌ 不支持真正的 UPDATE/DELETE（需 ReplacingMergeTree）
  ❌ 跨表 JOIN 复杂场景不如 MPP 数仓
  ❌ 事务支持弱（非 OLTP 场景）
```

### 🧠 记忆锚点

```
Snowflake = 企业标准 + 存算分离 + 贵但好用
BigQuery = Serverless + GCP + 按扫描量计费
Redshift = AWS 原生 + RA3 存算分离
ClickHouse = 实时 OLAP + 亚秒响应 + 开源

面试说: "根据延迟要求、云平台归属、团队规模和预算综合决策"
```

---

## Q4. StarRocks 存算分离架构

> 📌 **频率**: 2025 · ★★☆ （选读）  
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

### 🧠 记忆锚点

```
存算分离 = FE + Stateless CN + Shared Storage(S3) + Local Cache
好处: Independent Scaling + Low Cost + Multi-tenant Isolation
```

---

## Q5. Doris Rollup vs 物化视图

> 📌 **频率**: 2025 · ★☆☆ （选读）

### 🎯 核心要点

| 对比维度 | Rollup | Materialized View (Async MV) |
|----------|--------|------------------------------|
| 数据来源 | 单表 (Single Table) | 多表 JOIN / 复杂表达式 |
| 列要求 | 必须包含基础表部分列前缀作为 Key | 无限制 |
| 刷新方式 | **同步** (Synchronous) — 写入时自动更新 | **异步** (Asynchronous) — 定时/手动刷新 |
| 查询改写 | 优化器自动命中 (Auto Hit) | 优化器自动改写 (Query Rewrite) |
| 适用场景 | 简单聚合上卷 (Pre-aggregation) | 跨表预计算、复杂 ETL |

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

## Q6. Doris Colocate Join 原理

> 📌 **频率**: 2025 · ★★☆ （选读）  
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

### 🧠 记忆锚点

```
Colocate = 同 Group + 同 Bucket 数 + 同分桶列 → Zero-Shuffle Local Join
```
