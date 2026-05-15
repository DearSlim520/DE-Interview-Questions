# 📚 DE Interview Questions

> Data Engineering 面试题库 — FAANG / 美国大厂 Mid-Senior DE 导向 · 按技术栈分类 · 每日 7 题 · 艾宾浩斯复习法

## 📁 分类目录

| Category | 描述 | 题数 | 权重 |
|----------|------|------|------|
| [⚡ Flink](./Flink/main.md) | 实时流处理引擎 | 9 | 核心 |
| [🔥 Spark](./Spark/main.md) | 批/流处理引擎 | 7 | 核心 |
| [📨 Kafka](./Kafka/main.md) | 消息队列 | 4 | 核心 |
| [🚀 OLAP-Engines](./OLAP-Engines/main.md) | ClickHouse / Snowflake / BigQuery（Doris/StarRocks 选读）| 6 | 核心 |
| [🏔️ Data-Lakehouse](./Data-Lakehouse/main.md) | Iceberg / Paimon 湖仓一体 | 2 | 核心 |
| [🏗️ Architecture](./Architecture/main.md) | 架构设计 / Medallion / CDC / Cloud Cost | 14 | 核心 |
| [📐 Data-Modeling](./Data-Modeling/main.md) | 维度建模 / 数仓分层 | 8 | 核心 |
| [🧮 SQL](./SQL/main.md) | SQL 面试经典题 | 6 | 核心 |
| [🛡️ Data-Governance](./Data-Governance/main.md) | 数据治理 / 质量 / 血缘 / 安全 | 5 | 核心 |
| [🔧 dbt](./dbt/main.md) | Analytics Engineering · ELT Transform | 7 | 核心 |
| [🌀 Airflow](./Airflow/main.md) | DAG 调度 / Executor / XCom / Backfill | 7 | 核心 |
| [📋 Data-Contract-Observability](./Data-Contract-Observability/main.md) | Data Contract / Observability / SLO | 7 | 核心 |
| [💾 NoSQL-and-Cache](./NoSQL-and-Cache/main.md) | Redis（HBase 选读）| 1 | 核心 |
| [🎯 System-Design](./System-Design/main.md) | 系统设计 / 场景题 | 4 | 核心 |
| [🐝 Hive](./Hive/main.md) | 数据仓库引擎 & 存储格式 | 7 | 选读 |
| [🐘 Hadoop](./Hadoop/main.md) | HDFS / YARN / MapReduce | 4 | 选读 |

**Active Questions（主线）: 84 · Background Reading（选读）: 11 · Total: 95**

---

## 📅 每日进度 Tracker

> ✅ 点击 checkbox 标记已复习 · 点击题目跳转详细解答

### Day 1 — 现代数据栈概览 · Lakehouse · Medallion

- [ ] [Iceberg metadata 三层结构（snapshot / manifest list / manifest file）](./Data-Lakehouse/main.md#q1-iceberg-metadata-三层结构) `Data-Lakehouse`
- [ ] [Medallion Architecture：Bronze / Silver / Gold 各层职责与 Lakehouse 关系](./Architecture/main.md#q7-medallion-architecture) `Architecture`
- [ ] [Kappa vs Lambda（流批一体落地）](./Architecture/main.md#q1-kappa-vs-lambda) `Architecture`
- [ ] [Lakehouse 架构演进（三代演进 + 核心特性）](./Architecture/main.md#q6-lakehouse-架构演进) `Architecture`
- [ ] [数据血缘系统设计（Calcite 解析 + Neo4j 图存储）](./Data-Governance/main.md#q2-数据血缘系统设计) `Data-Governance`
- [ ] [ClickHouse MergeTree merge 策略 + Too many parts](./OLAP-Engines/main.md#q1-clickhouse-mergetree-merge-策略) `OLAP-Engines`
- [ ] [DQC 六大维度（完准一唯及有）](./Data-Governance/main.md#q1-dqc-六大维度) `Data-Governance`

### Day 2 — 实时架构 · OLAP · 存储选型

- [ ] [Paimon 与 Iceberg 区别（流式湖仓）](./Data-Lakehouse/main.md) `Data-Lakehouse`
- [ ] [ClickHouse 分布式表 vs 本地表写入策略](./OLAP-Engines/main.md#q2-clickhouse-分布式表-vs-本地表写入策略) `OLAP-Engines`
- [ ] [OLAP 引擎选型：Snowflake vs BigQuery vs Redshift vs ClickHouse](./OLAP-Engines/main.md#q3-olap-引擎选型cloud-dw-对比) `OLAP-Engines`
- [ ] [实时数仓分层（ODS-DWD-DWS-ADS 在实时下的取舍）](./Architecture/main.md#q2-实时数仓分层) `Architecture`
- [ ] [实时大屏架构与延迟优化](./Architecture/main.md#q3-实时大屏架构与延迟优化) `Architecture`
- [ ] [列存 vs 行存使用场景](./Architecture/main.md#q4-列存-vs-行存使用场景) `Architecture`
- [ ] [数据湖 vs 数据仓库（含 Data Swamp 问题）](./Architecture/main.md#q5-数据湖-vs-数据仓库) `Architecture`

### Day 3 — Flink 专题

- [ ] [Flink Checkpoint 与 Savepoint 区别](./Flink/main.md#q3-flink-checkpoint-与-savepoint-区别) `Flink`
- [ ] [Flink 反压定位与处理](./Flink/main.md#q4-flink-反压定位与处理) `Flink`
- [ ] [Flink State Backend 选型（RocksDB vs HashMap）](./Flink/main.md#q5-flink-state-backend-选型) `Flink`
- [ ] [Flink Watermark 乱序处理](./Flink/main.md#q6-flink-watermark-乱序处理) `Flink`
- [ ] [Exactly-Once 端到端实现（2PC sink）](./Flink/main.md#q7-exactly-once-端到端实现) `Flink`
- [ ] [Flink SQL vs DataStream 取舍](./Flink/main.md#q8-flink-sql-vs-datastream-取舍) `Flink`
- [ ] [Flink 双流 join 的状态膨胀治理](./Flink/main.md#q9-flink-双流-join-的状态膨胀治理) `Flink`

### Day 4 — Spark 专题

- [ ] [Spark AQE 三大特性](./Spark/main.md) `Spark`
- [ ] [Spark 数据倾斜定位与方案（加盐/MapJoin/局部聚合）](./Spark/main.md) `Spark`
- [ ] [Spark Shuffle 演进（Hash → Sort → Tungsten）](./Spark/main.md) `Spark`
- [ ] [RDD vs DataFrame vs DataSet](./Spark/main.md) `Spark`
- [ ] [Spark 内存模型（统一内存管理）](./Spark/main.md) `Spark`
- [ ] [Spark Stage 划分与 DAG](./Spark/main.md) `Spark`
- [ ] [Spark Catalyst 优化器原理](./Spark/main.md) `Spark`

### Day 5 — dbt 专题

- [ ] [什么是 dbt，与传统 ETL 的区别](./dbt/main.md#q1-什么是-dbt) `dbt`
- [ ] [dbt Models 分类：staging / intermediate / mart](./dbt/main.md#q2-dbt-models-分类) `dbt`
- [ ] [dbt Tests：generic vs singular](./dbt/main.md#q3-dbt-tests) `dbt`
- [ ] [dbt Sources & Freshness 配置](./dbt/main.md#q4-dbt-sources--freshness) `dbt`
- [ ] [dbt Lineage & DAG 可视化](./dbt/main.md#q5-dbt-lineage--dag) `dbt`
- [ ] [dbt vs Stored Procedures](./dbt/main.md#q6-dbt-vs-stored-procedures) `dbt`
- [ ] [dbt Incremental Models 原理](./dbt/main.md#q7-dbt-incremental-models) `dbt`

### Day 6 — Airflow 专题

- [ ] [Airflow 核心概念：DAG / Operator / Task / TaskInstance](./Airflow/main.md#q1-airflow-核心概念) `Airflow`
- [ ] [Scheduler 工作原理与 Executor 类型对比](./Airflow/main.md#q2-scheduler-与-executor) `Airflow`
- [ ] [XCom 机制与使用场景](./Airflow/main.md#q3-xcom-机制) `Airflow`
- [ ] [Sensor vs Operator 区别（poke vs reschedule 模式）](./Airflow/main.md#q4-sensor-vs-operator) `Airflow`
- [ ] [Airflow Backfill 原理与实战](./Airflow/main.md#q5-airflow-backfill) `Airflow`
- [ ] [任务重试、SLA 监控与告警](./Airflow/main.md#q6-重试与-sla) `Airflow`
- [ ] [Airflow vs Prefect vs Luigi 对比](./Airflow/main.md#q7-airflow-vs-prefect-vs-luigi) `Airflow`

### Day 7 — 数据建模专题

- [ ] [维度建模 vs 范式建模 vs Data Vault](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [拉链表设计与 SCD Type 2](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [一致性维度（Conformed Dimension）](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [缓慢变化维 6 种类型](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [事实表三种类型（事务/周期快照/累积快照）](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [雪花模型 vs 星型模型](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [数仓分层（ODS/DWD/DWS/DIM/ADS）职责](./Data-Modeling/main.md) `Data-Modeling`

### Day 8 — SQL 专题

- [ ] [连续登录 N 天 SQL（窗口函数 + 日期差分组）](./SQL/main.md) `SQL`
- [ ] [TopN 分组 SQL（row_number vs rank vs dense_rank）](./SQL/main.md) `SQL`
- [ ] [留存率 SQL（次日/7日/30日）](./SQL/main.md) `SQL`
- [ ] [漏斗转化 SQL](./SQL/main.md) `SQL`
- [ ] [同环比 SQL（lag/lead）](./SQL/main.md) `SQL`
- [ ] [行转列 / 列转行](./SQL/main.md) `SQL`
- [ ] [用户画像标签宽表设计](./Data-Modeling/main.md) `Data-Modeling`

### Day 9 — Kafka · Cloud Cost · Redis

- [ ] [Kafka 高吞吐原因（顺序写/零拷贝/批/分区）](./Kafka/main.md) `Kafka`
- [ ] [Kafka exactly-once 与幂等 Producer](./Kafka/main.md) `Kafka`
- [ ] [Kafka 分区分配策略（Range/RoundRobin/Sticky）](./Kafka/main.md) `Kafka`
- [ ] [Kafka ISR 机制](./Kafka/main.md) `Kafka`
- [ ] [Cloud Cost：Partition Pruning & Clustering](./Architecture/main.md#q12-cloud-cost-partition-pruning--clustering) `Architecture`
- [ ] [Cloud Cost：Compute vs Storage 分离架构](./Architecture/main.md#q13-cloud-cost-compute-vs-storage-分离) `Architecture`
- [ ] [Redis 缓存击穿/穿透/雪崩](./NoSQL-and-Cache/main.md) `NoSQL-and-Cache`

### Day 10 — Data Contract + Observability

- [ ] [什么是 Data Contract](./Data-Contract-Observability/main.md#q1-什么是-data-contract) `Data-Contract-Observability`
- [ ] [为什么需要 Data Contract](./Data-Contract-Observability/main.md#q2-为什么需要-data-contract) `Data-Contract-Observability`
- [ ] [Data Contract 如何实现（YAML + CI/CD）](./Data-Contract-Observability/main.md#q3-data-contract-如何实现) `Data-Contract-Observability`
- [ ] [Data Observability 四大维度（freshness/volume/schema/distribution）](./Data-Contract-Observability/main.md#q4-data-observability-四大维度) `Data-Contract-Observability`
- [ ] [Data Observability 工具对比（Monte Carlo / GX / dbt Tests / Soda）](./Data-Contract-Observability/main.md#q5-data-observability-工具对比) `Data-Contract-Observability`
- [ ] [SLA vs SLO vs SLI in Data Engineering](./Data-Contract-Observability/main.md#q6-sla-vs-slo-vs-sli) `Data-Contract-Observability`
- [ ] [Data Quality vs Data Observability 区别](./Data-Contract-Observability/main.md#q7-data-quality-vs-data-observability) `Data-Contract-Observability`

### Day 11 — CDC 原理深入 · 数据治理

- [ ] [什么是 CDC（Change Data Capture）](./Architecture/main.md#q8-什么是-cdc) `Architecture`
- [ ] [Debezium 架构与工作流程](./Architecture/main.md#q9-debezium-架构与工作流程) `Architecture`
- [ ] [Binlog-based vs Query-based CDC 对比](./Architecture/main.md#q10-binlog-based-vs-query-based-cdc) `Architecture`
- [ ] [CDC Snapshot vs Incremental 模式切换](./Architecture/main.md#q11-cdc-snapshot-vs-incremental) `Architecture`
- [ ] [Flink CDC 2.x 无锁全量+增量切换（DBLog/chunk 高低水位）](./Flink/main.md#q1-flink-cdc-2x-无锁全量增量切换) `Flink`
- [ ] [数据资产管理体系](./Data-Governance/main.md#q3-数据资产管理体系) `Data-Governance`
- [ ] [元数据管理系统设计（DataHub / Atlas / OpenMetadata）](./Data-Governance/main.md#q4-元数据管理系统设计) `Data-Governance`

### Day 12 — 系统设计 · 数据治理 · Cloud Cost 收尾

- [ ] [设计直播实时数仓](./System-Design/main.md) `System-Design`
- [ ] [设计搜索推荐数据链路](./System-Design/main.md) `System-Design`
- [ ] [一次大故障复盘（线上数据延迟/口径错误怎么定位）](./System-Design/main.md) `System-Design`
- [ ] [项目深挖：你最复杂的数仓项目](./System-Design/main.md) `System-Design`
- [ ] [数据安全分级与脱敏](./Data-Governance/main.md#q5-数据安全分级与脱敏) `Data-Governance`
- [ ] [Cloud Cost：查询成本优化（Result Cache / MV / Z-Order）](./Architecture/main.md#q14-cloud-cost-查询成本优化) `Architecture`
- [ ] [Flink 双流 join 的几种方式（interval/window/regular）](./Flink/main.md#q2-flink-双流-join-的几种方式) `Flink`

---

## 📖 选读专题（Background Reading）

> 国内大厂常考，FAANG 了解即可，不单独成 Day

### 🐝 Hive 专题（选读）

- [ ] [Hive 小文件治理](./Hive/main.md) `Hive`
- [ ] [Hive 数据倾斜（group by/join/count distinct）](./Hive/main.md) `Hive`
- [ ] [Hive 分区 vs 分桶](./Hive/main.md) `Hive`
- [ ] [ORC vs Parquet](./Hive/main.md) `Hive`
- [ ] [Hive on Tez vs MR vs Spark](./Hive/main.md) `Hive`

### 🐘 Hadoop / HBase 专题（选读）

- [ ] [HDFS 读写流程](./Hadoop/main.md) `Hadoop`
- [ ] [HDFS NameNode HA 方案](./Hadoop/main.md) `Hadoop`
- [ ] [YARN 资源调度（Capacity vs Fair）](./Hadoop/main.md) `Hadoop`
- [ ] [HBase RowKey 设计](./NoSQL-and-Cache/main.md) `NoSQL-and-Cache`
- [ ] [StarRocks 存算分离架构](./OLAP-Engines/main.md#q4-starrocks-存算分离架构) `OLAP-Engines`
- [ ] [Doris Rollup vs 物化视图](./OLAP-Engines/main.md#q5-doris-rollup-vs-物化视图) `OLAP-Engines`

---

## 🧠 艾宾浩斯复习计划（Day 13 起启动）

> 每个 Day 的内容在 **1d / 2d / 4d / 7d / 15d / 30d** 后各复习一次（共 6 轮）
> 每次只看题目 → 自己回忆要点 → 回忆不出再看答案

| 复习日 | 复习内容 | 间隔 |
|--------|----------|------|
| Day 13 | Day 12 的 7 题 | 1d |
| Day 14 | Day 11 + Day 12 | 1d / 2d |
| Day 15 | Day 10 + Day 11 | 1d / 2d |
| Day 16 | Day 9 + Day 10 + Day 12 | 1d / 2d / 4d |
| Day 17 | Day 8 + Day 9 + Day 11 | 1d / 2d / 4d |
| Day 18 | Day 7 + Day 8 + Day 10 | 1d / 2d / 4d |
| Day 19 | Day 6 + Day 7 + Day 9 + Day 12 | 1d / 2d / 4d / 7d |
| ... | 滚动 1/2/4/7/15/30 天间隔 | |

---

## 📖 使用说明

1. **勾选复习**：直接在 GitHub 页面点击 checkbox 标记已完成（会自动 commit）
2. **跳转详解**：点击题目链接直达对应分类文件的详细解答
3. **选读内容**：Hive / Hadoop / HBase / Doris / StarRocks 放在选读区，按需学习
4. **FAANG 重点**：dbt、Airflow、Data Contract、Observability、Medallion、CDC、Cloud Cost 是新增核心模块
