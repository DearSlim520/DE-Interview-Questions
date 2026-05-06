# 🛡️ Data Governance

> 数据治理 (Data Governance) / 数据质量 (Data Quality) / 血缘 (Lineage) / 安全 (Security)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [DQC 六大维度（完准一唯及有）](#q1-dqc-六大维度) | ⭐⭐ |
| 2 | [数据血缘系统设计（Calcite + Neo4j）](#q2-数据血缘系统设计) | ⭐⭐⭐ |
| 3 | [数据资产管理体系](#q3-数据资产管理体系) | ⭐⭐ |
| 4 | [元数据管理系统设计](#q4-元数据管理系统设计) | ⭐⭐⭐ |
| 5 | [数据安全分级与脱敏](#q5-数据安全分级与脱敏) | ⭐⭐ |

---

## Q1. DQC 六大维度

> 📌 **频率**: 2024-2025 · ★★☆  
> `Data Quality Control` · 数据质量监控

### 🎯 六大维度一览

| 维度 | English | 校验示例 |
|------|---------|----------|
| **完**整性 | **Completeness** | `order_id IS NULL` 占比 < 0.01% |
| **准**确性 | **Accuracy** | 值域 (Value Range)、枚举 (Enum)、与上游对账 (Reconciliation) |
| **一**致性 | **Consistency** | 跨表主外键 (FK)、跨系统口径 (Cross-system Alignment) |
| **唯**一性 | **Uniqueness** | `COUNT(DISTINCT pk) = COUNT(*)` |
| **及**时性 | **Timeliness** | 产出时间 (Production Time) ≤ SLA（如 09:00 前） |
| **有**效性 | **Validity** | 格式 (Format) / 类型 (Data Type) 合规 |

### 📝 实战示例：`dwd.order` 表

```yaml
rules:
  Completeness:  order_id IS NULL ratio < 0.01%
  Uniqueness:    COUNT(DISTINCT order_id) = COUNT(*)
  Accuracy:      今日记录数 vs 7日均值 (7-day avg) ±30%
  Timeliness:    产出时间 (ready_time) < 09:00，否则告警
  Consistency:   SUM(amount) vs 上游 ODS 偏差 < 0.1%
  Validity:      amount > 0, status IN ('paid','pending','cancelled')
```

### ⚡ 告警分级 (Alert Severity)

| 级别 | 条件 | 处理 |
|------|------|------|
| 🔴 Blocking | 主键重复 (PK Duplication) / NULL 率爆表 | 阻断下游任务 (Block Downstream) |
| 🟡 Warning | 记录数波动超阈值 (Row Count Fluctuation) | 通知 + 人工确认 |

### 🧠 记忆锚点

```
6 维度 = Completeness · Accuracy · Consistency · Uniqueness · Timeliness · Validity
口诀 = 完 · 准 · 一 · 唯 · 及 · 有
```

---

## Q2. 数据血缘系统设计

> 📌 **频率**: 2024-2025 上升题 · ★★☆  
> `Data Lineage System Design`

### 🎯 四层架构 (Four-layer Architecture)

```
┌──────────────────────────────────────────────────────┐
│  ④ Service Layer  API查询上下游 · Impact Analysis · 可视化 │
├──────────────────────────────────────────────────────┤
│  ③ Storage Layer  Neo4j / JanusGraph (Graph DB)      │
│             Nodes: Table · Column · Task             │
│             Edges: Produce · Consume · ColumnMapping │
├──────────────────────────────────────────────────────┤
│  ② Parsing Layer  Apache Calcite / Antlr + 自研     │
│             提取 input/output tables + Column Mapping│
├──────────────────────────────────────────────────────┤
│  ① Collection Layer  Hive Hook · Spark Listener ·    │
│             Airflow Callback → 采集执行的 SQL / DAG  │
└──────────────────────────────────────────────────────┘
```

### 📝 解析示例 (Parsing Example)

```sql
INSERT INTO dwd.user_order
SELECT a.uid, b.amount
FROM ods.user a JOIN ods.order b ON a.uid = b.uid
```

**Table-level Lineage（表级血缘）**：
```
ods.user ──┐
            ├──► dwd.user_order
ods.order ─┘
```

**Column-level Lineage（字段级血缘）**：
```
dwd.user_order.uid    ← ods.user.uid
dwd.user_order.amount ← ods.order.amount
```

### ✅ 应用场景 (Use Cases)

| 场景 | 说明 |
|------|------|
| **Impact Analysis（影响分析）** | `ods.user` 字段改类型 → 一键查所有下游受影响任务/报表 |
| **Data Tracing（数据溯源）** | 报表数据异常 → 逆向追踪到源头表 |
| **Access Control（权限治理）** | 敏感字段血缘链路 → 自动继承脱敏策略 (Masking Policy) |

### 🧠 记忆锚点

```
Collection → Parsing(Calcite) → Graph Storage(Neo4j) → Impact Analysis
```

---

## Q3. 数据资产管理体系

> 📌 **频率**: 2025 · ★★☆  
> `Data Asset Management`

### 🎯 核心概念

数据资产管理 (Data Asset Management) = 把数据当作企业资产来管理，包括盘点 (Inventory)、分类 (Classification)、评估 (Valuation)、运营 (Operation)。

### 📊 体系框架

```
┌─────────────────────────────────────────────────────┐
│              Data Asset Management                    │
├─────────────────────────────────────────────────────┤
│  1️⃣ Asset Inventory (资产盘点)                      │
│     → 有哪些表/库/报表/API？自动采集 Metadata        │
│                                                     │
│  2️⃣ Asset Classification (资产分类)                 │
│     → 按域 (Domain) / 层级 (Layer) / 安全等级分     │
│                                                     │
│  3️⃣ Asset Valuation (资产评估)                      │
│     → 热度 (Popularity)、引用数、业务价值打分        │
│                                                     │
│  4️⃣ Asset Operation (资产运营)                      │
│     → 冷数据归档 (Archive) / 无主表治理 / 生命周期   │
└─────────────────────────────────────────────────────┘
```

### 📝 核心指标

| 指标 | English | 含义 |
|------|---------|------|
| 表热度 | Table Popularity | 近 30 天被查询次数 |
| 无主表率 | Orphan Table Rate | 无 Owner 的表占比 |
| 资产覆盖率 | Asset Coverage | 已注册到资产平台的表/总表 |
| 血缘完整度 | Lineage Completeness | 有血缘关系的表/总表 |

### 💡 类比记忆

> 数据资产管理 = 公司固定资产管理 🏢
> - 盘点 = 清点有多少台电脑（有多少张表）
> - 分类 = 按部门/等级贴标签（按域/安全级别分）
> - 评估 = 哪台电脑最常用/最值钱（热度/业务价值）
> - 运营 = 淘汰旧电脑 / 维修 / 分配（归档/治理/生命周期）

### 🧠 记忆锚点

```
Inventory → Classification → Valuation → Operation
盘点 → 分类 → 评估 → 运营
```

---

## Q4. 元数据管理系统设计

> 📌 **频率**: 2025 · ★★☆  
> `Metadata Management System`

### 🎯 元数据三大类

| 类型 | English | 示例 |
|------|---------|------|
| **技术元数据** | Technical Metadata | Schema、字段类型、存储位置、分区信息 |
| **业务元数据** | Business Metadata | 表含义、字段口径、Owner、标签 |
| **操作元数据** | Operational Metadata | ETL 运行记录、数据量变化、产出时间 |

### 📊 系统架构

```
┌─────────────────────────────────────────────────────┐
│  用户层 (User Layer)                                 │
│  → 数据地图 (Data Map) / 搜索 / 血缘可视化          │
├─────────────────────────────────────────────────────┤
│  服务层 (Service Layer)                              │
│  → API: 搜索 / 标签 / 血缘查询 / 权限              │
├─────────────────────────────────────────────────────┤
│  存储层 (Storage Layer)                              │
│  → MySQL (实体属性) + Elasticsearch (搜索)          │
│  → Neo4j (血缘图) + Kafka (事件流)                  │
├─────────────────────────────────────────────────────┤
│  采集层 (Collection Layer)                           │
│  → Hive Metastore / Spark Catalog / Airflow         │
│  → Hook + API + Log 三种采集方式                     │
└─────────────────────────────────────────────────────┘
```

### 📝 开源方案对比

| 工具 | 特点 |
|------|------|
| **Apache Atlas** | Hadoop 生态原生，Type System 灵活 |
| **DataHub (LinkedIn)** | 现代架构，GraphQL API，社区活跃 |
| **Amundsen (Lyft)** | 注重搜索发现 (Data Discovery) |
| **OpenMetadata** | 新兴标准化方案，Schema-first |

### 💡 类比记忆

> 元数据管理 = 图书馆管理系统 📚
> - Technical Metadata = 书的 ISBN、页数、存放位置
> - Business Metadata = 书的内容简介、适合谁看
> - Operational Metadata = 借阅记录、还书时间

### 🧠 记忆锚点

```
三类元数据: Technical + Business + Operational
架构: Collection → Storage(MySQL+ES+Neo4j) → Service → User
```

---

## Q5. 数据安全分级与脱敏

> 📌 **频率**: 2025 · ★★☆  
> `Data Security Classification & Data Masking`

### 🎯 数据安全分级 (Data Classification)

| 级别 | English | 示例 | 保护措施 |
|------|---------|------|----------|
| C1 公开 | Public | 公司新闻、产品介绍 | 无特殊要求 |
| C2 内部 | Internal | 内部文档、非敏感报表 | 需身份认证 |
| C3 机密 | Confidential | 用户手机号、订单金额 | 加密 + 脱敏 + 审批 |
| C4 绝密 | Restricted | 身份证号、密码、银行卡 | 强加密 + 最小权限 + 审计 |

### 🛠️ 脱敏策略 (Masking Strategies)

| 策略 | English | 示例 |
|------|---------|------|
| 遮盖 | Masking | `138****1234` |
| 哈希 | Hashing | `SHA256(phone)` |
| 替换 | Substitution | 真名 → 假名 |
| 截断 | Truncation | 只保留前 4 位 |
| 加密 | Encryption | AES 加密存储 |
| 泛化 | Generalization | 年龄 25 → 20-30 |

### 📊 脱敏实施层级

```
┌── 存储层脱敏 (Storage-level) ──┐
│  数据写入时就脱敏，原文不落盘    │
├── 查询层脱敏 (Query-level) ────┤
│  存储原文，查询时动态脱敏       │  ← 推荐（灵活）
│  实现：Hive UDF / View / Policy │
├── 应用层脱敏 (App-level) ──────┤
│  API 返回前脱敏                 │
└─────────────────────────────────┘
```

### 💡 类比记忆

> 数据安全 = 酒店房间分级 🏨
> - C1 Public = 大堂（谁都能进）
> - C2 Internal = 普通客房（需房卡）
> - C3 Confidential = 行政楼层（需特殊授权）
> - C4 Restricted = 总统套房（安保 + 独立电梯 + 全程监控）

### 🧠 记忆锚点

```
分级: C1 Public → C2 Internal → C3 Confidential → C4 Restricted
脱敏: Masking / Hashing / Substitution / Truncation / Encryption / Generalization
```
