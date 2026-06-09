# Amazon Data Engineer 面试完整攻略 (一亩三分地)

> 来源: https://www.1point3acres.com/bbs/ + https://www.1point3acres.com/interview/problems/company/amazon  
> 整理时间: 2026-06-08  
> Amazon共851道面试题, 以下结合面经帖 + 题库, 按类别列出DE相关具体真题

---

## 🏗️ 面试流程

### DE面试流程:
```
海投/内推/猎头 → Recruiter Call (30min) → OA → Technical Phone Screen → Onsite (4-5轮)
```

| 阶段 | 时长 | 说明 |
|------|------|------|
| 投递→OA | 3-5天 | 海投后最快2天recruiter发邀请 |
| OA | ~2小时 | 全部SQL + Work Style Survey |
| OA→Phone Screen | 1-2天 | 通过后很快安排 |
| Phone Screen | 60min | SQL + 可能Python + LP |
| Phone Screen→Onsite | 2-4周 | |
| Onsite | 4-5轮×55min | 每轮都有LP |
| Onsite→Decision | 1-2周 | Bar Raiser write-up可能延长 |
| **全流程** | **~5周** | |

### Onsite轮次 (DE/BIE):
1. **Data Modeling / Advanced SQL** — 设计表结构 + 复杂SQL
2. **Coding** — SQL为主, 可能Python
3. **Data Visualization / Case Study** — Dashboard设计, 指标体系
4. **System Design** — 数据管道/ETL设计
5. **Bar Raiser** — LP行为面 (独立于团队, 决定offer)

### ⚡ 关键信号 (来自1P3A Guide):
- Bar Raiser write-ups **主导** HC讨论
- "Strong technical, weak LP" → Amazon **down-level** 而非reject
- 常见过会组合: 2 strong + 1 lean hire + 1 偏弱
- Phone Screen后 **3天内没回复基本凉了**
- 使用Amazon内部 **Livecode** 编辑器 (语法高亮, 代码不执行, 需手动dry-run)

---

## 🔥 SQL 真题 (DE核心 - 最最最重要!)

### OA SQL题 (LeetCode Easy + Medium)
| 真题描述 | 难度 | 来源 |
|----------|------|------|
| 音乐订阅: 选出subscriber听了more than 20 hours per month and listening in [某条件] | Medium | 2025-05 Phone Screen |
| 用Redshift语法写query | Medium | 2025-05 Onsite |
| OA全部SQL, LC easy+medium水平, 规定时间内可提前做完 | Easy-Med | 2025-07 OA |
| OA还会问SQL用法的概念题 | Concept | 2025-01 OA |

### Phone Screen / Onsite SQL高频考点:
| 考点 | 具体内容 | 注意事项 |
|------|----------|----------|
| **Window Functions** | ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD, SUM/AVG OVER (PARTITION BY ... ORDER BY ...) | 必考! |
| **CTE** | WITH clause, Recursive CTE | ⚠️ 写完后面试官会追问能否不用CTE |
| **CASE** | CASE WHEN ... THEN ... END | 配合聚合使用 |
| **Complex JOINs** | LEFT/RIGHT/FULL OUTER, self-join, cross join | |
| **GROUP BY + HAVING** | 聚合过滤 | |
| **Subqueries** | correlated subquery, EXISTS/IN | |

### 推荐LeetCode SQL刷题清单:
| 题号 | 题目 | 难度 | 考点 |
|------|------|------|------|
| 175 | Combine Two Tables | Easy | JOIN |
| 176 | Second Highest Salary | Medium | Subquery/Window |
| 177 | Nth Highest Salary | Medium | Window Function |
| 178 | Rank Scores | Medium | DENSE_RANK |
| 180 | Consecutive Numbers | Medium | LAG/LEAD |
| 184 | Department Highest Salary | Medium | Window + JOIN |
| 185 | Department Top 3 Salaries | Hard | DENSE_RANK + CTE |
| 197 | Rising Temperature | Easy | Self-JOIN / LAG |
| 262 | Trips and Users | Hard | CASE + GROUP BY |
| 550 | Game Play Analysis IV | Medium | CTE + Window |
| 1158 | Market Analysis I | Medium | LEFT JOIN + 聚合 |
| 1193 | Monthly Transactions I | Medium | CASE + GROUP BY |
| 1321 | Restaurant Growth (Window) | Medium | SUM OVER (ROWS) |
| 1341 | Movie Rating | Medium | CTE + UNION |
| 1934 | Confirmation Rate | Medium | LEFT JOIN + AVG |

---

## 🐍 Python Coding 真题

### 面经中出现的具体Python题:
| 真题描述 | 难度 | 关键点 | 来源 |
|----------|------|--------|------|
| 给一个json dict variable, 找最深子节点并保存高度 | Medium | 递归/while loop, **不能用json包** | 2022-05 电面 |
| JSON处理 (不能import json, 用基本语言写) | Medium | 字符串解析 | 2022-05 电面 |
| LeetCode 瑶瑶吴尔 (具体题号被隐藏) | Medium | 刷题 | 2022-06 电面 |

### 题库中DE相关Python/Coding题:
| 题目 | 时长 | 难度 | 说明 |
|------|------|------|------|
| **Log Aggregation and Group-By** | 45min | Medium | 解析日志流, 聚合分组, 生成排序报告(dedupe by event id, group by user, top-K) |
| **Calculating Message Latency from Two CSV Logs** | - | Medium | 两个CSV日志计算消息延迟 |
| **From logs, find the most frequent event sequence** | - | Medium | 日志中找最频繁事件序列 |
| **Detect duplicates/near-duplicates in streaming events** | - | Medium | 流式数据去重/近似去重 |
| **Get Nested Object by Path** | 30min | Medium | `get(object, "ab[1].c.d[2][13]")` 实现嵌套对象路径访问 |
| **Replace tokens in string using delimiter-based key-value rules** | - | Medium | 字符串token替换(ETL转换) |
| **Abusive Books in Reading Event Stream** | 60min | Medium | 流式读取事件, 检测>20%用户到达最后5%但未过10%的书 |
| **LRU Cache (LC 146)** | 30min | Medium | doubly-linked list + hashmap, O(1) get/put |
| **Service Shutdown / Topological Sort** | 45min | Medium | 服务依赖关系, 关闭后级联不可用 (Course Schedule变体) |
| **Merge time intervals from multiple detectors** | - | Medium | LC 56 区间合并变体 |
| **Find All Target Anagrams in a Character Stream (Sliding Window)** | - | Medium | LC 438 流式滑动窗口 |

### 推荐LeetCode Coding刷题清单:
| 题号 | 题目 | 难度 | 与DE关联 |
|------|------|------|---------|
| 42 | Trapping Rain Water | Hard | 高频Amazon |
| 48 | Rotate Image | Medium | 矩阵操作 |
| 56 | Merge Intervals | Medium | **时间区间合并=数据管道常见** |
| 79 | Word Search | Medium | DFS |
| 128 | Longest Consecutive Sequence | Medium | HashSet |
| 138 | Copy List with Random Pointer | Medium | 链表/HashMap |
| 146 | **LRU Cache** | Medium | **缓存策略=DE核心** |
| 200 | Number of Islands | Medium | BFS/DFS |
| 207 | **Course Schedule** | Medium | **拓扑排序=Airflow DAG** |
| 210 | **Course Schedule II** | Medium | **拓扑排序** |
| 323 | Connected Components | Medium | **Union-Find=数据分组** |
| 347 | Top K Frequent Elements | Medium | Heap |
| 438 | Find All Anagrams in a String | Medium | 滑动窗口 |
| 706 | Design HashMap | Easy | 底层数据结构 |

---

## 📐 Data Modeling 真题

### 面经中出现的具体Modeling题:
| 真题描述 | 场景 | 关键点 | 来源 |
|----------|------|--------|------|
| 给你个case, 让你设计table | 通用业务场景 | 考Bridge Table | 2022-06 Onsite |
| 棒球相关数据的model design | 体育数据 | 面试官是棒球迷, 不懂规则很棘手 | 2022-05 电面 |
| Star Schema vs Snowflake Schema | 数仓设计 | L5 BIE必考 | 2025-02 |

### Data Modeling必备知识:
| 考点 | 具体内容 |
|------|----------|
| **Star Schema** | Fact table + Dimension tables, denormalized |
| **Snowflake Schema** | Normalized dimension tables |
| **Bridge Table** | 多对多关系中间表 |
| **Slowly Changing Dimensions (SCD)** | Type 1/2/3 |
| **Normalization** | 1NF → 2NF → 3NF → BCNF |
| **OLTP vs OLAP** | 行存储vs列存储, 事务vs分析 |
| **Fact Table Types** | Transaction, Snapshot, Accumulating |
| **Grain定义** | 每行代表什么 |

---

## 🏗️ System Design 真题

### 面经中的DE System Design方向:
| 考点 | 具体内容 | 来源 |
|------|----------|------|
| ETL Pipeline设计 | 数据摄入→转换→加载 | Onsite |
| AWS服务选型 | S3, Glue, Athena, Redshift, EMR/Spark, Lambda, Kinesis | JD要求 |
| 数据血缘与治理 | Lineage, Quality, Governance | BIE方向 |
| Streaming data processing | Real-time analytics | Preferred |

### 题库中的System Design真题 (8题):
| 题目 | 时长 | 难度 | 与DE关联度 |
|------|------|------|-----------|
| **Distributed Training Data Pipeline** | 60min | Hard | ⭐⭐⭐ 从S3/streaming摄入, tokenize, deduplicate, quality-filter, 输出sharded训练数据 |
| **Pub-Sub Messaging System (OOD)** | 60min | Medium | ⭐⭐⭐ topics, subscribers, fan-out, 并发/交付保证 (Kafka-like) |
| **Permission Control System** | 60min | Medium | ⭐⭐ 文件共享权限, 首页列出可访问文件 (数据治理) |
| **Rate Limiter (OOD)** | 45min | Medium | ⭐⭐ 限流器设计 |
| **ML System Design — Search/Ranking** | 60min | Hard | ⭐ ranking + A/B design |
| Pizza/Restaurant System (OOD) | 50min | Medium | ⭐ |
| Minesweeper Game Design | 60min | Medium | ⭐ |
| Credit Card System (OOD) | 45min | Medium | ⭐ |

### DE System Design必备框架:
```
1. 需求澄清 (Functional + Non-functional)
2. 数据流设计 (Source → Ingestion → Processing → Storage → Serving)
3. 技术选型 (Batch: Spark/EMR | Stream: Kinesis/Kafka | Storage: S3/Redshift)
4. Schema设计 (数据模型)
5. 容错/监控 (Retry, Dead-letter queue, Data quality checks)
6. Scale估算 (QPS, Storage, Throughput)
```

---

## 🗣️ Behavioral / LP 真题 (每轮必问!)

### 🆕 2026新必考: GenAI Usage BQ
> **Nearly every Amazon round in 2026** asks a GenAI-themed BQ!

| 具体问法 | 准备思路 |
|----------|----------|
| "Tell me about a time you used GenAI" | 用AI辅助coding/data分析的实例 |
| "A GenAI failure you experienced" | AI生成错误结果如何发现和修正 |
| "How do you use AI day-to-day?" | 日常workflow中AI工具使用 |
| Agent/multi-turn design depth | 技术轮可能probe |

### LP高频真题 (2026):
| LP原则 | 具体问题 | STAR准备要点 |
|--------|----------|-------------|
| **Customer Obsession** | 讲一个你为客户做出超出预期的事 | 量化customer impact |
| **Ownership** | Most challenging project, why? | 展示全局视角 |
| **Dive Deep** | 讲一个你深入细节发现问题的经历 | 技术depth + impact |
| **Deliver Results** | Tight deadline如何完成 | 量化结果 |
| **Disagree & Commit** | 你跟队员或老板有相反意见, 如何处理? | 逻辑说服 + 最终commit |
| **Earn Trust** | Tough feedback (given AND received) | 具体例子 |
| **Have Backbone** | 讲一个你坚持己见的经历 | 数据支撑 |
| **Bias for Action** | Calculated risk你做过的 | 风险评估过程 |
| **Invent & Simplify** | Out-of-scope contribution | 创新 + 简化 |
| **Learn & Be Curious** | 讲一个你学新技术解决问题的经历 | 学习能力 |
| **(面经原题)** | 讲一个你和其他annotator工作的经历, 有什么challenge? | 协作 + 冲突解决 |

### LP准备策略:
- ⚠️ **准备15+个STAR故事** (8-10个不够!)
- 每轮都有20-30分钟LP, 不是只有BQ轮才问
- 同一事件可以从不同LP角度讲
- **量化结果** (具体数字!) 可以rescue borderline技术轮
- Bar Raiser write-up主导HC讨论

---

## 📚 OJ练习题库 (Amazon 806题, Page 1)

| # | 题目 | 对应LC | DE相关度 |
|---|------|--------|---------|
| 1 | Trapping Rain Water | LC 42 | ⭐ |
| 2 | Implement a HashMap Without Built-in Libraries | LC 706 | ⭐⭐ |
| 3 | Online Review Content Moderation | 原题 | ⭐ |
| 4 | Minimum Redistribution Cost in Circular Warehouses | 原题 | ⭐ |
| 5 | **Course Schedule Variant (Topological Sort)** | LC 207/210 | ⭐⭐⭐ |
| 6 | **From logs, find the most frequent event sequence** | 原题 | ⭐⭐⭐ |
| 7 | **Design an index to find sentences by word** | 原题 | ⭐⭐⭐ |
| 8 | BFS on grid/graph | LC 200 | ⭐⭐ |
| 9 | Word Search | LC 79 | ⭐ |
| 10 | **Calculating Message Latency from Two CSV Logs** | 原题 | ⭐⭐⭐ |
| 11 | **Replace tokens using delimiter-based key-value rules** | 原题 | ⭐⭐⭐ |
| 12 | Max Money From k Consecutive Bags | DP | ⭐ |
| 13 | **Merge time intervals from multiple detectors** | LC 56变体 | ⭐⭐⭐ |
| 14 | **Detect duplicates/near-duplicates in streaming events** | 原题 | ⭐⭐⭐ |
| 15 | **Find All Target Anagrams in a Character Stream** | LC 438变体 | ⭐⭐ |
| 16 | **Course Schedule Conflict Check (Topological Sort)** | LC 210变体 | ⭐⭐⭐ |
| 17 | **Merge Products into Categories (Union-Find)** | Union-Find | ⭐⭐⭐ |
| 18 | **Count connected components (Union-Find)** | LC 323 | ⭐⭐⭐ |
| 19 | Full-Stack Bug Fix: Recommendations Not Showing | 全栈 | ⭐ |
| 20 | Minimum Total Travel Time on Circular Hub Ring | 环形 | ⭐ |
| 21 | Minimum Conflicts in Merging Two Branches | 合并 | ⭐⭐ |
| 22 | Design Robot Human-Avoidance Navigation System | 架构 | ⭐ |
| 23 | Circular Array Shortest Path Sum | 前缀和 | ⭐ |
| 24 | Longest Near-Consecutive Sequence (Diff < k) | LC 128变体 | ⭐⭐ |
| 25 | Determine Relationship Between Two Tree Nodes | 树 | ⭐ |
| 26 | Student Printing Queue: All Valid Orders | 排列 | ⭐ |
| 27 | Grid BFS: Minimum steps in 2D matrix | BFS | ⭐⭐ |
| 28 | **Implement an LRU Cache** | LC 146 | ⭐⭐⭐ |
| 29 | Optimal Bucket Batching to Minimize Padding (DP) | DP | ⭐ |
| 30 | Rotate Square Matrix by 90 Degrees | LC 48 | ⭐ |

---

## 📝 JD关键要求 (真实JD摘录)

### 基本要求:
- Strong programming: **Python, Java, or Scala**
- Expertise in **SQL** + relational and NoSQL databases
- Cloud: **AWS** (S3, Glue, Athena, Redshift, EMR, Lambda, Kinesis)
- **Data modeling, data warehousing, ETL design patterns**
- Git + CI/CD pipelines

### 加分项:
- ML workflows and model deployment
- Infrastructure as Code (**CDK**)
- **Streaming data processing and real-time analytics**
- Big data: **Hadoop, Spark, Hive**

---

## 💰 薪资参考

**L5 BIE Offer (2025年初):**
| 组成 | 金额 |
|------|------|
| Base Salary | $140K |
| Sign-on Bonus Y1 | $25K |
| Sign-on Bonus Y2 | $15K |
| RSU (4年) | $60K |
| **Total Comp** | **~$180K** |

---

## 🎯 终极备战Checklist

### Week 1-2: SQL (每天2-3题)
- [ ] LeetCode SQL Top 50 (重点: Window Function, CTE, Complex JOIN)
- [ ] 练习不用CTE解决问题的能力
- [ ] 熟悉Redshift语法差异

### Week 2-3: Coding (每天1-2题)
- [ ] LRU Cache (LC 146) — 必须手写
- [ ] Course Schedule I & II (LC 207, 210) — 拓扑排序
- [ ] Merge Intervals (LC 56) — 区间合并
- [ ] Union-Find题型 (LC 323)
- [ ] 日志处理/流式数据题
- [ ] JSON嵌套结构递归处理

### Week 3-4: System Design + Data Modeling
- [ ] 设计ETL Pipeline (Batch + Stream)
- [ ] Star Schema vs Snowflake — 能画图讲解
- [ ] Pub-Sub系统设计
- [ ] AWS服务选型 (S3→Glue→Redshift vs Kinesis→Lambda)
- [ ] SCD Type 2实现

### Week 1-5: LP (贯穿整个准备期)
- [ ] 准备15+个STAR故事
- [ ] 每个故事映射2-3个LP
- [ ] 准备GenAI Usage BQ (2026必考!)
- [ ] 量化所有impact (数字!)
- [ ] 练习2分钟内讲完一个STAR

### 注意事项:
- ⚠️ 面试官很可能是印度裔, 多听印式英语
- ⚠️ Coding可以用Python (不一定要Java)
- ⚠️ Livecode编辑器代码不执行, 需手动dry-run
- ⚠️ 5个工作日内回复onsite结果

---

## 🔗 原帖链接

| # | 标题 | 链接 |
|---|------|------|
| 1 | 亚麻data第一轮phone screen面经 (2026-02) | [link](https://www.1point3acres.com/bbs/thread-1166264-1-1.html) |
| 2 | Amazon DE（偏BIE）面试备战 (2025-10) | [link](https://www.1point3acres.com/bbs/thread-1151863-1-1.html) |
| 3 | Amazon DE OA 面经 (2025-07) | [link](https://www.1point3acres.com/bbs/thread-1135736-1-1.html) |
| 4 | 亚麻 SR BIE Coding 挂经 (2025-05) | [link](https://www.1point3acres.com/bbs/thread-1130741-1-1.html) |
| 5 | 亚麻DE Phonescreen挂经 (2025-05) | [link](https://www.1point3acres.com/bbs/thread-1129534-1-1.html) |
| 6 | 亚麻 de面试 Onsite (2025-05) | [link](https://www.1point3acres.com/bbs/thread-1129003-1-1.html) |
| 7 | 亚麻DE timeline (2025-03) | [link](https://www.1point3acres.com/bbs/thread-1115925-1-1.html) |
| 8 | **L5 Amazon BIE 面经 (2025-02)** ⭐最详细 | [link](https://www.1point3acres.com/bbs/thread-1111686-1-1.html) |
| 9 | Amazon DE OA分享 (2025-01) | [link](https://www.1point3acres.com/bbs/thread-1105727-1-1.html) |
| 10 | 亚麻DE 印度br电面挂经 (2024-12) | [link](https://www.1point3acres.com/bbs/thread-1101573-1-1.html) |
| 11 | 亚麻 DE 过经 Onsite 5轮 (2022-06) | [link](https://www.1point3acres.com/bbs/thread-903428-1-1.html) |
| 12 | 亚麻数据工程师 电面 (2022-05) | [link](https://www.1point3acres.com/bbs/thread-897038-1-1.html) |
| 13 | 香蕉厂DE 面试流程 (2022-02) | [link](https://www.1point3acres.com/bbs/thread-855159-1-1.html) |
| 14 | 亚麻DE电面面经 (2022-03) | [link](https://www.1point3acres.com/bbs/thread-867825-1-1.html) |
| 15 | Amazon DE On site (2022-03) | [link](https://www.1point3acres.com/bbs/thread-864042-1-1.html) |
| - | **题库 (851题)** | [link](https://www.1point3acres.com/interview/problems/company/amazon) |

---

> ⚠️ 部分帖子详细题目被一亩三分地积分墙(188+)隐藏。题库具体题目描述需要1P3A会员权限。
