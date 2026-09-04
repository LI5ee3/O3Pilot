# O3Pilot — ARCHITECTURE.md

> Version: 1.0  
> Status: Architecture Baseline  
> Updated: 2026-09-03  
> Applies to: O3Pilot

# 1. 文档目的

`ARCHITECTURE.md` 定义 O3Pilot 的长期正式技术架构。

本文件负责回答：

- O3Pilot 以什么运行形态部署；
- API、Worker、Scheduler 如何协作；
- Ozon 与其他外部数据如何进入系统；
- Raw、Normalized、Derived 如何在物理和逻辑上分层；
- SQLite 如何作为长期正式主数据库运行；
- 后台任务如何持久化、调度、限流、恢复和降载；
- Webhook 如何与 Seller API 状态回读配合；
- 多店铺如何在运行时隔离；
- 数据如何长期保留、降采样、回收和重算；
- 单机运行如何避免重复实例、崩溃中断和半完成状态；
- Cloudflare Tunnel 与 Cloudflare R2 在系统中的正式角色；
- O3Pilot 如何从第一版开始保证性能、可恢复性、可观测性和长期演进能力；
- O3Pilot 如何跨版本安全升级并保持持久化兼容；
- O3Pilot 如何通过完整 Backup / Restore 在另一台受支持主机恢复同一实例。

本文件不重新定义：

- 产品能力与业务边界；
- Ozon API 的具体字段和验证状态；
- 业务实体最终字段与主键；
- 指标、Profit、Forecast 的最终公式；
- UI、导航和页面布局；
- 密码 Hash、Cookie、CSRF 等具体安全实现；
- 安装命令、升级命令和运维操作手册。

这些内容分别由 `PRODUCT.md`、`DATA_SOURCES.md`、`DATA_MODEL.md`、`METRICS.md`、未来的 `SECURITY.md`、`DESIGN.md`、`DEPLOYMENT.md` 等文档定义。

---

# 2. 架构总原则

O3Pilot 从 v1 开始按长期正式运行系统设计，不采用明确知道后续需要推倒重来的核心临时方案。

核心原则：

```text
Production-Grade from v1
Long-Term Data First
Modular Monolith
Single-Machine Native
Single User / Single Active Session
SQLite as Long-Term Primary Database
Raw → Normalized → Derived
Reprocessable by Design
Read-Only by Architecture
Persistent Jobs
Incremental by Default
Recoverable by Design
Observable by Default
Simple Infrastructure
No Throwaway Core Architecture
```

首版可以暂时缺少部分产品能力，但已经进入 v1 的核心架构必须能够长期维护和演进。

“长期正式”不等于引入不必要的分布式基础设施。

除非真实问题出现，否则 v1 不引入：

- Redis；
- Celery；
- RabbitMQ；
- Kafka；
- Elasticsearch；
- ClickHouse；
- Docker；
- Docker Compose；
- Kubernetes；
- 微服务体系。

---

# 3. 系统运行边界

## 3.1 单机原生运行

O3Pilot v1 正式支持：

```text
macOS
Linux
```

O3Pilot v1 不以以下方式作为正式运行方案：

```text
Docker
Docker Compose
Kubernetes
Windows Native
云厂商专属运行环境
```

O3Pilot 是单机自托管应用。

同一个正式实例的：

- 应用进程；
- SQLite；
- Local Raw Store；
- Local Backup；

位于同一台 Mac 或 Linux 主机。

---

## 3.2 公网访问

正式公网访问模型：

```text
Browser
   ↓ HTTPS
Cloudflare
   ↓
Cloudflare Tunnel
   ↓
127.0.0.1:<o3pilot-port>
   ↓
O3Pilot
```

O3Pilot 默认只监听 Loopback：

```text
127.0.0.1
```

不要求：

- 主机拥有公网 IP；
- 路由器端口映射；
- 直接开放 80 / 443；
- Nginx / Caddy 作为必须组件。

Cloudflare Tunnel 是 v1 正式远程访问方式。

---

# 4. Access Model

O3Pilot v1 是单用户系统。

任意时刻只允许一个有效登录 Session。

正式规则：

```text
Single User
Single Active Session
New Login Revokes Previous Session
No Multi-user RBAC
No Concurrent Multi-device Sessions
Server-side Session Validation
```

行为：

```text
Device A 已登录
↓
Device B 登录成功
↓
Device A Session 被立即撤销
↓
Device A 下一次受保护请求
↓
SESSION_REVOKED / Unauthorized
↓
返回登录状态
```

“设备”在 v1 中按 Active Session 定义，不尝试依赖硬件 ID 或设备指纹识别物理设备。

具体 Session Token、Cookie、密码 Hash、CSRF、登录限流、凭证保护进入 `SECURITY.md`。

---

# 5. 总体架构：Modular Monolith

O3Pilot v1 采用模块化单体架构。

```text
O3Pilot
├── Application / API
├── Auth
├── Source Gateways
├── Dataset Contracts
├── Scheduler
├── Job System
├── Worker Runtime
├── Sync
├── Import
├── Raw Store
├── Parser / Adapter
├── Normalization
├── Reconciliation
├── Data Quality
├── Metrics
├── Profit
├── Forecast
├── Alerts
├── Recommendations
├── Read Models
├── Backup
├── Maintenance
└── Infrastructure
```

所有模块：

- 位于同一代码仓库；
- 使用同一正式领域模型；
- 使用同一 SQLite 主数据库；
- 通过明确模块接口协作；
- 不允许通过“拆成微服务”替代正确的模块边界。

---

# 6. Runtime Topology

## 6.1 一个 O3Pilot 主进程

O3Pilot v1 使用一个常驻 OS 主进程。

```text
systemd / launchd
      │
      ▼
   O3Pilot
   ├── API Runtime
   ├── Worker Runtime
   └── Scheduler Runtime
```

API、Worker、Scheduler 是三个逻辑 Runtime Role，不是三个必须独立部署的常驻服务。

Cloudflare `cloudflared` 仍作为独立系统服务运行。

---

## 6.2 API Runtime

API Runtime 负责：

- 前端 HTTP API；
- 登录与 Session 验证入口；
- 文件上传入口；
- Ozon Push Webhook 接收；
- 查询本地 Normalized / Derived / Read Model；
- 用户触发后台任务；
- 系统状态和诊断读取。

API Runtime 不应该：

- 在请求事务中执行长时间 Ozon 同步；
- 在请求事务中执行大型文件解析；
- 在请求事务中执行大规模重算；
- 把 Ozon 当作前端的实时数据库。

用户触发长任务时：

```text
API Request
↓
Create / Coalesce Job
↓
Return Job Status
```

---

## 6.3 Worker Runtime

Worker Runtime 负责：

- Ozon 数据读取；
- Import；
- Parser / Adapter；
- Normalization；
- Reconciliation；
- Data Quality；
- Metric / Profit / Forecast 计算；
- Backfill；
- Reprocess；
- Backup；
- Maintenance。

Worker 按统一 Job System 执行，不直接以无持久化的 `asyncio.create_task()` 作为正式后台任务机制。

---

## 6.4 Scheduler Runtime

Scheduler 只负责判断“什么工作应该被创建”。

Scheduler：

```text
检查 Dataset / Maintenance Schedule
↓
检查已有 Coverage / Job
↓
Create / Coalesce Job
```

Scheduler 不直接调用 Ozon API，也不直接执行大型业务计算。

---

# 7. Source Architecture

O3Pilot 不提供给业务层一个可调用任意路径的通用 Ozon Client。

正式 Source Gateway：

```text
Seller API Read Gateway
Performance API Read Gateway
Ozon XAPI Read Gateway
Webhook Intake
Official Report Import
Seller Data Import
```

不同 Gateway 分别拥有：

- 认证方式；
- Endpoint Registry；
- 请求模型；
- Pagination；
- Rate Limit；
- Retry；
- 时间边界；
- Empty-result Policy；
- Schema Contract；
- History / Backfill Capability。

禁止使用一个“万能分页器”或“万能 Endpoint 调用器”假设所有 Ozon API 具有相同协议。

---

# 8. Read-Only Architecture

O3Pilot 对 Ozon 永久保持只读。

只读不是代码约定，而是架构能力边界。

正式规则：

```text
Default Deny
+
Explicit Read Allowlist
```

Seller API、Performance API、Ozon XAPI 分别维护只读 Endpoint Registry。

业务模块只能通过：

```text
Business / Dataset Handler
↓
Typed Read Gateway
↓
Explicit Read Endpoint
↓
Ozon
```

不得通过：

```text
business_code.request(arbitrary_path, arbitrary_payload)
```

访问任意 Ozon 接口。

即使用户提供的 API Key 拥有写权限，O3Pilot 也不得因此获得：

- 商品修改；
- 内容修改；
- 价格修改；
- 库存修改；
- 订单履约写操作；
- 广告修改；
- Webhook Subscription 修改；
- 评论 / 问答写操作；
- 促销配置修改；
- 其他 Ozon 写能力。

---

# 9. Endpoint Contract 与 Dataset Contract

## 9.1 Endpoint Contract

Endpoint Contract 描述单个外部接口的技术行为，例如：

```text
auth
request schema
response schema
pagination
rate limit behavior
time range
empty result behavior
retry class
schema fingerprint
```

---

## 9.2 Dataset Contract

Dataset Contract 位于 Endpoint Contract 之上。

一个业务 Dataset 可以由：

- 一个 Endpoint；
- 多个 Endpoint；
- List + Detail；
- Webhook + API Readback；
- API + Report Reconciliation；

共同组成。

Dataset Contract 至少描述：

```text
dataset_code
source_system
shop scope
required endpoints
coverage model
freshness model
backfillability
recoverability class
catch-up policy
empty-result policy
parser / normalization contract
reconciliation policy
priority class
```

Recoverability 至少支持：

```text
IRREPLACEABLE
REPLAYABLE
REGENERATABLE
```

时间窗口短、未来不可补回的数据必须拥有更高同步优先级。

---

# 10. 数据处理流水线

正式数据流：

```text
External Source
      ↓
Source Gateway
      ↓
Raw Store / Raw Metadata
      ↓
Parser / Adapter
      ↓
Normalized Facts
      ↓
Data Quality
      ↓
Reconciliation
      ↓
Derived Facts
      ↓
Read Models
      ↓
API
      ↓
Frontend
```

逻辑数据层保持：

```text
Raw
↓
Normalized
↓
Derived
```

Derived 永远不能覆盖 Raw / Normalized Source Fact。

Read Model 永远不是 Source of Truth。

---

# 11. SQLite 主数据库

## 11.1 正式长期数据库

O3Pilot v1 使用 SQLite 作为正式长期 Primary Database，不把 SQLite 定义为“以后一定替换”的临时数据库。

基线：

```text
SQLite
+
WAL
+
Foreign Keys
+
Busy Timeout
+
Short Transactions
+
Local Persistent Filesystem
```

SQLite 数据库必须位于本机持久化磁盘。

禁止把运行数据库直接放在：

- NFS；
- SMB；
- WebDAV；
- R2 Mount；
- 其他远程共享文件系统。

PostgreSQL 不属于 O3Pilot v1 的运行依赖。

未来只有当产品形态出现真实架构变化，例如：

- 多主机共享数据库；
- 多实例水平扩展；
- 大量并发 Writer；

才重新评估 PostgreSQL。

“数据量越来越大”本身不是迁移 PostgreSQL 的理由。

---

# 12. SQLite 性能架构

O3Pilot 从 v1 就按多年历史和大表运行设计 SQLite。

## 12.1 Query-driven Index

索引由正式查询路径决定。

高频业务事实优先围绕：

```text
shop_id
business_time / business_date
entity source key
internal entity id
```

建立组合索引。

禁止无计划地给所有字段创建单列索引。

---

## 12.2 Keyset Pagination

大表用户查询不得依赖深度 `OFFSET` Pagination。

正式采用：

```text
Keyset / Cursor Pagination
```

典型模式：

```sql
WHERE (business_time, id) < (?, ?)
ORDER BY business_time DESC, id DESC
LIMIT ?
```

---

## 12.3 Current State 与 History 分离

需要高频展示当前状态的数据应建立明确 Current State / Read Model，而不是每次从完整历史中寻找最新记录。

例如逻辑上：

```text
inventory_current
inventory_history

product_current
product_history

campaign_current
campaign_history
```

Current 不覆盖 History。

---

## 12.4 Dashboard 不扫描全量事实

Dashboard 与高频摘要页面优先查询：

- Daily Aggregate；
- Derived Result；
- Current State；
- Read Model。

禁止把“页面打开”设计成对多年 Raw / Fact 做大型全表 Join 和聚合。

---

## 12.5 Incremental by Default

正常运行时只重算受新事实影响的范围。

```text
New / Changed Fact
↓
Determine affected scope
↓
Incremental Normalize / Reconcile / Calculate
```

全量 Rebuild 仅用于：

- 显式重处理；
- Parser / Mapping / Metric Version 变化；
- Recovery；
- 数据修复。

---

## 12.6 Batch Write + Bounded Transaction

网络请求和文件解析必须在数据库事务之外完成。

正确模式：

```text
Fetch
↓
Parse / Validate
↓
BEGIN
↓
Batch INSERT / UPSERT
↓
COMMIT
```

大型写任务必须分批提交，并在 Batch 之间允许高优先级工作让路。

禁止：

```text
BEGIN
↓
HTTP Request / Long CPU Work
↓
COMMIT
```

---

## 12.7 Raw 不参与普通业务查询

Raw Content 不作为 Dashboard、Orders、Inventory、Finance、Advertising 等普通页面的查询主路径。

业务查询使用：

```text
Normalized
Derived
Read Model
```

Raw 主要用于：

- Audit；
- Debug；
- Reprocess；
- Lineage；
- Schema Diagnostics。

---

## 12.8 数据库维护

O3Pilot 自身负责必要的 SQLite 维护能力，包括：

- WAL Checkpoint；
- Query Planner Statistics；
- `PRAGMA optimize` 等安全维护；
- 完整性诊断；
- 空间与容量监控；
- Backup 状态监控。

Maintenance 通过低优先级 Job 执行。

---

## 12.9 性能回归

核心查询必须有大数据量 Benchmark / Regression Test。

测试数据集应覆盖百万级核心事实和长期历史规模，至少验证：

- Dashboard；
- Orders；
- SKU 查询；
- Finance；
- Profit；
- Advertising；
- 时间过滤；
- Cursor Pagination；
- 同步写入；
- Webhook 写入；
- Backup。

架构目标：

> 常用查询性能不应随着无关历史总量增长而近似线性恶化。

---

# 13. Raw Store

Raw 原始内容与主 SQLite 物理分离。

正式结构：

```text
O3Pilot Data Directory
├── o3pilot.db
├── raw/
│   ├── api/
│   ├── webhook/
│   ├── reports/
│   ├── erp/
│   └── logistics/
└── backups/
```

SQLite 保存 Raw Metadata / Lineage；Large Immutable Raw Content 存入 Local Raw Store。

---

## 13.1 Content-addressed Storage

Raw Store 采用：

```text
SHA-256
+
Immutable Object
+
zstd Compression
+
Deduplication
```

典型路径：

```text
raw/api/ab/cd/<sha256>.json.zst
```

SQLite Metadata 至少能够关联：

```text
raw object id
sha256
storage path
source system
source endpoint / report type
content type
compressed size
original size
fetched / imported time
schema fingerprint
sync / import run
```

最终字段名称由 `DATA_MODEL.md` 和实现 Migration 定义。

---

## 13.2 Raw File First

Raw Content 写入采用：

```text
write temporary file
↓
flush / close
↓
SHA-256 verify
↓
atomic rename to final path
↓
write SQLite metadata
```

禁止先提交数据库指针再写文件。

崩溃导致的 Orphan Raw Object 可以由 Maintenance Job 安全发现和回收；数据库指向不存在的 Raw Object 属于 Data Quality / Integrity Error。

---

# 14. Import Architecture

正式 Import Pipeline：

```text
User Upload
↓
Raw Store
↓
Source-specific Adapter
↓
Raw Import Row / Metadata
↓
Normalized Fact
↓
Reconciliation
```

Adapter 类型可以包括：

```text
Ozon Official Report Adapter
ERP Adapter
Logistics Provider Adapter
Seller Data Adapter
```

Import 必须：

- 校验 Schema；
- 记录 Parser / Adapter Version；
- 支持重复导入检测；
- 支持幂等执行；
- 记录 Mapping Status；
- 不静默覆盖其他来源事实。

---

# 15. Webhook Architecture

Webhook 定位为：

```text
Change Signal
```

不是最终业务状态。

正式流程：

```text
Ozon Webhook
↓
Persist Raw Event
↓
Minimal Metadata / Idempotency
↓
ACK
↓
Create / Coalesce P0 Refresh Job
↓
Seller API Readback
↓
Normalized Current State
↓
Reconciliation
```

系统不得假设 Webhook：

- 永不重复；
- 严格有序；
- 永不遗漏；
- 永远实时；
- Payload 永远等同于当前最终状态。

Webhook 失败不应破坏 API 驱动的最终一致性。

---

# 16. Reprocessing 与 Publication

Raw、Normalized、Derived 必须支持版本化重处理。

例如：

```text
Raw Fact
↓
Parser v1
↓
Normalized
↓
Metric v1
```

未来可以：

```text
同一 Raw Fact
↓
Parser v2
↓
Normalized v2
↓
Metric v2
```

需要版本化的对象包括但不限于：

- Parser；
- Adapter；
- Mapping；
- Finance Classification；
- Metric；
- Profit Formula；
- Forecast Model / Contract。

Derived 重建采用 Staging → Validate → Publish：

```text
Build new result
↓
Validate completeness
↓
Publish new active version
↓
Old version enters retention
```

禁止：

```text
Delete current result
↓
Rebuild in place
```

使用户在计算过程中看到半完成状态。

---

# 17. Persistent Job System

O3Pilot 使用一个统一持久化 Job Queue。

Job 负责 Orchestration，不替代业务 Run / Lineage 对象。

```text
job
= 工作调度与执行生命周期

sync_run / import_run / calculation_run / forecast_run / backup_run
= 业务执行结果与长期追溯
```

Job 基础 Contract 应支持：

```text
job_id
job_type
shop_id
status
priority_class
scheduled_at
created_at
started_at
finished_at
attempt_count
max_attempts
next_retry_at
idempotency_key
owner_instance_id
execution_id
error_code
error_message
payload
```

最终数据库字段由 Data Model / Migration 定义。

Job Status 至少支持：

```text
PENDING
RUNNING
RETRY_WAIT
SUCCEEDED
FAILED
CANCELLED
INTERRUPTED / ORPHANED
```

---

# 18. Job Priority

Job 分五级：

## P0 — Realtime Consistency

例如：

```text
Webhook Follow-up
REFRESH_ENTITY
关键状态 Reconciliation
```

## P1 — Expiring / Irreplaceable Capture

例如：

```text
Performance SKU Daily
其他不可补回短窗口 Dataset
关键日终采集
```

## P2 — Normal Incremental Sync

例如：

```text
Orders
Finance
Returns
Inventory
Products
Analytics
Performance
Rating
```

## P3 — Heavy User / Historical Work

例如：

```text
Historical Backfill
Large Import
Reprocess
Full Recalculation
Forecast Backtest
```

## P4 — Maintenance

例如：

```text
Backup
Inventory Compaction
Derived GC
Job GC
Log Rotation
Database Maintenance
Raw Orphan Cleanup
```

调度采用：

```text
Priority Class
+
Within-class Fairness / Aging
```

高优先级不得让普通历史任务永久饿死，但 P4 不因等待时间变长而压过 P0 / P1。

---

# 19. Job Idempotency 与 Coalescing

所有正式后台 Job 默认必须安全重复执行。

相同未完成任务通过 `idempotency_key` 合并，而不是重复创建。

Webhook 高频重复变化采用 Coalescing：

```text
same entity + same refresh purpose
↓
existing pending job
↓
merge
```

如果目标 Job 已运行，可以记录一次 `rerun_requested` 或等价状态，在完成后最多再次刷新一次，而不是为每个重复事件发起一次 API 请求。

---

# 20. Crash Recovery

## 20.1 Single Primary Instance

一个 Data Directory 同一时刻只允许一个 O3Pilot Primary Instance。

启动时获取 OS-level Exclusive Instance Lock。

Lock File 可以记录：

- PID；
- Instance ID；
- App Version；
- Started At；

用于诊断，但不能只凭“PID 文件存在”判断实例是否仍然运行。

第二个实例无法获得 Lock 时必须拒绝作为 Primary 启动。

---

## 20.2 Instance Identity

每次正常启动生成唯一：

```text
instance_id
```

用于：

- Job ownership；
- Crash recovery；
- Runtime diagnostics。

不使用 PID 作为唯一运行实例身份。

---

## 20.3 Job Claim

Job 通过短事务 Claim：

```text
BEGIN
↓
select eligible job
↓
RUNNING
owner_instance_id = current
execution_id = new
↓
COMMIT
```

真正的网络 / CPU 工作在事务之外执行。

---

## 20.4 Startup Recovery

如果进程崩溃：

```text
Job = RUNNING
owner = old instance
```

新实例获得 Primary Lock 后，将旧 RUNNING Job 识别为 Interrupted / Orphaned，并按照 Job Recovery Policy：

```text
RETRYABLE
RESUMABLE
RESTART_FROM_SCRATCH
```

执行恢复。

Checkpoint 是性能优化，不是正确性的唯一基础。

---

## 20.5 Graceful Shutdown

正常停止时：

```text
stop accepting new scheduled work
↓
stop claiming new jobs
↓
request cooperative cancellation / checkpoint
↓
finish open short transaction
↓
close SQLite
↓
release instance lock
```

长任务不得要求系统必须等待整个任务完成才能正常关闭。

---

# 21. Scheduler Catch-up

系统重启后不简单按当前时间继续，而是根据 Dataset Contract 判断错过任务如何处理。

Catch-up Policy 至少支持：

```text
CATCH_UP_REQUIRED
COALESCE
SKIP_MISSED
```

例如：

- 不可补回短窗口 Dataset：立即在仍可获得窗口内补抓；
- 可历史回补 Dataset：合并缺失 Coverage；
- Current State Snapshot：只恢复当前真实状态，不伪造关机期间不存在的历史 Snapshot；
- Maintenance：通常合并或跳过错过次数，不补跑所有历史 Cron。

Scheduler 以 Coverage 为依据，而不是只以“Cron 是否执行过”为依据。

---

# 22. Resource Governance

O3Pilot 不使用“不断增加 Worker 数量”解决积压。

正式原则：

```text
Backlog is a scheduling signal,
not an instruction to create unlimited concurrency.
```

每类 Job 可以声明：

```text
network source
DB write weight
CPU weight
Disk weight
```

Resource Governor 依据任务优先级与当前资源状态决定是否执行。

---

## 22.1 Network Governance

网络治理至少按来源隔离：

```text
Seller API
Performance API
Ozon XAPI
Cloudflare R2
```

Ozon 限流进一步按：

```text
source_system
+ shop_id
+ endpoint_group
```

隔离。

一个 Shop / Endpoint 的限流或故障不得无条件拖住其他数据源。

---

## 22.2 Adaptive Rate Limit

Architecture 不把未经长期验证的固定 Ozon Rate Limit 数字写死。

采用保守初始并发 + 自适应调节：

```text
normal request
↓
429
↓
Retry-After if available
or exponential backoff + jitter
↓
reduce affected limiter
↓
gradual recovery
```

400 / 403 / Unsupported Range 等确定性错误不得无限重试。

---

## 22.3 Circuit Breaker

来源持续失败时使用按来源隔离的 Circuit Breaker：

```text
CLOSED
↓ repeated temporary failures
OPEN
↓ cooldown
HALF_OPEN
↓ probe
CLOSED
```

Performance API 故障不得导致 Seller API、本地查询、Webhook Intake 等一起停止。

---

## 22.4 SQLite Write Governance

后台 Bulk Writer 默认同一时刻最多一个。

大型 Job：

```text
prepare outside transaction
↓
acquire write slot
↓
write bounded batch
↓
COMMIT
↓
release / yield
```

Session、Job Claim、Webhook Metadata 等极短控制写入不应被大型任务长期阻塞。

---

## 22.5 CPU / Disk Heavy Governance

默认：

```text
CPU_HEAVY concurrency = 1
DISK_HEAVY concurrency = 1
```

例如：

- Forecast Backtest；
- Full Recalculation；
- Large Reprocess；
- Backup；
- Large Import；
- Inventory Compaction；

不得无控制同时执行。

P4 Maintenance 只在低负载条件下运行，并应能够安全让路。

---

# 23. 数据生命周期

年龄本身不是删除 Source Fact 的理由。

数据生命周期根据数据性质、可恢复性和长期查询价值决定。

---

## 23.1 Normalized Core Facts

以下核心事实不按年龄自动删除：

- Mother Order / Posting / Posting Item；
- Return / Reverse Logistics / WHD；
- Finance Accrual；
- Settlement / Payout；
- Seller Cost；
- Seller Logistics Cost；
- Package Measurement；
- Analytics Daily Fact；
- Performance Daily Fact；
- Exchange Rate；
- Product Identity / Mapping History。

原则：

```text
Source Fact = Long-term Data Asset
```

---

## 23.2 State History

Product、Price、Rating、Campaign 等状态型对象采用：

```text
Current State
+
Change History
+
Observation Coverage
```

完全相同的连续状态允许物理去重，但不能丢失“期间持续成功观察且状态未变化”的历史语义。

---

## 23.3 Inventory Multi-resolution History

Inventory 采用：

```text
最近 90 天
→ 完整 Intraday Snapshot

超过 90 天
→ Verified Daily Inventory Fact 永久保留
```

Daily Inventory Fact 至少应能表达：

```text
opening stock
closing stock
min / max stock
first / last observed time
observation count
stock / out-of-stock state
coverage status
```

90 天是 v1 默认 Retention，可配置，不属于业务指标口径。

Compaction 必须：

```text
build daily
↓
validate coverage / aggregate
↓
mark compaction succeeded
↓
then GC intraday
```

Partial / Failed / Unknown Compaction 不得删除原始 Intraday 数据。

---

## 23.4 Raw Retention

Local Raw Store v1 默认不按年龄自动删除。

尤其 IRREPLACEABLE Raw 默认永久保存。

R2 不作为 Raw 生命周期成立的前提。

---

## 23.5 Derived Retention

Rebuildable Derived：

```text
Current Formal Version
→ keep

Previous / Superseded Results
→ default 90 days retention

Calculation Run Metadata / Version / Coverage
→ long-term retain
```

已实际发布给用户的历史业务产物属于历史事实，例如：

- Issued Forecast；
- Emitted Alert；
- Surfaced Recommendation；

不得因为新模型或新规则出现而改写或自动删除。

---

## 23.6 Job / Log Retention

默认：

```text
Detailed Job History = 180 days
Verbose Debug / Runtime Logs = 30 days
```

重要长期追溯进入结构化：

- Sync Run；
- Import Run；
- Calculation Run；
- Forecast Run；
- Backup Run；
- Data Quality Issue。

Read Model / Cache 属于 `REGENERATABLE`，不具有永久保留要求。

---

# 24. Garbage Collection

自动清理采用：

```text
Generate / Compact
↓
Validate
↓
Mark replacement complete
↓
Retention
↓
GC eligible data
```

禁止“先删旧数据再生成新数据”。

GC 通过低优先级、幂等、可中断 Job 执行，例如：

```text
COMPACT_INVENTORY
GC_DERIVED
GC_JOBS
ROTATE_LOGS
RAW_ORPHAN_CLEANUP
DATABASE_MAINTENANCE
```

---

# 25. Backup Architecture

Backup Capability 是 O3Pilot v1 必须实现的代码能力。

Cloudflare R2 配置是可选项。

未配置 R2 时 O3Pilot 必须完整正常运行。

正式关系：

```text
SQLite Primary Database
+
Local Raw Store
        │
        ▼
Consistent Local Backup
        │
        └── optional → Cloudflare R2
```

R2 是：

```text
Optional Off-device Backup Target
```

R2 不是：

- Transactional Database；
- SQLite Live Filesystem；
- Primary Raw Store；
- O3Pilot 正常运行的必须依赖。

Hyperdrive 不属于 O3Pilot v1 架构。

---

## 25.1 Last Known Good Backup

Backup 使用：

```text
consistent snapshot
↓
temporary backup
↓
integrity validation
↓
checksum
↓
atomic publish
↓
backup_run = SUCCEEDED
```

新的 Backup 失败不得破坏上一个已验证可恢复的 Backup。

R2 Upload 是本地 Backup 之后的独立阶段。

```text
Local Backup Success
!=
R2 Upload Success
```

R2 故障不把已经成功的本地 Backup 改写为失败。

Backup Package 的架构组成、可移植性、Restore 兼容性与跨主机恢复保证由本文件的 `Portable Instance Backup / Restore Contract` 定义。

具体命令、目录权限、用户操作流程、凭证加密算法与 Recovery Key 管理由 `DEPLOYMENT.md` 与 `SECURITY.md` 定义。

---

# 26. Time Architecture

O3Pilot 必须区分：

```text
Source Event Time
Source Business Time / Date
Fetched At
Coverage Start / End
Imported At
Calculated At
Published At
```

系统内部 Timestamp 必须具有明确时区语义。

UI 默认展示时区由产品基线决定，但不得覆盖或丢失原始时间。

正式原则：

```text
Schedule Time
!=
Data Complete Time
```

Scheduler 只决定“什么时候尝试采集”。

Dataset Contract 决定：

- 什么时间范围应该存在；
- 什么条件下认为 Coverage Complete；
- 延迟数据何时再次回补；
- 哪个 Business Date 归属最终事实。

---

# 27. Disk Capacity Protection

SQLite + Raw Store 是长期增长型本地数据资产。

O3Pilot v1 必须具备：

- Database Size Monitoring；
- Raw Store Size Monitoring；
- Backup Size Monitoring；
- Free Disk Space Monitoring。

容量状态至少逻辑区分：

```text
NORMAL
WARNING
CRITICAL
```

严重磁盘压力时：

- 优先停止或延后 P3 / P4 大型任务；
- 不无控制继续 Historical Backfill / Reprocess / Backup；
- 产生明确系统告警；
- 不自动删除不可替代 Source Fact / Raw。

具体阈值由实现 Benchmark 与 `DEPLOYMENT.md` 定义。

---

# 28. Data Quality 与 Reconciliation

外部来源失败、空集合、Schema 变化和跨来源差异必须显式表示。

Architecture 必须支持：

- Unexpected Empty Detection；
- Schema Fingerprint Change；
- Missing Required Field；
- Unmatched Entity；
- Sync Gap；
- Duplicate Source Object；
- Source Mismatch；
- Unknown Currency；
- Raw Integrity Error。

不同数据源对同一事实不允许静默覆盖。

Reconciliation 可以发生在：

```text
Webhook ↔ Seller API
API ↔ Official Report
Order ↔ Finance
Ozon Fact ↔ Seller Data
Current State ↔ Historical Fact
```

发现差异时保留来源与 Coverage，而不是通过覆盖原始事实“解决”冲突。

---

# 29. Failure Isolation 与 Recovery Mode

单个数据源故障不得导致整个 O3Pilot 不可用。

例如：

```text
Performance API unavailable
```

应允许：

- Seller API 继续运行；
- 已同步数据继续查询；
- 本地 Metric / Profit 继续按已有 Coverage 工作；
- UI 明确显示 Performance Freshness / Availability 状态。

如果发现严重本地完整性问题，例如：

- SQLite integrity failure；
- 关键 Migration failure；
- 严重 Raw Metadata mismatch；

O3Pilot 不自动执行破坏性修复。

进入：

```text
Recovery / Degraded Mode
```

在 Recovery Mode 中应停止：

- Scheduler 正常写任务；
- Worker 业务写入；
- 可能扩大损坏的后台操作。

同时允许必要的：

- 基础诊断；
- Backup 状态读取；
- 数据完整性状态读取；
- 恢复入口。

具体恢复操作由 `DEPLOYMENT.md` 定义。

---

# 30. Observability

O3Pilot 从 v1 开始提供结构化运行可观测性。

至少能够诊断：

```text
Application / Instance
Database
Raw Store
Disk Capacity
Job Queue
Sync Run
Import Run
Calculation Run
Backup Run
Data Quality
Source Freshness
Rate Limiter
Circuit Breaker
```

建议长期监控指标包括：

```text
DB size
WAL size
query duration
slow query count
write transaction duration
write wait
rows inserted / updated
job duration
pending job count
oldest pending age
retry wait count
source throttle state
circuit state
backup duration
backup last success
raw store size
free disk space
```

用户页面不必展示所有内部指标，但系统必须能够在故障时解释“慢在哪里、缺在哪里、失败在哪里”。

---

# 31. Startup Pipeline

正式启动顺序：

```text
1. Resolve Data Directory
2. Acquire OS-level Instance Lock
3. Load Configuration
4. Open SQLite
5. Verify basic DB integrity / compatibility
6. Apply / verify permitted schema migrations
7. Verify Raw Store basic state
8. Recover orphaned / interrupted Jobs
9. Check disk capacity
10. Initialize Worker Runtime
11. Initialize Scheduler Runtime
12. Start API serving
```

严重步骤失败时进入 Recovery / Degraded Mode，而不是带病继续同步。

Schema Upgrade 与 Application Compatibility 的完整 Contract 由第 34 节定义。

---

# 32. Deployment Process Management

Linux 正式进程管理：

```text
systemd
```

macOS 正式进程管理：

```text
launchd
```

O3Pilot 自身作为一个主服务运行。

Cloudflare Tunnel 由独立 `cloudflared` 服务运行。

具体安装目录、权限、自动启动、升级、回滚、卸载和恢复命令进入 `DEPLOYMENT.md`。

---

# 33. 测试要求

Architecture 必须通过以下类型测试验证，而不仅仅依赖单元测试：

## 33.1 Data Integrity

- 重复同步不产生重复业务记录；
- Crash + Retry 结果与一次成功执行一致；
- Raw Metadata 与 Raw File 可核验；
- Derived Publish 不出现半版本；
- Inventory Compaction 后长期事实不丢失。

## 33.2 Runtime Recovery

- 进程强杀；
- 主机突然关机模拟；
- RUNNING Job 恢复；
- Partial Import；
- Partial Backup；
- Partial Reprocess；
- Second Instance 启动拒绝。

## 33.3 Performance

- 百万级事实表查询；
- 长期历史 Cursor Pagination；
- 前端读取与批量同步并行；
- Webhook 高频输入；
- Backfill 与日常同步并存；
- Backup 与数据库读取并存。

## 33.4 Source Failure

- 429；
- Temporary 5xx；
- Timeout；
- Circuit Breaker；
- Empty Unexpectedly；
- Schema Change；
- Access Denied；
- 短窗口 Catch-up。

---

# 34. Upgrade & Compatibility Contract

O3Pilot 的升级机制属于核心架构能力。

长期运行后，一个实例会同时拥有多年积累的：

- SQLite Schema；
- Raw Store；
- Config；
- Pending / Retry / Interrupted Job；
- Parser / Adapter 历史；
- Dataset Contract；
- Metric / Profit / Forecast 版本；
- 已发布 Forecast / Alert / Recommendation。

因此升级不得假设“数据库可以重建”或“用户可以重新同步全部历史”。

正式原则：

> Upgrade must preserve historical facts, lineage, recoverability and published historical meaning.

## 34.1 版本维度分离

O3Pilot 不使用单一 Application Version 代替所有持久化格式版本。

至少独立维护：

```text
Application Version
Database Schema Version
Raw Store Format Version
Config Schema Version
Job Payload Version
Dataset Contract Version
Parser / Adapter Version
Mapping Version
Metric / Profit Version
Forecast Model Version
```

版本职责：

- Application Version 使用 SemVer；
- Database Schema Version 使用单调递增版本；
- Raw Store Format Version 使用单调递增版本；
- Config Schema Version 使用单调递增版本；
- Job Payload 按 Job Type 独立版本化；
- Dataset / Parser / Mapping / Metric / Forecast 由各自 Contract 管理版本。

这些版本不得因为 Application Version 更新而被静默重解释。

## 34.2 Compatibility Manifest

每个正式 Release 必须携带机器可读的 Compatibility Manifest。

它至少声明：

```text
application_version
supported_database_schema_range
target_database_schema_version
supported_raw_store_format_range
target_raw_store_format_version
supported_config_schema_range
target_config_schema_version
supported_job_payload_versions
```

运行时根据 Manifest 判断兼容状态，不依赖人工推测。

如果当前持久化状态超出 Application 支持范围，必须 Fail Closed。

## 34.3 同一 Major Version 的直接升级

同一 Major Version 内，正式 Release 默认必须支持从该 Major 此前正式版本直接升级到当前版本。

例如：

```text
1.1.x
↓
1.9.x
```

不得要求用户人工依次安装所有中间 Minor Release。

数据库 Migration Engine 必须能够从当前 Schema 按完整 Migration Chain 依次升级到目标 Schema。

未来 Major Version 如确需提高最低支持版本，必须存在明确、可验证的数据迁移路径。

## 34.4 Migration 不可变

已进入正式 Release 的 Migration 是不可变历史。

例如：

```text
migration_0027
```

发布后不得修改其内容。

若需要修正：

```text
migration_0028_fix_xxx
```

通过新的 Migration 完成。

Migration Ledger 至少保存：

```text
migration_id
migration_checksum
applied_at
application_version
status
```

启动时如发现已应用 Migration 的 Checksum 与当前代码不一致，应视为兼容性异常，不继续正常写入。

## 34.5 禁止散落式 Schema 修改

禁止在业务代码、Repository、API Handler 或普通 Startup Logic 中散落：

```text
if column missing → ALTER TABLE
if table missing → CREATE TABLE
```

所有正式持久化结构变更必须进入统一 Migration System。

Schema Version 是数据库结构状态的唯一正式演进依据。

## 34.6 Upgrade Preflight

任何可能修改持久化状态的升级，在执行 Migration 前必须先完成 Preflight。

顺序至少包括：

```text
Acquire Instance Lock
↓
Stop Scheduler / Worker from accepting new work
↓
Load Compatibility Manifest
↓
Read current persisted versions
↓
Verify SQLite basic integrity
↓
Verify Raw Store basic state
↓
Check disk capacity
↓
Verify migration path exists
↓
Create required Pre-upgrade Backup
↓
Begin migration
```

任一必要条件不满足：

> Migration must not start.

不得在持久化结构修改到一半后才发现缺少磁盘空间、缺少 Migration 或版本不兼容。

## 34.7 Pre-upgrade Last Known Good Backup

如果 Release 会修改以下任一内容：

- Database Schema；
- Persistent Database Data；
- Config Schema；
- Raw Store Format；
- 其他不可安全原地撤销的持久化状态；

则升级前必须生成 Last Known Good Backup。

Backup Manifest 至少记录：

```text
backup_id
created_at
application_version
database_schema_version
raw_store_format_version
config_schema_version
database_checksum
```

涉及 Raw Store 时，还必须能够验证对应 Raw Backup / Manifest 的完整性状态。

该 Backup 在新版本成功验证前不得被 Retention Policy 回收。

## 34.8 Migration Class

Migration 分为三类。

### STARTUP_ATOMIC

适用于能够快速、安全完成的结构修改，例如：

- 创建小型表；
- 增加 Metadata；
- 创建经过验证不会造成长时间阻塞的索引或结构；
- 轻量 Schema 调整。

应尽可能在数据库事务中原子完成。

### ONLINE_BACKFILL

适用于大型数据转换，例如：

- 数百万行历史字段回填；
- 历史 Normalized Fact 重建；
- 大规模 Parser Reprocess；
- 新 Derived / Read Model 历史生成。

流程：

```text
Schema Expand
↓
Application reaches compatible runtime
↓
Persistent Migration / Backfill Job
↓
Bounded batches
↓
Checkpoint
↓
Validate
↓
Publish
```

ONLINE_BACKFILL 不得为了完成数小时工作而长时间阻塞应用启动。

### OFFLINE_MIGRATION

仅用于无法维持在线兼容性的重大持久化格式变化。

系统进入 Maintenance / Migration Mode，完成并验证后才能恢复正常业务运行。

OFFLINE_MIGRATION 是例外，不是普通升级默认方式。

## 34.9 Expand → Migrate → Contract

大型 Schema 演进默认采用：

```text
Expand
↓
Migrate
↓
Validate
↓
Publish
↓
Contract
```

例如旧字段替换时：

1. 新旧结构先共存；
2. 新代码开始兼容新结构；
3. 后台回填历史；
4. 验证 Coverage 和一致性；
5. 切换 Active Contract；
6. 后续版本再删除旧结构。

禁止在大表上为了代码简洁直接执行高风险的“删除旧结构并一次性重建全部历史”。

## 34.10 Migration Crash Recovery

Migration 必须与普通后台任务一样具有 Crash Recovery Contract。

如果发生：

```text
Migration RUNNING
↓
power loss / process crash
```

重新启动后必须能够明确识别 Interrupted Migration。

对于 STARTUP_ATOMIC：

```text
transaction rollback
↓
retry
```

对于 ONLINE_BACKFILL：

```text
checkpoint
↓
resume / safe retry
```

Migration 必须满足：

- Idempotent；或
- 具有明确 Resume / Recovery Strategy。

不得通过猜测数据库当前状态继续执行。

## 34.11 不使用通用 Reverse Migration 作为回滚机制

O3Pilot 不要求每个 Migration 都实现通用 `downgrade()`。

复杂历史转换经常不可逆，强行 Reverse Migration 会增加：

- 数据丢失风险；
- Schema 与历史语义不一致；
- 未充分测试的逆向路径；
- 长期维护成本。

正式 Rollback 模型为：

```text
Previous Application Release
+
Pre-upgrade Last Known Good Backup
```

而不是：

```text
reverse every migration
```

## 34.12 Rollback Contract

如果持久化 Migration 尚未发生，新 Application 启动失败，可以直接回退到之前 Application Release。

如果 Migration 已经改变旧 Application 无法理解的持久化结构，则必须：

```text
Stop new application
↓
Restore Pre-upgrade Last Known Good Backup
↓
Restore previous application release
↓
Verify compatibility
↓
Start
```

旧版本程序不得直接打开高于自身支持范围的数据库 Schema 并尝试运行。

## 34.13 Fail Closed on Newer Persistent Format

例如：

```text
Application supports DB schema <= 50
Database schema = 53
```

程序必须进入：

```text
INCOMPATIBLE_NEWER_SCHEMA
```

并禁止：

- Scheduler；
- Worker；
- Business Write；
- 自动 Schema 降级。

可以保留有限诊断 / Recovery Surface。

相同原则适用于：

- Raw Store Format；
- Config Schema；
- 其他关键持久化格式。

## 34.14 Raw Store 向后可读

历史 Raw Object 是不可变原始证据。

Parser / Adapter 升级不得原地改写历史 Raw Content。

例如：

```text
Raw Object
├── created under Parser v1
├── reprocessed by Parser v4
└── reprocessed by Parser v8
```

三个 Parser 都引用同一个历史 Raw Object。

Raw 的意义不因当前 Parser Version 改变。

## 34.15 Raw Store Format Migration

如未来需要从 Raw Store Format v1 升级到 v2，迁移期新版 Runtime 应优先采用兼容窗口：

```text
read v1
read/write v2
```

然后后台：

```text
v1 object
↓
copy / transform to v2
↓
checksum / semantic validation
↓
mark v2 verified
↓
old representation becomes GC eligible
```

不得一次升级中原地覆盖全部多年 Raw 数据。

旧表示只有在新表示验证成功后才允许回收。

## 34.16 Job Payload Versioning

持久化 Job 可能跨 Application Upgrade 存在。

因此每个 Job 必须记录：

```text
job_type
job_payload_version
payload
```

新 Handler 面对旧 Payload 时只能：

1. 明确兼容并执行；
2. 通过正式 Payload Migration 升级；
3. 明确标记 Incompatible，并安全生成替代 Job（若 Contract 允许）。

禁止新版 Handler 按新版 Schema 盲目解释旧 `payload_json`。

## 34.17 Scheduler / Dataset Contract Version

Scheduler 创建的数据采集任务必须关联有效 Dataset Contract。

至少能够追溯：

```text
dataset_code
dataset_contract_version
```

当新 Dataset Contract 成为 Active：

- 新 Schedule 不再创建旧 Contract Job；
- 已存在旧 Job 按 Job Compatibility Contract 处理；
- Coverage 不得因 Contract 切换被静默重置。

## 34.18 Config Schema Versioning

长期 Config 必须拥有独立 Schema Version。

Config Migration 必须：

- 显式版本化；
- 修改前保留可恢复副本；
- 不静默丢弃未知重要配置；
- 不静默改变已有配置语义；
- Secret 迁移遵循 `SECURITY.md`。

配置字段废弃应经过正式 Migration，而不是仅在新代码中忽略旧字段。

## 34.19 Parser / Mapping Compatibility

Parser、Adapter 和 Mapping 升级必须保留版本身份。

Normalized Fact 必须能够追溯到生成它的：

```text
parser_version
mapping_version
raw source
calculation / processing run
```

发现旧 Parser 语义错误时，可以从历史 Raw 重新生成新的 Normalized Version，但不得改写 Raw 本身。

旧 Normalized 结果是否保留、替换或进入 Retention，由对应 Data Contract 决定；任何替代必须可追溯。

## 34.20 Derived Version Publication

Metric、Profit、Forecast 等 Derived Contract 升级采用版本化 Publication。

流程：

```text
new version calculation
↓
staging
↓
coverage validation
↓
semantic validation
↓
publish active version
↓
old rebuildable version enters retention
```

不得先删除当前正式结果再尝试生成新结果。

用户始终读取最后一个已成功发布的完整版本。

## 34.21 已发布历史结果不可被升级改写

以下对象一旦实际发布给用户，其历史语义永久保留：

- Forecast；
- Alert；
- Recommendation；
- 其他未来明确标记为 Published Historical Artifact 的结果。

例如 2026 年实际发布的 Forecast v2，不能因为 2028 年存在 Forecast v8 就改写成“当时采用 v8 会得到的结果”。

新模型可以产生新的回测或重算结果，但必须作为新的版本 / Run 保存。

## 34.22 Read Model Compatibility

Read Model、Cache 和其他 `REGENERATABLE` 数据不是 Source of Truth。

其结构变化默认采用：

```text
create new read model
↓
rebuild from source facts
↓
validate
↓
switch active
↓
old model becomes GC eligible
```

不为可重建缓存承担不必要的复杂长期数据 Migration。

## 34.23 Frontend / Backend Release Compatibility

O3Pilot 是 Modular Monolith，Frontend 与 Backend 默认属于同一个 Application Release Bundle。

v1 不建立长期支持“旧 Frontend + 新 Backend”的独立服务兼容矩阵。

部署时必须保证 Frontend 静态资源与 Backend 属于同一正式 Release。

浏览器缓存的旧静态资源应通过 hashed assets、版本检测或等效机制避免长期调用不兼容的新 API。

## 34.24 Backup Restore Compatibility

Restore 前必须读取 Backup Manifest 并验证：

```text
backup application version
backup database schema version
backup raw store format version
backup config schema version
current application supported ranges
```

如果 Backup 持久化格式高于当前 Application 支持范围：

> Restore may materialize data only through an explicitly supported recovery path; normal runtime must not open it as READY.

Restore 不绕过 Compatibility Contract。

## 34.25 Business Contract Changes Must Be Versioned

Application Upgrade 不得静默改变历史业务口径。

以下变化都必须进入其正式 Version Contract：

- Metric formula；
- Profit formula；
- Finance classification；
- Parser interpretation；
- Mapping rule；
- Seller Cost FX rule；
- Forecast model；
- 其他会改变历史业务结果语义的规则。

“代码升级”本身不能成为历史结果改变而无法解释的理由。

## 34.26 Upgrade Observability

每次正式 Upgrade 至少应拥有 `upgrade_run` 级追溯。

建议记录：

```text
upgrade_run_id
from_application_version
to_application_version
from_database_schema_version
to_database_schema_version
from_raw_store_format_version
to_raw_store_format_version
started_at
finished_at
pre_upgrade_backup_id
migration_count
current_migration
backfill_progress
status
error_code
error_message
```

典型状态：

```text
PRECHECK
BACKING_UP
MIGRATING
BACKFILLING
VALIDATING
SUCCEEDED
FAILED
RECOVERY_REQUIRED
```

用户不一定看到全部内部字段，但系统必须能解释升级停在哪一步。

## 34.27 Compatibility Runtime State

Compatibility 层至少支持：

```text
READY
MIGRATION_REQUIRED
MIGRATING
BACKFILLING
INCOMPATIBLE_NEWER_SCHEMA
RECOVERY_REQUIRED
```

### READY

当前 Application 与所有关键持久化格式兼容，可以正常运行。

### MIGRATION_REQUIRED

持久化版本低于当前 Application 目标版本，存在正式 Migration Path。

### MIGRATING

正在执行阻塞正常运行的 Migration。

### BACKFILLING

Application 已可安全运行，但部分新能力尚未完成历史回填或正式 Publish。

未完成能力必须明确表现为 Pending / Unavailable，不得展示半完成结果。

### INCOMPATIBLE_NEWER_SCHEMA

当前 Application 早于持久化格式，禁止业务写入。

### RECOVERY_REQUIRED

Migration、完整性检查或 Compatibility Validation 出现严重失败，需要进入 Recovery Mode。

## 34.28 正式 Upgrade Pipeline

长期正式升级流程固定为：

```text
Install New Release
↓
Acquire Instance Lock
↓
Compatibility Preflight
↓
Integrity + Capacity Check
↓
Pre-upgrade Last Known Good Backup
↓
STARTUP_ATOMIC Migrations
↓
Start Compatible Runtime
↓
ONLINE_BACKFILL Jobs if required
↓
Validate
↓
Publish
↓
READY
```

如果失败：

```text
Before persistent migration
→ previous release can restart

After incompatible persistent migration
→ restore Pre-upgrade Last Known Good Backup
→ previous release
```

具体安装命令、Release 获取、文件替换、服务重启和用户操作流程由 `DEPLOYMENT.md` 定义。

---


# 35. Portable Instance Backup / Restore Contract

O3Pilot 的长期实例必须能够从一台受支持的 macOS / Linux 主机迁移到另一台受支持主机，而不要求重新抓取已经获得的历史 Ozon 数据，也不要求用户重新上传历史报告、ERP 或物流商原始文件。

正式目标：

> An O3Pilot instance is portable across supported macOS / Linux hosts through a verified backup and restore path.

Portable Restore 恢复的是 **O3Pilot Instance Data**，不是旧主机的完整操作系统环境。

---

## 35.1 Instance Data Boundary

一个可恢复的 O3Pilot Instance 至少由以下持久化数据组成：

```text
O3Pilot Instance
├── SQLite Primary Database
├── Local Raw Store
├── Portable Application Config
├── Portable Encrypted Secrets
├── Backup / Compatibility Manifest
└── Required integrity metadata
```

只复制 `o3pilot.db` 不构成完整实例备份。

原因：Raw Content 已从主 SQLite 物理拆分；如果仅恢复数据库，将失去：

- Raw API Response；
- Webhook Payload；
- Ozon 官方报告原始内容；
- ERP / Seller Data 原始内容；
- 物流商导入原始内容；
- Raw-based Reprocessing；
- 完整 Lineage / Audit 能力。

因此：

> Database Backup != Portable Instance Backup.

---

## 35.2 Backup Classes

O3Pilot 至少区分三类 Backup Purpose。

### NORMAL_BACKUP

用于日常灾难恢复。

包含一个一致性数据库快照，并关联能够恢复该 Snapshot 所需的 Raw / Config / Secret 状态。

### PRE_UPGRADE_BACKUP

用于 Application / Schema / Raw Format / Config Format 升级前建立 Last Known Good。

它必须满足 `Upgrade & Compatibility Contract` 的回滚要求。

### PORTABLE_MIGRATION_BACKUP

用于跨主机迁移或重新安装。

它必须能够在另一台受支持主机上恢复完整 O3Pilot Instance，而不依赖旧主机仍然可访问。

三种用途可以共享同一个底层 Backup Repository 和对象去重机制，但 Manifest 必须记录 Backup Purpose。

---

## 35.3 Backup Repository Model

Backup 不采用“每天重新复制整个 Raw Store”的粗粒度方案。

由于 Raw Store 已采用：

```text
SHA-256
Immutable
Content-addressed
Compressed
Deduplicated
```

Backup Repository 应复用相同思想。

逻辑结构可以表示为：

```text
backup_repository/
├── manifests/
│   └── <backup_id>.json
├── databases/
│   └── <database_snapshot_id>.zst
├── configs/
│   └── <config_snapshot_id>
├── secrets/
│   └── <secret_snapshot_id>.enc
└── raw_objects/
    └── sha256/
        └── <content-addressed objects>
```

具体物理目录名不是 Architecture Contract；Contract 是：

- Snapshot 与对象通过稳定 ID / Hash 引用；
- Raw Object 不因多个 Backup 重复保存相同内容；
- Manifest 能完整描述一次恢复所需要的对象集合；
- Backup Repository 可以位于 Local，也可以拥有 R2 Replica。

---

## 35.4 Backup Manifest

每个正式 Backup 必须拥有不可歧义的 Manifest。

至少记录：

```text
backup_id
backup_purpose
created_at
source_instance_identity
application_version
database_schema_version
raw_store_format_version
config_schema_version
job_payload_compatibility_version / summary
platform_family

database_snapshot_id
database_checksum
raw_manifest_id
raw_object_count / coverage summary
config_snapshot_id
secret_snapshot_id

integrity_status
local_replica_status
remote_replica_status
```

Manifest 应能够回答：

- 这是哪个时间点的 Backup；
- 来自什么 Application / Schema；
- 数据库快照是哪一个；
- 恢复需要哪些 Raw Objects；
- Config / Secret Snapshot 是什么；
- 当前本地与 R2 副本是否完整；
- 当前 Application 是否能够 Restore / Migrate 它。

Manifest 本身也应具有 Checksum / Integrity Validation。

---

## 35.5 Consistent Backup Boundary

Portable Backup 必须形成逻辑一致的恢复点。

创建 Backup 时：

```text
Stop accepting new maintenance / bulk work as required
↓
Allow current short DB transaction to finish
↓
Create consistent SQLite snapshot
↓
Freeze Raw Manifest / required object set for this backup_id
↓
Snapshot portable config
↓
Snapshot encrypted secrets
↓
Write manifest candidate
↓
Validate all required references
↓
Atomic publish manifest
↓
backup_run = SUCCEEDED
```

Raw Object 是 Immutable，因此不要求为了复制全部 Raw 而让业务长时间停机。

一旦本次 `raw_manifest` 已冻结，后续新产生的 Raw Object 属于未来 Backup，不改变已经发布 Backup 的语义。

---

## 35.6 Incremental Raw Backup

Raw Backup 按 Content Hash 去重。

如果某个 Raw Object 已经存在于 Backup Repository：

```text
sha256(object) already present
→ verify / reuse
```

不得因新的 Backup 再复制同一个对象。

因此日常 Backup 增量主要由：

```text
new SQLite snapshot
+
new / changed config snapshot
+
new encrypted secret snapshot when needed
+
new Raw Objects since prior backup
+
new Manifest
```

组成。

这使多年 Raw History 不会导致每次 Backup 都重新复制全部历史内容。

---

## 35.7 Local Backup 与 R2 Replica

正式关系：

```text
Primary Instance
      ↓
Local Backup Repository
      ↓ optional replica
Cloudflare R2
```

R2 是 Off-device Backup Replica，不成为主实例运行依赖。

未配置 R2：

```text
Local Backup / Portable Export / Restore
```

必须完整可用。

配置 R2 后：

```text
Local backup succeeded
↓
replicate missing immutable objects + manifest to R2
↓
verify remote object integrity
↓
remote_replica_status = VERIFIED
```

本地 Backup 成功与 R2 Replica 成功是两个独立状态。

不得因为 R2 暂时不可用而破坏已经成功的 Local Backup。

---

## 35.8 Portable Export without R2

O3Pilot 必须支持不依赖 Cloudflare R2 的跨机器迁移。

可以通过：

- 外接磁盘；
- 局域网；
- `scp` / 等效文件传输；
- 用户选择的其他普通文件传输方式；

移动一个 Portable Backup Repository / Export Package。

Architecture 不绑定具体传输工具。

目标是：

> R2 improves transport and off-device durability; it is not required for portability.

---

## 35.9 Portable Secrets Requirement

真正无缝恢复要求 Ozon Credentials、Performance Credentials、R2 Credentials 与其他未来 Secret 能够跨主机恢复。

因此 O3Pilot Backup 必须支持：

```text
Portable Encrypted Secrets
```

禁止把 Secret 以明文形式写入 Backup Repository。

同时不能只依赖旧主机的：

- macOS Keychain device-bound state；
- Linux machine-bound key；
- 其他无法在新主机恢复的本地硬件绑定密钥；

作为唯一解密条件。

具体加密算法、KDF、Recovery Key / Backup Passphrase、Secret Rotation 和用户恢复流程由 `SECURITY.md` 定义。

Architecture 只规定：

> A verified portable backup must contain or reference a portable encrypted secret state sufficient to restore configured integrations when the authorized user supplies the required recovery secret.

如果用户选择不备份 Secrets，则 Backup 必须明确标记为：

```text
DATA_COMPLETE_BUT_CREDENTIALS_REQUIRED
```

而不能伪装成完全无缝恢复。

---

## 35.10 Runtime Identity Is Not Portable

以下运行时身份不能从旧主机原样继承：

```text
instance_id
process ownership
instance lock
PID
runtime lease
active login session
```

Restore 成功后必须生成新的 Runtime `instance_id`。

旧 Active Session 全部失效。

用户必须在恢复后的实例重新登录。

这是为了防止：

- 旧浏览器 Session 继续被新实例接受；
- 恢复后的 Job 被误认为仍由旧进程拥有；
- Instance Lock / Crash Recovery 状态跨机器污染。

---

## 35.11 Persistent Business Identity Is Portable

与 Runtime Identity 相反，业务与历史身份必须保持稳定。

例如：

```text
shop_id
product internal identity
posting identity
return identity
finance identity
raw_payload identity
sync / calculation lineage
metric / forecast history
```

Restore 后不得因为换了主机而重新生成这些持久化业务 ID。

否则历史关联、Lineage、Forecast Backtest 和跨版本对账都会断裂。

正式规则：

> Host identity may change; persistent business identity must not.

---

## 35.12 Cross-platform Path Portability

Backup Manifest 和数据库不得依赖旧主机绝对路径作为唯一 Raw Locator。

禁止把：

```text
/Users/old-user/O3Pilot/data/raw/...
```

作为不可迁移的长期事实。

持久化 Raw Locator 应使用：

- content hash；
- logical object key；
- repository-relative path；
- 等效平台无关标识。

Restore 时根据新 Data Directory 重新解析物理路径。

因此 macOS → Linux、Linux → macOS 的受支持迁移不会因为路径格式不同而失效。

---

## 35.13 Backup / Restore Compatibility

Portable Restore 必须完整遵守 `Upgrade & Compatibility Contract`。

Restore 开始前至少检查：

```text
backup application version
backup database schema version
backup raw store format version
backup config schema version
current application compatibility manifest
```

存在三种正常路径：

### SAME_COMPATIBLE_VERSION

当前 Application 可以直接打开恢复后的持久化格式。

### RESTORE_THEN_UPGRADE

Backup 比当前目标 Application 旧，但存在正式 Migration Path。

```text
materialize backup
↓
verify backup state
↓
run normal Upgrade Contract
```

### INCOMPATIBLE_NEWER_BACKUP

Backup 格式比当前 Application 新。

必须拒绝进入 READY，直到使用兼容 Application 或明确支持的 Recovery Path。

Restore 绝不能绕过版本检查。

---

## 35.14 Restore Is Staged, Not In-place Destructive

Restore 不直接覆盖当前正在工作的实例数据。

正式流程采用 Staging：

```text
Select backup
↓
Compatibility preflight
↓
Materialize to restore staging area
↓
Verify checksums
↓
Verify SQLite integrity
↓
Verify Raw references / required objects
↓
Verify config / secret state
↓
Validate schema compatibility
↓
Publish restored instance atomically / controlled switch
```

在新恢复状态完全验证前，原 Last Known Good 不得被破坏。

---

## 35.15 Restore Validation

Restore 至少验证：

```text
Backup Manifest integrity
Database checksum
SQLite integrity
Required Raw object presence
Raw object checksum
Config schema
Secret package integrity
Persistent version compatibility
Critical lineage references
```

验证失败：

```text
RESTORE_FAILED
```

不得进入正常 Worker / Scheduler 运行。

系统保留诊断信息并进入 Recovery Surface。

---

## 35.16 Partial / Interrupted Restore

Restore 可能因：

- 断电；
- 网络中断；
- R2 暂时失败；
- 本地空间不足；
- 用户终止；

被中断。

因此 Restore 过程必须：

- 有 `restore_run_id`；
- 可识别已下载 / 已验证 Immutable Object；
- 对大 Raw Repository 支持 Resume；
- 不把半完成 Staging 标记成 Active；
- 重新执行时安全复用已经验证的对象。

已经通过 Hash 验证的 Content-addressed Raw Object 不需要重复下载。

---

## 35.17 Restore from R2

配置 R2 时，新主机可以使用经过认证的 R2 Backup Repository 作为恢复来源。

流程：

```text
Install compatible O3Pilot
↓
Configure / unlock backup access
↓
Read remote backup manifests
↓
Select verified backup
↓
Download database/config/secret snapshots
↓
Materialize required Raw Objects incrementally
↓
Validate
↓
Publish restored instance
```

R2 上只存在部分对象或 Manifest 未验证时，不得将该 Backup 显示为 `FULLY_RESTORABLE`。

---

## 35.18 Lazy Raw Materialization Boundary

为了避免几十 GB Raw 导致迁移过程不必要地阻塞过久，Architecture 允许未来实现 Lazy Raw Materialization，但必须满足严格条件。

只有在：

- SQLite / Config / Secrets 已完整恢复；
- 所需 Raw Object 在可信 Backup Repository 中存在且已验证；
- 用户普通业务查询不依赖尚未本地 materialize 的 Raw；

时，系统才可以先恢复日常查询能力，再后台拉取 Cold Raw。

但是：

- Reprocess；
- Raw Audit；
- 依赖缺失 Raw 的诊断；

必须明确标记 Pending，而不能把 Raw 缺失解释为“原始数据不存在”。

v1 可以选择实现更简单的 Full Materialization；上述规则用于保证未来优化不会改变数据语义。

---

## 35.19 Old Host Split-brain Protection

跨主机迁移后，旧主机可能仍然存在完整实例副本。

由于 O3Pilot 是 Single-machine / Single-primary 产品，不支持两个恢复自同一 Backup 的实例同时作为同一个逻辑 Primary 长期运行。

迁移流程必须明确一个 Cutover Point：

```text
Old Primary
↓
Final portable backup / sync point
↓
Old Primary stops normal scheduling or is retired
↓
New host restores
↓
New Primary starts
```

Architecture 不依赖远程分布式锁来协调两台机器。

如果用户主动重新启动旧实例，它在技术上是另一个独立本地实例，但可能对同一 Ozon 店铺重复采集，因此正式迁移流程必须要求旧 Primary 退出服务。

具体 Cutover 操作进入 `DEPLOYMENT.md`。

---

## 35.20 Cloudflare Tunnel Is Host Deployment State

O3Pilot Instance Backup 不负责复制 `cloudflared` 主机进程本身。

以下属于 Deployment State：

- cloudflared binary；
- systemd / launchd service definition；
- 主机级 Tunnel 运行配置；
- 操作系统防火墙与服务权限。

Restore 后：

```text
O3Pilot Instance Data restored
+
Cloudflare Tunnel configured on new host
```

共同完成对外服务迁移。

是否复用同一 Tunnel / Hostname、如何切换 DNS / Tunnel Route，由 `DEPLOYMENT.md` 定义。

---

## 35.21 OS-specific State Is Not Part of Portable Data

以下默认不属于 Portable Instance Data：

- Python / runtime installation；
- Application executable / release files；
- Homebrew / apt packages；
- systemd / launchd runtime state；
- temporary files；
- OS logs；
- process PID；
- socket / lock files；
- generated cache that can be rebuilt。

新主机应先安装受支持 O3Pilot Release，再执行 Restore。

Backup 不尝试把整个旧操作系统镜像搬到新机器。

---

## 35.22 Restore and Pending Jobs

数据库 Backup 中可能存在：

```text
PENDING
RETRY_WAIT
RUNNING
INTERRUPTED
```

等 Job 状态。

Restore 后：

- 旧 `RUNNING` owner 必须视为无效；
- Job 进入 Startup Crash Recovery；
- Payload 必须通过 Job Compatibility Contract；
- 可安全 Retry 的任务重新调度；
- 无法兼容的任务进入显式 Incompatible / Replacement Flow；
- 不得因为跨机器恢复重复制造不可幂等业务事实。

---

## 35.23 Post-restore Catch-up

Backup 代表过去某一时间点。

新主机恢复后，O3Pilot 必须计算：

```text
backup coverage
↓
current expected coverage
↓
missing intervals / current-state refresh needs
```

然后按 Dataset Contract 执行 Catch-up。

优先级仍遵守：

```text
P0 realtime consistency
P1 expiring / irreplaceable capture
P2 normal incremental sync
P3 historical work
P4 maintenance
```

Restore 不把“Backup 创建之后发生的新业务数据”错误解释为永久缺失。

---

## 35.24 Backup Repository GC

Backup Repository 本身也需要 Retention / GC，但不能简单按文件年龄删除对象。

由于多个 Manifest 可能引用同一个 Content-addressed Raw Object，GC 必须先计算 Reference Reachability：

```text
Retained Backup Manifests
↓
Referenced DB / Config / Secret / Raw Objects
↓
Reachable Object Set
↓
Unreferenced Object becomes GC eligible
```

禁止删除仍被任何 Retained / Last Known Good Backup 引用的对象。

R2 Replica 的 GC 同样遵守 Manifest Reachability。

具体保留多少个日 / 周 / 月 Backup，由 `DEPLOYMENT.md` 的 Retention Policy 定义。

---

## 35.25 Backup Health State

Backup 至少区分：

```text
CREATING
LOCAL_VERIFIED
REMOTE_REPLICATING
REMOTE_VERIFIED
PARTIAL
FAILED
CORRUPT
```

其中 `LOCAL_VERIFIED` 表示本地恢复链完整。

`REMOTE_VERIFIED` 表示 R2 上恢复所需对象也已经验证完整。

不能因为存在一个 Manifest 就声称 Backup 可恢复。

---

## 35.26 Restore Observability

每次正式 Restore 至少记录：

```text
restore_run_id
backup_id
source_type        # local / portable / r2
source_application_version
target_application_version
started_at
finished_at
objects_required
objects_verified
objects_downloaded
database_validation_status
raw_validation_status
config_validation_status
secret_validation_status
compatibility_status
catch_up_status
status
error_code
error_message
```

典型状态：

```text
PRECHECK
MATERIALIZING
VERIFYING
MIGRATING
PUBLISHING
CATCHING_UP
SUCCEEDED
FAILED
RECOVERY_REQUIRED
```

---

## 35.27 Portable Restore Security Boundary

Architecture 不在此定义密码学实现，但规定以下边界：

- Backup 中 Secret 不得明文保存；
- Backup Integrity 与 Secret Confidentiality 是两个不同要求；
- 获取 Backup 文件本身不应等于自动获得所有 Ozon Credentials；
- Authorized Restore 必须存在明确解锁步骤；
- Recovery Secret 不应只存在于待恢复的同一台机器；
- 日志不得输出恢复出的 Secret。

具体实现由 `SECURITY.md` 定义。

---

## 35.28 Portable Instance Restore Pipeline

正式跨主机恢复流程：

```text
Install supported O3Pilot release on new macOS / Linux host
↓
Resolve empty / restore target Data Directory
↓
Acquire Instance Lock
↓
Select Local Portable Backup or R2 Backup
↓
Read Backup Manifest
↓
Compatibility Preflight
↓
Disk Capacity Check
↓
Materialize database / config / encrypted secrets
↓
Materialize or verify required Raw Objects
↓
Checksum + SQLite + Raw integrity validation
↓
Run required Upgrade Migration through normal Compatibility Contract
↓
Generate new runtime instance_id
↓
Invalidate restored active sessions / runtime leases
↓
Publish restored Data Directory
↓
Startup Crash Recovery for persistent Jobs
↓
Dataset Catch-up / Current-state Reconciliation
↓
READY
```

任何关键验证失败：

```text
RECOVERY_REQUIRED
```

不得启动 Scheduler / Worker 继续写业务数据。

---

## 35.29 Portability Acceptance

O3Pilot v1 的 Backup / Restore 实现至少应通过以下演练：

1. 同一台主机：从 Local Backup 恢复到一个全新 Data Directory；
2. macOS → 另一台 macOS 或 Linux → 另一台 Linux 的跨主机恢复；
3. 在可行的测试环境中验证 macOS ↔ Linux 的路径无关性；
4. 仅 Local Backup、完全没有 R2 的恢复；
5. 配置 R2 后从 Remote Verified Backup 恢复；
6. Database + Raw + Config 完整性验证；
7. Portable Encrypted Secrets 的授权恢复；
8. Restore 后旧 Active Session 失效；
9. Restore 后 Persistent Business Identity 不变；
10. Restore 后 Job Crash Recovery 正常；
11. Restore 后能够按 Dataset Contract Catch-up；
12. 中断 Restore 后可以安全重试 / Resume；
13. 缺少 Raw Object、Checksum 错误或 Schema 不兼容时拒绝进入 READY。

只有完成实际 Restore Drill，Backup 才视为“已验证可恢复能力”，而不是只验证“文件成功写出”。

---

# 36. Multi-Shop Runtime Isolation Contract

O3Pilot 从 v1 开始按照多店铺系统运行。

`shop_id` 不只是业务查询维度，也是运行时数据隔离、凭证解析、任务执行、限流、故障隔离与追溯的正式边界。

正式原则：

> Every shop-scoped operation must carry an explicit shop identity from ingress to persistence.

任何模块不得依赖“当前默认店铺”“最近一次选择的店铺”或进程级全局变量隐式决定业务归属。

## 36.1 Shop Scope 必须显式传播

所有 Shop-scoped 工作至少在以下链路中持续携带 `shop_id`：

```text
API / Scheduler / Webhook / Import
↓
Job
↓
Dataset Handler
↓
Source Gateway
↓
Raw Metadata
↓
Parser / Normalization
↓
Normalized Fact
↓
Derived / Read Model
```

任何一个环节丢失 `shop_id` 都视为隔离失败，而不是自动补默认值。

对于明确属于跨店公共参考数据的对象，必须由其 Contract 显式声明为 Global Scope，不能因为缺少 `shop_id` 就默认解释成全局数据。

## 36.2 Credential Resolution 按 Shop 隔离

Ozon Seller API、Performance API 等凭证必须按 Shop 解析。

正式流程：

```text
shop_id
↓
Credential Registry
↓
Resolve typed credential for source_system
↓
Source Gateway
```

业务模块不得：

- 直接传入任意 API Key；
- 从另一个 Shop 复制 Credential Handle；
- 使用进程级“当前 API Key”；
- 在 Seller API 与 Performance API 之间复用未经 Contract 明确允许的凭证对象。

Source Gateway 必须在发送外部请求前完成：

```text
shop scope
+
source system
+
credential scope
+
endpoint allowlist
```

的一致性校验。

具体 Credential 加密、存储介质与访问控制由 `SECURITY.md` 定义。

## 36.3 Source Response 进入系统时立即绑定 Shop

外部响应进入 Raw Metadata 时必须已经拥有明确 Shop Scope。

例如：

```text
raw_payload_id
shop_id
source_system
endpoint
fetched_at
sha256 / raw_object_ref
sync_run_id
```

不得先保存一个“无归属 Ozon Response”，以后再根据商品、订单号或 SKU 猜测属于哪个 Shop。

Webhook 同理：

```text
Webhook ingress
↓
resolve registered shop / endpoint identity
↓
store raw event with shop_id
↓
ACK
↓
create shop-scoped refresh job
```

如果无法可靠确定 Shop：

```text
QUARANTINED / UNMATCHED
```

而不是随机归属。

## 36.4 Normalized Fact 不允许跨 Shop 静默关联

Shop-scoped Normalized Fact 的内部关系必须保持同一 `shop_id`。

例如：

```text
posting(shop=A)
```

不得静默关联：

```text
product(shop=B)
finance_accrual(shop=B)
return_case(shop=B)
```

数据库 Schema 应通过：

- Shop-aware unique key；
- Shop-aware lookup；
- 必要的约束或应用层 invariant；
- Data Quality validation；

降低跨 Shop 错配风险。

如果来源数据存在相同：

```text
product_id
sku
offer_id
posting_number
campaign_id
```

也不能据此假设跨 Shop 唯一。

身份规则继续以 `DATA_MODEL.md` 为准。

## 36.5 Job / Sync / Import 全部保留 Shop Scope

Shop-scoped Job 至少包含：

```text
job_id
job_type
shop_id
payload_version
idempotency_key
```

`sync_run`、Import Batch、Calculation Run 等 Domain Run 也必须保存适用 Shop Scope。

Job 的 Idempotency Key 在适用时必须包含 Shop Identity。

例如逻辑上：

```text
SYNC_FINANCE
+
shop_id
+
coverage
+
dataset_contract_version
```

不能让 Shop A 与 Shop B 的同日期同步因为业务参数相同而错误 Coalesce。

## 36.6 Import 不允许猜 Shop

Ozon 官方报告、ERP、物流商与卖家自有数据导入必须通过明确方式获得 Shop Scope，例如：

- 用户在导入前明确选择 Shop；
- 文件 Contract 内存在可验证的 Shop Identifier；
- Import Adapter 根据已经登记且唯一的映射确定 Shop。

如果存在多个可能 Shop 或证据不足：

```text
UNMATCHED / UNVERIFIED
```

不得通过：

- 相似店铺名；
- SKU 重合；
- 最近使用店铺；
- 当前 UI 选择状态；

静默猜测归属。

## 36.7 跨店商品关系必须显式建立

跨店商品聚合不通过 Source ID 自动合并。

正式路径：

```text
Shop A Product ─┐
                ├─ explicit Seller Catalog Mapping
Shop B Product ─┘
                ↓
      seller_catalog_item
```

只有完成正式 Mapping 的对象才可以进入跨店商品分析。

这与 `METRICS.md` 中跨店聚合规则一致：跨店 Ratio 必须重算分子分母，Money 必须先按照正式 FX Contract 统一 Reporting Currency。

## 36.8 Cross-Shop Read Model 不改变业务身份

跨店 Dashboard / Read Model 可以聚合多个 Shop，但只能读取已经明确隔离的 Shop Fact。

```text
Shop A Facts ─┐
Shop B Facts ─┼─ explicit aggregation → Cross-shop Read Model
Shop C Facts ─┘
```

不得为了查询方便把多个 Shop 的 Source Entity 合并成同一个原始业务对象。

跨店 Read Model 是 Derived / Read Model，不成为 Source of Truth。

## 36.9 Rate Limit 与 Failure Isolation 必须 Shop-aware

Resource Governor 至少能够区分：

```text
source_system
+
shop_id
+
endpoint / dataset group
```

Shop A 因：

- 429；
- Credential 失效；
- Access Denied；
- Endpoint Error；

进入 Backoff / Circuit Breaker 时，不应无条件阻塞 Shop B 的同类同步。

全局 Ozon 服务异常除外；全局异常由 Source-level Circuit Breaker 表达。

## 36.10 Shop 删除 / 停用不得等同于历史删除

未来用户停用某个 Shop 时：

```text
enabled = false
```

只代表：

- Scheduler 不再创建新的日常同步 Job；
- Credential 可以被停用或撤销；

不代表：

- 删除该 Shop 历史订单；
- 删除 Finance；
- 删除 Raw；
- 删除历史 Metric / Forecast；
- 把其数据重新归属其他 Shop。

数据删除若未来成为产品能力，必须拥有独立、显式且可审计的 Data Deletion Contract；v1 Architecture 不把“停用 Shop”实现为数据级联删除。

## 36.11 Backup / Restore 保持 Shop Identity

Portable Restore 必须保持持久化 `shop_id` 与 Shop Business Identity。

新主机：

- 重新生成 Runtime `instance_id`；
- 不重新生成 Shop Identity；
- 不因 Credential 重新解密而创建新的 Shop；
- 不因本地路径变化而重新映射 Shop。

这样 Restore 后所有历史 lineage、Job、Raw、Metric 与 Mapping 仍指向同一业务 Shop。

## 36.12 Multi-Shop Observability

与 Shop 相关的运行指标至少应能够按 `shop_id` 区分：

```text
last_successful_sync
coverage_lag
pending_job_count
retry_wait_count
source_throttle_state
circuit_breaker_state
data_quality_issue_count
```

日志和诊断不得为了可观测性泄露 API Secret。

## 36.13 Multi-Shop Isolation Acceptance

v1 至少验证：

1. 两个 Shop 可以同时存在相同 `product_id / sku / offer_id` 而不发生身份冲突；
2. Shop A 的 Credential 不能被 Shop B Job 使用；
3. Shop A Job 不能写入 Shop B Fact；
4. Raw Payload 在写入时已经绑定明确 Shop；
5. Ambiguous Import 不静默归属 Shop；
6. Idempotency / Coalescing 不跨 Shop 合并；
7. 一个 Shop 的 429 / Credential Failure 不无条件阻断其他 Shop；
8. Cross-shop Product 只有显式 Seller Catalog Mapping 后才聚合；
9. 停用 Shop 不删除历史数据；
10. Portable Restore 后 Shop Identity 与历史 lineage 保持一致。

---

# 37. Configuration Architecture Contract

O3Pilot 的配置必须拥有明确所有权、作用域、版本、验证与迁移语义。

正式原则：

> Every configuration key has exactly one authoritative owner and one declared scope.

禁止同一参数长期同时被 `.env`、SQLite、配置文件、命令行和前端设置以不同优先级隐式覆盖。

具体文件格式、默认路径、OS 权限与 Secret 加密实现由 `DEPLOYMENT.md` / `SECURITY.md` 定义。

## 40.1 配置分为四类

### A. Host Runtime Config

与当前主机运行环境直接相关，例如逻辑上：

```text
data_directory
bind_address
listen_port
service_runtime options
local resource limits
```

特点：

- 主要由 Deployment 层拥有；
- 很多变更需要 Restart；
- 默认不作为 Portable Business Config 原样强制恢复；
- Restore 到另一台主机时允许根据新环境重新解析。

### B. Portable Application Config

属于 O3Pilot 实例业务行为、应随实例迁移的非 Secret 配置，例如逻辑上：

```text
shop enablement
sync policy overrides
retention policy overrides
reporting preferences
backup policy metadata
```

特点：

- Versioned；
- Validated；
- 包含在 Portable Backup；
- Restore 后保持业务语义。

### C. Secret Config

包括：

```text
Ozon Seller API credentials
Performance API credentials
R2 credentials
future external-service secrets
```

特点：

- 不进入普通明文 Config；
- 不写入普通日志；
- Portable Backup 只能以经过授权的 Encrypted Secret State 保存；
- 加密与 Recovery Key 规则由 `SECURITY.md` 定义。

### D. Derived Runtime State

例如：

```text
last_successful_sync_at
current circuit state
job lease
calculated readiness
migration progress
last backup status
```

它们是运行状态，不是用户配置。

不得因为某个 Derived State 存在于 SQLite，就把它当成可以人工修改的 Config。

## 40.2 配置作用域必须显式

正式 Config 至少声明 Scope：

```text
HOST
INSTANCE
SHOP
DATASET
```

Secret Config 还必须声明其适用 Source / Shop Scope。

例如：

```text
SHOP + Seller API credential
SHOP + Performance API credential
INSTANCE + backup retention policy
HOST + listen address
DATASET + sync override
```

禁止依赖目录位置或 Key 名称猜 Scope。

## 40.3 Portability 属性必须显式

每个 Config Key 应声明：

```text
HOST_LOCAL
INSTANCE_PORTABLE
PORTABLE_ENCRYPTED_SECRET
DERIVED_NOT_CONFIG
```

Portable Backup 根据该属性决定是否：

- 直接包含；
- 加密包含；
- 在目标主机重新配置；
- 完全排除。

这样跨 macOS/Linux Restore 不依赖旧主机绝对路径和主机级状态。

## 40.4 单一 Source of Truth

每个 Config Key 只能有一个正式 Owner。

例如如果某项长期 Application Config 由 Persistent Config Store 管理，则：

- Environment Variable 不得在每次启动时静默覆盖它；
- UI 不得维护另一份独立值；
- Worker 不得自己保留第三份配置副本作为事实来源。

允许存在：

```text
bootstrap input
↓
validated import
↓
authoritative config
```

但 Bootstrap Source 不是永久双重 Source of Truth。

## 40.5 Effective Configuration Resolver

所有 Runtime Role 使用统一 Config Resolver 获得已经验证的 Effective Config。

```text
Config Store / Host Bootstrap / Secret Store
↓
Schema Validation
↓
Scope Resolution
↓
Compatibility Check
↓
Effective Config Snapshot
↓
API / Scheduler / Worker / Gateway
```

不得让 API、Worker、Scheduler 各自实现不同的 Config Precedence。

## 40.6 Typed Config 与 Schema Version

所有长期配置必须拥有：

```text
config_schema_version
key
value type
scope
portability class
validation rule
default policy
apply mode
```

Config Schema Versioning 与 Upgrade Contract 的规则一致：

- Migration 显式；
- 不静默丢失未知重要配置；
- 新 App 不可理解的未来 Config 必须 Fail Closed 或进入兼容受限模式；
- 已废弃 Key 通过 Migration 迁移，不在读取时长期保留隐藏兼容逻辑。

## 37.7 Default 必须显式且可版本化

默认值是产品行为的一部分。

禁止：

```text
key missing
→ module A default = 5
→ module B default = 10
```

每个正式默认值必须由 Config Contract 唯一声明。

如果升级改变会影响业务行为的默认值：

- 必须通过版本或 Migration 明确处理；
- 已存在实例不能因为“新代码默认值改变”而静默改变长期业务语义。

## 37.8 Apply Mode

配置至少声明：

```text
HOT_APPLY
RESTART_REQUIRED
NEXT_JOB_ONLY
```

### HOT_APPLY

适用于可以安全原子发布的新配置。

正式过程：

```text
validate candidate
↓
persist new config version
↓
publish effective snapshot
↓
new requests/jobs use new version
```

不得先修改运行中状态，再发现配置无效。

### RESTART_REQUIRED

Host Runtime 或影响底层资源初始化的配置可以要求 Restart。

UI / API 应明确显示：

```text
saved = true
active = false
restart_required = true
```

不能让用户误以为已经立即生效。

### NEXT_JOB_ONLY

某些 Dataset / Job Policy 只影响之后创建或领取的 Job。

已经运行的 Job 保持其启动时记录的 Configuration Version / Execution Contract，避免执行中语义漂移。

## 37.9 Configuration Snapshot 与 Job Reproducibility

影响 Job 结果或数据采集行为的重要配置应能够追溯到：

```text
config_version / config_snapshot_id
```

至少对于：

- Dataset Sync Override；
- Retention / Compaction Policy；
- Calculation Policy；
- Backup Policy；

需要知道执行时使用了哪个正式配置版本。

不要求把所有无关 UI Preference 都复制到每个 Job。

## 40.10 Secret 与普通 Config 严格分离

普通 Config API / Export 不得返回 Secret 明文。

正式原则：

```text
Config metadata
!=
Secret material
```

可以查询：

```text
credential configured = true
credential last_verified_at
credential scope
```

但不能通过普通 Config Read Model 返回完整 Secret。

Secret Rotation、Reveal、Backup Encryption、Recovery Authorization 由 `SECURITY.md` 定义。

## 40.11 配置变更必须可追溯

重要配置变更至少记录：

```text
config_key / config_group
scope
old_version
new_version
changed_at
change_origin
apply_status
```

由于 v1 是 Single User，不需要复杂多用户审批流，但仍需能够回答：

> 当前行为为什么与之前不同？配置何时发生变化？

Secret Audit 不记录 Secret Value 本身。

## 40.12 Invalid Configuration

如果发现：

- Config Schema 不兼容；
- 必填 Key 缺失；
- 值超出 Contract；
- Shop Credential Scope 冲突；
- Portable Config 引用了不存在的资源；

不得通过猜测或使用危险默认值继续。

根据影响范围进入：

```text
CONFIG_INVALID
DEGRADED
RECOVERY_REQUIRED
```

例如某一个 Shop Credential 无效可以只降级该 Shop Source；核心数据库路径无效则必须阻止正常启动。

## 40.13 Restore Configuration Rules

Portable Restore 时：

```text
INSTANCE_PORTABLE
→ restore

PORTABLE_ENCRYPTED_SECRET
→ restore after authorization / unlock

HOST_LOCAL
→ validate or reconfigure on target host

DERIVED_NOT_CONFIG
→ reconstruct from runtime
```

目标主机路径、端口、service manager 状态不得因为旧主机 Config 被盲目覆盖。

## 40.14 Configuration Architecture Acceptance

v1 至少验证：

1. 每个正式 Config Key 有唯一 Owner、Scope 与 Type；
2. API / Worker / Scheduler 使用同一个 Effective Config Resolver；
3. 不存在多个长期 Source of Truth 静默互相覆盖；
4. Portable / Host-local / Secret / Derived State 可以明确区分；
5. Config Schema 可以版本化和迁移；
6. Invalid Config 不通过危险默认值继续运行；
7. `HOT_APPLY / RESTART_REQUIRED / NEXT_JOB_ONLY` 语义明确；
8. Secret 不通过普通 Config API、日志或 Portable Plaintext Export 泄露；
9. Portable Restore 正确恢复 Instance Config，同时重新解析 Host-local Config；
10. 重要配置变更能够追溯到正式 Config Version。

---

# 38. Runtime Health & Readiness Contract

O3Pilot 必须区分“进程仍然活着”“应用可以接受正常业务请求”“部分能力降级”“某个 Dataset 数据已经过期”。

正式原则：

> Liveness, Readiness, Degraded Health and Data Freshness are different signals.

不得使用一个简单 `healthy = true / false` 同时表达全部状态。

## 38.1 四类健康信号

### Liveness

回答：

> O3Pilot 主进程是否仍在正常运行并能够执行自身 Runtime Loop？

Liveness 不依赖：

- Ozon Seller API 当前可用；
- Performance API 当前可用；
- R2 当前可用；
- 某个 Shop Credential 当前有效；

这些外部故障不能直接把进程判定为死亡。

### Readiness

回答：

> 当前实例是否具备安全提供正常业务服务的最低条件？

至少依赖：

- Instance Lock 已获得；
- SQLite 可安全打开且 Schema 兼容；
- 必需 Migration 状态允许运行；
- Data Directory / Raw Store 基本可用；
- 核心 Config 有效；
- 不处于必须人工恢复的严重完整性状态。

### Degraded Health

回答：

> 应用仍可用，但是否有部分能力或数据源不能正常工作？

例如：

```text
Performance API circuit open
Shop A credential invalid
R2 backup replica unavailable
某 Dataset stale
某 Backfill 长期失败
```

应用可以保持 Readable，同时明确标记 Degraded。

### Data Freshness / Dataset Health

回答：

> 每个 Dataset 当前数据覆盖是否符合自身 Dataset Contract？

它依赖：

```text
last_successful_fetch_at
latest_source_business_time
expected coverage
source availability
backfillability
```

不得仅根据“最后一次 HTTP 请求成功”判断数据新鲜。

## 38.2 Runtime State Model

实例级 Runtime State 至少支持：

```text
STARTING
MIGRATING
BACKFILLING
READY
DEGRADED
RECOVERY_REQUIRED
STOPPING
```

语义：

### STARTING

正在完成启动 Pipeline，尚未发布正常 Readiness。

### MIGRATING

正在执行阻塞正常运行的兼容 Migration。

### BACKFILLING

新版本已经可以安全运行，但部分新能力或新字段仍在执行可恢复 Backfill。

允许哪些能力 Ready 必须由 Feature / Compatibility Contract 明确声明，不能把半完成数据伪装成完整数据。

### READY

核心运行条件满足，可以正常提供已声明能力。

### DEGRADED

核心应用仍然可以提供已有安全数据和部分功能，但至少一个非核心依赖、Shop、Dataset 或后台能力异常。

### RECOVERY_REQUIRED

存在可能危及数据正确性或持久化安全的严重问题。

在该状态：

- Scheduler 停止正常创建业务写 Job；
- Worker 不继续执行可能扩大损坏的任务；
- 允许必要的诊断 / Recovery API；
- 不把实例显示成正常 Ready。

### STOPPING

已停止领取新 Job，正在执行 Graceful Shutdown。

## 38.3 Readiness 不以所有外部系统在线为前提

O3Pilot 是 Local-first 数据系统。

例如 Seller API 暂时不可访问时：

```text
Liveness = OK
Readiness = READY
Overall Health = DEGRADED
Seller Dataset = STALE / UNAVAILABLE
Existing Local Data = readable
```

不能因为 Ozon 维护，就让用户连历史订单和已有 Profit 都无法查看。

同理 R2 故障只影响：

```text
remote backup capability
```

不能让没有 R2 依赖的核心 Application 变成 Not Ready。

## 38.4 Recovery Required 的典型条件

以下问题应优先进入 `RECOVERY_REQUIRED` 或等价的安全受限状态：

- SQLite Integrity Failure；
- Application 无法理解更高版本 DB Schema；
- Migration 处于无法自动恢复的半完成状态；
- 必需 Raw / Metadata 关系出现严重不可解释损坏；
- Core Config 无法安全解析；
- Data Directory 不可写且系统继续写入会造成风险；
- Restore Validation 未通过却试图发布目标实例。

外部 429、单 Shop Credential Failure 或单 Dataset Stale 通常不属于该级别。

## 38.5 Dataset Health 不重新定义业务 Metric Status

Runtime Dataset Health 用于描述数据采集与覆盖状态。

可以表达例如：

```text
FRESH
LAGGING
STALE
PARTIAL
UNAVAILABLE
UNVERIFIED
```

但最终 Metric 的：

```text
VALID
PARTIAL
UNAVAILABLE
STALE
ESTIMATED
...
```

仍由 `METRICS.md` 定义。

Architecture 只提供可靠的：

- Fetch Age；
- Coverage Lag；
- Coverage / Sync State；
- Source Health；

供 Metric Contract 判断。

## 38.6 Component Health

系统内部至少能够独立观察：

```text
SQLite
Raw Store
API Runtime
Worker Runtime
Scheduler Runtime
Job Queue
Seller API Gateway
Performance API Gateway
Ozon XAPI Gateway
Backup Local
Backup R2 Replica
Disk Capacity
```

每个 Component Health 至少应具有：

```text
status
reason_code
last_checked_at
last_success_at (when applicable)
```

避免只有自由文本错误，无法自动诊断。

## 38.7 Shop / Source / Dataset 健康必须可分层

健康状态应支持类似：

```text
Application
└── Shop A
    ├── Seller API
    │   ├── Orders
    │   ├── Finance
    │   └── Inventory
    └── Performance API
        └── Campaign SKU Daily
```

这样可以表达：

```text
Shop A Orders = FRESH
Shop A Performance = STALE
Shop B = READY
Overall App = DEGRADED
```

而不是把所有问题压缩成一个红灯。

## 38.8 Health Reason Code

核心健康状态应使用稳定 Reason Code，例如逻辑上：

```text
DB_INTEGRITY_FAILED
DB_SCHEMA_TOO_NEW
CONFIG_INVALID
DISK_SPACE_CRITICAL
SOURCE_RATE_LIMITED
SOURCE_ACCESS_DENIED
SOURCE_CIRCUIT_OPEN
DATASET_COVERAGE_LAG
BACKUP_REMOTE_FAILED
RAW_OBJECT_MISSING
```

用户展示可以提供本地化说明，但内部诊断、测试和自动恢复逻辑不能依赖错误字符串匹配。

## 38.9 Startup Pipeline 与 Readiness Publication

启动流程：

```text
STARTING
↓
Acquire Instance Lock
↓
Config / DB / Compatibility / Raw / Disk checks
↓
Migration / Crash Recovery
↓
initialize Worker / Scheduler / API
↓
publish READY or DEGRADED
```

在最低安全条件满足前，不得提前发布 `READY`。

发生严重错误：

```text
RECOVERY_REQUIRED
```

而不是让 API 表面返回 200、后台持续失败。

## 38.10 BACKFILLING 的部分可用性

大型 Online Backfill 期间可以让 O3Pilot 提前恢复安全功能，但必须区分：

```text
Application Readiness
Feature Readiness
Dataset Readiness
```

例如新 Metric 依赖尚未完成 Backfill：

```text
Historical Orders = READY
New Metric v2 = BACKFILLING
```

不能将该 Metric 以 0 或部分历史伪装成完整结果。

## 38.11 Health 不触发破坏性自动恢复

Health System 可以：

- 标记状态；
- 触发安全 Retry；
- 打开 Circuit Breaker；
- 暂停低优先级任务；
- 进入 Recovery Mode；

但不能仅根据一个 Health Check 失败就自动：

- 删除数据库；
- 删除 Raw；
- Restore 旧 Backup；
- 重置 Config；
- 清空业务表。

破坏性恢复必须遵守正式 Recovery / Restore Contract。

## 38.12 Process Manager Integration

systemd / launchd 负责主进程生命周期。

Process Manager 主要依据：

```text
process exit / liveness
```

进行重启管理。

不得因为 Ozon API 暂时不可达而让 O3Pilot 主进程不断 Crash / Restart。

Cloudflare Tunnel 可以将 HTTP 请求转发到 O3Pilot，但 Tunnel 可达也不等于 Application Ready；实际 UI / API 应读取 O3Pilot 自身 Readiness State。

## 38.13 Health Observability

至少记录：

```text
runtime_state
runtime_state_changed_at
component_status
component_reason_code
shop/source/dataset health
coverage lag
last successful sync
recovery_required reason
```

重要状态变化应该形成结构化事件，以便回答：

> 系统什么时候开始 Degraded？哪个 Shop / Dataset 导致？何时恢复？

## 38.14 Runtime Health Acceptance

v1 至少验证：

1. Ozon Seller API 断网时 Liveness 仍为 OK，已有本地数据可读；
2. 单个 Shop Credential Failure 只降级对应 Shop / Source；
3. R2 不可用不使核心应用 Not Ready；
4. SQLite Integrity Failure 不发布 READY，而进入 Recovery Required；
5. DB Schema Too New 时旧 Application Fail Closed；
6. Dataset Stale 可以独立于 Application Liveness 表达；
7. `READY / DEGRADED / RECOVERY_REQUIRED` 具有稳定 Reason Code；
8. Online Backfill 可以表达 Feature / Dataset 部分未 Ready；
9. Process Manager 不因外部 Source Failure 形成无意义重启循环；
10. Health Check 不执行破坏性自动修复。

---

# 39. v1 Architecture Acceptance

O3Pilot v1 Architecture 至少满足：

1. macOS / Linux 原生安装，不依赖 Docker；
2. Cloudflare Tunnel 可以作为正式公网访问路径；
3. 单用户、单 Active Session；
4. 同一 Data Directory 只允许一个 Primary Instance；
5. SQLite WAL 作为正式长期主数据库；
6. SQLite 采用大数据量查询与写入性能基线；
7. Raw Content 与 SQLite 物理分离；
8. Raw Store 支持 SHA-256、Immutable、Compression、Deduplication；
9. R2 Backup 能力在 v1 代码中存在，但配置 R2 可选；
10. 无 R2 时系统完整正常运行；
11. Ozon Read-Only 由 Explicit Allowlist 强制；
12. 不提供业务层任意 Ozon Endpoint 调用能力；
13. 每个 Shop-scoped 操作从入口到持久化始终携带明确 `shop_id`；
14. Credential、Job、Raw、Normalized Fact、Rate Limit 与 Failure Isolation 均保持 Shop Scope；
15. 跨店商品聚合只能通过显式 Seller Catalog Mapping，不通过 Source ID 自动合并；
16. API、Worker、Scheduler 职责分离但运行于同一主进程；
17. 所有长任务通过持久化 Job 执行；
18. Job 支持 Idempotency、Retry、Backoff、Priority、Coalescing、Crash Recovery；
19. Webhook 作为 Change Signal，最终状态通过 API Readback；
20. Scheduler 支持 Coverage-aware Catch-up；
21. 任务系统具有 Resource Governance、Adaptive Rate Limit 和 Circuit Breaker；
22. SQLite Bulk Write 使用短事务和受控单后台 Writer；
23. Core Source Fact 不因年龄自动删除；
24. Inventory 采用 90 天 Intraday + Daily 永久历史；
25. Raw v1 默认长期保留；
26. Superseded Rebuildable Derived 默认 90 天；
27. Detailed Job History 默认 180 天；
28. Verbose Debug Log 默认 30 天；
29. 已发布 Forecast / Alert / Recommendation 历史不可被新版本改写；
30. Backup、Compaction、Derived Rebuild 采用 Validate-before-Publish / Delete；
31. Upgrade 使用 Compatibility Manifest、不可变 Migration 与 Pre-upgrade Last Known Good；
32. 新持久化格式遇到旧 Application 时 Fail Closed；
33. Portable Backup 同时覆盖 SQLite、Required Raw、Portable Config 与 Encrypted Secrets；
34. 无 R2 时可以完整执行跨主机 Portable Restore；
35. R2 作为可选 Off-device Replica 支持远程 Restore；
36. Restore 保持 Persistent Business Identity，但生成新的 Runtime Instance Identity 并撤销旧 Session；
37. Restore / Upgrade 失败不得把半完成状态发布为 READY；
38. Configuration 具有单一 Owner、显式 Scope、Typed Schema、Portability Class 与版本；
39. Host Runtime Config、Portable Application Config、Secret Config 与 Derived Runtime State 明确分离；
40. API、Worker、Scheduler 使用统一 Effective Configuration Resolver，重要配置变化可追溯；
41. Runtime Health 明确区分 Liveness、Readiness、Degraded Health 与 Dataset Freshness；
42. 单个外部 Source / Shop 故障不会错误地把整个进程判定为死亡；
43. 严重完整性问题进入 Recovery Required / Degraded Mode，不执行破坏性自动修复；
44. 系统具备 Disk Capacity Protection；
45. 单个外部数据源失败不会导致整个应用已有数据不可查询；
46. 核心查询与任务具有长期大数据量性能回归测试；
47. 所有重要同步、计算、升级、恢复、备份、配置、健康状态和数据质量事件可追溯。

---

# 40. 与其他文档的关系

## 40.1 PRODUCT.md

定义：

```text
O3Pilot 是什么，以及允许做什么。
```

ARCHITECTURE 不扩大 PRODUCT 的产品边界。

## 40.2 DATA_SOURCES.md

定义：

```text
数据从哪里来，以及当前能可靠获得什么。
```

ARCHITECTURE 负责如何安全、持续、可恢复地采集这些数据。

## 40.3 DATA_MODEL.md

定义：

```text
业务 Fact 和实体如何保存与关联。
```

ARCHITECTURE 定义这些 Fact 的物理运行分层、生命周期、任务与存储拓扑。

## 40.4 METRICS.md

定义：

```text
Fact 如何形成 Metric / Profit / Forecast Contract。
```

ARCHITECTURE 必须保证 METRICS 所需历史粒度能够持续采集、重算和追溯。

## 40.5 SECURITY.md

后续定义：

- Session 实现；
- Password Hash；
- Cookie；
- CSRF；
- Secret / API Key；
- Backup Secret；
- Portable Encrypted Secrets；
- Recovery Key / Backup Passphrase；
- Restore authorization；
- Cloudflare Trust Boundary；
- 日志敏感信息；
- 文件权限与本地数据保护；
- Secret Config 的加密、Reveal、Rotation 与授权边界。

## 40.6 DEPLOYMENT.md

后续定义：

- macOS / Linux 安装；
- systemd / launchd；
- Cloudflare Tunnel 配置；
- Data Directory；
- Backup / Restore；
- Portable Migration / Host Cutover；
- Upgrade / Rollback；
- R2 Restore 操作；
- 运维流程；
- Host Runtime Config 的文件/环境变量/启动参数具体载体；
- systemd / launchd 对 Liveness / Readiness 的实际集成方式。

---

# 41. 核心架构原则

**首版允许少功能，但核心架构不采用明确会被废弃的临时实现。**

**SQLite 是正式长期数据库，不是临时过渡数据库。**

**主 SQLite 服务高性能查询，Raw 原始内容进入独立 Local Raw Store。**

**Source Fact 是长期数据资产，可重算结果不是原始事实。**

**不可补回的数据优先采集，不以未来还能重新获取为假设。**

**正常运行增量计算，完整重建只在明确需要时执行。**

**任何网络请求和大型计算都不得长期占用 SQLite 写事务。**

**后台任务必须持久化、可重试、可恢复、可解释。**

**任务积压通过优先级、合并、限流和资源治理处理，不通过无限增加并发处理。**

**Webhook 负责发现变化，API 负责确认最终状态。**

**Read Model 用于性能，不成为 Source of Truth。**

**删除必须发生在替代数据成功生成并验证之后。**

**备份必须保留 Last Known Good。**

**完整实例迁移不能只复制 SQLite，必须覆盖 Required Raw、Portable Config、Encrypted Secrets 与 Compatibility Manifest。**

**Backup 必须通过真实 Restore Drill 证明可恢复，而不仅是证明文件写出成功。**

**跨主机 Restore 保持业务身份，重新生成 Runtime Identity，并使旧 Active Session 失效。**

**R2 是可选异地备份，不是运行时数据库或必须依赖。**

**单个来源失败不能使已有数据不可用。**

**严重完整性异常宁可进入 Recovery Mode，也不自动做破坏性修复。**

**多店铺隔离是运行时边界，不允许通过隐式默认 Shop 或 Source ID 自动跨店合并。**

**每个配置项只有一个正式 Owner；Portable、Host-local、Secret 与 Runtime State 必须分开。**

**进程存活、应用可用、部分降级与 Dataset 新鲜度必须使用不同健康信号表达。**

**O3Pilot 永远只读 Ozon。**
