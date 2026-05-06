# 🔥 Spark

> Apache Spark 批/流处理引擎

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Spark AQE 三大特性](#q1-spark-aqe-三大特性) | ⭐⭐⭐ |
| 2 | [Spark 数据倾斜定位与方案](#q2-spark-数据倾斜定位与方案) | ⭐⭐⭐ |
| 3 | [Spark Shuffle 演进](#q3-spark-shuffle-演进) | ⭐⭐ |
| 4 | [RDD vs DataFrame vs DataSet](#q4-rdd-vs-dataframe-vs-dataset) | ⭐⭐ |
| 5 | [Spark 内存模型](#q5-spark-内存模型) | ⭐⭐⭐ |
| 6 | [Spark Stage 划分与 DAG](#q6-spark-stage-划分与-dag) | ⭐⭐ |
| 7 | [Spark Catalyst 优化器原理](#q7-spark-catalyst-优化器原理) | ⭐⭐⭐ |

---

## Q1. Spark AQE 三大特性

> 📌 **频率**: 2025 高频 · ★★☆  
> `Adaptive Query Execution` · Spark 3.0+

### 🎯 三大特性

```
┌─── AQE (Adaptive Query Execution) ───────────────────┐
│                                                       │
│  1️⃣ Coalesce Shuffle Partitions                      │
│     运行时自动合并过小的 shuffle partition             │
│     (之前要手动设 spark.sql.shuffle.partitions=200)   │
│                                                       │
│  2️⃣ Switch Join Strategy                             │
│     运行时根据实际数据量切换 Join 策略                 │
│     (本来 SortMergeJoin → 发现小表 → 自动切 BHJ)     │
│                                                       │
│  3️⃣ Optimize Skew Join                               │
│     检测到倾斜 partition 自动拆分 + 副本复制            │
│     (大 key 拆成多份，小表侧复制对应份数)             │
└───────────────────────────────────────────────────────┘
```

### 📊 对比

| 特性 | 之前（静态） | AQE（动态） |
|------|-------------|-------------|
| Partition 数 | 写死 200 | 运行时合并小分区 |
| Join 策略 | 按统计信息决定 | 按实际 shuffle 后大小决定 |
| 数据倾斜 | 手动加盐 | 自动拆分 skew partition |

### 💡 类比记忆

> AQE = 导航实时避堵 🗺️
> - 合并小分区 = 高速公路车道动态合并
> - 切换 Join = 发现前方不堵了改走高速（BroadcastHashJoin）
> - Skew 优化 = 堵车车道自动分流到多条辅路

### 🧠 记忆锚点

```
AQE 三板斧：合并小分区 + 动态切Join + 自动治倾斜
```

---

## Q2. Spark 数据倾斜定位与方案

> 📌 **频率**: 2025 超高频 · ★★★

### 🔍 定位方法

| 方法 | 操作 |
|------|------|
| Spark UI | 看 Stage 里各 Task 的执行时间，长尾 Task = 倾斜 |
| Shuffle Read | 看每个 Task 读取的数据量是否严重不均 |
| Key 分布 | `df.groupBy("key").count().orderBy(desc("count"))` |

### 🛠️ 解决方案

| 方案 | 原理 | 适用场景 |
|------|------|----------|
| **加盐（Salting）** | key 前缀加随机数 → 打散 → 聚合两次 | GroupBy 倾斜 |
| **Broadcast Join** | 小表广播到每个 Executor | 大小表 Join（小表 < 10MB） |
| **两阶段聚合** | 先局部聚合（加盐 + reduceByKey）→ 再全局聚合 | Count/Sum 类 |
| **Skew Hint** | `/*+ SKEW('table', 'col') */` | Spark 3.0+ SQL |
| **AQE 自动** | `spark.sql.adaptive.skewJoin.enabled=true` | 通用 |
| **过滤热 Key** | 单独处理 null / 热 key | 少数极端 key |

### 📝 加盐示例

```python
# 阶段1：加盐局部聚合
df_salted = df.withColumn("salted_key", concat(col("key"), lit("_"), (rand()*10).cast("int")))
partial = df_salted.groupBy("salted_key").agg(sum("value").alias("partial_sum"))

# 阶段2：去盐全局聚合  
result = partial.withColumn("key", split(col("salted_key"), "_")[0]) \
               .groupBy("key").agg(sum("partial_sum").alias("total"))
```

### 💡 类比记忆

> 数据倾斜 = 超市只开一个收银台 🏪 → 大量顾客堆在一个窗口
> 解法：加盐 = 发随机号分到不同窗口 | Broadcast = VIP 通道直接过

### 🧠 记忆锚点

```
定位：Spark UI 长尾 Task | 方案：加盐/BHJ/两阶段/AQE
```

---

## Q3. Spark Shuffle 演进

> 📌 **频率**: 2025 · ★★☆  
> `Hash Shuffle → Sort Shuffle → Tungsten Sort`

### 🎯 演进历程

```
Spark 0.x ─── Hash Shuffle
  │           每个 Map Task 为每个 Reduce 生成一个文件
  │           文件数 = M × R（爆炸💥）
  ▼
Spark 1.2 ── Sort Shuffle (默认)
  │           每个 Map Task 只输出一个排序文件 + 索引
  │           文件数 = M（大幅减少）
  ▼
Spark 1.5+ ── Tungsten Sort (Unsafe Shuffle)
              堆外内存 + 二进制排序
              避免 Java 对象开销 + 减少 GC
```

### 📊 对比

| 维度 | Hash Shuffle | Sort Shuffle | Tungsten Sort |
|------|-------------|-------------|---------------|
| 文件数 | M × R | M | M |
| 排序 | 无 | 有 | 有（二进制） |
| 内存效率 | 低 | 中 | 高（堆外） |
| 适用条件 | - | 通用 | 无聚合 + 分区数 < 16M |
| GC 压力 | 大 | 中 | 小 |

### 💡 类比记忆

> - **Hash** = 每个同学直接把作业扔到对应科目的筐里（筐太多桌子放不下）
> - **Sort** = 每个同学先把所有作业排好序装一个信封（信封数量可控）
> - **Tungsten** = 用机器自动分拣（二进制操作，不经人手）

### 🧠 记忆锚点

```
Hash(M×R文件) → Sort(M文件+排序) → Tungsten(堆外+二进制)
```

---

## Q4. RDD vs DataFrame vs DataSet

> 📌 **频率**: 2025 基础 · ★★☆

### 🎯 三者对比

| 维度 | RDD | DataFrame | DataSet |
|------|-----|-----------|---------|
| 类型安全 | ✅ 编译时 | ❌ 运行时 | ✅ 编译时 |
| 优化 | ❌ 无 Catalyst | ✅ Catalyst + Tungsten | ✅ Catalyst + Tungsten |
| API 风格 | 函数式 | SQL-like / 声明式 | 函数式 + SQL |
| 序列化 | Java Serialization | Tungsten 二进制 | Tungsten + Encoder |
| Schema | 无 | 有（StructType） | 有（Case Class） |
| 语言支持 | Java/Scala/Python | Java/Scala/Python/R | Java/Scala only |

### 📊 演进关系

```
RDD (Spark 1.0)
 └──► DataFrame (Spark 1.3) = RDD + Schema + Catalyst
       └──► DataSet (Spark 1.6) = DataFrame + 类型安全
             └──► Spark 2.0: DataFrame = DataSet[Row]
```

### 💡 何时用什么

| 场景 | 推荐 |
|------|------|
| 结构化数据 ETL | DataFrame / Spark SQL |
| 需要编译时类型检查 | DataSet (Scala) |
| 非结构化 / 底层操作 | RDD |
| Python 用户 | DataFrame（无 DataSet） |

### 🧠 记忆锚点

```
RDD=底层无优化 | DataFrame=有Schema有优化 | DataSet=DataFrame+类型安全
```

---

## Q5. Spark 内存模型

> 📌 **频率**: 2025 · ★★☆  
> `Unified Memory Management` (Spark 1.6+)

### 🎯 统一内存管理

```
┌─────────────────────────────────────────────┐
│          Executor Memory                     │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─── Unified Memory (60%) ───────────────┐ │
│  │                                         │ │
│  │  Storage Memory ◄══► Execution Memory   │ │
│  │  (缓存RDD/广播)      (Shuffle/Join/聚合) │ │
│  │                                         │ │
│  │     可互相借用，动态调整边界 ↕️           │ │
│  └─────────────────────────────────────────┘ │
│                                             │
│  ┌─── Reserved Memory (300MB) ────────────┐ │
│  │  系统预留，不可动                        │ │
│  └─────────────────────────────────────────┘ │
│                                             │
│  ┌─── User Memory (40%) ──────────────────┐ │
│  │  用户数据结构 / UDF 变量                 │ │
│  └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### 📊 关键参数

| 参数 | 默认值 | 含义 |
|------|--------|------|
| `spark.memory.fraction` | 0.6 | Unified Memory 占比 |
| `spark.memory.storageFraction` | 0.5 | Storage 初始占比（可被 Execution 借用） |

### ✅ 统一内存 vs 静态内存

| 维度 | 静态划分（旧） | 统一内存（新） |
|------|---------------|---------------|
| 边界 | Storage/Execution 固定 | 动态借用 |
| 利用率 | 一侧空闲另一侧 OOM | 互相借用，利用率高 |
| 淘汰策略 | - | Execution 需要时可驱逐 Storage |

### 💡 类比记忆

> 统一内存 = 合租房共享客厅 🏠
> - Storage = 你的储物柜（可以被室友临时借用）
> - Execution = 室友的工作桌（忙的时候可以占你的柜子空间）
> - 但 Execution 优先级更高（计算不能 OOM，缓存可以丢）

### 🧠 记忆锚点

```
Unified = Storage + Execution 动态借用 | Execution 优先级更高
```

---

## Q6. Spark Stage 划分与 DAG

> 📌 **频率**: 2025 基础 · ★★☆

### 🎯 核心规则

```
DAG (Directed Acyclic Graph) 划分规则：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
遇到 Shuffle 依赖(Wide Dependency) → 切分新 Stage

Narrow Dependency (窄依赖): map, filter, union
  → 同一 Stage 内，可 pipeline 执行
  
Wide Dependency (宽依赖): groupByKey, reduceByKey, join
  → 需要 Shuffle → 产生新 Stage 边界
```

### 📊 示例

```
rdd.map(f)           ← Stage 1 (窄依赖, pipeline)
   .filter(g)        ← Stage 1
   .groupByKey()     ← ✂️ Shuffle → Stage 2 开始
   .map(h)           ← Stage 2
   .reduceByKey(+)   ← ✂️ Shuffle → Stage 3 开始
```

### 🔄 执行流程

```
User Code → DAG → DAGScheduler → TaskScheduler → Executor
                      │
                      ├── 按 Shuffle 切 Stage
                      ├── Stage 内切 Task (= partition 数)
                      └── Stage 间有依赖顺序
```

### 💡 类比记忆

> Stage = 工厂流水线的工站 🏭
> - 窄依赖 = 同一工站内的工序（物料不需要离开工站）
> - 宽依赖 = 需要把半成品送到下个工站重新分配（Shuffle = 物流）

### 🧠 记忆锚点

```
宽依赖切Stage | Stage内pipeline | Task数=partition数
```

---

## Q7. Spark Catalyst 优化器原理

> 📌 **频率**: 2025 · ★★☆

### 🎯 四阶段优化流程

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Analysis │───►│ Logical  │───►│ Physical │───►│Code Gen  │
│          │    │Optimizing│    │ Planning │    │(Tungsten)│
└──────────┘    └──────────┘    └──────────┘    └──────────┘
 解析 SQL/DF     逻辑优化         物理计划选择     代码生成
 绑定 Catalog    谓词下推等       选择 Join 策略   Whole-Stage
                 列裁剪等         选择 Scan 方式   CodeGen
```

### 📊 各阶段详解

| 阶段 | 输入 | 输出 | 关键优化 |
|------|------|------|----------|
| **Analysis** | Unresolved Plan | Resolved Plan | 解析表名/列名，绑定 Catalog |
| **Logical Optimization** | Logical Plan | Optimized Plan | Predicate Pushdown, Column Pruning, Constant Folding |
| **Physical Planning** | Optimized Plan | Physical Plan | Join 策略选择（BHJ/SMJ/SHJ）、Cost-Based |
| **Code Generation** | Physical Plan | Java Bytecode | Whole-Stage CodeGen, 消除虚函数调用 |

### 📝 常见逻辑优化规则

| 规则 | 效果 |
|------|------|
| **Predicate Pushdown** | WHERE 条件推到数据源 → 减少扫描量 |
| **Column Pruning** | 只读需要的列 → 减少 IO |
| **Constant Folding** | `1+1` 编译时直接算成 `2` |
| **Combine Filters** | 多个 filter 合并为一个 |
| **Null Propagation** | 涉及 null 的表达式直接简化 |

### 💡 类比记忆

> Catalyst = 编译器优化 🔧
> - Analysis = 语法检查（这个变量存在吗？）
> - Logical Opt = 代码重构（把没用的循环删掉）
> - Physical Plan = 选算法（这里用快排还是归并）
> - CodeGen = 编译成机器码（极致性能）

### 🧠 记忆锚点

```
Analysis → Logical Opt → Physical Plan → CodeGen
谓词下推 + 列裁剪 + Join策略 + Whole-Stage CodeGen
```
