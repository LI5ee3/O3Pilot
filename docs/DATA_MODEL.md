# O3Pilot — DATA_MODEL.md

> Version: 1.0  
> Status: Data Model Baseline  
> Updated: 2026-09-03  
> Applies to: O3Pilot

# 1. 文档目的

`DATA_MODEL.md` 定义 O3Pilot 的统一业务数据模型。

本文件负责回答：

- O3Pilot 有哪些正式业务实体；
- 每个实体的身份如何确定；
- Ozon 原始技术字段如何保存；
- 不同接口中的同一业务对象如何关联；
- 多店铺如何隔离；
- 商品、订单、库存、履约、退货、Finance、广告、Rating 等对象如何建立关系；
- 原始事实、标准化事实和派生结果如何分层；
- 币种、汇率和金额转换如何保存；
- 卖家采购成本如何进入订单级利润模型；
- 数据来源、版本和计算过程如何追溯；
- 未知、无权限、未匹配、未验证等状态如何与真实 `0` 区分。

本文件不定义：

- 指标最终计算公式；
- 产品退货率公式；
- Profit 具体公式；
- Forecast 算法；
- 页面结构；
- API 调度周期；
- 数据库具体产品和部署方案。

上述内容分别进入 `METRICS.md`、`ARCHITECTURE.md`、`DESIGN.md` 等文档。

---

# 2. 数据模型基本原则

## 2.1 Shop First

所有 Ozon 业务事实必须归属于明确的 `shop_id`。

除 Ozon 全局字典或明确的跨店公共参考数据外，任何业务对象都不得假设跨店唯一。

例如：

```text
product_id = 123
```

本身不能作为 O3Pilot 内部全局主键。

正确身份至少应包含：

```text
shop_id + product_id
```

---

## 2.2 Internal ID 与 Source Key 分离

每个核心实体拥有 O3Pilot 自己生成的内部 ID。

例如：

```text
shop_id
product_id_internal
mother_order_id
posting_id
posting_item_id
return_case_id
finance_accrual_id_internal
campaign_id_internal
```

同时保留 Ozon 或其他来源提供的业务标识：

```text
ozon_product_id
sku
offer_id
order_number
posting_number
return_id
accrual_id
campaign_id
```

O3Pilot 内部关系优先使用内部 ID。

来源字段用于：

- 同步；
- 去重；
- 跨接口匹配；
- 对账；
- 用户展示；
- 原始事实追溯。

---

## 2.3 Raw Fact 与 Business Fact 分离

O3Pilot 不直接把某个 API 字段解释成最终业务事实。

数据至少分为三层：

```text
Raw
↓
Normalized
↓
Derived
```

### Raw

尽可能原样保存：

- API Response；
- Webhook Payload；
- Ozon 官方报告原始文件；
- ERP 导入原始文件。

### Normalized

建立统一实体和统一字段，例如：

- Shop；
- Product；
- MotherOrder；
- Posting；
- ReturnCase；
- FinanceAccrual；
- AdCampaign。

### Derived

建立可重新计算的结果，例如：

- 库存覆盖天数；
- 产品退货率；
- Profit；
- Forecast；
- Alert；
- Recommendation。

派生结果不能覆盖原始事实。

---

## 2.4 技术模式与业务模式分离

Ozon API 技术字段必须原样保存。

例如：

```text
delivery_schema
integration_type_flow
stock.type
schema
```

业务层再生成：

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

以及真正的：

```text
0
```

例如：

Review API 因订阅权限无法读取时：

```text
review_count = UNAVAILABLE
```

而不是：

```text
review_count = 0
```

---

## 2.6 Money 永远包含 Currency

任何金额类字段都必须保留：

```text
amount
currency
```

不得只保存金额而依赖 Shop Currency 推断。

如果原始 API 自身没有 currency：

必须保留：

```text
currency_state = UNKNOWN
```

而不是自行补上店铺结算币种。

---

## 2.7 历史事实不可被当前值覆盖

下列数据必须支持历史：

- 商品身份映射；
- 商品状态；
- 价格；
- 库存；
- Rating；
- Campaign 状态；
- 卖家采购成本；
- 汇率；
- 算法版本。

历史订单不能使用当前最新 SKU 成本重新解释为“实际历史采购成本”。

---

# 3. 数据层

O3Pilot 建议采用以下逻辑数据层。

## 3.1 Raw Layer

保存未经业务解释的来源数据。

核心对象：

- `raw_api_response`
- `raw_webhook_event`
- `raw_import_file`
- `raw_import_row`

Raw Layer 不承担最终业务查询。

---

## 3.2 Normalized Fact Layer

保存已经统一身份和字段的业务事实。

例如：

- Shop；
- Product；
- MotherOrder；
- Posting；
- PostingItem；
- InventorySnapshot；
- ReturnCase；
- FinanceAccrual；
- AdCampaign。

---

## 3.3 Derived Layer

保存可根据事实重新生成的分析结果。

例如：

- Metric；
- Profit；
- Forecast；
- Recommendation；
- Alert；
- Reconciliation Result。

---

# 4. 通用字段规范

核心实体建议统一包含以下字段。

```text
id
shop_id
created_at
updated_at
```

来源事实实体还应根据需要包含：

```text
source_system
source_endpoint
source_object_type
source_object_key
raw_payload_id
source_observed_at
source_business_time
parser_version
mapping_version
```

其中：

### `source_system`

例如：

```text
OZON_SELLER_API
OZON_PERFORMANCE_API
OZON_XAPI
OZON_WEBHOOK
OZON_OFFICIAL_REPORT
MABANG_ERP
SELLER_MANUAL
```

### `source_observed_at`

O3Pilot 实际取得该数据的时间。

### `source_business_time`

Ozon 或外部系统表示的业务发生时间。

两者不得混用。

---

# 5. Shop

## 5.1 `shop`

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

唯一约束：

```text
ozon_client_id
```

当前 API 凭证属于 Shop，但敏感凭证本身不进入普通业务数据表。

---

## 5.2 `shop_capability`

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

# 6. Source Lineage

## 6.1 `raw_payload`

统一保存 API / Webhook 原始 JSON。

建议字段：

```text
raw_payload_id
shop_id
source_system
source_endpoint
request_fingerprint
response_status
payload_json
fetched_at
source_coverage_from
source_coverage_to
sync_run_id
schema_fingerprint
```

---

## 6.2 `sync_run`

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

## 6.3 `data_quality_issue`

记录数据异常。

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

---

# 7. 商品模型总览

商品模型必须同时解决：

- Ozon Product ID；
- SKU；
- offer_id；
- 商品详情；
- 变体；
- 价格；
- 内容评分；
- 库存；
- 订单商品关联；
- Finance SKU 关联。

核心关系：

```text
SellerCatalogItem
 └── SellerCatalogProductMapping
      └── Shop
           └── Product
                ├── ProductIdentifierHistory
                ├── ProductSnapshot
                ├── ProductAttributeSnapshot
                ├── ProductContentRatingSnapshot
                ├── ProductPriceSnapshot
                └── InventorySnapshot
```

`SellerCatalogItem` 是卖家内部跨店商品实体；`Product` 仍然是 Shop-scoped 的 Ozon 商品实体。两者不能合并。

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

建议唯一约束：

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
raw_payload_id
```

用途：

- 处理 offer_id 改名；
- 处理跨接口历史匹配；
- 解释历史订单中的旧 offer_id；
- 防止当前商品映射覆盖过去真实身份。

`offer_id` 不作为永久主键。

---

## 9.2 `seller_catalog_item`

代表卖家内部的 canonical 商品，不属于 Ozon 原始事实。

它用于把多个店铺中实际属于同一真实商品的 Ozon Product 映射到同一个卖家内部商品。

建议字段：

```text
seller_catalog_item_id
internal_name
internal_category
supplier_id
brand
model
variant
created_at
updated_at
```

典型用途：

- 跨店合并库存；
- 跨店总销量；
- 跨店总利润；
- 统一采购成本；
- 供应商管理；
- 总退货表现；
- 跨店补货建议。

不得通过相同 `offer_id`、商品名称或 SKU 自动认定两个店铺商品一定相同。

---

## 9.3 `seller_catalog_product_mapping`

建立 Seller Catalog 与 Shop Product 的显式关系。

建议字段：

```text
seller_catalog_product_mapping_id
seller_catalog_item_id
product_id_internal
shop_id

mapping_status
mapping_method
confidence
mapping_version

valid_from
valid_to
created_at
updated_at
```

`mapping_status` 至少支持：

```text
MATCHED_EXACT
MATCHED_RULE
AMBIGUOUS
UNMATCHED
UNVERIFIED
```

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
raw_payload_id
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
raw_payload_id
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
raw_payload_id
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
raw_payload_id
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
raw_payload_id
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
raw_payload_id
```

它与 O3Pilot 自己计算的库存覆盖指标必须分开。

---

## 14.4 `inbound_supply`

代表正在进入 FBP 等仓库体系的补货 / 交货批次事实。

该实体只描述“已计划或正在运输中的入库库存”，不代表 O3Pilot 会创建、修改或取消任何 Ozon 交货申请。

建议字段：

```text
inbound_supply_id
shop_id

source_supply_id
source_status
fulfillment_mode

origin_location
destination_warehouse_id

prepared_at
shipped_at
accepted_at
completed_at
cancelled_at

source_system
raw_payload_id
created_at
updated_at
```

来源可能是：

- 已验证可读取的 Ozon 数据；
- Ozon 官方报告；
- 卖家自有数据；
- 物流商数据。

具体来源能力由 `DATA_SOURCES.md` 定义。

---

## 14.5 `inbound_supply_item`

建议字段：

```text
inbound_supply_item_id
inbound_supply_id
shop_id
product_id_internal
seller_catalog_item_id
sku
offer_id

planned_quantity
shipped_quantity
accepted_quantity

created_at
updated_at
```

由此可以形成：

```text
Current Inventory
+
In-transit Inventory
```

但二者必须作为不同事实保存。

库存覆盖与补货算法如何使用在途库存，由 `METRICS.md` 定义。

---

# 15. 母订单 MotherOrder

## 15.1 定义

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

## 15.2 `mother_order`

建议字段：

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

唯一约束：

```text
(shop_id, order_number)
```

MotherOrder 主要承担：

- 多 Posting 聚合；
- 用户统一查看；
- 订单级关联。

它不是 Ozon 所有接口中字段最丰富的实体。

---

# 16. Posting

## 16.1 定义

Ozon 原始：

```text
posting_number
```

O3Pilot 中文术语：

```text
订单号
```

---

## 16.2 `posting`

Posting 是 O3Pilot 订单、履约、物流、取消、Finance、退货的核心连接点。

建议字段：

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
raw_payload_id
created_at
updated_at
```

唯一约束：

```text
(shop_id, posting_number)
```

---

# 17. Source Posting Family

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

# 18. Fulfillment Mode

## 18.1 `fulfillment_mode`

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

## 18.2 映射依据

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

## 18.3 映射版本

所有标准化结果必须保存：

```text
fulfillment_mapping_version
```

因为未来 Ozon 字段或业务体系变化后，需要能够重新计算历史分类。

---

# 19. Posting Item

## 19.1 `posting_item`

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
raw_item_json
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

## 19.2 `posting_item_pricing_observation`

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
raw_payload_id
```

原则：

```text
Order-side Pricing Observation
!=
Final Finance Fact
```

订单接口中的 `commission`、`payout`、促销和买家支付金额可以用于展示和核验，但最终财务确认仍由 Finance 域负责。

---

# 20. Posting Status History

## 20.1 `posting_status_history`

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
raw_payload_id
```

来源可以包括：

- Seller API Snapshot；
- Webhook Event。

这样可以建立订单生命周期，而不是只保存最后状态。

---

## 20.2 `posting_schedule_history`

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
raw_payload_id
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

# 21. Logistics Event

## 21.1 `logistics_event`

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
raw_payload_id
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

## 21.2 `warehouse`

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

## 21.3 `logistics_provider`

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

## 21.4 `delivery_method`

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

## 21.5 `shipment_package`

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
raw_payload_id
created_at
updated_at
```

商品卡片中的重量和尺寸属于 Product Attribute。

物流商、Ozon 或 Seller Data 返回的包裹重量属于 Shipment Package。

两者不能互相覆盖。

---

# 22. Cancellation

## 22.1 `cancellation`

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
raw_payload_id
```

一条 Posting 原则上可以只有一个最终取消事实，但原始状态变化历史保留在 `posting_status_history`。

---

# 23. Return Case

## 23.1 `return_case`

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

raw_payload_id
created_at
updated_at
```

---

## 23.2 Return Source Type

当前至少包括：

```text
RFBS_RETURN
GENERAL_RETURN
```

未来如果出现其他正式返回源，可新增枚举。

---

# 24. Return Item

## 24.1 `return_item`

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

raw_item_json
```

买家评论、图片等可能包含个人数据。

实际保存策略由 `SECURITY.md` 定义。

---

# 25. WHD

WHD 属于逆向物流和二次销售场景。

不作为 `fulfillment_mode` 的正常首次销售值。

---

## 25.1 WHD 表达方式

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

## 25.2 `reverse_logistics_link`

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

## 25.3 `reverse_logistics_event`

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
raw_payload_id
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

# 26. Finance 类型字典

## 26.1 `finance_accrual_type`

来自：

```text
/v1/finance/accrual/types
```

建议字段：

```text
type_id
name
description
first_seen_at
last_seen_at
raw_payload_id
```

`type_id` 保留 Ozon 官方数字 ID。

O3Pilot 可以额外增加内部中文分类，但不能替换官方 Type。

---

# 27. Finance Accrual

## 27.1 `finance_accrual`

代表一个 Ozon Finance Accrual 根记录。

建议字段：

```text
finance_accrual_id_internal
shop_id

accrual_id
accrual_date

accrued_category
unit_number

total_amount
total_currency

posting_id
posting_number

source_system
raw_payload_id
created_at
```

唯一约束：

```text
(shop_id, accrual_id)
```

---

## 27.2 `settlement_period`

代表 Ozon 与卖家的一个结算周期。

当前模型允许其主要来源先来自 Ozon 官方财务报告。

建议字段：

```text
settlement_period_id
shop_id

period_from
period_to
settlement_currency

source_system
source_document_type
source_import_file_id

created_at
updated_at
```

Settlement 不替代 Accrual。

关系为：

```text
Finance Accrual
→ Settlement Reconciliation
→ Payout
```

---

## 27.3 `payout`

代表 Ozon 实际或计划向卖家支付的款项事实。

建议字段：

```text
payout_id
shop_id
settlement_period_id

payout_reference
status

amount
currency

planned_payment_date
actual_payment_date

withheld_amount
withheld_currency

source_system
source_import_file_id
raw_payload_id

created_at
updated_at
```

当前没有可靠自动 API Contract 时：

- 可以通过 Ozon 官方报告 / 结算文件导入；
- 不得因为模型存在就假设 Seller API 一定可读取；
- 不得根据 Finance Accrual 自行伪造“已付款”状态。

---

# 28. Finance Component

Finance 不能简单设计成“一条订单一条费用”。

不同 Accrual 有不同粒度。

---

## 28.1 `finance_component`

用于把 Accrual 内真正的收入 / 费用拆成可分析的 Component。

建议字段：

```text
finance_component_id
finance_accrual_id_internal
shop_id

type_id

scope
posting_id
posting_item_id
sku

amount
currency

quantity
seller_price_amount
seller_price_currency

source_component_type
raw_component_json
created_at
```

`scope`：

```text
SHOP
POSTING
SKU
CONTAINER
UNKNOWN
```

---

## 28.2 SKU 级事实

如果 Ozon 原始 Finance 明确携带 SKU：

```text
scope = SKU
sku = ...
```

可以直接用于 SKU 级事实。

例如当前已经验证的部分销售佣金。

---

## 28.3 Posting 级事实

如果 Ozon 只提供 Posting 级费用：

```text
scope = POSTING
posting_id = ...
sku = NULL
```

例如当前已验证的部分 rFBS 国际配送和代理费用。

不得为了方便直接把该费用复制到每一个 SKU。

---

# 29. Finance Allocation

SKU 利润可能需要把 Posting 级费用分摊到商品。

这属于 Derived Layer，而不是 Finance Fact。

---

## 29.1 `finance_allocation`

建议字段：

```text
finance_allocation_id
finance_component_id
posting_item_id

allocated_amount
currency

allocation_rule
allocation_rule_version
allocation_weight
calculation_run_id

created_at
```

例如分摊方法可能是：

```text
BY_QUANTITY
BY_REVENUE
BY_WEIGHT
BY_VOLUME
CUSTOM
```

具体哪一种用于哪项费用，在 `METRICS.md` 定义。

---

# 30. Money

Money 是概念 Value Object。

任何 Money 最低包含：

```text
amount DECIMAL
currency CHAR(3)
```

禁止使用 Binary Float 作为正式财务存储。

---

# 31. Ozon Exchange Rate

O3Pilot 将 Ozon 业务汇率与卖家采购成本汇率分开。

---

## 31.1 `ozon_exchange_rate_interval`

来源：

```text
OZON_XAPI
```

当前数据来自 Ozon 官方帮助页面实际调用的：

```text
GET https://xapi.ozon.ru/exchange-rates/sellers/exchange-rate/by-period
```

建议字段：

```text
ozon_exchange_rate_id

from_currency
to_currency

valid_from
valid_to

service_rate
sale_rate

marketplace_id

source_system
fetched_at
raw_payload_id
```

映射：

```text
service_rate = exchangeRate.rate
sale_rate    = exchangeRate.rateWithAdjustment
```

当前 Ozon 页面语义：

```text
service_rate -> 服务和罚款
sale_rate    -> 销售
```

---

## 31.2 时间匹配

汇率必须根据业务时间匹配：

```text
valid_from <= business_time < valid_to
```

不能仅通过北京时间日期 Join。

当前实测的时间段以 UTC 21:00 切分，对应莫斯科自然日 00:00。

Raw `valid_from` / `valid_to` 必须原样保存。

---

## 31.3 XAPI 稳定性状态

该接口已经取得真实响应，但并非 Seller API / Performance API 正式 Contract。

因此必须：

- 保存历史；
- 保存 Raw；
- 允许接口未来变化；
- 不能只在利润页面实时请求而不落库。

---

# 32. Money Conversion

所有派生币种转换建议使用统一记录。

## 32.1 `money_conversion`

建议字段：

```text
money_conversion_id

source_entity_type
source_entity_id
source_field

original_amount
original_currency

converted_amount
converted_currency

exchange_rate
exchange_rate_type
exchange_rate_provider
exchange_rate_reference_id

rate_basis_type
rate_basis_time
rate_policy_version

calculation_version
calculated_at
```

`exchange_rate_type` 示例：

```text
OZON_SALE
OZON_SERVICE
SELLER_COST
MANUAL
```

`rate_basis_type` 至少支持：

```text
ORDER_CREATED_AT
SERVICE_PROVIDED_AT
ACCRUAL_DATE
ERP_TRANSACTION_CONTEXT
MANUAL
UNKNOWN
```

例如：

- 商品销售和部分配送费用可能依据订单创建时间；
- 仓储、逆向物流、销毁等服务可能依据服务提供时间；
- ERP 采购成本使用 ERP 自己的订单 / 成本上下文。

不能只保存一个笼统的 `business_time`。

原始 Money 永远不能被转换结果覆盖。

---


# 33. 卖家成本模型

卖家采购成本、卖家自有物流费用和包装等成本属于 Seller-Owned Data。

它们不属于 Ozon 原始事实。

---

## 33.1 成本域原则

必须明确区分：

```text
Ozon Finance Logistics Fee
!=
Seller / Logistics Provider Actual Cost
```

Ozon 向卖家收取的物流相关 Finance Component 属于 Ozon Finance。

卖家向物流商、仓储方或其他服务商实际支付的费用属于 Seller Cost。

两者都可以进入 Profit，但来源与语义必须分开。

---

## 33.2 `seller_logistics_charge`

保存卖家侧订单级或包裹级实际物流成本。

建议字段：

```text
seller_logistics_charge_id
shop_id
posting_id
shipment_package_id

logistics_provider_id
charge_type

amount
currency

actual_weight
volumetric_weight
chargeable_weight
weight_unit

billed_at

source_system
source_import_batch_id
source_row_number
raw_row_json

mapping_status
mapping_version
created_at
```

`charge_type` 示例：

```text
DOMESTIC
CROSS_BORDER
LAST_MILE
RETURN
WAREHOUSING
PACKAGING
OTHER
```

具体导入字段由对应 Adapter 定义。

如果物流商数据只能关联到 Posting 而不能准确关联到 SKU：

不得静默分摊到商品。

SKU Profit 的分摊进入 Derived Layer。

---

# 34. Seller SKU Cost

## 34.1 `seller_sku_cost`

用于保存 SKU 的参考 / 当前采购成本配置。

建议字段：

```text
seller_sku_cost_id
shop_id
product_id_internal
sku
offer_id

cost_amount
cost_currency

valid_from
valid_to

supplier_id
source_system
source_import_batch_id

created_at
updated_at
```

用途：

- 当前参考成本；
- 预测；
- 没有订单实际成本时的 fallback 候选。

不能默认用当前 `seller_sku_cost` 重算历史订单实际成本。

---

# 35. Order Item Actual Cost

## 35.1 `order_item_cost`

保存某一个历史订单商品实际使用的采购成本事实。

建议字段：

```text
order_item_cost_id
posting_item_id
posting_id
shop_id

cost_basis_amount
cost_basis_currency

erp_exchange_rate
erp_exchange_rate_direction

transaction_cost_amount
transaction_cost_currency

source_system
source_import_batch_id
source_row_number

mapping_version
created_at
```

这是历史 Profit 计算最重要的卖家侧数据之一。

---

# 36. 马帮 ERP 成本 Adapter

当前已确认的马帮 ERP 导出字段包括：

```text
订单编号
平台SKU
平台SKU数量
平台SKU单个成本
汇率(原币)
平台链接
```

---

## 36.1 字段映射

建议 Adapter 映射：

```text
订单编号
→ posting.posting_number

平台SKU
→ offer_id / seller platform SKU reference

平台SKU数量
→ source_quantity

平台SKU单个成本
→ cost_basis_amount

汇率(原币)
→ erp_exchange_rate

平台链接
→ source_product_url
```

由于“平台SKU”是 ERP 自身命名，不得因为字段名包含 SKU 就直接映射到 Ozon `sku` 数字字段。

应通过：

```text
posting_number
+
产品标识映射
```

确定对应的 `posting_item_id`。

---

## 36.2 当前成本换算语义

当前真实样本支持：

- CNY 订单样本可见 `汇率(原币)=1.0`；
- USD 订单样本可见 `汇率(原币)=6.9`；
- 同一商品可以在不同订单出现不同 `汇率(原币)`；
- 同一商品历史采购成本也可能变化。

因此马帮汇率属于订单商品成本上下文的一部分。

不能：

```text
把 6.9 硬编码为 USD
```

而应通过对应 Ozon Posting / Product 的真实币种决定：

```text
transaction_cost_currency
```

---

## 36.3 当前 Adapter 换算规则

对于当前已验证马帮导出格式，Adapter 可以记录：

```text
transaction_cost_amount
=
cost_basis_amount / erp_exchange_rate
```

但必须同时保存：

```text
cost_basis_amount
erp_exchange_rate
transaction_cost_amount
transaction_cost_currency
mapping_version
```

这样未来马帮导出字段语义变化时，可以重算或审计。

---

## 36.4 Cost Basis Currency

`平台SKU单个成本` 的基础币种不能只从字段名推断。

当前数据与 Ozon 订单币种交叉验证支持其作为当前 ERP 成本基准金额使用，但正式导入 Profile 应显式定义：

```text
cost_basis_currency
```

而不是让 Parser 永久硬编码。

例如当前马帮导入 Profile 可以配置：

```text
cost_basis_currency = CNY
```

该配置属于 Adapter Contract。

---

# 37. Seller Cost FX 与 Ozon FX 分离

以下两类汇率永远分开：

## Ozon Business FX

来源：

```text
OZON_XAPI
```

用于解释：

- Ozon 销售；
- Ozon 服务；
- Ozon 罚款；
- Ozon 业务换算。

## Seller Cost FX

来源：

```text
MABANG_ERP
SELLER_MANUAL
其他 Seller Data
```

用于：

- 采购成本；
- 卖家内部成本。

禁止使用 Ozon `sale_rate` 自动替代 ERP 采购汇率。

禁止使用 ERP 采购汇率改写 Ozon Finance 原始金额。

---

# 38. Seller Cost Fallback

历史订单成本建议允许明确的来源优先级，但具体 Profit 公式由 `METRICS.md` 定义。

数据模型至少能够区分：

```text
ACTUAL_ORDER_ITEM_COST
SKU_COST_AT_ORDER_TIME
CURRENT_SKU_COST
MISSING
```

任何 fallback 必须保存：

```text
cost_source_type
```

避免把估算成本展示为实际历史采购成本。

---

# 39. Analytics — SKU Daily

## 39.1 `sku_daily_analytics`

来源：

```text
/v1/analytics/data
```

唯一粒度：

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
raw_payload_id
```

`revenue` 当前没有逐行显式 currency 时：

不得自行生成 currency。

可以另记录：

```text
currency_state = UNKNOWN
```

直到数据契约进一步确认。

---

# 40. Search Analytics

## 40.1 `sku_search_period_analytics`

粒度：

```text
shop_id + sku + period_from + period_to
```

建议字段：

```text
sku_search_period_analytics_id
shop_id
product_id_internal
sku
period_from
period_to

unique_search_users
position
unique_view_users
view_conversion
gmv_amount
gmv_currency

raw_payload_id
```

无数据 SKU 可能被 API 整体省略。

因此实体不存在不等于指标为 0。

---

## 40.2 `sku_search_query_analytics`

粒度：

```text
shop_id + sku + query + period_from + period_to
```

建议字段：

```text
sku_search_query_analytics_id
shop_id
product_id_internal
sku
query
query_index
period_from
period_to

unique_search_users
unique_view_users
position
view_conversion
order_count
gmv_amount
gmv_currency

raw_payload_id
```

---

# 41. Advertising Account

Performance API 凭证与 Seller API 凭证独立。

建议增加：

## 41.1 `advertising_account`

```text
advertising_account_id
shop_id
performance_client_id
is_active
first_seen_at
last_seen_at
```

敏感 `client_secret` 不进入普通业务数据模型。

---

# 42. Ad Campaign

## 42.1 `ad_campaign`

建议字段：

```text
ad_campaign_id_internal
advertising_account_id
shop_id

campaign_id
title
adv_object_type
created_at_source
first_seen_at
last_seen_at
```

唯一约束：

```text
(advertising_account_id, campaign_id)
```

---

## 42.2 `ad_campaign_snapshot`

Campaign 配置和状态需要历史。

建议字段：

```text
campaign_snapshot_id
ad_campaign_id_internal

state
placement_json
payment_type

product_campaign_mode
product_autopilot_strategy
expense_strategy
budget_type

daily_budget_raw
weekly_budget_raw
budget_raw

observed_at
raw_payload_id
```

由于 Performance Campaign List 预算字段存在特殊 Scale：

Raw 值必须保留。

标准化金额可以单独生成，但不得覆盖 Raw。

---

# 43. Ad Campaign Daily Metric

## 43.1 `ad_campaign_daily_metric`

粒度：

```text
campaign + day
```

建议字段：

```text
ad_campaign_daily_metric_id
ad_campaign_id_internal
shop_id
business_date

views
clicks
spend_amount
spend_currency
orders
orders_amount
orders_currency
to_cart

ctr_raw
cpc_raw
drr_raw

raw_payload_id
```

无数据日期可能没有记录。

因此：

```text
row missing
```

不能自动补成全部指标 `0`，除非指标计算规则明确允许。

---

# 44. Ad Campaign Period Metric

Campaign 区间统计可以作为对账和临时查询结果。

建议：

```text
ad_campaign_period_metric
```

字段：

```text
ad_campaign_period_metric_id
ad_campaign_id_internal
period_from
period_to
views
clicks
spend_amount
orders
orders_amount
to_cart
raw_payload_id
```

长期趋势主事实仍优先使用 Daily。

---

# 45. Ad Campaign SKU Daily Metric

## 45.1 `ad_campaign_sku_daily_metric`

粒度：

```text
Campaign + Day + SKU
```

建议字段：

```text
ad_campaign_sku_daily_metric_id
ad_campaign_id_internal
shop_id

product_id_internal
sku
business_date

views
clicks
spend_amount
spend_currency
orders
sales_amount
sales_currency

ctr_raw
cpc_raw
drr_raw

fetched_at
raw_payload_id
```

这是高价值不可补回数据。

由于当前 API 只允许今天 / 昨天：

每条成功结果都应作为 Historical Fact 持久化。

---

# 46. Ad Campaign Object

## 46.1 `ad_campaign_object`

当前状态：

PARTIAL。

建议字段：

```text
ad_campaign_object_id
ad_campaign_id_internal
object_id_raw
candidate_sku
mapping_status
observed_at
raw_payload_id
```

当前样本中：

```text
object_id == sku
```

不能升级为永久数据库约束。

必须保留：

```text
mapping_status
```

---

# 47. Performance Phrases

当前未取得非空真实 Rows。

因此 DATA_MODEL v1.0 不定义正式 `ad_phrase_metric` 业务表。

Raw 异步任务和报表可以保存，但不能建立依赖其字段的核心业务 Contract。

未来获得真实非空数据后再升级数据模型。

---

# 48. Question

## 48.1 `question`

建议字段：

```text
question_id_internal
shop_id

question_id
product_id_internal
sku

text
status
answers_count
created_at_source
product_url

author_name
raw_payload_id
created_at
updated_at
```

`author_name` 属于个人相关字段，应由 `SECURITY.md` 控制存储与展示。

---

# 49. Answer

## 49.1 `answer`

建议字段：

```text
answer_id_internal
question_id_internal
shop_id

answer_id
text
author_name
published_at
status_publication

raw_payload_id
```

Question 与 Answer：

```text
Question 1 → N Answer
```

不能只依赖 Question Detail 获取答案正文。

---

# 50. Review

Review 当前受订阅权限限制。

数据模型可以预留实体定义，但不可假定当前 Shop 有数据。

---

## 50.1 `review`

建议预留：

```text
review_id_internal
shop_id
review_id
product_id_internal
sku
rating
text
created_at_source
raw_payload_id
```

其数据可用性由：

```text
shop_capability
```

决定。

当当前能力为 `UNAVAILABLE` 时，不创建伪造的 0 Review Facts。

---

# 51. Shop Rating

## 51.1 `shop_rating_snapshot`

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
raw_payload_id
```

当前 Snapshot 可以看作：

```text
period_from = period_to = current observation
```

History 则使用 Ozon 返回的日区间。

---

# 52. FBS Error Index

## 52.1 `fbs_error_index_snapshot`

建议字段：

```text
fbs_error_index_snapshot_id
shop_id
index_value
processing_cost_amount
processing_cost_currency_state
observed_at
raw_payload_id
```

---

## 52.2 `fbs_error_defect_daily`

```text
fbs_error_defect_daily_id
shop_id
business_date
defect_value
raw_payload_id
```

---

## 52.3 `fbs_error_posting`

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
raw_payload_id
```

但这些非空字段在获得真实样本前不能被视为已验证 Contract。

---

# 53. Webhook Capability

## 53.1 `webhook_push_type`

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
raw_payload_id
```

它是配置观察，不代表 O3Pilot 会修改订阅。

---

# 54. Webhook Event

## 54.1 `webhook_event`

Webhook 原始事件必须先落 Raw。

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
raw_payload_id
```

由于真实 Payload Contract 尚未完全验证：

除 Raw Payload 外，所有提取字段都应允许 `NULL`。

---

## 54.2 Webhook 幂等

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

# 55. Ozon Official Report Import

## 55.1 `import_file`

统一表示用户主动导入的文件。

建议字段：

```text
import_file_id
shop_id
source_system

file_name
file_hash
mime_type
file_size

report_type
report_period_from
report_period_to

parser_version
imported_at
status
```

---

## 55.2 `import_batch`

一次文件解析过程。

```text
import_batch_id
import_file_id
shop_id
started_at
finished_at
status
rows_total
rows_success
rows_failed
parser_version
error_summary
```

---

## 55.3 `import_row`

可选 Raw 行层。

建议用于：

- Ozon Official Reports；
- ERP；
- 物流商；
- Seller Data。

字段：

```text
import_row_id
import_batch_id
source_row_number
raw_row_json
parse_status
entity_type
entity_id
error_message
```

这样导入数据可以逐行追溯。

---

# 56. Reconciliation

## 56.1 `reconciliation_result`

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
Finance Accrual vs Official Finance Report
Webhook vs Seller API
```

对账结果不覆盖任何一侧的原始事实。

---

# 57. Profit Calculation Run

Profit 是 Derived Layer。

DATA_MODEL 只定义可追溯结构，不定义公式。

---

## 57.1 `profit_calculation_run`

建议字段：

```text
profit_calculation_run_id
shop_id
period_from
period_to

reporting_currency
metric_version
allocation_version
fx_version
cost_fallback_version

calculated_at
status
```

---

# 58. Order Profit

## 58.1 `order_profit`

建议字段：

```text
order_profit_id
profit_calculation_run_id
posting_id
mother_order_id
shop_id

revenue_amount
fee_amount
ad_spend_amount
procurement_cost_amount
seller_logistics_cost_amount
other_cost_amount
profit_amount

currency
calculation_details_json
```

所有 Component 必须能追溯：

- Finance；
- Order Item Cost；
- Ad Metric；
- Seller Costs；
- Money Conversion。

---

# 59. Order Item Profit

## 59.1 `order_item_profit`

建议字段：

```text
order_item_profit_id
profit_calculation_run_id
posting_item_id
shop_id

revenue_amount
allocated_fee_amount
allocated_ad_spend_amount
procurement_cost_amount
allocated_other_cost_amount
profit_amount

currency
calculation_details_json
```

任何 Posting 级费用拆到 SKU 时，都必须引用：

```text
finance_allocation
```

而不是伪装成原始 SKU Finance Fact。

---

# 60. Forecast

Forecast 属于 Derived Layer。

## 60.1 `forecast_run`

```text
forecast_run_id
shop_id
forecast_type
model_name
model_version
training_data_from
training_data_to
created_at
```

---

## 60.2 `forecast_point`

```text
forecast_point_id
forecast_run_id
product_id_internal
sku
target_date
forecast_value
lower_bound
upper_bound
unit
```

---

## 60.3 `forecast_backtest`

```text
forecast_backtest_id
forecast_run_id
target_date
actual_value
forecast_value
error_value
metric_version
```

具体预测准确率进入 `METRICS.md`。

---

# 61. Recommendation

## 61.1 `recommendation`

建议字段：

```text
recommendation_id
shop_id
recommendation_type

entity_type
entity_id

generated_at
valid_until

priority
status

input_snapshot_json
model_version
explanation_json
```

例如：

```text
REPLENISHMENT
CLEARANCE
PRICE
AD_BID
AD_BUDGET
PRODUCT_OPTIMIZATION
```

Recommendation 永远不能直接转化成 Ozon 写操作。

---

# 62. Alert

## 62.1 `alert`

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

Alert 引用其他域的数据，而不是复制业务事实。

---

# 63. 数据可用性

为明确区分“0”和“没有数据”，建议建立：

## 63.1 `data_availability`

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

# 64. 时间模型

所有时间字段必须明确语义。

## 64.1 Timestamp

原始 Timestamp：

- 保留 Source Time；
- 标准化为带时区时间；
- 不因前端显示 UTC+8 而覆盖原值。

---

## 64.2 Business Date

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

## 64.3 默认显示时区

O3Pilot 默认：

```text
Asia/Shanghai
```

这只是展示和默认统计时区。

不是所有来源的业务日边界。

---

# 65. Snapshot 模型

以下数据默认使用 Snapshot 建立历史：

- Product；
- Product Attributes；
- Product Content Rating；
- Price；
- Inventory；
- Inventory Turnover；
- Campaign；
- Shop Rating；
- Shop Info；
- Capability。

Snapshot 至少包含：

```text
observed_at
```

相同内容是否去重属于实现层优化，不改变业务历史语义。

---

# 66. 事实表与快照表区别

## Fact

代表已经发生的业务事实，例如：

- Posting；
- Posting Item；
- Return；
- Finance Accrual；
- Question；
- Answer；
- Campaign Daily Metric。

Fact 不应因为下一次同步不存在就自动删除。

## Snapshot

代表某一时点观察，例如：

- Price；
- Inventory；
- Campaign State；
- Rating Current；
- Product Content Rating。

---

# 67. 删除与消失

API 全量列表中对象消失时：

不得物理删除历史实体。

例如 Campaign 曾发生短暂空集合。

正确处理是：

```text
last_seen_at
is_active / lifecycle_state
source_missing_since
```

而不是：

```text
DELETE FROM campaign
```

同样适用于：

- Product；
- Inventory Source Rows；
- Webhook Capability。

---

# 68. 主键与唯一约束总览

| 实体 | 推荐 Source Unique Key |
|---|---|
| Shop | `ozon_client_id` |
| Product | `(shop_id, ozon_product_id)` |
| Product Identifier History | `(product_id_internal, sku, offer_id, valid_from)` |
| MotherOrder | `(shop_id, order_number)` |
| Posting | `(shop_id, posting_number)` |
| PostingItem | Internal ID；不强制假设 SKU 唯一 |
| ReturnCase | `(shop_id, return_source_type, source_return_id)` |
| FinanceAccrual | `(shop_id, accrual_id)` |
| FinanceAccrualType | `type_id` |
| SKU Daily Analytics | `(shop_id, sku, business_date)` |
| Search Period Analytics | `(shop_id, sku, period_from, period_to)` |
| Search Query Analytics | `(shop_id, sku, query, period_from, period_to)` |
| AdCampaign | `(advertising_account_id, campaign_id)` |
| AdCampaignDaily | `(ad_campaign_id_internal, business_date)` |
| AdCampaignSkuDaily | `(ad_campaign_id_internal, sku, business_date)` |
| Question | `(shop_id, question_id)` |
| Answer | `(shop_id, answer_id)` |
| Rating History | `(shop_id, rating_code, period_from, period_to)` |
| Ozon FX | `(from_currency, to_currency, valid_from, valid_to)` |
| Import File | `file_hash` + import context |

真正数据库约束需在实现前根据所有真实源字段再次验证。

---

# 69. 核心关系总览

```text
SellerCatalogItem
└── SellerCatalogProductMapping
    └── Shop
        │
        ├── Product
        │   ├── ProductIdentifierHistory
        │   ├── ProductSnapshot
        │   ├── ProductAttributeSnapshot
        │   ├── ProductContentRatingSnapshot
        │   ├── ProductPriceSnapshot
        │   ├── InventorySnapshot
        │   ├── InventoryTurnoverSnapshot
        │   ├── SkuDailyAnalytics
        │   ├── SearchAnalytics
        │   └── SellerSkuCost
        │
        ├── InboundSupply
        │   └── InboundSupplyItem
        │
        ├── MotherOrder
        │   └── Posting
        │       ├── PostingItem
        │       │   ├── PostingItemPricingObservation
        │       │   ├── OrderItemCost
        │       │   └── OrderItemProfit
        │       ├── PostingStatusHistory
        │       ├── PostingScheduleHistory
        │       ├── LogisticsEvent
        │       ├── ShipmentPackage
        │       │   └── SellerLogisticsCharge
        │       ├── Cancellation
        │       ├── ReturnCase
        │       │   ├── ReturnItem
        │       │   └── ReverseLogisticsEvent
        │       ├── FinanceAccrual
        │       │   └── FinanceComponent
        │       │       └── FinanceAllocation
        │       └── OrderProfit
        │
        ├── SettlementPeriod
        │   └── Payout
        │
        ├── Warehouse
        ├── LogisticsProvider
        ├── DeliveryMethod
        │
        ├── AdvertisingAccount
        │   └── AdCampaign
        │       ├── AdCampaignSnapshot
        │       ├── AdCampaignDailyMetric
        │       ├── AdCampaignSkuDailyMetric
        │       └── AdCampaignObject
        │
        ├── Question
        │   └── Answer
        │
        ├── ShopRatingSnapshot
        ├── FbsErrorIndexSnapshot
        ├── WebhookEvent
        ├── DataAvailability
        ├── DataQualityIssue
        └── ImportFile
            └── ImportBatch
                └── ImportRow

OzonExchangeRateInterval
└── MoneyConversion

Derived
├── ProfitCalculationRun
├── ForecastRun
├── Recommendation
└── Alert
```

---

# 70. 跨域关键连接字段

## Product

核心跨域键：

```text
product_id_internal
sku
offer_id
ozon_product_id
```

其中内部关系优先使用：

```text
product_id_internal
```

---

## Order

核心跨域键：

```text
posting_id
posting_number
mother_order_id
order_number
```

---

## Finance

优先关联：

```text
posting_number
sku
```

必须先解析到内部：

```text
posting_id
posting_item_id
```

无法可靠关联时：

```text
mapping_status = UNMATCHED
```

不得猜测。

---

## Returns

优先：

```text
posting_number
return_id
```

`order_number` 可以用于辅助，但 Detail 中可能为空。

---

## ERP Cost

优先：

```text
订单编号 → posting_number
```

然后根据：

```text
平台SKU
商品映射
```

关联到 `posting_item_id`。

---

## Seller Logistics

优先：

```text
订单号 / 发货号
→ posting_number
```

再根据物流商原始字段关联：

```text
shipment_package_id
logistics_provider_id
```

无法可靠识别包裹时，可以保留 Posting 级 Seller Logistics Fact，但不能猜测 SKU 归属。

---

## Seller Catalog

跨店商品通过显式：

```text
seller_catalog_product_mapping
```

关联。

不得仅通过：

```text
offer_id
name
sku
```

自动合并。

---

# 71. Mapping Status

所有跨系统映射建议使用：

```text
MATCHED_EXACT
MATCHED_RULE
AMBIGUOUS
UNMATCHED
UNVERIFIED
```

例如 ERP SKU 与 Ozon 商品映射不能只返回 NULL。

必须知道为什么没有匹配。

---

# 72. 数据版本

以下规则必须版本化：

```text
parser_version
product_mapping_version
fulfillment_mapping_version
finance_allocation_version
fx_mapping_version
cost_mapping_version
metric_version
forecast_model_version
recommendation_model_version
```

历史派生结果必须能够解释使用的是哪一版规则。

---

# 73. 不进入 Normalized Fact 的数据

以下内容在没有可靠证据前只能停留 Raw / Partial：

- Performance Phrases 行级业务字段；
- Analytics Stocks 非空 Item 结构；
- Review List / Detail 当前真实字段；
- FBS Error Posting 非空行；
- 未验证 Webhook Payload 字段；
- 无可靠来源的竞争对手库存；
- 无可靠来源的竞争对手销量。

不得为了“数据库字段完整”提前虚构 Schema。

---

# 74. 数据安全边界

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

Recommendation 只表示建议。

---

# 75. Phase 0 最低数据模型

第一阶段真正必须落地的实体至少包括：

```text
shop
shop_capability

raw_payload
sync_run
data_quality_issue

product
product_identifier_history
product_snapshot
product_attribute_snapshot
product_price_snapshot
inventory_snapshot

mother_order
posting
posting_item
posting_status_history
cancellation

return_case
return_item

finance_accrual_type
finance_accrual
finance_component

sku_daily_analytics
sku_search_period_analytics
sku_search_query_analytics
inventory_turnover_snapshot

advertising_account
ad_campaign
ad_campaign_snapshot
ad_campaign_daily_metric
ad_campaign_sku_daily_metric

shop_rating_snapshot

question
answer

webhook_event

ozon_exchange_rate_interval

import_file
import_batch
import_row

seller_catalog_item
seller_catalog_product_mapping

warehouse
logistics_provider
delivery_method

inbound_supply
inbound_supply_item

shipment_package
seller_logistics_charge

seller_sku_cost
order_item_cost

settlement_period
payout

posting_item_pricing_observation
posting_schedule_history

data_availability
```

其他 Derived 实体可在对应 Phase 实施。

---

# 76. Phase 1 扩展

Core Operations 阶段增加或强化：

```text
logistics_event
reverse_logistics_link
reverse_logistics_event
fbs_error_index_snapshot
fbs_error_defect_daily
alert
```

---

# 77. Phase 2 扩展

Finance & Profit 阶段增加：

```text
finance_allocation
money_conversion
profit_calculation_run
order_profit
order_item_profit
```

---

# 78. Phase 3 扩展

Growth Analytics 阶段增加：

```text
review
ad_campaign_object
VOC derived models
product optimization outputs
```

仅当对应数据源已经可用。

---

# 79. Phase 4 扩展

Decision Support 增加：

```text
forecast_run
forecast_point
forecast_backtest
recommendation
```

---

# 80. DATA_MODEL 与 METRICS 的边界

DATA_MODEL 定义：

```text
数据是什么
数据之间什么关系
数据如何追溯
```

METRICS 定义：

```text
如何计算
```

例如产品退货率：

DATA_MODEL 只需要保证存在：

```text
PostingItem
ReturnItem
ReturnCase
Product
Time
```

至于：

- 分子；
- 分母；
- 件数口径；
- 订单口径；
- 跨期处理；

全部进入 `METRICS.md`。

---

# 81. DATA_MODEL 与 DATA_SOURCES 的边界

DATA_SOURCES 定义：

```text
数据从哪里来
来源能提供什么
来源有哪些限制
```

DATA_MODEL 定义：

```text
拿到以后怎么统一保存
```

因此任何新增实体必须能够回溯到：

- 已验证数据源；
- 卖家明确提供的数据；
- 明确的 Derived 过程。

---

# 82. 当前跨文档同步状态

本版与 `DATA_SOURCES.md v1.0` 已同步以下内容：

1. Ozon Exchange Rate XAPI；
2. Ozon Business FX 与 Seller Cost FX 分离；
3. 马帮 ERP 订单商品实际采购成本 Adapter；
4. ERP 成本支持订单级历史；
5. Seller Cost Currency / FX Mapping 版本化；
6. 物流商 / 卖家物流费用作为 Seller-Owned Data；
7. 包裹实际重量、体积重量和计费重量分层；
8. 在途库存 / Inbound Supply；
9. Ozon 官方结算与付款文件作为 Settlement / Payout 的人工来源；
10. 发货截止时间和承诺配送窗口历史；
11. 跨店 Seller Catalog 显式映射。

后续 `METRICS.md` 应直接建立在上述数据模型上，不再重新发明另一套成本、汇率、库存或退货基础结构。

---

# 83. 当前待验证的数据模型问题

以下问题保持开放，不在 v1.0 中猜测：

1. Analytics Stocks 非空结构；
2. Performance Phrases 非空结构；
3. Review List / Detail 真实结构；
4. FBS Error Posting 非空结构；
5. Webhook 各 Push Type 完整 Payload；
6. Type 29 / 32 在 Finance 多 SKU Posting 中的实际粒度；
7. Ozon Exchange Rate XAPI 的长期稳定性；
8. 马帮 ERP 未来版本中 `汇率(原币)` 的字段语义是否保持一致；
9. 其他 ERP / Seller Cost Source 的成本基准币种；
10. WHD 二次销售与原退货的可靠一对一关联字段；
11. Ozon Inbound Supply 是否存在稳定且完整的只读 API Contract；
12. 物流商导出数据对 Posting / Package 的稳定关联键；
13. 不同履约模式下包裹实际重量、体积重量和计费重量来源优先级；
14. Settlement / Payout 是否存在可替代人工报告的稳定只读 API；
15. 跨店 Seller Catalog 自动辅助匹配规则的可靠边界。

---

# 84. 数据模型验收原则

在数据库正式实现前，本模型必须满足：

1. 所有 Ozon 业务 Fact 都有 Shop；
2. 所有 Source Key 与 Internal ID 分离；
3. Product ID / SKU / offer_id 不混用；
4. MotherOrder / Posting 不混用；
5. FBP / realFBS / OGL 与 API 技术 schema 分离；
6. WHD 不进入正常首次履约模式；
7. Finance SKU Fact 与 Posting Fact 分离；
8. Posting 级费用不会伪造成 SKU 原始费用；
9. Ozon FX 与 Seller Cost FX 分离；
10. 原始 Money 不被换算金额覆盖；
11. 历史采购成本不会被当前成本覆盖；
12. ERP Cost 可以关联到具体 Posting Item；
13. Webhook Raw Payload 可追溯；
14. Webhook 不作为最终状态唯一来源；
15. 无权限数据不会存成 0；
16. 空集合不会触发无条件历史删除；
17. Snapshot 与 Fact 区分；
18. 派生结果具有规则版本；
19. 每个正式指标都有可追溯数据基础；
20. 未验证接口不会提前生成虚假业务字段；
21. Product Weight、Package Weight、Chargeable Weight 不混用；
22. Ozon Finance Logistics Fee 与 Seller Logistics Cost 分离；
23. 当前库存与在途库存分离；
24. Posting 当前承诺时间不会覆盖其历史承诺时间；
25. Settlement / Payout 不会由 Accrual 推断伪造；
26. 跨店商品只通过显式 Seller Catalog Mapping 合并；
27. FX Conversion 明确保存 Rate Basis；
28. 所有 Ozon 集成保持严格只读。

---

# 85. 核心原则

**先保存原始事实，再建立业务解释。**

**内部身份与外部字段分离。**

**订单以 Posting 为核心连接点，母订单负责聚合。**

**商品三个标识全部保留，不把 offer_id 当永久主键。**

**技术履约模式不等于业务履约模式。**

**WHD 属于逆向物流。**

**Finance 的原始粒度不得被分摊结果替代。**

**原始金额永远保留原币。**

**Ozon Business FX 与 Seller Cost FX 永远分离。**

**历史订单优先使用历史实际采购成本。**

**Product Weight、Package Weight 与 Chargeable Weight 分离。**

**Ozon Finance 物流费用与卖家实际物流成本分离。**

**当前库存与在途库存分离。**

**跨店商品只通过显式 Seller Catalog 映射合并。**

**汇率必须记录适用时间依据，而不是只有一个笼统业务时间。**

**未知不等于 0。**

**无法可靠关联时标记 Unmatched，不猜测。**

**所有派生结果必须可以重新计算和解释。**

**O3Pilot 永远只读 Ozon。**
