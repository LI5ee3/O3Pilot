# O3Pilot — ARCHITECTURE.md

> Version: 1.1  
> Status: Active Architecture Baseline  
> Updated: 2026-09-04  
> Applies to: O3Pilot

# 1. 文档目的

`ARCHITECTURE.md` 定义 O3Pilot 的长期技术架构边界。

本文件负责回答：

- O3Pilot 以什么运行形态工作；
- API、Worker、Scheduler 如何协作；
- 外部数据如何安全进入系统；
- Raw、Normalized、Derived 如何分层；
- SQLite 与 Raw Store 如何共同构成长期本地数据层；
- 后台任务如何持久化、重试、恢复和调度；
- Webhook 如何与 Seller API Readback 配合；
- 多店铺如何保持严格 Scope；
- Backup / Restore / Upgrade 如何保证持久数据不被破坏；
- 系统如何在外部数据源故障时继续安全提供已有数据。

本文件不重新定义：

- 产品能力与 Feature Phase：由 `PRODUCT.md` 定义；
- Endpoint 验证事实、分页、时间窗口和 Backfillability 证据：由 `DATA_SOURCES.md` 定义；
- 实体、字段、主键与 Source Lineage Schema：由 `DATA_MODEL.md` 定义；
- Metric 公式与口径：由 `METRICS.md` 定义；
- Session、Secret、认证与密码学：由 `SECURITY.md` 定义；
- 安装目录、OS 支持矩阵、服务配置与具体 Backup 操作：由 `DEPLOYMENT.md` 定义；
- 页面、交互和视觉：由 `DESIGN.md` 定义。

---

# 2. 核心架构原则

O3Pilot v1 架构基线：

```text
Modular Monolith
Single-Machine Native
Single User
SQLite as Long-Term Primary Database
Automatic Raw Stored Outside SQLite
Raw → Normalized → Derived
Reprocessable by Design
Read-Only toward Ozon
Persistent Jobs
Shop First
Incremental by Default
Recoverable by Design
Simple Infrastructure
```

首版允许功能不完整，但已经进入正式架构的持久数据和身份规则不得依赖明显会被推翻的临时方案。

同时：

```text
Production-grade
!=
Pre-build every future platform
```

没有真实需求证据的通用调度平台、配置平台、资源治理平台、兼容状态机和多层抽象不得因为“以后可能需要”提前进入 v1。

---

# 3. 运行边界

O3Pilot 是：

```text
Single-machine
Native-installed
Self-hosted
```

正式运行环境：

```text
macOS
Linux
```

具体 OS / Architecture 支持矩阵由 `DEPLOYMENT.md` 定义。

O3Pilot 不把以下方案作为 v1 正式架构：

```text
Docker
Kubernetes
Microservices
Remote Primary Database
Redis
RabbitMQ
Kafka
Celery
External Job Broker
Cloudflare R2 as Primary Storage
```

公网访问通过：

```text
Cloudflare Tunnel
```

O3Pilot 应只监听本地安全入口；具体地址与服务管理由 `DEPLOYMENT.md` 定义。

Cloudflare R2：

```text
Optional Backup Replica Only
```

没有 R2 时系统必须完整运行。

---

# 4. Modular Monolith 与 Runtime

O3Pilot 使用一个模块化单体应用。

一个正式实例使用一个主应用进程，包含三个逻辑 Runtime Role：

```text
O3Pilot Process
├── API Runtime
├── Worker Runtime
└── Scheduler Runtime
```

三者不是三个必须独立部署的应用服务。

## 4.1 API Runtime

负责：

- Authenticated application API；
- Business Query；
- Local Settings / Import actions；
- Webhook ingress；
- Runtime status exposure。

API Request 不执行长时间 Historical Sync、Reprocessing 或大型 Backup。

## 4.2 Worker Runtime

负责执行持久化 Job：

- Sync；
- Reconciliation；
- Reprocessing；
- Import processing；
- Backup；
- Maintenance。

## 4.3 Scheduler Runtime

负责根据 Dataset Acquisition Policy 与 Backfillability：

- 创建到期 Job；
- 发现 Coverage Gap；
- 优先安排不可回补 / 短窗口采集；
- 在重启后恢复必要 Catch-up。

Scheduler 不保存另一套业务事实；Job / Sync Run / Raw Capture 才是持久状态。

---

# 5. Source Adapter 与 Ozon 只读边界

O3Pilot 只允许通过受限 Source Adapter 访问外部数据。

正式 Source：

```text
Ozon Seller API
Ozon Performance API
Ozon Exchange Rate API
Ozon Webhook
Ozon Official Report Import
Seller-owned Data Import
```

业务模块不得获得：

```text
arbitrary Ozon URL client
arbitrary endpoint path
arbitrary HTTP method
```

Ozon 外部调用必须满足：

```text
Default Deny
+
Explicit Read Allowlist
+
Typed / Restricted Source Adapter
+
Outbound Host Allowlist
```

即使凭证具有写权限，O3Pilot 也不得调用 Ozon 商品、价格、库存、订单、推广或配置写接口。

Endpoint 是否已验证、具体 HTTP Method、Pagination、Empty Result、Time Window 与 Known Boundary 由 `DATA_SOURCES.md` 决定。

---

# 6. Dataset Acquisition Contract

Architecture 只定义 Dataset 调度所需的最小 Contract。

每个自动 Dataset 至少表达：

```text
dataset_code
source_endpoints
acquisition_policy
coverage_freshness
backfillability
priority
```

`acquisition_policy` 使用 Batch 0 冻结枚举：

```text
REQUIRED_CONTINUOUS
PERIODIC_SNAPSHOT
BACKFILLABLE_SYNC
ON_DEMAND
MANUAL
TBD
```

`priority` 只需要表达：

```text
CRITICAL_CAPTURE
NORMAL
```

> Terminology: `CRITICAL_CAPTURE / NORMAL` supersedes the earlier draft vocabulary `REALTIME / NORMAL` and is the only formal priority naming from Batch A.1 onward.

其中 `CRITICAL_CAPTURE` 仅用于不可回补或极短回补窗口的数据。

以下内容不在 Architecture 再定义第二份：

- Endpoint status；
- Endpoint pagination；
- Endpoint retention；
- Parser field mapping；
- Normalized Schema；
- Metric formula。

---

# 7. 数据处理模型

正式逻辑数据流：

```text
Source Adapter
      ↓
Raw
      ↓
Normalized
      ↓
Derived
      ↓
Read Model
```

`Read Model` 是按真实查询需求建立的可选性能层，不是 Source of Truth。

API 位于数据层之上，不作为新的数据处理层。

## 7.1 Raw

保存自动来源的原始事实。

## 7.2 Normalized

把 Source-specific 结构转换成稳定业务事实。

Data Quality 与 Reconciliation 属于 Normalized 阶段的操作，不单独形成新的架构层。

## 7.3 Derived

保存可重新计算的：

- Metric result；
- Aggregate；
- Profit / Forecast / Alert / Recommendation（进入对应 Feature Phase 后）。

Derived 永远不能覆盖 Source Fact。

## 7.4 Time Semantics

系统必须区分：

```text
Source Event Time
Source Business Time / Date
Fetched / Observed At
Coverage Start / End
Imported At
Calculated At
Published At
```

系统内部 Timestamp 必须具有明确时区语义，并以 UTC 作为持久化基准；原始 Source Time / Offset 在有业务意义时必须可追溯。

展示时区由产品与设计规则决定，但展示转换不得覆盖或丢失原始时间语义。

正式原则：

```text
Schedule Time
!=
Data Complete Time
```

Scheduler 只决定：

```text
什么时候尝试采集
```

Dataset Contract 的 `coverage_freshness` 决定：

- 什么时间范围应该存在；
- 什么条件下认为 Coverage Complete；
- 延迟数据何时需要再次回补。

Backfillability 决定：

```text
缺失时间范围还能否补回
```

具体 Business Date 归属、事实时间字段与 Metric 时间口径由 `DATA_MODEL.md` / `METRICS.md` 定义。

---

# 8. SQLite

SQLite 是 O3Pilot 的正式长期 Primary Database。

基线：

```text
SQLite
WAL
Foreign Keys
Busy Timeout
Short Transactions
Local Persistent Filesystem
```

禁止：

```text
SQLite over NFS
SQLite over SMB
SQLite over WebDAV
SQLite over synchronized cloud-drive folder
R2-mounted SQLite
```

## 8.1 Query-driven Index

索引由真实 SQL 路径决定。

自然键唯一性与查询性能索引必须分开考虑：

```text
PRIMARY KEY / UNIQUE
→ identity correctness

INDEX
→ query performance
```

不机械创建所有可能的复合索引。

## 8.2 Pagination

大型表 API 默认使用稳定 Keyset Pagination；只有小型或明确适合的查询才使用大 Offset。

## 8.3 Write Transaction

网络请求、文件 IO、大型 JSON Parser、大型计算不得长期占用 SQLite 写事务。

写入流程应：

```text
prepare outside transaction
→ short transaction
→ commit
```

## 8.4 Incremental by Default

正常运行优先：

```text
incremental sync
incremental aggregate
incremental publication
```

Full Rebuild 只在显式 Repair / Reprocess 场景执行。

---

# 9. Raw Store

Automatic Raw Payload Body 与 SQLite 物理分离。

正式模型：

```text
Automatic Source Raw
=
Local Immutable Content-addressed File Store
+
SQLite Raw Capture Metadata
```

Raw Payload Body 只有一个长期物理存储路径：

```text
Local Raw Object Store
```

不使用：

```text
small payload → SQLite
large payload → filesystem
```

的双存储模式。

## 9.1 Raw Capture 与 Content Object

必须区分：

```text
Raw Capture
=
某 Shop 在某一时间的一次真实 Observation

Raw Content Object
=
一份不可变原始字节内容
```

相同 Payload 的两次采集仍是两个 Capture，但可以引用同一个 Content Object。

## 9.2 `raw_ref`

业务 Fact 的：

```text
raw_ref
```

指向：

```text
Raw Capture Metadata
```

而不是：

```text
filesystem path
SHA-256 directly
JSON body
```

关系：

```text
Normalized Fact
→ raw_ref
→ raw_capture
→ content object locator/hash
→ Raw Object
```

## 9.3 Write Order

Raw durable write 必须先于 Normalized publication。

正式顺序：

```text
write temp object in Raw filesystem
→ flush/fsync
→ hash
→ atomic rename
→ short SQLite transaction for raw_capture
→ parse / normalize
```

正常协议允许：

```text
orphan Raw Object
```

但不允许正常完成状态存在：

```text
raw_capture → missing Raw Object
```

发现后者必须视为 Integrity Error。

## 9.4 Compression

Compression 是 RawStore 实现细节，不是业务身份。

Raw Content Identity 基于：

```text
original raw bytes SHA-256
```

如果存储层压缩，Metadata 必须知道 Storage Encoding，Parser 不得依赖文件扩展名猜测。

## 9.5 Retention

v1 对 Automatic Raw 默认：

```text
No age-based automatic deletion
```

尤其不可回补 / 短窗口 Raw 不得自动删除。

未来只有在明确 Retention Contract 证明可以安全删除时才允许 GC。

---

# 10. Manual Import

Manual Upload 原文件不属于 Raw Store。

流程：

```text
Upload
↓
Staging
↓
Validate
↓
Dataset-specific Parser
↓
Structured Business Facts + Import Lineage
↓
Delete uploaded source file
```

不得长期保存：

```text
original CSV/XLS/XLSX
full Base64
opaque future-phase file
```

P0 提供通用 Import Infrastructure。

某 MANUAL Dataset 的 `Acquisition Phase` 表示从该阶段开始必须正式支持其：

```text
Parser
Validation
Normalized Persistence
```

如果未来 Phase 的外部报告存在不可接受的历史保留风险，必须显式提前该 Dataset 的最小 Import Adapter，而不是长期保存未知原文件。

---

# 11. Webhook

Webhook 是：

```text
Change Signal
```

不是最终业务事实。

流程：

```text
Webhook Request
↓
basic request validation
↓
durable Raw Capture
↓
ACK
↓
create/coalesce shop-scoped refresh Job
↓
Seller API Readback
↓
Normalized Current State
```

Webhook：

```text
负责时效
```

Seller API Readback / Reconciliation：

```text
负责最终状态校准
```

Webhook 重复、乱序、延迟或缺失不能被静默当成最终状态。

---

# 12. Reprocessing

Raw 必须允许未来 Parser 修正后重新处理。

最小关系：

```text
Raw Capture
↓
Parser Version
↓
Normalized Fact
↓
Derived Version
```

Parser 通过 RawStore Interface 读取原始字节，不直接依赖：

```text
固定目录
固定文件扩展名
固定压缩算法
```

重新处理新版本不得静默改写已经发布的历史业务含义。

需要替换旧 Derived 时：

```text
calculate new version
→ validate
→ publish atomically
→ old rebuildable version becomes GC-eligible
```

---

# 13. Persistent Job System

后台长任务必须持久化。

最小 Job Contract：

```text
job_id
job_type
shop_id
status
payload
idempotency_key
attempt_count
next_retry_at
last_error
created_at
started_at
finished_at
```

正式 Status：

```text
PENDING
RUNNING
RETRY_WAIT
SUCCEEDED
FAILED
```

Job 是执行生命周期，不替代长期业务 Run / Lineage。

## 13.1 Durable State

以下状态转换必须及时落库：

```text
PENDING → RUNNING
RUNNING → RETRY_WAIT
RUNNING → SUCCEEDED
RUNNING → FAILED
```

进入 `RETRY_WAIT` 时：

```text
status
attempt_count
next_retry_at
last_error
```

必须在短事务中一起持久化。

不得为了减少 SQLite 写入把关键重试计划主要留在进程内存。

## 13.2 Idempotency / Coalescing

Shop-scoped Job 使用：

```text
(job_type, shop_id, idempotency_key)
```

表达幂等与 Coalescing。

重复 Webhook / Scheduler Trigger 不应产生无限重复 Job。

## 13.3 Priority

只有：

```text
CRITICAL_CAPTURE
NORMAL
```

`CRITICAL_CAPTURE` 用于：

- 不可回补数据；
- 极短回补窗口数据；
- 必须尽快完成的 durable capture。

普通历史回补、用户计算、Backup 与 Maintenance 不建立 P0-P4 通用层级。

## 13.4 Polling

Worker 不做固定毫秒级高频数据库扫描。

无可执行 Job 时使用 bounded backoff。

单进程可以使用内存 wake-up signal 降低延迟，但 SQLite 始终是 Job Durable State Source of Truth。

---

# 14. Crash Recovery

一个 Data Directory 同一时刻只允许一个 Primary O3Pilot Process。

启动时必须获得 OS-level exclusive instance lock。

在 Single Primary Instance 模型下：

```text
startup 时仍为 RUNNING 的 Job
=
上一次进程中断留下
```

恢复流程根据：

```text
idempotency
attempt_count
job type retry policy
```

把它们转换为：

```text
RETRY_WAIT
或
FAILED
```

不需要 `owner_instance_id` / `execution_id` 通用分布式 Job Claim 平台。

任何外部副作用型动作都必须通过幂等设计避免 Crash Retry 产生重复业务事实。

Ozon 侧永远只读，因此 O3Pilot 不存在通过 Job Retry 重复修改 Ozon 的风险。

---

# 15. Scheduler 与 Catch-up

Scheduler 根据：

```text
Acquisition Policy
Backfillability
Coverage
Freshness
```

决定下一次采集。

基本规则：

### REQUIRED_CONTINUOUS

漏采风险最高，优先创建 `CRITICAL_CAPTURE`。

### PERIODIC_SNAPSHOT

按当前状态 Snapshot cadence 采集；关机期间不存在的 Snapshot 不伪造。

### BACKFILLABLE_SYNC

重启后合并缺失 Coverage，在来源允许的窗口内补齐。

### ON_DEMAND

只有明确请求 / 对象需要时执行。

Scheduler 不需要再维护一套独立的通用 Catch-up Policy 枚举。

Batch 0 R1 要求对未验证历史窗口继续补证；如果发现窗口有限，应提前 Acquisition Phase 或提高 cadence。

---

# 16. Resource & Source Protection

v1 只保留有直接正确性价值的资源约束：

```text
bounded concurrency
source-aware rate limiting
retry with backoff
SQLite short transactions
CRITICAL_CAPTURE priority
disk free-space protection
```

不建立通用：

```text
CPU weight
Disk weight
Global Resource Governor
Multi-level Circuit State Platform
```

如果真实压力测试证明需要，再按证据引入。

外部 Source 连续失败时，应降低重试频率并隔离该 Source / Shop，不能拖垮其他已可用来源。

---

# 17. Multi-Shop Runtime Isolation

O3Pilot 从 v1 起是 Multi-Shop。

正式不变量：

## 17.1 Explicit Shop Scope

所有 Shop-scoped ingress 必须显式解析 `shop_id`。

链路中：

```text
API / Scheduler / Webhook / Import
→ Job
→ Source Adapter
→ Raw Capture / Import Lineage
→ Normalized Fact
→ Derived / Read Model
```

不得丢失 Shop Scope。

## 17.2 Credential Scope

Seller API / Performance API Credential 按：

```text
shop_id + source_system
```

解析。

不得存在“当前全局 API Key”。

## 17.3 No Guessing

不得通过：

```text
SKU
Product ID
Order Number
Posting Number
```

猜测 Shop。

来源无法可靠解析 Shop 时必须拒绝或进入显式 Unmatched/Quarantine 流程。

## 17.4 Cross-Shop Identity

同一个 Ozon Source ID 不能在不同 Shop 间自动合并。

Cross-shop Product Relationship 只能通过显式 Seller Catalog Mapping 建立。

---

# 18. Data Quality 与 Reconciliation

O3Pilot 必须显式区分：

```text
0
missing
unavailable
unverified
failed
not applicable
```

Data Quality 与 Reconciliation 在 Normalized 阶段执行。

至少支持发现：

```text
unexpected empty result
missing required field
duplicate source object
sync gap
source mismatch
unknown currency
raw integrity error
```

来源冲突不得静默覆盖。

典型 Reconciliation：

```text
Webhook ↔ Seller API
API ↔ Official Report
Order ↔ Finance
Ozon Fact ↔ Seller-owned Data
```

具体规则由对应 Domain / Data Source Contract 定义。

---

# 19. Data Lifecycle

年龄本身不是删除 Source Fact 的充分理由。

## 19.1 Raw

Automatic Raw v1 默认长期保留，尤其不可回补 Raw。

## 19.2 Normalized

历史业务事实默认保留。

Current State 与 History 必须区分，不能用 Current State 覆盖历史事实。

## 19.3 Derived / Read Model

可重建 Derived / Read Model 可以在新版本已经：

```text
生成
验证
发布
```

之后回收旧版本。

删除流程必须始终：

```text
generate replacement
→ validate
→ publish
→ mark old rebuildable data GC-eligible
```

不得先删后建。

---

# 20. Disk Protection

长期需要监控：

```text
SQLite size
Raw size
Backup repository size
Free disk
Daily growth
```

磁盘接近安全下限时：

1. 暂停新的大型 Historical Backfill / Reprocess；
2. 暂停非关键 Maintenance；
3. 保留 CRITICAL_CAPTURE；
4. 发出明确告警；
5. 不自动删除不可替代 Raw / Source Fact。

不在 Architecture 固定通用容量阈值；实际阈值由 Deployment / Runtime Configuration 决定。

---

# 21. Backup / Restore

Backup 是 v1 必需能力。

正式恢复边界：

```text
O3Pilot Instance Restore Point
=
Consistent SQLite Snapshot
+
Raw Objects referenced by that Snapshot
+
Portable Application Config
+
Authorized Encrypted Secret State
+
Integrity / Compatibility Metadata
```

Manual Upload 原文件不属于 Backup，因为它们不是长期 Raw。

## 21.1 Consistency

Backup 必须：

```text
create consistent SQLite snapshot
→ enumerate required Raw references
→ verify every Raw Object
→ write manifest
→ mark backup verified
```

SQLite Snapshot 创建后的新 Capture 不属于该 Restore Point。

## 21.2 Local + Optional R2

正式关系：

```text
Local Backup Repository
→ optional encrypted R2 replica
```

R2 不参与正常运行数据读写。

## 21.3 Restore

Restore 必须先在 staging / recovery path 验证：

```text
SQLite integrity
schema compatibility
Raw reference completeness
Raw hash integrity
config validity
authorized secret recovery
```

验证成功后才能发布 READY。

数据库引用的 Raw 缺失属于 Recovery Error，不能静默降级为普通 Warning。

---

# 22. Upgrade & Persistent Compatibility

升级不得假设：

```text
历史数据可以全部从 Ozon 重新抓取
```

必须保留以下最小规则：

## 22.1 Database Schema

Database Schema 使用明确单调版本。

正式 Release 已发布 Migration 不得修改；修复通过新的 Migration 完成。

Schema 修改只能进入统一 Migration 流程，不允许散落 Runtime DDL。

## 22.2 Pre-upgrade Backup

任何会改变 Persistent State 的正式升级前必须建立并验证 Backup。

## 22.3 Fail Closed

如果 Application 发现当前 Persistent Schema / Raw Metadata 明显比自己支持的版本更新：

```text
do not write
→ refuse normal startup
→ require compatible version / recovery
```

## 22.4 Crash-safe Migration

Migration 必须可明确判断：

```text
not started
completed
failed / recovery required
```

不能以半完成 Schema 继续正常 Sync。

## 22.5 Semantic Versioning where Needed

Parser / Mapping / Metric 等会影响历史解释的逻辑在需要时保存版本。

不建立统一的“所有东西都必须拥有独立 Compatibility Platform”。

---

# 23. Configuration Boundary

Architecture 只冻结以下原则：

```text
one authoritative owner per setting
explicit scope
Secret separate from normal config
host-local separate from portable instance config
invalid critical config fails closed
```

至少区分：

```text
HOST
INSTANCE
SHOP
```

Secret Config 必须额外带 Source / Shop Scope。

具体：

- 文件位置；
- UI Settings；
- Secret Encryption；
- OS Service environment；
- 默认端口；

分别由 `DEPLOYMENT.md` / `SECURITY.md` / `DESIGN.md` 负责。

不在 v1 建立独立的 Configuration Platform、Apply Mode 状态机或每个 Job 的完整 Config Snapshot 平台。

---

# 24. Runtime Health

必须区分：

```text
Liveness
Readiness
Degraded
Dataset Freshness
```

## Liveness

主进程是否能继续执行 Runtime Loop。

外部 Ozon / R2 故障不能直接等同进程死亡。

## Readiness

实例是否可以安全提供正常业务服务。

至少要求：

```text
instance lock acquired
SQLite usable and compatible
Raw Store usable
critical config valid
no unresolved integrity failure
```

## Degraded

实例仍可读，但某 Source / Shop / Dataset / Backup replica 异常。

## Dataset Freshness

根据 Dataset 自己的 Coverage/Freshness Contract 判断，不用单个全局 `healthy=true/false` 代替。

实例长期状态只需要：

```text
STARTING
READY
DEGRADED
RECOVERY_REQUIRED
STOPPING
```

Migration / Backfill 作为操作过程暴露，不建立额外永久状态体系。

---

# 25. Failure Isolation

单个 Source / Shop 故障不得让整个实例不可查询。

例如：

```text
Performance API unavailable
```

仍应允许：

```text
Seller API sync
existing local queries
unrelated shops
```

继续工作。

但以下本地完整性错误必须 Fail Closed：

```text
SQLite integrity failure
unsupported schema
dangling raw_capture reference
failed critical migration
invalid critical secret/config state
```

不得自动执行破坏性“修复”。

---

# 26. Startup

最小启动顺序：

```text
1. resolve instance root
2. acquire exclusive instance lock
3. load critical config / secrets
4. open SQLite and verify compatibility
5. apply permitted migrations
6. verify Raw Store
7. recover interrupted Jobs
8. verify disk safety
9. start Worker + Scheduler
10. publish API readiness
```

如果出现会危及数据正确性的错误：

```text
RECOVERY_REQUIRED
```

而不是带病继续采集。

具体 systemd / launchd 配置归 `DEPLOYMENT.md`。

---

# 27. Observability

v1 至少能够诊断：

```text
application state
SQLite
Raw Store
disk
job queue
sync/import/backup runs
source/shop failures
dataset freshness
data quality issues
```

日志与 Telemetry 必须避免：

```text
credentials
tokens
full raw payload duplication
sensitive imported rows
```

具体 Security Logging 规则由 `SECURITY.md` 定义。

---

# 28. Architecture Testing

实现必须至少覆盖：

## Data Integrity

- 重复同步不产生重复业务事实；
- Raw Object / raw_capture 可以核验；
- Current State 不覆盖 History；
- Missing 不被转换成 0；
- Cross-shop identity 不被静默合并。

## Crash Recovery

- 强杀进程后 RUNNING Job 可恢复；
- Raw write / Metadata write 中断不会产生正常可见的 dangling reference；
- Partial migration 不会继续正常 Sync；
- second Primary Instance 被拒绝。

## Read-only

- 未 Allowlist Endpoint 无法调用；
- Ozon 写 Endpoint 不可达；
- 业务代码不能绕过 typed Source Adapter。

## Backup / Restore

- SQLite + Raw 一致性可验证；
- 缺失 Raw 的 Restore 失败；
- 无 R2 时完整可 Backup / Restore。

## Performance

对真实高频查询路径做基准测试；是否增加 Index / Read Model 必须由实际结果驱动。

---

# 29. Deferred Architecture

以下不进入 v1 正式架构：

```text
Multi-user RBAC
Microservices
Redis / MQ
Distributed Worker Fleet
Remote Primary DB
Generic Resource Governor
Generic Circuit-breaker Platform
Five-level Job Priority
Generic Compatibility State Platform
Generic Configuration Apply-mode Platform
Automatic Raw Age-based Deletion
Cross-shop Canonical Product Platform without real feature need
```

重新进入必须有：

```text
真实产品需求
或
真实性能 / 运维证据
```

---

# 30. 与其他 Contract 的权威关系

| Concern | Authority |
|---|---|
| Product scope / Feature Phase | PRODUCT.md |
| Endpoint availability / verification / window | DATA_SOURCES.md |
| Entity / key / lineage schema | DATA_MODEL.md |
| Metric semantics / formula | METRICS.md |
| Runtime architecture | ARCHITECTURE.md |
| Authentication / Session / Secret / Security | SECURITY.md |
| OS / paths / installation / backup operations | DEPLOYMENT.md |
| UI / UX / presentation | DESIGN.md |

ARCHITECTURE 可以引用这些事实，但不得重新定义第二套业务 Contract。

---

# 31. v1 Architecture Acceptance

v1 Architecture 至少满足：

1. Modular Monolith；
2. Single-machine Native；
3. SQLite 是正式长期 Primary Database；
4. WAL / Foreign Keys / Busy Timeout / Short Transactions；
5. Automatic Raw Body 与 SQLite 分离；
6. `raw_ref` 指向 Raw Capture，而不是物理 Path；
7. Manual Upload 不进入长期 Raw Store；
8. Ozon 永久只读，由 Explicit Allowlist + Restricted Adapter 强制；
9. 一个主进程包含 API / Worker / Scheduler 三逻辑角色；
10. 长任务使用 Persistent Job；
11. Job Contract 为 12 字段、5 Status、2 Priority；
12. Job Durable State 不主要依赖内存；
13. Scheduler 依据 Acquisition Policy / Backfillability；
14. Webhook 是 Signal，Seller API Readback 校准最终状态；
15. Shop Scope 从 ingress 持续到 persistence；
16. Cross-shop identity 不自动合并；
17. Raw / Normalized / Derived 可追溯、可重处理；
18. Missing / Unknown 不等于 0；
19. Source Event / Business / Observed / Coverage / Import / Calculate / Publish 时间语义显式区分，Timestamp 具有明确时区语义；
20. `Schedule Time != Data Complete Time`，Scheduler 不替代 Coverage 完整性判断；
21. Backup 可恢复 SQLite + Raw 一致 Restore Point；
22. R2 可选且不是 Runtime Dependency；
23. Upgrade 不破坏历史持久数据；
24. Source 故障被隔离，本地完整性错误 Fail Closed；
25. 没有真实需求证据时不提前引入外部 Queue、Microservice、Remote DB 或通用治理平台。

---

# 32. 核心不变量

**O3Pilot 对 Ozon 永久只读。**

**SQLite 是长期 Primary Database。**

**Automatic Raw Payload Body 只进入 Local Raw Object Store；SQLite 保存 Capture Metadata。**

**Manual Upload 是一次性 Import Medium，不是长期 Raw。**

**Raw → Normalized → Derived，Derived 不覆盖 Source Fact。**

**不可回补数据的采集优先级高于普通可回补工作。**

**Persistent Job 的关键状态必须持久化。**

**Shop Scope 不允许隐式推断。**

**Current State 不覆盖历史事实。**

**Missing / Unknown / Unavailable 不等于 0。**

**时间语义必须显式区分且可追溯；`Schedule Time != Data Complete Time`。**

**网络请求和大型计算不长期占用 SQLite 写事务。**

**Backup 必须能恢复 SQLite 与其引用的 Raw。**

**R2 不参与正常运行。**

**外部 Source 故障允许降级，本地完整性错误必须 Fail Closed。**

**只有真实需求或测量证据才能增加新的基础设施层。**
