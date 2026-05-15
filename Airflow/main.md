# 🌀 Apache Airflow

> Workflow Orchestration · DAG Scheduling · Pipeline Automation

## 题目列表

| # | 题目 | 难度 |
|---|------|------|
| 1 | [Airflow 核心概念：DAG / Operator / Task](#q1-airflow-核心概念) | ⭐⭐ |
| 2 | [Scheduler 工作原理与 Executor 类型](#q2-scheduler-与-executor) | ⭐⭐⭐ |
| 3 | [XCom 机制与使用场景](#q3-xcom-机制) | ⭐⭐ |
| 4 | [Sensor vs Operator 区别](#q4-sensor-vs-operator) | ⭐⭐ |
| 5 | [Airflow Backfill 原理与实战](#q5-airflow-backfill) | ⭐⭐⭐ |
| 6 | [任务重试、SLA 监控与告警](#q6-重试与-sla) | ⭐⭐ |
| 7 | [Airflow vs Prefect vs Luigi 对比](#q7-airflow-vs-prefect-vs-luigi) | ⭐⭐⭐ |

---

## Q1. Airflow 核心概念

> 📌 **频率**: FAANG 2024-2025 高频 · ★★☆  
> `DAG` · `Operator` · `Task` · `TaskInstance`

### 🎯 核心组件关系

```
DAG (Directed Acyclic Graph)
  └── 定义工作流的整体结构（不含执行逻辑）
       ├── Task A (Operator 实例)
       │     └── TaskInstance = Task 在某次 DagRun 的执行记录
       ├── Task B
       └── Task C
             ↑ Task B >> Task C  (依赖声明)

DagRun = DAG 的一次具体执行（带 execution_date）
```

### 📊 核心概念一览

| 概念 | 说明 | 类比 |
|------|------|------|
| **DAG** | 有向无环图，定义任务顺序 | 工厂流水线设计图 |
| **Operator** | 任务类型模板（做什么）| 流水线工位类型 |
| **Task** | Operator 的实例（一个具体工位）| 具体的工位 |
| **TaskInstance** | Task 在某次运行的执行记录 | 某天某工位的生产日志 |
| **DagRun** | DAG 的一次完整执行 | 某天整条流水线的运转 |
| **execution_date** | 逻辑运行时间（非实际触发时间）| 报表的"数据日期" |

### 📝 常用 Operator 类型

```python
from airflow.operators.python import PythonOperator    # 执行 Python 函数
from airflow.operators.bash import BashOperator        # 执行 Shell 命令
from airflow.operators.empty import EmptyOperator      # 占位/分组用
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator
from airflow.providers.dbt.cloud.operators.dbt import DbtCloudRunJobOperator
```

### 📝 DAG 定义示例

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

with DAG(
    dag_id='daily_etl',
    schedule_interval='0 6 * * *',      # 每天 6:00 UTC
    start_date=datetime(2024, 1, 1),
    catchup=False,                       # 不补跑历史（见 Q5）
    default_args={
        'retries': 2,
        'retry_delay': timedelta(minutes=5),
    }
) as dag:

    extract = PythonOperator(task_id='extract', python_callable=run_extract)
    transform = PythonOperator(task_id='transform', python_callable=run_transform)
    load = PythonOperator(task_id='load', python_callable=run_load)

    extract >> transform >> load         # 依赖链
```

### 🧠 记忆锚点

```
DAG = 图结构（不执行）| Operator = 任务类型 | Task = 实例 | TaskInstance = 一次运行记录
execution_date = 逻辑时间（数据日期），不是实际运行时间
```

---

## Q2. Scheduler 与 Executor

> 📌 **频率**: 2025 · ★★★  
> `Scheduler` · `Executor` · `Worker` · `CeleryExecutor` vs `KubernetesExecutor`

### 🎯 Airflow 架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    Airflow Architecture                      │
│                                                             │
│  ┌─────────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │  Webserver  │    │  Scheduler   │    │   Metadata DB  │  │
│  │  (UI/API)   │◄───│  (心跳循环)   │───►│  (PostgreSQL)  │  │
│  └─────────────┘    └──────┬───────┘    └───────────────┘  │
│                            │ 提交任务                        │
│                            ▼                                │
│  ┌─────────────────────────────────────────────────┐       │
│  │               Executor                           │       │
│  │  LocalExecutor / CeleryExecutor / K8sExecutor   │       │
│  └──────────────────────┬──────────────────────────┘       │
│                         │ 分发执行                           │
│                         ▼                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Worker 1 │  │ Worker 2 │  │ Worker 3 │  (Celery/K8s Pod) │
│  └──────────┘  └──────────┘  └──────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

### 📝 Scheduler 工作循环

```
每隔 heartbeat_interval (默认 5s):
  1. 扫描 DAG 文件目录 → 解析新 DAG
  2. 检查 DagRun 触发条件（schedule_interval + 依赖）
  3. 将可执行的 TaskInstance 放入队列
  4. Executor 从队列取出 → 分发到 Worker 执行
```

### 📊 Executor 类型对比

| Executor | 原理 | 适用场景 | 缺点 |
|----------|------|----------|------|
| **SequentialExecutor** | 单线程顺序执行 | 开发/测试 | 不能并行，不能生产用 |
| **LocalExecutor** | 本机多进程 | 小规模单机 | 无法横向扩展 |
| **CeleryExecutor** | Celery + Redis/RabbitMQ 消息队列 | 中大规模，稳定 | 需维护 Celery + Broker |
| **KubernetesExecutor** | 每个 Task 起一个 K8s Pod | 资源隔离，弹性伸缩 | 冷启动慢（Pod 启动开销） |
| **CeleryKubernetesExecutor** | 混合模式 | 大规模 + 资源隔离 | 最复杂 |

### 💡 FAANG 常见选型

> 大厂基本都用 **KubernetesExecutor** 或 managed Airflow（MWAA/Cloud Composer）
> - KubernetesExecutor = 完美资源隔离，无 idle Worker 浪费
> - MWAA (AWS) / Cloud Composer (GCP) = 托管版，省去运维

### 🧠 记忆锚点

```
Scheduler = 心跳 + 解析 DAG + 触发 TaskInstance → 推入队列
Executor = 队列消费者 + 分发给 Worker
生产推荐: CeleryExecutor(稳) / KubernetesExecutor(弹性)
```

---

## Q3. XCom 机制

> 📌 **频率**: 2025 · ★★☆  
> `Cross-Communication` · Task 间数据传递

### 🎯 XCom 是什么

XCom (Cross-Communication) = Airflow 中 Task 之间**传递小数据**的机制，数据存在 Metadata DB 里。

```
Task A (xcom_push) ──► Metadata DB (xcom 表) ──► Task B (xcom_pull)
```

### 📝 使用示例

```python
# Task A: 推送数据
def extract_fn(**context):
    result = {'row_count': 1000, 'file_path': 's3://bucket/data.parquet'}
    context['ti'].xcom_push(key='extract_result', value=result)

# Task B: 拉取数据
def transform_fn(**context):
    ti = context['ti']
    result = ti.xcom_pull(task_ids='extract', key='extract_result')
    print(f"Processing {result['row_count']} rows from {result['file_path']}")

# 或者用 TaskFlow API（Airflow 2.0+，更简洁）
from airflow.decorators import task

@task
def extract() -> dict:
    return {'row_count': 1000, 'file_path': 's3://bucket/data.parquet'}

@task
def transform(data: dict):
    print(f"Got {data['row_count']} rows")

# DAG 中直接传递（XCom 自动处理）
result = extract()
transform(result)
```

### ⚠️ XCom 使用限制

| 限制 | 说明 | 替代方案 |
|------|------|----------|
| **大小限制** | 默认存 DB，大数据会撑爆 | 用 S3/GCS 存数据，XCom 只传路径 |
| **序列化** | 必须可序列化（JSON/Pickle） | 避免传复杂对象 |
| **可见性** | 存在 DB，可在 UI 查看 | 注意不要存敏感数据 |
| **性能** | 频繁大量 XCom 会拖慢 Metadata DB | 保持 XCom 轻量 |

### 💡 最佳实践

```
✅ XCom 传:  文件路径、行数、状态标志、小型配置
❌ XCom 传:  整个 DataFrame、大型 JSON、二进制文件
正确做法: Task A 写 S3 → XCom 传 s3://path → Task B 从 S3 读
```

### 🧠 记忆锚点

```
XCom = Task 间的"便利贴" 📝
xcom_push / xcom_pull | TaskFlow API 自动处理
只传小数据（路径/状态），大数据用外部存储
```

---

## Q4. Sensor vs Operator

> 📌 **频率**: 2025 · ★★☆  
> `Sensor` · `Operator` · `poke_interval` · `timeout`

### 🎯 核心区别

| 维度 | Operator | Sensor |
|------|----------|--------|
| **行为** | 执行一个动作（做事） | 等待某个条件为真（守望） |
| **运行模式** | 立即执行，完成即退出 | 轮询检查，条件满足才继续 |
| **典型用途** | 跑 SQL、调 API、触发 Spark Job | 等文件出现、等上游 DAG 完成、等 API ready |
| **资源消耗** | 运行期间占用 Worker slot | 轮询期间持续占用 Worker slot（poke 模式）|

### 📝 常用 Sensor

```python
from airflow.sensors.filesystem import FileSensor           # 等文件出现
from airflow.sensors.external_task import ExternalTaskSensor # 等其他 DAG 完成
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor  # 等 S3 文件
from airflow.sensors.sql import SqlSensor                    # 等 SQL 查询结果
from airflow.sensors.http import HttpSensor                  # 等 HTTP 接口 ready

# 示例：等待上游 DAG 完成
wait_for_upstream = ExternalTaskSensor(
    task_id='wait_upstream',
    external_dag_id='upstream_pipeline',
    external_task_id='final_task',
    timeout=3600,           # 最多等 1 小时
    poke_interval=60,       # 每 60 秒检查一次
    mode='reschedule',      # ✅ 推荐：等待期间释放 Worker slot
)
```

### ⚠️ Sensor 模式对比

| 模式 | 行为 | 适用场景 |
|------|------|----------|
| **poke** | 持续占用 Worker slot，定期检查 | 短等待（< 几分钟）|
| **reschedule** | 每次检查完释放 slot，等下次再申请 | 长等待（> 几分钟）✅ 推荐 |
| **deferrable** | Airflow 2.2+ 异步触发器，极低资源占用 | 大规模部署 |

### 💡 类比记忆

> - **Operator** = 厨师（接单就开始做菜）👨‍🍳
> - **Sensor (poke)** = 服务员站在门口等客人（持续占用人力）🧍
> - **Sensor (reschedule)** = 服务员定时去门口看一眼，没客人就回去干别的 🚶

### 🧠 记忆锚点

```
Operator = 执行动作 | Sensor = 等待条件
poke = 占 slot 等待 | reschedule = 释放 slot 等待（推荐）
Sensor timeout 必须设，否则永远阻塞
```

---

## Q5. Airflow Backfill

> 📌 **频率**: 2025 · ★★★  
> `Backfill` · `catchup` · `execution_date` · 历史数据补跑

### 🎯 什么是 Backfill

Backfill = 对**历史时间段**重新执行 DAG，补充过去的数据。

```
场景：
  - 数据管道上线前已有 2 个月历史数据需要处理
  - 某天任务失败，需要重跑那天的数据
  - 业务逻辑变更，需要重新计算过去 N 天数据
```

### 📝 Backfill 两种方式

**方式1：catchup=True（自动补跑）**
```python
with DAG(
    dag_id='daily_etl',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=True,   # 从 start_date 到今天，每天都补跑一次
) as dag:
    ...
# ⚠️ 危险：突然产生大量 DagRun，可能压垮系统
# 建议配合 max_active_runs=1 限制并发
```

**方式2：CLI 手动 Backfill（推荐）**
```bash
# 补跑指定日期范围
airflow dags backfill \
    --dag-id daily_etl \
    --start-date 2024-01-01 \
    --end-date 2024-01-31

# 带并发控制
airflow dags backfill \
    --dag-id daily_etl \
    --start-date 2024-01-01 \
    --end-date 2024-01-31 \
    --max-active-runs 3       # 最多 3 个 DagRun 并行

# 仅 Dry Run（不实际执行，只显示会跑哪些）
airflow dags backfill --dag-id daily_etl -s 2024-01-01 -e 2024-01-31 --dry-run
```

### ⚠️ Backfill 注意事项

| 注意点 | 说明 |
|--------|------|
| **幂等性 (Idempotency)** | Task 必须设计为幂等，重跑不产生重复数据 |
| **execution_date** | Backfill 以逻辑时间（数据日期）运行，不是当前时间 |
| **资源控制** | 大范围 backfill 加 `--max-active-runs` 限速 |
| **依赖数据** | 确保源数据在 backfill 日期范围内存在 |

### 💡 类比记忆

> Backfill = 补发历史报纸 📰
> - catchup=True = 一次性补发所有缺失的报纸（可能淹没邮箱）
> - CLI backfill = 指定日期范围、控制速度地补发

### 🧠 记忆锚点

```
catchup=True = 自动补跑（生产慎用）
CLI backfill = 手动控制范围和并发（推荐）
Task 必须幂等 | execution_date = 数据日期（不是运行时间）
```

---

## Q6. 重试与 SLA

> 📌 **频率**: 2025 · ★★☆  
> `Retry` · `SLA Miss` · `on_failure_callback` · 可靠性保障

### 🎯 重试配置

```python
from datetime import timedelta

default_args = {
    'retries': 3,                           # 失败后最多重试 3 次
    'retry_delay': timedelta(minutes=5),    # 每次重试间隔 5 分钟
    'retry_exponential_backoff': True,      # 指数退避（5min → 10min → 20min）
    'max_retry_delay': timedelta(hours=1),  # 退避上限 1 小时
}

# 或在单个 Task 上覆盖
PythonOperator(
    task_id='flaky_api_call',
    retries=5,
    retry_delay=timedelta(seconds=30),
    on_retry_callback=lambda ctx: send_slack_alert(ctx, 'retrying'),
)
```

### 📝 SLA（Service Level Agreement）配置

```python
from airflow import DAG
from datetime import timedelta

with DAG(
    dag_id='critical_pipeline',
    sla_miss_callback=sla_miss_handler,    # SLA 违约时回调
    default_args={
        'sla': timedelta(hours=2),         # Task 必须在 2 小时内完成
    }
) as dag:
    ...

def sla_miss_handler(dag, task_list, blocking_task_list, slas, blocking_tis):
    # 发 PagerDuty / Slack 告警
    send_pagerduty_alert(f"SLA missed: {task_list}")
```

### 📊 告警回调类型

| 回调 | 触发时机 |
|------|----------|
| `on_failure_callback` | Task 失败（重试耗尽） |
| `on_retry_callback` | 每次重试前 |
| `on_success_callback` | Task 成功 |
| `on_skipped_callback` | Task 被跳过 |
| `sla_miss_callback` | DAG 级别 SLA 超时 |

### 💡 生产实践

```
告警链路: Airflow callback → Slack / PagerDuty / OpsGenie
监控指标: task_duration / dag_run_duration / success_rate / queue_wait_time
健康检查: Airflow 内置 /health 接口 + Prometheus + Grafana
```

### 🧠 记忆锚点

```
retry = retries + retry_delay + exponential_backoff
SLA = 预期完成时间 | sla_miss_callback = 违约告警
生产必备: on_failure_callback + SLA 监控
```

---

## Q7. Airflow vs Prefect vs Luigi

> 📌 **频率**: 面试系统设计常考 · ★★★  
> 调度框架选型 · Workflow Orchestration Tools

### 🎯 核心对比

| 维度 | **Airflow** | **Prefect** | **Luigi** |
|------|-------------|-------------|-----------|
| **成熟度** | 最成熟（2015，Apache）| 现代（2018）| 较老（2012，Spotify）|
| **DAG 定义** | Python 代码（有一定学习曲线）| Python-native，更直觉 | Python，面向对象 |
| **动态 DAG** | 支持但复杂 | 一等公民，天然支持 | 有限支持 |
| **UI** | 丰富（监控/历史/日志）| 现代 UI，Cloud 版 | 基础 UI |
| **调度** | 内置 Cron 调度器 | 内置 + 事件驱动 | 依赖 cron 外部触发 |
| **Backfill** | 内置支持 | 内置支持 | 手动 |
| **生态** | 最丰富（500+ Providers）| 增长快 | 较少 |
| **托管版** | MWAA / Cloud Composer | Prefect Cloud | 无 |
| **学习曲线** | 中高（概念多）| 低（Pythonic）| 低 |
| **适用场景** | 复杂企业级 ETL | 现代数据团队，快速迭代 | 简单依赖管道 |

### 📝 同一逻辑的写法对比

```python
# Airflow
with DAG('etl', schedule_interval='@daily') as dag:
    t1 = PythonOperator(task_id='extract', python_callable=extract)
    t2 = PythonOperator(task_id='transform', python_callable=transform)
    t1 >> t2

# Prefect（更 Pythonic）
from prefect import flow, task

@task
def extract(): ...

@task
def transform(data): ...

@flow(name='etl')
def etl_pipeline():
    data = extract()
    transform(data)

etl_pipeline.serve(cron='0 0 * * *')
```

### ✅ 选型建议

| 场景 | 推荐 |
|------|------|
| 大厂 / 已有 Airflow 生态 | **Airflow** (MWAA / Cloud Composer) |
| 初创公司 / 快速迭代 | **Prefect** (Cloud 版省运维) |
| 简单依赖链，轻量需求 | **Luigi** |
| 事件驱动流式触发 | **Prefect** 或 **Dagster** |

### 🧠 记忆锚点

```
Airflow = 最成熟 + 最多 Providers + 学习曲线高 → 大厂首选
Prefect = 更 Pythonic + 动态 DAG + 现代 → 新项目首选
Luigi = 最轻量 + 最老 → 遗留系统
面试说: "团队已有 Airflow 就继续用；新项目考虑 Prefect 降低维护成本"
```
