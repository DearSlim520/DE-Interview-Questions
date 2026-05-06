# 📐 Data Modeling

> 维度建模 (Dimensional Modeling) / 数仓分层 (DW Layering) / 模型设计

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [维度建模 vs 范式建模 vs Data Vault](#q1-维度建模-vs-范式建模-vs-data-vault) | ⭐⭐⭐ |
| 2 | [拉链表设计与 SCD Type 2](#q2-拉链表设计与-scd-type-2) | ⭐⭐⭐ |
| 3 | [一致性维度](#q3-一致性维度) | ⭐⭐ |
| 4 | [缓慢变化维 6 种类型](#q4-缓慢变化维-6-种类型) | ⭐⭐⭐ |
| 5 | [事实表三种类型](#q5-事实表三种类型) | ⭐⭐ |
| 6 | [雪花模型 vs 星型模型](#q6-雪花模型-vs-星型模型) | ⭐⭐ |
| 7 | [数仓分层职责](#q7-数仓分层职责) | ⭐⭐⭐ |
| 8 | [用户画像标签宽表设计](#q8-用户画像标签宽表设计) | ⭐⭐ |

---

## Q1. 维度建模 vs 范式建模 vs Data Vault

> 📌 **频率**: 2025 高频 · ★★☆  
> `Dimensional Modeling vs 3NF vs Data Vault`

### 🎯 三种建模方法

| 维度 | Dimensional Modeling | 3NF (范式建模) | Data Vault |
|------|---------------------|----------------|-----------|
| 代表人物 | Kimball | Inmon | Dan Linstedt |
| 设计理念 | 面向分析 (Analysis-first) | 面向存储 (Storage-first) | 面向集成 (Integration-first) |
| 冗余 | 允许冗余 (Denormalized) | 消除冗余 (Normalized) | 适度冗余 |
| 核心结构 | 事实表 + 维度表 (Fact + Dim) | 实体关系 (ER) | Hub + Link + Satellite |
| 查询性能 | 快（少 JOIN） | 慢（多 JOIN） | 中等 |
| 适用 | BI / OLAP / 数仓 | ODS / 数据整合 | 企业级数据仓库 |
| 灵活性 | 低（维度固定） | 中 | 高（易扩展） |

### 💡 类比记忆

> - **Kimball (Dimensional)** = 超市货架 🛒 — 按用户购买习惯摆放，方便拿取
> - **Inmon (3NF)** = 仓库管理 📦 — 按类别严格分类，不重复摆放
> - **Data Vault** = 乐高积木 🧱 — Hub(核心)+ Link(关系)+ Satellite(属性)，随时拼接

### 🧠 记忆锚点

```
Kimball = Star Schema + Bottom-Up + Analysis-first
Inmon = 3NF + Top-Down + Storage-first  
Data Vault = Hub/Link/Satellite + Integration-first + Agile
```

---

## Q2. 拉链表设计与 SCD Type 2

> 📌 **频率**: 2025 高频 · ★★★  
> `Slowly Changing Dimension Type 2` / `Zipper Table`

### 🎯 核心思想

拉链表 (Zipper Table) = SCD Type 2 的实现方式，保留维度的**历史版本**。

```
┌────────────────────────────────────────────────────┐
│  user_id │ name   │ city     │ start_date │ end_date│
│──────────│────────│──────────│────────────│─────────│
│  1001    │ 张三   │ 北京     │ 2024-01-01 │ 2024-06-30│ ← 旧版本
│  1001    │ 张三   │ 上海     │ 2024-07-01 │ 9999-12-31│ ← 当前版本
└────────────────────────────────────────────────────┘
```

### 📝 更新流程

```sql
-- 1. 关闭旧记录 (Close old record)
UPDATE dim_user 
SET end_date = '2024-06-30'
WHERE user_id = 1001 AND end_date = '9999-12-31';

-- 2. 插入新记录 (Insert new version)
INSERT INTO dim_user VALUES (1001, '张三', '上海', '2024-07-01', '9999-12-31');
```

### ✅ 查询方式

```sql
-- 查当前有效 (Current State)
SELECT * FROM dim_user WHERE end_date = '9999-12-31';

-- 查历史某天状态 (Point-in-Time)
SELECT * FROM dim_user 
WHERE start_date <= '2024-05-15' AND end_date >= '2024-05-15';
```

### 💡 类比记忆

> 拉链表 = 员工档案 📂
> - 每次调岗/搬迁不是删掉旧记录，而是关闭旧的、新开一条
> - 想查"2024年5月时他在哪个部门" → 查日期区间匹配的那条

### 🧠 记忆锚点

```
SCD Type 2 = 新增行 + start_date/end_date 标记生效区间
当前态: end_date = '9999-12-31' | 历史态: 日期范围查询
```

---

## Q3. 一致性维度

> 📌 **频率**: 2025 · ★★☆  
> `Conformed Dimension`

### 🎯 定义

一致性维度 (Conformed Dimension) = 在多个事实表 / 多个业务域中**共享的、定义统一的维度表**。

### 📝 示例

```
dim_date (日期维度) → 被 fact_orders, fact_clicks, fact_payments 共同使用
dim_user (用户维度) → 被订单、行为、支付等多个 Fact Table 引用

要求:
  - 相同的粒度 (Grain)
  - 相同的主键 (PK)
  - 相同的属性定义 (Attribute Definition)
```

### ⚠️ 不一致的后果

```
如果 订单域的"城市"字段 和 支付域的"城市"字段 定义不同:
  → 跨域分析时口径不一致 (Cross-domain Inconsistency)
  → 无法做 Drill-across（跨事实表关联分析）
```

### 💡 类比记忆

> Conformed Dimension = 全公司统一的工号系统 🪪
> - 所有部门用同一套工号 → 跨部门统计不会对不上人

### 🧠 记忆锚点

```
Conformed Dimension = 跨业务域共享 + 统一定义 + 支持 Drill-across
```

---

## Q4. 缓慢变化维 6 种类型

> 📌 **频率**: 2025 · ★★☆  
> `Slowly Changing Dimension (SCD) Types`

### 🎯 6 种 SCD 类型

| Type | English | 策略 | 保留历史? |
|------|---------|------|-----------|
| Type 0 | Retain Original | 不变，保留最初值 | ❌ |
| Type 1 | Overwrite | 直接覆盖旧值 | ❌ |
| Type 2 | Add New Row | 新增行 + 生效/失效日期 | ✅ 完整历史 |
| Type 3 | Add New Column | 加"上一个值"列 | ✅ 仅上一次 |
| Type 4 | Mini-Dimension | 变化频繁的列拆出来单独表 | ✅ |
| Type 6 | Hybrid (1+2+3) | 同时有当前值、历史行、前一次列 | ✅ |

### 📊 最常用

```
Type 1 = 不在意历史（覆盖）→ 简单
Type 2 = 完整历史（拉链表）→ 最常用
Type 3 = 只保留上一次（加列）→ 轻量级
```

### 💡 类比记忆

> - Type 1 = 橡皮擦掉重写（没有历史）
> - Type 2 = 翻新页写新的，旧的保留（完整历史）📖
> - Type 3 = 在旁边标注"原来是xxx"（只记上次）

### 🧠 记忆锚点

```
面试答 Type 1(覆盖) + Type 2(拉链表) + Type 3(加列) 即可
Type 2 最重要 = start_date + end_date + is_current
```

---

## Q5. 事实表三种类型

> 📌 **频率**: 2025 · ★★☆  
> `Three Types of Fact Tables`

### 🎯 三种类型

| 类型 | English | 粒度 | 示例 |
|------|---------|------|------|
| 事务事实表 | **Transaction Fact** | 每笔交易一行 | 订单明细表 |
| 周期快照事实表 | **Periodic Snapshot Fact** | 固定周期一行 | 月度库存快照 |
| 累积快照事实表 | **Accumulating Snapshot Fact** | 一个业务实体一行（多次更新） | 订单生命周期表 |

### 📊 对比

| 维度 | Transaction | Periodic Snapshot | Accumulating Snapshot |
|------|-------------|-------------------|----------------------|
| 粒度 | 一事件一行 | 一周期一行 | 一实体一行 |
| 更新 | Insert Only | Insert Only | 多次 Update |
| 时间列 | 事件时间 | 快照时间 | 多个里程碑时间 |
| 示例 | 每笔支付 | 每日账户余额 | 订单(下单时间/付款时间/发货时间/完成时间) |

### 📝 累积快照示例

```sql
-- 订单生命周期 (Accumulating Snapshot)
order_id | order_time | pay_time | ship_time | deliver_time | status
1001     | 2025-01-01 | 2025-01-01 | 2025-01-02 | 2025-01-05  | delivered
1002     | 2025-01-02 | 2025-01-02 | NULL       | NULL         | shipped
```

### 💡 类比记忆

> - **Transaction** = 银行流水 💳 — 每笔交易一行
> - **Periodic Snapshot** = 月度体检报告 🏥 — 每月一次快照
> - **Accumulating Snapshot** = 快递追踪 📦 — 一个包裹一行，不断更新状态

### 🧠 记忆锚点

```
Transaction = 每事件一行(Insert Only)
Periodic Snapshot = 每周期一行(定期快照)
Accumulating Snapshot = 每实体一行(多里程碑Update)
```

---

## Q6. 雪花模型 vs 星型模型

> 📌 **频率**: 2025 · ★★☆  
> `Star Schema vs Snowflake Schema`

### 🎯 对比

| 维度 | Star Schema (星型) | Snowflake Schema (雪花) |
|------|-------------------|------------------------|
| 维度表 | 扁平化 (Denormalized) | 规范化 (Normalized)，有子维度 |
| JOIN 数 | 少（Fact + Dim 一层） | 多（Dim 还要 JOIN 子 Dim） |
| 查询性能 | 快 ✅ | 较慢 |
| 冗余 | 有冗余 | 消除冗余 |
| ETL 复杂度 | 高（维度宽表打平） | 低（直接按关系存） |
| 适用 | OLAP / BI 查询 | 维度数据量大 + 要节省存储 |

### 📊 图示

```
Star Schema:
  Fact ──► dim_product (product_id, name, category_name, brand_name)
  
Snowflake Schema:
  Fact ──► dim_product (product_id, name, category_id, brand_id)
                                        ↓              ↓
                              dim_category        dim_brand
```

### 💡 类比记忆

> - **Star Schema** = 一张名片写完所有信息 📇（读取快但冗余）
> - **Snowflake** = 名片 + 附件 + 附件的附件 📎（存储省但读取多次查）

### 🧠 记忆锚点

```
Star = Denormalized Dim + 少JOIN + 快 + OLAP首选
Snowflake = Normalized Dim + 多JOIN + 省空间 + 大维度表时考虑
```

---

## Q7. 数仓分层职责

> 📌 **频率**: 2025 超高频 · ★★★  
> `Data Warehouse Layering: ODS/DWD/DWS/DIM/ADS`

### 🎯 五层架构

| 层级 | English | 职责 | 数据特征 |
|------|---------|------|----------|
| **ODS** | Operational Data Store | 原始数据落地，不做清洗 | 1:1 还原源系统 |
| **DWD** | Detail Warehouse Detail | 数据清洗 (Cleansing) + 标准化 (Standardization) | 明细粒度 |
| **DWS** | Service Warehouse Summary | 轻度聚合 (Light Aggregation) | 主题宽表 |
| **DIM** | Dimension Layer | 维度管理 (SCD / 码值映射) | 维度表 |
| **ADS** | Application Data Service | 直接服务报表/API | 高度聚合 |

### 📊 数据流向

```
源系统 → ODS(原始) → DWD(清洗明细) → DWS(轻度聚合) → ADS(报表)
                              ↑
                          DIM(维度)
```

### 📝 各层典型操作

| 层级 | 典型操作 |
|------|----------|
| ODS → DWD | 去重 (Dedup)、过滤脏数据、字段标准化、类型转换 |
| DWD → DWS | 按主题关联维度、打宽表 (Wide Table)、轻度聚合 |
| DWS → ADS | 按需求聚合、计算指标、产出报表数据 |

### 💡 类比记忆

> 数仓分层 = 食品加工厂 🏭
> - ODS = 原材料仓库（刚到货的蔬菜，不处理）
> - DWD = 分拣清洗车间（洗净、切好、分类）
> - DWS = 半成品仓库（预制菜，轻度加工）
> - ADS = 成品出货（直接上餐桌的菜品）
> - DIM = 调料仓库（被各环节引用）

### 🧠 记忆锚点

```
ODS(原始1:1) → DWD(清洗明细) → DWS(轻度聚合宽表) → ADS(报表服务)
DIM = 维度表独立管理 (SCD)
```

---

## Q8. 用户画像标签宽表设计

> 📌 **频率**: 2025 · ★★☆  
> `User Profile / Tag Wide Table Design`

### 🎯 设计思路

```
用户画像 (User Profile) = 给每个用户贴标签 (Tags)
标签宽表 (Tag Wide Table) = 一行一用户，列 = 各种标签
```

### 📊 标签分类

| 标签类型 | English | 示例 | 计算方式 |
|----------|---------|------|----------|
| 基础属性 | **Static Tag** | 性别、年龄段、注册时间 | 直接取源表 |
| 统计标签 | **Statistical Tag** | 月消费金额、活跃天数 | SQL 聚合 |
| 规则标签 | **Rule-based Tag** | 高价值用户、流失风险 | IF-ELSE 规则 |
| 模型标签 | **ML-based Tag** | 购买倾向、兴趣偏好 | 机器学习模型预测 |

### 📝 宽表结构

```sql
CREATE TABLE dws_user_profile (
  user_id         BIGINT,
  -- 基础属性 (Static)
  gender          STRING,
  age_group       STRING,
  register_days   INT,
  -- 统计标签 (Statistical)
  order_cnt_30d   INT,
  pay_amount_30d  DECIMAL,
  active_days_7d  INT,
  -- 规则标签 (Rule-based)
  is_vip          BOOLEAN,
  churn_risk      STRING,    -- 'high'/'medium'/'low'
  -- 模型标签 (ML-based)  
  purchase_prob   DOUBLE,
  interest_tags   ARRAY<STRING>
) PARTITIONED BY (dt STRING);
```

### ⚠️ 设计注意事项

| 问题 | 解决方案 |
|------|----------|
| 标签太多列爆炸 | KV 结构存储（tag_name, tag_value） |
| 更新频率不一 | 按更新频率分表（日更/周更/实时） |
| 标签依赖复杂 | DAG 管理标签间依赖 |
| 存储膨胀 | 只存有值标签 / 稀疏存储 |

### 💡 类比记忆

> 用户画像 = 人物角色卡 🎮
> - Static Tag = 固有属性（种族、职业）
> - Statistical Tag = 统计数据（战力值、等级）
> - Rule Tag = 称号（"黄金会员"、"战斗达人"）
> - ML Tag = AI 预测（下次购买概率）

### 🧠 记忆锚点

```
标签分类: Static + Statistical + Rule-based + ML-based
存储: 宽表(列多) 或 KV表(灵活) | 按更新频率分层
```
