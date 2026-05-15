# 🏗️ Architecture

> 架构设计 (Architecture Design) / 流批一体 (Stream-Batch Unified) / 存储选型 (Storage Selection) / CDC / Cloud Cost

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Kappa vs Lambda](#q1-kappa-vs-lambda) | ⭐⭐ |
| 2 | [实时数仓分层](#q2-实时数仓分层) | ⭐⭐⭐ |
| 3 | [实时大屏架构与延迟优化](#q3-实时大屏架构与延迟优化) | ⭐⭐ |
| 4 | [列存 vs 行存使用场景](#q4-列存-vs-行存使用场景) | ⭐⭐ |
| 5 | [数据湖 vs 数据仓库](#q5-数据湖-vs-数据仓库) | ⭐⭐ |
| 6 | [Lakehouse 架构演进](#q6-lakehouse-架构演进) | ⭐⭐⭐ |
| 7 | [Medallion Architecture：Bronze / Silver / Gold](#q7-medallion-architecture) | ⭐⭐ |
| 8 | [什么是 CDC（Change Data Capture）](#q8-什么是-cdc) | ⭐⭐ |
| 9 | [Debezium 架构与工作流程](#q9-debezium-架构与工作流程) | ⭐⭐⭐ |
| 10 | [Binlog-based vs Query-based CDC](#q10-binlog-based-vs-query-based-cdc) | ⭐⭐⭐ |
| 11 | [CDC Snapshot vs Incremental 模式](#q11-cdc-snapshot-vs-incremental) | ⭐⭐⭐ |
| 12 | [Cloud Cost：Partition Pruning & Clustering](#q12-cloud-cost-partition-pruning--clustering) | ⭐⭐ |
| 13 | [Cloud Cost：Compute vs Storage 分离](#q13-cloud-cost-compute-vs-storage-分离) | ⭐⭐ |
| 14 | [Cloud Cost：查询成本优化策略](#q14-cloud-cost-查询成本优化) | ⭐⭐⭐ |

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

---

## Q7. Medallion Architecture

> 📌 **频率**: FAANG 2024-2025 高频 · ★★☆  
> `Bronze / Silver / Gold` · Lakehouse 分层设计模式

### 🎯 三层架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                   Medallion Architecture                     │
│                                                             │
│  Source Systems                                             │
│  (MySQL / Kafka / S3 / APIs)                                │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────────────────────────────────┐               │
│  │  🥉 Bronze Layer (Raw / Landing Zone)   │               │
│  │  - 原始数据，不做任何转换                 │               │
│  │  - Append-only，保留完整历史             │               │
│  │  - Schema: as-is from source            │               │
│  └──────────────────┬──────────────────────┘               │
│                     │                                       │
│                     ▼                                       │
│  ┌─────────────────────────────────────────┐               │
│  │  🥈 Silver Layer (Cleansed / Enriched)  │               │
│  │  - 清洗、去重、标准化、类型转换           │               │
│  │  - 跨源 JOIN，字段统一口径               │               │
│  │  - 面向工程师和数据科学家               │               │
│  └──────────────────┬──────────────────────┘               │
│                     │                                       │
│                     ▼                                       │
│  ┌─────────────────────────────────────────┐               │
│  │  🥇 Gold Layer (Business / Aggregated)  │               │
│  │  - 面向业务的宽表 / 聚合表               │               │
│  │  - Fact + Dimension 建模                │               │
│  │  - 直接对接 BI 工具 / API               │               │
│  └─────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

### 📊 各层职责详解

| 层 | 别名 | 数据状态 | 操作 | 消费者 |
|----|------|----------|------|--------|
| **Bronze** | Raw / Landing / Ingestion | 原始态（as-is）| 只 Append，不修改 | 数据工程师 |
| **Silver** | Cleansed / Enriched / Conformed | 标准化、清洗后 | 清洗、JOIN、去重 | 工程师 + 数据科学家 |
| **Gold** | Business / Aggregated / Serving | 聚合、面向业务 | 建模（Fact/Dim）、预聚合 | BI / 业务分析师 / API |

### 📝 与 Lakehouse 的关系

```
Medallion Architecture + Lakehouse 技术栈 = 现代数据平台标配

┌── Lakehouse 技术 ───────────────────────────────┐
│  Storage: S3 / ADLS / GCS                      │
│  Format: Delta Lake / Iceberg / Hudi            │
│                                                 │
│  Bronze ─────────────── Delta Table (raw)       │
│  Silver ─────────────── Delta Table (cleaned)   │
│  Gold   ─────────────── Delta Table (serving)   │
│                                                 │
│  Compute: Spark / Databricks / dbt              │
└─────────────────────────────────────────────────┘

Bronze 对应: ODS 层
Silver 对应: DWD / DWS 层
Gold 对应:   ADS / DM 层
```

### 💡 类比记忆

> Medallion = 矿石冶炼流程 ⛏️
> - **Bronze** = 原矿石（未加工，保留原貌）
> - **Silver** = 精炼矿（提纯、去杂质、标准化）
> - **Gold** = 成品金条（面向客户，可直接使用）

### 🧠 记忆锚点

```
Bronze = Raw + Append-only + 不转换
Silver = Cleansed + Enriched + 标准化
Gold = Business-facing + Aggregated + BI-ready
对应国内: ODS → DWD/DWS → ADS
```

---

## Q8. 什么是 CDC

> 📌 **频率**: FAANG 2024-2025 · ★★☆  
> `Change Data Capture` · 实时数据同步 · 增量摄取

### 🎯 一句话定义

CDC (Change Data Capture) = **捕获数据库中行级别变更（INSERT / UPDATE / DELETE）并将变更流式传输到下游**的技术。

```
源数据库 (MySQL / PostgreSQL / Oracle)
  │  用户在 App 上下了一单 → INSERT INTO orders ...
  │  用户修改地址 → UPDATE orders SET address = ...
  │  订单取消 → DELETE FROM orders WHERE ...
  │
  ▼ CDC 捕获所有变更（实时、增量）
  │
  ├──► Kafka Topic (orders-cdc)
  │      {"op":"c","before":null,"after":{"id":1001,"amount":99.9,...}}
  │      {"op":"u","before":{...},"after":{...}}
  │      {"op":"d","before":{...},"after":null}
  │
  └──► 下游消费
       ├── Flink → 实时数仓
       ├── Elasticsearch → 搜索索引同步
       └── Data Lakehouse → 增量摄取
```

### 📊 CDC 的核心价值

| 传统方案 | CDC 方案 | 优势 |
|----------|----------|------|
| 全量抽取（每次 SELECT *）| 只传变更数据 | 减少 90%+ 数据传输量 |
| 定时批量（每小时/每天）| 实时流式 | 延迟从小时级 → 秒级 |
| 无法捕获 DELETE | 捕获 INSERT/UPDATE/DELETE | 数据完整性 |
| 对源库压力大 | 读 binlog，对主库影响极小 | 低侵入 |

### 💡 类比记忆

> CDC = 数据库的"聊天记录同步" 💬
> - 全量同步 = 每次都把所有聊天记录发一遍
> - CDC = 只发"新消息"（增量），接收方按顺序应用

### 🧠 记忆锚点

```
CDC = 捕获 INSERT/UPDATE/DELETE 变更 → 实时流到下游
核心价值: 增量(非全量) + 实时(非批量) + 低侵入(读日志)
```

---

## Q9. Debezium 架构与工作流程

> 📌 **频率**: 2025 · ★★★  
> `Debezium` · Kafka Connect · CDC 平台

### 🎯 Debezium 是什么

Debezium = **开源 CDC 平台**，基于 Kafka Connect，支持 MySQL/PostgreSQL/MongoDB/Oracle 等主流数据库。

### 📊 架构图

```
┌────────────────────────────────────────────────────────────┐
│                    Debezium 架构                            │
│                                                            │
│  ┌──────────────┐    ┌─────────────────────────────────┐  │
│  │ Source DB    │    │  Kafka Connect Cluster          │  │
│  │              │    │  ┌──────────────────────────┐   │  │
│  │  MySQL       │───►│  │  Debezium Source Connector│  │  │
│  │  (binlog)    │    │  │  (MySQL Connector)        │  │  │
│  │              │    │  └──────────────┬───────────┘   │  │
│  └──────────────┘    └─────────────────┼───────────────┘  │
│                                        │ 变更事件           │
│                                        ▼                   │
│                      ┌─────────────────────────────────┐  │
│                      │  Apache Kafka                   │  │
│                      │  Topic: db.schema.table         │  │
│                      │  {"op":"c","after":{...}}       │  │
│                      └─────────────────────────────────┘  │
│                                        │                   │
│              ┌─────────────────────────┼──────────────┐   │
│              ▼                         ▼              ▼   │
│           Flink               Spark Streaming      ES/DW  │
└────────────────────────────────────────────────────────────┘
```

### 📝 Debezium MySQL Connector 配置示例

```json
{
  "name": "mysql-orders-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-prod.internal",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "${file:/secrets.properties:db.password}",
    "database.server.id": "1",
    "database.include.list": "ecommerce",
    "table.include.list": "ecommerce.orders,ecommerce.customers",
    "database.history.kafka.topic": "schema-changes.ecommerce",
    "transforms": "route",
    "transforms.route.type": "org.apache.kafka.connect.transforms.ReplaceField$Value"
  }
}
```

### 📝 Debezium 事件格式

```json
{
  "op": "u",              // c=create, u=update, d=delete, r=read(snapshot)
  "ts_ms": 1699000000000,
  "before": {             // UPDATE/DELETE 时有 before
    "id": 1001,
    "status": "placed",
    "amount": 99.90
  },
  "after": {              // INSERT/UPDATE 时有 after
    "id": 1001,
    "status": "shipped",
    "amount": 99.90
  },
  "source": {
    "db": "ecommerce",
    "table": "orders",
    "pos": 12345678       // binlog position
  }
}
```

### 🧠 记忆锚点

```
Debezium = Kafka Connect + CDC Source Connector
支持: MySQL(binlog) / PG(WAL) / MongoDB(oplog) / Oracle(LogMiner)
事件格式: op(c/u/d/r) + before + after + source
```

---

## Q10. Binlog-based vs Query-based CDC

> 📌 **频率**: 2025 · ★★★  
> 两种 CDC 实现方式对比

### 🎯 两种主流实现

| 维度 | **Binlog-based CDC** | **Query-based CDC** |
|------|---------------------|---------------------|
| **原理** | 读取数据库事务日志（binlog/WAL/redo log）| 定期查询 `WHERE updated_at > last_run` |
| **实时性** | 毫秒级（日志实时推送）| 分钟级（定时 polling）|
| **捕获 DELETE** | ✅ 完整捕获 | ❌ 无法捕获（行已删除）|
| **对源库压力** | 极低（读日志，不查表）| 较高（全表扫描 or 索引查询）|
| **数据完整性** | 完整（含 before/after）| 只有当前值，无 before |
| **要求** | 需开启 binlog / WAL | 需有 `updated_at` 字段 |
| **部署复杂度** | 较复杂（Debezium + Kafka）| 简单（SQL 查询）|
| **代表工具** | Debezium, Canal, Maxwell | Airbyte (poll mode), Fivetran (基础版) |

### 📝 Query-based CDC 示例（简单但有缺陷）

```python
# 每 5 分钟运行一次
def incremental_extract(last_updated_at):
    return spark.read.jdbc(
        url=jdbc_url,
        table="orders",
        predicates=[f"updated_at > '{last_updated_at}'"]
    )
# ❌ 问题: DELETE 行被丢失 | updated_at 未索引则慢 | 有 polling 延迟
```

### 📝 Binlog-based CDC 示例（完整）

```
MySQL binlog → Debezium Connector → Kafka Topic → Flink Consumer
  全程实时，捕获所有 DML (INSERT/UPDATE/DELETE)
  毫秒级延迟，对源库压力极小
```

### 🧠 记忆锚点

```
Binlog-based = 读日志 + 实时 + 捕获DELETE + 低压力 → 生产首选
Query-based  = SQL polling + 有延迟 + 丢DELETE + 简单易实现 → 简单场景
选型: 需要 DELETE + 实时 → Binlog | 只需增量 + 简单 → Query-based
```

---

## Q11. CDC Snapshot vs Incremental

> 📌 **频率**: 2025 · ★★★  
> `Initial Snapshot` · `Incremental Streaming` · 全量+增量切换

### 🎯 两个阶段

```
CDC 通常分为两个阶段:

Phase 1: Initial Snapshot (全量快照)
  读取数据库当前所有数据
  → 为 op=r (read) 事件
  → 解决"历史数据"问题

Phase 2: Incremental Streaming (增量流)
  从 snapshot 结束的 binlog 位置继续
  → 捕获 INSERT / UPDATE / DELETE
  → 持续实时同步
```

### 📊 快照策略对比

| 策略 | 实现方式 | 优缺点 |
|------|----------|--------|
| **Lock-based Snapshot** | `FLUSH TABLES WITH READ LOCK` 全表锁 | 数据一致，但锁表影响生产 ❌ |
| **Chunk-based Snapshot** | 按主键范围分块读取（Flink CDC 2.x / DBLog）| 无锁，但复杂度高 ✅ |
| **Export-based** | mysqldump / pg_dump 导出 | 简单，但会产生大文件 |

### 📝 Flink CDC 2.x 无锁切换（DBLog 算法）

```
核心思路: 每个 chunk 读取时记录 high/low watermark

For each chunk (e.g., id 1-1000):
  1. 记录 high watermark (当前 binlog pos: pos_h)
  2. SELECT * FROM table WHERE id BETWEEN 1 AND 1000
  3. 记录 low watermark (读取完成: pos_l)
  4. 用 [pos_l, pos_h] 之间的 binlog 修正 SELECT 结果
  → 合并得到一致快照，同时不锁表

所有 chunk 完成后 → 从 max(high_watermark) 切换到增量模式
```

### ⚠️ 常见问题

```
问题1: Snapshot 期间源库有大量写入 → chunk 修正量大，内存压力
解决: 合理设置 chunk 大小（默认 8096 行）

问题2: 大表 Snapshot 时间很长（亿级行）→ 占用大量资源
解决: 限速 + 低峰期执行 + 并行度控制

问题3: Snapshot 与 Incremental 切换点丢数据
解决: 正确记录 binlog pos，DBLog 算法保证 Exactly-Once
```

### 🧠 记忆锚点

```
Snapshot = 历史全量（op=r）→ Incremental = 实时增量（op=c/u/d）
切换点: snapshot 结束时记录 binlog pos → 增量从此处继续
无锁方案: Flink CDC 2.x / DBLog = chunk + watermark 修正
```

---

## Q12. Cloud Cost：Partition Pruning & Clustering

> 📌 **频率**: FAANG 2024-2025 · ★★☆  
> 云数仓查询优化 · Cost Optimization

### 🎯 为什么云数仓要关注 Cost

```
云数仓计费模式 (Snowflake / BigQuery / Redshift):
  BigQuery: 按扫描数据量计费（$5 / TB）
  Snowflake: 按计算时长计费（Credit/hour）
  Redshift: 按节点时长 + 数据扫描

→ 全表扫描 = 直接烧钱 💸
→ Partition Pruning + Clustering = 减少扫描量 = 省钱 + 快
```

### 📊 Partition Pruning

```sql
-- 表按 date 分区 (BigQuery / Iceberg / Delta)
CREATE TABLE orders
PARTITION BY DATE(order_date);   -- 每天一个分区

-- ❌ 不走分区裁剪（全表扫描）
SELECT * FROM orders WHERE customer_id = 123;

-- ✅ 分区裁剪（只扫描 1/365 的数据）
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';
-- 扫描量从 10TB → 800GB，费用降低 92%
```

### 📊 Clustering（聚簇）

```sql
-- Snowflake Clustering Key
ALTER TABLE orders CLUSTER BY (customer_id, order_date);

-- BigQuery Clustering
CREATE TABLE orders
PARTITION BY DATE(order_date)
CLUSTER BY customer_id, product_category;

-- 效果: 同一 cluster 的行物理上相邻存储
--       查询 WHERE customer_id = 123 只扫描相关 micro-partition
```

### 📊 分区 vs 聚簇

| 维度 | Partition | Clustering |
|------|-----------|------------|
| **粒度** | 粗粒度（日/月/年）| 细粒度（行级排序）|
| **适用列** | 低基数时间列（date, year）| 中高基数（user_id, region）|
| **效果** | 跳过整个分区文件 | 跳过 micro-partition |
| **维护** | 分区需预先规划 | 自动维护（Snowflake）|
| **组合** | 先分区再聚簇（双重优化）| ✅ |

### 🧠 记忆锚点

```
Partition = 按列把文件分组，查询跳过无关分区（粗粒度）
Clustering = 同值行物理相邻，查询跳过无关 micro-partition（细粒度）
组合使用: PARTITION BY date + CLUSTER BY user_id → 双重剪枝
```

---

## Q13. Cloud Cost：Compute vs Storage 分离

> 📌 **频率**: 2025 · ★★☆  
> 云原生架构核心特征 · 弹性伸缩

### 🎯 存算分离架构

```
┌─── 存算一体（传统）───────────────────────────────┐
│  Node 1: CPU + Memory + Local SSD               │
│  Node 2: CPU + Memory + Local SSD               │
│  Node 3: CPU + Memory + Local SSD               │
│                                                 │
│  ⚠️ 扩容必须同时加计算和存储                     │
│  ⚠️ 闲置时计算浪费，无法单独缩减                  │
└─────────────────────────────────────────────────┘

┌─── 存算分离（云原生）────────────────────────────┐
│  Compute Tier          Storage Tier             │
│  ┌──────────────┐      ┌──────────────────┐    │
│  │ Query Engine │◄────►│  S3 / ADLS / GCS │    │
│  │ (弹性伸缩)    │      │  (无限容量)       │    │
│  │              │      │  (低成本)         │    │
│  └──────────────┘      └──────────────────┘    │
│                                                 │
│  ✅ 计算按需启停（闲置 = $0 计算费）             │
│  ✅ 存储独立扩展，单价低（S3 = $0.023/GB/月）    │
│  ✅ 多计算集群共享同一份存储                     │
└─────────────────────────────────────────────────┘
```

### 📊 主流云数仓存算分离实现

| 产品 | Compute | Storage | 弹性特性 |
|------|---------|---------|----------|
| **Snowflake** | Virtual Warehouse (VW) | S3 / Azure Blob / GCS | VW 按秒计费，自动暂停 |
| **BigQuery** | Serverless Slot | Colossus | 完全 Serverless，按查询计费 |
| **Databricks** | Spark Cluster | Delta Lake on S3 | Cluster 自动启停 |
| **Redshift Serverless** | AQUA + RPU | S3 (RA3 + Spectrum) | 按 RPU 计费 |
| **Athena** | Presto Serverless | S3 | 完全按扫描量计费 |

### 💡 Cost 优化策略

```
Snowflake:
  - 设置 AUTO_SUSPEND = 60 (秒)，空闲自动暂停
  - 用 Multi-cluster Warehouse 处理并发高峰
  - Result Cache：相同查询 24h 内免费

BigQuery:
  - 用 Reservation (flat-rate) 替代 On-demand（高用量时省钱）
  - Materialized Views 缓存结果
  - BI Engine 加速频繁查询
```

### 🧠 记忆锚点

```
存算分离 = Compute(弹性) + Storage(便宜, S3)
优势: 独立扩缩 + 闲置省钱 + 多集群共享数据
Snowflake AUTO_SUSPEND + BigQuery Serverless = 云成本最优
```

---

## Q14. Cloud Cost：查询成本优化

> 📌 **频率**: 2025 · ★★★  
> Result Cache · Materialized Views · Z-Order · 查询优化全链路

### 🎯 成本优化全链路

```
查询进来
  │
  ▼
① Result Cache（结果缓存）
  → 完全相同的查询 → 直接返回缓存结果，零计算费
  │ Cache Miss
  ▼
② Partition Pruning + Clustering（见 Q12）
  → 减少扫描数据量
  │
  ▼
③ Materialized Views / Pre-aggregation
  → 预计算聚合，查询命中 MV 而非原表
  │
  ▼
④ Compute Tier 优化（Auto-suspend / 规格选型）
  → 匹配查询负载，避免过度配置
```

### 📊 具体优化手段

| 手段 | 原理 | 效果 | 适用场景 |
|------|------|------|----------|
| **Result Cache** | 缓存查询结果 24h | 重复查询零费用 | BI 报表（同一天多次查）|
| **Materialized View** | 预计算存储聚合结果 | 减少实时聚合开销 | 高频聚合查询 |
| **Z-Order Clustering** | 多列联合排序（Delta Lake）| 多维过滤时减少文件扫描 | 多条件过滤 |
| **Partition Pruning** | 跳过无关分区 | 减少 90%+ 扫描量 | 时间范围查询 |
| **Column Pruning** | 列存只读需要的列 | 减少 I/O | SELECT 少列 |
| **Approximate Agg** | `APPROX_COUNT_DISTINCT` | 10x 提速，5% 误差 | 允许近似的分析场景 |

### 📝 Snowflake 结果缓存

```sql
-- 第一次查询：需要计算
SELECT region, SUM(revenue) FROM sales GROUP BY region;
-- 执行时间: 45s，消耗 8 Credit

-- 第二次（24h 内完全相同）：直接命中 Result Cache
SELECT region, SUM(revenue) FROM sales GROUP BY region;
-- 执行时间: 0.3s，消耗 0 Credit ✅ 免费！

-- 注意: 数据更新后 Result Cache 自动失效
```

### 📝 BigQuery Materialized View

```sql
-- 创建物化视图（预聚合）
CREATE MATERIALIZED VIEW mv_daily_revenue AS
SELECT
  DATE(order_date) AS d,
  region,
  SUM(revenue) AS total_revenue
FROM orders
GROUP BY 1, 2;

-- 查询自动命中 MV（BigQuery 智能改写）
SELECT region, SUM(revenue)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY region;
-- BigQuery 自动识别并使用 MV 数据，节省扫描量
```

### 🧠 记忆锚点

```
Cost 优化四板斧:
  Result Cache → 相同查询免费
  Partition/Cluster → 减少扫描量（见 Q12）
  Materialized View → 预聚合，命中 MV 不算原表
  Auto-suspend → 计算闲置 = $0
最重要: 写好查询 + 分区设计（两者效果最显著）
```
