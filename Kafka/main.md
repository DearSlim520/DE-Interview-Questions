# 📨 Kafka

> Apache Kafka 消息队列 (Message Queue) / 流平台 (Streaming Platform)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Kafka 高吞吐原因](#q1-kafka-高吞吐原因) | ⭐⭐⭐ |
| 2 | [Kafka Exactly-Once 与幂等 Producer](#q2-kafka-exactly-once-与幂等-producer) | ⭐⭐⭐ |
| 3 | [Kafka 分区分配策略](#q3-kafka-分区分配策略) | ⭐⭐ |
| 4 | [Kafka ISR 机制](#q4-kafka-isr-机制) | ⭐⭐⭐ |

---

## Q1. Kafka 高吞吐原因

> 📌 **频率**: 2025 高频 · ★★☆

### 🎯 四大核心技术

| 技术 | English | 原理 |
|------|---------|------|
| 顺序写 | **Sequential I/O** | 磁盘顺序写 ≈ 内存随机读写速度 (600MB/s+) |
| 零拷贝 | **Zero-Copy (sendfile)** | 数据从磁盘 → 网卡，不经过用户态 (User Space) |
| 批处理 | **Batching** | Producer 攒批发送 + Consumer 批量拉取 |
| 分区 | **Partitioning** | 水平扩展 (Horizontal Scaling)，并行生产消费 |

### 📊 零拷贝 (Zero-Copy) 详解

```
传统方式 (4次拷贝):
  Disk → Kernel Buffer → User Buffer → Socket Buffer → NIC

Zero-Copy (2次拷贝):
  Disk → Kernel Buffer ─────────────────────────► NIC
                    (sendfile syscall, 跳过 User Space)
```

### 📝 其他优化

| 优化 | 说明 |
|------|------|
| Page Cache | 利用 OS Page Cache 做读缓存 |
| Compression | 支持 Snappy/LZ4/ZSTD 压缩 |
| Append-Only Log | 只追加不修改，无随机写 |
| Index 文件 | 稀疏索引 (Sparse Index) 快速定位 |

### 💡 类比记忆

> Kafka 高吞吐 = 高速公路系统 🛣️
> - Sequential I/O = 单向多车道，不掉头
> - Zero-Copy = ETC 不停车直接通过
> - Batching = 大货车一次拉满
> - Partitioning = 多条高速并行

### 🧠 记忆锚点

```
四大法宝: Sequential I/O + Zero-Copy + Batching + Partitioning
```

---

## Q2. Kafka Exactly-Once 与幂等 Producer

> 📌 **频率**: 2025 · ★★☆  
> `Exactly-Once Semantics (EOS)` + `Idempotent Producer`

### 🎯 三种语义

| 语义 | English | 保证 |
|------|---------|------|
| 至多一次 | **At-Most-Once** | 可能丢消息 |
| 至少一次 | **At-Least-Once** | 可能重复 |
| 精确一次 | **Exactly-Once** | 不丢不重 |

### 📊 幂等 Producer (Idempotent Producer)

```
开启: enable.idempotence = true

原理:
  Producer 每个 Partition 维护:
  - PID (Producer ID)
  - Sequence Number (单调递增)
  
  Broker 检测: 
  - seq == expected → 正常写入
  - seq < expected → 重复，丢弃 (Dedup)
  - seq > expected → 乱序/丢失，报错
```

**幂等的局限**：仅保证单 Partition + 单 Session 的 Exactly-Once

### 📊 事务 Producer (Transactional Producer)

```
开启: transactional.id = "my-tx-id"

能力: 跨 Partition + 跨 Session 的 Exactly-Once

流程:
  1. producer.initTransactions()
  2. producer.beginTransaction()
  3. producer.send(record1); producer.send(record2);
  4. producer.commitTransaction()  // 或 abortTransaction()
```

### 💡 类比记忆

> - **At-Most-Once** = 发微信不管已读（可能没收到）
> - **At-Least-Once** = 发短信 + 电话确认（可能重复告知）
> - **Exactly-Once** = 银行转账（转账单号去重 + 事务保证）

### 🧠 记忆锚点

```
Idempotent = PID + SeqNum → 单Partition去重
Transactional = 跨Partition原子写 → 端到端 Exactly-Once
```

---

## Q3. Kafka 分区分配策略

> 📌 **频率**: 2025 · ★★☆  
> `Partition Assignment Strategy`

### 🎯 三种策略

| 策略 | English | 原理 | 特点 |
|------|---------|------|------|
| Range | **RangeAssignor** | 按 Topic 的 Partition 范围均分 | 可能不均匀 (多 Topic 时前面 Consumer 多分) |
| RoundRobin | **RoundRobinAssignor** | 所有 Partition 轮询分配 | 均匀，但 Rebalance 时大量变动 |
| Sticky | **StickyAssignor** | 尽量保持上次分配 + 均匀 | **推荐** — 减少 Rebalance 开销 |

### 📝 示例（2 Consumer, 3 Partition）

```
Range:        C0 → [P0, P1]    C1 → [P2]          不均匀!
RoundRobin:   C0 → [P0, P2]    C1 → [P1]          均匀
Sticky:       C0 → [P0, P2]    C1 → [P1]          均匀 + Rebalance 少迁移
```

### ⚠️ Rebalance 问题

```
Rebalance = Consumer 加入/退出时重新分配 Partition
  → 期间所有 Consumer 暂停消费 (Stop-the-World) ⚠️
  → Sticky 策略可以最小化迁移量
```

### 💡 类比记忆

> - **Range** = 按座位号分配（1-5号给A组，6-10号给B组）→ 可能前面组多分
> - **RoundRobin** = 发扑克牌，一人一张轮着来 🃏 → 均匀但换人要重发
> - **Sticky** = 发完牌后尽量不换手（只调整必要的）🤝 → 稳定

### 🧠 记忆锚点

```
Range=按Topic范围分(不均) | RoundRobin=轮询(均匀) | Sticky=保持+均匀(推荐)
```

---

## Q4. Kafka ISR 机制

> 📌 **频率**: 2025 高频 · ★★☆  
> `In-Sync Replicas (ISR)`

### 🎯 核心概念

```
每个 Partition:
  Leader Replica ← Producer 写入 / Consumer 读取
  Follower Replicas ← 从 Leader 拉取同步

ISR (In-Sync Replicas):
  = 与 Leader 保持同步的 Replica 集合（包含 Leader 自身）
  
OSR (Out-of-Sync Replicas):
  = 落后太多的 Replica（被踢出 ISR）
```

### 📊 关键参数

| 参数 | 默认值 | 含义 |
|------|--------|------|
| `replica.lag.time.max.ms` | 30000 | Follower 落后超过此时间 → 踢出 ISR |
| `min.insync.replicas` | 1 | 写入成功所需最少 ISR 数量 |
| `acks` | all/-1 | Producer 等待 ISR 全部确认才算成功 |

### 🔄 工作流程

```
1. Producer → acks=all → Leader 写入
2. Leader 等待 ISR 中所有 Follower 拉取成功
3. 全部确认 → 返回 Success
4. Follower 拉取超时 → 被踢出 ISR → 降级为 OSR
5. OSR 追上 Leader → 重新加入 ISR
```

### ⚠️ 高可用保证

```
设置: acks=all + min.insync.replicas=2 + replication.factor=3

含义: 至少 2 个 Replica 确认写入 → 即使 1 个节点宕机数据不丢
如果 ISR < min.insync.replicas → 拒绝写入 (NotEnoughReplicasException)
```

### 💡 类比记忆

> ISR = 班级里能跟上进度的学生 🎓
> - Leader = 老师讲课
> - ISR Follower = 跟上进度的学生（能抄到笔记）
> - OSR Follower = 掉队的学生（需要课后补课才能回来）
> - `min.insync.replicas` = 至少要有 N 个学生跟上才继续讲

### 🧠 记忆锚点

```
ISR = 与Leader同步的Replica集合
acks=all + min.insync.replicas=2 → 高可靠不丢消息
Follower落后 → 踢出ISR → 追上后回归
```
