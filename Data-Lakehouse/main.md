# 🏔️ Data Lakehouse

> Iceberg / Paimon 湖仓一体 (Lakehouse)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Iceberg metadata 三层结构](#q1-iceberg-metadata-三层结构) | ⭐⭐⭐ |
| 2 | [Paimon 与 Iceberg 区别](#q2-paimon-与-iceberg-区别) | ⭐⭐⭐ |

---

## Q1. Iceberg metadata 三层结构

> 📌 **频率**: 2025 H2 密集出现 · ★☆☆ · 中高级岗  
> `Apache Iceberg Metadata Architecture`

### 🎯 核心要点

Iceberg 元数据分 **三层套娃结构 (Three-layer Metadata)**：

```
┌─────────────────────────────────────────────────────┐
│  metadata.json  (Table-level)                        │
│  ├── Schema / Partition Spec / Properties           │
│  └── Snapshot 列表 (指向 Manifest List)              │
├─────────────────────────────────────────────────────┤
│  Manifest List  (每个 Snapshot 一个)                 │
│  └── 列出该 Snapshot 所属的 Manifest Files           │
├─────────────────────────────────────────────────────┤
│  Manifest File  (实际数据清单)                       │
│  └── 列出数据文件路径 + Column-level Stats           │
│      (min/max/null_count per column)                │
└─────────────────────────────────────────────────────┘
```

### 💡 类比记忆

> 想象你拍照备份，每天一个相册 (Snapshot)：
> - **metadata.json** = 相册总目录 (Master Catalog)
> - **Manifest List** = 每个相册的封面页，写着「今天有 A、B、C 三个袋子」
> - **Manifest File** = 每个袋子的清单，写着「file1.parquet (min_id=1, max_id=100)...」

### ✅ 设计收益 (Design Benefits)

| 能力 | 原理 |
|------|------|
| **Data Pruning（查询剪枝）** | `WHERE id=50` → 读 Manifest 中的 min/max 统计 → 跳过不相关文件 |
| **Time Travel（时间旅行）** | 切换 metadata.json 指向旧 Snapshot，O(1) 操作 |
| **ACID Transactions** | 原子替换 (Atomic Swap) metadata.json 指针即可 |

### 🧠 记忆锚点

```
Three layers: metadata.json → Manifest List → Manifest File
三层套娃 = Master Catalog / Snapshot Index / Data File Registry
```

---

## Q2. Paimon 与 Iceberg 区别

> 📌 **频率**: 2025 新兴题 · ★★☆  
> `Apache Paimon vs Apache Iceberg`

### 🎯 核心区别

| 维度 | Apache Iceberg | Apache Paimon |
|------|---------------|---------------|
| 定位 | **Table Format**（表格式标准） | **Streaming Lakehouse**（流式湖仓存储） |
| 设计理念 | 静态数据管理 + 批处理优先 | 流批一体 (Stream-Batch Unified) |
| 实时写入 | Append 为主，Update 需 COW/MOR | 原生 Streaming Sink (Changelog) |
| Changelog 支持 | 需额外组件 | **原生支持** (+I/-D/-U/+U) |
| 主键更新 (PK Update) | Copy-on-Write / Merge-on-Read | 原生 LSM-tree 结构，天生支持 |
| Compaction | Rewrite 整个文件 | LSM Compaction（增量合并） |
| 生态 | Spark / Flink / Trino / Presto 广泛 | Flink 深度绑定，Spark 支持中 |
| 成熟度 | 成熟（Netflix → Apache 顶级项目） | 新兴（2023 年 Apache 孵化） |

### 📊 场景选型

```
选 Iceberg:
  ├── 批处理为主 (Batch-first)
  ├── 需要广泛引擎兼容 (Multi-engine)
  ├── 历史数据管理 + Time Travel
  └── 数据湖标准化

选 Paimon:
  ├── 流式写入 + 实时更新 (Streaming Write + Real-time Update)
  ├── CDC 入湖（需要 Changelog 语义）
  ├── Flink 深度集成场景
  └── 替代 Hudi 的 MOR 场景
```

### 💡 类比记忆

> - **Iceberg** = 图书馆管理系统 📚 — 书（数据文件）放好后分类管理，偶尔改改目录
> - **Paimon** = 实时新闻编辑室 📰 — 稿件（数据）不断更新、撤回、修改，需要实时生效

### 📝 技术细节

| 特性 | Iceberg 实现 | Paimon 实现 |
|------|-------------|-------------|
| 写入模式 | Append / COW / MOR | LSM-tree + Changelog |
| 读时合并 (Merge-on-Read) | Positional Delete + Read Merge | Native LSM Merge |
| Snapshot 隔离 | ✅ 通过 metadata.json 版本 | ✅ 通过 Snapshot 管理 |
| 分区演进 (Partition Evolution) | ✅ 无需重写数据 | ✅ 支持 |
| 流式消费 (Streaming Read) | 需 Incremental Read API | **原生 Changelog Stream** |

### 🧠 记忆锚点

```
Iceberg = Table Format + Batch-first + 广泛生态
Paimon = Streaming Lakehouse + LSM + Changelog Native + Flink 深绑
```
