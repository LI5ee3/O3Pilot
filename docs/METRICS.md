# O3Pilot — METRICS.md

> Version: 1.0  
> Status: Metric Contract Baseline  
> Updated: 2026-09-03  
> Applies to: O3Pilot

# 1. 文档目的

`METRICS.md` 定义 O3Pilot 的统一指标口径、计算规则、时间归属、聚合方式、状态语义和指标版本。

本文件负责回答：

- 一个指标到底在计算什么；
- 分子和分母是什么；
- 使用哪个业务时间；
- 统计粒度是什么；
- Money 使用什么币种；
- 是否来自 Ozon 原始指标；
- 是否由 O3Pilot 自己计算；
- 是否属于估算；
- 缺失值、0、未知和无权限如何区分；
- 多店铺、多 SKU、多币种如何聚合；
- 历史指标如何重算；
- Profit、Return、Inventory、Advertising、Forecast 等指标如何避免重复计费或口径混乱。

本文件不定义：

- 数据从哪里获取；
- 数据库 Schema；
- API Endpoint；
- 同步周期；
- Scheduler / Worker；
- 页面布局；
- 具体告警阈值；
- 预测模型实现代码；
- 自动执行任何 Ozon 写操作。

这些内容分别由 `DATA_SOURCES.md`、`DATA_MODEL.md`、`ARCHITECTURE.md`、`DESIGN.md` 等文档定义。

---

# 2. 指标设计总原则

## 2.1 一个名字只能对应一个口径

禁止出现：

```text
销售额
```

在不同页面分别代表：

- Ozon Analytics Revenue；
- 订单标价；
- Buyer Paid；
- Finance Gross Sales；
- Finance Net Accrual；
- Profit Revenue。

必须使用明确指标名称和 Metric Code。

---

## 2.2 Ozon 官方指标与 O3Pilot 自算指标分离

Ozon Seller Center、Seller API、Performance API 返回的官方指标，不得因为 O3Pilot 可以自行计算近似值，就被覆盖成同一个指标。

例如：

```text
Ozon Official Buyout Rate
!=
O3Pilot Buyout Rate
```

```text
Ozon Availability
!=
O3Pilot Availability
```

```text
Ozon Shipment Delay Rating
!=
O3Pilot On-time Handover Rate
```

---

## 2.3 Ratio 不平均 Ratio

跨 SKU、跨店铺、跨日期聚合比例时，必须重新聚合分子和分母。

禁止：

```text
AVG(SKU Return Rate)
```

代替：

```text
SUM(Returned Units)
/
SUM(Eligible Delivered Units)
```

除非产品明确展示的是“SKU 指标的简单平均值”。

---

## 2.4 Money 不跨币种直接相加

允许：

```text
100 USD + 50 USD
```

禁止：

```text
100 USD + 500 CNY
```

跨币种聚合必须：

1. 保留原始 Money；
2. 使用已定义 Exchange Rate；
3. 记录 Rate Basis；
4. 转换到明确的 Reporting Currency；
5. 保存计算版本。

---

## 2.5 缺失不等于 0

统一状态：

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

例如：

Review API 无权限：

```text
review_count = UNAVAILABLE
```

不能：

```text
review_count = 0
```

---

## 2.6 Denominator 为 0 时不返回 0%

统一函数：

```text
safe_div(numerator, denominator)
```

规则：

```text
denominator > 0
→ numerator / denominator

denominator = 0
→ value = NULL
→ status = NO_DENOMINATOR
```

---

## 2.7 历史指标必须可重算

所有 O3Pilot 派生指标必须保存：

```text
metric_code
metric_version
calculation_run_id
calculated_at
source_coverage
```

影响指标的以下规则必须版本化：

- Fulfillment Mapping；
- Product Mapping；
- Return Status Mapping；
- Finance Type Classification；
- Finance Allocation；
- FX Policy；
- Cost Fallback；
- Forecast Model；
- Metric Formula。

---

# 3. 指标来源分类

所有指标必须属于以下一种 `metric_origin`。

## 3.1 `OZON_SOURCE`

Ozon API 直接返回的指标。

例如：

```text
analytics.ordered_units
analytics.revenue
content_rating
search.position
search.view_conversion
rating.current_value
performance.ctr_raw
```

规则：

- 原值保存；
- 不改变语义；
- Ozon 改规则后允许历史值变化。

---

## 3.2 `OZON_REFERENCE`

Ozon 官方知识库明确公布的公式或 Seller Center 指标，但当前不一定有稳定 API 直接返回。

例如：

- 最近 50 个订单认购水平；
- FBP 商品可用性；
- FBP 错失销售；
- Seller Center 取消率报告；
- Seller Center 逾期交货率。

O3Pilot 可以实现参考计算，但必须标记：

```text
metric_origin = OZON_REFERENCE
```

且不能宣称一定与 Seller Center 100% 相同，除非完成逐项对账。

---

## 3.3 `O3P_DERIVED`

完全由 O3Pilot 根据标准化 Fact 计算。

例如：

- Product Return Rate；
- Order-to-Delivery P50；
- Current Days of Cover；
- Profit Margin；
- Cost Coverage。

---

## 3.4 `O3P_ESTIMATE`

依赖假设、缺失数据补全或近似模型。

例如：

- Lost Sales Estimate；
- 使用参考成本替代订单实际采购成本的 Profit；
- 未取得真实 Seller Logistics Charge 时的物流成本估算；
- 基于近期销量的简单补货量。

必须明确显示 `ESTIMATED`。

---

## 3.5 `O3P_FORECAST`

预测模型输出。

例如：

- Forecast Demand；
- Future Stockout Date；
- Forecasted Days of Cover。

预测值不得与 Actual Fact 混在同一字段。

---

# 4. Metric Contract

每个正式指标至少需要定义以下属性：

```text
metric_code
display_name
metric_origin
domain

grain
dimensions

time_basis
window

numerator
denominator

unit
currency_policy

eligibility
exclusions

aggregation_rule

metric_version
status
```

示例：

```text
metric_code = o3p.product.return_rate
grain = PRODUCT + PERIOD
time_basis = DELIVERY_COHORT
numerator = completed_returned_units_as_of
denominator = delivered_units_in_cohort
unit = RATIO
```

---

# 5. 通用粒度

正式粒度允许包括：

```text
SHOP
SELLER_CATALOG_ITEM
PRODUCT
SKU
MOTHER_ORDER
POSTING
POSTING_ITEM
WAREHOUSE
LOGISTICS_PROVIDER
DELIVERY_METHOD
RETURN_CASE
CAMPAIGN
CAMPAIGN_SKU
QUESTION
FINANCE_TYPE
DATE
PERIOD
```

复杂粒度组合示例：

```text
SHOP + SKU + DAY
CAMPAIGN + DAY
CAMPAIGN + SKU + DAY
PRODUCT + DELIVERY_COHORT
SHOP + FINANCE_TYPE + MONTH
```

---

# 6. 时间规则

## 6.1 时间必须指定 Basis

不得只写：

```text
2026-08-01 Sales
```

而不说明是哪种时间。

正式 `time_basis` 至少包括：

```text
ORDER_CREATED
POSTING_CREATED
PROCESSING_STARTED
HANDOVER
DELIVERED
CANCELLED
RETURN_CREATED
RETURN_COMPLETED
ACCRUAL_DATE
BUSINESS_DATE
SNAPSHOT_TIME
CAMPAIGN_DATE
IMPORT_DATE
DELIVERY_COHORT
ORDER_COHORT
SHIPPED_COHORT
BUYOUT_ELIGIBLE_COHORT
ACCRUAL_PERIOD
```

---

## 6.2 默认业务展示时区

O3Pilot 默认前端展示：

```text
Asia/Shanghai
UTC+8
```

但指标必须保留来源业务日语义。

不能把：

- Ozon Analytics Day；
- Ozon Finance Date；
- Ozon Exchange Rate Interval；
- Performance Date；

强行统一成北京时间日期后再计算。

---

## 6.3 Period Comparison

同比 / 环比 / 前期比较必须使用等长可比窗口。

例如：

```text
current = 2026-08-01 ~ 2026-08-07
previous = 2026-07-25 ~ 2026-07-31
```

变化率：

```text
(current - previous) / abs(previous)
```

如果：

```text
previous = 0
```

返回：

```text
NO_DENOMINATOR
```

不能返回无穷大。

---

# 7. Money 与 Reporting Currency

## 7.1 原始 Money 指标

原始金额指标必须保留：

```text
amount
currency
```

---

## 7.2 Reporting Currency

跨币种分析必须显式选择：

```text
reporting_currency
```

可以是：

- Shop settlement currency；
- USD；
- CNY；
- RUB；
- 用户指定币种。

不得假设所有店铺默认使用同一个币种。

---

## 7.3 FX Metric Rule

每次转换必须能够追溯：

```text
exchange_rate
exchange_rate_type
rate_basis_type
rate_basis_time
rate_policy_version
```

Ozon Business FX 与 Seller Cost FX 分开。

---

## 7.4 金额口径词典

以下金额不是同义词：

| 指标 | 含义 |
|---|---|
| `order_gross_value` | Order / Posting Item 侧卖家商品价格 × 数量 |
| `buyer_paid_value` | 订单侧 Buyer / Customer 实际支付观察值 |
| `ozon.analytics.revenue` | Ozon Analytics 返回的 Revenue |
| `finance_gross_sales` | Finance Type 69 seller_price × quantity 重建的 Finance Gross Sales |
| `finance_net_accrual` | Finance Accrual 全部 Signed Amount 的净和 |
| `performance_orders_money` | Performance 广告归因订单金额 |
| `payout_amount` | Ozon 财务报告中的实际 / 计划付款事实 |

页面不得把这些全部显示成同一个：

```text
销售额
```

而不标明 Metric Contract。

---

## 7.5 无 Reporting Currency 时的多币种结果

如果一个查询范围包含多个 Currency，且用户没有选择 Reporting Currency：

正确结果是：

```text
USD 100
CNY 500
```

或按 Currency 分组展示。

禁止自动求和成：

```text
600
```

---

# 8. Ratio、Percentile 与平均值

## 8.1 Ratio

内部统一保存：

```text
0.0 ~ 1.0
```

展示时转换成百分比。

Ozon 原始指标如果本身返回 0–100，则 Raw 保持原值，并在标准化层声明 Scale。

---

## 8.2 Average

平均值只用于适合均值解释的指标。

时效类指标默认同时提供：

```text
P50
P90
Average
Sample Count
```

P50 为主观察值。

---

## 8.3 Percentile

P50 / P90 必须从原始样本重新计算。

禁止：

```text
AVG(daily_p50)
```

作为整个 Period 的 P50。

---

# 9. 指标可用性

每个 Metric Result 建议同时拥有：

```text
value
status
sample_count
eligible_count
coverage_ratio
as_of_time
```

如果源数据不完整：

```text
status = PARTIAL
```

而不是静默返回“完整”的数字。

---

## 9.1 默认订单资格规则：仅排除发货前取消

O3Pilot 对订单类自算指标采用统一的第一层资格过滤：

```text
eligible_business_posting
=
已经创建
AND NOT confirmed_pre_shipment_cancelled
```

也就是说，**只有已经确认在实际发货 / 转交物流之前就取消的订单，才从默认经营统计中排除。**

以下订单都继续属于 `eligible_business_posting`：

```text
待备货且未取消
待发货且未取消
已发货
运输中
已签收
发货后取消
其他没有被确认属于发货前取消的有效订单
```

因此：

```text
未发货
!=
无效订单
```

不能因为：

```text
shipped_at IS NULL
```

就把订单从销售、订单金额、AOV、ADS 或 Forecast Actual 中删除。

### 发货前取消

统一定义：

```text
confirmed_pre_shipment_cancelled
=
Posting 已最终取消
AND
能够可靠确认取消发生在实际发货 / 转交物流节点之前
```

此类订单：

```text
不进入 eligible_business_posting
```

默认从以下 O3Pilot 经营指标排除：

- Sales Posting Count；
- Sales Units；
- Order Gross Value；
- Buyer Paid Value；
- AOV；
- Average Unit Price；
- SKU Sales Performance；
- Inventory ADS；
- Replenishment Demand；
- Default Cancellation Rate；
- Order / SKU Contribution Profit；
- Forecast Actual Demand；
- Cross-shop Sales Aggregation。

### 待发货但未取消

如果订单尚未发货，但截至 `as_of_time` 仍然有效、没有取消：

```text
pending_unshipped_active = true
```

则：

```text
posting IN eligible_business_posting
```

它应继续进入不要求后续生命周期节点的经营统计。

如果该订单未来最终在发货前取消，历史 Cohort 在重新计算时应将其排除。

因此近期订单 Cohort 可以处于：

```text
OPEN_COHORT
```

并允许指标随订单最终结果修订。

### 第二层：Metric-specific Eligibility

通过第一层资格过滤后，每个指标仍必须应用自身的生命周期要求。

例如：

```text
Sales Posting Count
→ eligible_business_posting
→ 待发货未取消订单包含

Delivery Duration
→ eligible_business_posting
→ 还必须存在 delivery / handover 所需时间节点

Product Return Rate
→ eligible_business_posting
→ 还必须进入对应 Delivered Cohort

Buyout Rate
→ eligible_business_posting
→ 还必须达到 Buyout Eligibility Stage
```

所以：

**“待发货未取消订单没有进入某个指标”必须是因为该指标自身需要更晚的生命周期节点，而不能只是因为它尚未发货。**

### 无法确认取消发生在发货前还是发货后

如果订单已经取消，但无法可靠确认取消发生在发货前还是发货后：

```text
cancellation_stage = UNVERIFIED
```

由于尚未确认它属于发货前取消，不能直接按发货前取消将其从所有经营指标删除。

相关 Cancellation / Fulfillment 指标应：

```text
status = PARTIAL
```

并展示 Cancellation Stage Coverage。

### 明确例外

以下事实不得因为本规则被删除或改写：

1. **Created Demand**

用于分析完整下单需求，包含发货前取消。

2. **OZON_SOURCE / OZON_REFERENCE**

保持 Ozon 自己的原始 / 官方口径。

3. **Finance / Settlement / Payout**

发货前取消可能已经产生真实费用，这些 Money Fact 必须保留。

4. **非订单 Cohort 指标**

例如 Product、Price、Inventory Snapshot、Rating、Data Quality。

---

# 10. 订单与销售基础指标

本章节的 `o3p.sales.*` 默认先使用：

```text
eligible_business_posting
```

即：

```text
所有已创建订单
- 已确认发货前取消订单
```

待发货但尚未取消的订单继续保留。

如果需要观察完整原始下单需求，包括发货前取消，则使用独立的 `o3p.demand.*` 指标。

---

# 10.1 有效母订单数

```text
metric_code:
o3p.sales.mother_order_count
```

定义：

```text
COUNT(DISTINCT mother_order_id)
FROM eligible_business_posting
```

只要一个母订单至少存在一个未被确认属于发货前取消的 Posting，即进入有效母订单统计。

Time Basis：

```text
ORDER_CREATED
```

近期未完结订单允许属于 `OPEN_COHORT`。

---

# 10.2 有效订单数 / Posting Count

```text
metric_code:
o3p.sales.posting_count
```

公式：

```text
COUNT(DISTINCT posting_id)
FROM eligible_business_posting
```

O3Pilot 中文“订单号”对应 `posting_number`。

不得与母订单数混用。

包含：

- 待备货未取消；
- 待发货未取消；
- 已发货；
- 已签收；
- 发货后取消。

排除：

- 已确认发货前取消。

---

# 10.3 有效销售商品件数 / Ordered Units

```text
metric_code:
o3p.sales.ordered_units
```

公式：

```text
SUM(posting_item.quantity)
WHERE posting IN eligible_business_posting
```

粒度：

```text
Posting Item
```

Time Basis：

```text
POSTING_CREATED
```

该指标包括待发货但未取消订单的商品件数。

它与：

```text
ozon.analytics.ordered_units
```

属于不同指标。

---

# 10.4 实际已发货商品件数

```text
metric_code:
o3p.fulfillment.shipped_units
```

公式：

```text
SUM(posting_item.quantity)
WHERE posting IN eligible_business_posting
  AND reliable shipped_at exists
```

这是履约阶段指标。

待发货未取消订单不进入该指标，是因为它尚未达到发货节点，而不是因为它被视为无效订单。

---

# 10.5 已签收商品件数

```text
metric_code:
o3p.sales.delivered_units
```

公式：

```text
SUM(posting_item.quantity)
WHERE posting IN eligible_business_posting
  AND posting normalized final state = DELIVERED
```

Time Basis：

```text
DELIVERED
```

状态映射必须版本化。

待发货未取消订单不进入该指标，是因为尚未签收。

---

# 10.6 发货后取消商品件数

```text
metric_code:
o3p.sales.post_shipment_cancelled_units
```

公式：

```text
SUM(posting_item.quantity)
WHERE posting IN eligible_business_posting
  AND reliable shipped_at exists
  AND final state = CANCELLED
  AND cancellation occurred after shipment
```

发货前取消件数不计入本指标。

---

# 10.7 Order Gross Value

```text
metric_code:
o3p.sales.order_gross_value
```

公式：

```text
SUM(unit_price_amount * quantity)
WHERE posting IN eligible_business_posting
```

使用 Posting Item 原始 seller/order price。

该指标是**排除发货前取消后的有效订单金额**，不是 Finance Revenue。

包含尚未发货但仍有效的订单金额。

必须按币种分组或转换。

---

# 10.8 Buyer Paid Value

```text
metric_code:
o3p.sales.buyer_paid_value
```

来源：

```text
posting_item_pricing_observation.customer_price
```

Eligibility：

```text
posting IN eligible_business_posting
```

规则：

仅当对应订单价格结构存在明确 Currency 时计算。

该指标用于：

- 买家支付观察；
- 促销分析；
- Order vs Finance 对账。

不作为 Profit 主收入字段。

---

# 10.9 平均客单价

必须提供两个明确版本。

## Mother Order AOV

```text
o3p.sales.aov_mother_order
=
order_gross_value
/
mother_order_count
```

## Posting AOV

```text
o3p.sales.aov_posting
=
order_gross_value
/
posting_count
```

分子和分母均使用同一个：

```text
eligible_business_posting
```

待发货未取消订单继续保留。

禁止仅显示“客单价”而不说明哪一种。

---

# 10.10 平均商品售价

```text
o3p.sales.average_unit_price
=
order_gross_value
/
ordered_units
```

必须在同一币种下计算。

---

# 10.11 下单需求指标

为了保留买家原始下单需求，单独定义：

```text
o3p.demand.created_mother_order_count
=
COUNT(DISTINCT mother_order_id)
by ORDER_CREATED
```

```text
o3p.demand.created_posting_count
=
COUNT(DISTINCT posting_id)
by POSTING_CREATED
```

```text
o3p.demand.created_ordered_units
=
SUM(posting_item.quantity)
by POSTING_CREATED
```

```text
o3p.demand.created_order_gross_value
=
SUM(unit_price_amount * quantity)
by POSTING_CREATED
```

这些指标包含发货前取消订单。

因此：

```text
Created Demand
- Pre-shipment Cancellation
= Eligible Sales Order Set
```

在订单仍待发货且未取消时，它同时存在于：

```text
Created Demand
Eligible Sales Order Set
```

如果未来在发货前取消，则从后者移除。

---

# 10.12 Ozon Analytics 销售指标

以下属于 `OZON_SOURCE`：

```text
ozon.analytics.revenue
ozon.analytics.ordered_units
ozon.analytics.returns
ozon.analytics.cancellations
ozon.analytics.delivered_units
```

这些值不得自动应用 O3Pilot 的发货前取消过滤，也不得覆盖 O3Pilot Order Fact 指标。

主要用途：

- Ozon 经营趋势；
- Ozon 口径观察；
- 与 O3Pilot Order Fact 对账；
- Forecast Feature。

---

# 11. 销售趋势

## 11.1 Sales Growth

```text
o3p.sales.growth_rate
=
(current_sales - previous_sales)
/
abs(previous_sales)
```

`current_sales` 必须指定使用哪一个金额指标。

推荐：

- Finance Analysis：`finance_gross_sales`；
- Traffic Analytics：`ozon.analytics.revenue`；
- Order Analysis：`order_gross_value`，默认排除已确认发货前取消；
- Demand Analysis：`o3p.demand.created_order_gross_value`，包含发货前取消。

禁止在同一趋势图中途切换 Source Metric。

---

## 11.2 Unit Growth

```text
o3p.sales.unit_growth_rate
=
(current_ordered_units - previous_ordered_units)
/
previous_ordered_units
```

如果 previous = 0：

```text
NO_DENOMINATOR
```

---

# 12. 流量与经营 Analytics

以下指标直接来自 `/v1/analytics/data`，属于 `OZON_SOURCE`：

```text
ozon.analytics.hits_view_search
ozon.analytics.hits_view_pdp
ozon.analytics.hits_view
ozon.analytics.hits_tocart_search
ozon.analytics.hits_tocart_pdp
ozon.analytics.hits_tocart
ozon.analytics.session_view_search
ozon.analytics.session_view_pdp
ozon.analytics.session_view
```

解析必须使用 DATA_SOURCES 中的 Metric Allowlist 与响应顺序校验。

---

# 13. 浏览 → 加购

```text
metric_code:
o3p.funnel.view_to_cart_rate
```

公式：

```text
hits_tocart
/
hits_view
```

要求：

- 分子和分母来自同一 `SKU + Day`；
- 相同 Analytics Contract；
- 同一时间窗口。

如果 Ozon 某日缺失对应 Metric：

```text
PARTIAL
```

---

# 14. 搜索表现指标

来自 Product Queries 的以下指标属于 `OZON_SOURCE`：

```text
ozon.search.unique_search_users
ozon.search.unique_view_users
ozon.search.position
ozon.search.view_conversion
ozon.search.gmv
ozon.search.query_order_count
```

`view_conversion` 直接保存 Ozon 原值。

O3Pilot v1.0 不重新定义一个同名 `view_conversion`。

---

# 15. 搜索词贡献

## 15.1 Query GMV Share

```text
o3p.search.query_gmv_share
=
query_gmv
/
sum(all_query_gmv_for_same_sku_period)
```

要求所有 GMV Currency 相同。

---

## 15.2 Query Order Share

```text
o3p.search.query_order_share
=
query_order_count
/
sum(all_query_order_count_for_same_sku_period)
```

---

# 16. 商品内容指标

# 16.1 Ozon Content Rating

```text
metric_code:
ozon.product.content_rating
```

来源：

```text
/v1/product/rating-by-sku
```

属于 `OZON_SOURCE`。

包括：

```text
total_rating
group_rating
group_weight
condition_fulfilled
improve_attributes
```

O3Pilot 不自行改变 Ozon Score 权重。

---

# 16.2 待完善属性数

```text
o3p.product.missing_recommended_attribute_count
=
COUNT(DISTINCT improve_attribute_id)
```

来源：

Ozon Content Rating 的 `improve_attributes`。

它表示 Ozon 当前评分系统建议完善的属性数量。

不是“类目全部缺失属性数”。

---

# 16.3 内容质量趋势

```text
o3p.product.content_rating_change
=
latest_content_rating - previous_content_rating
```

必须基于 Snapshot。

---

# 17. 价格与价格竞争力

# 17.1 当前 Seller Price

```text
ozon.price.seller_price
```

来源：

Product Price Snapshot。

---

# 17.2 Marketing Seller Price

```text
ozon.price.marketing_seller_price
```

属于 Source Fact。

---

# 17.3 Discount Rate

仅在 `old_price > 0` 时：

```text
o3p.price.discount_rate
=
(old_price - seller_price)
/
old_price
```

如果：

```text
old_price <= 0
```

返回：

```text
NOT_APPLICABLE
```

---

# 17.4 Ozon Price Index

以下保留为 Source Metric：

```text
ozon.price.color_index
ozon.price.external_index_value
ozon.price.ozon_index_value
ozon.price.self_marketplaces_index_value
```

不同比较价格可能拥有独立 Currency。

不得直接用不同币种金额做 Gap。

---

# 17.5 Competitive Price Gap

如果卖家价格与比较价格已经转换到同一 Reporting Currency：

```text
o3p.price.competitive_gap
=
seller_price_reporting
/
reference_price_reporting
- 1
```

解释：

```text
< 0
→ Seller Price 低于参考价

> 0
→ Seller Price 高于参考价
```

必须保存 FX Policy。

---

# 18. 促销效果

促销前后效果属于 O3Pilot Derived。

基础计算：

```text
lift
=
metric_during_promotion
/
metric_baseline
- 1
```

Baseline 必须使用 `baseline_policy_version`。

v1.0 推荐：

```text
同等长度、紧邻促销前、排除已知促销日的窗口
```

但如果季节性明显：

必须标记：

```text
ESTIMATED
```

不得把简单 Before / After 直接解释成因果效果。

---

# 19. 库存基础指标

# 19.1 Sellable Stock

```text
metric_code:
o3p.inventory.sellable_stock
```

当前 `/v4/product/info/stocks` 的 `present` 在已验证业务解释中作为在库可售数量。

因此：

```text
sellable_stock
=
SUM(present)
```

按选择的：

```text
shop
sku
stock_type
warehouse
```

聚合。

不得自动计算：

```text
present - reserved
```

作为可售库存。

---

# 19.2 Reserved Stock

```text
o3p.inventory.reserved_stock
=
SUM(reserved)
```

Reserved 独立展示。

---

# 19.3 In-transit Stock

```text
o3p.inventory.in_transit_stock
=
SUM(eligible inbound_supply_item quantity)
```

Eligible 状态由 `inbound_mapping_version` 定义。

当前库存与在途库存必须分开。

---

# 20. 日均销量

O3Pilot 同时定义两种 ADS。

ADS 的订单侧销量默认基于 `eligible_business_posting`：排除已确认发货前取消，但保留待发货且未取消订单。近期窗口因此可能属于 `OPEN_COHORT`。

# 20.1 Calendar ADS

```text
o3p.inventory.calendar_ads_units
=
eligible_ordered_units_in_window
/
calendar_days_in_window
```

适合：

- 稳定供货商品；
- 简单趋势；
- 透明展示。

---

# 20.2 Availability-adjusted ADS

```text
o3p.inventory.adjusted_ads_units
=
eligible_ordered_units_on_available_days
/
available_days
```

用于降低缺货造成的销量低估。

如果：

```text
available_days = 0
```

返回：

```text
NO_DENOMINATOR
```

默认分析窗口不在本文件硬编码。

允许：

```text
7
14
28
56
```

等版本化窗口。

---

# 21. Ozon 商品可用性 Reference

Ozon 当前 Seller Center FBP 商品可售性规则属于 `OZON_REFERENCE`。

单商品：

```text
ozon.reference.product_availability
=
商品在 FBP 或 rFBS 至少一种模式可销售的时间
/
总观察时间
```

官方资料当前主要以过去 28 天说明。

O3Pilot 不把该公式等同于自己的库存 Snapshot 可用率，除非完成对账。

---

# 22. O3Pilot Availability

```text
metric_code:
o3p.inventory.availability_rate
```

公式：

```text
available_eligible_days
/
eligible_observed_days
```

`available_eligible_day`：

```text
该日所选业务模式中至少一个模式 sellable_stock > 0
```

要求：

- 当天库存数据覆盖可靠；
- 没有 Sync Gap；
- 日状态生成规则版本化。

缺失库存数据的日期：

不得直接按“缺货日”处理。

---

# 23. 缺货天数

```text
o3p.inventory.out_of_stock_days
=
eligible_observed_days
-
available_eligible_days
```

---

# 24. 库存覆盖天数

# 24.1 Current Days of Cover

```text
metric_code:
o3p.inventory.days_of_cover
```

推荐：

```text
sellable_stock
/
adjusted_ads_units
```

如果：

```text
adjusted_ads_units = 0
```

返回：

```text
NO_RECENT_DEMAND
```

不能返回一个伪造的“999 天”。

---

# 24.2 Supply Days of Cover

包含在途库存：

```text
o3p.inventory.supply_days_of_cover
=
(sellable_stock + eligible_in_transit_stock)
/
adjusted_ads_units
```

该指标不等于当前库存覆盖天数。

---

# 25. Ozon Lost Sales Reference

Ozon 当前 FBP Seller Center Reference：

```text
daily_average_sales_amount
=
28-day sales amount
/
days product was available
```

```text
lost_sales
=
daily_average_sales_amount
*
out_of_stock_days
```

对于仅以 rFBS 销售的商品，Ozon 当前参考规则还会在结果基础上增加 30%。

因此该指标必须标记：

```text
metric_origin = OZON_REFERENCE
```

O3Pilot 不默认把 +30% 规则应用到自己的 Lost Sales Estimate。

---

# 26. O3Pilot Lost Sales Estimate

```text
metric_code:
o3p.inventory.lost_sales_estimate
```

基础公式：

```text
average_sales_amount_per_available_day
*
out_of_stock_days
```

属于：

```text
O3P_ESTIMATE
```

要求：

- 使用同币种；
- 排除数据缺口日；
- 保留观察窗口；
- 保留估算版本。

该值表示：

```text
estimated unrealized sales
```

不是 Finance Fact。

---

# 27. Ozon 补货建议 Reference

Ozon 当前 Reference 中：

```text
< 10 days
→ 紧急交货

10 ~ 30 days
→ 即将交货

> 30 days
→ 目前足够
```

另外存在：

- rFBS 日均销量 > 1 时建议转 FBP；
- 过去 60 天无 FBP 销售或库存可维持 90 天以上时可能标记“不交货”；
- 推荐量会考虑当前库存和在途库存；
- 销售分析周期可以使用 7 / 14 / 28 / 56 天。

这些阈值属于：

```text
OZON_REFERENCE
```

不得自动成为 O3Pilot 永久经营规则。

---

# 28. O3Pilot Forecast Replenishment

```text
metric_code:
o3p.inventory.recommended_replenishment_units
```

标准公式：

```text
required_units
=
forecast_demand(
    lead_time_days + target_cover_days
)
+
safety_stock_units
```

```text
recommended_replenishment_units
=
MAX(
    0,
    CEIL(
        required_units
        - sellable_stock
        - eligible_in_transit_stock
    )
)
```

其中：

```text
lead_time_days
```

来自卖家采购 / 备货 / 运输参数。

```text
target_cover_days
```

为经营 Policy。

不得在 METRICS v1.0 中永久固定成某一个天数。

---

# 29. Baseline Replenishment Estimate

当 Forecast 尚不可用时：

```text
baseline_demand
=
adjusted_ads_units
*
(lead_time_days + target_cover_days)
```

再使用同一补货公式。

此时：

```text
metric_origin = O3P_ESTIMATE
```

不能标成 Forecast。

---

# 30. 履约时效基础指标

所有 Duration 使用：

```text
seconds
```

作为内部标准单位。

前端可展示：

- 小时；
- 天；
- P50 / P90。

---

# 31. Order → Handover

```text
o3p.fulfillment.order_to_handover_duration
=
handover_to_delivery_at
-
created_at_source
```

要求：

```text
handover_to_delivery_at >= created_at_source
```

异常负值：

```text
UNVERIFIED / DATA_QUALITY_ISSUE
```

---

# 32. Processing → Handover

```text
o3p.fulfillment.processing_to_handover_duration
=
handover_to_delivery_at
-
in_process_at
```

---

# 33. Handover → Delivery

```text
o3p.fulfillment.delivery_transit_duration
=
fact_delivery_date
-
handover_to_delivery_at
```

---

# 34. Order → Delivery

```text
o3p.fulfillment.order_to_delivery_duration
=
fact_delivery_date
-
created_at_source
```

---

# 35. 时效分位数

对任何 Duration：

```text
P50 = percentile(samples, 0.50)
P90 = percentile(samples, 0.90)
Average = mean(samples)
```

必须同时展示：

```text
sample_count
```

如果某业务模式实际妥投时间缺失较多：

状态：

```text
PARTIAL
```

---

# 36. 按时发货率

```text
metric_code:
o3p.fulfillment.on_time_handover_rate
```

Eligible Population：

具有：

```text
shipment_deadline_at
handover_to_delivery_at
```

且未在发货前取消的 Posting。

公式：

```text
COUNT(handover_at <= shipment_deadline_at)
/
COUNT(eligible postings)
```

它是 O3Pilot Operational Metric。

不等同于 Ozon Rating。

---

# 37. 承诺配送达成率

```text
o3p.fulfillment.promise_delivery_hit_rate
```

Eligible：

同时拥有：

```text
promised_delivery_to
fact_delivery_date
```

公式：

```text
COUNT(fact_delivery_date <= promised_delivery_to)
/
COUNT(eligible delivered postings)
```

---

# 38. 发货截止时间变化

```text
o3p.fulfillment.deadline_shift_hours
=
latest_shipment_deadline
-
initial_shipment_deadline
```

正值：

Ozon 将 Deadline 向后延。

负值：

Deadline 提前。

基于：

```text
posting_schedule_history
```

---

# 39. 承诺配送窗口变化

```text
o3p.fulfillment.promise_window_shift_hours
```

必须分别保存：

```text
from_shift
to_shift
```

不要只保存一个“改变了 X 小时”。

---

# 40. 时效数据完整度

## Handover Completeness

```text
o3p.quality.handover_time_completeness
=
postings_with_handover_time
/
eligible_postings
```

## Delivery Completeness

```text
o3p.quality.delivery_time_completeness
=
delivered_postings_with_fact_delivery
/
delivered_postings
```

---

# 41. Ozon 官方取消指标 Reference

当前知识库中存在两个不同的官方取消相关口径。

不能合并。

## 41.1 Seller Center Fulfillment Report — 7 Day

按包裹数：

```text
ozon.reference.cancellation_rate_7d_count
=
seller_cancelled_shipments_in_7d
/
created_shipments_in_7d
```

按包裹货值：

```text
ozon.reference.cancellation_rate_7d_value
=
value_of_seller_cancelled_shipments
/
value_of_created_shipments
```

报告中存在：

```text
Accounted in numerator
Accounted in denominator
```

因此如用户导入官方报告，应优先使用这些官方标志对账。

---

## 41.2 Ozon Service Quality — 14 Day

官方服务质量资料描述：

```text
seller-responsible cancelled shipments
/
all eligible shipments
```

观察最近 14 天，并不统计当天。

卖家责任当前包括官方资料列出的：

- 卖家主动取消；
- Ozon 因延迟发货超过 7 天取消等情况。

该口径与 7 天 Fulfillment Report 指标不假定完全相同。

---

# 42. O3Pilot Cancellation Rate

O3Pilot 将取消拆成两个生命周期阶段：

```text
PRE_SHIPMENT_CANCELLATION
POST_SHIPMENT_CANCELLATION
```

默认经营 Cancellation Rate **只排除已经确认的发货前取消订单**。

待发货但未取消订单仍保留在默认分母中。

---

# 42.1 发货前取消率

```text
metric_code:
o3p.cancellation.pre_shipment_rate
```

公式：

```text
pre_shipment_cancelled_postings
/
created_postings
```

Cohort：

```text
POSTING_CREATED
```

其中：

```text
pre_shipment_cancelled_posting
=
Posting 在形成可靠 shipped_at 之前已经最终取消
```

该指标单独观察发货前损失。

---

# 42.2 默认订单取消率

```text
metric_code:
o3p.cancellation.posting_rate
```

公式：

```text
post_shipment_cancelled_postings
/
eligible_business_postings
```

其中分母：

```text
eligible_business_postings
=
created_postings
- confirmed_pre_shipment_cancelled_postings
```

因此分母包含：

- 尚未发货且未取消；
- 已发货；
- 运输中；
- 已签收；
- 发货后取消。

分母排除：

- 已确认发货前取消。

分子只包含：

- 已确认发生在发货后的取消 Posting。

因此待发货未取消订单会进入分母，但不会进入分子。

近期 Cohort 可能因为待发货订单未来发生发货前取消而被修订，允许标记：

```text
OPEN_COHORT
```

---

# 42.3 买家责任取消率

```text
metric_code:
o3p.cancellation.buyer_rate
```

公式：

```text
buyer_responsible_post_shipment_cancelled_postings
/
eligible_business_postings
```

买家责任必须根据：

- Cancellation Initiator；
- Reason；
- Ozon Raw Fields；
- Mapping Version；

判断。

无法确认：

```text
UNKNOWN
```

不得自动归为买家责任。

---

# 42.4 卖家责任取消率

```text
metric_code:
o3p.cancellation.seller_responsible_rate
```

公式：

```text
seller_responsible_post_shipment_cancelled_postings
/
eligible_business_postings
```

责任同样必须基于标准化 Cancellation Mapping。

注意：

该指标是 O3Pilot 自算指标。

它不等于 Ozon Service Quality 官方卖家责任取消率，因为官方指标可能拥有不同时间窗、资格规则和责任判定。

---

# 42.5 取消商品件数率

```text
metric_code:
o3p.cancellation.unit_rate
```

公式：

```text
post_shipment_cancelled_units
/
eligible_ordered_units
```

`eligible_ordered_units` 包含待发货未取消订单件数，排除已确认发货前取消件数。

---

# 42.6 Cancellation Stage Coverage

```text
o3p.cancellation.stage_coverage
=
cancelled_postings_with_verified_pre_or_post_shipment_stage
/
all_cancelled_postings_requiring_stage_classification
```

如果无法可靠区分：

```text
发货前取消
vs
发货后取消
```

相关取消指标必须标记：

```text
PARTIAL
```

不能把未知取消阶段自动归到发货前取消，也不能据此从默认经营集合删除。

---

# 43. Ozon 逾期交货 Reference

Seller Center 当前 Reference：

观察最近 14 天。

```text
ozon.reference.shipment_delay_rate_14d
=
shipments_delayed_due_to_seller
/
shipments_that_should_have_been_handed_over
```

官方报告可包含：

```text
Accounted in numerator
Accounted in denominator
Delay in calendar days
```

因此官方报告导入可以用于核对 O3Pilot 的时效派生指标。

---

# 44. 退货指标分类

退货必须区分：

```text
RETURN_REQUEST
CUSTOMER_RETURN
UNCLAIMED
CANCELLATION_REVERSE_FLOW
WHD
RETURN_TO_SELLER
DESTROYED
COMPENSATED
```

不能把 `/v1/returns/list` 中所有 `Cancellation` 都当作“客户退货”。

---

# 45. Return Request Rate

如果需要观察买家发起退货行为：

```text
o3p.return.request_rate
=
return_requested_units
/
delivered_units_in_eligible_cohort
```

包括被拒绝的 Request 与否必须由 Metric Variant 明确。

建议两个 Variant：

```text
request_all
request_accepted
```

---

# 46. 产品退货率

这是 O3Pilot 核心派生指标。

```text
metric_code:
o3p.product.return_rate
```

## 46.1 Primary Definition

Cohort：

```text
DELIVERY_COHORT
```

分母：

```text
delivered_units_in_cohort
```

分子：

```text
completed_customer_return_units
linked back to those delivered units
as of calculation time
```

公式：

```text
completed_customer_return_units_as_of
/
delivered_units_in_cohort
```

---

## 46.2 不计入分子

默认不计入产品退货率：

- 发货前取消；
- 未提件；
- 仅创建但被拒绝的退货申请；
- 无法可靠关联到原销售的逆向记录；
- WHD 二次销售本身。

这些应使用其他指标观察。

---

## 46.3 Open Cohort

最近交付的商品未来仍可能退货。

因此近期 Product Return Rate 必须保存：

```text
as_of_date
cohort_status = OPEN_COHORT
```

当前资料不足以为所有商品类目硬编码统一“退货成熟期”。

因此 v1.0 不定义固定：

```text
30 天后一定 FINAL
```

---

## 46.4 Return Unit 去重

同一个 Posting Item 可能出现：

- 多次状态更新；
- List / Detail 重复观察；
- 多个逆向物流阶段；
- WHD 后续事件。

因此每个原始售出 Unit 在 Product Return Rate 分子中最多计入一次。

原则：

```text
completed_returned_units
<=
eligible_delivered_units
```

如果来源关系无法可靠判定是否为同一实物：

```text
status = PARTIAL
```

不得把所有逆向记录直接相加。

---

# 47. 退货原因占比

```text
o3p.return.reason_share
=
returned_units_for_reason
/
all_returned_units
```

按标准化 Reason Group 聚合。

Raw Reason 必须保留。

---

# 48. Ozon 认购水平 Reference

Ozon 当前“已认购商品”处于测试模式。

官方 Reference 对每件商品使用：

```text
最近最多 50 个订单
```

如果少于 50 个，则使用所有订单。

公式：

```text
ozon.reference.buyout_rate_last_50_orders
=
(order_count - return_count - cancellation_count)
/
order_count
```

官方还根据：

- 类目；
- 价格区段；

把结果划分为低 / 中 / 高水平。

这些阈值不是当前 O3Pilot API Contract 的稳定数据，因此不能自行猜测。

---

# 49. O3Pilot 认购率

O3Pilot 自算认购率先应用全局订单资格规则：发货前取消订单完全排除，待发货未取消订单仍属于有效订单集合。

但认购率本身还需要达到认购资格阶段，因此待发货订单不会仅因为“有效”就进入认购率分母。

```text
metric_code:
o3p.product.buyout_rate
```

Cohort：

```text
BUYOUT_ELIGIBLE_COHORT
```

分母：

```text
buyout_eligible_units
```

分子：

```text
buyout_eligible_units
- post_shipment_cancelled_or_unclaimed_units
- completed_customer_return_units_as_of
```

公式：

```text
buyout_success_units_as_of
/
buyout_eligible_units
```

其中：

```text
buyout_success_units_as_of
=
buyout_eligible_units
- eligible_post_shipment_non_buyout_units
- completed_customer_return_units_as_of
```

要求：

- 同一实物不能因多个逆向物流阶段重复扣减；
- 发货前取消不进入分母；
- 无法区分未认购、发货后取消、客户退货时标记 `PARTIAL`；
- Recent Cohort 可以保持 `OPEN_COHORT`。

属于：

```text
O3P_DERIVED
```

与 Ozon 最近 50 个订单官方 Reference 分开。

---

# 50. Non-buyout Rate

```text
o3p.product.non_buyout_rate
=
1 - o3p.product.buyout_rate
```

分母同样是：

```text
buyout_eligible_units
```

可展示组成：

```text
post_shipment_cancellation_component
unclaimed_component
return_component
```

发货前取消不属于 Non-buyout。

它属于独立：

```text
pre_shipment_cancellation
```

---

# 51. WHD 指标

只有在 `reverse_logistics_link` 可靠匹配时才计算。

## WHD Resale Rate

```text
o3p.whd.resale_rate
=
units_resold_from_whd
/
units_entered_whd
```

## WHD Recovery Revenue

```text
o3p.whd.recovery_revenue
=
recognized_revenue_from_linked_whd_resales
```

如果无法可靠建立：

```text
Return → WHD → Resale
```

状态：

```text
UNVERIFIED
```

不能按相同 SKU 猜测关联。

---

# 52. Finance 基础规则

Finance 原始金额保持 Signed Money。

例如：

```text
收入 / Credit
→ positive

费用 / Debit
→ negative
```

如果某接口实际符号不同，以 Source Fact 为准。

Metric Layer 可以生成 Positive Cost View，但必须与 Raw Signed Amount 分开。

---

# 53. Finance Net Accrual

```text
metric_code:
o3p.finance.net_accrual
```

公式：

```text
SUM(finance_accrual.total_amount)
```

按：

```text
ACCRUAL_DATE
```

统计。

这是 Ozon 财务净应计视图。

---

# 54. Finance Gross Sales

当前已完成全账期实测验证：

```text
metric_code:
o3p.finance.gross_sales
```

公式：

```text
SUM(
    Type69.seller_price.amount
    *
    Type69.quantity
)
```

当前实测中：

```text
Type 69 = SaleCommission
```

且该公式能够精确重建旧 Finance `accruals_for_sale`。

要求：

- seller_price 非空；
- quantity 非空；
- Currency 相同或转换。

该指标属于：

```text
O3P_DERIVED_FROM_VERIFIED_FINANCE
```

在 Metric Origin 中归 `O3P_DERIVED`。

---

# 55. Finance Commission Cost

```text
o3p.finance.sale_commission_cost
=
- SUM(Type69 accrued signed amount)
```

如果某期存在正向冲销：

应参与 Net Sum。

因此最终 Cost 可以降低，甚至在极端情况下为负。

不能简单：

```text
SUM(ABS(each row))
```

---

# 56. Finance 已验证费用组

当前新旧 Finance 全账期对账已经验证以下聚合关系。

## Processing & Delivery

```text
Type 67
+ Type 32
+ Type 29
+ Type 98
```

## Refunds & Cancellations

```text
Type 59
+ Type 45
```

## Services

当前已验证样本：

```text
Type 66
+ Type 41
+ Type 52
+ Type 54
+ Type 15
```

## Money Transfer

当前样本：

```text
Type 10
```

## Acquiring

```text
Type 1
```

这些映射必须带：

```text
finance_classification_version
```

`compensation_amount` 的非零新版映射当前仍未验证。

---

# 57. Finance Fee Ratio

```text
o3p.finance.fee_ratio
=
net_ozon_cost
/
finance_gross_sales
```

其中：

```text
net_ozon_cost
```

只包含明确分类为 Cost 的 Finance Component。

若大量 Finance Amount 仍未分类：

```text
status = PARTIAL
```

---

# 58. Finance Unclassified Ratio

```text
o3p.finance.unclassified_amount_ratio
=
ABS(unclassified_finance_amount)
/
SUM(ABS(all_finance_amount))
```

用于检测 Finance Classification 是否足以支持 Profit。

---

# 59. Settlement 与 Payout

以下指标只有在导入对应官方财务报告后才可用。

```text
ozon.report.payout_amount
ozon.report.planned_payment_date
ozon.report.actual_payment_date
ozon.report.withheld_amount
```

Finance Accrual 不得推断“已付款”。

---

# 60. Payout Reconciliation Difference

只有在 Settlement Period 已明确建立：

```text
Accrual
↔ Settlement
↔ Payout
```

映射后才能计算。

```text
o3p.finance.payout_reconciliation_difference
=
actual_payout
-
expected_payout_under_reconciliation_contract
```

`expected_payout` 不在 v1.0 中用简单：

```text
SUM(all accrual)
```

硬编码。

需要具体结算报告 Contract。

---

# 61. Profit 的两种时间视图

O3Pilot 必须区分：

```text
ACCRUAL_PERIOD
ORDER_COHORT
```

---

## 61.1 Accrual Period Profit

回答：

> 这个财务期间 Ozon 实际记了多少钱，再扣除卖家外部成本后剩多少？

Accrual Period Profit **不应用“排除发货前取消”的销售 Cohort 规则**。

原因：

发货前取消订单仍可能真实产生：

- Ozon 取消费用；
- 服务费；
- 卖家物流 / 包装成本；
- 其他真实经营成本。

这些已经发生的 Money Fact 必须保留在财务期间利润中。

---


## 61.2 Order Cohort Profit

回答：

> 某批有效订单截至当前最终贡献了多少利润？

默认订单集合：

```text
eligible_business_posting
```

即排除已确认发货前取消，但保留待发货未取消订单。

对于仍未完成履约或 Finance 尚未充分产生的订单：

```text
status = OPEN_COHORT / PARTIAL
```

发货前取消订单不进入 Sales Order Contribution Profit。

它们产生的真实费用单独归入：

```text
Pre-shipment Cancellation Financial Impact
```

并继续进入 Shop / Accrual Period Profit。

订单创建后发生的：

- 后续 Finance；
- 退货；
- 逆向物流；
- 赔偿；

应在 As-of 计算中归回原订单。

两个 Profit 不能混成同一个数字。

---

## 61.3 Pre-shipment Cancellation Financial Impact

发货前取消订单虽然不进入销售经营 Cohort，但其真实成本不能消失。

```text
metric_code:
o3p.profit.pre_shipment_cancellation_financial_impact
```

包括已经可靠归属于发货前取消订单的：

- Ozon Finance 取消 / 服务费用；
- 已发生 Seller Logistics Cost；
- 已发生 Packaging / Handling Cost；
- 其他实际外部成本；
- Compensation / Credit。

计算仍遵循：

```text
Actual Money Fact First
```

该指标用于回答：

> 发货前取消实际造成了多少经营损失？

它不属于 Sales Revenue，也不把发货前取消订单重新放回 `eligible_business_posting`。

---

# 62. 卖家采购成本

## 62.1 Actual Procurement Cost

优先：

```text
order_item_cost.transaction_cost_amount
```

Cost Source：

```text
ACTUAL_ORDER_ITEM_COST
```

---

## 62.2 Cost Fallback

如果历史订单缺少实际成本：

优先级可按 Policy：

```text
1. ACTUAL_ORDER_ITEM_COST
2. SKU_COST_AT_ORDER_TIME
3. CURRENT_SKU_COST
4. MISSING
```

任何使用第 2 / 3 类的结果：

```text
status = ESTIMATED
```

---

# 63. Procurement Cost Coverage

```text
o3p.profit.procurement_cost_coverage
=
units_with_actual_order_item_cost
/
eligible_business_ordered_units
```

建议 Profit 页面始终展示。

---

# 64. Seller Logistics Cost

```text
o3p.cost.seller_logistics
=
SUM(seller_logistics_charge.amount)
```

按 Seller-owned 原始账单 Currency 处理。

它不等于 Ozon Finance 中的物流费用。

---

# 65. Seller Logistics Coverage

```text
o3p.profit.seller_logistics_coverage
=
eligible_postings_with_mapped_seller_logistics_cost
/
eligible_postings_requiring_seller_logistics_cost
```

如果物流商导入只能到 Posting：

不能假装 SKU Cost Coverage 为 100%。

---

# 66. Profit Cost Convention

Metric Layer 的成本统一使用正数 Magnitude：

```text
procurement_cost = positive
seller_logistics_cost = positive
ozon_fee_cost = positive
ad_cost = positive
```

Raw Finance 仍保持 Signed Amount。

---

# 67. Shop Net Operating Profit

```text
metric_code:
o3p.profit.shop_net_operating_profit
```

Accrual Period 版本：

```text
finance_net_accrual
-
procurement_cost_of_recognized_sales
-
seller_logistics_cost
-
packaging_cost
-
other_external_seller_cost
```

重要：

`finance_net_accrual` 已经包含 Ozon Finance 中的：

- 佣金；
- 物流费用；
- 广告费用；
- 订阅；
- 收单；
- 其他平台应计；

因此不得再把同一笔 Finance Expense 重复扣除。

---

# 68. Order / SKU Contribution Profit

订单或 SKU 层使用：

```text
recognized_gross_sales
+ allocated_finance_credits
- attributable_ozon_finance_cost
- procurement_cost
- seller_logistics_cost
- packaging_cost
- other_attributable_external_cost
```

Posting 级 Finance 需要先通过：

```text
finance_allocation
```

分摊。

如果费用无法可靠分摊：

Profit Result：

```text
PARTIAL
```

而不是把费用忽略后展示成完整利润。

---

# 69. 广告费用在 Profit 中的去重

Performance Advertising 与 Finance Advertising 必须遵守：

```text
Finance = Booked Monetary Truth
Performance = Attribution Truth
```

如果 Finance 已经包含广告扣费：

Performance `moneySpent` 不得再次作为额外成本扣除。

推荐方式：

```text
Finance Advertising Cost
↓
使用 Performance 数据作为分摊权重
↓
Campaign / SKU Advertising Cost Allocation
```

如果 Finance 广告费用暂时不可用，而使用 Performance Spend：

```text
cost_source = PERFORMANCE_ESTIMATE
status = ESTIMATED
```

---

# 70. Profit Margin

## Shop Net Profit Margin

```text
o3p.profit.net_margin
=
shop_net_operating_profit
/
finance_gross_sales
```

---

## Contribution Margin

```text
o3p.profit.contribution_margin
=
contribution_profit
/
recognized_gross_sales
```

Denominator <= 0：

```text
NO_DENOMINATOR
```

---

# 71. Profit per Unit

```text
o3p.profit.per_unit
=
contribution_profit
/
recognized_eligible_ordered_units
```

---

# 72. Profit Coverage

Profit 不使用单一“可信度分数”。

必须展示独立 Coverage：

```text
finance_coverage
procurement_cost_coverage
seller_logistics_coverage
finance_allocation_coverage
fx_coverage
product_mapping_coverage
```

这样用户知道 Profit 缺的是哪一部分。

---

# 73. Minimum Profitable Price

价格建议属于 Decision Metric。

不能用一个永久简化公式处理所有佣金和费率。

定义：

```text
o3p.price.minimum_profitable_price
=
minimum price P
such that
scenario_profit(P) >= target_profit
```

Scenario Engine 必须使用：

- 对应履约模式；
- 佣金；
- Ozon Costs；
- Seller Costs；
- FX；
- Target Margin / Profit。

属于：

```text
O3P_ESTIMATE
```

除非所有输入均为确定性 Contract。

---

# 74. Advertising Source Metrics

Performance API 以下属于 `OZON_SOURCE`：

```text
ozon.ads.views
ozon.ads.clicks
ozon.ads.spend
ozon.ads.orders
ozon.ads.orders_money
ozon.ads.to_cart
ozon.ads.ctr_raw
ozon.ads.cpc_raw
ozon.ads.drr_raw
```

Campaign List Budget 与 Statistics Money 使用不同 Scale Contract。

不得统一 `/ 1,000,000`。

---

# 75. Advertising CTR

审计计算：

```text
o3p.ads.ctr
=
clicks
/
views
```

展示百分比。

用于：

- 与 `ctr_raw` 核验；
- 聚合多个 Campaign 时重新计算。

禁止直接平均 Campaign CTR。

---

# 76. Advertising CPC

```text
o3p.ads.cpc
=
spend
/
clicks
```

Currency：

Performance Stats 当前实测为 RUB。

实现仍保存实际 Currency Contract。

---

# 77. Advertising DRR

```text
o3p.ads.drr
=
spend
/
orders_money
```

展示百分比。

当前实测样本：

```text
6878.41 / 64671 ≈ 10.6%
```

与 Performance Raw DRR 一致。

DRR 是广告消耗占广告归因订单金额的比例。

---

# 78. Advertising ROAS

```text
o3p.ads.roas
=
orders_money
/
spend
```

与 DRR 是倒数关系：

```text
ROAS = 1 / DRR
```

仅在 DRR 使用 Ratio 形式时成立。

---

# 79. Ad Order Conversion

```text
o3p.ads.click_to_order_rate
=
orders
/
clicks
```

---

# 80. Ad Cart Rate

```text
o3p.ads.click_to_cart_rate
=
to_cart
/
clicks
```

---

# 81. Ad Revenue Share

```text
o3p.ads.attributed_revenue_share
=
performance_orders_money
/
total_sales_same_period_reporting_currency
```

用途：

衡量销售对广告归因的依赖程度。

Performance `orders_money` 是广告归因金额。

不能把它当成全店 Sales。

---

# 82. Ad Spend Share

```text
o3p.ads.spend_share_total_sales
=
performance_spend
/
total_sales_same_period_reporting_currency
```

这是经营分析 Ratio。

不是新的费用。

不能因为计算了 Spend Share，就在 Profit 再次扣广告费用。

---

# 83. Campaign Budget Utilization

仅当统计 Period 与预算 Period 可以严格匹配时：

```text
o3p.ads.budget_utilization
=
spend_in_budget_period
/
effective_budget
```

如果 Campaign 使用 Weekly Budget：

必须按 Campaign 自身：

```text
startWeekDay
endWeekDay
```

确定周期。

不能拿任意 7 天窗口除以 Weekly Budget。

---

# 84. Campaign 数据异常

单次：

```text
Campaign List = []
```

不构成 Metric：

```text
campaign_count = 0
```

如果历史已有大量 Campaign，应：

```text
metric_status = UNVERIFIED
```

等待重采样。

---

# 85. Questions

# 85.1 Question Count

```text
o3p.voc.question_count
=
COUNT(question)
```

---

# 85.2 Answered Question Rate

```text
o3p.voc.question_answer_rate
=
questions_with_at_least_one_answer
/
eligible_questions
```

---

# 85.3 First Response Time

```text
o3p.voc.first_answer_duration
=
first_answer.published_at
-
question.created_at_source
```

提供：

```text
P50
P90
Average
Sample Count
```

---

# 85.4 Unanswered Aging

当前仍未回答的问题：

```text
o3p.voc.unanswered_age
=
as_of_time
-
question.created_at_source
```

---

# 86. Reviews

Review 当前属于：

```text
ACCESS_DEPENDENT
```

在权限不可用时：

```text
review metrics = UNAVAILABLE
```

不得通过 Seller Info 的店铺平均商品评分反推出 Review Count。

---

# 87. Shop Rating

以下属于 `OZON_SOURCE`：

```text
ozon.rating.current_value
ozon.rating.history_value
ozon.rating.status_danger
ozon.rating.status_warning
ozon.rating.status_premium
```

O3Pilot 不重新定义 Ozon Threshold。

---

# 88. Seller Info Rating Snapshot

例如：

```text
rating_review_avg_score_total
rating_shipment_delay_cb
rating_price_green
rating_price_yellow
rating_price_red
```

都作为 Source Metric 保存。

即使 O3Pilot 自己能计算类似指标：

也不能覆盖官方值。

---

# 89. Price Zone Share

若 Seller Info 返回：

```text
green
yellow
red
super
```

各区占比，则直接保存 Source。

O3Pilot 可以计算变化：

```text
latest_share - previous_share
```

但不重新定义 Ozon Color Index 分类规则。

---

# 90. FBS Error Index

```text
ozon.rating.fbs_error_index
```

属于 Source Metric。

Error Posting 当前缺少非空真实样本时：

相关明细原因指标：

```text
UNVERIFIED
```

---

# 91. Multi-shop 聚合

只有已明确映射到同一个：

```text
seller_catalog_item
```

的 Product 才能做跨店商品聚合。

---

# 92. Cross-shop Units

```text
o3p.catalog.total_ordered_units
=
SUM(eligible_ordered_units across mapped products)
```

单位必须一致。

---

# 93. Cross-shop Revenue

O3Pilot 默认跨店销售收入使用已映射商品中排除发货前取消后的 Order Gross Value。

先统一 Reporting Currency：

```text
o3p.catalog.total_revenue
=
SUM(converted order_gross_value from eligible_business_postings)
```

如果展示：

```text
Ozon Analytics Revenue
Finance Gross Sales
```

必须使用独立 Metric Code，不得与本指标混合。

必须保存：

```text
fx_coverage
```

---

# 94. Cross-shop Return Rate

禁止：

```text
AVG(shop_return_rate)
```

正确：

```text
SUM(returned_units)
/
SUM(delivered_units)
```

仅包括完成 Seller Catalog Mapping 的商品。

---

# 95. 数据新鲜度

# 95.1 Fetch Age

```text
o3p.data.fetch_age
=
now
-
last_successful_fetch_at
```

---

# 95.2 Coverage Lag

```text
o3p.data.coverage_lag
=
now
-
latest_source_business_time
```

适用于：

- Analytics；
- Performance；
- Finance；
- Search。

Fetch Age 与 Coverage Lag 不得混用。

---

# 96. 数据完整度

通用：

```text
o3p.data.completeness
=
usable_expected_records
/
expected_records
```

`expected_records` 必须有来源 Contract。

如果无法定义 Expected：

不能伪造一个完整度百分比。

---

# 97. Mapping Coverage

```text
o3p.data.mapping_coverage
=
matched_records
/
eligible_records
```

适用于：

- ERP → Posting Item；
- Logistics → Posting / Package；
- Finance → Posting；
- Seller Catalog Mapping。

---

# 98. Reconciliation Match Rate

```text
o3p.data.reconciliation_match_rate
=
matched_comparable_fields
/
all_comparable_fields
```

或按 Record：

```text
matched_records
/
compared_records
```

必须在 Metric Variant 中说明。

---

# 99. Duplicate Rate

```text
o3p.data.duplicate_rate
=
duplicate_source_objects
/
received_source_objects
```

同一幂等重采集不应被算成业务重复。

---

# 100. Sync Gap

同步缺口不建议压缩成单一百分比。

至少保存：

```text
gap_start
gap_end
source
domain
```

可生成：

```text
o3p.data.gap_duration
```

---

# 101. Forecast 基本定义

Forecast 必须有：

```text
forecast_origin_date
target_start
target_end
horizon_days
model_version
```

实际值出现后才能回测。

---

# 102. Forecast Demand

```text
o3p.forecast.demand_units
```

表示：

```text
预测目标区间内的实际可履约需求件数
```

O3Pilot 默认 Forecast Target：

```text
actual = eligible_ordered_units
```

即：

```text
所有创建订单件数
- 已确认发货前取消件数
```

待发货但未取消订单继续进入 Forecast Actual；近期窗口可以是 `OPEN_COHORT`，如果订单之后在发货前取消，回测 Actual 应随最终事实修订。

如果未来需要预测“原始下单需求”，必须使用独立 Forecast Variant：

```text
forecast_target = CREATED_ORDER_DEMAND
```

不能与默认库存消耗 / 补货预测混用。

不能与：

```text
recommended_replenishment_units
```

混为一个值。

---

# 103. Forecast Error

定义：

```text
error = forecast - actual
absolute_error = ABS(forecast - actual)
```

解释：

```text
error > 0
→ 高估

error < 0
→ 低估
```

---

# 104. MAE

```text
o3p.forecast.mae
=
AVG(ABS(forecast - actual))
```

单位：

```text
pcs
```

直观，但跨 SKU 汇总时不考虑规模差异。

---

# 105. RMSE

```text
o3p.forecast.rmse
=
SQRT(
    AVG(
        (forecast - actual)^2
    )
)
```

更强调大误差。

---

# 106. WAPE

O3Pilot Forecast Backtest 的主要总体误差指标。

```text
o3p.forecast.wape
=
SUM(ABS(forecast - actual))
/
SUM(actual)
```

如果：

```text
SUM(actual) = 0
```

返回：

```text
NO_DENOMINATOR
```

WAPE 比简单 MAPE 更适合存在低销量 / 0 销量 SKU 的组合评估。

---

# 107. Forecast Bias

```text
o3p.forecast.bias
=
SUM(forecast - actual)
/
SUM(actual)
```

解释：

```text
> 0
→ 系统整体高估需求

< 0
→ 系统整体低估需求
```

---

# 108. MAPE

仅作为辅助：

```text
o3p.forecast.mape_nonzero
=
AVG(
    ABS(forecast - actual)
    /
    actual
)
WHERE actual > 0
```

必须明确：

```text
nonzero actual only
```

不能把 0 Actual 样本静默删除而仍称“全部 SKU MAPE”。

---

# 109. Forecast Accuracy Score

如果 UI 需要一个“准确率”百分比，可派生：

```text
o3p.forecast.accuracy_score
=
MAX(0, 1 - WAPE)
```

展示：

```text
0% ~ 100%
```

但正式分析仍应同时展示：

```text
WAPE
Bias
MAE
```

因为 Accuracy Score 会把所有 `WAPE > 100%` 压成 0%。

---

# 110. Forecast Backtest Grain

回测必须按：

```text
SKU + Forecast Origin + Horizon
```

保存。

不能只保存某个月一个“总体准确率”。

---

# 111. Horizon-specific Accuracy

必须分开：

```text
7-day
14-day
30-day
60-day
```

或其他 Forecast Horizon。

禁止把不同 Horizon 的预测误差混成一个 WAPE。

---

# 112. Forecast Coverage

```text
o3p.forecast.coverage
=
forecasted_eligible_skus
/
all_eligible_skus
```

如果只对畅销 SKU 预测：

总体 Accuracy 不能宣称覆盖全商品。

---

# 113. Stockout Projection

```text
o3p.forecast.projected_stockout_date
```

根据：

- 当前 sellable stock；
- eligible in-transit；
- Forecast Demand；

模拟未来库存路径。

属于：

```text
O3P_FORECAST
```

---

# 114. Projected Days of Cover

```text
o3p.forecast.projected_days_of_cover
```

与当前：

```text
o3p.inventory.days_of_cover
```

分开。

前者使用 Forecast。

后者使用近期销售速度。

---

# 115. Recommendation Priority

Recommendation 不是原始 Metric。

可根据多个 Metric 生成 Priority。

例如补货建议可以考虑：

- projected stockout；
- days of cover；
- lost sales estimate；
- lead time；
- in-transit stock；
- forecast error；
- profit contribution。

Priority 算法属于 Recommendation Model，并必须版本化。

---

# 116. Alert Metric Rule

Alert 不应重新实现业务公式。

Alert 结构：

```text
Metric
+
Threshold
+
Window
+
Rule Version
```

例如：

```text
metric = o3p.inventory.days_of_cover
condition = < 10
```

该 10 天是 Alert Policy，不是 Metric Definition。

---

# 117. 异常变化指标

基础变化：

```text
delta
=
current - baseline
```

```text
delta_rate
=
(current - baseline)
/
abs(baseline)
```

“异常”本身还需要：

- Threshold；
- Seasonality；
- Sample Size；
- Model；

因此 `METRICS.md` 不把一个简单 `-20%` 永久定义成“异常”。

---

# 118. 指标状态优先级

当同一个 Metric 同时存在多个风险时，建议状态优先级：

```text
UNAVAILABLE
UNVERIFIED
STALE
PARTIAL
ESTIMATED
OPEN_COHORT
VALID
```

例如：

数据只覆盖一半且又使用成本 Fallback：

主状态：

```text
PARTIAL
```

同时可以保存：

```text
estimate_flag = true
```

---

# 119. 指标展示最小上下文

任何 Ratio 至少展示：

```text
value
numerator
denominator
period
status
```

任何 P50 / P90 至少展示：

```text
value
sample_count
period
status
```

任何 Money 至少展示：

```text
amount
currency
period
```

跨币种 Money：

还应能查看：

```text
reporting_currency
fx_policy
```

---

# 120. Source Metric 与 Derived Metric 对账

对于能够双算的指标，O3Pilot 应允许：

```text
Source Value
Derived Value
Difference
Difference %
```

但 Derived 不能覆盖 Source。

典型：

- Analytics Ordered Units vs O3Pilot Shipped Units / Created Demand Units；
- Ozon Rating vs O3Pilot Operational Rate；
- Performance Raw CTR vs Recomputed CTR；
- Finance Gross Sales vs Order Gross Value；
- Official Report vs API。

---

# 121. 指标差异状态

建议：

```text
MATCH
WITHIN_TOLERANCE
DIFFERENT
NOT_COMPARABLE
PARTIAL
```

Money Difference 的 Tolerance 必须在 Reconciliation Contract 中定义。

不能全系统共用一个固定误差。

---

# 122. 核心 Metric Registry

| Domain | Metric Code | Origin | Primary Grain |
|---|---|---|---|
| Sales | `o3p.sales.mother_order_count` | O3P_DERIVED | Shop + Period |
| Sales | `o3p.sales.posting_count` | O3P_DERIVED | Shop + Eligible Order Cohort |
| Demand | `o3p.demand.created_posting_count` | O3P_DERIVED | Shop + Created Cohort |
| Demand | `o3p.demand.created_ordered_units` | O3P_DERIVED | SKU + Created Cohort |
| Sales | `o3p.sales.ordered_units` | O3P_DERIVED | SKU + Eligible Order Cohort |
| Sales | `o3p.sales.delivered_units` | O3P_DERIVED | SKU + Period |
| Sales | `o3p.sales.order_gross_value` | O3P_DERIVED | SKU + Period |
| Analytics | `ozon.analytics.revenue` | OZON_SOURCE | SKU + Day |
| Analytics | `ozon.analytics.ordered_units` | OZON_SOURCE | SKU + Day |
| Funnel | `o3p.funnel.view_to_cart_rate` | O3P_DERIVED | SKU + Period |
| Search | `ozon.search.view_conversion` | OZON_SOURCE | SKU + Period |
| Product | `ozon.product.content_rating` | OZON_SOURCE | SKU + Snapshot |
| Price | `o3p.price.discount_rate` | O3P_DERIVED | Product + Snapshot |
| Inventory | `o3p.inventory.sellable_stock` | O3P_DERIVED | SKU + Snapshot |
| Inventory | `o3p.inventory.in_transit_stock` | O3P_DERIVED | SKU + Snapshot |
| Inventory | `o3p.inventory.days_of_cover` | O3P_DERIVED | SKU + Snapshot |
| Inventory | `o3p.inventory.availability_rate` | O3P_DERIVED | SKU + Period |
| Inventory | `o3p.inventory.lost_sales_estimate` | O3P_ESTIMATE | SKU + Period |
| Replenishment | `o3p.inventory.recommended_replenishment_units` | O3P_FORECAST / ESTIMATE | SKU + Run |
| Fulfillment | `o3p.fulfillment.order_to_handover_duration` | O3P_DERIVED | Posting |
| Fulfillment | `o3p.fulfillment.delivery_transit_duration` | O3P_DERIVED | Posting |
| Fulfillment | `o3p.fulfillment.on_time_handover_rate` | O3P_DERIVED | Mode + Period |
| Fulfillment | `o3p.fulfillment.promise_delivery_hit_rate` | O3P_DERIVED | Mode + Period |
| Cancellation | `o3p.cancellation.pre_shipment_rate` | O3P_DERIVED | Shop + Created Cohort |
| Cancellation | `o3p.cancellation.posting_rate` | O3P_DERIVED | Shop + Eligible Order Cohort |
| Returns | `o3p.product.return_rate` | O3P_DERIVED | Product + Delivery Cohort |
| Buyout | `ozon.reference.buyout_rate_last_50_orders` | OZON_REFERENCE | Product |
| Buyout | `o3p.product.buyout_rate` | O3P_DERIVED | Product + Cohort |
| Finance | `o3p.finance.net_accrual` | O3P_DERIVED | Shop + Accrual Period |
| Finance | `o3p.finance.gross_sales` | O3P_DERIVED | Shop/SKU + Period |
| Finance | `o3p.finance.sale_commission_cost` | O3P_DERIVED | Shop/SKU + Period |
| Profit | `o3p.profit.shop_net_operating_profit` | O3P_DERIVED / ESTIMATE | Shop + Period |
| Profit | `o3p.profit.net_margin` | O3P_DERIVED / ESTIMATE | Shop + Period |
| Ads | `o3p.ads.ctr` | O3P_DERIVED | Campaign + Period |
| Ads | `o3p.ads.cpc` | O3P_DERIVED | Campaign + Period |
| Ads | `o3p.ads.drr` | O3P_DERIVED | Campaign + Period |
| Ads | `o3p.ads.roas` | O3P_DERIVED | Campaign + Period |
| VOC | `o3p.voc.question_answer_rate` | O3P_DERIVED | Shop + Period |
| Rating | `ozon.rating.current_value` | OZON_SOURCE | Shop + Snapshot |
| Data | `o3p.data.coverage_lag` | O3P_DERIVED | Source |
| Data | `o3p.data.mapping_coverage` | O3P_DERIVED | Domain |
| Forecast | `o3p.forecast.wape` | O3P_DERIVED | SKU Set + Horizon |
| Forecast | `o3p.forecast.bias` | O3P_DERIVED | SKU Set + Horizon |
| Forecast | `o3p.forecast.accuracy_score` | O3P_DERIVED | SKU Set + Horizon |

---

# 123. Phase 0 必须实现的 Metric

Data Foundation 和 Core Operations 首版至少需要：

```text
mother_order_count
posting_count
ordered_units
o3p.fulfillment.shipped_units
delivered_units
post_shipment_cancelled_units

created_posting_count
created_ordered_units
pre_shipment_cancellation_rate
post_shipment_cancellation_rate
cancellation_stage_coverage

order_gross_value
average_unit_price

analytics ordered_units
analytics revenue

sellable_stock
reserved_stock
in_transit_stock

calendar_ads_units
adjusted_ads_units
days_of_cover

order_to_handover_duration
delivery_transit_duration
order_to_delivery_duration
P50 / P90
on_time_handover_rate
promise_delivery_hit_rate

product_return_rate
product_buyout_rate

finance_net_accrual
finance_gross_sales
finance_sale_commission_cost
finance_unclassified_amount_ratio

procurement_cost_coverage
seller_logistics_coverage

shop_net_operating_profit
net_profit_margin
profit coverage dimensions

ads views
ads clicks
ads spend
ads orders_money
CTR
CPC
DRR
ROAS

question_count
question_answer_rate

rating source metrics

data freshness
coverage lag
mapping coverage
reconciliation status
```

---

# 124. Phase 1 / 2 扩展

进一步增加：

```text
availability_rate
lost_sales_estimate
replenishment_estimate

WHD resale metrics
return financial impact

promotion lift
price-profit simulation
minimum profitable price

settlement / payout reconciliation

campaign budget utilization
ad revenue share
ad spend share
```

---

# 125. Phase 3 / 4 扩展

Decision Support：

```text
forecast demand
forecast WAPE
forecast Bias
forecast MAE / RMSE
projected stockout date
projected days of cover
forecast-based replenishment
recommendation priority
```

---

# 126. 当前明确不定义的指标

以下内容 v1.0 不建立正式 Metric Contract：

- Performance Phrases ROI；
- Phrase Conversion；
- Review Detail 指标；
- FBS Error Posting 原因分布；
- Analytics Stocks Item 指标；
- Webhook End-to-End Latency SLA；
- WHD 精确二次销售回收率；
- Compensation 非零场景完整 Finance Mapping；
- 未经验证的竞争对手销量；
- 未经验证的竞争对手库存。

原因：

当前 Source Contract 不足。

---

# 127. 当前官方口径冲突处理

当 Ozon 不同页面存在不同时间窗口或相关指标时：

禁止自动合并。

例如当前资料同时存在：

```text
Fulfillment Report Cancellation
→ 最近 7 天

Service Quality Seller-responsible Cancellation
→ 最近 14 天，不含当天
```

O3Pilot 分别建 Metric Code。

只有完成来源和分子 / 分母对账后，才能说明两者关系。

---

# 128. Metric Version 规则

只要以下任一项改变：

- Numerator；
- Denominator；
- Eligibility；
- Exclusion；
- Time Basis；
- Return Status Mapping；
- Finance Classification；
- FX Policy；
- Allocation；
- Cost Fallback；

必须提升 `metric_version`。

UI 名称可以不变。

历史 Result 必须保留旧版本。

---

# 129. Metric Code 稳定性

Metric Code 一旦作为数据库或 API Contract 发布：

不得因为 UI 中文改名而改 Code。

例如：

```text
o3p.product.return_rate
```

即使 UI 后来显示：

```text
商品退货率
实际退货率
历史退货率
```

基础 Code 仍保持稳定，通过 Variant / Dimension 表达。

---

# 130. 派生数据不可反写 Source Fact

禁止：

```text
O3Pilot Return Rate
→ 覆盖 Ozon Return Metric
```

禁止：

```text
O3Pilot Profit
→ 覆盖 Finance Accrual
```

禁止：

```text
Estimated Logistics Cost
→ 覆盖 Seller Logistics Import
```

---

# 131. 指标验收原则

在开发正式 Dashboard 前，Metric Engine 至少满足：

1. Mother Order 与 Posting 指标分离；
2. Source Metric 与 Derived Metric 分离；
3. Ozon Reference Metric 单独标记；
4. Ratio 使用分子 / 分母聚合；
5. Denominator 0 返回 N/A；
6. Missing 不等于 0；
7. Money 不跨币种直接求和；
8. 每次 FX 转换保存 Rate Basis；
9. Analytics 与 Order Fact 不静默互相覆盖；
10. Product Weight 不进入库存件数计算；
11. Sellable Stock 与 Reserved 分离；
12. Current Stock 与 In-transit 分离；
13. Days of Cover 不在零销量时返回伪造大数；
14. Lost Sales 明确标记 Estimate；
15. Ozon Lost Sales Reference 与 O3Pilot Estimate 分离；
16. Ozon 补货阈值不成为 O3Pilot 永久阈值；
17. 发货 / 配送 Duration 使用真实节点；
18. P50 / P90 从原始样本计算；
19. 时效指标展示 Sample Count；
20. 经营结果类订单指标默认先排除已确认发货前取消；
21. 待发货但未取消订单不得因为尚未发货而从 Sales / AOV / ADS / 默认取消率 / Forecast Actual 中删除；
22. Created Demand 与 Eligible Sales 使用不同 Metric Code；
23. Ozon Source / Reference Metric 不被强制套用 O3Pilot 的发货前取消过滤；
24. Finance / Settlement / Payout 保留发货前取消产生的真实 Money Fact；
25. Ozon 取消率与 O3Pilot 取消率分离；
26. 产品退货率使用 Cohort；
27. 近期退货 Cohort 可以保持 OPEN；
28. 取消、未认购、客户退货不混为一个 Return Type；
29. Ozon 最近 50 单认购率与 O3Pilot Buyout Rate 分离；
30. Finance v1 是新财务 Metric 基础；
31. Type 69 Gross Sales 使用已验证公式；
32. Finance Classification 版本化；
33. Unclassified Finance 可量化；
34. Profit 不重复扣除 Performance Advertising；
35. Finance Advertising 与 Performance Attribution 分离；
36. 历史订单优先使用实际采购成本；
37. Profit 显示独立 Coverage，不伪造单一可信度；
38. Settlement / Payout 不由 Accrual 猜测；
39. Advertising CTR / CPC / DRR 聚合时重新计算；
40. Campaign Ratio 不直接平均；
41. Performance SKU Daily 缺采时不能用未来数据伪造；
42. Review 无权限显示 N/A；
43. Shop Rating 保留 Ozon Source Value；
44. 跨店 Product 必须通过 Seller Catalog Mapping；
45. Forecast Accuracy 按 Horizon 分开；
46. WAPE 作为总体 Forecast 主误差；
47. Forecast Bias 必须同时可见；
48. 预测值、估算值、实际值明确区分；
49. Alert 不重新定义 Metric；
50. 所有 Metric Result 可追溯到 Metric Version 和 Source Coverage。
---

# 132. 与 DATA_SOURCES.md 的关系

`DATA_SOURCES.md` 定义：

```text
数据从哪里来
```

`METRICS.md` 只能使用：

- DATA_SOURCES 已验证自动来源；
- 明确 MANUAL 来源；
- 明确 PARTIAL 并带状态的来源。

不能为了公式完整而发明不存在的数据。

---

# 133. 与 DATA_MODEL.md 的关系

`DATA_MODEL.md` 定义：

```text
Fact 如何保存
```

`METRICS.md` 定义：

```text
Fact 如何计算成 Metric
```

例如：

```text
ReturnCase
ReturnItem
PostingItem
```

是数据模型。

```text
Product Return Rate
```

是指标。

---

# 134. 与 PRODUCT.md 的关系

`PRODUCT.md` 定义产品能力。

`METRICS.md` 提供这些能力可执行的量化 Contract。

例如：

```text
PRODUCT:
库存风险与补货建议

METRICS:
Days of Cover
Lost Sales Estimate
Forecast Demand
Recommended Replenishment Units
```

---

# 135. 与 ARCHITECTURE.md 的关系

本文件不规定：

```text
每天几点同步
库存每几分钟拉一次
Worker 数量
Queue
Cron
```

但 ARCHITECTURE 必须保证：

Metric Contract 所需的历史粒度能够被采集。

尤其是：

- Inventory Snapshot；
- Campaign SKU Daily；
- Schedule History；
- FX Interval；
- Seller Cost History。

---

# 136. 核心指标原则

**一个指标只有一个正式口径。**

**官方指标、内部计算和估算必须分开。**

**分子和分母比显示名称更重要。**

**Ratio 跨维度聚合时重新计算，不平均 Ratio。**

**Money 不跨币种直接相加。**

**原始金额永远不被 Reporting Currency 覆盖。**

**缺失不等于 0。**

**Denominator 为 0 时返回 N/A。**

**订单、母订单和商品件数是不同计数单位。**

**经营结果类订单指标默认只排除已确认发货前取消订单。**

**待发货但未取消订单仍是有效订单；只有已确认发货前取消才从默认经营统计中排除。**

**Created Demand 与 Eligible Sales 必须分开。**

**真实 Finance Fact 不因订单发货前取消而删除。**

**Ozon Analytics 与订单事实是两个来源口径。**

**当前库存与在途库存分开。**

**Sellable Stock 与 Reserved 分开。**

**Product Weight、Package Weight 与 Chargeable Weight 分开。**

**Ozon Reference 补货规则不等于 O3Pilot 永久算法。**

**产品退货率使用可追溯 Cohort。**

**认购率必须区分 Ozon 最近 50 单 Reference 与 O3Pilot 自算。**

**Finance 是财务事实，Performance 是广告归因事实。**

**同一笔广告费用只能扣一次。**

**历史利润优先使用历史实际采购成本。**

**利润必须展示数据覆盖情况。**

**Forecast 必须接受历史回测。**

**WAPE 和 Bias 比一个漂亮的“准确率”更重要。**

**指标变化必须版本化。**

**O3Pilot 永远只读 Ozon。**


---

# 137. 本版参考基线

本版 Metric Contract 以 2026-09-03 当前 O3Pilot 基线和开发参考资料为基础，主要包括：

```text
PRODUCT.md v1.0
DATA_SOURCES.md v1.0
DATA_MODEL.md v1.0

Ozon Seller API 当前实测资料
Ozon Performance API 当前实测资料
Finance Accrual v1 新旧对账资料

已认购商品.md
商品可售性和错失的销售.md
服务质量指标.md
报告.md
```

使用规则：

1. API 实测只证明当前真实字段与行为；
2. Ozon 知识库用于解释官方业务口径；
3. 如果 API Source Metric 与知识库 Reference Metric 不是同一个 Contract，分别建指标；
4. 如果两个官方页面存在不同窗口或分子分母，分别保存，不强行统一；
5. 当前无法验证的公式保持 `UNVERIFIED`，不补全猜测。

---

# 138. 当前跨文档同步提示

当前 `DATA_SOURCES.md v1.0` 已正式加入：

```text
Ozon Exchange Rate API
Seller Cost FX
马帮 ERP Cost Adapter
Seller Logistics
In-transit Inventory
Settlement / Payout
```

`METRICS.md v1.0` 已按照这些来源定义对应 Metric Contract。

如果后续 `PRODUCT.md` 的“运行时数据源”列表仍未显式列出 `Ozon Exchange Rate API`，应在下一次 PRODUCT 基线复核时同步文字描述；这不改变当前 Metric 与 Data Source Contract。
