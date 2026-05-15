# 📋 Data Contract & Data Observability

> Data Contract · Data Observability · Data Reliability Engineering

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [什么是 Data Contract](#q1-什么是-data-contract) | ⭐⭐ |
| 2 | [为什么需要 Data Contract](#q2-为什么需要-data-contract) | ⭐⭐ |
| 3 | [Data Contract 如何实现](#q3-data-contract-如何实现) | ⭐⭐⭐ |
| 4 | [Data Observability 四大维度](#q4-data-observability-四大维度) | ⭐⭐⭐ |
| 5 | [Data Observability 工具对比](#q5-data-observability-工具对比) | ⭐⭐ |
| 6 | [SLA vs SLO vs SLI in Data Engineering](#q6-sla-vs-slo-vs-sli) | ⭐⭐ |
| 7 | [Data Quality vs Data Observability 区别](#q7-data-quality-vs-data-observability) | ⭐⭐⭐ |

---

## Q1. 什么是 Data Contract

> 📌 **频率**: FAANG 2025 新热题 · ★★☆  
> `Data Contract` · Producer-Consumer Agreement · Schema Agreement

### 🎯 一句话定义

Data Contract = **数据生产者（Producer）和消费者（Consumer）之间的正式协议**，约定数据的 Schema、语义、质量标准、SLA 等。

```
┌─────────────────────────────────────────────────────────────┐
│                      Data Contract                           │
│                                                             │
│  Producer (上游团队/系统)          Consumer (下游团队/系统)   │
│  ┌──────────────┐                 ┌──────────────────┐      │
│  │ orders table │ ──合同──────────► │ analytics team   │      │
│  │              │                 │ ml team          │      │
│  └──────────────┘                 └──────────────────┘      │
│                                                             │
│  合同内容:                                                    │
│  ✅ Schema: order_id BIGINT NOT NULL, amount DECIMAL(10,2)  │
│  ✅ 语义: amount = tax-inclusive total                       │
│  ✅ 质量: NULL 率 < 0.1%, PK 唯一性                          │
│  ✅ SLA: 每天 06:00 UTC 前数据就绪                           │
│  ✅ 版本策略: 向后兼容，破坏性变更提前 30 天通知              │
└─────────────────────────────────────────────────────────────┘
```

### 📊 Data Contract 包含哪些内容

| 部分 | 内容 | 示例 |
|------|------|------|
| **Schema** | 字段名、类型、nullable | `order_id: BIGINT NOT NULL` |
| **语义定义** | 业务含义、计算口径 | `amount` 含税还是不含税 |
| **质量规则** | 完整性、唯一性、值域 | PK 唯一，amount > 0 |
| **SLA** | 数据就绪时间、更新频率 | 每日 06:00 UTC 前 |
| **Owner** | 谁负责、谁维护 | `team: data-platform, owner: alice` |
| **版本策略** | 如何管理 breaking changes | Semantic Versioning (v1.2.0) |

### 💡 类比记忆

> Data Contract = API 合同 📜
> - REST API 有 OpenAPI Spec (Swagger) 保证接口不随意变
> - Data Contract 对数据表做同样的事
> - 破坏合同 = Breaking Change → 需要提前通知、版本迭代

### 🧠 记忆锚点

```
Data Contract = Schema + 语义 + 质量规则 + SLA + Owner + 版本策略
类比: 数据的 OpenAPI Spec
生产者保证提供，消费者知道能依赖什么
```

---

## Q2. 为什么需要 Data Contract

> 📌 **频率**: 2025 · ★★☆  
> 解决数据工程的核心痛点

### 🎯 没有 Data Contract 会发生什么

```
常见场景 (Data Engineering 最痛的痛点):

场景1: Schema 悄悄变了
  上游工程师把 user_id INT → user_id BIGINT
  → 下游 Spark Job 类型不匹配 → 凌晨 3 点 pipeline 崩溃
  → DE 团队锅，但根因在上游

场景2: 口径理解不一致
  上游: revenue = 含退款的 gross revenue
  下游: 以为是 net revenue（去除退款后）
  → 财报数字错误，高管会议上暴雷 💣

场景3: 数据迟到无告警
  上游 ETL 延迟 4 小时
  → 下游模型跑了，但用的是昨天的维度数据
  → 数据静默错误，无人知晓

Data Contract 的价值：
  ✅ 预防: 合同明确约束上游行为
  ✅ 发现: 违约自动检测 + 告警
  ✅ 沟通: 减少跨团队沟通成本
  ✅ 信任: 下游团队可以放心依赖
```

### 📊 有 vs 没有 Data Contract

| 维度 | 没有 Contract | 有 Contract |
|------|--------------|-------------|
| Schema 变更 | 悄悄变，下游崩溃 | 强制版本化，破坏性变更需审批 |
| 数据延迟 | 下游默默失败 | SLA 违约自动告警 |
| 口径分歧 | 会议上争论不休 | 合同明确语义定义 |
| 责任归属 | 互相甩锅 | Owner 明确 |
| 发现问题 | 用户投诉才知道 | 实时监控，主动发现 |

### 💡 类比记忆

> 没有 Data Contract = 口头约定承包建房 🏚️
> - 施工队（Producer）随意改方案
> - 业主（Consumer）不知情，等入住才发现墙倒了
>
> 有 Data Contract = 正式合同 + 验收标准 📋
> - 改动需签变更单，业主审批
> - 完工前有验收检查（quality tests）

### 🧠 记忆锚点

```
三大核心痛点: Schema 悄悄变 + 口径理解不一致 + 数据迟到无感
Data Contract 解法: 预防(约束) + 发现(监控) + 沟通(文档) + 信任(可依赖)
```

---

## Q3. Data Contract 如何实现

> 📌 **频率**: 2025 · ★★★  
> YAML 定义 · CI/CD 验证 · Open Standards

### 🎯 实现路径

```
┌─────────────────────────────────────────────────────────────┐
│                 Data Contract 实现流程                        │
│                                                             │
│  1. YAML 定义合同                                            │
│       ↓                                                     │
│  2. Git 版本控制（PR 审查）                                   │
│       ↓                                                     │
│  3. CI 管道自动验证（Schema / Quality）                       │
│       ↓                                                     │
│  4. 运行时监控（持续检查生产数据）                             │
│       ↓                                                     │
│  5. 违约告警（Slack / PagerDuty）                            │
└─────────────────────────────────────────────────────────────┘
```

### 📝 YAML 合同示例（Data Contract Specification 标准）

```yaml
# orders_contract.yaml  (datacontract.com 开源规范)
dataContractSpecification: 0.9.3
id: orders-v1
info:
  title: Orders Data Contract
  version: 1.2.0
  owner: data-platform-team
  contact:
    name: Alice Smith
    email: alice@company.com

servers:
  production:
    type: bigquery
    project: prod-warehouse
    dataset: dw_sales

models:
  orders:
    description: "Completed customer orders (tax-inclusive)"
    fields:
      order_id:
        type: bigint
        required: true
        unique: true
        description: "Surrogate key, auto-generated"
      amount:
        type: decimal(12,2)
        required: true
        description: "Total order amount, tax-inclusive"
      status:
        type: string
        enum: [placed, shipped, delivered, returned, cancelled]
      order_date:
        type: date
        required: true

quality:
  type: great_expectations   # 或 dbt tests / soda
  specification:
    - column: order_id
      tests: [not_null, unique]
    - column: amount
      tests: [{expect_column_values_to_be_between: {min: 0, max: 1000000}}]

servicelevels:
  availability: 99.9%
  freshness:
    threshold: 6h             # 数据不超过 6 小时旧
  completeness: 99.5%         # 至少 99.5% 行完整

```

### 📝 CI/CD 验证示例

```yaml
# .github/workflows/contract-check.yml
- name: Validate Data Contract
  run: |
    datacontract lint orders_contract.yaml          # 格式验证
    datacontract test orders_contract.yaml          # 质量测试
    datacontract diff orders_contract_v1.yaml \
                        orders_contract_v2.yaml     # 检测 Breaking Changes
```

### 📊 主流实现方案

| 方案 | 特点 |
|------|------|
| **datacontract.com (开源)** | YAML 标准规范，CLI 工具，与 dbt/GX 集成 |
| **Soda Data Contracts** | 结合 Soda Core 质量检测 |
| **dbt contracts** (dbt 1.5+) | 在 schema.yml 中定义 `contracts: true`，强制类型检查 |
| **AsyncAPI / OpenAPI** | 借用 API 规范管理数据契约 |
| **自研 YAML + CI** | 灵活，但需自建验证工具链 |

### 🧠 记忆锚点

```
实现 = YAML 定义 + Git PR 审查 + CI 验证 + 运行时监控 + 违约告警
标准: datacontract.com 规范 | dbt contracts (1.5+)
Breaking Change 必须: 版本升级 + 通知消费者 + 过渡期
```

---

## Q4. Data Observability 四大维度

> 📌 **频率**: FAANG 2024-2025 热词 · ★★★  
> `Freshness` · `Volume` · `Schema` · `Distribution`

### 🎯 核心类比

> Data Observability = **数据的监控运维（DevOps for Data）** 🔭
> 就像应用监控有 CPU/内存/延迟指标，数据也需要健康指标

### 📊 四大维度详解

| 维度 | 监控什么 | 异常示例 | 检测方法 |
|------|----------|----------|----------|
| **Freshness（新鲜度）** | 数据多久没更新 | `orders` 表 8 小时没新数据 | MAX(updated_at) 距今时长 |
| **Volume（数量）** | 行数是否在预期范围内 | 今天只来了昨天 10% 的数据 | 行数 vs 历史均值 ±N σ |
| **Schema（结构）** | 字段是否增删改 | `amount` 列从 FLOAT 变 STRING | 元数据变更检测 |
| **Distribution（分布）** | 值分布是否异常 | `status` 中 90% 突然变 'error' | 统计分布对比、Z-score |

### 📝 各维度监控示例

```sql
-- Freshness 检查（多久没更新）
SELECT
  TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), MAX(updated_at), HOUR) AS hours_since_update
FROM orders;
-- 阈值: > 6h → WARNING, > 24h → CRITICAL

-- Volume 检查（行数异常检测）
WITH daily_counts AS (
  SELECT DATE(order_date) AS d, COUNT(*) AS cnt
  FROM orders
  GROUP BY 1
)
SELECT
  d,
  cnt,
  AVG(cnt) OVER (ORDER BY d ROWS 7 PRECEDING) AS rolling_avg,
  cnt / AVG(cnt) OVER (ORDER BY d ROWS 7 PRECEDING) AS ratio
FROM daily_counts
WHERE ratio < 0.5 OR ratio > 2.0;  -- 偏离均值 50% 以上告警

-- Distribution 检查（值分布异常）
SELECT status, COUNT(*) / SUM(COUNT(*)) OVER() AS pct
FROM orders
WHERE order_date = CURRENT_DATE()
GROUP BY status;
-- 与历史均值对比，发现 pct 突变
```

### 📊 五大维度（有时 +Lineage）

```
四大核心:
  Freshness     ← 新不新鲜
  Volume        ← 多不多
  Schema        ← 结构变没变
  Distribution  ← 值分布正不正常

第五维度 (部分工具):
  Lineage       ← 血缘完整性（上游是否都到了）
```

### 💡 类比记忆

> Data Observability = 体检报告 🏥
> - **Freshness** = 最后一次体检是多久以前（数据新鲜度）
> - **Volume** = 体重是否在正常范围（数据量）
> - **Schema** = 身高/体重指标项有没有变（字段结构）
> - **Distribution** = 各项指标的分布是否正常（值分布）

### 🧠 记忆锚点

```
四维度: Freshness + Volume + Schema + Distribution
Freshness = MAX(updated_at) 距今时长
Volume = 行数 vs 历史均值（统计异常检测）
Schema = 字段变更检测（元数据监控）
Distribution = 值分布异常（Z-score / 统计检验）
```

---

## Q5. Data Observability 工具对比

> 📌 **频率**: 2025 · ★★☆  
> Monte Carlo · Great Expectations · dbt Tests · Soda

### 🎯 工具生态图谱

```
┌────────────────────────────────────────────────────────────┐
│              Data Observability 工具全景                     │
│                                                            │
│  商业平台 (Managed)                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Monte Carlo  │  │  Acceldata   │  │  Bigeye      │     │
│  │ (领导者)      │  │              │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                            │
│  开源 / 半开源                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Great Expect.│  │ Soda Core    │  │  dbt Tests   │     │
│  │ (代码定义)    │  │ (YAML定义)   │  │ (轻量内置)    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└────────────────────────────────────────────────────────────┘
```

### 📊 主流工具对比

| 工具 | 定位 | 优势 | 劣势 | 适用场景 |
|------|------|------|------|----------|
| **Monte Carlo** | 企业级 Data Observability 平台 | 自动检测异常（ML-based）、全链路血缘 | 贵（商业）、需集成 | 大厂，数据可靠性要求高 |
| **Great Expectations** | 代码定义数据质量测试 | 灵活、开源、丰富断言库 | 学习曲线，需写代码 | 工程师驱动的质量检测 |
| **Soda Core** | YAML 定义数据检测 | 简单声明式、与 Airflow 集成好 | 商业功能需付费 | 中小团队，声明式偏好 |
| **dbt Tests** | dbt 内置测试 | 零额外依赖、与 dbt 天然集成 | 只覆盖 SQL 层，不含 Freshness/Volume 监控 | 已用 dbt 的团队 |
| **Apache Griffin** | 开源数据质量框架 | 开源免费、大数据原生 | 社区不活跃、配置复杂 | 国内大数据团队 |

### 📝 工具组合实践

```
推荐组合 (中型团队):
  dbt Tests       → Schema + 基本质量规则（已有 dbt）
  + Great Expectations → 复杂分布/异常检测
  + 自建 SQL 监控    → Freshness + Volume (简单查询 + 告警)

推荐组合 (大厂):
  Monte Carlo     → 自动全链路监控（ML-based 异常检测）
  + dbt contracts → Schema 强制校验
  + Data Contracts → 跨团队 SLA 约定
```

### 🧠 记忆锚点

```
Monte Carlo = ML自动检测 + 全链路 → 大厂首选（贵）
Great Expectations = 代码定义 + 丰富断言 → 工程师友好（开源）
Soda = YAML声明式 + 简单 → 快速上手（半商业）
dbt Tests = 零成本 + 已有 dbt → 入门首选（内置）
```

---

## Q6. SLA vs SLO vs SLI

> 📌 **频率**: 2025 · ★★☆  
> 数据可靠性指标体系 · Data SRE

### 🎯 三个概念定义

| 概念 | 全称 | 含义 | 数据工程示例 |
|------|------|------|-------------|
| **SLI** | Service Level Indicator | 实际测量的指标 | 今天数据就绪时间 = 07:23 |
| **SLO** | Service Level Objective | 内部目标（我们要达到的）| 数据就绪时间 ≤ 07:00，95% 月份达标 |
| **SLA** | Service Level Agreement | 对外承诺（合同）| 数据就绪时间 ≤ 08:00，违约赔偿 |

```
SLI (实际表现) ───► SLO (内部目标) ───► SLA (对外合同)
      ↑                   ↑                   ↑
  测量结果           比 SLA 严格           面向用户
  07:23             95% of time ≤ 07:00    ≤ 08:00
```

### 📝 数据工程常见 SLI/SLO

```yaml
# Data Pipeline SLO 示例
pipeline: daily_order_etl

SLIs:
  - freshness: MAX(order_date) 距今小时数     # 实际测量
  - completeness: 今日行数 / 昨日行数         # 实际测量
  - accuracy: 与上游对账偏差率               # 实际测量

SLOs:
  - freshness_slo: 数据就绪时间 ≤ 07:00 UTC，月达标率 ≥ 99%
  - completeness_slo: 行数偏差 ≤ 5%，月达标率 ≥ 99.5%
  - accuracy_slo: 金额偏差 ≤ 0.1%，月达标率 ≥ 99.9%

SLAs (对下游团队承诺):
  - data_ready_sla: 08:00 UTC 前数据就绪
  - 违约处理: 延误超 2h → 主动通知下游团队
```

### 💡 类比记忆

> - **SLI** = 体温计读数（37.8°C）🌡️
> - **SLO** = 健康目标（体温保持 36-37.5°C，90% 时间达标）🎯
> - **SLA** = 医疗保险合同（发烧超 38.5°C 报销住院费）📋

### 🧠 记忆锚点

```
SLI = 测量值（实际是多少）
SLO = 内部目标（我们要多好）→ 比 SLA 严格作为 buffer
SLA = 对外承诺（合同约定）→ 违约有后果
Error Budget = 1 - SLO 目标 = 允许失败的空间
```

---

## Q7. Data Quality vs Data Observability

> 📌 **频率**: 2025 概念区分题 · ★★★  
> 常被混淆的两个概念

### 🎯 核心区别

```
Data Quality:      我定义规则 → 批量校验 → 报告结果
                   （主动、规则驱动、时间点检查）

Data Observability: 持续监控 → 自动发现异常 → 实时告警
                   （被动、指标驱动、持续检查）
```

| 维度 | Data Quality | Data Observability |
|------|-------------|--------------------|
| **触发方式** | 定时运行（pipeline 后运行 dbt test）| 持续实时监控 |
| **规则来源** | 人工定义（not_null, unique...）| 自动学习历史模式（ML-based）|
| **覆盖范围** | 已知规则（已知未知）| 未知异常（未知未知）|
| **目标** | 验证数据符合预期 | 发现意外的数据问题 |
| **类比** | 食品安全检测（按标准验收）🔍 | 医院体检（全面扫描未知问题）🏥 |
| **代表工具** | Great Expectations, dbt Tests | Monte Carlo, Bigeye |

### 📝 协同使用示例

```
完整的数据可靠性体系:

  Data Contract  →  定义约束（合同）
       ↓
  Data Quality   →  验证已知规则（dbt tests / GX）
       ↓
  Data Observability → 监控未知异常（Monte Carlo）
       ↓
  Data SLA/SLO   →  衡量整体可靠性指标
```

### 💡 一个比喻看清区别

> **餐厅质量管理** 🍽️:
> - **Data Quality** = 厨房卫生检查（按标准逐项核查：食材新鲜度 ✅、温度 ✅、清洁度 ✅）
> - **Data Observability** = 顾客体验监控（观察翻台率、差评率、投诉数的异常趋势，发现你没想到的问题）

### 🧠 记忆锚点

```
Data Quality = 规则驱动 + 已知校验 + 时间点 + "已知的未知"
Data Observability = 持续监控 + ML异常检测 + 实时 + "未知的未知"
两者互补: Quality 保基线，Observability 兜底盲区
```
