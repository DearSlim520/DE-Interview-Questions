# 📚 DE Interview Questions

> Data Engineering 面试题库 — 按技术栈分类 · 每日 7 题 · 艾宾浩斯复习法

## 📁 分类目录

| Category | 描述 | 题数 |
|----------|------|------|
| [⚡ Flink](./Flink/main.md) | 实时流处理引擎 | 9 |
| [🔥 Spark](./Spark/main.md) | 批/流处理引擎 | 7 |
| [🐝 Hive](./Hive/main.md) | 数据仓库引擎 & 存储格式 | 7 |
| [📨 Kafka](./Kafka/main.md) | 消息队列 | 4 |
| [🚀 OLAP-Engines](./OLAP-Engines/main.md) | ClickHouse / Doris / StarRocks | 5 |
| [🏔️ Data-Lakehouse](./Data-Lakehouse/main.md) | Iceberg / Paimon 湖仓一体 | 2 |
| [🐘 Hadoop](./Hadoop/main.md) | HDFS / YARN / MapReduce | 4 |
| [📐 Data-Modeling](./Data-Modeling/main.md) | 维度建模 / 数仓分层 | 8 |
| [🧮 SQL](./SQL/main.md) | SQL 面试经典题 | 6 |
| [🛡️ Data-Governance](./Data-Governance/main.md) | 数据治理 / 质量 / 血缘 / 安全 | 5 |
| [🏗️ Architecture](./Architecture/main.md) | 架构设计 / 流批一体 / 存储选型 | 6 |
| [💾 NoSQL-and-Cache](./NoSQL-and-Cache/main.md) | HBase / Redis | 3 |
| [🎯 System-Design](./System-Design/main.md) | 系统设计 / 场景题 | 4 |

**Total: 70 Questions**

---

## 📅 每日进度 Tracker

> ✅ 点击 checkbox 标记已复习 · 点击题目跳转详细解答

### Day 1 — 新趋势 · 低频优先

- [ ] [Iceberg metadata 三层结构（snapshot/manifest list/manifest file）](./Data-Lakehouse/main.md#q1-iceberg-metadata-三层结构) `Data-Lakehouse`
- [ ] [Flink CDC 2.x 无锁全量+增量切换（DBLog/chunk高低水位）](./Flink/main.md#q1-flink-cdc-2x-无锁全量增量切换) `Flink`
- [ ] [Doris Rollup vs 物化视图](./OLAP-Engines/main.md#q1-doris-rollup-vs-物化视图) `OLAP-Engines`
- [ ] [数据血缘系统设计（Calcite解析 + Neo4j图存储）](./Data-Governance/main.md#q2-数据血缘系统设计) `Data-Governance`
- [ ] [Kappa vs Lambda（流批一体落地）](./Architecture/main.md#q1-kappa-vs-lambda) `Architecture`
- [ ] [ClickHouse MergeTree merge 策略 + Too many parts](./OLAP-Engines/main.md#q2-clickhouse-mergetree-merge-策略) `OLAP-Engines`
- [ ] [DQC 六大维度（完准一唯及有）](./Data-Governance/main.md#q1-dqc-六大维度) `Data-Governance`

### Day 2 — 实时架构 · OLAP 深入

- [ ] [Paimon 与 Iceberg 区别（流式湖仓）](./Data-Lakehouse/main.md) `Data-Lakehouse`
- [ ] [StarRocks 存算分离架构](./OLAP-Engines/main.md) `OLAP-Engines`
- [ ] [实时数仓分层（ODS-DWD-DWS-ADS 在实时下的取舍）](./Architecture/main.md) `Architecture`
- [ ] [Flink 双流 join 的几种方式（interval/window/regular）](./Flink/main.md) `Flink`
- [ ] [ClickHouse 分布式表 vs 本地表写入策略](./OLAP-Engines/main.md) `OLAP-Engines`
- [ ] [Doris Colocate Join 原理](./OLAP-Engines/main.md) `OLAP-Engines`
- [ ] [实时大屏架构与延迟优化](./Architecture/main.md) `Architecture`

### Day 3 — Flink 专题

- [ ] [Flink Checkpoint 与 Savepoint 区别](./Flink/main.md) `Flink`
- [ ] [Flink 反压定位与处理](./Flink/main.md) `Flink`
- [ ] [Flink State Backend 选型（RocksDB vs HashMap）](./Flink/main.md) `Flink`
- [ ] [Flink Watermark 乱序处理](./Flink/main.md) `Flink`
- [ ] [Exactly-Once 端到端实现（2PC sink）](./Flink/main.md) `Flink`
- [ ] [Flink SQL vs DataStream 取舍](./Flink/main.md) `Flink`
- [ ] [Flink 双流 join 的状态膨胀治理](./Flink/main.md) `Flink`

### Day 4 — Spark 专题

- [ ] [Spark AQE 三大特性](./Spark/main.md) `Spark`
- [ ] [Spark 数据倾斜定位与方案（加盐/MapJoin/局部聚合）](./Spark/main.md) `Spark`
- [ ] [Spark Shuffle 演进（Hash → Sort → Tungsten）](./Spark/main.md) `Spark`
- [ ] [RDD vs DataFrame vs DataSet](./Spark/main.md) `Spark`
- [ ] [Spark 内存模型（统一内存管理）](./Spark/main.md) `Spark`
- [ ] [Spark Stage 划分与 DAG](./Spark/main.md) `Spark`
- [ ] [Spark Catalyst 优化器原理](./Spark/main.md) `Spark`

### Day 5 — Hive 专题

- [ ] [Hive 小文件治理（合并/CombineHiveInputFormat/归档）](./Hive/main.md) `Hive`
- [ ] [Hive 数据倾斜（group by/join/count distinct）](./Hive/main.md) `Hive`
- [ ] [Hive 分区 vs 分桶](./Hive/main.md) `Hive`
- [ ] [Hive on Tez vs MR vs Spark](./Hive/main.md) `Hive`
- [ ] [Hive UDF/UDAF/UDTF 区别与场景](./Hive/main.md) `Hive`
- [ ] [ORC vs Parquet](./Hive/main.md) `Hive`
- [ ] [Hive 索引与 Bloom Filter](./Hive/main.md) `Hive`

### Day 6 — 数据建模专题

- [ ] [维度建模 vs 范式建模 vs Data Vault](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [拉链表设计与 SCD Type 2](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [一致性维度（Conformed Dimension）](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [缓慢变化维 6 种类型](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [事实表三种类型（事务/周期快照/累积快照）](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [雪花模型 vs 星型模型](./Data-Modeling/main.md) `Data-Modeling`
- [ ] [数仓分层（ODS/DWD/DWS/DIM/ADS）职责](./Data-Modeling/main.md) `Data-Modeling`

### Day 7 — SQL 专题

- [ ] [连续登录 N 天 SQL（窗口函数 + 日期差分组）](./SQL/main.md) `SQL`
- [ ] [TopN 分组 SQL（row_number vs rank vs dense_rank）](./SQL/main.md) `SQL`
- [ ] [留存率 SQL（次日/7日/30日）](./SQL/main.md) `SQL`
- [ ] [漏斗转化 SQL](./SQL/main.md) `SQL`
- [ ] [同环比 SQL（lag/lead）](./SQL/main.md) `SQL`
- [ ] [行转列 / 列转行](./SQL/main.md) `SQL`
- [ ] [用户画像标签宽表设计](./Data-Modeling/main.md) `Data-Modeling`

### Day 8 — Kafka · NoSQL · Cache

- [ ] [Kafka 高吞吐原因（顺序写/零拷贝/批/分区）](./Kafka/main.md) `Kafka`
- [ ] [Kafka exactly-once 与幂等 Producer](./Kafka/main.md) `Kafka`
- [ ] [Kafka 分区分配策略（Range/RoundRobin/Sticky）](./Kafka/main.md) `Kafka`
- [ ] [Kafka ISR 机制](./Kafka/main.md) `Kafka`
- [ ] [HBase RowKey 设计](./NoSQL-and-Cache/main.md) `NoSQL-and-Cache`
- [ ] [HBase 读写流程](./NoSQL-and-Cache/main.md) `NoSQL-and-Cache`
- [ ] [Redis 缓存击穿/穿透/雪崩](./NoSQL-and-Cache/main.md) `NoSQL-and-Cache`

### Day 9 — Hadoop · 存储选型

- [ ] [HDFS 读写流程](./Hadoop/main.md) `Hadoop`
- [ ] [HDFS NameNode HA 方案](./Hadoop/main.md) `Hadoop`
- [ ] [YARN 资源调度（Capacity vs Fair）](./Hadoop/main.md) `Hadoop`
- [ ] [MapReduce shuffle 全过程](./Hadoop/main.md) `Hadoop`
- [ ] [列存 vs 行存使用场景](./Architecture/main.md) `Architecture`
- [ ] [数据湖 vs 数据仓库](./Architecture/main.md) `Architecture`
- [ ] [Lakehouse 架构演进](./Architecture/main.md) `Architecture`

### Day 10 — 系统设计 · 数据治理

- [ ] [设计直播实时数仓](./System-Design/main.md) `System-Design`
- [ ] [设计搜索推荐数据链路](./System-Design/main.md) `System-Design`
- [ ] [一次大故障复盘（线上数据延迟/口径错误怎么定位）](./System-Design/main.md) `System-Design`
- [ ] [数据资产管理体系](./Data-Governance/main.md) `Data-Governance`
- [ ] [元数据管理系统设计](./Data-Governance/main.md) `Data-Governance`
- [ ] [数据安全分级与脱敏](./Data-Governance/main.md) `Data-Governance`
- [ ] [项目深挖：你最复杂的一个数仓项目](./System-Design/main.md) `System-Design`

---

## 🧠 艾宾浩斯复习计划（Day 11 起启动）

> 每个 Day 的内容在 **1d / 2d / 4d / 7d / 15d / 30d** 后各复习一次（共 6 轮）
> 每次只看题目 → 自己回忆要点 → 回忆不出再看答案

| 复习日 | 复习内容 | 间隔 |
|--------|----------|------|
| Day 11 | Day 10 的 7 题 | 1d |
| Day 12 | Day 9 + Day 10 | 1d / 2d |
| Day 13 | Day 8 + Day 9 | 1d / 2d |
| Day 14 | Day 7 + Day 8 + Day 10 | 1d / 2d / 4d |
| Day 15 | Day 6 + Day 7 + Day 9 | 1d / 2d / 4d |
| Day 16 | Day 5 + Day 6 + Day 8 | 1d / 2d / 4d |
| Day 17 | Day 4 + Day 5 + Day 7 + Day 10 | 1d / 2d / 4d / 7d |
| ... | 滚动 1/2/4/7/15/30 天间隔 | |

---

## 📖 使用说明

1. **勾选复习**：直接在 GitHub 页面点击 checkbox 标记已完成（会自动 commit）
2. **跳转详解**：点击题目链接直达对应分类文件的详细解答
3. **新增题目**：后续 Day 内容会持续补充到对应分类文件夹中
