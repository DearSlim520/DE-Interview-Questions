# 🐘 Hadoop

> HDFS / YARN / MapReduce 分布式基础设施 (Distributed Infrastructure)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [HDFS 读写流程](#q1-hdfs-读写流程) | ⭐⭐ |
| 2 | [HDFS NameNode HA 方案](#q2-hdfs-namenode-ha-方案) | ⭐⭐⭐ |
| 3 | [YARN 资源调度](#q3-yarn-资源调度) | ⭐⭐ |
| 4 | [MapReduce Shuffle 全过程](#q4-mapreduce-shuffle-全过程) | ⭐⭐⭐ |

---

## Q1. HDFS 读写流程

> 📌 **频率**: 2025 · ★★☆  
> `HDFS Read/Write Path`

### 🎯 写入流程 (Write Path)

```
1. Client → NameNode: 请求创建文件 (Create Request)
2. NameNode: 检查权限 + 创建元数据 → 返回 DataNode 列表 (Pipeline)
3. Client → DataNode Pipeline: DN1 → DN2 → DN3 (副本链式写入)
4. 数据按 Packet (64KB) 发送，每个 Block 默认 128MB
5. 所有副本确认 (ACK) → Client 通知 NameNode 完成
```

### 🎯 读取流程 (Read Path)

```
1. Client → NameNode: 获取文件 Block 位置 (Block Locations)
2. NameNode 返回各 Block 所在 DataNode（就近优先）
3. Client → 最近 DataNode: 直接读取 Block 数据
4. 如果节点故障 → 自动切换到其他副本
```

### 📊 关键参数

| 参数 | 默认值 | 含义 |
|------|--------|------|
| Block Size | 128MB | 数据切块大小 |
| Replication Factor | 3 | 副本数 |
| Rack Awareness | - | 副本分布在不同机架 (Anti-affinity) |

### 💡 类比记忆

> HDFS 写入 = 寄大型快递 📦
> - NameNode = 物流调度中心（告诉你该送哪 3 个仓库）
> - Pipeline = 快递接力（仓库1收到后转发给仓库2和3）
> - ACK = 三个仓库都签收了才算成功

### 🧠 记忆锚点

```
Write: Client → NameNode(元数据) → DataNode Pipeline(链式复制) → ACK
Read: Client → NameNode(Block位置) → 最近DataNode(直接读)
```

---

## Q2. HDFS NameNode HA 方案

> 📌 **频率**: 2025 · ★★☆  
> `NameNode High Availability`

### 🎯 HA 架构

```
┌─────────────────────────────────────────────────────┐
│  Active NameNode ◄──── 心跳 ────► Standby NameNode  │
│       ↕                                   ↕          │
│  JournalNode Cluster (QJM)                           │
│  (JN1, JN2, JN3) ← 共享 EditLog                     │
│       ↕                                   ↕          │
│  ZooKeeper (ZKFC 选主)                               │
└─────────────────────────────────────────────────────┘
```

### 📊 关键组件

| 组件 | English | 作用 |
|------|---------|------|
| Active NameNode | Primary Node | 处理所有读写请求 |
| Standby NameNode | Hot Standby | 实时同步元数据，随时接管 |
| JournalNode (QJM) | Quorum Journal Manager | 共享 EditLog 存储（多数派写入） |
| ZKFC | ZooKeeper Failover Controller | 监控 NameNode 健康 + 自动切换 |
| ZooKeeper | Leader Election | 选主 + Fencing（防止脑裂） |

### 🔄 Failover 流程

```
1. ZKFC 检测 Active NameNode 宕机
2. ZKFC 通过 ZooKeeper 竞选 Leader
3. 新 Leader ZKFC → Fencing 旧 Active（确保不会脑裂）
4. Standby → 升级为 Active
5. 从 JournalNode 同步最新 EditLog → 对外服务
```

### ⚠️ 脑裂 (Split-Brain) 防护

```
Fencing 机制:
  - SSH Fencing: SSH 到旧 Active 强制 kill 进程
  - Shell Fencing: 执行自定义脚本
  - 目的: 确保同一时刻只有一个 Active NameNode
```

### 💡 类比记忆

> NameNode HA = 双机长飞行 ✈️👨‍✈️👨‍✈️
> - Active = 主机长操作
> - Standby = 副机长随时准备接管
> - JournalNode = 黑匣子（双方都能写，保证日志一致）
> - ZKFC = 塔台监控（发现主机长失能立即切换）
> - Fencing = 确认主机长真的不行了（防误切）

### 🧠 记忆锚点

```
HA = Active/Standby + JournalNode(共享EditLog) + ZKFC(ZK选主) + Fencing(防脑裂)
```

---

## Q3. YARN 资源调度

> 📌 **频率**: 2025 · ★★☆  
> `YARN Resource Scheduling: Capacity vs Fair`

### 🎯 两种调度器对比

| 维度 | Capacity Scheduler | Fair Scheduler |
|------|-------------------|----------------|
| 设计理念 | 按队列分配固定容量 (Capacity Guarantee) | 所有作业公平共享 (Fair Share) |
| 资源分配 | 每个队列有最小/最大容量保证 | 动态均分，空闲时可抢占 |
| 弹性 | 队列可临时借用空闲容量 | 天然弹性，按需分配 |
| 抢占 (Preemption) | 支持（归还借用资源） | 支持（保证公平） |
| 适用场景 | 多租户、SLA 保证 | 交互式查询、小作业优先 |
| 默认 | Apache Hadoop 默认 | CDH 默认 |

### 📊 Capacity Scheduler 示例

```xml
root
├── production (60%)   ← 生产队列，保证 60% 资源
├── development (30%)  ← 开发队列
└── default (10%)      ← 默认队列

如果 production 空闲 → development 可临时借用
production 有作业 → 借的资源归还 (Elasticity)
```

### 💡 类比记忆

> - **Capacity** = 公司固定工位分配 🏢 — 每个部门有固定区域，空位可临时借
> - **Fair** = 共享会议室 🏫 — 没人用就归你，有人来了平分

### 🧠 记忆锚点

```
Capacity = 固定容量 + 多租户SLA + 弹性借用
Fair = 动态均分 + 抢占保证公平 + 小作业友好
```

---

## Q4. MapReduce Shuffle 全过程

> 📌 **频率**: 2025 · ★★☆  
> `MapReduce Shuffle Process`

### 🎯 完整流程

```
┌─── Map Side ──────────────────────────────────────┐
│  1. Map() 输出 → 写入环形缓冲区 (Ring Buffer, 100MB) │
│  2. 缓冲区 80% → Spill (溢写) 到磁盘               │
│     → 分区 (Partition by hash(key) % reduceNum)    │
│     → 排序 (Sort by key)                           │
│     → 可选 Combiner (本地预聚合)                    │
│  3. 多次 Spill → Merge 为一个有序文件               │
├─── Shuffle (网络传输) ────────────────────────────┤
│  4. Reducer 主动拉取 (Pull) 对应 Partition 的数据   │
├─── Reduce Side ───────────────────────────────────┤
│  5. 合并排序 (Merge Sort) 所有 Map 输出             │
│  6. 按 Key 分组 → 调用 Reduce() 函数               │
└───────────────────────────────────────────────────┘
```

### 📊 关键步骤

| 步骤 | English | 说明 |
|------|---------|------|
| Partition | 分区 | 决定 KV 去哪个 Reducer |
| Sort | 排序 | 按 Key 排序（MapReduce 保证 Key 有序） |
| Combiner | 本地聚合 | 减少网络传输量（可选） |
| Spill | 溢写 | 缓冲区满时写磁盘 |
| Merge | 合并 | 多个 Spill 文件归并为一个 |
| Copy/Fetch | 拉取 | Reducer 从各 Mapper 拉取数据 |

### 💡 类比记忆

> Shuffle = 考试收卷分发 📝
> - Map Side = 每个考场（Map）把试卷按科目分好、排序、打包
> - Shuffle = 送卷车把各考场的对应科目卷子送到阅卷组
> - Reduce Side = 阅卷组收到所有卷子，合并排序后开始批改

### 🧠 记忆锚点

```
Map: Buffer → Spill(Partition+Sort+Combiner) → Merge
Shuffle: Reducer Pull from Mappers
Reduce: Merge Sort → Group by Key → Reduce()
```
