# 🐝 Hive

> Hive 数据仓库引擎 & 存储格式

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Hive 小文件治理](#q1-hive-小文件治理) | ⭐⭐ |
| 2 | [Hive 数据倾斜](#q2-hive-数据倾斜) | ⭐⭐⭐ |
| 3 | [Hive 分区 vs 分桶](#q3-hive-分区-vs-分桶) | ⭐⭐ |
| 4 | [Hive on Tez vs MR vs Spark](#q4-hive-on-tez-vs-mr-vs-spark) | ⭐⭐ |
| 5 | [Hive UDF/UDAF/UDTF 区别与场景](#q5-hive-udfudafudtf-区别与场景) | ⭐⭐ |
| 6 | [ORC vs Parquet](#q6-orc-vs-parquet) | ⭐⭐ |
| 7 | [Hive 索引与 Bloom Filter](#q7-hive-索引与-bloom-filter) | ⭐⭐ |

---

## Q1. Hive 小文件治理

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 为什么小文件有害？

```
大量小文件 → NameNode 内存压力（每个文件 ~150B 元数据）
            → Map Task 数量爆炸（1 文件 = 1 Map Task）
            → HDFS 读取效率低（频繁寻址）
```

### 🛠️ 治理方案

| 方案 | 阶段 | 原理 |
|------|------|------|
| **CombineHiveInputFormat** | 读取时 | 多个小文件合并为一个 split 给一个 Map |
| **Merge 小文件** | 写入后 | `hive.merge.mapfiles=true` / `hive.merge.mapredfiles=true` |
| **INSERT OVERWRITE** | 重写 | 定期重写分区，合并为大文件 |
| **HAR 归档** | 存储层 | `ALTER TABLE ARCHIVE PARTITION` 打包为 HAR |
| **动态分区控制** | 写入时 | 限制分区数 / `DISTRIBUTE BY` 控制 Reducer 数 |
| **Concatenate** | 命令 | `ALTER TABLE t CONCATENATE` (仅 ORC/RCFile) |

### 📝 常用配置

```sql
-- 合并 Map 输出的小文件
SET hive.merge.mapfiles = true;
SET hive.merge.mapredfiles = true;
SET hive.merge.size.per.task = 256000000;   -- 256MB
SET hive.merge.smallfiles.avgsize = 16000000; -- 16MB 以下触发合并

-- 读取时合并
SET hive.input.format = org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
```

### 💡 类比记忆

> 小文件 = 搬家时打包了 1000 个小塑料袋 🛍️ → 搬运工（NameNode）记不住每个袋子
> 治理 = 把小袋子装进大纸箱（Merge / CombineInputFormat）

### 🧠 记忆锚点

```
读时合并(CombineInputFormat) + 写后Merge + 定期重写 + 控制分区数
```

---

## Q2. Hive 数据倾斜

> 📌 **频率**: 2025 超高频 · ★★★

### 🎯 三种倾斜场景 + 方案

| 场景 | 原因 | 解决方案 |
|------|------|----------|
| **GROUP BY 倾斜** | 某个 key 数据量极大 | `hive.groupby.skewindata=true`（两阶段聚合） |
| **JOIN 倾斜** | 大表 join 时热 key | Map Join / Skew Join / 加盐 |
| **COUNT DISTINCT 倾斜** | 去重时某个值特别多 | 改写为 `GROUP BY + COUNT` |

### 📝 具体方案

**GROUP BY 倾斜：**
```sql
-- 开启两阶段聚合（自动加随机前缀 → 局部聚合 → 全局聚合）
SET hive.groupby.skewindata = true;
```

**JOIN 倾斜：**
```sql
-- 方案1: Map Join（小表 < 25MB）
SET hive.auto.convert.join = true;
SET hive.mapjoin.smalltable.filesize = 25000000;

-- 方案2: Skew Join（自动识别热key单独处理）
SET hive.optimize.skewjoin = true;
SET hive.skewjoin.key = 100000;  -- 超过10w条认为倾斜

-- 方案3: 手动加盐
SELECT /*+ MAPJOIN(b) */ ...
```

**COUNT DISTINCT 倾斜：**
```sql
-- 原始（倾斜）
SELECT COUNT(DISTINCT user_id) FROM t;

-- 优化（先 GROUP BY 去重，再 COUNT）
SELECT COUNT(1) FROM (SELECT user_id FROM t GROUP BY user_id) tmp;
```

### 💡 类比记忆

> 数据倾斜 = 班级值日 🧹 — 一个同学分到整栋楼，其他人分到一间教室
> 解法：两阶段 = 先各人扫各人，再统一倒垃圾 | Map Join = 小桌子搬到各教室

### 🧠 记忆锚点

```
GroupBy→两阶段 | Join→MapJoin/SkewJoin/加盐 | Distinct→改GroupBy+Count
```

---

## Q3. Hive 分区 vs 分桶

> 📌 **频率**: 2025 基础 · ★★☆

### 🎯 对比

| 维度 | Partition 分区 | Bucket 分桶 |
|------|--------------|-------------|
| 物理形式 | HDFS 子目录 | 文件（按 hash 分） |
| 作用 | 按业务维度裁剪（如日期） | 数据均匀打散 + 高效采样 |
| 裁剪方式 | WHERE 条件匹配 → 跳过不相关目录 | Hash 匹配 → 跳过不相关文件 |
| 典型列 | 日期 / 地区 / 业务线 | 用户 ID / 订单 ID |
| 数量级 | 几百～几千分区 | 通常 32/64/128/256 桶 |
| 优势场景 | 按日期查询 | Bucket Map Join / 数据采样 |

### 📝 示例

```sql
-- 分区表
CREATE TABLE orders (
  order_id BIGINT, user_id BIGINT, amount DECIMAL
) PARTITIONED BY (dt STRING)
STORED AS ORC;

-- 分桶表
CREATE TABLE user_behavior (
  user_id BIGINT, action STRING, ts TIMESTAMP  
) CLUSTERED BY (user_id) INTO 128 BUCKETS
STORED AS ORC;
```

### ✅ Bucket Map Join 条件

两表都按同一列分桶 + 桶数成倍数关系 → 可以桶对桶 join，无需全量 Shuffle

### 💡 类比记忆

> - **分区** = 图书馆按楼层分区（小说区/历史区）→ 找小说只去 3 楼 📚
> - **分桶** = 每个区域内按编号分柜子 → 找 ID=123 的书直接去 123%128 号柜 🗄️

### 🧠 记忆锚点

```
分区=目录裁剪(WHERE) | 分桶=Hash均匀分布(Join优化+采样)
```

---

## Q4. Hive on Tez vs MR vs Spark

> 📌 **频率**: 2025 · ★★☆

### 🎯 三种引擎对比

| 维度 | MapReduce | Tez | Spark |
|------|-----------|-----|-------|
| 执行模型 | Map → Reduce（严格两阶段） | DAG（多阶段无需落盘） | DAG + 内存计算 |
| 中间结果 | 写 HDFS | 可 pipeline，减少 IO | 内存中缓存 |
| 延迟 | 高（启动 JVM 慢） | 中 | 低（常驻进程） |
| 容错 | 强（中间结果在 HDFS） | 中 | 中（RDD lineage） |
| 资源模型 | 每次申请释放 Container | Container 复用 | Executor 常驻 |
| 适用场景 | 超大批量、稳定性优先 | Hive 默认推荐 | 交互查询、迭代计算 |

### 📊 性能关系

```
MapReduce < Tez ≈ 3-5x 提升 < Spark ≈ 10-100x 提升 (迭代场景)
```

### 💡 类比记忆

> - **MR** = 每次出差都重新订机票酒店（重量级）✈️
> - **Tez** = 出差一次连着办多件事，中间不回家（Container 复用）🚄
> - **Spark** = 直接在目的地常驻办公（Executor 常驻）🏢

### 🧠 记忆锚点

```
MR=两阶段落盘 | Tez=DAG+Container复用 | Spark=DAG+内存+常驻
```

---

## Q5. Hive UDF/UDAF/UDTF 区别与场景

> 📌 **频率**: 2025 · ★★☆

### 🎯 三种自定义函数

| 类型 | 全称 | 输入→输出 | 示例 |
|------|------|-----------|------|
| **UDF** | User Defined Function | 一行 → 一行 (1:1) | `UPPER()`, `CONCAT()` |
| **UDAF** | User Defined Aggregate Function | 多行 → 一行 (N:1) | `SUM()`, `AVG()`, `COUNT()` |
| **UDTF** | User Defined Table-generating Function | 一行 → 多行 (1:N) | `EXPLODE()`, `LATERAL VIEW` |

### 📊 对比图

```
UDF:    [row] ──► [row]           1进1出
UDAF:   [row1, row2, ...rowN] ──► [result]    N进1出
UDTF:   [row] ──► [row1, row2, ...rowN]       1进N出
```

### 📝 使用场景

| 场景 | 用哪个 |
|------|--------|
| 字符串清洗 / 格式化 | UDF |
| 自定义聚合（如分位数） | UDAF |
| JSON 数组展开 / 行转多行 | UDTF + LATERAL VIEW |

### 💡 类比记忆

> - **UDF** = 翻译器 📝（一句话进，一句话出）
> - **UDAF** = 统计员 📊（收集所有试卷，算平均分）
> - **UDTF** = 拆包员 📦→📦📦📦（一个包裹拆成多个小件）

### 🧠 记忆锚点

```
UDF=1:1 | UDAF=N:1 | UDTF=1:N
```

---

## Q6. ORC vs Parquet

> 📌 **频率**: 2025 · ★★☆

### 🎯 对比

| 维度 | ORC | Parquet |
|------|-----|---------|
| 生态 | Hive 原生首选 | Spark / Iceberg / Flink 首选 |
| 压缩 | ZLIB（压缩比更高） | Snappy（速度更快） |
| 索引 | 内置 Bloom Filter + 行组统计 | 列统计 min/max + Page 级 |
| 嵌套类型 | 支持但不如 Parquet | 原生支持（Dremel 编码） |
| ACID | 原生支持（Hive ACID） | 不直接支持（需 Iceberg/Delta） |
| 更新能力 | 支持行级 update/delete | 需配合 Lakehouse 格式 |
| 社区 | Apache Hive 为主 | 更广泛（Spark/Presto/Flink） |

### 📊 选型建议

```
用 ORC: ← Hive 生态 + 需要 ACID + 高压缩比
用 Parquet: ← Spark/Flink/Iceberg + 嵌套数据 + 跨引擎兼容
```

### 💡 类比记忆

> - **ORC** = Hive 的"亲儿子" 👶 — 在 Hive 里性能最佳，ACID 内置
> - **Parquet** = "国际通用" 🌍 — 哪个引擎都支持，嵌套数据强

### 🧠 记忆锚点

```
ORC=Hive亲生+ACID+高压缩 | Parquet=通用+嵌套+Spark首选
```

---

## Q7. Hive 索引与 Bloom Filter

> 📌 **频率**: 2025 · ★★☆

### 🎯 Hive 索引演进

```
Hive 0.7: 引入索引（Bitmap Index / Compact Index）
Hive 3.0: 移除旧索引 ❌（性能收益不明显 + 维护成本高）
当前推荐: ORC 内置统计 + Bloom Filter 取代传统索引
```

### 📊 ORC 文件内置优化

| 机制 | 层级 | 效果 |
|------|------|------|
| **File Footer 统计** | 文件级 | 整个文件的 min/max/count |
| **Stripe 统计** | Stripe 级 (~250MB) | 跳过不相关 Stripe |
| **Row Group 统计** | Row Group 级 (~10000行) | 细粒度跳过 |
| **Bloom Filter** | 列级 | 精确判断某值「一定不在」 |

### 📝 Bloom Filter 配置

```sql
-- 建表时指定 Bloom Filter 列
CREATE TABLE orders (
  order_id BIGINT, user_id BIGINT, status STRING
) STORED AS ORC
TBLPROPERTIES (
  'orc.bloom.filter.columns' = 'order_id,user_id',
  'orc.bloom.filter.fpp' = '0.01'  -- 误报率 1%
);
```

### 💡 Bloom Filter 原理

```
写入时: value → N 个 Hash 函数 → 设置 Bit Array 对应位
查询时: WHERE order_id = 123
  → Hash(123) → 检查 Bit Array
  → 全为 1 → 可能存在（继续读）
  → 有 0   → 一定不存在（跳过该 Stripe）✅
```

### 💡 类比记忆

> Bloom Filter = 门卫名单 🚪
> - "名单上没有你" → 你一定进不去（精确排除）
> - "名单上有你" → 不一定真有（可能误报，还要再核实）

### 🧠 记忆锚点

```
旧索引已废弃 → ORC内置统计(min/max) + Bloom Filter(精确排除)
```
