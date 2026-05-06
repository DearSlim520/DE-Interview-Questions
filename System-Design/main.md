# 🎯 System Design

> 系统设计 (System Design) / 场景题 (Scenario Questions)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [设计直播实时数仓](#q1-设计直播实时数仓) | ⭐⭐⭐ |
| 2 | [设计搜索推荐数据链路](#q2-设计搜索推荐数据链路) | ⭐⭐⭐ |
| 3 | [大故障复盘](#q3-大故障复盘) | ⭐⭐⭐ |
| 4 | [项目深挖：你最复杂的一个数仓项目](#q4-项目深挖) | ⭐⭐⭐ |

---

## Q1. 设计直播实时数仓

> 📌 **频率**: 2025 · ★★☆  
> `Live Streaming Real-time Data Warehouse Design`

### 🎯 业务场景

```
直播场景核心指标:
  - 实时在线人数 (Real-time Online Count)
  - 实时 GMV (Gross Merchandise Value)
  - 礼物打赏排行 (Gift Ranking)
  - 主播带货转化率 (Conversion Rate)
  - 用户停留时长 (Stay Duration)
  
延迟要求: < 3 秒端到端
```

### 📊 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│  数据源 (Sources)                                            │
│  ├── 用户行为日志 (Behavior Log): 进房/送礼/点击/购买         │
│  ├── 交易系统 (Trade): 订单/支付                              │
│  └── 直播控制流 (Control): 开播/下播/房间状态                  │
│                         ↓                                    │
│  Kafka (ODS Layer)                                           │
│                         ↓                                    │
│  Flink Real-time ETL (DWD)                                   │
│  ├── 数据清洗 + 用户/主播维度关联 (Dim Join)                  │
│  ├── 去重 (Exactly-Once with Checkpoint)                     │
│  └── 打宽表 (Wide Table)                                     │
│                         ↓                                    │
│  Flink Window Aggregation (DWS)                              │
│  ├── 实时在线数 (Tumbling Window 10s)                         │
│  ├── 实时 GMV (Sliding Window 1min)                          │
│  └── 礼物 TopN (Session Window)                              │
│                         ↓                                    │
│  Serving Layer (ADS)                                         │
│  ├── Redis: 实时计数器 + 排行榜                               │
│  ├── Doris/StarRocks: 多维分析 (Slice & Dice)                │
│  └── WebSocket → 前端大屏                                    │
└─────────────────────────────────────────────────────────────┘
```

### ⚠️ 关键挑战 & 解法

| 挑战 | English | 解法 |
|------|---------|------|
| 峰值流量 (Traffic Spike) | Peak Load | Kafka 扩分区 + Flink Auto-scaling |
| 数据去重 (Dedup) | Deduplication | Flink State + Bloom Filter |
| 维表关联延迟 | Dim Join Latency | Async I/O + Local Cache |
| 排行榜实时性 | Ranking Freshness | Redis Sorted Set (ZADD) |
| 口径一致 | Metric Consistency | 流批统一 DWD 层口径 |

### 🧠 记忆锚点

```
直播数仓: Kafka(ODS) → Flink(DWD清洗+DWS聚合) → Redis(计数)+Doris(分析)
关键: 低延迟(<3s) + 去重 + 维表Cache + Redis排行榜
```

---

## Q2. 设计搜索推荐数据链路

> 📌 **频率**: 2025 · ★★☆  
> `Search & Recommendation Data Pipeline Design`

### 🎯 业务场景

```
搜索推荐系统的数据需求:
  - 用户特征 (User Features): 画像/偏好/历史行为
  - 物品特征 (Item Features): 商品属性/统计指标
  - 实时信号 (Real-time Signals): 最近点击/加购/搜索词
  - 样本数据 (Training Samples): 曝光/点击/转化
  - A/B 实验数据 (Experiment Data): 分桶/指标
```

### 📊 数据链路架构

```
┌─── Offline Pipeline (T+1) ────────────────────────────┐
│  Hive/Spark                                            │
│  ├── 用户画像宽表 (User Profile Wide Table)             │
│  ├── 物品特征表 (Item Feature Table)                   │
│  ├── 训练样本 (Training Sample: impression+click+buy)  │
│  └── 模型训练 (Model Training) → 模型上线              │
├─── Near-realtime Pipeline (分钟级) ──────────────────┤
│  Flink                                                 │
│  ├── 实时用户行为特征 (Real-time Behavior Feature)      │
│  ├── 实时物品统计 (Item Real-time Stats: CTR/CVR)      │
│  └── 写入 Feature Store (Redis/HBase)                  │
├─── Online Serving ────────────────────────────────────┤
│  推荐引擎 (Rec Engine)                                  │
│  ├── 召回 (Recall): ES/Faiss + 用户 Embedding          │
│  ├── 粗排 (Pre-ranking): 轻量模型                      │
│  ├── 精排 (Ranking): DNN 模型 + Feature Store           │
│  └── 重排 (Re-ranking): 多样性/业务规则                 │
└────────────────────────────────────────────────────────┘
```

### 📝 Feature Store 设计

| 维度 | Offline Feature | Online Feature |
|------|----------------|----------------|
| 更新频率 | T+1 | 实时/分钟级 |
| 存储 | Hive / HDFS | Redis / HBase |
| 示例 | 30天购买次数 | 最近5分钟点击品类 |
| 一致性 | Training = Serving (避免 Train-Serve Skew) |

### ⚠️ 关键挑战

| 挑战 | 解法 |
|------|------|
| Train-Serve Skew (训练服务特征不一致) | Feature Store 统一管理 |
| 样本延迟 (Label Delay) | 等转化窗口后回填 |
| 特征穿越 (Feature Leakage) | 严格用事件时间而非处理时间 |
| 冷启动 (Cold Start) | 用物品属性特征 / 热门推荐 |

### 🧠 记忆锚点

```
Offline(Spark训练样本+模型) + Near-RT(Flink实时特征) + Online(Feature Store+推荐引擎)
核心: Feature Store统一离线/在线特征 → 避免 Train-Serve Skew
```

---

## Q3. 大故障复盘

> 📌 **频率**: 2025 高频 · ★★★  
> `Incident Postmortem: Data Delay / Metric Inconsistency`

### 🎯 回答框架 (STAR Method)

```
Situation → Task → Action → Result
```

### 📊 常见故障场景

| 故障类型 | English | 典型原因 |
|----------|---------|----------|
| 数据延迟 | Data Delay / SLA Breach | 上游 Kafka 堆积 / Flink Backpressure / 资源不足 |
| 口径错误 | Metric Inconsistency | DWD 层逻辑变更未同步 / 维表更新不及时 |
| 数据丢失 | Data Loss | Checkpoint 失败 + Job 重启 / Kafka Retention 过期 |
| 数据重复 | Data Duplication | At-Least-Once 未去重 / 幂等写入失败 |

### 📝 标准排查流程

```
1️⃣ 发现 (Detection)
   → 监控告警 (DQC / SLA / 同环比)

2️⃣ 定位 (Root Cause Analysis)
   → 数据血缘追踪上游 → Flink UI 看反压 → 日志排查

3️⃣ 止血 (Mitigation)
   → 扩容 / 重启 / 降级 / 切换备链路

4️⃣ 修复 (Recovery)
   → 数据回刷 (Backfill) / 手动修正 / 从 Savepoint 恢复

5️⃣ 复盘 (Postmortem)
   → Timeline → Root Cause → Impact → Action Items → Follow-up
```

### 📝 复盘模板

```markdown
## Incident Postmortem

**Date**: 2025-01-15  
**Duration**: 3 hours  
**Impact**: DWS 层延迟 4 小时，影响 XX 报表

### Timeline
- 09:00 SLA 告警触发
- 09:15 定位到 Flink Job Backpressure
- 09:30 发现上游 Kafka Partition 数据倾斜
- 09:45 热 Key 单独处理 + 扩并行度
- 12:00 数据回刷完成，恢复正常

### Root Cause
上游业务方新增大客户，某 user_id 数据量暴增 100x → 单 Partition 热点

### Action Items
- [ ] 加盐打散热 Key (Owner: XX, Due: 01/20)
- [ ] 增加 Kafka Partition 数 (Owner: YY, Due: 01/18)
- [ ] 补充 Key 粒度监控 (Owner: ZZ, Due: 01/22)
```

### 🧠 记忆锚点

```
排查: 告警→血缘追踪→Flink UI→日志
流程: Detection → RCA → Mitigation → Recovery → Postmortem
复盘: Timeline + Root Cause + Impact + Action Items
```

---

## Q4. 项目深挖

> 📌 **频率**: 2025 必问 · ★★★  
> `Tell Me About Your Most Complex DW Project`

### 🎯 回答框架

```
┌─── STAR + 技术深度 ──────────────────────────────────┐
│                                                       │
│  1. Background (业务背景 + 规模)                       │
│     - 什么业务？多少数据量？多少表？                    │
│                                                       │
│  2. Challenge (技术挑战)                               │
│     - 最难的点是什么？为什么难？                        │
│                                                       │
│  3. Solution (你的方案)                                │
│     - 架构选型 + 关键设计决策 + 为什么这样选            │
│                                                       │
│  4. Result (量化结果)                                  │
│     - 性能提升 X%、成本降低 Y%、延迟从 A→B             │
│                                                       │
│  5. Reflection (反思)                                  │
│     - 如果再做一次会怎么改进？                          │
└───────────────────────────────────────────────────────┘
```

### 📝 准备建议

| 准备项 | 要点 |
|--------|------|
| 数据规模 | 日增量、总量、表数量、QPS |
| 架构图 | 能画出来讲清楚 |
| 技术选型原因 | 为什么用 X 不用 Y（对比过什么） |
| 踩过的坑 | 遇到什么问题、怎么解决的 |
| 量化指标 | 一定要有数字（延迟/成本/效率） |

### 💡 面试官关注点

| 关注点 | 他在考什么 |
|--------|-----------|
| 你的角色 | 是架构者还是执行者？ |
| 技术深度 | 能不能讲到底层原理？ |
| Trade-off | 为什么选 A 不选 B？权衡了什么？ |
| 影响力 | 这个项目对业务/团队有多大价值？ |
| 成长性 | 从中学到了什么？下次会怎么改？ |

### 🧠 记忆锚点

```
STAR: Background → Challenge → Solution → Result → Reflection
核心: 有数字 + 有取舍 + 有深度 + 有反思
```
