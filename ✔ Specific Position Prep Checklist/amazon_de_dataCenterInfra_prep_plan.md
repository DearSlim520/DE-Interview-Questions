# Amazon DE II — Interview Prep Plan
> 目标：Strong Hire | 时间窗口：1-2周 | 更新：2026-05-28

---

## 核心原则
- LP权重60-70%，是重中之重
- 技术题考的是**这个team的stack**，不是通用DE题库
- Phase 5 只补 CDK 概念，其余跳过
- 面试前2天只mock，不学新东西

---

## 5/28 Wed — Day 0

- [ ] 写出3个LP故事要点（格式：发生什么 / 我做了什么 / 结果数字）
  - 故事1：Ownership 或 Deliver Results
  - 故事2：Dive Deep（数据问题排查）
  - 故事3：Invent and Simplify 或 Bias for Action
- [ ] 发给Claude开始mock

---

## 5/29 Thu — Day 1

**SQL**
- [ ] LC 185 — Department Top Three Salaries（DENSE_RANK）
- [ ] LC 184 — Department Highest Salary
- [ ] LC 178 — Rank Scores
- [ ] LC 601 — Human Traffic of Stadium（Gap & Island）

**系统设计热身**
- [ ] 口述一遍Xenith ETL pipeline设计（对着镜子/录音，5分钟）
  - 数据源 → Glue Spark → Redshift Pool/RCA/Owner层 → QuickSight/Athena/FLAIR
  - 要讲到：幂等性、增量逻辑、数据质量、分区策略

---

## 5/30 Fri — Day 2

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
- [ ] 和Claude做1轮LP mock（追问练习）

---

## 5/31 Sat — Day 3

**系统设计**
- [ ] Hello Interview：Design a Multi-Source ETL Pipeline（口述45分钟）
  - 对标Xenith，讲完后和Claude对比差距
- [ ] 补充掌握：Step Functions为什么不用Lambda链式调用
  - 状态管理 / 自动retry / 绕过15分钟限制 / 可视化执行历史

**Python/Pandas**
- [ ] 增量ETL去重写出来
- [ ] 滚动7日均值写出来
- [ ] 找每组最新一条记录（两种写法）
- [ ] SCD Type 2有效记录过滤写出来

**LP Mock**
- [ ] 和Claude做1轮LP mock

---

## 6/1 Sun — Day 4

**SQL**
- [ ] LC 1321 — Restaurant Growth（滑动窗口AVG）
- [ ] LC 1532 — The Most Recent Three Orders（ROW_NUMBER+分区）
- [ ] LC 1613 — Find the Missing IDs（递归CTE）

**Glue深度**
- [ ] DPU概念：1 DPU = 4 vCPU + 16GB，auto-scaling
- [ ] Job bookmark原理：watermark记录增量位置
- [ ] Glue Catalog必要性：Athena/Spectrum查询的元数据必须在这里注册
- [ ] Glue vs EMR：serverless托管 vs 自管集群，cost vs 灵活性

**CDK概念（2-3小时，只需能聊）**
- [ ] Stack / Construct / App三层结构
- [ ] cdk deploy / cdk diff 基本命令
- [ ] 能说出"我们用CDK TypeScript管理所有AWS基础设施"并展开

---

## 6/2 Mon — Day 5

**系统设计**
- [ ] Hello Interview：Design a Data Quality Monitoring System（口述45分钟）
  - 对标他们Lambda DQ + CloudWatch告警架构
  - 要讲到：行数检查、空值率、分布异常、SNS告警、结果写S3

**MWAA/Airflow概念（只需能聊）**
- [ ] DAG基本结构：task依赖、sensor、retry
- [ ] 为什么用Airflow而不是Step Functions：复杂依赖图、Python灵活性、监控UI
- [ ] 能说出他们DCCD那条线为什么用MWAA

**DynamoDB深度**
- [ ] 为什么FLAIR用DynamoDB存state：无schema、毫秒延迟、serverless
- [ ] GSI：非主键列的二级索引
- [ ] 和RDS的tradeoff

**LP Mock**
- [ ] 和Claude做1轮LP mock（换新题）

---

## 6/3 Tue — Day 6

**全真Mock #1**
- [ ] LP 2题（英文，计时，不看提纲）
- [ ] SQL 1题（Hard，10分钟内）
- [ ] 系统设计1题（45分钟口述）
- [ ] 录音/复盘：哪里说散了？哪里缺数字？

**Iceberg复习**（你已经很熟，快速过）
- [ ] ACID / Time Travel / Schema Evolution三个核心能力能各用一句话解释
- [ ] 为什么Iceberg比Hive表好：hidden partitioning、snapshot isolation

---

## 6/4 Wed — Day 7

**SQL专项补强**
- [ ] Redshift特有语法能手写：COPY、UNLOAD、LISTAGG
- [ ] PIVOT/行转列写法
- [ ] 留存率计算（Day 1/7/30 retention）

**系统设计**
- [ ] Hello Interview：Design a Data Warehouse on AWS（口述45分钟）
  - Redshift选型/分层/Spectrum归档全讲到

**LP故事打磨**
- [ ] 把所有LP故事再说一遍，确保每个控制在90秒内
- [ ] 每个故事确认：有数字？"我"vs"我们"清晰？有技术细节？

---

## 6/5 Thu — Day 8

**全真Mock #2（提高难度）**
- [ ] LP 3题（加入追问：如果重来？你的具体贡献？利益相关方？）
- [ ] SQL 2题
- [ ] 系统设计1题 + 深挖追问

**FLAIR架构深度**
- [ ] 能完整口述：SIM ticket → Step Functions → Bedrock Agent → Redshift查询 → 结果回写ticket
- [ ] 为什么SQS解耦：削峰、保证至少一次处理、解耦ticket系统和agent系统

---

## 6/6-6/7 — 面试前2天

- [ ] 每天2个LP mock（语音，不打草稿）
- [ ] 每天1个系统设计完整口述（计时）
- [ ] 不学任何新东西
- [ ] 准备5个问面试官的问题（见下方）

---

## 问面试官的问题（准备5个）

- [ ] "How does the team prioritize between Xenith pipeline reliability and new feature development?"
- [ ] "What does the roadmap look like for FLAIRAgentCore — are you moving toward more autonomous multi-agent workflows?"
- [ ] "How does the team handle schema changes from upstream Procore data — is that a frequent pain point?"
- [ ] "What's the biggest scaling challenge you're facing right now with the Redshift setup?"
- [ ] "How much ownership does a DE II have over system design decisions vs following established patterns?"

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

---

## LP故事库（填写区）

### 故事1：Ownership / Deliver Results
**情境（S）：**

**任务（T）：**

**行动（A）：**

**结果（R，要有数字）：**

**能回答的追问：**
- 为什么选这个方案？
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

## LP高频追问备用答案框架

每个故事都要能回答：
1. "What would you have done differently?" → 说一个小的改进点，不否定整体决策
2. "What was the impact on the customer/business?" → 必须有数字或可量化的改善
3. "How did you influence others who weren't directly under you?" → 体现横向影响力
4. "Tell me more about your specific contribution" → 把"we"换成"I"，说清楚你做了什么

