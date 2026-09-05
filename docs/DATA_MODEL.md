# O3Pilot — DATA_MODEL.md

> Version: 1.1  
> Status: Data Model Baseline  
> Updated: 2026-09-04  
> Revision: Batch A.2  
> Applies to: O3Pilot

---

# 1. 文档目的

`DATA_MODEL.md` 定义 O3Pilot 的统一业务数据模型与持久化身份边界。

本文件负责回答：

- 当前阶段有哪些正式 Normalized 业务实体；
- 每个实体的业务身份如何确定；
- 自然业务键如何由数据库强制唯一；
- 自动来源 Raw Capture 如何追溯；
- JSON 字段何时可以作为结构化业务快照保留；
- 多店铺、商品、订单、库存、履约、退货、Analytics、Rating 等对象如何建立关系；
- Raw、Normalized、Derived 如何分层；
- 未知、无权限、未匹配、未验证如何与真实 `0` 区分；
- 哪些未来阶段模型当前只保留 Acquisition / Deferred 边界，而不提前冻结完整 Schema。

本文件不定义：

- 指标最终计算公式；
- Profit / Forecast / Recommendation 的最终模型；
- API 调度周期与 Job 实现；
- 数据库具体部署产品；
- Endpoint 当前验证状态、分页与时间窗口细节；
- 产品页面与导航结构；
- 跨页面视觉语言、通用交互表现与数据呈现规则。

其中，产品页面与导航结构由 `PRODUCT.md` 定义；跨页面视觉语言、通用交互表现与数据呈现规则由 `DESIGN.md` 定义；其余内容仍由 `METRICS.md`、`ARCHITECTURE.md`、`DATA_SOURCES.md`、`DEPLOYMENT.md` 等对应 Authority 定义。

正式原则：

```text
Data Acquisition Phase
!=
Normalized Phase
!=
Feature Phase
```

未来 Feature 所需的不可回补 Raw Data 可以提前采集，但不能因此提前冻结未来阶段的完整 Normalized Schema。

---

# 2. 数据模型基本原则

## 2.1 Shop First

所有 Ozon 业务事实必须归属于明确的 `shop_id`。

除 Ozon 全局字典或明确的跨店公共参考数据外，任何业务对象都不得假设跨店唯一。

例如：

```text
product_id = 123
```

本身不能作为 O3Pilot 内部全局业务身份。

正确身份至少应包含 Shop Scope。

---

## 2.2 Internal ID 逐实体裁决

O3Pilot 不建立“所有核心实体都必须拥有 Internal ID”或“所有表统一包含 `id`”的全局规则。

Internal ID 与自然业务身份逐实体裁决：

- `product`：保留 `product_id_internal`，同时强制 `(shop_id, ozon_product_id)` 唯一；
- `mother_order`：业务身份为 `(shop_id, order_number)`，不强制 Surrogate ID；
- `posting`：业务身份为 `(shop_id, posting_number)`，不强制 Surrogate ID；
- `posting_item`：保留内部身份，因为当前没有经过验证的可靠外部自然键能唯一定位 Posting 内商品行；
- `raw_capture`、`sync_run`、`import_batch`：属于 O3Pilot 自身技术记录，使用内部 ID；
- Future Finance / Advertising / Questions / Inbound Supply：在对应 Normalized Phase 到来前不提前裁决最终 Internal ID Strategy。

Source Key 继续原样保留，用于同步、去重、跨接口匹配、对账、展示和追溯。

---

## 2.3 Raw Fact 与 Business Fact 分离

数据逻辑分层：

```text
Raw
↓
Normalized
↓
Derived
```

### Raw

自动来源 API / Webhook 的原始字节正文进入不可变 Content-addressed Raw Store；SQLite 只保存 `raw_capture` metadata。

人工 CSV / XLS / XLSX 不属于长期 Raw Store，只作为一次性 Import Medium：

```text
Upload
→ Staging
→ Validate
→ Parse
→ Persist structured facts + lineage
→ Delete source file
```

### Normalized

只为当前 Normalized Phase 已进入的业务域建立稳定业务事实，例如 Shop、Product、Posting、Return、Analytics Daily、Rating。

### Derived

保存可重新计算的 Metric、Alert 等结果。Profit、Forecast、Recommendation 等模型只在对应 Phase 进入后建立。

Derived 永远不能覆盖 Source Fact。

---

## 2.4 技术模式与业务模式分离

Ozon API 技术字段必须原样保存，例如：

```text
delivery_schema
integration_type_flow
stock.type
schema
```

业务层再生成标准化字段，例如：

```text
fulfillment_mode
```

技术字段不能被业务字段替代。

---

## 2.5 未知不等于 0

O3Pilot 必须区分：

```text
KNOWN
NULL
UNAVAILABLE
NOT_APPLICABLE
UNMATCHED
UNVERIFIED
```

以及真正的 `0`。

无权限、空响应、未验证结构不得被静默解释为业务 0。

---

## 2.6 Money 永远包含 Currency

任何金额类事实必须同时表达金额与币种；如果来源没有显式 currency，保存 `currency_state = UNKNOWN`，不得通过 Shop Currency 猜测。

---

## 2.7 历史事实不可被当前值覆盖

当前阶段至少必须保留以下历史语义：

- 商品身份映射；
- 商品状态；
- 商品属性；
- 价格；
- 库存；
- Posting 状态与承诺时间；
- Return / Reverse Logistics；
- Rating；
- Data Availability / Capability。

未来阶段模型重新进入时同样必须遵守“历史事实不可被当前值覆盖”。

---

# 3. 数据层

## 3.1 Raw Layer

长期 Raw Store 只用于 automatic machine-acquired payload body。

SQLite 对应的核心 metadata 实体是：

```text
raw_capture
```

人工导入文件不进入长期 Raw Store。

## 3.2 Normalized Fact Layer

保存已经统一身份和字段的当前阶段业务事实。

当前 P0/P1 Active Normalized Model 见 §25。

## 3.3 Derived Layer

保存进入当前 Feature Phase 后允许存在的可重算结果。

当前 v1 允许 P0/P1 Derived，例如 Basic Alert；P2–P4 Derived Model 进入 Deferred Register。

## 3.4 Read Model

Read Model 仅在真实查询路径需要时建立，是可重建性能层，不是 Source of Truth。

---

# 4. Source / Capture Model

## 4.1 `raw_capture`

`raw_capture` 表示一次真实的数据观察：某 Shop 在某一时间，通过某 Source / Endpoint 实际取得一次 API Response 或 Webhook Event。

建议最小字段：

```text
raw_capture_id
shop_id
source_system
source_endpoint
observed_at

content_sha256
content_type
storage_encoding
original_size_bytes
stored_size_bytes

sync_run_id
request_fingerprint
coverage_from
coverage_to
```

其中以下字段按来源允许为空：

```text
sync_run_id
request_fingerprint
coverage_from
coverage_to
```

`raw_capture` 只保存 metadata。完整 Raw Body 不进入 SQLite。

Raw Content 的物理布局、atomic write、backup consistency、retention 由 ADR-001 / `ARCHITECTURE.md` 定义。

禁止在 DATA_MODEL 中建立第二套 Locator Authority，例如：

```text
file_path
object_path
raw_object_path
storage_path
```

也不新增独立 `raw_object` 业务实体。

---

## 4.2 `raw_ref`

Normalized Fact 对自动来源的追溯统一使用：

```text
raw_ref
→ FK
→ raw_capture.raw_capture_id
```

`raw_ref` 不直接指向 filesystem path、SHA-256 或 JSON body。

Source Fact 根据需要可包含：

```text
source_system
source_endpoint
source_object_type
source_object_key
raw_ref
source_observed_at
source_business_time
parser_version
mapping_version
```

并非所有实体都必须拥有完全相同的通用字段模板。

---

## 4.3 `sync_run`

代表一次同步任务执行。

建议字段：

```text
sync_run_id
shop_id
source_system
source_endpoint
started_at
finished_at
status
requested_from
requested_to
records_received
records_written
records_skipped
retry_count
error_code
error_message
```

---

## 4.4 `data_quality_issue`

建议字段：

```text
issue_id
shop_id
domain
issue_type
severity
entity_type
entity_id
source_system
detected_at
resolved_at
status
details_json
```

典型 `issue_type`：

```text
SOURCE_EMPTY_UNEXPECTEDLY
SOURCE_MISMATCH
MISSING_REQUIRED_FIELD
UNMATCHED_PRODUCT
UNMATCHED_POSTING
UNKNOWN_CURRENCY
SCHEMA_CHANGED
DUPLICATE_SOURCE_OBJECT
SYNC_GAP
```

原基线只定义了 `details_json` 字段和 `issue_type`，没有定义其内容边界。Batch A.2 新增：

```text
data_quality_issue.details_json
= bounded diagnostic context
!= raw source payload
```

允许保存字段名、expected / actual、reason、matching context、validation result 等有限诊断信息；禁止复制完整 API Response、Webhook Payload 或完整 Source Object。

---

# 5. JSON Field Semantics

ADR-001 禁止的是 Raw Body 长期存入 SQLite，不是 JSON 数据类型本身。

DATA_MODEL 中所有 `*_json` 字段必须属于以下三类之一。

## 5.1 RAW_BODY — 禁止进入 Normalized / Derived Row

以下历史字段只作为禁止模式记录，不再属于正式字段定义：

```text
payload_json
posting_item.raw_item_json
return_item.raw_item_json
raw_component_json
raw_row_json
```

处理规则：

```text
Raw Body
→ Content-addressed Raw Store

Normalized / Derived Fact
→ raw_ref
→ raw_capture
```

Future Finance 重新设计时也不得恢复 `raw_component_json` 这类 Raw Body-in-row 模式。

## 5.2 NORMALIZED_STRUCTURED_SNAPSHOT — 允许保留

当前合法字段包括：

```text
attributes_json
complex_attributes_json
images_json
barcodes_json
conditions_json
improve_attributes_json
price_index_json
commissions_json
marketing_actions_json
promotion_actions_json
source_place_json
target_place_json
storage_json
logistic_json
visual_json
```

保留条件：

1. 有明确业务语义；
2. 表达 Normalized Snapshot / Fact 自身的动态结构；
3. 不是完整 Source Object 的无解释副本；
4. 不承担 Raw Payload 追溯职责。

## 5.3 DIAGNOSTIC_OR_DERIVED_CONTEXT — 允许 bounded context

例如：

```text
details_json
calculation_details_json
input_snapshot_json
explanation_json
```

允许表达诊断、计算解释、规则输入等有限上下文，但禁止承载完整 Raw Payload。

当前 Active Model 中 `data_quality_issue.details_json` 与 `alert.details_json` 按此规则使用；其他字段只有在对应 Future Derived Model 进入正式 Phase 后才能重新定义。

---

# 6. Shop

## 6.1 `shop`

代表一个独立 Ozon Seller Account / Shop。

建议字段：

```text
shop_id
ozon_client_id
display_name
legal_name
country_code
settlement_currency
subscription_type
is_active
created_at
updated_at
```

数据库级唯一约束：

```text
ozon_client_id
```

当前 API 凭证属于 Shop，但敏感凭证本身不进入普通业务数据表。

---

## 6.2 `shop_capability`

记录某 Shop 当前数据能力。

例如：

```text
shop_id
capability_code
availability_status
reason
verified_at
```

例如 Review：

```text
capability_code = REVIEW_V2
availability_status = UNAVAILABLE
reason = SUBSCRIPTION_REQUIRED
```

这样 UI 和业务逻辑不会把“没有权限”误解释成“没有数据”。

---

# 7. 商品模型总览

当前 Active Product Model 保持 Shop-scoped：

```text
Shop
└── Product
    ├── ProductIdentifierHistory
    ├── ProductSnapshot
    ├── ProductAttributeSnapshot
    ├── ProductContentRatingSnapshot
    ├── ProductPriceSnapshot
    ├── InventorySnapshot
    └── InventoryTurnoverSnapshot
```

商品必须同时保留 Ozon Product ID、SKU、offer_id，不把 `offer_id` 当永久主键。

跨店 Seller Catalog 不属于当前 Active Schema；不同 Shop 的商品不得仅根据相同 `offer_id`、SKU 或商品名称自动合并。完整 Seller Catalog 进入 Deferred Register。

---

# 8. Product

## 8.1 `product`

代表“某个 Shop 中的 Ozon 商品实体”。

建议字段：

```text
product_id_internal
shop_id
ozon_product_id
current_sku
current_offer_id
first_seen_at
last_seen_at
is_archived
created_at
updated_at
```

数据库级唯一约束：

```text
(shop_id, ozon_product_id)
```

`ozon_product_id` 是当前验证条件下最适合作为 Shop 内 canonical source key 的标识。

但仍保留 O3Pilot 内部 ID，避免未来 Ozon 身份语义变化直接破坏数据库关系。

---

# 9. Product Identifier History

## 9.1 `product_identifier_history`

用于保存：

- `product_id`
- `sku`
- `offer_id`

之间在不同时间的对应关系。

建议字段：

```text
product_identifier_history_id
product_id_internal
shop_id
ozon_product_id
sku
offer_id
valid_from
valid_to
observed_at
source_system
raw_ref
```

用途：

- 处理 offer_id 改名；
- 处理跨接口历史匹配；
- 解释历史订单中的旧 offer_id；
- 防止当前商品映射覆盖过去真实身份。

`offer_id` 不作为永久主键。

---

# 10. Product Snapshot

## 10.1 `product_snapshot`

记录商品当前状态的历史快照。

建议字段：

```text
product_snapshot_id
product_id_internal
shop_id
name
category_id
type_id
status_name
status_code
is_archived
availability
primary_image
model_id
model_count
observed_at
raw_ref
```

注意：

以下字段必须分开：

```text
status
availability
inventory
```

不得生成一个简单的：

```text
is_sellable
```

并把其当成 Ozon 原始事实。

如产品需要“综合可售判断”，该值只能属于 Derived Layer。

---

# 11. Product Attribute Snapshot

## 11.1 `product_attribute_snapshot`

代表一次完整商品属性观察。

建议字段：

```text
attribute_snapshot_id
product_id_internal
shop_id
observed_at
height
width
depth
dimension_unit
weight
weight_unit
attributes_json
complex_attributes_json
images_json
barcodes_json
raw_ref
```

类目属性高度动态，因此不建议把所有 Ozon Attribute 强制拆成固定列。

高频、核心、跨类目稳定的字段可以标准化为列。

其余属性保留结构化 JSON。

---

# 12. Product Content Rating

## 12.1 `product_content_rating_snapshot`

建议字段：

```text
content_rating_snapshot_id
product_id_internal
shop_id
sku
total_rating
observed_at
raw_ref
```

评分组单独保存：

## 12.2 `product_content_rating_group`

```text
content_rating_group_id
content_rating_snapshot_id
group_key
group_name
rating
weight
conditions_json
improve_attributes_json
improve_at_least
```

这样 Ozon 修改评分规则后，历史评分仍然可解释。

---

# 13. Price

## 13.1 `product_price_snapshot`

代表某商品某时间点的 Ozon 价格事实。

建议字段：

```text
price_snapshot_id
product_id_internal
shop_id
sku
offer_id

seller_price_amount
seller_price_currency

marketing_seller_price_amount
marketing_seller_price_currency

old_price_amount
old_price_currency

min_price_amount
min_price_currency

acquiring_amount
acquiring_currency_state

price_index_json
commissions_json
marketing_actions_json
volume_weight

observed_at
raw_ref
```

金额字段是否具有独立 currency 以 API 实际返回为准。

没有显式 currency 的字段不得自动使用 `settlement_currency`。

---

# 14. Inventory

库存必须区分：

- 当前状态；
- 历史快照；
- 技术库存类型；
- 仓库级库存。

---

## 14.1 `inventory_snapshot`

建议字段：

```text
inventory_snapshot_id
shop_id
product_id_internal
sku
offer_id

stock_type_raw
shipment_type_raw

present
reserved

warehouse_id
warehouse_scope

observed_at
source_system
source_endpoint
raw_ref
```

其中：

```text
warehouse_scope
```

可表示：

```text
AGGREGATED
WAREHOUSE
```

---

## 14.2 技术库存类型

原始值原样保存，例如：

```text
fbo
rfbs
fbp
```

这些不是最终业务模式。

---

## 14.3 `inventory_turnover_snapshot`

保存 Ozon Analytics 自己返回的库存周转指标。

建议字段：

```text
turnover_snapshot_id
shop_id
product_id_internal
sku
average_daily_sales
current_stock
inventory_days_cover
turnover_grade
observed_at
raw_ref
```

它与 O3Pilot 自己计算的库存覆盖指标必须分开。

---

# 15. In-transit Inventory — Phase / Source Boundary

```text
Current Inventory
!=
In-transit Inventory
```

In-transit Inventory 是 P1 Core Operations 数据域；Manual path 在 P1 必须可用，自动来源仍为 `TBD`。

当前不冻结具体 `Supply + SupplyItem` 两级 Normalized Schema，也不因为旧设计存在就假设 Ozon 自动来源一定按该结构提供数据。

正式边界：

```text
Acquisition Phase = P1 manual path / TBD automatic
Acquisition Policy = MANUAL / TBD automatic
Backfillability = Source-dependent; state history required
Normalized Phase = P1
Metrics Phase = P1
Feature Phase = P1
```

具体 Inbound Supply Schema 进入 Deferred Register，只有在真实自动来源验证，或 P1 Manual 输入格式确定且证明需要两级身份模型后重新 Review。 P1 实现前仍必须为已确定的 Manual Input 建立最小必要 Normalized Persistence，但不得直接复活旧的 Supply / SupplyItem 两表设计。

---

# 16. 母订单 MotherOrder

## 16.1 定义

Ozon 原始：

```text
order_number
```

O3Pilot 中文术语：

```text
母订单号
```

实体：

```text
mother_order
```

---

## 16.2 `mother_order`

建议字段（`mother_order_id` 为可选 Surrogate Identity，不替代自然业务身份）：

```text
mother_order_id
shop_id
order_number
order_created_at
first_seen_at
last_seen_at
created_at
updated_at
```

数据库级唯一约束：

```text
(shop_id, order_number)
```

MotherOrder 主要承担：

- 多 Posting 聚合；
- 用户统一查看；
- 订单级关联。

它不是 Ozon 所有接口中字段最丰富的实体。

---

# 17. Posting

## 17.1 定义

Ozon 原始：

```text
posting_number
```

O3Pilot 中文术语：

```text
订单号
```

---

## 17.2 `posting`

Posting 是 O3Pilot 订单、履约、物流、取消、退货的核心连接点；未来 Finance 进入 P2 后也通过 Posting 建立业务关联。

建议字段（`posting_id` / `mother_order_id` 可作为内部关系优化，但不替代自然业务身份）：

```text
posting_id
shop_id
mother_order_id

posting_number
order_number

source_posting_family
status_raw
substatus_raw

created_at_source
in_process_at

shipment_deadline_at
shipment_date_without_delay
shipment_date
handover_to_delivery_at
delivering_date

promised_delivery_from
promised_delivery_to
fact_delivery_date
cancelled_at

warehouse_id
delivery_method_id
logistics_provider_id
tracking_number

delivery_schema_raw
integration_type_flow_raw

fulfillment_mode
fulfillment_mapping_version

first_seen_at
last_seen_at
raw_ref
created_at
updated_at
```

数据库级唯一约束：

```text
(shop_id, posting_number)
```

---

## 17.3 Source Posting Family

`source_posting_family` 用于记录原始 API 技术来源。

例如：

```text
FBS
FBO
FBP
```

该字段回答：

> 这条 Posting 是从哪类 API Contract 获得的？

而不是回答：

> 业务上最终属于哪一种 Ozon Global 履约模式？

---

## 17.4 Fulfillment Mode

### 17.4.1 `fulfillment_mode`

标准化业务模式当前允许：

```text
FBP
REALFBS
OGL
OTHER_VERIFIED
UNKNOWN
```

WHD 不进入正常首次销售 `fulfillment_mode`。

---

### 17.4.2 映射依据

映射必须综合真实字段，例如：

```text
source_posting_family
delivery_schema_raw
integration_type_flow_raw
stock_type_raw
delivery_method
warehouse
```

不能仅使用：

```text
/v4/posting/fbs/list
```

就把所有记录标记为 FBS。

---

### 17.4.3 映射版本

所有标准化结果必须保存：

```text
fulfillment_mapping_version
```

因为未来 Ozon 字段或业务体系变化后，需要能够重新计算历史分类。

---

## 17.5 Cancellation

### 17.5.1 `cancellation`

取消作为独立业务事实，不与 Return 混合。

建议字段：

```text
cancellation_id
posting_id
mother_order_id
shop_id

cancel_reason_id
cancel_reason
cancel_reason_raw

initiator
affects_rating
cancelled_at

source_system
raw_ref
```

一条 Posting 原则上可以只有一个最终取消事实，但原始状态变化历史保留在 `posting_status_history`。

---

# 18. Posting Item

## 18.1 `posting_item`

代表一个 Posting 中的商品行。

建议字段：

```text
posting_item_id
posting_id
mother_order_id
shop_id

product_id_internal
sku
offer_id
ozon_product_id

quantity

unit_price_amount
unit_price_currency

line_index
raw_ref
created_at
updated_at
```

不要假设：

```text
posting_id + sku
```

永远唯一。

内部仍使用：

```text
posting_item_id
```

避免未来出现：

- 同 SKU 多行；
- Bundle；
- 不同价格行；
- API Contract 改变。

---

## 18.2 `posting_item_pricing_observation`

订单商品侧同时可能出现多种价格和促销观察值。

这些值用于解释订单发生时的商业上下文，但不能替代最终 Finance Accrual。

建议字段：

```text
posting_item_pricing_observation_id
posting_item_id
posting_id
shop_id

seller_price_amount
seller_price_currency

customer_price_amount
customer_price_currency

old_price_amount
old_price_currency

discount_amount
discount_currency
discount_percent

payout_amount
payout_currency_state

commission_amount
commission_currency
commission_percent

promotion_actions_json

source_system
observed_at
raw_ref
```

原则：

```text
Order-side Pricing Observation
!=
Final Finance Fact
```

订单接口中的 `commission`、`payout`、促销和买家支付金额可以用于展示和核验，但最终财务确认仍由 Finance 域负责。

---

# 19. Posting Status / Schedule History

## 19.1 `posting_status_history`

建议字段：

```text
posting_status_history_id
posting_id
shop_id
status_raw
substatus_raw
status_time
source_system
source_event_type
observed_at
raw_ref
```

来源可以包括：

- Seller API Snapshot；
- Webhook Event。

这样可以建立订单生命周期，而不是只保存最后状态。

---

## 19.2 `posting_schedule_history`

用于保存 Ozon 对发货截止时间和承诺配送窗口的历史变化。

建议字段：

```text
posting_schedule_history_id
posting_id
shop_id

shipment_deadline_at
promised_delivery_from
promised_delivery_to

change_type
effective_at
observed_at

source_system
source_event_type
raw_ref
```

`change_type` 示例：

```text
INITIAL
SHIPMENT_DEADLINE_CHANGED
DELIVERY_WINDOW_CHANGED
API_REFRESH
UNKNOWN
```

如果 Ozon 修改了承诺时间，历史值不得被当前值覆盖。

该实体用于后续计算：

- 按时发货；
- 延迟发货；
- 承诺配送达成；
- Ozon 调整时限前后的差异。

具体指标公式进入 `METRICS.md`。

---

# 20. Logistics / Warehouse / Delivery / Shipment

## 20.1 `logistics_event`

用于统一物流节点。

建议字段：

```text
logistics_event_id
posting_id
shop_id

event_type
event_time

event_type_raw
warehouse_id
provider_id
tracking_number

source_system
raw_ref
```

标准 `event_type` 可以包括：

```text
ORDER_CREATED
PROCESSING_STARTED
READY_TO_SHIP
SHIPPED
DELIVERING
DELIVERED
CANCELLED
RETURN_STARTED
RETURNED
```

只有在原始数据足够明确时才能生成标准事件。

---

## 20.2 `warehouse`

仓库是标准化业务维度，不应长期只以 Posting 文本字段存在。

建议字段：

```text
warehouse_id_internal
shop_id
source_warehouse_id
name
warehouse_type
country_code
city
is_active
first_seen_at
last_seen_at
```

同一来源仓库的名称变化不应生成新的业务身份。

---

## 20.3 `logistics_provider`

建议字段：

```text
logistics_provider_id
source_provider_id
name
provider_type
first_seen_at
last_seen_at
```

用于统一：

- Ozon 物流合作伙伴；
- OGL；
- 卖家自有物流商；
- 其他已验证承运商。

---

## 20.4 `delivery_method`

建议字段：

```text
delivery_method_id
shop_id
source_delivery_method_id
name

warehouse_id_internal
logistics_provider_id

fulfillment_mode
service_level
raw_name

first_seen_at
last_seen_at
```

原始配送方式名称仍需保存，标准化字段不能覆盖来源字符串。

---

## 20.5 `shipment_package`

代表一个实际 Posting 下的包裹 / 货件测量事实。

必须区分：

```text
Product Weight
Package Actual Weight
Package Volumetric Weight
Package Chargeable Weight
```

建议字段：

```text
shipment_package_id
posting_id
shop_id

package_index
tracking_number

actual_weight
actual_weight_unit

length
width
height
dimension_unit

volume
volume_unit

volumetric_weight
volumetric_weight_unit

chargeable_weight
chargeable_weight_unit

measurement_source
measured_at

source_system
source_import_batch_id
raw_ref
created_at
updated_at
```

商品卡片中的重量和尺寸属于 Product Attribute。

物流商、Ozon 或 Seller Data 返回的包裹重量属于 Shipment Package。

两者不能互相覆盖。

---

# 21. Returns / Reverse Logistics

## 21.1 `return_case`

统一表示 Ozon 退货 / 逆向物流业务 Case。

建议字段：

```text
return_case_id
shop_id
posting_id
mother_order_id

return_source_type
source_return_id
return_number

return_type_raw
schema_raw

state_raw
state_name_raw
group_state_raw

return_reason
return_reason_id
is_defect

created_at_source
return_date
final_moment

warehouse_id
source_place_json
target_place_json
storage_json
logistic_json
visual_json
compensation_status

raw_ref
created_at
updated_at
```

数据库级唯一约束：

```text
(shop_id, return_source_type, source_return_id)
```

---

## 21.2 Return Source Type

当前至少包括：

```text
RFBS_RETURN
GENERAL_RETURN
```

未来如果出现其他正式返回源，可新增枚举。

---

## 21.3 Return Item

### 21.3.1 `return_item`

建议字段：

```text
return_item_id
return_case_id
posting_id
shop_id

product_id_internal
sku
offer_id

quantity

original_price_amount
original_price_currency

return_reason
return_reason_id
is_defect

client_comment
client_photo_count
return_method_type
return_tracking_number

raw_ref
```

买家评论、图片等可能包含个人数据。

实际保存策略由 `SECURITY.md` 定义。

---

## 21.4 WHD

WHD 属于逆向物流和二次销售场景。

不作为 `fulfillment_mode` 的正常首次销售值。

---

### 21.4.1 WHD 表达方式

当前无需单独建立一个完全独立的“WHD Order”根实体。

优先通过：

```text
return_case.schema_raw = Whd
```

以及：

```text
source_place
target_place
return logistics
```

保存 WHD 逆向事实。

如果后续能够可靠建立：

```text
原订单
→ Return
→ WHD Stock
→ WHD 二次销售 Posting
```

则通过关联表：

### 21.4.2 `reverse_logistics_link`

```text
reverse_logistics_link_id
shop_id
return_case_id
source_posting_id
resale_posting_id
link_type
confidence
mapping_version
```

不得仅通过相同 SKU 自动认定某次二次销售就是某次退货的后续销售。

---

### 21.4.3 `reverse_logistics_event`

用于记录退货 / 未认购 / 取消件进入逆向物流后的生命周期事件。

建议字段：

```text
reverse_logistics_event_id
return_case_id
posting_id
shop_id

event_type
event_type_raw
event_time

warehouse_id_internal
logistics_provider_id
tracking_number

source_system
raw_ref
```

标准 `event_type` 可在证据充分时映射为：

```text
RETURN_ACCEPTED
REVERSE_TRANSIT_STARTED
WHD_RECEIVED
STORAGE_STARTED
MARKDOWN_APPLIED
RESALE_READY
RESOLD
RETURNED_TO_SELLER
DESTROYED
COMPENSATED
```

不能根据最终状态反推出不存在的中间事件。

---

# 22. Analytics — Daily SKU Metrics

## 22.1 `sku_daily_analytics`

来源：

```text
/v1/analytics/data
```

业务粒度 / 数据库级唯一身份：

```text
shop_id + sku + business_date
```

建议字段：

```text
sku_daily_analytics_id
shop_id
product_id_internal
sku
business_date

revenue
ordered_units

hits_view_search
hits_view_pdp
hits_view

hits_tocart_search
hits_tocart_pdp
hits_tocart

session_view_search
session_view_pdp
session_view

returns
cancellations
delivered_units

source_metrics_signature
raw_ref
```

`revenue` 当前没有逐行显式 currency 时：

不得自行生成 currency。

可以另记录：

```text
currency_state = UNKNOWN
```

直到数据契约进一步确认。

---

# 23. Shop Health / Rating / FBS Error

## 23.1 `shop_rating_snapshot`

统一保存 Current 与 History。

建议字段：

```text
shop_rating_snapshot_id
shop_id

rating_code
group_name
rating_name

value
value_type
direction

danger_threshold
warning_threshold
premium_threshold

status_danger
status_warning
status_premium

period_from
period_to
observed_at
raw_ref
```

当前 Snapshot 可以看作：

```text
period_from = period_to = current observation
```

History 则使用 Ozon 返回的日区间。

---

## 23.2 FBS Error Index

### 23.2.1 `fbs_error_index_snapshot`

建议字段：

```text
fbs_error_index_snapshot_id
shop_id
index_value
processing_cost_amount
processing_cost_currency_state
observed_at
raw_ref
```

---

### 23.2.2 `fbs_error_defect_daily`

```text
fbs_error_defect_daily_id
shop_id
business_date
defect_value
raw_ref
```

---

### 23.2.3 `fbs_error_posting`

当前真实列表为空。

因此实体可以预留：

```text
fbs_error_posting_id
shop_id
posting_id
source_error_id
error_type
amount
currency
raw_ref
```

但这些非空字段在获得真实样本前不能被视为已验证 Contract。

---

## 23.3 Alert

### 23.3.1 `alert`

建议字段：

```text
alert_id
shop_id

alert_type
severity

entity_type
entity_id

triggered_at
resolved_at
status

metric_code
metric_value
threshold_value

rule_version
details_json
```

Alert 引用其他域的数据，而不是复制业务事实。`details_json` 只允许 bounded diagnostic / derived context，不得保存完整 Raw Payload。

---

# 24. Source / Sync / Import / Data Integrity

## 24.1 Webhook Capability

### 24.1.1 `webhook_push_type`

记录 Shop 当前可用 Push Type。

建议字段：

```text
webhook_push_type_id
shop_id
push_type
description
endpoint_bound
endpoint_id
endpoint_url
observed_at
raw_ref
```

它是配置观察，不代表 O3Pilot 会修改订阅。

---

## 24.2 Webhook Event

### 24.2.1 `webhook_event`

Webhook 原始事件必须先完成 Raw durable capture。完整事件正文由 Raw Store 保存，`webhook_event` 只保存可索引字段与 `raw_ref`。

建议标准索引字段：

```text
webhook_event_id
shop_id

message_type
source_event_id
posting_number
order_number
sku

event_time
received_at

dedup_fingerprint
processing_status
raw_ref
```

由于真实 Payload Contract 尚未完全验证：

除 Raw Store 中的原始正文外，所有尚未验证的提取字段都应允许 `NULL`。

---

### 24.2.2 Webhook 幂等

在 Ozon 重复行为未完全验证前：

必须假设 Webhook 可能：

- 重复；
- 乱序；
- 延迟。

因此：

```text
dedup_fingerprint
```

不能只依赖 `received_at`。

最终订单状态仍由 Seller API 回读确认。

---

## 24.3 Manual Import Tracking

人工导入文件仅作为一次性输入介质。O3Pilot 不持久化 CSV / XLSX 等原文件内容，也不把原文件复制到数据库、R2 或其他长期对象存储。

持久化对象只包括：

- 导入后的结构化业务数据；
- 导入批次记录；
- 必要的逐行处理结果或错误记录。

### 24.3.1 `import_batch`

代表一次用户上传并解析数据的导入批次，也是人工导入的主要追溯记录。

建议字段：

```text
import_batch_id
shop_id
source_system
source_type

original_file_name
file_hash
mime_type
file_size

report_type
report_period_from
report_period_to

parser_version
started_at
finished_at
status
rows_total
rows_success
rows_failed
error_summary

supersedes_import_batch_id
```

其中：

- `original_file_name`、`file_hash`、`mime_type`、`file_size` 只属于导入元数据，不代表保存了文件内容；
- `file_hash` 可用于重复导入检测；
- `supersedes_import_batch_id` 可用于表示修正版或替代关系；
- 不保存文件 BLOB、Base64、持久化文件路径或对象存储 Key。

---

### 24.3.2 `import_row_result`

可选的逐行处理结果，只用于需要定位失败、未匹配或异常行的导入。

字段：

```text
import_row_result_id
import_batch_id
source_row_number
parse_status
mapping_status
entity_type
entity_id
error_code
error_message
```

不保存完整 `raw_row_json`。

对于成功导入且业务实体已经保存 `source_import_batch_id + source_row_number` 的记录，可以不额外创建 `import_row_result`，避免重复存储。

---

## 24.4 Reconciliation

### 24.4.1 `reconciliation_result`

用于比较两个来源。

建议字段：

```text
reconciliation_result_id
shop_id
domain

left_source
right_source

entity_type
entity_key

field_name
left_value
right_value

difference_type
status
checked_at
```

例如：

```text
Seller API vs Ozon Official Report
Finance Accrual vs Official Finance Report（P2 进入后）
Webhook vs Seller API
```

对账结果不覆盖任何一侧的原始事实。

---

## 24.5 数据可用性

为明确区分“0”和“没有数据”，建议建立：

### 24.5.1 `data_availability`

```text
data_availability_id
shop_id
domain
source_system
status
reason
coverage_from
coverage_to
last_success_at
checked_at
```

`status`：

```text
AVAILABLE
PARTIAL
UNAVAILABLE
NOT_APPLICABLE
UNVERIFIED
STALE
```

---

## 24.6 时间模型

所有时间字段必须明确语义。

### 24.6.1 Timestamp

原始 Timestamp：

- 保留 Source Time；
- 标准化为带时区时间；
- 不因前端显示 UTC+8 而覆盖原值。

---

### 24.6.2 Business Date

对于：

- Analytics Day；
- Campaign Day；
- Rating Day；

应保存：

```text
business_date
source_date_semantics
```

不能把所有 Date 自动解释成北京时间自然日。

---

### 24.6.3 默认显示时区

O3Pilot 默认：

```text
Asia/Shanghai
```

这只是展示和默认统计时区。

不是所有来源的业务日边界。

---

## 24.7 Snapshot 模型

以下数据默认使用 Snapshot 建立历史：

- Product；
- Product Attributes；
- Product Content Rating；
- Price；
- Inventory；
- Inventory Turnover；
- Shop Rating；
- Shop Info；
- Capability。

Snapshot 至少包含：

```text
observed_at
```

相同内容是否去重属于实现层优化，不改变业务历史语义。

---

## 24.8 Fact 与 Snapshot

### Fact

代表已经发生的业务事实，例如：

- Posting；
- Posting Item；
- Return；
- Analytics SKU × Day；
- Webhook Event。

Fact 不应因为下一次同步不存在就自动删除。

### Snapshot

代表某一时点观察，例如：

- Price；
- Inventory；
- Rating Current；
- Product Content Rating。

---

## 24.9 删除与消失

API 全量列表中对象消失时：

不得物理删除历史实体。

正确处理是：

```text
last_seen_at
is_active / lifecycle_state
source_missing_since
```

而不是：

```text
DELETE FROM business_fact
```

同样适用于：

- Product；
- Inventory Source Rows；
- Webhook Capability。

---

## 24.10 Mapping Status

所有跨系统映射建议使用：

```text
MATCHED_EXACT
MATCHED_RULE
AMBIGUOUS
UNMATCHED
UNVERIFIED
```

例如跨来源商品或订单映射不能只返回 NULL。

必须知道为什么没有匹配。

---

## 24.11 数据版本

当前阶段需要版本化的解释规则至少包括：

```text
parser_version
mapping_version
fulfillment_mapping_version
metric_version
```

Future Finance / Forecast / Recommendation 的版本字段在对应模型重新进入时再裁决，不提前形成 Active Contract。


---

## 24.12 未验证结构

以下内容在没有可靠证据前只能停留 Raw / Partial，或进入对应 Deferred Phase：

- Performance Phrases 非空行级业务结构；
- Analytics Stocks 非空 Item 结构；
- Review List / Detail 当前真实字段；
- FBS Error Posting 非空行；
- 未验证 Webhook Payload 字段；
- 无可靠来源的竞争对手库存 / 销量。

不得为了“数据库字段完整”提前虚构 Schema。


---

## 24.13 数据安全边界

数据模型不得引导任何 Ozon 写操作。

以下数据即使读取到配置，也只能作为 Observation：

- Webhook Endpoint；
- Campaign Config；
- Price；
- Inventory；
- Product Attributes；
- Warehouse Config。

不存在：

```text
pending_write
auto_publish
auto_price_update
auto_inventory_update
auto_bid_update
```

等执行型业务实体。

Future Recommendation 只表示建议，不具备 Ozon 写入能力。

---

# 25. Phase Matrix

本节是 DATA_MODEL 的 Phase 边界。Endpoint 当前验证状态、分页、时间窗口仍以 `DATA_SOURCES.md` 为权威。

```text
Feature Delivery Phase
!=
Data Acquisition Phase
```

v1 Feature Delivery = P0 + P1。P2 Finance & Profit、P3 Growth Analytics、P4 Decision Support 不属于 v1 Feature Delivery。

| Dataset / Domain | Acquisition Phase | Acquisition Policy | Backfillability | Normalized Phase | Metrics Phase | Feature Phase |
|---|---|---|---|---|---|---|
| Shop Info / Roles / Capability | P0 | PERIODIC_SNAPSHOT | Current-state oriented | P0 | P0/1 | P0 |
| Product Catalog | P0 | PERIODIC_SNAPSHOT | Current state only | P0 | P1 | P1 |
| Product Main Info | P0 | PERIODIC_SNAPSHOT | Current state only | P0/1 | P1 | P1 |
| Product Attributes | P0 | PERIODIC_SNAPSHOT / ON_DEMAND | Current state only | P0/1 | P1/P3 consumer | P1 |
| Product Content Rating | P0 | PERIODIC_SNAPSHOT | Current snapshot | P1 | P1/P3 consumer | P1 |
| Price | P0 | PERIODIC_SNAPSHOT | Current state only | P1 | P1 | P1 |
| Current Inventory | P0 | PERIODIC_SNAPSHOT | Current state only | P1 | P1 | P1 |
| Turnover | P0 | PERIODIC_SNAPSHOT | Current / rolling analytics；历史依赖持续快照 | P1 | P1 | P1 |
| Orders | P0 | BACKFILLABLE_SYNC | Yes, within API rules | P0/1 | P1 | P1 |
| FBO Detail | P0 | ON_DEMAND | 按 Posting 查询可补充 | P1 | P1 | P1 |
| Returns | P0 | BACKFILLABLE_SYNC | Yes, within API rules | P1 | P1 | P1 |
| Analytics SKU × Day | P0 | BACKFILLABLE_SYNC | Historical date interval available | P1 | P1 | P1 |
| Rating Current / History | P0 | PERIODIC_SNAPSHOT / BACKFILLABLE_SYNC | Current available history range | P1 | P1 | P1 |
| FBS Error Index / Defects | P0 | PERIODIC_SNAPSHOT / BACKFILLABLE_SYNC | Current index verified；部分明细仍 PARTIAL | P1 | P1 | P1 |
| Webhook Event | P0 | REQUIRED_CONTINUOUS | No replay guarantee | P0/1 | — | P0/1 |
| Webhook Config / Push Types | P0 | PERIODIC_SNAPSHOT | Current state only | P0 | — | P0 |
| Search Analytics | P0 | REQUIRED_CONTINUOUS / PERIODIC_SNAPSHOT | 可回补已保留区间；实际最大 lookback 待验证 | P3 | P3 | P3 |
| Finance Accrual | P0 | BACKFILLABLE_SYNC | By-day backfill；长期完整保留保证不单独假设 | P2 | P2 | P2 |
| Ozon Exchange Rate | P0 | BACKFILLABLE_SYNC / PERIODIC_SNAPSHOT | Period supported；长期稳定性未承诺 | P2 | P2 | P2 |
| Campaign Current State | P0 | PERIODIC_SNAPSHOT | Current Campaign set；历史状态依赖快照 | P3 | P3 | P3 |
| Campaign Interval Stats | P3* | ON_DEMAND / BACKFILLABLE_SYNC | Historical interval query available；max lookback UNVERIFIED | P3 | P3 | P3 |
| Campaign Daily | P0 | BACKFILLABLE_SYNC | Historical date data available；max lookback UNVERIFIED | P3 | P3 | P3 |
| Campaign SKU Daily | P0 | REQUIRED_CONTINUOUS | Today / yesterday only | P3 | P3 | P3 |
| Performance Phrases | TBD | TBD | 当前不能作为可靠历史主来源；非空 Rows 未验证 | P3 after verification | P3 | P3 |
| Questions / Answers | TBD | TBD | Backfillability 未验证 | P3 | P3 | P3 |
| Reviews | P3 when available | ON_DEMAND / TBD | ACCESS_DEPENDENT；真实 List/Detail 尚未验证 | P3 | P3 | P3 |
| Ozon Official Reports | As needed | MANUAL | Depends on Seller Center report range | Corresponding domain | Corresponding domain | Supporting source |
| Settlement / Payout | P2 | MANUAL | Depends on Seller Center report retention；P2 = domain import support mandatory | P2 | P2 | P2 |
| Seller Cost / ERP Actual Cost | P2 | MANUAL | Depends on user ERP export/history；P2 = domain import support mandatory | P2 | P2 | P2 |
| Seller Logistics Cost | P2 | MANUAL | Depends on logistics provider bill/export；P2 = domain import support mandatory | P2 | P2 | P2 |
| Package Measurement | P2 / As needed | MANUAL | Source-dependent | P2 / As needed | P2 | P2 |
| In-transit Inventory | P1 manual path / TBD automatic | MANUAL / TBD automatic | Source-dependent；必须保存状态历史 | P1 | P1 | P1 |
| Seller Catalog Mapping | Later / when needed | MANUAL | User-maintained；保存 validity/history | Later | Later | Later |

## 25.1 Priority Vocabulary

Priority 只允许：

```text
CRITICAL_CAPTURE
NORMAL
```

`P0`–`P4` 只表示 Phase，不得作为 Priority。

`CRITICAL_CAPTURE` 只用于不可回补或极短回补窗口的数据。当前明确包括：

```text
Webhook Event
Campaign SKU Daily
```

其他 Dataset 默认 `NORMAL`；如后续真实窗口验证证明存在不可接受的数据丢失风险，再依据 Architecture Dataset Contract 调整 Priority。不能因为 Acquisition Phase = P0 就自动标记为 `CRITICAL_CAPTURE`。

## 25.2 P1 保护项

以下内容不得在 Phase 污染清理中误删：

```text
Product Content Rating
In-transit Inventory manual path
Analytics SKU × Day / sku_daily_analytics
```

其中 Search Analytics / Advertising 仍属于 P3 Normalized / Feature Phase。

---

# 26. Identity & Key Rules

## 26.1 数据库级唯一性

自然业务身份必须由数据库强制唯一，不能只依赖 Application Code 去重。

允许：

```text
Composite PRIMARY KEY
Composite UNIQUE
Surrogate PRIMARY KEY + Composite UNIQUE
```

`Identity Constraint` 与 `Performance Index` 分离：

```text
PRIMARY KEY / UNIQUE
→ identity correctness

INDEX
→ query performance
```

查询索引继续遵循 Query-driven Index，不机械复制自然键约束。

## 26.2 当前已裁决自然业务身份

| 实体 | 数据库级业务身份 |
|---|---|
| Shop | `UNIQUE(ozon_client_id)` |
| Product | `UNIQUE(shop_id, ozon_product_id)` |
| MotherOrder | `UNIQUE(shop_id, order_number)` |
| Posting | `UNIQUE(shop_id, posting_number)` |
| ReturnCase | `UNIQUE(shop_id, return_source_type, source_return_id)` |
| SKU Daily Analytics | `UNIQUE(shop_id, sku, business_date)` |
| PostingItem | 保留 Internal Identity；不伪造 SKU 自然键 |

Product 保留 `product_id_internal`。MotherOrder / Posting 不因为存在自然键就强制取消或强制增加 Surrogate ID；实现可以基于工程需要选择，但上述业务身份约束必须存在。

## 26.3 当前 Active 核心关系

以下为核心关系示意，不是对所有技术 / 子实体的穷举清单。

```text
Shop
├── ShopCapability
├── Product
│   ├── ProductIdentifierHistory
│   ├── ProductSnapshot
│   ├── ProductAttributeSnapshot
│   ├── ProductContentRatingSnapshot
│   │   └── ProductContentRatingGroup
│   ├── ProductPriceSnapshot
│   ├── InventorySnapshot
│   ├── InventoryTurnoverSnapshot
│   └── SkuDailyAnalytics
├── MotherOrder
│   └── Posting
│       ├── PostingItem
│       │   └── PostingItemPricingObservation
│       ├── PostingStatusHistory
│       ├── PostingScheduleHistory
│       ├── Cancellation
│       ├── LogisticsEvent
│       ├── ShipmentPackage
│       ├── ReverseLogisticsLink
│       └── ReturnCase
│           ├── ReturnItem
│           └── ReverseLogisticsEvent
├── Warehouse
├── LogisticsProvider
├── DeliveryMethod
├── ShopRatingSnapshot
├── FbsErrorIndexSnapshot
├── FbsErrorDefectDaily
├── WebhookPushType
├── WebhookEvent
├── DataAvailability
├── DataQualityIssue
├── ReconciliationResult
├── Alert
└── ImportBatch
    └── ImportRowResult
```

Seller Catalog、Inbound Supply、Finance、Advertising、Questions / Answers、Forecast / Recommendation 不属于当前 Active 关系图。

## 26.4 跨域关键连接

Product 主要通过 `product_id_internal` 连接内部 Fact，同时保留 `sku` / `offer_id` / `ozon_product_id` 作为来源身份与匹配字段。

Order 以 `posting_number` / Posting 为业务连接核心，MotherOrder 负责聚合。

Return 优先通过 `posting_number` / `posting_id` 与 `source_return_id` 连接；无法可靠匹配时必须保留 `UNMATCHED / UNVERIFIED`，不得猜测。

不同 Shop 商品不得只通过 `offer_id`、SKU 或名称自动合并。

---

# 27. Acceptance Criteria

在数据库正式实现前，本模型必须满足：

- [ ] 所有 Ozon 业务 Fact 都有明确 Shop Scope；
- [ ] 自动来源 Raw metadata 统一由 `raw_capture` 表达；
- [ ] Raw lineage 统一使用 `raw_ref`，且 `raw_ref → raw_capture.raw_capture_id`；
- [ ] SQLite 不长期保存完整 API / Webhook Raw Body；
- [ ] 不存在第二套 Raw Storage Locator Authority；
- [ ] `posting_item` 与 `return_item` 不保存行级 Raw Body 副本；
- [ ] Future Finance 不恢复 Raw Component Body-in-row 模式；
- [ ] 所有 `*_json` 字段均已归类为 RAW_BODY / NORMALIZED_STRUCTURED_SNAPSHOT / DIAGNOSTIC_OR_DERIVED_CONTEXT；
- [ ] Snapshot JSON 有明确业务语义，不因 ADR-001 被误删；
- [ ] Diagnostic / Derived JSON 只保存 bounded context，不变相保存 Raw Payload；
- [ ] 自然业务身份由数据库级 PRIMARY KEY / UNIQUE 保证；
- [ ] Identity Constraint 与 Performance Index 分离；
- [ ] Product ID / SKU / offer_id 不混用；
- [ ] MotherOrder / Posting 不混用；
- [ ] PostingItem 不伪造 `(posting_id, sku)` 唯一性；
- [ ] `product_snapshot` 与 `product_attribute_snapshot` 双表保留；
- [ ] Seller Catalog 不属于当前 Active Schema；
- [ ] 跨店商品不通过弱标识自动合并；
- [ ] Inbound Supply 两级 Schema 不属于当前 Active Schema；
- [ ] In-transit Inventory 仍是 P1 独立数据域，manual path 未被误删，automatic source 保持 TBD；
- [ ] Warehouse / Logistics Provider / Delivery Method / Shipment Package 保留为 Core Operations；
- [ ] `sku_daily_analytics` 继续属于 P1 Active Normalized Model；
- [ ] Analytics Daily 未与 P3 Search / Advertising Analytics 一并 Deferred；
- [ ] `cancellation` 继续作为独立业务事实，收纳于 Posting，不与 Return 合并；
- [ ] Product Content Rating 的 P1 定位未被误删；
- [ ] Webhook Event 保持 P0 REQUIRED_CONTINUOUS，并通过 Seller API readback / reconciliation 校准业务最终状态；
- [ ] Priority 只使用 `CRITICAL_CAPTURE / NORMAL`；
- [ ] `P0`–`P4` 只作为 Phase 使用；
- [ ] P2 / P3 / Later 完整 Normalized Entity 不再污染 v1 Active Model；
- [ ] Future Feature 的 Early Raw Acquisition 仍按 Phase Matrix 保留；
- [ ] Current Inventory 与 In-transit Inventory 不混用；
- [ ] Product Weight、Package Weight、Chargeable Weight 不混用；
- [ ] Snapshot 与 Fact 区分；
- [ ] 无权限 / 未验证 / 空集合不会被存成业务 0；
- [ ] 空集合不会触发无条件历史删除；
- [ ] Ozon 集成保持严格只读；
- [ ] DATA_MODEL 与 ADR-001、`ARCHITECTURE.md`、Batch 0 Phase Matrix 无冲突。

---

# 28. Deferred Model Register

Deferred 不等于删除数据采集。Early Raw Acquisition 继续以 §25 Phase Matrix 为准。

## 28.1 Seller Catalog — Later

当前不建立完整：

```text
seller_catalog_item
seller_catalog_product_mapping
```

Active 约束仅保留：不同 Shop 商品不得通过相同 SKU / offer_id / name 自动认定为同一商品。

重新进入条件：出现明确跨店统一商品身份需求，且真实业务证明 Shop-scoped Product 无法满足。

## 28.2 Inbound Supply Normalized Model — Deferred

当前不建立完整：

```text
inbound_supply
inbound_supply_item
```

Reason：当前只有 In-transit Inventory 数据域要求，自动 Ozon 来源与两级身份模型尚未验证。

重新进入条件：

1. Ozon 自动入库计划来源完成真实验证；或
2. P1 Manual In-transit Inventory 实际输入格式确定；
3. 且真实数据证明需要 Supply / SupplyItem 两级身份模型。

## 28.3 Phase 2 — Finance & Profit

以下完整 Normalized / Derived Schema 不属于 v1 Active Model：

```text
finance_accrual_type
finance_accrual
finance_component
finance_allocation
ozon_exchange_rate_interval
money_conversion
seller_logistics_charge
seller_sku_cost
order_item_cost
settlement_period
payout
profit_calculation_run
order_profit
order_item_profit
```

Finance Accrual / Ozon Exchange Rate 等 P0 Early Acquisition 仍按 §25 保存 Raw Capture；Settlement / Payout、Seller Cost、Seller Logistics Cost 等 Manual Domain 从 P2 起必须正式支持领域 Parser / Persistence。

Future Finance redesign 禁止恢复 Raw Body-in-row 字段，来源正文统一通过 `raw_ref` 追溯。

## 28.4 Phase 3 — Growth Analytics / Customer Voice

以下完整 Schema 当前不建立：

```text
sku_search_period_analytics
sku_search_query_analytics
advertising_account
ad_campaign
ad_campaign_snapshot
ad_campaign_daily_metric
ad_campaign_period_metric
ad_campaign_sku_daily_metric
ad_campaign_object
question
answer
review
performance_phrase_row
```

Search / Campaign 的 Early Acquisition 继续按 §25；Questions / Answers Acquisition 与 Backfillability 保持 `TBD`；Reviews 保持 access-dependent。

## 28.5 Phase 4 — Decision Support

以下模型进入 P4 后再重新设计：

```text
forecast_run
forecast_point
forecast_backtest
recommendation
```

不因为未来可能使用而提前冻结 v1 数据表。

---

# 29. 文档边界与开放项

## 29.1 DATA_MODEL 与 METRICS

DATA_MODEL 定义数据是什么、身份是什么、如何追溯；`METRICS.md` 定义如何计算。

例如产品退货率所需的 PostingItem / ReturnItem / ReturnCase / Product / Time 由本模型保证，分子、分母、跨期、件数口径由 `METRICS.md` 定义。

## 29.2 DATA_MODEL 与 DATA_SOURCES

`DATA_SOURCES.md` 定义来源能提供什么、当前验证状态、分页、时间窗口和已知边界；DATA_MODEL 只定义取得数据后的统一保存方式。

任何新增 Active Entity 必须能够回溯到：

- 已验证数据源；
- 卖家明确提供的数据；
- 明确的 Derived 过程。

## 29.3 当前开放问题

以下问题继续保持开放，不在 A.2 猜测：

1. Analytics Stocks 非空结构；
2. Performance Phrases 非空结构；
3. Review List / Detail 真实结构；
4. FBS Error Posting 非空结构；
5. Webhook 各 Push Type 完整 Payload；
6. Ozon Exchange Rate XAPI 的长期稳定性；
7. WHD 二次销售与原退货的可靠一对一关联字段；
8. In-transit Inventory 自动 Ozon 来源；
9. 物流商导出对 Posting / Package 的稳定关联键；
10. Settlement / Payout 是否存在可替代人工报告的稳定只读 API；
11. Questions / Answers 的历史 Backfillability。

---

# 30. 核心原则

**Automatic Source 先完成 durable Raw Capture，再建立业务解释。**

**Raw Body 只有一个长期物理路径；SQLite 只保存 capture metadata / lineage。**

**人工导入是一次性 Import Medium，不进入长期 Raw Store。**

**Internal ID 逐实体裁决，自然业务身份必须由数据库强制唯一。**

**订单以 Posting 为核心连接点，MotherOrder 负责聚合。**

**商品三个标识全部保留，不把 offer_id 当永久主键。**

**技术履约模式不等于业务履约模式。**

**Cancellation 是 Posting 独立事实，不与 Return 混合。**

**WHD 属于逆向物流。**

**Current Inventory 与 In-transit Inventory 分离。**

**未来阶段不可回补数据可以提前 Raw Capture，但不能提前冻结未来完整 Normalized Schema。**
