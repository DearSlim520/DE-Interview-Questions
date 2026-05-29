# Whatnot Data Engineer — Interview Prep Plan
> 仅包含 Amazon DE II prep plan 里没有的内容
> 基于真实面经（Glassdoor/Blind 2023-2025）
> 更新：2026-05-28

---

## 面试结构（来自真实面经）

Whatnot DE面试**不像Amazon有明确LP框架**，更接近startup风格：

| 轮次 | 内容 | 特点 |
|------|------|------|
| Recruiter Call | 背景+薪资 | 标准 |
| HM Call | Mini case study + 角色期望 | 重点：business impact |
| Tech Screen | SQL（重）+ Python（轻）+ mini case study | 4题SQL/30分钟 |
| Virtual Onsite Panel | Case study + Collaboration deep dive + Technical deep dive | 提前给case study材料 |
| Culture Interview | 工程leader，给你问问题为主 | 低压力 |

**关键信息：**
- Case study **提前发给你**，要提前认真准备
- SQL考4题/30分钟，节奏快，难度Medium-Hard
- **没有Amazon式的LP追问**，但behavioral问题仍然存在
- 面试官风格：collaborative，不是对抗性的
- 从申请到offer约33天，节奏比Amazon快

---

## Whatnot特有的面试重点（Amazon没有的）

### 🔴 Case Study（最特殊，必须准备）

Whatnot会提前发case study材料，面试时围绕它讨论。典型场景：

**可能的case study类型（针对DE岗）：**
- 设计一个livestream事件的数据pipeline（GMV、viewer count、bid events）
- 现有慢查询/数据质量问题，如何诊断和解决
- 设计seller analytics数据模型

**Case study答题框架：**
```
1. 理解业务背景（Whatnot是什么，卖家/买家的核心行为是什么）
2. 明确数据需求（谁用？用来做什么决策？延迟要求？）
3. 提出方案 + tradeoff分析
4. 说明如何保证数据质量和可观测性
5. 主动提出改进方向
```

**提前做的功课：**
- [ ] 下载Whatnot app，走一遍买家+卖家流程，记录核心事件
  - 卖家开播 → 买家进房间 → 出价/购买 → 支付 → 发货
  - 这些事件就是pipeline的数据源
- [ ] 想好：如果我是Whatnot的DE，最重要的3张表是什么？
  - `livestream_events`（impression/join/leave/bid/purchase）
  - `seller_performance`（GMV、buyer count、conversion rate）
  - `transaction_fact`（订单、支付、退款）

---

### 🔴 Product Sense（Amazon完全没有）

Whatnot会考你对产品的理解，这是startup特有的：

- [ ] 深度使用Whatnot app（买家+卖家两个视角都要体验）
- [ ] 准备回答：
  - "What would you improve about Whatnot's data infrastructure?"
  - "What metrics would you track for a new seller's first 30 days?"
  - "How would you detect fraudulent bids in a livestream?"
- [ ] 了解Whatnot商业模式：平台抽佣、直播电商、社区驱动

---

### 🔴 Snowflake深度（Amazon考Redshift，Whatnot大概率用Snowflake/BigQuery）

JD里明确提到Snowflake。核心知识点：

**必须能讲的：**
- [ ] Virtual Warehouse概念：计算和存储分离，按需启停
- [ ] Snowflake vs Redshift核心差异：
  - Snowflake：存算分离，多cluster，自动扩缩容，Time Travel原生支持
  - Redshift：存算耦合，需要手动管理节点，成本更可控
- [ ] Micro-partitioning：Snowflake自动分区，不需要手动指定partition key
- [ ] Clustering Key：类似Redshift的Sort Key，大表查询优化
- [ ] Time Travel：`AT(TIMESTAMP => ...)` 或 `BEFORE(STATEMENT => ...)` 查历史数据
- [ ] Zero-Copy Cloning：瞬间克隆表/库，不复制数据，用于测试环境
- [ ] Snowpipe：持续自动从S3/GCS加载数据，类似Kinesis Firehose的作用
- [ ] Streams + Tasks：CDC变更捕获 + 定时任务，实现增量处理

**能手写的SQL：**
```sql
-- Time Travel查询
SELECT * FROM orders AT(TIMESTAMP => '2026-01-01 00:00:00'::timestamp);

-- Stream：捕获表变更
CREATE STREAM orders_stream ON TABLE orders;
SELECT * FROM orders_stream WHERE METADATA$ACTION = 'INSERT';

-- Clustering信息查询
SELECT SYSTEM$CLUSTERING_INFORMATION('large_table', '(date_col, category)');
```

---

### 🔴 dbt深度（Whatnot JD核心要求，Amazon不考）

- [ ] dbt incremental model：`is_incremental()` 宏 + unique_key去重
- [ ] dbt tests：`not_null`、`unique`、`accepted_values`、`relationships`
- [ ] dbt source freshness：监控上游数据是否按时到达
- [ ] dbt contract：`contract: enforced: true` 强制schema不被破坏
- [ ] dbt semantic layer：MetricFlow定义metrics，暴露给BI工具
- [ ] 能解释dbt在pipeline里的位置：数据进仓之后的T层（Transform）

**面试时能说的一句话定位：**
> "dbt handles the T in ELT — after raw data lands in the warehouse, dbt applies business logic, enforces data contracts, and exposes tested, documented models to downstream consumers."

---

### 🔴 Data Vault建模（JD明确提到，Amazon没考）

- [ ] 三个核心概念：
  - **Hub**：业务主键（seller_id, buyer_id, item_id）
  - **Link**：关系（purchase = seller + buyer + item的关系）
  - **Satellite**：属性和历史（seller的name、GMV、随时间变化的属性）
- [ ] Data Vault vs Star Schema：
  - Star Schema：查询简单，适合稳定的报表场景
  - Data Vault：支持schema evolution，适合源系统频繁变化的场景
  - Whatnot作为快速成长的公司，Data Vault更合适
- [ ] 面试时不需要深到设计完整Data Vault，但要能解释为什么选它

---

### 🔴 Kafka + Debezium CDC（Amazon用Kinesis，Whatnot用Kafka）

你的ProjectAtoZ做过MSK，这块不陌生。补充Debezium：

- [ ] Debezium是什么：读取数据库binlog，把每一行变更发到Kafka topic
- [ ] CDC事件格式：`before`/`after`/`op`(c=create, u=update, d=delete)
- [ ] 典型架构：PostgreSQL → Debezium → Kafka → Flink/Spark → Snowflake
- [ ] 为什么用CDC而不是定时批量查询：
  - 实时性（毫秒级延迟）
  - 不遗漏DELETE事件
  - 对源数据库压力小

---

### 🟡 Dagster（JD提到，Amazon不考）

- [ ] Dagster vs Airflow核心差异：
  - Airflow：task-centric（任务为中心），DAG是工作流
  - Dagster：asset-centric（数据资产为中心），知道每个asset的依赖关系
- [ ] 核心概念：Asset、Op、Job、Schedule、Sensor
- [ ] 面试不需要写Dagster代码，能解释asset-based orchestration的优势即可
- [ ] 一句话：
  > "Dagster's asset-centric model means you track the data itself, not just the tasks — so you can see lineage, freshness, and dependencies at the asset level, which makes debugging much easier."

---

### 🟡 数据可观测性（Monte Carlo / Great Expectations）

JD明确提到，Amazon没考：

- [ ] Monte Carlo：商业数据可观测性平台，自动检测数据异常（量、分布、schema变化、freshness）
- [ ] Great Expectations：开源数据质量框架，写expectation suite做断言测试
- [ ] 面试时能说的框架：
  ```
  数据质量 = 完整性 + 准确性 + 时效性 + 一致性 + 唯一性
  
  检测手段：
  - 行数异常：今天比昨天少50%以上 → 告警
  - Null率超阈值：关键字段null率 > 1% → 阻断pipeline
  - Schema变化：新增/删除列 → 自动通知
  - Freshness：上游数据超过N小时未更新 → 告警
  ```

---

### 🟡 Semantic Layer / Data Contracts

JD高频提到：

- [ ] **Semantic Layer**：在数据和BI工具之间加一层，统一定义metrics
  - 例：`GMV`的定义在semantic layer里只有一个，所有报表都用同一个
  - 工具：dbt MetricFlow、Cube.dev、Looker LookML
- [ ] **Data Contract**：上下游团队之间的schema协议
  - 生产者承诺：字段名、类型、nullable、freshness SLA
  - 消费者依赖：不会在没有通知的情况下schema变化
  - 实现方式：dbt contract + Schema Registry（Kafka场景）

---

## SQL题型补充（Whatnot特有场景）

Amazon的SQL题是通用业务场景，Whatnot会考**电商/直播场景**：

- [ ] **GMV计算**：按seller/category/时间段聚合交易金额
- [ ] **转化漏斗**：viewer → bidder → buyer的转化率
- [ ] **留存分析**：seller第30/60/90天的活跃率
- [ ] **异常检测**：某个seller的bid数量突然是均值的10倍（可能刷单）
- [ ] **时间序列**：每个seller最近7场直播的平均GMV趋势

**必练题（补充Amazon list之外的）：**
- [ ] LC 1193 — Monthly Transactions I（按月聚合，电商场景）
- [ ] LC 1174 — Immediate Food Delivery II（转化率，行为漏斗）
- [ ] LC 1204 — Last Person to Fit in the Bus（累积和+窗口函数）
- [ ] LC 1341 — Movie Rating（多表join+排名）
- [ ] LC 1393 — Capital Gain/Loss（状态追踪，类似订单状态）
- [ ] LC 1767 — Find the Subtasks That Did Not Execute（递归CTE）

---

## Python补充（Whatnot考mini case study风格）

不是纯算法，而是**给你一个数据问题，用Python解决**：

- [ ] 给一个JSON格式的livestream事件流，统计每个seller的实时GMV
- [ ] 检测异常bid：同一买家在1分钟内出价超过10次
- [ ] 实现简单的幂等incremental load（带dedup）

```python
# 幂等incremental load模板（面试必备）
def incremental_load(new_df, existing_df, key_col, timestamp_col):
    # 去掉已存在的记录
    existing_keys = set(existing_df[key_col].values)
    new_records = new_df[~new_df[key_col].isin(existing_keys)]
    # 合并并按时间排序
    result = pd.concat([existing_df, new_records]).sort_values(timestamp_col)
    return result.drop_duplicates(subset=[key_col], keep='last')
```

---

## 系统设计补充（Whatnot特有题型）

Hello Interview上这些题**Amazon prep没覆盖但Whatnot会考**：

- [ ] **Design a Real-time Seller Analytics Dashboard**
  - 直播结束后5分钟内seller能看到GMV、buyer数、转化率
  - 关键：Kafka → Flink/Spark Streaming → Snowflake → API → Dashboard
- [ ] **Design a Fraud Detection Pipeline for Livestream Bids**
  - 实时检测刷单行为
  - 关键：事件流 + 滑动窗口 + 规则引擎 + 人工复核队列
- [ ] **Design a Data Contract Enforcement System**
  - 如何保证上游schema变化不破坏下游
  - 关键：Schema Registry + 版本兼容性检查 + 告警

---

## Behavioral（Whatnot版，不是Amazon LP框架）

Whatnot的behavioral更casual，但这几个方向要准备：

- [ ] "Tell me about a time you worked closely with a non-technical stakeholder to solve a data problem"
  → 重点：翻译技术问题为业务语言，建立信任

- [ ] "Describe a time you had to make a technical trade-off under time pressure"
  → 重点：速度 vs 质量，如何决策，结果如何

- [ ] "How do you approach building data systems that other teams can trust and rely on?"
  → 重点：data contracts、documentation、data quality、SLA

- [ ] "Tell me about a pipeline you built that you're most proud of"
  → 直接用你的ProjectAtoZ素材，要有数字

---

## 问Whatnot面试官的问题

- [ ] "How does the team currently handle data contracts between upstream services and the warehouse?"
- [ ] "What does the data stack look like today — are you primarily on Snowflake or still migrating?"
- [ ] "How much of the pipeline is real-time vs batch, and where do you see that shifting?"
- [ ] "How does the DE team collaborate with product and ML teams on data model ownership?"
- [ ] "What's the biggest data reliability challenge you're dealing with right now?"

---

## 与Amazon Prep的优先级对比

| 内容 | Amazon | Whatnot | 行动 |
|------|--------|---------|------|
| Redshift深度 | 🔴必须 | 🟡了解 | 已在Amazon plan |
| Snowflake | ❌不考 | 🔴必须 | **新增，本文件** |
| dbt | ❌不考 | 🔴必须 | **新增，本文件** |
| LP/behavioral | 🔴严格框架 | 🟡casual版 | 故事复用，换语气 |
| Case study | ❌没有 | 🔴核心 | **新增，本文件** |
| Product sense | ❌没有 | 🟡有 | **新增，本文件** |
| SQL | 🔴通用场景 | 🔴电商场景 | 补充LC题目 |
| 系统设计 | 🔴AWS优先 | 🔴业务场景优先 | 补充题目 |
| Kafka/CDC | 🟡了解 | 🔴深度 | 补充Debezium |
| Data Vault | ❌不考 | 🟡了解 | **新增，本文件** |

