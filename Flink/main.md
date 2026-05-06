# ⚡ Flink

> Apache Flink 实时流处理引擎

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Flink CDC 2.x 无锁全量+增量切换](#q1-flink-cdc-2x-无锁全量增量切换) | ⭐⭐⭐ |
| 2 | Flink 双流 join 的几种方式（interval/window/regular） | - |
| 3 | Flink Checkpoint 与 Savepoint 区别 | - |
| 4 | Flink 反压定位与处理 | - |
| 5 | Flink State Backend 选型（RocksDB vs HashMap） | - |
| 6 | Flink Watermark 乱序处理 | - |
| 7 | Exactly-Once 端到端实现（2PC sink） | - |
| 8 | Flink SQL vs DataStream 取舍 | - |
| 9 | Flink 双流 join 的状态膨胀治理 | - |

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
