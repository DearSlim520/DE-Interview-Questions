# 🏗️ Architecture

> 架构设计 (Architecture Design) / 流批一体 (Stream-Batch Unified) / 存储选型 (Storage Selection)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Kappa vs Lambda](#q1-kappa-vs-lambda) | ⭐⭐ |
| 2 | [实时数仓分层](#q2-实时数仓分层) | ⭐⭐⭐ |
| 3 | [实时大屏架构与延迟优化](#q3-实时大屏架构与延迟优化) | ⭐⭐ |
| 4 | [列存 vs 行存使用场景](#q4-列存-vs-行存使用场景) | ⭐⭐ |
| 5 | [数据湖 vs 数据仓库](#q5-数据湖-vs-数据仓库) | ⭐⭐ |
| 6 | [Lakehouse 架构演进](#q6-lakehouse-架构演进) | ⭐⭐⭐ |

---

## Q1. Kappa vs Lambda

> 📌 **频率**: 2025 · ★★☆  
> `Lambda Architecture` vs `Kappa Architecture`

### 🎯 架构对比

```
┌─── Lambda Architecture ───────────────────────────────┐
│   Source ──► Batch Layer (Hive/Spark) ──┐             │
│         │                               ├──► Serving  │
│         └► Speed Layer (Flink/Storm) ──┘             │
│   ⚠️ 两套代码 (Dual Codebase) · 两套口径 · 结果需合并 │
└───────────────────────────────────────────────────────┘

┌─── Kappa Architecture ────────────────────────────────┐
│   Source ──► Stream Layer (Flink) ──► Serving         │
│                    └── Replay from Kafka/Paimon       │
│   ✅ Single Codebase · Single Source of Truth        │
└───────────────────────────────────────────────────────┘
```

### 📊 详细对比

| 维度 | Lambda | Kappa |
|------|--------|-------|
| 代码维护 (Codebase) | 两套（Batch + Stream） | 一套 (Unified) |
| 数据一致性 (Consistency) | 需对账合并 (Reconciliation) | 天然一致 |
| 离线兜底 (Batch Fallback) | 有（Batch Layer） | Replay from Kafka/Paimon |
| 复杂度 (Complexity) | 高 | 低 |
| 落地依赖 | 传统大数据栈 | Flink + Lakehouse Storage |

### 💡 类比记忆

> - **Lambda** = 同时养"白班团队"和"夜班团队"做同一份报表，结果还要对账 🤦
> - **Kappa** = 只一个团队 24h 流水线，需要历史数据时从备份磁带 (Kafka/Paimon) 从头跑一遍 🏃

### ✅ 2025 趋势：为什么大家抛弃 Lambda？

- **Flink** 成熟 → Stream Processing 能力足够
- **Iceberg / Paimon** → Streaming Write + Batch Read + Time Travel
- **Lakehouse** → 一份存储同时服务 Real-time & Offline

### 🧠 记忆锚点

```
Lambda = Dual Path + Reconciliation
Kappa = Single Path + Replayable Source
```

---

## Q2. 实时数仓分层

> 📌 **频率**: 2025 · ★★☆  
> `Real-time Data Warehouse Layering`

### 🎯 实时 vs 离线分层对比

| 层级 | 离线 (Offline) | 实时 (Real-time) | 取舍 |
|------|---------------|------------------|------|
| ODS | Hive 原始表 | Kafka Topic (原始消息) | 实时 ODS = Kafka |
| DWD | Hive 清洗表 | Flink 清洗 → Kafka/Paimon | 明细层仍需保留 |
| DWS | Hive 聚合表 | Flink 窗口聚合 → OLAP/Redis | **实时轻量化** |
| ADS | 报表/API | OLAP (Doris/StarRocks) / Redis | 直接查询服务 |

### 📊 实时数仓分层架构

```
┌─────────────────────────────────────────────────────────┐
│  Source (MySQL/Log) → Kafka (ODS)                        │
│       ↓                                                  │
│  Flink ETL (清洗/打宽/关联维表) → Kafka/Paimon (DWD)     │
│       ↓                                                  │
│  Flink Window Aggregation → Doris/Redis (DWS/ADS)       │
└─────────────────────────────────────────────────────────┘
```

### ⚠️ 实时下的取舍 (Trade-offs)

| 取舍项 | 说明 |
|--------|------|
| DWS 层精简 | 实时不适合做太重的聚合，尽量下推到 OLAP |
| 维表关联 (Dim Join) | 用 Async I/O + Cache，避免流关联维度流 |
| 数据回溯 (Backfill) | 需配合 Paimon/Iceberg 做历史回刷 |
| 口径一致 | 实时/离线共用 DWD 层口径（流批一体核心） |

### 💡 类比记忆

> 实时数仓 = 快餐流水线 🍔
> - ODS = 食材到货（原始消息）
> - DWD = 切配间（清洗切好）
> - DWS = 装盘区（聚合成品）→ 实时尽量薄
> - ADS = 出餐口（直接上桌）

### 🧠 记忆锚点

```
ODS=Kafka | DWD=Flink清洗 | DWS=轻量聚合(或直接跳过) | ADS=OLAP/Redis
核心：DWS层实时要轻，重活让OLAP干
```

---

## Q3. 实时大屏架构与延迟优化

> 📌 **频率**: 2025 · ★★☆  
> `Real-time Dashboard Architecture & Latency Optimization`

### 🎯 典型架构

```
数据源 → Kafka → Flink (实时聚合) → Redis/Doris → WebSocket → 前端大屏
                                         ↑
                                   Pre-aggregation
                                   (预聚合减少查询压力)
```

### 🛠️ 延迟优化手段

| 层级 | 优化手段 | 效果 |
|------|----------|------|
| **Source** | Binlog 实时采集 (CDC) | 秒级延迟 |
| **Flink** | Mini-batch / 增量聚合 (Incremental Agg) | 减少状态操作 |
| **存储** | Redis (热数据) + Doris (明细) | Redis = ms 级响应 |
| **推送** | WebSocket / SSE (Server-Sent Events) | 避免前端轮询 |
| **前端** | 局部刷新 (Partial Refresh) + 动画过渡 | 用户体感流畅 |

### 📊 延迟分段

```
End-to-End Latency = Source采集 + Flink处理 + 存储写入 + 前端渲染
                     (~1s)       (~1-3s)     (~100ms)    (~200ms)
目标: < 5s 端到端
```

### 💡 类比记忆

> 实时大屏 = 体育赛事直播 📺
> - Source = 现场摄像机（采集）
> - Flink = 导播切换台（处理聚合）
> - Redis = 转播信号（低延迟传输）
> - WebSocket = 电视信号推送（不需要观众刷新）

### 🧠 记忆锚点

```
Kafka→Flink预聚合→Redis(热)/Doris(明细)→WebSocket推送
优化: Pre-agg + Redis + WebSocket + Partial Refresh
```

---

## Q4. 列存 vs 行存使用场景

> 📌 **频率**: 2025 · ★★☆  
> `Columnar Storage vs Row Storage`

### 🎯 对比

| 维度 | 行存 (Row Store) | 列存 (Column Store) |
|------|-----------------|---------------------|
| 存储方式 | 一行数据连续存储 | 同一列数据连续存储 |
| 读取模式 | 读整行 (Full Row) | 读指定列 (Column Pruning) |
| 压缩率 | 低（列类型混合） | 高（同类型压缩效果好） |
| 适合查询 | 点查 (Point Query) / OLTP | 分析查询 (Analytical Query) / OLAP |
| 写入性能 | 快（追加一行） | 较慢（需写多个列文件） |
| 典型系统 | MySQL, PostgreSQL | Parquet, ORC, ClickHouse |

### 📊 选型决策

```
OLTP (事务处理): → 行存 (MySQL/PG)
  - 频繁 INSERT/UPDATE
  - SELECT * WHERE id = 123 (点查)

OLAP (分析查询): → 列存 (Parquet/ClickHouse)
  - SELECT SUM(amount) WHERE date > '2025-01'
  - 只需几列，扫描海量数据
```

### 💡 类比记忆

> - **行存** = 一份完整简历放一起 📄 → 看一个人的所有信息很快
> - **列存** = 所有人的"薪资"列放一起 💰 → 统计全公司平均薪资超快

### 🧠 记忆锚点

```
Row Store = OLTP + Point Query + Full Row
Column Store = OLAP + Aggregation + High Compression + Column Pruning
```

---

## Q5. 数据湖 vs 数据仓库

> 📌 **频率**: 2025 · ★★☆  
> `Data Lake vs Data Warehouse`

### 🎯 对比

| 维度 | Data Warehouse | Data Lake |
|------|---------------|-----------|
| Schema | Schema-on-Write（写入时定义） | Schema-on-Read（读取时定义） |
| 数据类型 | 结构化 (Structured Only) | 结构化 + 半结构化 + 非结构化 |
| 存储格式 | 专有格式 (Proprietary) | 开放格式 (Parquet/ORC/JSON) |
| 数据质量 | 高（ETL 清洗后入库） | 原始态 (Raw Data) |
| 用户 | 业务分析师 (BI Analyst) | 数据科学家 + 工程师 |
| 成本 | 高（专用存储） | 低（S3/HDFS 对象存储） |
| 典型系统 | Teradata, Redshift, Snowflake | S3 + Hadoop/Spark |

### ⚠️ Data Lake 的问题 (Data Swamp)

```
Data Lake → 没有治理 → Data Swamp (数据沼泽) 🐊
  - 没有 Schema → 读不懂
  - 没有版本 → 改坏了无法回滚
  - 没有 ACID → 读写冲突
  → 解决方案: Lakehouse!
```

### 🧠 记忆锚点

```
Warehouse = Schema-on-Write + Structured + High Quality + Expensive
Lake = Schema-on-Read + All Types + Raw + Cheap + Risk of Swamp
```

---

## Q6. Lakehouse 架构演进

> 📌 **频率**: 2025 · ★★☆  
> `Lakehouse Architecture Evolution`

### 🎯 三代演进

```
Gen 1: Data Warehouse (数据仓库)
  └── Structured only, Expensive, Siloed
  
Gen 2: Data Lake (数据湖)
  └── All data types, Cheap, But: No ACID, No Schema, Data Swamp
  
Gen 3: Lakehouse (湖仓一体) ✅
  └── Lake 的开放性 + Warehouse 的管理能力
      = Open Format + ACID + Schema + Time Travel + BI Ready
```

### 📊 Lakehouse 关键特性

| 特性 | 实现方式 |
|------|----------|
| **ACID Transactions** | Iceberg / Delta Lake / Hudi |
| **Schema Enforcement** | Table Format 强制校验 |
| **Time Travel** | Snapshot 版本管理 |
| **Open Format** | Parquet + Metadata Layer |
| **BI 直连** | 直接对接 Presto/Trino/Spark |
| **Streaming + Batch** | 统一存储，流批一体 |

### 📝 典型技术栈

```
Storage: S3 / HDFS (Open, Cheap)
Table Format: Iceberg / Delta Lake / Paimon
Compute: Spark / Flink / Trino
Catalog: Hive Metastore / Nessie / Unity Catalog
```

### 💡 类比记忆

> - **Warehouse** = 精装别墅 🏠 — 贵但整洁
> - **Lake** = 毛坯仓库 🏚️ — 便宜但脏乱
> - **Lakehouse** = 精装仓库 🏭✨ — 又大又整洁又便宜

### 🧠 记忆锚点

```
Lakehouse = Open Format(Parquet) + ACID(Iceberg) + Schema + Time Travel
= Data Lake 的成本 + Data Warehouse 的质量
```
