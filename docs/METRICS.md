# O3Pilot — METRICS.md

> Version: 1.1  
> Status: Active Metric Baseline  
> Updated: 2026-09-05  
> Revision: Batch A.3  
> Applies to: O3Pilot

---

# 1. 文档目的

`METRICS.md` 定义 O3Pilot 当前正式指标的唯一口径、计算规则、时间语义、聚合方式、结果状态和版本语义。

本文件负责回答：

- 当前 P0 / P1 哪些 Metric 是正式 Active；
- 一个 Metric 的身份、来源或计算公式是什么；
- 分子、分母、资格规则和排除规则是什么；
- 使用哪个业务时间、粒度和币种；
- Source、Ozon Reference、O3Pilot Derived、Estimate 如何区分；
- Missing、0、未知、无权限、未验证、开放 Cohort 如何区分；
- Ratio、Money、Percentile 如何正确聚合；
- Persisted Derived Result 如何保留历史重算能力；
- Deferred Metric 何时允许重新进入 Active Baseline。

本文件不定义：

- API Endpoint、Pagination、真实历史窗口和 Endpoint Verification；
- 数据库实体、字段、自然键和 Source Lineage Schema；
- 数据采集频率、Scheduler、Worker、Job 实现；
- 页面布局和视觉交互；
- Alert / Recommendation 的业务触发阈值和策略生命周期；
- Forecast 模型实现；
- 任何 Ozon 写操作。

这些内容分别由 `DATA_SOURCES.md`、`DATA_MODEL.md`、`ARCHITECTURE.md`、`DESIGN.md`、`ALERTS_AND_RECOMMENDATIONS.md`、`FORECASTING.md` 等对应 Authority 定义。

O3Pilot 对 Ozon 永久保持只读。

---

# 2. Upstream Authorities & Frozen Constraints

A.3 的迁移依据来自 Batch 0 Final，但 A.3 完成后，当前 Metric Contract 的正式含义必须由本文件自身完整表达；实现者不需要再次读取 Batch 0 才能解释当前 Metric Contract。

| Authority | Single Authority Area |
|---|---|
| `PRODUCT.md` | 产品能力边界与 Feature Phase |
| `DATA_SOURCES.md` | Endpoint、Pagination、Backfillability 证据、真实窗口与验证状态 |
| `DATA_MODEL.md` | Active Schema、业务身份、Source Lineage 与跨层 Phase 执行映射 |
| `ARCHITECTURE.md` | Runtime、数据流、Time Semantics 与 Observability |
| Batch 0 Final | 本次 A.3 迁移的冻结溯源与执行依据 |
| `METRICS.md` | 当前正式 Metric Registry、Definition Contract、Result Semantics 与计算规则 |

本文件消费上述 Authority，不独立重定义 Feature Phase、Endpoint 能力或底层 Schema。

Batch 0 对 A.3 的有效冻结约束已吸收如下：

```text
v1 Feature Delivery = P0 + P1

Feature Phase != Acquisition Phase

Unified Metric Contract:
17 fields
→
Source Contract = 8 fields
Derived Contract = 9 fields

Future-phase formal Metric definitions in v1 Active Body = 0

One Name = One Meaning

OZON_REFERENCE remains distinct

Ozon remains read-only
```

Metric Phase 的直接执行映射以当前 `DATA_MODEL.md §25` 为准，并必须与 `PRODUCT.md` 的 Feature Phase Authority 一致。

特别注意：

```text
Search Analytics
Acquisition = P0
Metrics / Feature = P3
```

以及：

```text
Campaign SKU Daily
Acquisition = P0
Feature = P3
```

提前采集不意味着提前实现对应 Metric。

---

# 3. Metric Identity 与 Origin

## 3.1 One Name = One Meaning

一个 `metric_code` 只能对应一个正式口径。

不得让同一个 Code 在不同页面、来源或时间窗口下静默改变：

- 分子；
- 分母；
- Eligibility；
- Exclusions；
- Time Basis；
- Currency；
- Formula。

UI 显示名称变化不得自动改变 `metric_code`。

---

## 3.2 Active `metric_origin`

当前 Active Metric Origin 只有：

```text
OZON_SOURCE
OZON_REFERENCE
O3P_DERIVED
O3P_ESTIMATE
```

`O3P_FORECAST` 随 P4 Forecast 一并 Deferred，不属于当前 Active Origin 集。

### `OZON_SOURCE`

Ozon API / 已验证 Ozon Source 直接返回且 O3Pilot 不改变语义的值。

### `OZON_REFERENCE`

Ozon 官方知识库、Seller Center 或官方报告明确给出的指标、公式或参考口径。

### `O3P_DERIVED`

O3Pilot 根据已标准化 Fact 按确定公式计算。

### `O3P_ESTIMATE`

依赖估算假设、缺失补全或近似业务模型。

Estimate 必须通过 Result `status` 或等价可识别语义向用户暴露，不得伪装成 Source Fact。

---

## 3.3 `metric_origin != contract_shape`

Origin 描述指标语义来源；Contract Shape 描述这个指标如何被 O3Pilot 表达。

正式规则：

```text
OZON_SOURCE
→ SOURCE

O3P_DERIVED
→ DERIVED

O3P_ESTIMATE
→ DERIVED
```

`OZON_REFERENCE` 根据实际形态选择：

```text
官方直接可指认观察值 / 字段
→ SOURCE

O3Pilot 按官方公布公式复算
→ DERIVED
```

`OZON_REFERENCE + SOURCE` 必须存在可指认的官方 `source_field`；禁止为了满足 Source 8 而填写虚假的 `N/A` 来源字段。

---

# 4. Metric Registry 与 Definition Contract

## 4.1 Metric Registry — 6 fields

Metric Registry 只承担身份与治理元数据：

```text
metric_code
display_name
metric_origin
contract_shape
domain
feature_phase
```

Registry 不重复维护 `grain`、`time_basis`、`unit` 等 Definition Contract 字段。

---

## 4.2 Source Contract — 8 fields

所有 `contract_shape = SOURCE` 的正式 Metric 必须完整定义：

```text
metric_code
source
source_field
grain
time_basis
unit
currency_policy
aggregation_rule
```

---

## 4.3 Derived Contract — 9 fields

所有 `contract_shape = DERIVED` 的正式 Metric 必须完整定义：

```text
metric_code
grain
time_basis
formula
eligibility
unit
currency_policy
aggregation_rule
metric_version
```

当前 v1.1 Active Derived Metric 若无单独说明，初始：

```text
metric_version = 1
```

任何公式语义变化必须提升版本，不能覆盖旧 Result 的解释能力。

---

## 4.4 Optional Extensions

Batch 0 Frozen：

```text
numerator
denominator
exclusions
coverage
```

A.3 基于当前真实 Active Metric 额外保留：

```text
window
```

`window` 的保留理由是当前 ADS、Availability、Lost Sales 和 Ozon Reference 中存在真实窗口依赖；删除会直接丢失指标口径。

`dimensions` 从 Active Contract 移除。

重新进入条件：

> 出现真实 Active Metric，证明合法切片无法由 `grain` + 明确 Variant / Result Context 表达时，再重新 Review。

任何未来新增 Contract 字段必须同时说明：

1. 字段名；
2. 真实使用证据；
3. 现有字段为什么无法表达；
4. Required / Optional。

禁止使用没有明确边界的兜底字段重新扩张 Contract。

---

## 4.5 旧 17 字段迁移

| Old Field | v1.1 Authority |
|---|---|
| `metric_code` | Source / Derived Contract |
| `display_name` | Metric Registry |
| `metric_origin` | Metric Registry |
| `domain` | Metric Registry |
| `grain` | Source / Derived Contract |
| `dimensions` | Removed from Active Contract；Deferred with Re-entry Gate |
| `time_basis` | Source / Derived Contract |
| `window` | A.3 justified Optional Extension |
| `numerator` | Optional Extension |
| `denominator` | Optional Extension |
| `unit` | Source / Derived Contract |
| `currency_policy` | Source / Derived Contract |
| `eligibility` | Derived Contract |
| `exclusions` | Optional Extension |
| `aggregation_rule` | Source / Derived Contract |
| `metric_version` | Derived Contract |
| `status` | Metric Result Semantics |

迁移结果：

```text
17 / 17 mapped
```

Source 8 中的：

```text
source
source_field
```

不是旧 17 字段迁移项；它们是 Batch 0 拆分 Source Contract 后冻结的来源指针。

---

# 5. Metric Result Semantics

本文件不建立第三套“六字段强制 Result Contract”。

所有正式 Metric Result 必须能够表达：

```text
value
status
```

该要求同样适用于 `OZON_SOURCE`：`value` 为 Source Value，`status` 表达该结果当前是否可用、可靠或适用。

以下字段按 Metric 适用性提供：

```text
sample_count
eligible_count
coverage_ratio
as_of_time
```

当前统一结果状态：

```text
VALID
PARTIAL
UNAVAILABLE
NOT_APPLICABLE
UNVERIFIED
STALE
ESTIMATED
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
```

Missing / Unknown 不得被写成业务 `0`。

Denominator 为 0：

```text
value = NULL
status = NO_DENOMINATOR
```

库存覆盖天数在近期有效需求为 0 时：

```text
value = NULL
status = NO_RECENT_DEMAND
```

当同一个 Result 同时存在多个风险时，建议主状态优先级：

```text
UNAVAILABLE
UNVERIFIED
STALE
PARTIAL
ESTIMATED
OPEN_COHORT
VALID
```

这是建议优先级，不替代 Metric-specific 状态语义。

Ratio 展示至少应能查看：

```text
value
numerator
denominator
period
status
```

Percentile 展示至少应能查看：

```text
value
sample_count
period
status
```

Money 展示至少应能查看：

```text
amount
currency
period
```

---

# 6. Historical Recalculation Invariant

触发条件：

```text
Persisted Metric Result
AND
contract_shape = DERIVED
```

必须能够追溯：

```text
metric_code
metric_version
calculated_at
source_coverage
```

这条规则同时覆盖：

- `O3P_DERIVED`；
- `O3P_ESTIMATE`；
- `OZON_REFERENCE + DERIVED`。

METRICS 不再要求所有派生结果拥有通用：

```text
calculation_run_id
```

如果未来某个 Domain 存在真实 Run Entity，其关系由该 Domain / `DATA_MODEL.md` 单独定义，不反向变成所有 Metric 的统一要求。

以下三个概念保持分离：

```text
coverage
→ Complex Metric Definition 中按需声明的 coverage requirement

coverage_ratio
→ 某次 Metric Result 在适用时提供的覆盖度结果

source_coverage
→ Persisted Derived Result 的来源覆盖追溯信息
```

本文件不提前为 `source_coverage` 发明新的通用 JSON Schema。

---

# 7. 通用计算不变量

## 7.1 Ratio 不平均 Ratio

跨 SKU、店铺、日期聚合比例时：

```text
SUM(numerator)
/
SUM(denominator)
```

禁止：

```text
AVG(ratio)
```

除非产品明确要求展示“多个独立 Ratio 的简单平均”。

---

## 7.2 Money 不跨币种直接相加

允许：

```text
100 USD + 50 USD
```

禁止：

```text
100 USD + 500 CNY
```

跨币种分析必须：

1. 保留原始 Money 与 Currency；
2. 使用已定义 FX；
3. 保存 Rate Basis；
4. 转换到明确 Reporting Currency；
5. 不覆盖 Source Money。

---

## 7.3 Percentile 从原始样本重算

P50 / P90：

```text
P50 = percentile(samples, 0.50)
P90 = percentile(samples, 0.90)
```

禁止：

```text
AVG(daily_p50)
```

作为整个 Period 的 P50。

---

## 7.4 Period & Baseline Comparison

基础变化：

```text
delta = current - baseline
```

变化率：

```text
delta_rate =
(current - baseline)
/
abs(baseline)
```

如果：

```text
baseline = 0
```

返回：

```text
NULL + NO_DENOMINATOR
```

旧的通用：

```text
o3p.sales.growth_rate
o3p.sales.unit_growth_rate
```

不再作为可切换底层 Source 的独立正式 Metric Code；具体趋势使用被比较的 Canonical Metric + 本节 Comparison Rule，避免一个 Code 对应多个含义。

“变化”本身不等于“异常”。Threshold、Seasonality、Anomaly Model 属于应用策略 Authority。

---

# 8. Order Eligibility 与 Cohort

## 8.1 默认经营订单资格

O3Pilot 默认经营订单集合：

```text
eligible_business_posting
=
created posting
AND NOT confirmed_pre_shipment_cancelled
```

以下 Posting 仍属于有效经营集合：

- 待备货且未取消；
- 待发货且未取消；
- 已发货；
- 运输中；
- 已签收；
- 发货后取消；
- 其他未确认属于发货前取消的有效订单。

因此：

```text
shipped_at IS NULL
!=
invalid order
```

不能因为尚未发货就从 Sales、AOV、ADS 或默认经营指标删除。

---

## 8.2 Created Demand 独立

原始下单需求必须单独统计：

```text
Created Demand
```

它包含后来发生发货前取消的订单。

因此：

```text
Created Demand
- confirmed pre-shipment cancellation
=
Eligible Sales Order Set
```

---

## 8.3 Source / Reference 不继承 O3Pilot Eligibility

`OZON_SOURCE` 和 `OZON_REFERENCE` 保持官方原口径，不自动套用 `eligible_business_posting`。

---

## 8.4 Open Cohort

近期订单、退货、认购等结果可能继续变化。

可以：

```text
status = OPEN_COHORT
```

不得因为实现方便而为所有类目硬编码一个统一“成熟天数”。

---

# 9. Active Metric Registry

当前 v1 Active Metric 只允许 P0 / P1。


| metric_code | display_name | metric_origin | contract_shape | domain | feature_phase |
|---|---|---|---|---|---|
| `o3p.data.coverage_lag` | Data Coverage Lag | O3P_DERIVED | DERIVED | Data Quality | P0 |
| `o3p.data.mapping_coverage` | Mapping Coverage | O3P_DERIVED | DERIVED | Data Quality | P0 |
| `o3p.product.buyout_rate` | O3Pilot Product Buyout Rate | O3P_DERIVED | DERIVED | Buyout | P1 |
| `o3p.product.non_buyout_rate` | Product Non-buyout Rate | O3P_DERIVED | DERIVED | Buyout | P1 |
| `ozon.reference.buyout_rate_last_50_orders` | Ozon Buyout Rate — Last up to 50 Orders Reference | OZON_REFERENCE | DERIVED | Buyout | P1 |
| `o3p.cancellation.buyer_rate` | Buyer-responsible Post-shipment Cancellation Rate | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `o3p.cancellation.posting_rate` | Post-shipment Posting Cancellation Rate | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `o3p.cancellation.pre_shipment_rate` | Pre-shipment Cancellation Rate | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `o3p.cancellation.seller_responsible_rate` | Seller-responsible Post-shipment Cancellation Rate | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `o3p.cancellation.stage_coverage` | Cancellation Stage Coverage | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `o3p.cancellation.unit_rate` | Post-shipment Cancelled Unit Rate | O3P_DERIVED | DERIVED | Cancellation | P1 |
| `ozon.reference.cancellation_rate_7d_count` | Ozon Cancellation Rate 7D by Shipment Count | OZON_REFERENCE | DERIVED | Cancellation | P1 |
| `ozon.reference.cancellation_rate_7d_value` | Ozon Cancellation Rate 7D by Shipment Value | OZON_REFERENCE | DERIVED | Cancellation | P1 |
| `o3p.demand.created_mother_order_count` | Created Mother Order Count | O3P_DERIVED | DERIVED | Demand | P1 |
| `o3p.demand.created_order_gross_value` | Created Order Gross Value | O3P_DERIVED | DERIVED | Demand | P1 |
| `o3p.demand.created_ordered_units` | Created Ordered Units | O3P_DERIVED | DERIVED | Demand | P1 |
| `o3p.demand.created_posting_count` | Created Posting Count | O3P_DERIVED | DERIVED | Demand | P1 |
| `o3p.fulfillment.deadline_shift_hours` | Shipment Deadline Shift Hours | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.delivery_transit_duration` | Handover-to-Delivery Duration | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.on_time_handover_rate` | On-time Handover Rate | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.order_to_delivery_duration` | Order-to-Delivery Duration | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.order_to_handover_duration` | Order-to-Handover Duration | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.processing_to_handover_duration` | Processing-to-Handover Duration | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.promise_delivery_hit_rate` | Promise Delivery Hit Rate | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.fulfillment.shipped_units` | Shipped Units | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.quality.delivery_time_completeness` | Delivery Time Completeness | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `o3p.quality.handover_time_completeness` | Handover Time Completeness | O3P_DERIVED | DERIVED | Fulfillment | P1 |
| `ozon.reference.shipment_delay_rate_14d` | Ozon Shipment Delay Rate 14D Reference | OZON_REFERENCE | DERIVED | Fulfillment | P1 |
| `o3p.inventory.adjusted_ads_units` | Availability-adjusted ADS Units | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.availability_rate` | O3Pilot Availability Rate | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.calendar_ads_units` | Calendar ADS Units | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.days_of_cover` | Current Days of Cover | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.in_transit_stock` | In-transit Stock | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.lost_sales_estimate` | Lost Sales Estimate | O3P_ESTIMATE | DERIVED | Inventory | P1 |
| `o3p.inventory.out_of_stock_days` | Out-of-stock Days | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.reserved_stock` | Reserved Stock | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.sellable_stock` | Sellable Stock | O3P_DERIVED | DERIVED | Inventory | P1 |
| `o3p.inventory.supply_days_of_cover` | Supply Days of Cover | O3P_DERIVED | DERIVED | Inventory | P1 |
| `ozon.reference.product_availability` | Ozon Product Availability Reference | OZON_REFERENCE | DERIVED | Inventory | P1 |
| `o3p.price.competitive_gap` | Competitive Price Gap | O3P_DERIVED | DERIVED | Price | P1 |
| `o3p.price.discount_rate` | Discount Rate | O3P_DERIVED | DERIVED | Price | P1 |
| `ozon.price.color_index` | Ozon Price Color Index | OZON_SOURCE | SOURCE | Price | P1 |
| `ozon.price.external_index_value` | Ozon External Price Index Value | OZON_SOURCE | SOURCE | Price | P1 |
| `ozon.price.marketing_seller_price` | Ozon Marketing Seller Price | OZON_SOURCE | SOURCE | Price | P1 |
| `ozon.price.ozon_index_value` | Ozon Ozon-Index Value | OZON_SOURCE | SOURCE | Price | P1 |
| `ozon.price.self_marketplaces_index_value` | Ozon Self-Marketplaces Index Value | OZON_SOURCE | SOURCE | Price | P1 |
| `ozon.price.seller_price` | Ozon Seller Price | OZON_SOURCE | SOURCE | Price | P1 |
| `o3p.product.content_rating_change` | Content Rating Change | O3P_DERIVED | DERIVED | Product | P1 |
| `o3p.product.missing_recommended_attribute_count` | Missing Recommended Attribute Count | O3P_DERIVED | DERIVED | Product | P1 |
| `ozon.product.content_rating` | Ozon Product Content Rating | OZON_SOURCE | SOURCE | Product | P1 |
| `o3p.product.return_rate` | Product Return Rate | O3P_DERIVED | DERIVED | Returns | P1 |
| `o3p.return.reason_share` | Return Reason Share | O3P_DERIVED | DERIVED | Returns | P1 |
| `o3p.return.request_rate` | Return Request Rate | O3P_DERIVED | DERIVED | Returns | P1 |
| `o3p.whd.recovery_revenue` | WHD Recovery Revenue | O3P_DERIVED | DERIVED | Returns | P1 |
| `o3p.whd.resale_rate` | WHD Resale Rate | O3P_DERIVED | DERIVED | Returns | P1 |
| `o3p.sales.aov_mother_order` | Mother Order AOV | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.aov_posting` | Posting AOV | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.average_unit_price` | Average Unit Price | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.buyer_paid_value` | Buyer Paid Value | OZON_SOURCE | SOURCE | Sales | P1 |
| `o3p.sales.delivered_units` | Delivered Units | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.mother_order_count` | Eligible Mother Order Count | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.order_gross_value` | Eligible Order Gross Value | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.ordered_units` | Eligible Ordered Units | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.post_shipment_cancelled_units` | Post-shipment Cancelled Units | O3P_DERIVED | DERIVED | Sales | P1 |
| `o3p.sales.posting_count` | Eligible Posting Count | O3P_DERIVED | DERIVED | Sales | P1 |
| `ozon.analytics.cancellations` | Ozon Analytics Cancellations | OZON_SOURCE | SOURCE | Sales | P1 |
| `ozon.analytics.delivered_units` | Ozon Analytics Delivered Units | OZON_SOURCE | SOURCE | Sales | P1 |
| `ozon.analytics.ordered_units` | Ozon Analytics Ordered Units | OZON_SOURCE | SOURCE | Sales | P1 |
| `ozon.analytics.returns` | Ozon Analytics Returns | OZON_SOURCE | SOURCE | Sales | P1 |
| `ozon.analytics.revenue` | Ozon Analytics Revenue | OZON_SOURCE | SOURCE | Sales | P1 |
| `ozon.rating.current_value` | Ozon Current Rating Value | OZON_SOURCE | SOURCE | Shop Health | P1 |
| `ozon.rating.fbs_error_index` | Ozon FBS Error Index | OZON_SOURCE | SOURCE | Shop Health | P1 |
| `ozon.rating.history_value` | Ozon Historical Rating Value | OZON_SOURCE | SOURCE | Shop Health | P1 |
| `ozon.rating.status_danger` | Ozon Rating Danger Status | OZON_SOURCE | SOURCE | Shop Health | P1 |
| `ozon.rating.status_premium` | Ozon Rating Premium Status | OZON_SOURCE | SOURCE | Shop Health | P1 |
| `ozon.rating.status_warning` | Ozon Rating Warning Status | OZON_SOURCE | SOURCE | Shop Health | P1 |


---

# 10. Active Source Metric Contracts

Source Metric 保留来源原义，不因为 O3Pilot 存在类似 Derived Metric 就覆盖或改写。



## 10.1 Sales


| metric_code | source | source_field | grain | time_basis | unit | currency_policy | aggregation_rule |
|---|---|---|---|---|---|---|---|
| `o3p.sales.buyer_paid_value` | `posting_item_pricing_observation` | `customer_price_amount` | SHOP + POSTING_ITEM + PERIOD | POSTING_CREATED | MONEY | Use customer_price_currency when explicit; otherwise UNKNOWN; never cross-currency sum | Preserve posting-item observation; higher-grain additive aggregation requires verified source amount semantics |
| `ozon.analytics.revenue` | `sku_daily_analytics` | `revenue` | SHOP + SKU + DAY | BUSINESS_DATE | MONEY | Source currency if explicit; otherwise UNKNOWN; no cross-currency sum | SUM across compatible SKU/day rows |
| `ozon.analytics.ordered_units` | `sku_daily_analytics` | `ordered_units` | SHOP + SKU + DAY | BUSINESS_DATE | UNITS | NOT_APPLICABLE | SUM across compatible SKU/day rows |
| `ozon.analytics.returns` | `sku_daily_analytics` | `returns` | SHOP + SKU + DAY | BUSINESS_DATE | UNITS | NOT_APPLICABLE | SUM across compatible SKU/day rows |
| `ozon.analytics.cancellations` | `sku_daily_analytics` | `cancellations` | SHOP + SKU + DAY | BUSINESS_DATE | UNITS | NOT_APPLICABLE | SUM across compatible SKU/day rows |
| `ozon.analytics.delivered_units` | `sku_daily_analytics` | `delivered_units` | SHOP + SKU + DAY | BUSINESS_DATE | UNITS | NOT_APPLICABLE | SUM across compatible SKU/day rows |


规范性说明：

- `ozon.analytics.*` 来自 Ozon Analytics 日事实，不能被 O3Pilot Order Fact 覆盖；
- `o3p.sales.buyer_paid_value` 只是订单侧买家支付观察值，不是最终 Finance Revenue；其 Source Value 本身不改变语义；若用于默认经营视图，可在查询层使用 `eligible_business_posting` 选择范围，但不得因此改写 Source Value；
- 当前没有额外证据证明所有来源形态下 `customer_price_amount` 都可以用同一种数量乘法或高层 SUM 规则聚合，因此 A.3 不补造该公式；
- Analytics `revenue` 当前若没有逐行显式 Currency，必须保持 Currency Unknown，不能猜测 Shop Currency。


## 10.2 Product


| metric_code | source | source_field | grain | time_basis | unit | currency_policy | aggregation_rule |
|---|---|---|---|---|---|---|---|
| `ozon.product.content_rating` | `product_content_rating_snapshot` | `total_rating` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | SCORE | NOT_APPLICABLE | Latest snapshot for point-in-time view; never sum scores across snapshots |


规范性说明：

- Ozon Content Rating 的权重、条件和改进建议保持 Source 语义；
- O3Pilot 不重新定义 Ozon Content Rating Score。


## 10.3 Price


| metric_code | source | source_field | grain | time_basis | unit | currency_policy | aggregation_rule |
|---|---|---|---|---|---|---|---|
| `ozon.price.seller_price` | `product_price_snapshot` | `seller_price_amount` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | MONEY | Use seller_price_currency; UNKNOWN if source has no explicit currency | Latest snapshot for point-in-time view |
| `ozon.price.marketing_seller_price` | `product_price_snapshot` | `marketing_seller_price_amount` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | MONEY | Use marketing_seller_price_currency; UNKNOWN if source has no explicit currency | Latest snapshot for point-in-time view |
| `ozon.price.color_index` | `product_price_snapshot` | `price_index_json.color_index` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | CATEGORY | NOT_APPLICABLE | Latest snapshot; no arithmetic aggregation |
| `ozon.price.external_index_value` | `product_price_snapshot` | `price_index_json.external_index_data.price_index_value` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | VALUE | Source semantics and currency preserved | Latest snapshot; no implicit cross-currency comparison |
| `ozon.price.ozon_index_value` | `product_price_snapshot` | `price_index_json.ozon_index_data.price_index_value` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | VALUE | Source semantics and currency preserved | Latest snapshot; no implicit cross-currency comparison |
| `ozon.price.self_marketplaces_index_value` | `product_price_snapshot` | `price_index_json.self_marketplaces_index_data.price_index_value` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | VALUE | Source semantics and currency preserved | Latest snapshot; no implicit cross-currency comparison |


规范性说明：

- Source Price 及 Index 不同字段可能具有不同 Currency 语义；
- 不得在没有同币种 / Reporting Currency 的情况下直接做价格差；
- Source Snapshot 永远不被 Derived Price Metric 反写。


## 10.4 Shop Health


| metric_code | source | source_field | grain | time_basis | unit | currency_policy | aggregation_rule |
|---|---|---|---|---|---|---|---|
| `ozon.rating.current_value` | `shop_rating_snapshot` | `value` | SHOP + RATING + SNAPSHOT | SNAPSHOT_TIME | VALUE | NOT_APPLICABLE | Latest snapshot by rating_code |
| `ozon.rating.history_value` | `shop_rating_snapshot` | `value` | SHOP + RATING + PERIOD | BUSINESS_DATE | VALUE | NOT_APPLICABLE | Use source historical period; do not average unless source semantics permit |
| `ozon.rating.status_danger` | `shop_rating_snapshot` | `status_danger` | SHOP + RATING + SNAPSHOT | SNAPSHOT_TIME | BOOLEAN | NOT_APPLICABLE | Latest snapshot |
| `ozon.rating.status_warning` | `shop_rating_snapshot` | `status_warning` | SHOP + RATING + SNAPSHOT | SNAPSHOT_TIME | BOOLEAN | NOT_APPLICABLE | Latest snapshot |
| `ozon.rating.status_premium` | `shop_rating_snapshot` | `status_premium` | SHOP + RATING + SNAPSHOT | SNAPSHOT_TIME | BOOLEAN | NOT_APPLICABLE | Latest snapshot |
| `ozon.rating.fbs_error_index` | `fbs_error_index_snapshot` | `index_value` | SHOP + SNAPSHOT | SNAPSHOT_TIME | VALUE | NOT_APPLICABLE | Latest snapshot |


规范性说明：

- Ozon Rating Threshold 由 Ozon Source 决定，METRICS 不重新定义；
- Seller Info 中尚未拥有 Canonical Metric Code 的 Rating/Price-Zone 字段仍属于 Source Fact，不在 A.3 为其凭空创建新 Metric Code；
- FBS Error Posting 当前非空样本不足的明细原因指标继续保持未正式定义。



---

# 11. Active Derived Metric Contracts

以下每个正式 Active Derived / Estimate / Derived Reference Metric 都必须完整满足 Derived 9。



## 11.1 Data Quality


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.data.coverage_lag` | SOURCE | SOURCE_BUSINESS_TIME | now - latest_source_business_time | A source/domain with an authoritative latest_source_business_time | DURATION | NOT_APPLICABLE | Recompute at query time from the latest eligible source business time | 1 |
| `o3p.data.mapping_coverage` | DOMAIN + PERIOD | BUSINESS_DATE | matched_records / eligible_records | A domain where mapping is required and eligible population is authoritative; Seller Catalog Mapping applies only when that Later domain is active | RATIO | NOT_APPLICABLE | Recompute from SUM(matched_records) / SUM(eligible_records); never AVG ratios | 1 |


规则：

- `coverage_lag` 回答业务数据覆盖到哪里，不等于 Pipeline Fetch Age；
- `mapping_coverage` 只在该 Domain 的 Mapping 已进入当前 Phase 且 Eligible Population 可定义时适用；
- Seller Catalog Mapping 为 Later，因此 Seller Catalog Mapping Coverage 也不能提前变成 P0 Active 实现要求。


## 11.2 Sales


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.sales.mother_order_count` | SHOP + PERIOD | ORDER_CREATED | COUNT(DISTINCT mother_order_id) FROM eligible_business_posting | Mother order has at least one posting in eligible_business_posting | COUNT | NOT_APPLICABLE | COUNT DISTINCT at requested grain | 1 |
| `o3p.sales.posting_count` | SHOP + PERIOD | POSTING_CREATED | COUNT(DISTINCT posting_id) FROM eligible_business_posting | posting IN eligible_business_posting | COUNT | NOT_APPLICABLE | COUNT DISTINCT at requested grain | 1 |
| `o3p.sales.ordered_units` | SHOP + SKU + PERIOD | POSTING_CREATED | SUM(posting_item.quantity) WHERE posting IN eligible_business_posting | posting IN eligible_business_posting | UNITS | NOT_APPLICABLE | SUM quantity | 1 |
| `o3p.sales.delivered_units` | SHOP + SKU + PERIOD | DELIVERED | SUM(posting_item.quantity) WHERE posting IN eligible_business_posting AND normalized final state = DELIVERED | posting IN eligible_business_posting AND normalized final state = DELIVERED | UNITS | NOT_APPLICABLE | SUM quantity; delivery-state mapping remains versioned | 1 |
| `o3p.sales.post_shipment_cancelled_units` | SHOP + SKU + PERIOD | CANCELLED | SUM(posting_item.quantity) WHERE posting is eligible, reliable shipped_at exists, final state = CANCELLED, and cancellation occurred after shipment | posting IN eligible_business_posting AND verified post-shipment cancellation | UNITS | NOT_APPLICABLE | SUM quantity | 1 |
| `o3p.sales.order_gross_value` | SHOP + SKU + PERIOD | POSTING_CREATED | SUM(posting_item.unit_price_amount * posting_item.quantity) WHERE posting IN eligible_business_posting | posting IN eligible_business_posting AND unit price is available | MONEY | Group by source currency or convert using an explicit Reporting Currency and FX policy | SUM only within a single currency/reporting currency | 1 |
| `o3p.sales.aov_mother_order` | SHOP + PERIOD | ORDER_CREATED | o3p.sales.order_gross_value / o3p.sales.mother_order_count | Same eligible_business_posting population; denominator > 0 | MONEY | Numerator must be a single currency/reporting currency | Recompute from aggregated numerator / aggregated denominator | 1 |
| `o3p.sales.aov_posting` | SHOP + PERIOD | POSTING_CREATED | o3p.sales.order_gross_value / o3p.sales.posting_count | Same eligible_business_posting population; denominator > 0 | MONEY | Numerator must be a single currency/reporting currency | Recompute from aggregated numerator / aggregated denominator | 1 |
| `o3p.sales.average_unit_price` | SHOP + SKU + PERIOD | POSTING_CREATED | o3p.sales.order_gross_value / o3p.sales.ordered_units | Same eligible_business_posting population; ordered_units > 0 | MONEY | Numerator must be a single currency/reporting currency | Recompute from aggregated numerator / aggregated denominator | 1 |


规则：

- Mother Order、Posting、Posting Item Quantity 是不同计数单位；
- `order_gross_value` 是订单侧有效订单金额，不等于 Analytics Revenue，也不等于 Finance Revenue；
- 发货后取消 Unit 只有在可靠 Shipment Node 已存在且取消发生在其后时才能进入分子。


## 11.3 Demand


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.demand.created_mother_order_count` | SHOP + PERIOD | ORDER_CREATED | COUNT(DISTINCT mother_order_id) over all created demand | All created mother orders, including pre-shipment cancellations | COUNT | NOT_APPLICABLE | COUNT DISTINCT at requested grain | 1 |
| `o3p.demand.created_posting_count` | SHOP + PERIOD | POSTING_CREATED | COUNT(DISTINCT posting_id) over all created demand | All created postings, including pre-shipment cancellations | COUNT | NOT_APPLICABLE | COUNT DISTINCT at requested grain | 1 |
| `o3p.demand.created_ordered_units` | SHOP + SKU + PERIOD | POSTING_CREATED | SUM(posting_item.quantity) over all created demand | All created posting items, including pre-shipment cancellations | UNITS | NOT_APPLICABLE | SUM quantity | 1 |
| `o3p.demand.created_order_gross_value` | SHOP + SKU + PERIOD | POSTING_CREATED | SUM(posting_item.unit_price_amount * posting_item.quantity) over all created demand | All created posting items with price, including pre-shipment cancellations | MONEY | Group by source currency or convert using an explicit Reporting Currency and FX policy | SUM only within a single currency/reporting currency | 1 |


规则：

Created Demand 包含发货前取消，用于观察真实下单需求；不得与默认经营 Sales Metric 合并。


## 11.4 Product


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.product.missing_recommended_attribute_count` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | COUNT(DISTINCT improve_attribute_id) | Current Ozon Content Rating improve_attributes are available | COUNT | NOT_APPLICABLE | COUNT DISTINCT within the selected snapshot | 1 |
| `o3p.product.content_rating_change` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | latest_content_rating - previous_content_rating | At least two ordered Product Content Rating snapshots | SCORE_DELTA | NOT_APPLICABLE | Do not sum snapshot deltas; compare the selected snapshot pair | 1 |


规则：

`missing_recommended_attribute_count` 只统计 Ozon 当前 Content Rating `improve_attributes`，不是类目全部理论缺失属性。


## 11.5 Price


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.price.discount_rate` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | (old_price - seller_price) / old_price | old_price > 0 and both prices are comparable in the same currency | RATIO | Prices must be same currency/reporting currency | Recompute from comparable price values; no AVG across products unless explicitly requested | 1 |
| `o3p.price.competitive_gap` | SHOP + PRODUCT + SNAPSHOT | SNAPSHOT_TIME | seller_price_reporting / reference_price_reporting - 1 | An explicit reference price is selected and both prices are converted to the same Reporting Currency | RATIO | Both prices must share Reporting Currency; retain FX policy | Do not aggregate across different reference-price contexts as one value | 1 |


规则：

`competitive_gap` 必须明确当前使用的 Reference Price Context。不同 Reference Source 的结果不得在缺少上下文时混成一个值。


## 11.6 Inventory


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.inventory.sellable_stock` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | SUM(inventory_snapshot.present) | Selected shop/SKU/stock_type/warehouse scope; observed snapshot is valid | UNITS | NOT_APPLICABLE | SUM present within the same snapshot scope; never compute present - reserved | 1 |
| `o3p.inventory.reserved_stock` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | SUM(inventory_snapshot.reserved) | Selected shop/SKU/stock_type/warehouse scope; observed snapshot is valid | UNITS | NOT_APPLICABLE | SUM reserved within the same snapshot scope | 1 |
| `o3p.inventory.in_transit_stock` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | SUM(eligible normalized in-transit quantity) | P1 manual in-transit input path is normalized; eligible state mapping is known for the selected source | UNITS | NOT_APPLICABLE | SUM only eligible current in-transit quantities; never merge into Current Inventory | 1 |
| `o3p.inventory.calendar_ads_units` | SHOP + SKU + PERIOD | POSTING_CREATED | eligible_ordered_units_in_window / calendar_days_in_window | Window is explicit; ordered units follow eligible_business_posting; calendar_days_in_window > 0 | UNITS_PER_DAY | NOT_APPLICABLE | Recompute from aggregated numerator / calendar-day denominator | 1 |
| `o3p.inventory.adjusted_ads_units` | SHOP + SKU + PERIOD | POSTING_CREATED | eligible_ordered_units_on_available_days / available_days | Window is explicit; availability coverage is reliable; available_days > 0 | UNITS_PER_DAY | NOT_APPLICABLE | Recompute from aggregated numerator / available-day denominator | 1 |
| `ozon.reference.product_availability` | SHOP + SKU + PERIOD | SNAPSHOT_TIME | time product was sellable in at least one eligible FBP/rFBS mode / total observation time | Ozon reference applicability and observation coverage are available | RATIO | NOT_APPLICABLE | Recompute reference numerator / denominator; do not equate to O3Pilot snapshot availability without reconciliation | 1 |
| `o3p.inventory.availability_rate` | SHOP + SKU + PERIOD | SNAPSHOT_TIME | available_eligible_days / eligible_observed_days | Inventory coverage is reliable; Sync Gap days are excluded; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from aggregated available days / eligible observed days | 1 |
| `o3p.inventory.out_of_stock_days` | SHOP + SKU + PERIOD | SNAPSHOT_TIME | eligible_observed_days - available_eligible_days | Observed-day coverage is reliable | DAYS | NOT_APPLICABLE | SUM day classifications only over non-overlapping observation days | 1 |
| `o3p.inventory.days_of_cover` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | sellable_stock / adjusted_ads_units | Current sellable_stock available; adjusted_ads_units > 0; otherwise NO_RECENT_DEMAND | DAYS | NOT_APPLICABLE | Recompute from current stock and compatible ADS; never average Days of Cover across SKU unless explicitly requested | 1 |
| `o3p.inventory.supply_days_of_cover` | SHOP + SKU + SNAPSHOT | SNAPSHOT_TIME | (sellable_stock + eligible_in_transit_stock) / adjusted_ads_units | Current stock and eligible in-transit stock available; adjusted_ads_units > 0 | DAYS | NOT_APPLICABLE | Recompute from stock components and compatible ADS | 1 |
| `o3p.inventory.lost_sales_estimate` | SHOP + SKU + PERIOD | BUSINESS_DATE | average_sales_amount_per_available_day * out_of_stock_days | Same-currency sales observations; data-gap days excluded; explicit observation window | MONEY | Use one source/reporting currency; never cross-currency multiply/sum | Recompute from compatible available-day sales basis and out-of-stock days | 1 |


规则：

- `sellable_stock` 使用已验证 `present` 语义；不得计算 `present - reserved` 伪造可售库存；
- Current Stock 与 In-transit Stock 永远分开；
- In-transit P1 当前必须支持 Manual Path，但 A.3 不重新复活 Deferred 的完整 Supply / SupplyItem Schema；
- `adjusted_ads_units` 的不可用日必须依赖可靠库存覆盖，不得把 Missing Snapshot 当成缺货；
- `days_of_cover` 在 ADS 为 0 时返回 `NO_RECENT_DEMAND`，不得返回 999 天等伪造大数。

Ozon Lost Sales 仍保留为 Reference Rule，而不是在缺少既有 Canonical Code 时由 A.3 新造 Metric Code：

```text
daily_average_sales_amount
=
28-day sales amount
/
days product was available

lost_sales
=
daily_average_sales_amount
*
out_of_stock_days
```

当前 Ozon 对仅 rFBS 商品的参考规则还可能包含 +30%。该 Reference 不自动成为 O3Pilot `lost_sales_estimate` 的公式。


## 11.7 Fulfillment


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.fulfillment.shipped_units` | SHOP + SKU + PERIOD | HANDOVER | SUM(posting_item.quantity) WHERE posting IN eligible_business_posting AND reliable shipped_at exists | posting IN eligible_business_posting AND reliable shipped_at exists | UNITS | NOT_APPLICABLE | SUM quantity | 1 |
| `o3p.fulfillment.order_to_handover_duration` | POSTING | ORDER_CREATED → HANDOVER | handover_at - order_created_at | Both timestamps exist and handover_at >= order_created_at | SECONDS | NOT_APPLICABLE | Percentiles/mean must be recomputed from raw posting-level durations | 1 |
| `o3p.fulfillment.processing_to_handover_duration` | POSTING | PROCESSING_STARTED → HANDOVER | handover_at - in_process_at | Both timestamps exist and handover_at >= in_process_at | SECONDS | NOT_APPLICABLE | Percentiles/mean must be recomputed from raw posting-level durations | 1 |
| `o3p.fulfillment.delivery_transit_duration` | POSTING | HANDOVER → DELIVERED | fact_delivery_date - handover_at | Both timestamps exist and fact_delivery_date >= handover_at | SECONDS | NOT_APPLICABLE | Percentiles/mean must be recomputed from raw posting-level durations | 1 |
| `o3p.fulfillment.order_to_delivery_duration` | POSTING | ORDER_CREATED → DELIVERED | fact_delivery_date - order_created_at | Both timestamps exist and fact_delivery_date >= order_created_at | SECONDS | NOT_APPLICABLE | Percentiles/mean must be recomputed from raw posting-level durations | 1 |
| `o3p.fulfillment.on_time_handover_rate` | SHOP + FULFILLMENT_MODE + PERIOD | HANDOVER | COUNT(handover_at <= shipment_deadline_at) / COUNT(eligible_postings) | Posting not pre-shipment-cancelled; shipment_deadline_at and handover_at exist; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed on-time count / eligible count | 1 |
| `o3p.fulfillment.promise_delivery_hit_rate` | SHOP + FULFILLMENT_MODE + PERIOD | DELIVERED | COUNT(fact_delivery_date <= promised_delivery_to) / COUNT(eligible_delivered_postings) | promised_delivery_to and fact_delivery_date exist; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed hit count / eligible delivered count | 1 |
| `o3p.fulfillment.deadline_shift_hours` | POSTING | SNAPSHOT_TIME | (latest_shipment_deadline - initial_shipment_deadline) in hours | At least initial and latest posting_schedule_history values exist | HOURS | NOT_APPLICABLE | Compare selected initial/latest schedule values; do not sum repeated snapshots | 1 |
| `o3p.quality.handover_time_completeness` | SHOP + PERIOD | POSTING_CREATED | postings_with_handover_time / eligible_postings | An authoritative eligible posting population exists; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.quality.delivery_time_completeness` | SHOP + PERIOD | DELIVERED | delivered_postings_with_fact_delivery / delivered_postings | An authoritative delivered posting population exists; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `ozon.reference.shipment_delay_rate_14d` | SHOP + PERIOD | HANDOVER | shipments_delayed_due_to_seller / shipments_that_should_have_been_handed_over | Official service-quality/report reference population is available; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from official-reference numerator / denominator | 1 |


规则：

所有 Duration 内部使用 seconds。

如果：

```text
end_time < start_time
```

结果：

```text
UNVERIFIED
```

并进入 Data Quality investigation，不静默取绝对值。

P50 / P90 / Average 从 Posting-level Duration 原始样本重新计算，并在适用时展示 `sample_count`。

旧 `o3p.fulfillment.promise_window_shift_hours` 不再作为一个多值正式 Metric。承诺窗口变更必须分别比较：

```text
promised_delivery_from shift
promised_delivery_to shift
```

该规则保留在 Schedule Comparison，不强迫一个 Metric Result 同时装两个 value。


## 11.8 Cancellation


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `ozon.reference.cancellation_rate_7d_count` | SHOP + PERIOD | POSTING_CREATED | seller_cancelled_shipments_in_7d / created_shipments_in_7d | Official Fulfillment Report reference population/flags are available; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from official-reference numerator / denominator; do not merge with 14-day service-quality metric | 1 |
| `ozon.reference.cancellation_rate_7d_value` | SHOP + PERIOD | POSTING_CREATED | value_of_seller_cancelled_shipments / value_of_created_shipments | Official Fulfillment Report reference population/flags are available; denominator > 0 | RATIO | Numerator and denominator values must be comparable in the same currency | Recompute from official-reference value numerator / denominator | 1 |
| `o3p.cancellation.pre_shipment_rate` | SHOP + POSTING_CREATED_COHORT | POSTING_CREATED | pre_shipment_cancelled_postings / created_postings | Created posting population authoritative; cancellation stage verified; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.cancellation.posting_rate` | SHOP + ELIGIBLE_ORDER_COHORT | POSTING_CREATED | post_shipment_cancelled_postings / eligible_business_postings | eligible_business_postings = created - confirmed pre-shipment cancellations; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.cancellation.buyer_rate` | SHOP + ELIGIBLE_ORDER_COHORT | POSTING_CREATED | buyer_responsible_post_shipment_cancelled_postings / eligible_business_postings | Responsibility determined by normalized cancellation mapping; UNKNOWN is not assigned to buyer; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.cancellation.seller_responsible_rate` | SHOP + ELIGIBLE_ORDER_COHORT | POSTING_CREATED | seller_responsible_post_shipment_cancelled_postings / eligible_business_postings | Responsibility determined by normalized cancellation mapping; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.cancellation.unit_rate` | SHOP + SKU + ELIGIBLE_ORDER_COHORT | POSTING_CREATED | post_shipment_cancelled_units / eligible_ordered_units | Eligible ordered units exclude confirmed pre-shipment cancellations; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed numerator / denominator | 1 |
| `o3p.cancellation.stage_coverage` | SHOP + PERIOD | CANCELLED | cancelled_postings_with_verified_pre_or_post_shipment_stage / all_cancelled_postings_requiring_stage_classification | Cancelled postings require lifecycle-stage classification; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed classified / required-to-classify counts | 1 |


规则：

- Ozon 7-day Fulfillment Report 与 Ozon 14-day Service Quality 不是同一个口径；
- O3Pilot Cancellation 指标不宣称等于 Ozon 官方 Rating；
- 无法确认 Pre/Post-shipment Stage 时不得猜测，相关结果至少为 `PARTIAL`；
- 责任无法确认时保持 Unknown Classification，不自动归给买家或卖家。


## 11.9 Returns


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `o3p.return.request_rate` | SHOP + PRODUCT + DELIVERY_COHORT | DELIVERY_COHORT | return_requested_units / delivered_units_in_eligible_cohort | Return-request variant (all or accepted) is explicit; delivered cohort authoritative; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed request units / delivered units for the same variant | 1 |
| `o3p.product.return_rate` | SHOP + PRODUCT + DELIVERY_COHORT | DELIVERY_COHORT | completed_customer_return_units_as_of / delivered_units_in_cohort | Delivered cohort is authoritative; returns are reliably linked to original sold units; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from deduplicated returned units / delivered cohort units | 1 |
| `o3p.return.reason_share` | SHOP + REASON_GROUP + PERIOD | RETURN_COMPLETED | returned_units_for_reason / all_returned_units | Standardized Reason Group is available; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from summed reason units / all returned units | 1 |
| `o3p.whd.resale_rate` | SHOP + PRODUCT + PERIOD | RETURN_COMPLETED | units_resold_from_whd / units_entered_whd | reverse_logistics_link reliably connects Return → WHD → Resale; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from linked WHD units; never match by SKU alone | 1 |
| `o3p.whd.recovery_revenue` | SHOP + PRODUCT + PERIOD | BUSINESS_DATE | SUM(recognized_revenue_from_linked_whd_resales) | reverse_logistics_link reliably connects Return → WHD → Resale | MONEY | Group by currency or convert with explicit Reporting Currency/FX policy | SUM only linked recognized revenue in a single currency/reporting currency | 1 |


产品退货率为核心 P1 Metric。

同一售出 Unit 在 Product Return Rate 分子最多计一次：

```text
completed_returned_units
<=
eligible_delivered_units
```

默认不计入 Product Return Rate 分子：

- 发货前取消；
- 未提件；
- 仅创建但被拒绝的 Return Request；
- 无法可靠关联到原销售的逆向记录；
- WHD 二次销售本身。

Recent Delivery Cohort 可以保持 `OPEN_COHORT`。

Return Request Rate 的 `request_all` / `request_accepted` 语义必须作为明确 Result Variant Context 暴露，不能静默切换。


## 11.10 Buyout


| metric_code | grain | time_basis | formula | eligibility | unit | currency_policy | aggregation_rule | metric_version |
|---|---|---|---|---|---|---|---|---|
| `ozon.reference.buyout_rate_last_50_orders` | SHOP + PRODUCT + ORDER_COHORT | BUYOUT_ELIGIBLE_COHORT | (order_count - return_count - cancellation_count) / order_count | Ozon reference cohort contains up to the latest 50 eligible orders; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from official-reference components; do not invent category/price-band thresholds | 1 |
| `o3p.product.buyout_rate` | SHOP + PRODUCT + BUYOUT_ELIGIBLE_COHORT | BUYOUT_ELIGIBLE_COHORT | buyout_success_units_as_of / buyout_eligible_units | Pre-shipment cancellations excluded; unit has reached buyout eligibility; denominator > 0 | RATIO | NOT_APPLICABLE | Recompute from deduplicated success units / buyout eligible units | 1 |
| `o3p.product.non_buyout_rate` | SHOP + PRODUCT + BUYOUT_ELIGIBLE_COHORT | BUYOUT_ELIGIBLE_COHORT | 1 - o3p.product.buyout_rate | o3p.product.buyout_rate is available for the same cohort | RATIO | NOT_APPLICABLE | Use same denominator as buyout rate; components must not double-count | 1 |


规则：

- Ozon 最近最多 50 单 Reference 与 O3Pilot Buyout Rate 是两个独立 Metric；
- Ozon 类目 / 价格区间的低中高阈值当前不属于稳定 Metric Contract，不自行猜测；
- O3Pilot Buyout 先排除发货前取消，但只有达到 Buyout Eligibility 的 Unit 才进入分母；
- 无法可靠区分未认购、发货后取消和客户退货时，Result 为 `PARTIAL`。



---

# 12. Optional Extension Bindings

Optional Extension 只在需要时出现，不形成所有 Metric 的第二套大模板。

## 12.1 Numerator / Denominator


| metric_code | numerator | denominator |
|---|---|---|
| `o3p.data.mapping_coverage` | matched_records | eligible_records |
| `o3p.sales.aov_mother_order` | o3p.sales.order_gross_value | o3p.sales.mother_order_count |
| `o3p.sales.aov_posting` | o3p.sales.order_gross_value | o3p.sales.posting_count |
| `o3p.sales.average_unit_price` | o3p.sales.order_gross_value | o3p.sales.ordered_units |
| `o3p.price.discount_rate` | old_price - seller_price | old_price |
| `o3p.price.competitive_gap` | seller_price_reporting | reference_price_reporting |
| `o3p.inventory.calendar_ads_units` | eligible_ordered_units_in_window | calendar_days_in_window |
| `o3p.inventory.adjusted_ads_units` | eligible_ordered_units_on_available_days | available_days |
| `ozon.reference.product_availability` | sellable_observation_time | total_observation_time |
| `o3p.inventory.availability_rate` | available_eligible_days | eligible_observed_days |
| `o3p.inventory.days_of_cover` | sellable_stock | adjusted_ads_units |
| `o3p.inventory.supply_days_of_cover` | sellable_stock + eligible_in_transit_stock | adjusted_ads_units |
| `o3p.fulfillment.on_time_handover_rate` | on_time_handover_postings | eligible_postings |
| `o3p.fulfillment.promise_delivery_hit_rate` | promise_hit_postings | eligible_delivered_postings |
| `o3p.quality.handover_time_completeness` | postings_with_handover_time | eligible_postings |
| `o3p.quality.delivery_time_completeness` | delivered_postings_with_fact_delivery | delivered_postings |
| `ozon.reference.cancellation_rate_7d_count` | seller_cancelled_shipments_in_7d | created_shipments_in_7d |
| `ozon.reference.cancellation_rate_7d_value` | value_of_seller_cancelled_shipments | value_of_created_shipments |
| `o3p.cancellation.pre_shipment_rate` | pre_shipment_cancelled_postings | created_postings |
| `o3p.cancellation.posting_rate` | post_shipment_cancelled_postings | eligible_business_postings |
| `o3p.cancellation.buyer_rate` | buyer_responsible_post_shipment_cancelled_postings | eligible_business_postings |
| `o3p.cancellation.seller_responsible_rate` | seller_responsible_post_shipment_cancelled_postings | eligible_business_postings |
| `o3p.cancellation.unit_rate` | post_shipment_cancelled_units | eligible_ordered_units |
| `o3p.cancellation.stage_coverage` | cancelled_postings_with_verified_stage | all_cancelled_postings_requiring_stage_classification |
| `ozon.reference.shipment_delay_rate_14d` | shipments_delayed_due_to_seller | shipments_that_should_have_been_handed_over |
| `o3p.return.request_rate` | return_requested_units | delivered_units_in_eligible_cohort |
| `o3p.product.return_rate` | completed_customer_return_units_as_of | delivered_units_in_cohort |
| `o3p.return.reason_share` | returned_units_for_reason | all_returned_units |
| `ozon.reference.buyout_rate_last_50_orders` | order_count - return_count - cancellation_count | order_count |
| `o3p.product.buyout_rate` | buyout_success_units_as_of | buyout_eligible_units |
| `o3p.product.non_buyout_rate` | eligible_non_buyout_units_as_of | buyout_eligible_units |
| `o3p.whd.resale_rate` | units_resold_from_whd | units_entered_whd |


## 12.2 Window


| metric_code | window |
|---|---|
| `o3p.inventory.calendar_ads_units` | Explicit query window; current supported examples include 7 / 14 / 28 / 56 days; no global default is frozen |
| `o3p.inventory.adjusted_ads_units` | Explicit query window; current supported examples include 7 / 14 / 28 / 56 days; no global default is frozen |
| `ozon.reference.product_availability` | Current Ozon reference primarily describes a 28-day observation; re-review when official rule changes |
| `o3p.inventory.availability_rate` | Explicit observation period |
| `o3p.inventory.lost_sales_estimate` | Explicit observation window; data-gap days excluded |
| `ozon.reference.cancellation_rate_7d_count` | 7 days |
| `ozon.reference.cancellation_rate_7d_value` | 7 days |
| `ozon.reference.shipment_delay_rate_14d` | 14 days; current official reference excludes the current day |
| `ozon.reference.buyout_rate_last_50_orders` | Latest up to 50 eligible orders; order-count window rather than fixed calendar days |


## 12.3 Exclusions


| metric_code | exclusions |
|---|---|
| `o3p.inventory.adjusted_ads_units` | Days without reliable availability coverage are not silently treated as out-of-stock/available |
| `o3p.inventory.lost_sales_estimate` | Exclude source coverage gaps and incompatible currencies |
| `o3p.product.return_rate` | Exclude pre-shipment cancellation, unclaimed items, rejected-only requests, unmatched reverse records, WHD resale itself; deduplicate each sold unit |
| `o3p.product.buyout_rate` | Exclude pre-shipment cancellations; do not double-count post-shipment non-buyout and completed returns |
| `o3p.product.non_buyout_rate` | Pre-shipment cancellation is not Non-buyout |
| `o3p.whd.resale_rate` | Do not infer Return → WHD → Resale link from SKU equality alone |
| `o3p.whd.recovery_revenue` | Do not infer Return → WHD → Resale link from SKU equality alone |


## 12.4 Coverage


| metric_code | coverage |
|---|---|
| `o3p.inventory.adjusted_ads_units` | Inventory-day availability coverage must be reliable for the selected window |
| `o3p.inventory.availability_rate` | Missing inventory observations are excluded rather than classified as out-of-stock |
| `o3p.inventory.lost_sales_estimate` | Source sales and availability coverage must support the full selected window |
| `o3p.cancellation.stage_coverage` | Unknown cancellation stage remains unknown; affected downstream cancellation metrics become PARTIAL |
| `o3p.product.return_rate` | Return-to-original-sale linkage coverage determines VALID/PARTIAL semantics |
| `o3p.product.buyout_rate` | Lifecycle classification coverage determines VALID/PARTIAL semantics |
| `o3p.whd.resale_rate` | Requires reliable reverse_logistics_link; otherwise result is UNVERIFIED |
| `o3p.whd.recovery_revenue` | Requires reliable reverse_logistics_link; otherwise result is UNVERIFIED |


---

# 13. Data Quality、Observability 与 Reconciliation 边界

## 13.1 保留的用户面向质量 Metric

当前 Core Registry 只保留：

```text
o3p.data.coverage_lag
o3p.data.mapping_coverage
```

它们直接影响用户解释业务数据是否覆盖到正确时间或对象。

---

## 13.2 Completeness 是 Invariant，不是通用 Metric

旧：

```text
o3p.data.completeness
```

不再作为一个全系统通用 Business Metric。

正式规则：

> 只有存在权威 Expected Population 时，才允许计算 Completeness Ratio。

没有权威 Expected 时：

```text
不得伪造 completeness %
```

具体 Domain 若未来存在真实 Expected Contract，可建立该 Domain 自己的正式 Metric。

---

## 13.3 Pipeline Telemetry 不进入 Business Metric Registry

以下旧 Metric Code 不再属于 Business Metric Contract：

```text
o3p.data.fetch_age
o3p.data.duplicate_rate
o3p.data.gap_duration
```

职责 Authority：

```text
ARCHITECTURE.md §27 Observability
```

时间语义：

```text
ARCHITECTURE.md §7.4 Time Semantics
```

底层事实来自当前：

- `raw_capture`；
- `sync_run`；
- `data_quality_issue`。

如果 Observability UI 需要 Freshness、Duplicate Rate 或 Gap Duration，可按这些事实计算诊断值；不得反过来宣称 `data_quality_issue` 已经一对一持久保存这些聚合值。

---

## 13.4 Reconciliation Match 是 Diagnostic

旧：

```text
o3p.data.reconciliation_match_rate
```

不再作为 Core Business Metric。

对账仍允许计算：

```text
matched
/
comparable
```

但属于 Reconciliation Diagnostic。

能够双算的 Source / Derived 可以展示：

```text
Source Value
Derived Value
Difference
Difference %
```

Derived 永远不能覆盖 Source。

建议 Difference Status：

```text
MATCH
WITHIN_TOLERANCE
DIFFERENT
NOT_COMPARABLE
PARTIAL
```

Money Tolerance 必须由对应 Reconciliation Context 定义，不建立一个全系统固定误差。

---

# 14. Source / Reference Conflict Rules

Ozon 不同页面、报告或 API 的相似指标不得自动合并。

例如：

```text
Fulfillment Report Cancellation
→ 7-day reference

Service Quality Cancellation
→ 14-day reference
```

它们拥有不同人口、时间窗和责任规则，因此保持独立。

`OZON_REFERENCE` 与 O3Pilot 自算 Metric 只有完成逐项对账后，才能说明两者是否接近；不得先宣称 100% 相同。

---

# 15. Metric Version 与 Code Stability

以下任一计算语义改变时必须提升 `metric_version`：

- Formula；
- Numerator；
- Denominator；
- Eligibility；
- Exclusions；
- Time Basis；
- Window；
- Mapping used by the Metric；
- FX Policy；
- Cost / fallback semantics。

UI 名称变化不自动提升 `metric_version`。

`metric_code` 一旦作为数据库或 API Contract 发布，不得因为 UI 文案修改而改变。

Derived Result 不得覆盖 Source Fact。

---

# 16. Deferred Metric Register

Deferred 不等于删除 Source Acquisition，也不等于把未来完整算法继续留在 Active Baseline。

只有：

```text
DEFER_P2
DEFER_P3
DEFER_P4
DEFER_LATER
```

可以进入本 Register。


| metric_code_or_group | target_phase | reason | re_entry_condition | archive_ref |
|---|---|---|---|---|
| Traffic Analytics source metrics (`ozon.analytics.hits_view*`, `hits_tocart*`, `session_view*`) | P3 | Growth Analytics / traffic analysis is outside v1 P0+P1 although acquisition may start earlier | P3 Growth Analytics enters delivery and DATA_MODEL/DATA_SOURCES contracts for the required traffic fields remain valid | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| `o3p.funnel.view_to_cart_rate` | P3 | Conversion funnel is Growth Analytics | P3 Growth Analytics enters delivery with stable source coverage | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Search Analytics (`ozon.search.*`, `o3p.search.query_*`) | P3 | Search is P3 Metrics/Feature even when Acquisition is P0 | P3 Growth Analytics enters delivery; retained-window evidence remains sufficient for implementation | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Promotion Lift (former §18) | P3 | Promotion performance comparison belongs Growth Analytics | P3 Growth Analytics enters delivery and promotion participation/source coverage is authoritative | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Finance metrics (`o3p.finance.*`, `ozon.report.*`) | P2 | Finance & Settlement are outside v1 Feature Delivery | P2 Finance & Profit enters delivery with deferred normalized Finance model activated | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Profit / Cost metrics (`o3p.profit.*`, `o3p.cost.seller_logistics`, `o3p.price.minimum_profitable_price`) | P2 | Profit and cost attribution depend on P2 models and seller-owned cost inputs | P2 Finance & Profit enters delivery with cost/finance coverage rules activated | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Advertising source and derived metrics (`ozon.ads.*`, `o3p.ads.*`) | P3 | Advertising analytics is Growth Analytics | P3 enters delivery; advertising normalized models and historical-window evidence are active | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Questions / Answers metrics (`o3p.voc.*`) | P3 | VOC is P3 and Questions Acquisition/Backfillability remain TBD | P3 enters delivery and acquisition/backfillability has been verified without guessing | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Reviews metric family | P3 | Reviews are access-dependent P3; current List/Detail contract is not fully verified | P3 enters delivery and Review capability/list/detail evidence is verified | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Cross-shop catalog metrics (`o3p.catalog.total_ordered_units`, `o3p.catalog.total_revenue`, cross-shop return-rate group) | Later | Cross-shop canonical product aggregation depends on Seller Catalog Mapping | Seller Catalog Mapping re-enters with stable identity/history and PRODUCT authorizes the feature | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Ozon replenishment guidance reference (former §27) | P4 | Replenishment recommendation thresholds are Decision Support policy/reference | P4 Decision Support enters delivery and current Ozon reference is re-reviewed | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| `o3p.inventory.recommended_replenishment_units` + baseline replenishment estimate group | P4 | Replenishment recommendation/forecast is Decision Support | P4 enters delivery with Forecast/Recommendation model and lead-time inputs activated | METRICS v1.0 Drive revision history (NON-NORMATIVE) |
| Forecast metric family (`o3p.forecast.*`) | P4 | Forecast/backtest/projected stock metrics are Decision Support | P4 enters delivery; FORECASTING authority/model is defined and historical backtest inputs are available | METRICS v1.0 Drive revision history (NON-NORMATIVE) |


完整旧算法不复制到 Deferred Appendix。历史版本通过 Drive / Git Revision History 找回，Archive Ref 永远是 NON-NORMATIVE。

特别边界：

```text
Search Analytics
Acquisition = P0
Metrics / Feature = P3
```

因此 Search 仍在 Deferred Metric Register，但对应有限窗口数据可以提前采集。

Questions / Answers：

```text
Acquisition Phase = TBD
Backfillability = TBD
Metrics / Feature = P3
```

METRICS 不重新裁决 Questions / Answers 的采集机制或回补能力；这些仍属于 `DATA_SOURCES.md`。

Campaign SKU Daily：

```text
Feature = P3
```

它的提前采集不允许把广告 Metric 提前变成 P0/P1 Active。

---

# 17. Delegated、Rule-only 与 Diagnostic Disposition

以下内容不能进入 Deferred Metric Register：


| source_item | disposition | target_phase | target_authority | reason |
|---|---|---|---|---|
| Recommendation Priority (former §115) | DELEGATE | — | ALERTS_AND_RECOMMENDATIONS.md | Application / Decision Support policy, not a Metric Definition |
| Alert Metric Rule / thresholds (former §116) | DELEGATE | P1 Basic Alerts feature can consume metrics | ALERTS_AND_RECOMMENDATIONS.md | Alert threshold/window/rule lifecycle is application policy |
| Anomaly classification / threshold / seasonality / model (former §117 tail) | DELEGATE | — | ALERTS_AND_RECOMMENDATIONS.md | Basic delta math remains in METRICS; anomaly decision policy does not |
| `o3p.data.fetch_age` | DIAGNOSTIC_ONLY | P0 | ARCHITECTURE.md §27 Observability | Pipeline freshness diagnostic; not a business Metric Contract |
| `o3p.data.duplicate_rate` | DIAGNOSTIC_ONLY | P0 | ARCHITECTURE.md §27 Observability + DATA_MODEL.md §4.4 | Pipeline data-quality diagnostic computed from underlying facts when needed |
| `o3p.data.gap_duration` | DIAGNOSTIC_ONLY | P0 | ARCHITECTURE.md §27 Observability + DATA_MODEL.md §4.3/§4.4 | Sync-gap diagnostic; not a core business metric |
| `o3p.data.reconciliation_match_rate` | DIAGNOSTIC_ONLY | P0 | METRICS Reconciliation Rules | Reconciliation diagnostic semantic; not Core Registry |
| `o3p.data.completeness` | RULE_ONLY | P0 | METRICS Data Quality Invariants | No global percentage unless an authoritative expected population exists |
| `o3p.sales.growth_rate` / `o3p.sales.unit_growth_rate` | RULE_ONLY | P1 | METRICS Period & Baseline Comparison | Generic comparison must not create one Metric Code with multiple underlying source meanings |
| `o3p.fulfillment.promise_window_shift_hours` | RULE_ONLY | P1 | METRICS Schedule Comparison Rules | One old code attempted to carry from/to shifts; keep rule to compare both bounds separately rather than force a multi-value Metric Result |


Basic Alerts 仍然属于 P1 Product Scope；这里只把 Alert Policy / Threshold Authority 从 METRICS 中剥离，不删除 Basic Alerts Feature。

---

# 18. Evidence Boundaries — 当前不建立正式 Metric Contract

当前证据不足时，不为“看起来完整”而创建 Metric Code。

至少包括：

- Performance Phrase ROI / Phrase Conversion：P3 且非空历史主来源仍需验证；
- Review Detail 指标：P3，真实 List / Detail 受权限和验证状态限制；
- FBS Error Posting 原因分布：当前非空真实样本不足；
- Analytics Stocks Item 指标：当前没有已冻结的 Active Metric Contract；
- `inventory_turnover_snapshot` 的 `average_daily_sales / current_stock / inventory_days_cover / turnover_grade` 已是 P1 Source Fact，但 v1.0 没有对应 Canonical Metric Code；A.3 不凭空发明 Code，现阶段继续作为 P1 Source Fact 使用；
- `fbs_error_defect_daily.defect_value` 已进入 P1 Data Model，但 v1.0 没有独立 Canonical Metric Code；A.3 只保留已存在的 `ozon.rating.fbs_error_index` 正式 Metric，不凭空扩展 Code；
- Webhook End-to-End Latency SLA：属于 Observability / Service-Level 设计，不是当前业务 Metric；
- 精确 WHD 财务回收率：没有可靠 Return → WHD → Resale + Finance 关联时不建立；
- Compensation 非零完整 Finance Mapping：P2 再 Review；
- 未经验证的竞争对手销量 / 库存：不定义。

这类 Evidence Boundary 不得通过填充假字段、假公式或假 Source 来消除 TBD。

---

# 19. Formal Metric Definition Reverse Scan

A.3 以后，审计不能只搜索：

```text
metric_code:
```

必须同时覆盖：

1. Active / Deferred Registry；
2. 正文全部显式 `metric_code`；
3. 公式；
4. Numerator / Denominator；
5. Origin 声明；
6. Reference Metric；
7. Estimate / Forecast；
8. Ratio / Rate；
9. Metric-specific Calculation Rule；
10. Metric-like Threshold。

每一个被识别的 Formal Metric Definition / Metric-like 定义必须且只能得到一个 Disposition：

```text
ACTIVE_P0
ACTIVE_P1
DEFER_P2
DEFER_P3
DEFER_P4
DEFER_LATER
DELEGATE
RULE_ONLY
DIAGNOSTIC_ONLY
REMOVE_DUPLICATE
```

并拥有唯一最终 `target_authority` / final location。

只有 `DEFER_*` 进入 Deferred Metric Register。

数学 / Eligibility 边界留在 METRICS；业务 Alert / Recommendation 触发阈值进入对应 Application Authority。

---

# 20. Acceptance Gates

## 20.1 Layer A — Contract Gates

必须全部满足：

```text
Old 17-field migration = 17 / 17 mapped

Source Contract required fields = 8

Derived Contract required fields = 9

Active Optional Extensions:
Batch 0 Frozen = numerator / denominator / exclusions / coverage
A.3 Justified = window

Unjustified new Contract fields = 0

Canonical vocabulary violations = 0

Active formal Metric without one complete SOURCE or DERIVED definition = 0

Multi-origin Registry cells = 0

Generic calculation_run_id requirement = 0
```

---

## 20.2 Layer B — Scope Gates

必须全部满足：

```text
P2/P3/P4/Later formal Metric definitions remaining in Active Body = 0

Formal Metric Definition Reverse Scan unresolved entries = 0
```

Markdown 行数不作为 Gate。

执行完成后只报告真实：

```text
before_lines
after_lines
```

作为审计统计。

---

## 20.3 Cross-layer Gates

必须确认：

```text
Feature Phase is not redefined by METRICS

Endpoint / Pagination / Backfillability is not redefined by METRICS

DATA_MODEL Schema is not invented by METRICS

Ozon remains read-only
```

命名检查：

- METRICS 不再使用旧的 Performance 域 SKU-Daily 歧义名称；
- 当前正式名称统一为 `Campaign SKU Daily`；
- `DATA_SOURCES.md` 中历史采集语境的旧命名不属于 A.3 修改范围。

---

# 21. 关系到其他文档

## 21.1 `DATA_SOURCES.md`

定义：

- Endpoint；
- Pagination；
- Verification；
- Window；
- Backfillability Evidence。

METRICS 不能为了公式完整而发明 Source。

---

## 21.2 `DATA_MODEL.md`

定义：

- Fact；
- Identity；
- Field；
- Source Lineage；
- Phase execution mapping。

METRICS 只定义 Fact 如何计算成 Metric。

---

## 21.3 `PRODUCT.md`

定义 Capability 与 Feature Phase。

METRICS 只为当前 Active Capability 提供量化 Contract；未来 Capability 可以存在于 PRODUCT，但其 Metric Definition 在对应 Phase 前保持 Deferred。

---

## 21.4 `ARCHITECTURE.md`

定义 Runtime、Time Semantics、Observability 和 Reprocessing。

Pipeline Freshness / Duplicate / Sync Gap 诊断归 `ARCHITECTURE.md §27 Observability`，不重新包装成业务 Metric Catalog。

---

# 22. Core Invariants

**一个 Metric Code 只有一个正式口径。**

**Source、Ozon Reference、Derived、Estimate 必须可区分。**

**`metric_origin != contract_shape`。**

**Ratio 聚合重新汇总分子 / 分母，不平均 Ratio。**

**Denominator 0 返回 `NULL + NO_DENOMINATOR`。**

**Money 不跨币种直接相加。**

**原始 Money 不被 Reporting Currency 覆盖。**

**Missing / Unknown 不等于 0。**

**Mother Order、Posting、Unit 是不同计数单位。**

**默认经营订单只排除已确认发货前取消。**

**待发货未取消仍属于有效订单。**

**Created Demand 与 Eligible Sales 分开。**

**Analytics Source 与 Order Fact 不互相覆盖。**

**Sellable Stock、Reserved、In-transit 分开。**

**缺失库存 Snapshot 不等于缺货。**

**Product Return Rate 使用可追溯 Delivery Cohort。**

**Ozon Buyout Reference 与 O3Pilot Buyout 分开。**

**Ozon 取消 / 时效 Reference 与 O3Pilot Operational Metric 分开。**

**Persisted DERIVED Result 必须满足 Historical Recalculation Invariant。**

**Alert / Recommendation 不重新定义 Metric。**

**Future Feature 的提前 Acquisition 不等于提前实现 Future Metric。**

**O3Pilot 永远只读 Ozon。**
