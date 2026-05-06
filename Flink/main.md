# ⚡ Flink

> Apache Flink 实时流处理引擎

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Flink CDC 2.x 无锁全量+增量切换](#q1-flink-cdc-2x-无锁全量增量切换) | ⭐⭐⭐ |
| 2 | [Flink 双流 join 的几种方式](#q2-flink-双流-join-的几种方式) | ⭐⭐⭐ |
| 3 | [Flink Checkpoint 与 Savepoint 区别](#q3-flink-checkpoint-与-savepoint-区别) | ⭐⭐ |
| 4 | [Flink 反压定位与处理](#q4-flink-反压定位与处理) | ⭐⭐⭐ |
| 5 | [Flink State Backend 选型](#q5-flink-state-backend-选型) | ⭐⭐ |
| 6 | [Flink Watermark 乱序处理](#q6-flink-watermark-乱序处理) | ⭐⭐⭐ |
| 7 | [Exactly-Once 端到端实现](#q7-exactly-once-端到端实现) | ⭐⭐⭐ |
| 8 | [Flink SQL vs DataStream 取舍](#q8-flink-sql-vs-datastream-取舍) | ⭐⭐ |
| 9 | [Flink 双流 join 的状态膨胀治理](#q9-flink-双流-join-的状态膨胀治理) | ⭐⭐⭐ |

---

## Q1. Flink CDC 2.x 无锁全量+增量切换

> 📌 **频率**: 2025 起常考 · ★☆☆

### 🎯 核心要点

基于 **Netflix DBLog 算法**：

```
┌────────────────────────────────────────────────┐
│           Full Snapshot Phase (无锁)            │
│                                                │
│  Table ──► chunk 切分（按主键范围）              │
│                                                │
│  每个 chunk:                                    │
│    1. 记录高水位 binlog pos                     │
│    2. SELECT * FROM chunk WHERE pk BETWEEN ...  │
│    3. 记录低水位 binlog pos                     │
│    4. 用 [低,高] 之间的 binlog 修正 SELECT 结果  │
├────────────────────────────────────────────────┤
│        Incremental Phase (增量切换)             │
│                                                │
│  所有 chunk 完成后 → 从最大水位继续消费 binlog   │
│  ✅ Exactly-Once 保证                          │
└────────────────────────────────────────────────┘
```

### 💡 类比记忆

> 好比给整个图书馆盘点（全量），同时图书馆还在借书还书（binlog）：
> - **1.x** = 锁住全馆盘点 🔒
> - **2.x** = 把书架按编号切块，每块「快门式」记下盘点前后的借还流水，最后用流水修正盘点结果 📷

**无需** `FLUSH TABLES WITH READ LOCK`，对线上零影响。

### ✅ 关键优势

| 对比项 | CDC 1.x | CDC 2.x |
|--------|---------|---------|
| 全量阶段 | 全局锁 | 无锁 chunk 并行 |
| 一致性 | 锁保证 | 高低水位修正 |
| 对线上影响 | 大（锁表） | 零 |
| 并行度 | 单线程 | 多 chunk 并行 |

### 🧠 记忆锚点

```
chunk + 高低水位修正 = 盘点不锁库
```

---

## Q2. Flink 双流 join 的几种方式

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 三种 Join 方式

```
┌─────────────────────────────────────────────────────────┐
│  Regular Join (常规 Join)                                │
│  ├── 两侧状态无限增长                                    │
│  ├── 适合：维表关联（配合 TTL）                          │
│  └── SELECT * FROM A JOIN B ON A.id = B.id              │
├─────────────────────────────────────────────────────────┤
│  Interval Join (区间 Join)                               │
│  ├── 基于时间范围匹配，状态自动清理                       │
│  ├── 适合：订单 + 支付（30分钟内匹配）                   │
│  └── A.ts BETWEEN B.ts - INTERVAL '30' MIN AND B.ts    │
├─────────────────────────────────────────────────────────┤
│  Window Join (窗口 Join)                                 │
│  ├── 在同一窗口内做 join，窗口结束即释放                  │
│  ├── 适合：固定周期聚合后关联                            │
│  └── 两流必须落在同一窗口才能 join                       │
└─────────────────────────────────────────────────────────┘
```

### 📊 详细对比

| 维度 | Regular Join | Interval Join | Window Join |
|------|-------------|---------------|-------------|
| 状态大小 | 无限增长 ⚠️ | 自动过期清理 | 窗口结束即清 |
| 时间语义 | 无要求 | 需要事件时间 | 需要窗口对齐 |
| 延迟 | 来一条算一条 | 来一条算一条 | 等窗口关闭 |
| 适用场景 | 维表 Join | 订单-支付匹配 | 固定周期报表 |
| 状态 TTL | 需手动设置 | 自动按区间 | 窗口生命周期 |

### 💡 类比记忆

> - **Regular** = 两个无限长的记事本，每来一条都翻另一本找匹配 📒📒
> - **Interval** = 快递柜，超过 30 分钟没人取就自动清走 📦⏰
> - **Window** = 考试交卷，铃响前交的才算同一批 🔔

### 🧠 记忆锚点

```
Regular=无限状态 | Interval=时间范围自清 | Window=批次对齐
```

---

## Q3. Flink Checkpoint 与 Savepoint 区别

> 📌 **频率**: 2025 基础必考 · ★★☆

### 🎯 核心对比

| 维度 | Checkpoint | Savepoint |
|------|-----------|-----------|
| 触发方式 | **自动**（周期触发） | **手动**（命令触发） |
| 目的 | 故障恢复 | 版本升级 / 代码变更 / 集群迁移 |
| 生命周期 | 作业取消后默认删除 | 永久保留，需手动清理 |
| 格式依赖 | 可能依赖 State Backend 格式 | 标准化格式，跨版本兼容 |
| 性能影响 | 轻量，增量快照 | 全量快照，较重 |
| 恢复方式 | 自动从最近一次恢复 | 指定路径手动恢复 |

### 💡 类比记忆

> - **Checkpoint** = 游戏自动存档 🎮（死了自动从上个存档点恢复）
> - **Savepoint** = 游戏手动存档 💾（想换装备/升级版本前主动存）

### 📝 常用命令

```bash
# 触发 Savepoint
flink savepoint <jobId> s3://bucket/savepoints/

# 从 Savepoint 恢复
flink run -s s3://bucket/savepoints/savepoint-xxx ...

# Checkpoint 配置
env.enableCheckpointing(60000)  # 每 60s
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)
```

### 🧠 记忆锚点

```
Checkpoint = 自动存档防崩溃 | Savepoint = 手动存档做升级
```

---

## Q4. Flink 反压定位与处理

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 反压原理

```
┌───────┐    ┌───────┐    ┌───────┐
│Source │───►│ Map   │───►│ Sink  │  ← Sink 处理慢
└───────┘    └───────┘    └───────┘
                              ▲
                              │ 反压传导方向
                              │ (下游慢 → 上游阻塞)
```

**本质**：下游处理能力 < 上游发送速率 → 缓冲区满 → 逐级反压

### 🔍 定位三板斧

| 步骤 | 工具 | 看什么 |
|------|------|--------|
| 1️⃣ 找瓶颈 | Flink Web UI | `backPressured` 状态的算子（红色） |
| 2️⃣ 看指标 | Metrics | `inputQueueLength` / `outputQueueLength` |
| 3️⃣ 定根因 | Thread Dump / 火焰图 | 算子内部是 CPU 密集 or IO 阻塞 |

### 🛠️ 解决方案

| 根因 | 解决方案 |
|------|----------|
| **算子计算慢** | 增加并行度 / 优化逻辑 |
| **外部 IO 阻塞** | 异步 IO（AsyncFunction）/ 批量写 |
| **数据倾斜** | keyBy 加盐打散 / rebalance |
| **GC 频繁** | 增大 TaskManager 内存 / 用 RocksDB |
| **Checkpoint 太大** | 增量 Checkpoint / 减小状态 |
| **Sink 写不动** | 攒批写入 / 增加 Sink 并行度 |

### 💡 类比记忆

> 反压 = 高速公路堵车 🚗 — 前面收费站（Sink）太慢 → 车辆排队往后堵 → 一直堵到入口（Source）。
> 解法：加收费窗口（增并行度）/ ETC（异步IO）/ 分流（rebalance）

### 🧠 记忆锚点

```
定位：Web UI 找红色 → Metrics 看队列 → 火焰图看根因
处理：加并行度 / 异步IO / 加盐打散 / 攒批写
```

---

## Q5. Flink State Backend 选型

> 📌 **频率**: 2025 基础 · ★★☆

### 🎯 两大 State Backend

| 维度 | HashMapStateBackend | RocksDBStateBackend |
|------|--------------------|--------------------|
| 存储位置 | **JVM 堆内存** | **本地磁盘**（RocksDB） |
| 访问速度 | 极快（ns 级） | 较慢（需序列化 + 磁盘IO） |
| 容量上限 | 受 JVM 内存限制 | 仅受磁盘大小限制 |
| GC 影响 | 有（大状态 GC 严重） | 无（堆外存储） |
| 增量 Checkpoint | ❌ 不支持 | ✅ 支持 |
| 适用场景 | 状态小 + 延迟敏感 | 状态大 + 稳定优先 |

### 📊 选型决策树

```
状态大小?
├── < 几百 MB → HashMapStateBackend (快!)
└── > 几 GB → RocksDBStateBackend (稳!)
         │
         └── 需要增量 Checkpoint? → 必须 RocksDB
```

### 💡 类比记忆

> - **HashMap** = 把所有东西放桌面上 🖥️ — 拿取快，但桌子就那么大
> - **RocksDB** = 放柜子抽屉里 🗄️ — 开抽屉慢一点，但容量无限

### 🧠 记忆锚点

```
HashMap = 堆内快但小 | RocksDB = 磁盘慢但大 + 增量 CP
```

---

## Q6. Flink Watermark 乱序处理

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 核心概念

```
事件时间轴:  1  2  3  [5]  4  6  7  [8]  ...
                       ↑        ↑
               Watermark(5-3=2) Watermark(8-3=5)
               表示 ≤2 的数据已到齐
               
乱序容忍度 = maxOutOfOrderness = 3s
Watermark = 当前最大事件时间 - 乱序容忍度
```

**含义**：Watermark(t) 表示「不会再有时间戳 ≤ t 的数据到来」

### 🔄 完整流程

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  数据流入    │────►│ 生成 Watermark│────►│ 窗口触发计算 │
│ (带事件时间) │     │ (周期/标点式) │     │ 当 WM ≥ 窗口│
└─────────────┘     └──────────────┘     │  end time   │
                                          └─────────────┘
                                                │
                                          迟到数据?
                                          ├── allowedLateness → 更新结果
                                          └── sideOutput → 侧输出兜底
```

### 📝 三道防线

| 防线 | 机制 | 说明 |
|------|------|------|
| 1️⃣ | Watermark 容忍度 | 允许一定程度乱序 |
| 2️⃣ | allowedLateness | 窗口关闭后仍接受迟到数据 |
| 3️⃣ | sideOutput | 超过所有容忍的数据输出到侧流 |

### 💡 类比记忆

> Watermark = 考试交卷时间 ⏰
> - 容忍度 = 延迟 5 分钟交卷（乱序容忍）
> - allowedLateness = 迟到 10 分钟还能补交但扣分（窗口更新）
> - sideOutput = 迟到 1 小时的直接去教务处（单独处理）

### 🧠 记忆锚点

```
WM = maxEventTime - 乱序度 | 三道防线：容忍/迟到/侧输出
```

---

## Q7. Exactly-Once 端到端实现

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 端到端 Exactly-Once 三个环节

```
┌──────────┐      ┌───────────┐      ┌──────────┐
│  Source  │─────►│  Flink    │─────►│   Sink   │
│(可重放)  │      │(Checkpoint)│      │(2PC/幂等)│
└──────────┘      └───────────┘      └──────────┘
     ↑                  ↑                  ↑
  可回溯消费      Barrier 对齐          预提交+确认
  (Kafka offset)  (状态一致快照)       (事务/幂等)
```

### 📊 各环节保证

| 环节 | 机制 | 示例 |
|------|------|------|
| **Source** | 可重放 + 记录偏移量 | Kafka offset 存入 State |
| **内部** | Checkpoint barrier 对齐 | 保证状态一致性快照 |
| **Sink** | 2PC 事务 或 幂等写入 | Kafka 2PC / MySQL 幂等 |

### 🔄 2PC Sink 流程

```
1. Checkpoint 触发 → Sink 开启事务 → 预提交(preCommit)
2. 所有算子 Checkpoint 完成 → JM 通知确认
3. Sink 收到通知 → 正式提交(commit)
4. 如果失败 → 回滚(abort)
```

### 💡 类比记忆

> 端到端 Exactly-Once = 银行转账 💰
> - Source（可重放）= ATM 可以重新查账
> - 内部（Checkpoint）= 转账记录本一致
> - Sink（2PC）= 收款方确认收到才算成功，否则退回

### ⚠️ 注意事项

- 2PC 会增加延迟（需等 Checkpoint 完成才提交）
- Kafka Sink：需设 `transaction.timeout.ms` > Checkpoint 间隔
- 幂等替代：如果下游支持 upsert（如 MySQL ON DUPLICATE KEY），可用幂等代替 2PC

### 🧠 记忆锚点

```
三环节：Source可重放 + 内部Checkpoint + Sink 2PC/幂等
```

---

## Q8. Flink SQL vs DataStream 取舍

> 📌 **频率**: 2025 · ★★☆

### 🎯 对比一览

| 维度 | Flink SQL | DataStream API |
|------|-----------|---------------|
| 开发效率 | ⭐⭐⭐ 声明式，简洁 | ⭐ 命令式，代码量大 |
| 灵活度 | 受 SQL 语义限制 | 完全自由 |
| 优化 | 自动优化（Calcite） | 手动优化 |
| 状态管理 | 隐式（框架管理） | 显式（自定义 State） |
| 适合场景 | ETL / 聚合 / Join | 复杂事件处理 / 自定义窗口 |
| 学习门槛 | 低（会 SQL 就行） | 高（需理解流处理模型） |

### 📊 选型决策

```
你的需求是?
├── 标准 ETL / 过滤 / 聚合 / Join → ✅ Flink SQL
├── 需要自定义 State / Timer / 复杂 CEP → ✅ DataStream
├── 快速迭代 + 业务方自助 → ✅ Flink SQL
└── 两者混用 → SQL 做主逻辑 + UDF 做复杂计算
```

### 💡 类比记忆

> - **Flink SQL** = 自动挡汽车 🚗 — 好开，覆盖 80% 场景
> - **DataStream** = 手动挡赛车 🏎️ — 灵活极致，但门槛高

### 🧠 记忆锚点

```
SQL = 声明式+自动优化+简单 | DataStream = 命令式+灵活+复杂
```

---

## Q9. Flink 双流 join 的状态膨胀治理

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 问题根因

Regular Join 两侧状态无限增长 → OOM / Checkpoint 超时

```
Stream A ──┐                    State A: 无限增长 📈
           ├──► Regular Join ──►
Stream B ──┘                    State B: 无限增长 📈
```

### 🛠️ 治理方案

| 方案 | 原理 | 适用场景 |
|------|------|----------|
| **设置 State TTL** | 状态超时自动清理 | 业务容忍丢失旧匹配 |
| **改 Interval Join** | 限定时间范围匹配 | 有明确时间关联 |
| **改 Window Join** | 窗口内 join | 批次化处理 |
| **异步维表 Join** | 一侧当维表 lookup | 大小表关联 |
| **预聚合降维** | 先 group by 减少记录 | 聚合后再 join |

### 📝 State TTL 配置

```java
// SQL 方式
tableEnv.getConfig().setIdleStateRetention(Duration.ofHours(24));

// DataStream 方式
StateTtlConfig ttl = StateTtlConfig.newBuilder(Time.hours(24))
    .setUpdateType(UpdateType.OnCreateAndWrite)
    .setStateVisibility(NeverReturnExpired)
    .build();
```

### 💡 类比记忆

> 状态膨胀 = 家里从不扔东西 🏠📦📦📦 → 总有一天放不下
> 治理 = 定期断舍离（TTL）/ 只留近期的（Interval）/ 按季度整理（Window）

### ⚠️ 注意

- TTL 设短了 → 可能丢匹配（数据不准）
- TTL 设长了 → 状态依然大（资源浪费）
- 最佳实践：结合业务 SLA，订单场景一般 24h-72h TTL

### 🧠 记忆锚点

```
TTL兜底 | Interval替代 | 预聚合降维 | 业务容忍度决定TTL时长
```
