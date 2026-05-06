# 💾 NoSQL & Cache

> HBase / Redis 存储与缓存 (Storage & Cache)

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [HBase RowKey 设计](#q1-hbase-rowkey-设计) | ⭐⭐⭐ |
| 2 | [HBase 读写流程](#q2-hbase-读写流程) | ⭐⭐⭐ |
| 3 | [Redis 缓存击穿/穿透/雪崩](#q3-redis-缓存击穿穿透雪崩) | ⭐⭐⭐ |

---

## Q1. HBase RowKey 设计

> 📌 **频率**: 2025 高频 · ★★☆  
> `RowKey Design Principles`

### 🎯 设计原则

| 原则 | English | 说明 |
|------|---------|------|
| 唯一性 | **Uniqueness** | RowKey 必须全局唯一 |
| 散列性 | **Even Distribution** | 避免热点 (Hotspot)，数据均匀分布到各 Region |
| 长度适中 | **Short & Fixed Length** | 建议 10-100 字节，过长浪费存储 |
| 排序友好 | **Scan-friendly** | 利用字典序 (Lexicographic Order) 做范围查询 |

### 🛠️ 避免 Hotspot 的方法

| 方法 | English | 原理 |
|------|---------|------|
| 加盐 | **Salting** | 前缀加随机数/Hash，打散写入 |
| 反转 | **Reversing** | 手机号/时间戳反转，打散前缀 |
| Hash 前缀 | **Hash Prefix** | `MD5(key)[0:4] + key`，均匀分布 |

### 📝 设计示例

```
场景: 用户行为表，按 user_id + timestamp 查询

❌ 错误: timestamp_userid (时间单调递增 → 写入集中在最新 Region)
✅ 正确: reverse(userid) + timestamp
   或: MD5(userid)[0:4] + userid + timestamp
```

### 💡 类比记忆

> RowKey = 图书馆书架编号 📚
> - 散列 = 按随机号码摆放（每个书架都有书）
> - 时间前缀 = 全按入库时间排（最新的架子堆满，旧的空着）❌

### 🧠 记忆锚点

```
RowKey = Unique + Even Distribution + Short + Scan-friendly
避免Hotspot: Salting / Reversing / Hash Prefix
```

---

## Q2. HBase 读写流程

> 📌 **频率**: 2025 · ★★☆  
> `HBase Read/Write Path`

### 🎯 写入流程 (Write Path)

```
Client → ZooKeeper (查 Meta Table 位置)
       → Meta Table (查目标 RegionServer)
       → RegionServer:
           1. 写 WAL (Write-Ahead Log) — 持久化保障
           2. 写 MemStore (内存) — 按 RowKey 排序
           3. MemStore 满 → Flush to HFile (磁盘)
           4. HFile 多了 → Compaction (合并)
```

### 🎯 读取流程 (Read Path)

```
Client → ZooKeeper → Meta → RegionServer:
  1. 查 Block Cache (读缓存)
  2. 查 MemStore (内存中未落盘数据)
  3. 查 HFile (磁盘) — 利用 Bloom Filter + Block Index
  4. 合并结果 (Merge) 返回最新版本
```

### 📊 关键组件

| 组件 | English | 作用 |
|------|---------|------|
| WAL | Write-Ahead Log | 写入前先记日志，防宕机丢数据 |
| MemStore | Memory Store | 内存排序缓冲，攒批后 Flush |
| HFile | Hadoop File | 磁盘上的有序文件 (SSTable) |
| Block Cache | Read Cache | LRU 读缓存，加速热数据读取 |
| Compaction | Minor/Major Merge | 合并小 HFile → 减少读取 IO |

### 💡 类比记忆

> HBase 写入 = 记笔记 ✏️
> - WAL = 先在草稿纸记一笔（防丢）
> - MemStore = 脑子里排好序
> - Flush = 写到正式笔记本
> - Compaction = 定期整理笔记本（合并多本为一本）

### 🧠 记忆锚点

```
Write: WAL → MemStore → Flush → HFile → Compaction
Read: Block Cache → MemStore → HFile (Bloom Filter加速)
```

---

## Q3. Redis 缓存击穿/穿透/雪崩

> 📌 **频率**: 2025 超高频 · ★★★  
> `Cache Penetration / Cache Breakdown / Cache Avalanche`

### 🎯 三大问题对比

| 问题 | English | 原因 | 危害 |
|------|---------|------|------|
| 缓存穿透 | **Cache Penetration** | 查询不存在的数据（缓存和 DB 都没有） | 每次都打 DB |
| 缓存击穿 | **Cache Breakdown** | 热点 Key 过期瞬间大量请求打 DB | 单点压力 |
| 缓存雪崩 | **Cache Avalanche** | 大量 Key 同时过期 / Redis 宕机 | DB 被打崩 |

### 🛠️ 解决方案

**Cache Penetration（穿透）：**

| 方案 | 原理 |
|------|------|
| Bloom Filter | 前置过滤，不存在的直接拦截 |
| 缓存空值 (Cache Null) | 查不到也缓存 `null`（短 TTL） |
| 参数校验 | 非法请求前端拦截 |

**Cache Breakdown（击穿）：**

| 方案 | 原理 |
|------|------|
| 互斥锁 (Mutex Lock) | `SETNX` 加锁，只放一个请求去 DB |
| 永不过期 + 异步刷新 | 逻辑过期时间 + 后台线程更新 |
| 热点 Key 预加载 | 提前续期热点 Key |

**Cache Avalanche（雪崩）：**

| 方案 | 原理 |
|------|------|
| 随机 TTL | 过期时间加随机偏移，避免同时失效 |
| 多级缓存 (Multi-level Cache) | Local Cache + Redis + DB |
| 限流降级 (Rate Limiting) | 熔断保护 DB |
| Redis 高可用 (HA) | Sentinel / Cluster 防单点故障 |

### 💡 类比记忆

> - **Penetration（穿透）** = 有人一直问图书馆找不存在的书 📖❌ → 每次都去仓库翻（解法：门口贴份"确认不存在"清单）
> - **Breakdown（击穿）** = 明星签名过期了，1万粉丝同时冲进去找 🌟💥（解法：排队，只放一个人进去拿新签名）
> - **Avalanche（雪崩）** = 图书馆所有书同时到期下架 📚💨（解法：错开归还日期）

### 🧠 记忆锚点

```
Penetration = 查不存在的 → Bloom Filter + Cache Null
Breakdown = 热Key过期 → Mutex Lock + 永不过期
Avalanche = 批量过期 → Random TTL + Multi-level Cache + HA
```
