# 软件详细设计说明 - OM 子系统

| 项目 | 内容 |
| --- | --- |
| 项目名称 | 认知作战平台（v1.0） |
| 文档版本 | V1.0 |
| 密级 | 内部使用 |
| 编制 | 石建国 |
| 编制日期 | 2026-07-13 |

---

## 1 引言

### 1.1 标识

本文件为「认知作战平台」（以下简称"系统"或"平台"）运维监控子系统（OM，软部件标识 M-OM-00）的软件详细设计说明（SDD）。它是《软件需求规格说明-OM子系统》V3.8 功能需求 R-OM-001/002 与《软件概要设计说明》V2.4 模块 M-OM-01~04 的详细设计下沉，描述 OM 子系统各模块的内部结构、类与接口、数据结构、算法、状态机与部署单元，作为编码实现与单元测试的依据。

### 1.2 系统概述

运维监控子系统（OM）是基础设施层的运维数据底座，汇聚全系统日志与指标，支撑常态化运维与告警。OM 采用「**复用 KubeSphere 开源版可观测底座 + 薄分析层**」定位：日志/指标的采集、存储、告警引擎、看板渲染均复用 KubeSphere 自带能力（FluentBit / Prometheus / OpenSearch / Alertmanager / Grafana），OM 自身作为单一微服务（Java + Spring Boot），承担「**只读观测代理 + 业务数据分析入仓 + 运维作业**」三类职责，不重复实现采集/存储/告警引擎。

OM 是一个**单一微服务**（不内部分拆为独立部署单元），M-OM-01~04 为 Java 包/模块级划分，同进程内调用。OM 与 DC（Python）语言异构，经 Kafka / REST 解耦。数据流上，OM 既消费 KubeSphere 底座的只读查询能力（OpenSearch / Prometheus / Alertmanager / Grafana），又消费 Kafka 总线上的业务数据（动作日志、业务指标）做分析加工后入 ClickHouse 长期存储。

OM 子系统由四个模块组成（对应 2 项功能需求与 8 个功能点）：

| 模块 | 标识 | 对应需求 | 功能点 |
| --- | --- | --- | --- |
| 日志汇聚与检索 | M-OM-01 | R-OM-001（日志） | F-OM-01-01~02 |
| 指标采集与时序存储 | M-OM-02 | R-OM-001（指标） | F-OM-02-01~02 |
| 告警引擎 | M-OM-03 | R-OM-002（告警） | F-OM-03-01~02 |
| 运维可视化与作业 | M-OM-04 | R-OM-002（看板+运维） | F-OM-04-01~02 |

> 说明：基于「复用底座 + 只读观测」定位，M-OM-01~04 的实际职责为「**只读查询/观测代理 + 分析入仓 + 运维作业**」，采集、存储、告警引擎、看板渲染本身由 KubeSphere 底座承担（详见 §3.1、§5）。

### 1.3 文档概述

本文件章节安排如下：第 2 章引用文件；第 3 章总体设计，给出 OM 技术栈、部署单元、模块间调用关系与数据流；第 4 章数据结构设计，给出核心数据模型与存储表结构；第 5 章按模块给出详细设计（类/接口/算法/状态机）；第 6 章接口详细设计；第 7 章错误处理与可靠性设计；第 8 章部署与运维设计。需求追溯以《软件需求跟踪矩阵.xlsx》为唯一权威源，本文件第 9 章仅做概要引用。

### 1.4 术语和缩略语

沿用《软件需求规格说明-总册》第 1.4 节术语与缩略语，本文件补充以下技术术语：

| 术语 | 说明 |
| --- | --- |
| KubeSphere | 开源 PaaS 容器管理平台，本系统统一容器编排与可观测底座；本设计基于其开源版（社区版），不含企业版 Whizard |
| OpenSearch | KubeSphere 4.x 默认日志后端（Elasticsearch 分支），存储容器日志与动作日志，支持全文检索 |
| FluentBit | 轻量级日志采集器，以 DaemonSet 形式采集容器 stdout，输出到 OpenSearch |
| ServiceMonitor / PodMonitor | Prometheus Operator 的 CRD，声明式定义 Prometheus 抓取目标（Service 或 Pod 暴露的 `/metrics`） |
| Alertmanager | Prometheus 生态的告警路由组件，负责告警分组、去重、抑制、静默与多渠道通知 |
| consumer group | Kafka 消费者组机制，同组内消费者分摊分区，天然避免重复消费 |
| 只读观测 | OM 管控面不向下发告警规则/采集配置/看板定义，仅查询展示底座状态；告警规则配置由运维人员经 KubeSphere 控制台操作 |
| 选主 | 多副本场景下经分布式锁竞争 leader，仅 leader 执行定时任务；当前版本单副本无需选主，V2 演进多副本时启用 |

---

## 2 引用文件

| 文件 | 说明 |
| --- | --- |
| 《软件需求规格说明-OM子系统》V3.8 | OM 功能需求 R-OM-001/002、非功能需求（NR-P-06 日志检索≤3s、NR-M-03/04、NR-F-05）、接口（II-04/II-11）的直接输入 |
| 《软件概要设计说明》V2.4 | OM 模块划分 M-OM-01~04、§4.3 OM 模块组成与关键设计、§3.7 环境部署 |
| 《软件概要设计-架构图.md》 | OM 模块架构图与模块内功能架构图（§5 运维监控子系统） |
| 《软件概要设计-模块功能拆分-v2.xlsx》 | OM 4 模块 8 功能点的权威依据 |
| 《软件需求跟踪矩阵.xlsx》 | OM 需求双向追溯唯一权威源 |
| 《软件详细设计说明-DC子系统.md》V1.2 | DC 部署单元、Kafka 总线、ClickHouse 集群、II-04 上报侧实现的参照依据 |

---

## 3 总体设计

### 3.1 技术选型

OM 子系统技术栈遵循「复用 KubeSphere 开源版可观测底座 + 单一微服务」定位，管控面语言为 **Java + Spring Boot**（与 DC 的 Python/FastAPI 异构，经 Kafka/REST 解耦；异构原因：OM 复用 KubeSphere/JVM 生态且团队 Java 栈成熟）。

| 维度 | 选型 | 说明 |
| --- | --- | --- |
| 管控面语言/框架 | **Java 17 + Spring Boot 3.x** | OM 单一微服务，提供只读查询代理、告警观测 webhook、运维作业 API |
| 微服务治理 | **KubeSphere 微服务治理**（Spring Cloud Kubernetes） | 服务注册/发现/网关/配置复用 KubeSphere，不另起注册中心 |
| 日志采集 | **FluentBit**（KubeSphere 自带 DaemonSet） | 采集容器 stdout（含 MC 动作日志 JSON 行），输出 OpenSearch 与 Kafka 双路 |
| 日志后端 | **OpenSearch**（KubeSphere 4.x 默认） | 容器日志与动作日志全文检索（NR-P-06，近 7 日 ≤3s） |
| 指标采集 | **Prometheus**（KubeSphere 自带，Pull 模型） | 经 ServiceMonitor/PodMonitor CRD 抓取各子系统 `/metrics` |
| 时序存储 | **Prometheus TSDB**（KubeSphere 自带） | 实时指标存储（短期，默认 ~15 天） |
| 告警引擎 | **Alertmanager**（KubeSphere 自带） | 告警分组/去重/抑制/静默与多渠道通知；OM 不下发规则，仅观测 |
| 看板渲染 | **Grafana**（KubeSphere 内置） | 总览/设备/任务/爬虫/异常多维看板由 Grafana 渲染；OM 只镜像看板元数据 |
| 业务数据总线 | **Kafka**（复用 DC 集群） | 动作日志（II-11）、业务指标（II-04）经采集器推 Kafka，OM 消费 |
| 分析仓 | **ClickHouse**（复用 DC 集群，独立库 `om_analysis`） | 动作审计明细 `om_action_log` + 指标长期聚合 `om_metric_daily` |
| 元数据库 | **PostgreSQL**（复用 DC 实例，独立 schema `om`） | 告警事件归档 `alert_event` + 运维作业记录 `ops_job` + 看板元数据镜像 `dashboard_meta` |
| 分布式锁 | **Redis**（复用 DC 实例） | V2 多副本定时巡检选主（当前单副本不启用） |

### 3.2 部署单元

OM 作为**单一微服务**部署于 K8s（KubeSphere 管理），当前版本**单副本**（V2 演进多副本 HA，对应 NR-R-01/04 的 V2 目标）。M-OM-01~04 为同进程内的 Java 包/模块，不拆独立 Pod。

```mermaid
flowchart TB
    subgraph K8s["Kubernetes 集群（KubeSphere 纳管）"]
        subgraph OM_NS["Namespace: om-app"]
            OM_SVC["om-service<br/>Spring Boot Deployment ×1<br/>（V2 演进 ×N 多副本）<br/>M-OM-01~04 同进程"]
        end
    end

    KS_OBS["KubeSphere 可观测底座<br/>（复用，非 OM 部署）"]
    FB["FluentBit DaemonSet<br/>日志采集"]
    PROM["Prometheus<br/>指标 Pull"]
    OS[("OpenSearch<br/>日志检索")]
    AM["Alertmanager<br/>告警引擎"]
    GRAF["Grafana<br/>看板渲染"]

    KF[("Kafka<br/>业务数据总线")]
    CH[("ClickHouse<br/>om_analysis 库")]
    PG[("PostgreSQL<br/>om schema")]
    RD[("Redis<br/>分布式锁")]

    SRC["各子系统<br/>(DC/MC/SWM/OCC/IRS)"]
    MC["MC 执行网关<br/>动作日志 stdout(JSON)"]

    MC -->|"stdout"| FB
    SRC -->|"/metrics 暴露"| PROM
    SRC & MC -.->|"业务数据推"| KF

    FB -->|"日志"| OS
    FB -->|"动作日志"| KF
    PROM --> OS
    AM -.->|"告警事件 webhook"| OM_SVC

    OM_SVC -->|"只读查询代理"| OS & PROM & AM & GRAF
    OM_SVC -->|"消费分析入仓"| KF & CH
    OM_SVC --> PG
    OM_SVC -.-> RD
```

各部署单元职责：

| 部署单元 | 实现 | 对应模块 | 副本 |
| --- | --- | --- | --- |
| om-service | Spring Boot 单一微服务 | M-OM-01~04（同进程内 Java 包划分） | ×1（V2 演进 ×N） |

> 外部依赖（FluentBit/Prometheus/OpenSearch/Alertmanager/Grafana/Kafka/ClickHouse/PostgreSQL/Redis）均为 KubeSphere 底座或 DC 已有组件，OM 仅消费，不负责其部署运维。

### 3.3 模块间调用关系

OM 内部 M-OM-01~04 同进程调用，外部与 KubeSphere 底座各组件交互：

```mermaid
flowchart LR
    subgraph OM["om-service 单一微服务"]
        M01["M-OM-01 日志检索代理"]
        M02["M-OM-02 指标查询代理"]
        M03["M-OM-03 告警观测"]
        M04["M-OM-04 运维可视化与作业"]
        M01 -.->|"日志检索结果"| M04
        M02 -.->|"指标查询结果"| M04
        M03 -.->|"告警状态/事件"| M04
    end

    OS[("OpenSearch")] & PROM[("Prometheus")] & AM["Alertmanager"] & GRAF["Grafana"]
    KF[("Kafka")] & CH[("ClickHouse")] & PG[("PostgreSQL")]

    M01 -->|"只读检索代理"| OS
    M01 -->|"消费动作日志"| KF
    M01 -->|"审计入仓"| CH
    M02 -->|"只读查询代理"| PROM
    M02 -->|"消费业务指标"| KF
    M02 -->|"聚合入仓"| CH
    M03 -->|"只读观测规则/告警"| AM
    AM -.->|"告警事件 webhook"| M03
    M03 -->|"事件归档"| PG
    M04 -->|"看板元数据镜像"| GRAF
    M04 -->|"作业记录"| PG
```

### 3.4 数据流设计

OM 数据流分三类：

1. **只读查询流**：运维人员经 om-service REST → M-OM-01 检索 OpenSearch 日志 / M-OM-02 查询 Prometheus 指标 / M-OM-03 观测 Alertmanager 告警 / M-OM-04 查 Grafana 看板。OM 仅作只读代理转发。
2. **业务数据分析入仓流**：MC 动作日志（II-11，stdout JSON）与各子系统业务指标（II-04）→ FluentBit/采集器 → Kafka → om-service 消费 → 分析加工 → ClickHouse（`om_action_log` 审计明细 / `om_metric_daily` 长期聚合）。
3. **告警事件归档流**：Alertmanager 触发告警 → webhook 回调 om-service → 写 PostgreSQL `alert_event`（告警历史归档与处置追踪）。

---

## 4 数据结构设计

OM 不重复存储原始日志/指标（由 OpenSearch/Prometheus 存储），仅存「OM 自身分析产出 + 管控面元数据」。存储分两层：ClickHouse 分析仓（`om_analysis` 库）与 PostgreSQL 元库（`om` schema）。

### 4.1 分析仓（ClickHouse `om_analysis` 库）

#### 4.1.1 动作审计明细表 `om_action_log`（M-OM-01 分析入仓，来自 II-11）

来自 MC 执行网关经 FluentBit→Kafka 的结构化动作日志，按时间分区，高压缩列式存储，供审计查询与复盘统计（CON-07 可审计）：

```sql
CREATE TABLE om_analysis.om_action_log (
    event_id        String,           -- 动作事件唯一标识
    occurred_at     DateTime64(3),    -- 动作发生时间
    subsystem       LowCardinality(String),  -- 来源子系统(MC/DC/SWM/OCC/IRS)
    gateway_node    String,           -- 执行网关节点
    action_type     LowCardinality(String),  -- 动作类型(tap/swipe/input/cdp/...)
    terminal_type   LowCardinality(String),  -- 终端类型(mobile_real/cloud_phone/arm/virtual/browser_profile)
    terminal_id     String,           -- 终端标识
    account_id      String,           -- 关联账号标识(引用 MC account_id)
    agent_id        String,           -- 智能体标识(脚本模式为空)
    task_id         String,           -- 关联任务标识
    result          LowCardinality(String),  -- success/failed/blocked
    fuse_flag       UInt8,            -- 是否触发熔断(0/1)
    fuse_reason     String,           -- 熔断原因(风控信号/异常频率/...)
    latency_ms      UInt32,           -- 动作耗时
    extra           String,           -- 扩展字段(JSON)
    ingest_at       DateTime64(3)     -- 入仓时间
) ENGINE = MergeTree
PARTITION BY toYYYYMMDD(occurred_at)
ORDER BY (subsystem, occurred_at, terminal_id)
TTL occurred_at + INTERVAL 12 MONTH;
```

#### 4.1.2 指标长期聚合表 `om_metric_daily`（M-OM-02 分析入仓，来自 II-04）

各子系统业务指标经 Kafka 消费后按天聚合，弥补 Prometheus 短期存储（~15 天）限制，支撑容量观测与长期趋势分析：

```sql
CREATE TABLE om_analysis.om_metric_daily (
    day             Date,             -- 统计日
    subsystem       LowCardinality(String),
    metric_name     LowCardinality(String),  -- device_online_rate/task_success_rate/crawl_throughput/queue_backlog
    label_json      String,           -- 维度标签(platform/account_id/terminal_id 等, JSON)
    value_avg       Float64,          -- 日均值
    value_max       Float64,          -- 日最大
    value_min       Float64,          -- 日最小
    sample_count    UInt64,           -- 采样数
    agg_at          DateTime64(3)     -- 聚合时间
) ENGINE = MergeTree
PARTITION BY toYYYYMM(day)
ORDER BY (subsystem, metric_name, day)
TTL day + INTERVAL 24 MONTH;
```

### 4.2 元数据库（PostgreSQL `om` schema）

#### 4.2.1 告警事件归档表 `alert_event`（M-OM-03，Alertmanager webhook 回调写入）

```sql
CREATE TABLE om.alert_event (
    event_id        UUID PRIMARY KEY,         -- 告警事件标识(Alertmanager fingerprint)
    alert_rule      VARCHAR(256) NOT NULL,    -- 触发的告警规则名
    severity        VARCHAR(16) NOT NULL,     -- critical/warning/info
    status          VARCHAR(16) NOT NULL,     -- firing/resolved
    labels_json     JSONB NOT NULL,           -- 告警标签
    annotations_json JSONB,                   -- 告警注解(摘要/描述)
    fired_at        TIMESTAMPTZ NOT NULL,     -- 触发时间
    resolved_at     TIMESTAMPTZ,              -- 恢复时间
    handled_by      VARCHAR(64),              -- 处置人
    handle_note     TEXT,                     -- 处置备注
    received_at     TIMESTAMPTZ NOT NULL DEFAULT now()  -- OM 接收时间
);
CREATE INDEX idx_alert_event_fired ON om.alert_event(fired_at);
CREATE INDEX idx_alert_event_status ON om.alert_event(status);
```

#### 4.2.2 运维作业记录表 `ops_job`（M-OM-04，om-service 执行运维作业时写入）

```sql
CREATE TABLE om.ops_job (
    job_id          UUID PRIMARY KEY,
    job_type        VARCHAR(32) NOT NULL,     -- inspection(巡检)/diagnosis(故障定位)/capacity(容量观测)
    triggered_by    VARCHAR(64) NOT NULL,     -- manual/scheduled
    params_json     JSONB,                    -- 作业参数(时间范围/目标/检查项)
    status          VARCHAR(16) NOT NULL,     -- running/succeeded/failed
    result_summary  TEXT,                     -- 结果摘要
    result_detail   JSONB,                    -- 详细结果(检查项/指标快照/异常项)
    started_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    finished_at     TIMESTAMPTZ
);
CREATE INDEX idx_ops_job_type_time ON om.ops_job(job_type, started_at);
```

#### 4.2.3 看板元数据镜像表 `dashboard_meta`（M-OM-04，定时从 Grafana API 拉取）

```sql
CREATE TABLE om.dashboard_meta (
    dashboard_uid   VARCHAR(64) PRIMARY KEY,  -- Grafana dashboard uid
    title           VARCHAR(256) NOT NULL,
    category        VARCHAR(32),              -- overview/device/task/crawler/anomaly
    grafana_url     VARCHAR(512),             -- Grafana 访问地址
    panel_count     INT,                      -- 面板数
    definition_json JSONB,                    -- 看板定义快照(Grafana API 返回)
    synced_at       TIMESTAMPTZ NOT NULL DEFAULT now()  -- 最近同步时间
);
CREATE INDEX idx_dashboard_meta_category ON om.dashboard_meta(category);
```

### 4.3 Kafka Topic 设计

复用 DC Kafka 集群，OM 消费侧 topic：

| Topic | 方向 | 内容 | 生产方 | 消费方 |
| --- | --- | --- | --- | --- |
| `om-action-log` | 入 | MC 执行网关动作日志（II-11，JSON 行） | FluentBit（采 MC stdout） | om-service（入 CH `om_action_log`） |
| `om-business-metric` | 入 | 各子系统业务指标（II-04，设备在线率/任务成功率/爬虫吞吐量/队列积压） | 各子系统采集器/Metricbeat | om-service（聚合入 CH `om_metric_daily`） |
| `om-alert-event` | 入 | Alertmanager 告警事件（webhook 转 Kafka 可选，或直推 om-service） | Alertmanager | om-service（入 PG `alert_event`） |

> 说明：MC 动作日志经 stdout 由 FluentBit 采集，FluentBit 配置双路 OUTPUT（OpenSearch + Kafka `om-action-log`），实现「检索（OpenSearch）+ 审计入仓（ClickHouse）」双消费（见 §6.2）。

### 4.4 Redis Key 设计

| Redis Key 模式           | 类型                 | 用途                                     |
| ---------------------- | ------------------ | -------------------------------------- |
| `om:lock:ops_leader`   | STRING（NX，TTL 30s） | V2 多副本定时巡检选主锁（当前单副本不启用）                |
| `om:metric:agg:cursor` | HASH               | 指标聚合消费位点（按 metric_name 记录最近聚合窗口，防重复聚合） |

---

## 5 模块详细设计

### 5.1 M-OM-01 日志汇聚与检索模块（R-OM-001 日志）

#### 5.1.1 模块组成与类图

只读化后，M-OM-01 职责为「**OpenSearch 检索代理 + 动作日志消费入仓**」：

```mermaid
classDiagram
    class LogSearchService {
        +search(keyword, time_range, filters) SearchResult
        +search_by_field(field, value) SearchResult
        +get_log_context(event_id) LogContext
    }
    class ActionLogConsumer {
        +consume(records)  %% 消费 Kafka om-action-log
        +parse_and_validate(raw) ActionLogEvent
        +sink_to_clickhouse(events)
    }
    class OpenSearchClient {
        +query(dsl) SearchResult
        +build_dsl(keyword, filters) SearchDSL
    }
    class ClickHouseSink {
        +batch_insert(table, rows)
    }
    class LogSearchService ..> OpenSearchClient : 只读检索代理
    class ActionLogConsumer ..> ClickHouseSink : 审计入仓
```

#### 5.1.2 关键类说明

- **LogSearchService**：日志检索代理。封装 OpenSearch 查询，提供全文检索（NR-P-06，近 7 日 ≤3s）、字段检索、日志上下文（前后 N 行）能力。查询结果供 M-OM-04 运维作业与看板消费。
- **ActionLogConsumer**：Kafka 消费者。消费 `om-action-log` topic（MC 动作日志），解析校验 JSON 结构化字段，批量写入 ClickHouse `om_action_log`。消费失败进入重试，耗尽记死信。

#### 5.1.3 动作日志消费算法

```
consume_loop():
  while running:
    records = kafka.poll(om-action-log, timeout=5s)
    if records.empty: continue
    events = []
    dead = []
    for r in records:
      try:
        event = parse_and_validate(r.value)   # 校验必填字段(occurred_at/action_type/...)
        events.append(event)
      except ParseError:
        dead.append(r)   # 格式错误转死信
    if events:
      clickhouse.batch_insert(om_action_log, events)  # 批量插入
    if dead:
      kafka.send(om-action-log-dlq, dead)   # 死信 topic
    kafka.commit(om-action-log)
```

#### 5.1.4 全文检索性能保障（NR-P-06）

- OpenSearch 索引按日滚动（`logstash-YYYY.MM.dd`），近 7 日索引常驻热节点。
- 检索默认限定时间范围（近 7 日），超过 7 日的检索降级为冷查询（提示用户缩小范围）。
- 检索结果分页（默认 100 条/页），避免大结果集拖慢响应。

### 5.2 M-OM-02 指标采集与时序存储模块（R-OM-001 指标）

#### 5.2.1 模块组成与类图

只读化后，M-OM-02 职责为「**Prometheus 查询代理 + 业务指标消费聚合入仓**」：

```mermaid
classDiagram
    class MetricQueryService {
        +query_instant(promql) MetricResult
        +query_range(promql, start, end, step) RangeMetricResult
        +list_metrics(subsystem) list[MetricDef]
    }
    class BusinessMetricConsumer {
        +consume(records)  %% 消费 Kafka om-business-metric
        +aggregate_daily(metric, window) DailyAgg
        +sink_to_clickhouse(agg)
    }
    class PrometheusClient {
        +query(promql) MetricResult
        +query_range(promql, range) RangeMetricResult
    }
    class ClickHouseSink {
        +batch_insert(table, rows)
    }
    class MetricQueryService ..> PrometheusClient : 只读查询代理
    class BusinessMetricConsumer ..> ClickHouseSink : 聚合入仓
```

#### 5.2.2 关键类说明

- **MetricQueryService**：指标查询代理。封装 Prometheus HTTP API（`/api/v1/query`、`/api/v1/query_range`），供 M-OM-04 看板与运维作业查询实时指标（设备在线率、任务成功率、爬虫吞吐量、队列积压）。
- **BusinessMetricConsumer**：Kafka 消费者。消费 `om-business-metric` topic，按天窗口聚合业务指标（avg/max/min/count），写入 ClickHouse `om_metric_daily`。聚合游标记 Redis 防重复。

#### 5.2.3 指标聚合算法

```
aggregate_loop():
  while running:
    records = kafka.poll(om-business-metric, timeout=10s)
    if records.empty: continue
    # 按 (subsystem, metric_name, day, label) 分组聚合
    groups = group_by(records, [subsystem, metric_name, to_day(ts), label])
    aggs = []
    for key, samples in groups:
      cursor = redis.hget(om:metric:agg:cursor, key)
      if cursor and key.day <= cursor: continue   # 已聚合跳过
      agg = {avg: mean(samples), max: max(samples), min: min(samples), count: len(samples)}
      aggs.append(agg)
    if aggs:
      clickhouse.batch_insert(om_metric_daily, aggs)
      for key in groups: redis.hset(om:metric:agg:cursor, key, key.day)
    kafka.commit(om-business-metric)
```

#### 5.2.4 业务指标定义（概要）

业务指标由各子系统暴露 `/metrics`（Prometheus exposition format），经采集器推 Kafka。核心指标：

| 指标名 | 来源子系统 | 说明 |
| --- | --- | --- |
| `device_online_rate` | MC | 设备在线率（在线设备数/总设备数） |
| `task_success_rate` | MC/DC | 任务成功率（成功任务数/总任务数） |
| `crawl_throughput` | DC | 爬虫吞吐量（条/分钟） |
| `queue_backlog` | DC/MC | 队列积压（待处理任务数） |

> 各指标的具体 PromQL 与各子系统 `/metrics` 暴露字段清单，待与各子系统对齐后补充（见 §10 待补④）。

### 5.3 M-OM-03 告警引擎模块（R-OM-002 告警）

#### 5.3.1 模块组成与类图

只读化后，M-OM-03 职责为「**告警观测**」（查 Alertmanager 规则/告警状态 + webhook 接收归档），**不下发告警规则**（规则配置由运维人员经 KubeSphere 控制台操作）：

```mermaid
classDiagram
    class AlertObservationService {
        +list_alert_rules() list[AlertRuleView]
        +list_alerts(status) list[AlertView]  %% firing/resolved
        +get_alert_detail(fingerprint) AlertDetailView
    }
    class AlertEventReceiver {
        +receive_webhook(payload)  %% Alertmanager webhook 回调
        +parse_and_persist(payload)
        +dedup_by_fingerprint(event)
    }
    class AlertmanagerClient {
        +get_rules() list[AlertRule]
        +get_alerts(status) list[Alert]
    }
    class AlertEventRepository {
        +save(event)
        +find_by_fingerprint(fp) AlertEvent
        +update_status(fp, status, resolved_at)
    }
    class AlertObservationService ..> AlertmanagerClient : 只读观测
    class AlertEventReceiver ..> AlertEventRepository : 事件归档
```

#### 5.3.2 关键类说明

- **AlertObservationService**：告警只读观测。封装 Alertmanager API（`/api/v2/rules`、`/api/v2/alerts`），查询当前生效告警规则与告警状态（firing/resolved），供 M-OM-04 看板与运维作业展示。
- **AlertEventReceiver**：告警事件归档。暴露 `/api/v1/alerts/webhook` 端点接收 Alertmanager 告警回调，按 fingerprint 去重，写入 PostgreSQL `alert_event`，并更新状态（firing→resolved）。

#### 5.3.3 告警事件归档状态机

```mermaid
stateDiagram-v2
    [*] --> firing: Alertmanager webhook(status=firing)
    firing --> firing: 重复告警(fingerprint 去重,更新 fired_at)
    firing --> resolved: Alertmanager webhook(status=resolved)
    firing --> handled: 运维人员处置(填 handled_by/handle_note)
    resolved --> [*]
    handled --> resolved: 处置后告警恢复
```

#### 5.3.4 告警规则配置说明（只读观测边界）

依据《软件需求规格说明-OM子系统》V3.8 R-OM-002，告警规则（阈值/异常检测）的**配置**由运维人员经 KubeSphere 控制台操作（在 Alertmanager/Prometheus Rule CRD 中定义），OM **不提供配置下发接口**，仅提供规则与告警事件的**只读观测**。多渠道通知（邮件/IM webhook）由 Alertmanager 原生接收器承担；电话/语音告警渠道当前不支持，入 §10 待补①。

### 5.4 M-OM-04 运维可视化与作业模块（R-OM-002 看板+运维）

#### 5.4.1 模块组成与类图

M-OM-04 职责为「**看板元数据镜像 + 只读运维作业**」（严格只读，不执行任何写运维操作）：

```mermaid
classDiagram
    class DashboardMetaService {
        +list_dashboards(category) list[DashboardView]
        +sync_from_grafana()  %% 定时拉取 Grafana 看板定义
        +get_grafana_url(uid) String
    }
    class OpsJobService {
        +run_inspection(params) JobResult  %% 巡检
        +run_diagnosis(params) JobResult   %% 故障定位
        +run_capacity_observation(params) JobResult  %% 容量观测
        +list_job_history(type) list[JobRecord]
    }
    class GrafanaClient {
        +search_dashboards() list[Dashboard]
        +get_dashboard(uid) DashboardDef
    }
    class InspectionRunner {
        +run_check_items() list[CheckResult]
        +build_report(results) Report
    }
    class OpsJobRepository {
        +save(job)
        +find_by_id(id) OpsJob
    }
    class DashboardMetaService ..> GrafanaClient : 元数据镜像
    class OpsJobService ..> InspectionRunner : 只读作业执行
    class OpsJobService ..> OpsJobRepository : 作业记录
```

#### 5.4.2 关键类说明

- **DashboardMetaService**：看板元数据镜像。定时（每小时）从 Grafana API 拉取看板定义快照，写入 PostgreSQL `dashboard_meta`，供 OM 侧检索/引用。看板渲染本身由 Grafana 承担，OM 不自研看板 UI。
- **OpsJobService**：只读运维作业。提供巡检（定时跑只读检查项）、故障定位（跨日志+指标+告警关联查询）、容量观测（指标趋势分析）三类作业，均为**只读分析**，不执行任何写操作。作业执行记录写 PostgreSQL `ops_job`。
- **InspectionRunner**：巡检执行器。跑一组只读检查项（指标阈值/健康度/留存水位），产出巡检报告。

#### 5.4.3 运维作业只读边界（严格只读）

M-OM-04 所有运维操作（日志查询/故障定位/巡检/容量观测）均为**只读分析**，OM 不执行任何写运维操作（不重启服务、不扩缩容、不清理日志）。需要写操作时，OM 仅触发告警或产出建议，由人工经 KubeSphere 控制台或 MC 执行网关落地（遵循 CON-07 动作收口）。

#### 5.4.4 巡检作业算法

```
run_inspection(params):
    job = create_ops_job(type=inspection, params)
    results = []
    for check in check_items:   # 预定义只读检查项
        try:
            value = query_metric(check.metric, check.window)  # 查 Prometheus
            status = evaluate(value, check.threshold)   # ok/warn/critical
            results.append({check, value, status})
        except QueryError:
            results.append({check, status: error})
    report = build_report(results)
    job.result_summary = report.summary
    job.result_detail = report.detail
    job.status = succeeded
    save(job)
    if any critical in results:
        alert_service.notify(...)   # 触发告警(经 Alertmanager)
```

### 5.5 看板分类（R-OM-002 多维看板）

看板由 Grafana 渲染，OM 镜像元数据并按 R-OM-002 五维分类：

| 看板类别 | dashboard_meta.category | 数据源 | 内容 |
| --- | --- | --- | --- |
| 总览 | overview | Prometheus + OpenSearch | 系统整体健康度、关键指标总览、告警概览 |
| 设备 | device | Prometheus | 设备在线率、终端状态、Agent 连接状态 |
| 任务 | task | Prometheus + ClickHouse | 任务成功率、任务执行趋势、队列积压 |
| 爬虫 | crawler | Prometheus + ClickHouse | 爬虫吞吐量、采集成功率、封禁率 |
| 异常 | anomaly | Alertmanager + OpenSearch | 告警事件、熔断记录、异常日志 |

> 各看板的具体 Grafana 面板定义（PromQL/布局），待实施阶段配置（见 §10 待补⑤）。

---

## 6 接口详细设计

### 6.1 外部接口（OM 侧无新增外部接口）

OM 不直接对接系统外部（EI），外部社交平台/代理服务等由 DC/MC 对接。OM 消费 KubeSphere 底座组件 API（属平台内部能力，非 GJB 438C 外部接口范畴）。

### 6.2 内部接口（OM 侧实现/消费）

#### 6.2.1 II-04 日志与指标上报（OM 为消费方）

SRS 定义的 II-04 为「各子系统 → OM 上报日志与指标」。落地实现采用「**自动采集（Pull/采集）模型**」，数据流向仍为 各子系统→OM，但实现上：

- **日志**：各子系统容器 stdout 由 KubeSphere FluentBit DaemonSet 自动采集入 OpenSearch（OM 配置采集规则，不需各子系统主动推）。
- **指标**：各子系统暴露 `/metrics`（Prometheus exposition format），由 Prometheus 经 ServiceMonitor CRD 自动 Pull 抓取（OM 配置 scrape 规则，不需各子系统主动推）。
- **业务数据入仓**：动作日志（II-11）与业务指标（II-04）经 FluentBit/采集器推 Kafka，OM 消费分析入 ClickHouse。

> 语义澄清：II-04「上报」描述的是数据流向（各子系统→OM），实现可为 Pull/采集模型，不违背需求。

#### 6.2.2 II-11 执行网关日志上报（OM 为消费方）

SRS 定义的 II-11 为「MC 执行网关 → OM 全量动作日志 + 熔断记录」。落地实现：

- MC 执行网关将结构化动作日志以 **JSON 行写入 stdout**（含动作类型/终端/账号/结果/熔断标志等字段）。
- FluentBit 采集 stdout，**双路 OUTPUT**：
  - → OpenSearch（供运维全文检索，NR-P-06）
  - → Kafka `om-action-log`（供 OM 消费分析入 ClickHouse `om_action_log`，审计）
- MC 侧本地落盘文件 + FluentBit filesystem 缓冲兜底「全量不丢」（CON-07 可审计、R-MC-010 全量留痕）。

动作日志 JSON 结构：

```json
{
  "event_id": "evt_2026_001",
  "occurred_at": "2026-07-13T10:00:00.123Z",
  "subsystem": "MC",
  "gateway_node": "mc-gw-01",
  "action_type": "tap",
  "terminal_type": "cloud_phone",
  "terminal_id": "term_001",
  "account_id": "acc_001",
  "agent_id": "",
  "task_id": "task_001",
  "result": "success",
  "fuse_flag": 0,
  "fuse_reason": "",
  "latency_ms": 120,
  "extra": {}
}
```

#### 6.2.3 Alertmanager 告警事件 webhook（OM 为接收方）

OM 暴露 webhook 端点接收 Alertmanager 告警回调：

- 端点：`POST /api/v1/alerts/webhook`
- 输入：Alertmanager 标准 webhook payload（version/groupKey/status/alerts[]）
- 输出：HTTP 200（接收确认）
- 处理：按 fingerprint 去重，写入 PostgreSQL `alert_event`，更新状态。

#### 6.2.4 OM 管理 REST 接口（II-06，OM 为提供方）

OM 向前端/运维提供只读观测与运维作业 REST 接口：

| 路径 | 方法 | 功能 |
| --- | --- | --- |
| `/api/v1/logs/search` | GET | 日志全文检索（M-OM-01） |
| `/api/v1/logs/context/{event_id}` | GET | 日志上下文（M-OM-01） |
| `/api/v1/metrics/query` | GET | 指标即时查询（M-OM-02） |
| `/api/v1/metrics/range` | GET | 指标范围查询（M-OM-02） |
| `/api/v1/alerts/rules` | GET | 告警规则只读观测（M-OM-03） |
| `/api/v1/alerts` | GET | 告警状态只读观测（M-OM-03） |
| `/api/v1/alerts/events` | GET | 告警历史事件查询（M-OM-03） |
| `/api/v1/alerts/events/{id}/handle` | POST | 告警处置记录（M-OM-03） |
| `/api/v1/dashboards` | GET | 看板列表（M-OM-04） |
| `/api/v1/ops/jobs/inspection` | POST | 触发巡检作业（M-OM-04） |
| `/api/v1/ops/jobs/diagnosis` | POST | 触发故障定位（M-OM-04） |
| `/api/v1/ops/jobs/capacity` | POST | 触发容量观测（M-OM-04） |
| `/api/v1/ops/jobs` | GET | 作业历史查询（M-OM-04） |

> 所有接口经 COM 统一认证鉴权（NR-S-01），仅运维角色可访问。

---

## 7 错误处理与可靠性设计

### 7.1 可靠性总体策略（NR-F-05）

OM 采用薄管控面定位，管控面（Spring Boot 单一微服务）与 KubeSphere 底座采集层（FluentBit/Prometheus DaemonSet）进程隔离、部署解耦，OM 管控面故障不影响底座日志/指标采集与实时观测（NR-F-05）。OM 各故障点处理：

| 故障点 | 处理 |
| --- | --- |
| om-service 故障 | 底座采集（FluentBit/Prometheus）继续运行；运维人员直连 KubeSphere/Grafana/OpenSearch 原生界面观测；Kafka 消息保留，service 恢复后续采 |
| Kafka 消费积压 | ClickHouse 入库延迟，但不影响 OpenSearch 检索（双消费，OpenSearch 走 FluentBit 不经 Kafka）；积压超阈值告警 |
| ClickHouse 写入失败 | Kafka 消息保留兜底，CH 恢复后补写；实时观测不受影响（走 Prometheus/OpenSearch） |
| PostgreSQL 写入失败 | Alertmanager webhook 失败重试；作业记录暂存本地文件，恢复后补写 |
| OpenSearch/Prometheus/Alertmanager 故障 | 由 KubeSphere 底座自身可靠性保障（非 OM 职责）；OM 检测到查询失败时降级返回缓存或提示 |

### 7.2 Kafka 消费容错（NR-F-04）

- 消费失败的消息转死信 topic（`om-action-log-dlq`、`om-business-metric-dlq`）。
- 消费者启用幂等（按 event_id 去重，防止重启后重复入仓）。
- 提交位点采用「处理成功后提交」，避免消息丢失。

### 7.3 幂等性

- 动作日志入仓以 `event_id` 幂等（ClickHouse ReplacingMergeTree 按 event_id 去重）。
- 告警事件归档以 `event_id`（Alertmanager fingerprint）幂等，重复 webhook 仅更新状态。
- 指标聚合以 `(day, subsystem, metric_name, label)` 幂等，Redis 游标防重复聚合。

---

## 8 部署与运维设计

### 8.1 K8s 部署架构（KubeSphere 纳管）

#### 8.1.1 部署架构图

OM 部署于与 DC 同一 Kubernetes 集群，由 KubeSphere 纳管，复用 DC 的数据组件与 KubeSphere 可观测底座。

```mermaid
flowchart TB
    subgraph KS["KubeSphere 管理面（PaaS）"]
        KSUI["控制台<br/>应用负载/监控告警/日志/看板"]
    end

    subgraph K8S["Kubernetes 集群"]
        KS -.纳管.-> K8S

        subgraph OM_NS["Namespace: om-app"]
            OM_SVC["om-service<br/>Spring Boot Deployment ×1<br/>（V2 演进 ×N 多副本 HA）<br/>M-OM-01~04 同进程"]
        end

        subgraph KS_OBS["Namespace: kubesphere-monitoring（底座，复用）"]
            PROM["Prometheus"]
            AM["Alertmanager"]
            GRAF["Grafana"]
        end

        subgraph KS_LOG["Namespace: kubesphere-logging（底座，复用）"]
            FB["FluentBit DaemonSet"]
            OS[("OpenSearch")]
        end

        subgraph DC_DATA["Namespace: dc-data（复用 DC 数据组件）"]
            KF[("Kafka")]
            CH[("ClickHouse<br/>om_analysis 库")]
            PG[("PostgreSQL<br/>om schema")]
            RD[("Redis")]
        end
    end

    subgraph SRC["各子系统"]
        MC["MC 执行网关<br/>动作日志 stdout(JSON)"]
        SS["DC/SWM/OCC/IRS<br/>日志 stdout + /metrics"]
    end

    MC -->|"stdout"| FB
    SS -->|"stdout"| FB
    SS -->|"/metrics"| PROM
    SS & MC -.->|"业务数据推"| KF

    FB --> OS
    FB --> KF
    PROM --> AM
    AM -.->|"webhook"| OM_SVC

    OM_SVC --> OS & PROM & AM & GRAF & KF & CH & PG
```

#### 8.1.2 部署清单

| 资源 | 类型 | 副本 | 说明 |
| --- | --- | --- | --- |
| om-service | Deployment | ×1（V2 演进 ×N） | Spring Boot 单一微服务，M-OM-01~04 同进程 |
| om-service Service | Service | - | ClusterIP，供前端/网关访问 |
| om-config | ConfigMap | - | OM 配置（KubeSphere 底座地址、Kafka topic、巡检检查项） |
| om-secret | Secret | - | 底座组件凭据（OpenSearch/PG/CH 连接信息，NR-S-03 加密） |

> 外部依赖（Prometheus/Alertmanager/Grafana/FluentBit/OpenSearch/Kafka/ClickHouse/PostgreSQL/Redis）由 KubeSphere 底座或 DC 部署，OM 仅消费。

### 8.2 配置化（NR-M-02）

- KubeSphere 底座地址（OpenSearch/Prometheus/Alertmanager/Grafana URL）经 ConfigMap 配置，不硬编码。
- Kafka topic 名、消费组、巡检检查项、告警 webhook 路径经 ConfigMap 配置。
- 各子系统 `/metrics` 暴露与 ServiceMonitor 配置由运维人员经 KubeSphere 控制台管理。

### 8.3 可观测（NR-M-03/04）

- om-service 自身日志经 stdout 由 FluentBit 采集入 OpenSearch（OM 自观测）。
- om-service 自身指标（JVM/请求量/消费 lag）暴露 `/metrics`，由 Prometheus 采集。
- 运维仪表盘与告警由 KubeSphere/Grafana/Alertmanager 提供（NR-M-04）。

---

## 9 需求追溯（概要引用）

OM 子系统需求双向追溯的明细以《软件需求跟踪矩阵.xlsx》（RTM，GJB 438C）为唯一权威源。本文件设计对需求的覆盖概要如下：

| 需求 | 模块/功能点 | 设计章节 |
| --- | --- | --- |
| R-OM-001 日志与指标汇聚 | M-OM-01 / F-OM-01-01~02；M-OM-02 / F-OM-02-01~02 | §5.1、§5.2 |
| R-OM-002 告警与运维 | M-OM-03 / F-OM-03-01~02；M-OM-04 / F-OM-04-01~02 | §5.3、§5.4 |

非功能需求覆盖：NR-P-06（§5.1.4 日志检索≤3s）、NR-M-03/04（§8.3 全链路日志与仪表盘）、NR-F-04（§7.2 Kafka 消费容错）、NR-F-05（§7.1 降级，管控面与底座解耦）。NR-R-01/04 为 V2 目标（当前版本单副本，见 §3.2）。约束覆盖：CON-06（§3.1 私有化，复用 KubeSphere 开源版不依赖外部公有云）、CON-07（§5.4.3 运维作业只读，动作收口）。

---

## 10 待后续补充事项

1. **电话/语音告警渠道**：当前告警通知仅支持 Alertmanager 原生渠道（邮件/IM webhook/通用 webhook），电话/语音告警需自研薄适配器（FastAPI/Spring Boot 调语音网关 API，仅 P0 级告警触发），语音网关供应商待定。
2. **复杂异常检测**：当前告警以 PromQL 阈值表达（阈值告警），复杂异常检测（突变检测/异常序列/趋势预测）待后续基于 ClickHouse 长期数据实现。
3. **多集群统一纳管**：当前基于 KubeSphere 开源版单集群，多集群统一监控告警需企业版 Whizard，待后续评估。
4. **业务指标 metric 字段清单**：各子系统（DC/MC/SWM/OCC/IRS）`/metrics` 暴露的具体字段与 PromQL 定义，待与各子系统对齐后补充（见 §5.2.4）。
5. **Grafana 五类看板模板**：总览/设备/任务/爬虫/异常五类看板的具体 Grafana 面板定义（PromQL/布局），待实施阶段配置（见 §5.5）。
