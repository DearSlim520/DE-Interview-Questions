# 🔧 dbt (data build tool)

> Analytics Engineering · ELT Transform Layer · SQL-first Data Modeling

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [什么是 dbt，与传统 ETL 的区别](#q1-什么是-dbt) | ⭐⭐ |
| 2 | [dbt Models 分类：staging / intermediate / mart](#q2-dbt-models-分类) | ⭐⭐ |
| 3 | [dbt Tests：generic vs singular](#q3-dbt-tests) | ⭐⭐ |
| 4 | [dbt Sources & Freshness 配置](#q4-dbt-sources--freshness) | ⭐⭐ |
| 5 | [dbt Lineage & DAG 可视化](#q5-dbt-lineage--dag) | ⭐⭐ |
| 6 | [dbt vs Stored Procedures](#q6-dbt-vs-stored-procedures) | ⭐⭐⭐ |
| 7 | [dbt Incremental Models 原理](#q7-dbt-incremental-models) | ⭐⭐⭐ |

---

## Q1. 什么是 dbt

> 📌 **频率**: FAANG 2024-2025 高频 · ★★☆  
> `data build tool` · Analytics Engineering · ELT Transform Layer

### 🎯 一句话定义

dbt = **SQL-first transformation framework**，负责数据仓库里的 **T（Transform）** 那一步。

```
传统 ETL:   Extract → Transform (外部工具) → Load → DW
ELT + dbt:  Extract → Load (raw data) → Transform (dbt inside DW)
                                                ↑
                                        SQL + Jinja 模板
                                        版本控制 / 测试 / 文档 一体化
```

### 📊 dbt 做什么 vs 不做什么

| dbt 负责 | dbt 不负责 |
|----------|-----------|
| SQL 转换 (Transform) | 数据抽取 (Extract) |
| 模型依赖管理 (DAG) | 数据加载 (Load to DW) |
| 数据测试 (Testing) | 调度 (Scheduling) → 交给 Airflow |
| 文档生成 (Docs) | 流处理 (Streaming) |
| 版本控制 (Version Control) | 机器学习 (ML) |

### 📝 核心特性

```
1. SQL + Jinja2 模板  → 写普通 SQL，dbt 编译成目标方言
2. ref() 函数        → 声明模型依赖，自动构建 DAG
3. Git-native        → 所有模型都是 .sql 文件，PR review
4. 自动文档          → dbt docs generate + dbt docs serve
5. 内置测试          → dbt test 验证数据质量
```

### 💡 类比记忆

> dbt = 数据仓库里的 **Rails / Django** 🚂
> - 约定优于配置（Convention over Configuration）
> - SQL 文件 = Controller，ref() = URL 路由，tests = Unit Tests

### 🧠 记忆锚点

```
dbt = ELT 里的 T | SQL-first | Git-native | DAG + Test + Docs 一体化
不做: Extract / Load / Scheduling / Streaming
```

---

## Q2. dbt Models 分类

> 📌 **频率**: 2024-2025 · ★★☆  
> `Staging / Intermediate / Mart` · Layered Architecture

### 🎯 三层模型结构

```
sources (raw data in DW)
    ↓
┌─────────────────────────────────────────────┐
│  staging/          ← 1:1 mapping to source  │
│  stg_orders.sql    ← rename + cast + light  │
│  stg_customers.sql    clean, no business    │
│                       logic                 │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│  intermediate/     ← cross-source joins     │
│  int_order_items.sql  business logic lives  │
│                       here                  │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│  marts/            ← business-facing output │
│  finance/          ← organized by domain    │
│    fct_orders.sql  ← fact tables            │
│    dim_customers.sql ← dimension tables     │
└─────────────────────────────────────────────┘
```

### 📊 各层职责

| 层级 | 职责 | 物化方式 | 命名惯例 |
|------|------|----------|----------|
| **staging** | 1:1 源表映射，重命名，类型转换 | `view` | `stg_<source>__<table>` |
| **intermediate** | 跨源 JOIN，业务逻辑，中间计算 | `ephemeral` 或 `view` | `int_<entity>_<verb>` |
| **mart** | 面向业务的宽表/聚合表 | `table` 或 `incremental` | `fct_` / `dim_` |

### 📝 ref() 函数示例

```sql
-- models/marts/finance/fct_orders.sql
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    o.amount
FROM {{ ref('stg_orders') }} o          -- 引用 staging 模型
JOIN {{ ref('int_customers_enriched') }} c  -- 引用 intermediate 模型
    ON o.customer_id = c.customer_id
```

### 💡 类比记忆

> - **staging** = 食材清洗区 🥦（洗净切好，不加工）
> - **intermediate** = 备餐区 🔪（复杂切配、腌制）
> - **mart** = 出餐区 🍽️（面向客人的成品菜）

### 🧠 记忆锚点

```
staging=view(轻) → intermediate=ephemeral(中间态) → mart=table(重，对外)
ref() = 依赖声明 + DAG 自动构建
```

---

## Q3. dbt Tests

> 📌 **频率**: 2025 · ★★☆  
> `Generic Tests` vs `Singular Tests` · Data Quality in dbt

### 🎯 两类测试

| 类型 | 定义 | 配置位置 | 适用场景 |
|------|------|----------|----------|
| **Generic Tests** | 可复用的参数化测试 | `schema.yml` | 通用规则：唯一/非空/枚举/外键 |
| **Singular Tests** | 自定义 SQL 查询，返回行=失败 | `tests/` 目录下 `.sql` 文件 | 复杂业务逻辑校验 |

### 📝 Generic Tests 示例

```yaml
# models/schema.yml
models:
  - name: fct_orders
    columns:
      - name: order_id
        tests:
          - unique              # 内置：唯一性
          - not_null            # 内置：非空
      - name: status
        tests:
          - accepted_values:   # 内置：枚举校验
              values: ['placed', 'shipped', 'delivered', 'returned']
      - name: customer_id
        tests:
          - relationships:     # 内置：外键引用完整性
              to: ref('dim_customers')
              field: customer_id
```

### 📝 Singular Tests 示例

```sql
-- tests/assert_total_revenue_positive.sql
-- 返回行 > 0 则测试失败
SELECT order_id
FROM {{ ref('fct_orders') }}
WHERE amount <= 0
  AND status = 'delivered'
```

### 💡 类比记忆

> - **Generic Test** = 表单验证规则（必填/格式/枚举）📋 → 配置即可
> - **Singular Test** = 业务规则单元测试 🧪 → 写 SQL 断言

### 🧠 记忆锚点

```
Generic = not_null / unique / accepted_values / relationships → schema.yml 配置
Singular = 自定义 SQL → tests/ 目录 → 返回行=FAIL
dbt test 命令运行所有测试
```

---

## Q4. dbt Sources & Freshness

> 📌 **频率**: 2025 · ★★☆  
> `Sources` · `Freshness Check` · Raw Data Declaration

### 🎯 为什么需要 Sources

Sources = dbt 对**原始数据（raw data）的声明**，让 dbt 知道数据从哪来、有多新鲜。

```
没有 Sources:  SELECT * FROM raw.orders  ← 硬编码，无法追踪血缘
有了 Sources:  {{ source('raw', 'orders') }} ← dbt 追踪来源 + Freshness 检查
```

### 📝 Sources 配置示例

```yaml
# models/sources.yml
sources:
  - name: raw                           # source 组名
    database: my_warehouse
    schema: raw_data
    tables:
      - name: orders
        description: "Raw orders from Stripe"
        freshness:
          warn_after: {count: 6, period: hour}   # 6h 未更新 → 警告
          error_after: {count: 24, period: hour}  # 24h 未更新 → 错误
        loaded_at_field: _loaded_at              # 用哪个字段判断新鲜度
      - name: customers
        freshness:
          warn_after: {count: 1, period: day}
        loaded_at_field: updated_at
```

### 🛠️ Freshness 检查命令

```bash
dbt source freshness          # 检查所有 source 新鲜度
dbt source freshness --select source:raw.orders  # 检查单个
```

### 💡 类比记忆

> Sources = 超市进货单 📦
> - 声明原材料来源（哪个供应商）
> - Freshness = 保质期检查（超过 6 小时未到货就报警）

### 🧠 记忆锚点

```
source() 函数 = 声明原始表来源 + 血缘追踪
freshness = warn_after / error_after + loaded_at_field
dbt source freshness 命令触发检查
```

---

## Q5. dbt Lineage & DAG

> 📌 **频率**: 2025 · ★★☆  
> `Data Lineage` · `DAG Visualization` · `dbt docs`

### 🎯 dbt 如何构建 DAG

dbt 解析所有 `.sql` 文件中的 `ref()` 和 `source()` 调用，自动推导出**有向无环图（DAG）**。

```
source('raw', 'orders') ──┐
                           ├──► stg_orders ──┐
source('raw', 'customers')─┘                 ├──► int_order_enriched ──► fct_orders
                                             │
source('raw', 'customers')──► stg_customers─┘
```

### 📝 查看与使用 DAG

```bash
# 生成文档（含 lineage DAG）
dbt docs generate

# 启动本地文档服务（含交互式 DAG 图）
dbt docs serve           # 默认 http://localhost:8080

# 选择性运行：基于 DAG 的上下游
dbt run --select fct_orders+      # fct_orders 及所有下游
dbt run --select +fct_orders      # fct_orders 及所有上游
dbt run --select 1+fct_orders+1   # 直接上下游各一层
```

### 📊 DAG 的实用价值

| 场景 | 用法 |
|------|------|
| **影响分析 (Impact Analysis)** | 某 source 表改动 → 查哪些 mart 会受影响 |
| **CI/CD 选择性测试** | PR 只跑被改动模型及其下游 |
| **并行执行** | dbt 自动识别无依赖模型并行运行 |
| **数据血缘文档** | 自动生成列级别血缘（dbt + OpenLineage） |

### 💡 类比记忆

> dbt DAG = Airflow DAG 的数据版 🕸️
> - ref() = Airflow 的 depends_on
> - dbt docs = 自动生成的系统设计图

### 🧠 记忆锚点

```
ref() + source() → 自动构建 DAG → dbt docs serve 可视化
选择器: + 代表依赖方向 | --select model+ 选模型及下游
```

---

## Q6. dbt vs Stored Procedures

> 📌 **频率**: 面试必考对比题 · ★★★  
> `dbt` vs `Stored Procedures` · Modern Analytics Engineering

### 🎯 核心对比

| 维度 | Stored Procedures | dbt |
|------|-------------------|-----|
| **版本控制** | 难（存在 DB 里，不易 Git） | 天然 Git-native (.sql 文件) |
| **测试** | 无内置测试框架 | 内置 generic + singular tests |
| **文档** | 无自动文档 | `dbt docs generate` 自动生成 |
| **依赖管理** | 手动维护执行顺序 | ref() 自动 DAG + 并行执行 |
| **可移植性** | 强绑定数据库方言 | Jinja 模板 + adapter 多方言 |
| **可审查性** | 难以 Code Review | PR-based review + CI/CD |
| **调试** | 难以追踪问题 | Compiled SQL 可直接检查 |
| **幂等性** | 需手动保证 | 默认幂等（view/table 覆盖写） |

### 📝 同一逻辑的对比实现

```sql
-- ❌ Stored Procedure（黑盒、难测试、难版本控制）
CREATE PROCEDURE update_fct_orders()
BEGIN
  DELETE FROM fct_orders WHERE date = CURDATE();
  INSERT INTO fct_orders
  SELECT ... FROM raw.orders JOIN raw.customers ...;
END;

-- ✅ dbt Model（透明、可测试、可版本控制）
-- models/marts/fct_orders.sql
{{ config(materialized='incremental', unique_key='order_id') }}

SELECT
    o.order_id,
    c.customer_name,
    o.amount
FROM {{ ref('stg_orders') }} o
JOIN {{ ref('stg_customers') }} c ON o.customer_id = c.customer_id
{% if is_incremental() %}
WHERE o.order_date >= (SELECT MAX(order_date) FROM {{ this }})
{% endif %}
```

### ✅ 什么时候还用 Stored Procedures

- 复杂的 DML 事务（OLTP 场景）
- 数据库原生特性（如 PL/pgSQL 游标）
- 遗留系统维护成本过高

### 🧠 记忆锚点

```
SP = 黑盒 + DB 锁定 + 难测试 + 难 Git
dbt = 透明 + Git-native + 内置测试 + 自动 DAG + 文档
面试回答: "dbt 把 analytics engineering 带入了软件工程的最佳实践"
```

---

## Q7. dbt Incremental Models

> 📌 **频率**: 2025 · ★★★  
> `Incremental Materialization` · `unique_key` · `is_incremental()`

### 🎯 为什么需要 Incremental

```
Full Refresh（全量）: 每次重跑整张大表 → 慢 + 贵
Incremental:          只处理新增/变更数据 → 快 + 省
```

### 📝 Incremental Model 写法

```sql
-- models/marts/fct_orders.sql
{{ config(
    materialized='incremental',
    unique_key='order_id',           -- 用于 MERGE/UPDATE 去重
    incremental_strategy='merge'     -- merge / delete+insert / insert_overwrite
) }}

SELECT
    order_id,
    customer_id,
    amount,
    order_date,
    updated_at
FROM {{ ref('stg_orders') }}

{% if is_incremental() %}
-- 增量运行时只取新数据（比最新时间戳更新的）
WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
{% endif %}
```

### 📊 三种 Incremental Strategy

| 策略 | 原理 | 适用场景 |
|------|------|----------|
| **append** | 只 INSERT 新行 | 不可变事件流（无更新） |
| **merge** | MERGE (UPSERT) by unique_key | 有更新的维度/事实表 ✅ 推荐 |
| **delete+insert** | 先删 partition，再全插 | 分区表批量更新 |
| **insert_overwrite** | 覆盖整个分区 | BigQuery / Spark 大分区 |

### ⚠️ 常见陷阱

```
陷阱1: unique_key 列有 NULL → MERGE 行为异常
陷阱2: 漏设 is_incremental() 过滤 → 全量扫描源表，失去增量意义
陷阱3: Late-arriving data → 需扩大过滤窗口（如 3 天而非 1 天）
陷阱4: 第一次运行是全量，之后才增量 → dbt run --full-refresh 强制全量
```

### 💡 类比记忆

> Incremental = 每天只同步"今天新增的邮件" 📧
> - append = 只收新邮件
> - merge = 收新邮件 + 更新已有邮件的已读状态
> - delete+insert = 把今天的邮件箱清空再重新导入

### 🧠 记忆锚点

```
is_incremental() 控制过滤条件 | unique_key 控制 MERGE key
策略: append(不变) / merge(UPSERT) / delete+insert(分区覆盖)
--full-refresh 强制全量重跑
```
