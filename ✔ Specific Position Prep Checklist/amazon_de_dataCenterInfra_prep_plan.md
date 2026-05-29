# Amazon DE II — Interview Prep Plan
> 目标：Strong Hire | 时间窗口：1-2周 | 更新：2026-05-28
> 面经来源：Blind 2025 offer holder、AWS Glassdoor Dec 2024、Medium Feb 2026、Exponent Q2 2026

---

## 核心原则
- LP权重60-70%，每轮都有2-3题，是重中之重
- Technical phone screen是75分钟重量级，不是热身——SQL+ETL+1个LP
- 技术题考的是**这个team的stack**，不是通用DE题库
- Manager + Bar Raiser轮权重最高，决定最终结果
- 面试前2天只mock，不学新东西

---

## 面试结构（2026真实面经）

| 轮次 | 时长 | 内容 | 重点 |
|------|------|------|------|
| OA | 90分钟 | SQL MCQ + 1 medium Python + behavioral | 筛选基础 |
| Technical Phone Screen | 75分钟 | SQL heavy + ETL设计 + 1个LP | **重量级，认真准备** |
| Loop Round 1 | 60分钟 | SQL Deep Dive + LP | 窗口函数/CTE/Top-K |
| Loop Round 2 | 60分钟 | Coding/ETL Design + LP | 幂等性/late data/streaming vs batch |
| Loop Round 3 | 60分钟 | Data Modeling + LP | Fact/Dim/SCD/Star vs Snowflake |
| Loop Round 4 | 60分钟 | System Design / Big Data概念 + LP | MPP架构/存算分离/pipeline设计 |
| Manager + Bar Raiser | 60分钟 | 技术判断 + LP深挖 | **权重最高，决定offer** |

---

## 5/28 Thu — Day 0（今晚下班后）

- [ ] 写出3个LP故事要点（格式：发生什么 / 我做了什么 / 结果数字）
  - 故事1：Ownership 或 Deliver Results
  - 故事2：Dive Deep（数据问题排查）
  - 故事3：Invent and Simplify 或 Bias for Action
- [ ] 发给Claude，今晚开始LP mock

---

## 5/29 Fri — Day 1

**SQL**
- [ ] LC 185 — Department Top Three Salaries（DENSE_RANK）
- [ ] LC 184 — Department Highest Salary
- [ ] LC 178 — Rank Scores
- [ ] LC 601 — Human Traffic of Stadium（Gap & Island连续行）

**系统设计热身**
- [ ] 口述一遍Xenith ETL pipeline设计（对着镜子/录音，5分钟）
  - 数据源 → Glue Spark → Redshift Pool/RCA/Owner层 → QuickSight/Athena/FLAIR
  - 要讲到：幂等性、增量逻辑、数据质量、分区策略

**LP Mock**
- [ ] 和Claude做1轮LP mock（Dive Deep题）

---

## 5/30 Sat — Day 2

**SQL**
- [ ] LC 1454 — Active Users（连续登录）
- [ ] LC 262 — Trips and Users（复杂过滤+聚合）
- [ ] LC 197 — Rising Temperature（LAG/日期比较）

**Redshift深度**（能自然说出来，不用背）
- [ ] DISTKEY选择逻辑：大表用join key，小表用ALL distribution
- [ ] SORTKEY两种：compound（范围查询）vs interleaved（多列过滤）
- [ ] 为什么要VACUUM：DELETE只标记不释放空间，VACUUM REINDEX重排
- [ ] Serverless vs Provisioned：按查询付费 vs 固定容量，各自适合场景
- [ ] Spectrum原理：leader node把外表查询下推到S3，不占cluster storage
- [ ] 能手写COPY/UNLOAD基本语法

**LP Mock**
- [ ] 和Claude做1轮LP mock（Deliver Results题）

---

## 5/31 Sun — Day 3

**系统设计**
- [ ] Hello Interview：Design a Multi-Source ETL Pipeline（口述45分钟）
  - 对标Xenith，讲完后和Claude对比差距
- [ ] 掌握：Step Functions为什么不用Lambda链式调用
  - 状态管理 / 自动retry / 绕过15分钟限制 / 可视化执行历史

**Python/Pandas + DSA**
- [ ] 增量ETL去重写出来
- [ ] 滚动7日均值写出来
- [ ] 找每组最新一条记录（两种写法）
- [ ] SCD Type 2有效记录过滤写出来
- [ ] LC 198 — House Robber（DP，真实考题）
- [ ] LC 1 — Two Sum（秒出）
- [ ] LC 56 — Merge Intervals

**LP Mock**
- [ ] 和Claude做1轮LP mock（Ownership题）

---

## 6/1 Mon — Day 4

**SQL**
- [ ] LC 1321 — Restaurant Growth（滑动窗口AVG）
- [ ] LC 1532 — The Most Recent Three Orders（ROW_NUMBER+分区）
- [ ] LC 1613 — Find the Missing IDs（递归CTE）

**Data Modeling专题**（2026面经独立考点）
- [ ] Star Schema vs Snowflake Schema：一句话说清楚各自适合场景
  - Star：denormalized，查询快，BI首选（90%场景）
  - Snowflake：normalized，适合大型维度表/层级复杂/存储敏感
- [ ] Fact表 vs Dimension表：grain是什么，surrogate key vs natural key
- [ ] SCD Type 1/2/3各自特点能口述：
  - Type 1：直接覆盖，无历史
  - Type 2：新增行+start/end_date，保留全历史（最常考）
  - Type 3：加列，只保留上一个值
- [ ] 练习：口述设计shipment defect数据模型（对标这个team）
  - fact_shipment_defect、dim_transport_mode、dim_owner、dim_defect_bucket
- [ ] 练习：口述设计clickstream/session数据模型（通用考题）

**CDK概念（2-3小时，只需能聊）**
- [ ] Stack / Construct / App三层结构
- [ ] cdk deploy / cdk diff 基本命令

---

## 6/2 Tue — Day 5

**系统设计**
- [ ] Hello Interview：Design a Data Quality Monitoring System（口述45分钟）
  - 对标他们Lambda DQ + CloudWatch告警架构
  - 要讲到：行数检查、空值率、分布异常、SNS告警、结果写S3

**MPP数据库概念**（2026 Blind L5 onsite真实考点）
- [ ] Single node SQL vs MPP架构差异：
  - Single node：一台机器处理所有数据，简单但受内存/CPU限制
  - MPP：数据分布在多节点，并行查询，leader node协调
- [ ] Redshift作为MPP的query execution：
  - Leader node解析SQL、生成执行计划、分发给compute nodes
  - Compute nodes并行执行、结果返回leader汇总
- [ ] 存算分离tradeoffs（Redshift Serverless/Snowflake/BigQuery都用这个）：
  - 优点：弹性扩缩容、按需付费、存储独立扩展
  - 缺点：网络I/O开销、data locality丧失、冷启动延迟
- [ ] Data skew问题：某个partition数据量远多于其他 → 热节点 → 整体变慢
  - 解法：选择高基数DISTKEY、使用DISTSTYLE EVEN、salting

**MWAA/Airflow概念**
- [ ] DAG基本结构：task依赖、sensor、retry
- [ ] 为什么用Airflow而不是Step Functions：复杂依赖图、Python灵活性、监控UI
- [ ] 能说出他们DCCD那条线为什么用MWAA

**DynamoDB**
- [ ] 为什么FLAIR用DynamoDB存state：无schema、毫秒延迟、serverless
- [ ] GSI：非主键列的二级索引
- [ ] 和RDS的tradeoff

**LP Mock**
- [ ] 和Claude做1轮LP mock（Invent and Simplify题）

---

## 6/3 Wed — Day 6

**全真Mock #1**
- [ ] LP 2题（英文，计时，不看提纲）
- [ ] SQL 1题（Hard，10分钟内）
- [ ] 系统设计1题（45分钟口述）
- [ ] 录音/复盘：哪里说散了？哪里缺数字？

**SQL Query Optimization专题**（面经独立考点）
- [ ] 能口述慢查询优化的系统性步骤：
  1. EXPLAIN ANALYZE看执行计划
  2. 检查full table scan / missing index
  3. Predicate pushdown（过滤下推，越早越好）
  4. Partition pruning（WHERE用date range不用函数）
  5. 避免WHERE里对列用函数：`DATE(col) = x` ❌ → `col >= x AND col < x+1` ✅
  6. 广播小表（DISTSTYLE ALL）
  7. 物化视图 / 预聚合
  8. VACUUM + ANALYZE更新统计信息
- [ ] 能解释Redshift WLM（Workload Management）：queue优先级管理

**Iceberg复习**（快速过）
- [ ] ACID / Time Travel / Schema Evolution各一句话
- [ ] 为什么Iceberg比Hive表好：hidden partitioning、snapshot isolation

---

## 6/4 Thu — Day 7

**SQL专项补强**
- [ ] MoM收入增长Top 3（真实考题，LAG+窗口函数）
- [ ] 留存率计算（Day 1/7/30 retention）
- [ ] PIVOT/行转列写法
- [ ] Redshift特有语法：COPY、UNLOAD、LISTAGG手写

**系统设计**
- [ ] Hello Interview：Design a Data Warehouse on AWS（口述45分钟）
  - Redshift选型/分层/Spectrum归档全讲到

**Glue深度**
- [ ] DPU概念：1 DPU = 4 vCPU + 16GB，auto-scaling
- [ ] Job bookmark原理：watermark记录增量位置
- [ ] Glue Catalog必要性：Athena/Spectrum查询的元数据必须在这里注册
- [ ] Glue vs EMR：serverless托管 vs 自管集群，cost vs 灵活性

**LP故事打磨**
- [ ] 把所有LP故事再说一遍，每个控制在90秒内
- [ ] 每个故事确认：有数字？"我"vs"我们"清晰？有DE技术细节？

---

## 6/5 Fri — Day 8

**全真Mock #2（提高难度，模拟Bar Raiser）**
- [ ] LP 3题（加入追问：如果重来？你的具体贡献？利益相关方？）
- [ ] SQL 2题（含query optimization题）
- [ ] Data Modeling题：口述设计shipment tracking schema
- [ ] 系统设计1题 + 深挖追问

**FLAIR架构深度**
- [ ] 能完整口述：SIM ticket → Step Functions → Bedrock Agent → Redshift查询 → 结果回写ticket
- [ ] 为什么SQS解耦：削峰、保证至少一次处理、解耦ticket系统和agent系统

**LP补充故事**
- [ ] 整理Insist on Highest Standards故事要点（2026高频）
- [ ] 整理Are Right A Lot故事要点（2026高频）

---

## 6/6 Sat — Day 9（面试前2天）

- [ ] 每天2个LP mock（英文，不打草稿，计时90秒）
- [ ] 系统设计1题完整口述（计时45分钟）
- [ ] 把5个问面试官的问题背熟
- [ ] 不学任何新东西

---

## 6/7 Sun — Day 10（面试前1天）

- [ ] 2个LP mock（重点练追问的回答）
- [ ] 快速过一遍Data Modeling要点（15分钟）
- [ ] 快速过一遍MPP概念（15分钟）
- [ ] 睡够8小时

---

## 问面试官的问题（准备5个）

- [ ] "How does the team prioritize between Xenith pipeline reliability and new feature development?"
- [ ] "What does the roadmap look like for FLAIRAgentCore — are you moving toward more autonomous multi-agent workflows?"
- [ ] "How does the team handle schema changes from upstream Procore data — is that a frequent pain point?"
- [ ] "What's the biggest scaling challenge you're facing right now with the Redshift setup?"
- [ ] "How much ownership does a DE II have over system design decisions vs following established patterns?"

---

## SQL题单（按优先级）

**P0 必须秒出思路：**
- [ ] LC 185 — Department Top Three Salaries
- [ ] LC 184 — Department Highest Salary
- [ ] LC 178 — Rank Scores
- [ ] LC 601 — Human Traffic of Stadium（Gap & Island）
- [ ] LC 1454 — Active Users（连续登录）
- [ ] LC 262 — Trips and Users
- [ ] LC 197 — Rising Temperature

**P1 需要熟练：**
- [ ] LC 1321 — Restaurant Growth（滑动窗口）
- [ ] LC 1532 — Most Recent Three Orders
- [ ] LC 1613 — Find the Missing IDs（递归CTE）
- [ ] MoM收入增长Top 3（真实考题，手写）
- [ ] 留存率Day 1/7/30（手写）

**P2 加分：**
- [ ] LC 1193 — Monthly Transactions I
- [ ] LC 1204 — Last Person to Fit in the Bus
- [ ] LC 1393 — Capital Gain/Loss

---

## Python/DSA题单

**DSA（面经真实出现）：**
- [ ] LC 198 — House Robber（DP，2025真实考题）
- [ ] LC 1 — Two Sum（hash map）
- [ ] LC 56 — Merge Intervals
- [ ] LC 239 — Sliding Window Maximum
- [ ] LC 347 — Top K Frequent Elements

**ETL/Pandas（必须能当场写出来）：**
```python
# 1. 增量ETL去重（最高频）
df_new = df_new[~df_new['id'].isin(df_existing['id'])]

# 2. 滚动7日均值
df['rolling_7'] = df.groupby('user_id')['revenue']\
    .transform(lambda x: x.rolling(7, min_periods=1).mean())

# 3. 每组最新一条记录
df.sort_values('ts').groupby('user_id').last()

# 4. 展开nested JSON
pd.json_normalize(df['metadata'])

# 5. SCD Type 2当前有效记录
df[df['end_date'].isna() | (df['end_date'] > pd.Timestamp.now())]

# 6. 展开nested list（真实考题）
import json
[item for row in df['tags'] for item in json.loads(row)]
```

---

## Data Modeling速查

**Star Schema口述模板：**
> "I'd go with a star schema here. The fact table holds the measures — in this case shipment_defect with fields like defect_count, transport_mode_id, owner_id, and date_key. Dimensions are denormalized: dim_transport_mode, dim_owner, dim_defect_bucket. This keeps queries simple and fast for BI consumption, which is the primary use case."

**SCD Type 2口述模板：**
> "For slowly changing dimensions like owner attribution, I'd use SCD Type 2 — each change creates a new row with a surrogate key, start_date, end_date, and is_current flag. This preserves full history so we can accurately attribute defects to whoever owned the shipment at that point in time."

**何时选Star vs Snowflake：**
- Star：BI报表、查询速度优先、维度表不太大 → 90%场景
- Snowflake：维度表极大且有共享子维度、存储成本敏感 → 少数场景

---

## MPP概念速查（2026新增考点）

**口述模板：**
> "Redshift is an MPP database. The leader node parses the query and generates an execution plan, then distributes work to compute nodes based on the distribution key. Each compute node processes its slice of data in parallel and returns results to the leader. The key design choice is the DISTKEY — it determines which node holds each row, so choosing the right one minimizes data movement during joins."

**存算分离tradeoffs口述：**
> "Separating compute and storage lets you scale them independently — you can spin up more compute during peak hours and scale back down, without paying for idle capacity. The trade-off is network I/O cost: data has to travel from storage to compute for every query, whereas tightly coupled systems benefit from data locality. For most analytics workloads, the elasticity benefit outweighs the network cost."

---

## LP故事库（填写区）

### 故事1：Ownership / Deliver Results
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

**能回答的追问：**
- 为什么选这个方案而不是其他？
- 你的个人贡献 vs 团队贡献？
- 如果重来会怎么做？

---

### 故事2：Dive Deep
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

### 故事3：Invent and Simplify / Bias for Action
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

### 故事4：Earn Trust / Have Backbone
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

### 故事5：Customer Obsession
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

### 故事6：Insist on Highest Standards（2026高频新增）
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

### 故事7：Are Right A Lot（2026高频新增）
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

---

## LP高频追问备用框架

每个故事都要能回答：
1. "What would you have done differently?" → 说一个小改进点，不否定整体决策
2. "What was the impact on the customer/business?" → 必须有数字或可量化改善
3. "How did you influence others who weren't directly under you?" → 横向影响力
4. "Tell me more about your specific contribution" → 把"we"换成"I"

---

## 技术速查：这个Team的Stack对照表

| 他们用的 | 通用等价物 | 面试时怎么说 |
|---------|-----------|------------|
| Datanet/Caravan | SQL ETL + 依赖调度 | "类似dbt+Airflow的内部工具" |
| BDT Maestro | Pipeline lifecycle管理 | "pipeline配置管理框架" |
| SIM/Tickety | Jira-like ticket系统 | "内部ticket管理，FLAIR的输入源" |
| Harmony | Web app hosting | "内部PaaS，部署FLAIR UI" |
| Brazil | Build system | "类似Maven/Gradle的内部构建工具" |
| Amazon Pipelines | CI/CD | "内部CI/CD，类似GitHub Actions" |

