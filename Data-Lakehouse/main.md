# 🏔️ Data Lakehouse

> Iceberg / Paimon 湖仓一体

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Iceberg metadata 三层结构](#q1-iceberg-metadata-三层结构) | ⭐⭐⭐ |
| 2 | Paimon 与 Iceberg 区别（流式湖仓） | - |

---

## Q1. Iceberg metadata 三层结构

> 📌 **频率**: 2025 H2 密集出现 · ★☆☆ · 中高级岗

### 🎯 核心要点

Iceberg 元数据分 **三层套娃结构**：

```
┌─────────────────────────────────────────────────────┐
│  metadata.json  (表级)                                │
│  ├── schema / partition spec / properties           │
│  └── snapshot 列表 (指向 manifest list)              │
├─────────────────────────────────────────────────────┤
│  manifest list  (每个 snapshot 一个)                  │
│  └── 列出该 snapshot 所属的 manifest files            │
├─────────────────────────────────────────────────────┤
│  manifest file  (实际数据清单)                        │
│  └── 列出数据文件路径 + 列级统计 (min/max/null count) │
└─────────────────────────────────────────────────────┘
```

### 💡 类比记忆

> 想象你拍照备份，每天一个相册（snapshot）：
> - **metadata.json** = 相册总目录
> - **manifest list** = 每个相册的封面页，写着「今天有 A、B、C 三个袋子」
> - **manifest file** = 每个袋子的清单，写着「袋子里有 file1.parquet (min_id=1, max_id=100)...」

### ✅ 设计收益

| 能力 | 原理 |
|------|------|
| **查询剪枝** | `WHERE id=50` → 读 manifest 中的 min/max 统计 → 跳过不相关文件 |
| **Time Travel** | 切换 metadata.json 指向旧 snapshot，O(1) 操作 |
| **ACID 事务** | 原子替换 metadata.json 指针即可 |

### 🧠 记忆锚点

```
三层套娃 = 总目录 / 相册封面 / 袋子清单
```
