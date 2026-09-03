# O3Pilot — DATA_SOURCES.md

> Version: 1.0  
> Status: Data Source Contract Baseline  
> Updated: 2026-09-03  
> Applies to: O3Pilot

# 1. 文档目的

`DATA_SOURCES.md` 定义 O3Pilot 在运行时可以使用的数据来源，以及每类数据的获取方式、验证状态、数据粒度、历史回溯能力、同步要求和已知边界。

本文件回答：

- O3Pilot 的数据从哪里来；
- 哪些数据可以自动获取；
- 哪些数据需要人工导入；
- 哪些数据已经通过真实店铺验证；
- 哪些数据只完成部分验证；
- 哪些数据受订阅、权限或时间窗口限制；
- 哪些接口已经废弃，不能继续成为运行时依赖；
- 不同数据源发生差异时应该如何处理；
- 哪些数据必须持续采集，否则未来无法补回。

本文件不定义：

- 业务实体最终数据库结构；
- 指标公式；
- 利润计算公式；
- 产品退货率公式；
- 页面或导航；
- 具体任务调度时间；
- 技术服务拆分；
- 数据库选型。

这些内容分别由 `DATA_MODEL.md`、`METRICS.md`、`ARCHITECTURE.md` 等文档定义。

---

# 2. 数据源范围

O3Pilot 当前运行时数据源分为六类：

1. Ozon Seller API；
2. Ozon Performance API；
3. Ozon Exchange Rate API；
4. Ozon Push Webhook；
5. Ozon 官方报告；
6. 卖家自有数据。

Google Drive、开发参考资料、历史 O3Pilot 代码、API 测试报告等不属于 O3Pilot 运行时数据源。

它们只用于开发和验证，不参与产品正常运行。

---

# 3. 数据源验证状态

为避免把“接口存在”误写成“数据已经可用”，DATA_SOURCES 使用以下验证状态。

| 状态 | 含义 |
|---|---|
| `VERIFIED` | 已在真实店铺实际请求，并获得足以确认主要字段、粒度或业务用途的结果 |
| `PARTIAL` | 接口已实际请求，但关键非空样本、完整粒度、分页、历史范围或部分字段仍未验证 |
| `ACCESS_DEPENDENT` | 数据源存在，但当前店铺受订阅或权限限制，不能正常读取 |
| `DOCUMENTED` | 官方资料支持该能力存在，但当前 O3Pilot 验证基线尚未进行真实数据验证 |
| `MANUAL` | 需要用户主动导出、上传或维护 |
| `DEPRECATED` | 已废弃或已明确进入关闭流程，只允许迁移、历史核验，不得作为新运行时依赖 |
| `PROHIBITED` | 即使接口存在，因 O3Pilot Read-Only 产品边界禁止调用 |

验证状态描述的是“当前证据强度”，不是接口永久能力。

Ozon 修改接口、订阅规则或业务结构后，状态必须重新验证。

---

# 4. 数据来源基本原则

## 4.1 原始数据优先

O3Pilot 应尽可能保留 Ozon 原始响应。

标准化字段、业务分类和派生指标不能替代原始事实。

当后续发现字段理解错误、API 语义变化或需要重新计算历史指标时，应能够从原始数据重新处理。

---

## 4.2 数据来源必须可追溯

进入 O3Pilot 的重要数据至少需要能够追溯：

- `shop_id`；
- 数据源类型；
- API / Report / Seller Data 类型；
- 原始对象标识；
- 获取时间；
- 原始业务时间；
- 数据覆盖时间范围；
- 同步批次；
- 原始响应或原始导入文件；
- 解析器或标准化规则版本。

最终字段名称由 `DATA_MODEL.md` 定义。

---

## 4.3 不跨来源静默覆盖

如果两个数据源对同一个业务事实给出不同结果：

- 两份原始事实都必须保留；
- 必须记录来源；
- 必须允许产生数据差异告警；
- 不允许静默使用其中一份覆盖另一份。

只有明确的数据契约才能指定某个来源作为某个字段的主来源。

---

## 4.4 未返回不等于 0

以下情况必须区分：

- API 返回 `0`；
- API 返回 `null`；
- 字段整体缺失；
- 对象被省略；
- 当前店铺无权限；
- 查询无数据；
- 同步失败；
- 数据尚未验证。

任何一种情况都不能自动转换成 `0`。

---

## 4.5 POST 不等于写操作

Ozon Seller API 大量只读接口使用 HTTP `POST`。

因此 O3Pilot 不能根据 HTTP Method 判断接口是否安全。

O3Pilot 必须维护显式只读接口 allowlist。

所有未进入 allowlist 的 Ozon 接口默认禁止调用。

---

# 5. 数据源优先关系

不同业务域使用不同主来源，不建立一个简单的“所有数据统一优先级”。

| 业务事实 | 主来源 | 辅助 / 核验来源 |
|---|---|---|
| 店铺主体、订阅、权限 | Seller API | — |
| 商品目录 | Seller API | Ozon 官方报告 |
| 商品属性与内容 | Seller API | Ozon 官方报告 |
| 当前价格 | Seller API | Ozon 官方报告 |
| 当前库存 | Seller API | Ozon 官方报告 |
| 订单事实 | Seller API | Webhook、Ozon 官方报告 |
| 当前订单变化 | Webhook + Seller API | — |
| 退货与逆向物流 | Seller API | Ozon 官方报告 |
| 财务应计 | Finance Accrual v1 | Ozon 官方财务报告 |
| 经营 Analytics | Seller API Analytics | 订单事实用于核验 |
| 广告活动与广告统计 | Performance API | Finance 中部分广告计费可用于财务侧核验 |
| 店铺 Rating | Seller API Rating | Seller Info 中部分指标 |
| Questions / Answers | Seller API | — |
| Reviews | Seller API，受订阅权限影响 | — |
| Ozon 业务汇率 | Ozon Exchange Rate API | Ozon 官方财务规则 / 报告核验 |
| 商品采购成本 | 卖家自有数据 / ERP | — |
| 卖家实际物流成本 | 卖家自有数据 / 物流商导出 | Ozon 订单用于关联 |
| 包裹实际 / 体积 / 计费重量 | 物流商 / Seller Data | Ozon 订单和商品属性用于核验 |
| 在途库存 | 卖家自有数据 / Ozon 可获得交货事实 | Ozon 官方报告 |
| Settlement / Payout | Ozon 官方财务报告 | Finance Accrual 用于对账 |
| 最终利润 | 多来源派生 | Finance + Seller Data + Advertising + FX |

“主来源”只表示该业务事实的首选运行时来源，不代表其数据永远没有错误。

---

# 6. Seller API

## 6.1 定位

Seller API 是 O3Pilot 日常经营数据的主要自动化来源。

当前已验证的数据域包括：

- 店铺与权限；
- 商品目录；
- 商品详情与属性；
- 价格与佣金；
- 当前库存；
- 仓级库存；
- 订单与履约；
- 退货与售后；
- Finance；
- Analytics；
- Questions / Answers；
- Rating；
- Webhook 配置与 Push Type；
- 商品内容质量。

Reviews 当前受到店铺订阅权限限制。

---

## 6.2 认证

Seller API 使用：

- Client ID；
- API Key。

API Key 具有角色和授权方法集合。

O3Pilot 必须在运行时检查凭证是否可用，但不能因为 API Key 拥有 Admin 或其他高权限角色，就扩大 O3Pilot 自身的只读权限。

凭证能够访问写接口不代表 O3Pilot 可以调用写接口。

---

# 7. 店铺与权限

## 7.1 `/v1/seller/info`

状态：`VERIFIED`

用途：

- 店铺名称及主体信息；
- 注册国家；
- 结算货币；
- 订阅类型；
- 部分店铺 Rating 当前值。

数据性质：

- 当前状态快照；
- 店铺级。

同步要求：

- 不需要高频同步；
- 应保存历史快照，以观察结算币种、订阅状态等配置变化。

注意：

店铺结算货币不能用于推断其他 API 金额的币种。

---

## 7.2 `/v1/roles`

状态：`VERIFIED`

用途：

- API Key 角色；
- 授权方法列表；
- 凭证有效期。

用途边界：

该接口用于检查“凭证允许什么”。

O3Pilot 自己允许调用什么，仍由 O3Pilot 的只读 allowlist 决定。

---

# 8. 商品主数据

## 8.1 `/v3/product/list`

状态：`VERIFIED`

定位：

商品目录枚举的主要来源。

已验证可以获得：

- `product_id`；
- `offer_id`；
- `sku`；
- 归档状态；
- FBO / FBS 库存存在性相关标记。

当前验证中，商品目录主标识集合能够与商品详情批量接口对应。

数据性质：

- 当前目录快照。

历史要求：

该接口不是历史商品状态仓库。

如果 O3Pilot 需要商品生命周期历史，应周期性保存快照。

---

## 8.2 `/v3/product/info/list`

状态：`VERIFIED`

定位：

商品主数据批量详情的主要来源。

当前已验证可以获得：

- 商品名称；
- 商品状态；
- 审核错误；
- 类目；
- 商品类型；
- 条形码；
- 图片；
- 主图；
- 变体模型；
- availability；
- 价格和佣金摘要；
- 库存摘要；
- 商品来源相关信息。

已知边界：

- `availability` 不能等同于“当前有库存”；
- 审核状态不能与实时库存合并成一个简单的“可售”布尔值；
- `primary_image` 与 `images[]` 必须分别保存；
- 变体 `model_info.count` 不能直接解释为当前店铺实际成员数量。

---

## 8.3 `/v4/product/info/attributes`

状态：`VERIFIED`

用途：

- 商品完整属性；
- 类目和类型；
- 尺寸；
- 重量；
- 图片；
- 条形码；
- 类目属性；
- complex attributes；
- Rich Content 等属性数据。

数据性质：

当前商品内容快照。

历史要求：

如需要追踪商品内容变化，O3Pilot 自行建立版本历史。

---

## 8.4 商品标识来源规则

Seller API 中同时存在：

- `product_id`；
- `sku`；
- `offer_id`。

O3Pilot 必须保留三者。

当前实测表明三者都可以稳定用于跨接口关联，但 `offer_id` 可能发生卖家侧改名，因此不能仅依赖 `offer_id` 建立永久主键。

最终实体主键规则由 `DATA_MODEL.md` 定义。

---

# 9. 商品内容质量

## 9.1 `/v1/product/rating-by-sku`

状态：`VERIFIED`

用途：

- 商品卡片总评分；
- Media 评分；
- 文本评分；
- 其他属性评分；
- 已满足评分条件；
- 待补充属性；
- 可改进项。

适合支持：

- 商品内容诊断；
- 属性完整度分析；
- 商品优化建议。

限制：

评分规则属于 Ozon 当前规则，不应由 O3Pilot 固化为永久规则。

应保存 API 返回的原始评分条件，而不是只保存最终总分。

---

# 10. 价格、佣金与促销

## 10.1 `/v5/product/info/prices`

状态：`VERIFIED`

当前已验证可以获得：

- 当前卖家价格；
- Marketing Seller Price；
- Old Price；
- Minimum Price；
- 价格币种；
- Acquiring；
- 不同技术履约类型对应的佣金和费用字段；
- 营销活动信息；
- Price Index；
- 外部 / Ozon / 自有平台比价数据；
- Volume Weight。

数据性质：

当前价格快照。

历史要求：

价格历史需要由 O3Pilot 周期采集形成。

关键规则：

- 每个显式 Money / Currency 字段按实际返回值保存；
- 没有 currency 的费用字段不推断币种；
- 商品价格接口中的渠道佣金率与真实订单 Finance 佣金不是同一个口径；
- Price Index 缺失不能记为 0；
- Marketing Actions 只能解释为 API 当前返回的活动信息，不能自行推断活动来源。

---

# 11. 当前库存

## 11.1 `/v4/product/info/stocks`

状态：`VERIFIED`

定位：

O3Pilot 当前商品库存的主要自动化来源。

当前已验证字段包括：

- `product_id`；
- `offer_id`；
- `sku`；
- `type`；
- `present`；
- `reserved`；
- `shipment_type`；
- `warehouse_ids`。

当前实测技术库存类型包括：

- `fbo`；
- `rfbs`；
- `fbp`。

注意：

这些是 API 技术字段。

最终 FBP / realFBS / OGL 等业务分类由 `DATA_MODEL.md` 的标准化规则定义。

历史要求：

接口提供当前库存。

库存历史由 O3Pilot 周期保存快照建立。

---

## 11.2 `/v1/product/info/stocks-by-warehouse/fbo`

状态：`VERIFIED`

用途：

FBO 库存下钻到 `warehouse_id` 粒度。

已验证：

- SKU / offer_id；
- warehouse_id；
- present；
- reserved；
- cursor 分页。

已观察到真实 `429` 后重试成功。

因此调用方必须支持限流错误的安全重试，而不能把 429 当成“没有库存”。

该接口只负责其自身 FBO 仓级覆盖，不替代全渠道 `/v4/product/info/stocks`。

---

## 11.3 `/v1/analytics/stocks`

状态：`PARTIAL`

已验证：

- API 可访问；
- 请求需要 1～100 个 SKU；
- 两个当前验证店铺均返回 200。

但：

即使使用真实 `fbo present > 0` SKU，当前实测仍然返回：

```json
{"items":[]}
```

因此当前不能确认：

- 非空 `items` 的真实结构；
- 仓库 / Cluster 字段；
- 适用履约模式；
- 空集合原因。

结论：

该接口不能作为 O3Pilot 当前库存主来源。

只有取得非空真实样本并重新验证后，才能升级状态。

---

# 12. 库存周转

## 12.1 `/v1/analytics/turnover/stocks`

状态：`VERIFIED`

当前验证可以取得 SKU 级库存周转数据。

已确认的核心业务信号包括：

- 平均每日销售；
- 当前库存；
- 库存可支撑天数；
- 库存 / 周转等级。

当前实测中：

- `ads` 表示过去 60 天平均每日售出件数；
- `idc` 表示按平均日销估算的库存可支撑天数。

规则：

这些字段属于 Ozon Analytics 指标。

如果 O3Pilot 后续建立自己的库存覆盖、补货或销量预测算法，应与 Ozon 原始 Analytics 指标分开保存。

---

# 13. 订单与履约

## 13.1 `/v4/posting/fbs/list`

状态：`VERIFIED`

用途：

FBS 技术订单列表，也是当前 FBP 等 Global 履约订单的重要来源之一。

当前已验证可以获得：

- `order_number`；
- `posting_number`；
- order_id；
- 商品；
- SKU；
- offer_id；
- 数量；
- 商品价格与币种；
- status / substatus；
- cancellation；
- delivery method；
- warehouse；
- provider；
- tracking number；
- analytics data；
- financial data；
- 创建 / 处理 / 发货 / 配送相关时间；
- `delivery_schema`；
- `integration_type_flow`。

关键证据：

当前真实样本中可以出现：

```text
delivery_schema = fbs
integration_type_flow = FBP
```

因此：

接口名 `fbs` 不能直接等同于业务层“FBS 履约模式”。

O3Pilot 必须保存原始技术字段，再由业务映射规则确定最终履约分类。

分页：

cursor + `has_next`。

历史：

支持时间范围查询；当前验证覆盖多日区间。

具体最大历史范围不得依据当前测试自行外推。

---

## 13.2 `/v3/posting/fbo/list`

状态：`VERIFIED`

用途：

FBO 技术订单列表。

当前已验证：

- 母订单 / 订单；
- 商品与数量；
- 状态；
- 创建和处理时间；
- 仓库；
- Analytics；
- Financial Data。

边界：

部分实际妥投时间在列表接口中可能不完整，需要 Detail 补充。

---

## 13.3 `/v2/posting/fbo/get`

状态：`VERIFIED`

用途：

补充单个 FBO Posting 的详细字段。

当前验证中，已签收样本可以获得非空 `fact_delivery_date`。

因此对于需要精确配送时效的 FBO 订单：

FBO List 负责发现和基础事实，FBO Detail 用于补足必要时间节点。

---

## 13.4 `/v1/posting/fbp/get`

状态：`VERIFIED`，接口为 Beta

用途：

读取 FBP 原生 Posting Detail。

当前已验证：

- FBP Posting 详情；
- 状态；
- 价格；
- 佣金；
- 促销等结构化信息。

规则：

Beta 接口必须保存原始响应，并接受字段或 Contract 未来发生变化的可能。

不能让 Beta 接口成为唯一不可替代的核心订单事实来源。

---

## 13.5 发货截止时间与承诺配送窗口

订单来源中已经确认存在多种不同时间语义，包括：

- 订单创建时间；
- 进入处理时间；
- 发运日期；
- 不逾期发运日期 / 发货截止时间；
- 实际转移配送时间；
- 承诺配送开始 / 结束时间；
- 实际配送时间；
- 取消时间。

这些字段不得压缩成单一 `shipment_date` 或 `delivery_date`。

如果 Ozon 后续通过 API 或 Webhook 修改发货截止时间或承诺配送窗口：

- 当前值应更新；
- 历史值必须保留；
- 具体历史实体由 `DATA_MODEL.md` 定义。

这类数据用于时效指标，但指标公式进入 `METRICS.md`。

---

## 13.6 仓库、物流商、配送方式与包裹

当前订单来源可以提供：

- Warehouse；
- Logistics Provider；
- Delivery Method；
- Tracking Number；
- 部分商品重量；
- 部分 Posting Volume Weight；
- Region / City；
- Cluster From / Cluster To。

但 O3Pilot 必须区分：

```text
Product Weight
Package Actual Weight
Package Volumetric Weight
Package Chargeable Weight
```

商品属性接口中的重量 / 尺寸不能自动作为物流实际计费重量。

订单 API 的 `volume_weight` 也不能在没有来源语义确认时自动解释为所有物流商最终账单计费重量。

物流商实际账单或导出数据属于 Seller-Owned Data。

---

# 14. 取消

取消数据主要来自：

- FBS / FBO Posting；
- Analytics；
- Rating；
- Ozon 官方质量报告。

Seller API 订单中能够获得取消状态和部分取消原因、发起方、是否影响 Rating 等信息。

`/v1/analytics/data` 可提供聚合的 `cancellations` 指标。

原则：

订单级取消事实优先来自订单 / Posting。

Analytics 的取消指标用于经营聚合与交叉验证。

具体取消率公式在 `METRICS.md` 定义。

---

# 15. 退货与逆向物流

## 15.1 `/v2/returns/rfbs/list`

状态：`VERIFIED`

用途：

rFBS 退货申请列表。

当前已验证：

- return_id；
- return_number；
- order_number；
- posting_number；
- 商品；
- SKU；
- offer_id；
- 商品价格和币种；
- 当前退货状态；
- 创建时间。

分页：

使用 `last_id`。

---

## 15.2 `/v2/returns/rfbs/get`

状态：`VERIFIED`

用途：

rFBS 单条退货详情。

当前真实样本已经验证：

- 退货原因；
- 缺陷标记；
- 买家评论；
- 买家图片；
- 退回方式；
- 部分逆向物流跟踪号；
- warehouse_id。

边界：

- 部分字段并非所有样本都存在；
- `client_name` 在当前样本中没有出现，但不能据此判定字段永久不存在；
- Detail 中 `order_number` 可能为空，因此不能用 Detail 替代 List 的所有关联事实。

建议：

`return_id` 与 `posting_number` 应保留为重要关联标识。

---

## 15.3 `/v1/returns/list`

状态：`VERIFIED`

用途：

平台综合退货、取消件、未提件和逆向物流信息。

当前可以获得：

- return id；
- order / posting；
- return reason；
- type；
- schema；
- WHD place / target place；
- storage；
- product；
- quantity；
- return / final moment；
- barcode；
- visual status；
- compensation status 等。

当前真实样本存在：

```text
schema = Whd
```

因此 WHD 应作为逆向物流来源事实保存，而不是自动当成正常首次销售履约方式。

---

## 15.4 产品退货率的数据来源

产品退货率是派生指标，不是任何单一接口的直接字段。

数据基础可能组合：

- 订单 / 销售事实；
- rFBS 退货；
- 综合退货；
- Analytics 中的退货指标。

具体分子、分母、归属日期和跨期逻辑由 `METRICS.md` 定义。

DATA_SOURCES 只负责保证底层订单与退货事实可以追溯。

---

## 15.5 逆向物流生命周期

Ozon 当前业务规则中，取消、未认购和客户退货可能继续进入：

- 逆向运输；
- WHD 入库；
- 仓储；
- 减价；
- 再销售；
- 退回卖家；
- 销毁；
- 赔偿。

当前 Seller API / Returns 数据可以提供其中部分事实，但并不能保证每个阶段都有完整事件。

因此：

- 已取得的阶段事实进入 Normalized Fact；
- 未取得的中间阶段不得根据最终状态反推；
- Ozon 官方报告可以作为人工补充 / 对账来源；
- 生命周期实体由 `DATA_MODEL.md` 定义。

---

# 16. Finance

## 16.1 新运行时 Finance 主链路

O3Pilot 新系统 Finance 主链路固定为：

- `/v1/finance/accrual/types`；
- `/v1/finance/accrual/by-day`；
- `/v1/finance/accrual/postings`。

旧 Finance v3 不再作为新系统运行时依赖。

---

## 16.2 `/v1/finance/accrual/types`

状态：`VERIFIED`

用途：

Finance Accrual 类型字典。

当前验证店铺均返回相同的 124 种官方类型定义。

每个类型包含：

- id；
- name；
- description。

原则：

必须保存官方 `type_id`。

O3Pilot 的中文费用分类只能作为标准化层，不能替换 Ozon 原始 type。

---

## 16.3 `/v1/finance/accrual/by-day`

状态：`VERIFIED`，分页边界部分待验证

用途：

按单日获取 Finance Accrual。

当前已验证三类结构：

- `ITEM`；
- `POSTING`；
- `NON_ITEM`。

核心字段包括：

- accrual_id；
- date；
- total_amount；
- currency；
- unit_number；
- accrued_category；
- posting；
- item_fees；
- non_item_fee；
- container_fees。

历史回溯：

按日期逐日拉取。

分页：

接口存在 `last_id`。

当前测试期每天第一页 `last_id` 都为空，因此：

- 首页行为已验证；
- 非空 `last_id` 的真实第二页尚未物理触发。

实现仍必须保留游标续页能力。

---

## 16.4 `/v1/finance/accrual/postings`

状态：`VERIFIED`

用途：

按 Posting 获取应计明细。

已验证粒度差异：

### Type 69 — SaleCommission

当前多 SKU 实测中：

- 携带 SKU；
- 携带 quantity；
- 携带 seller_price。

可以直接形成 SKU 级财务事实。

### Type 66 — RfbsGlobalAgentFee

当前多 SKU rFBS 实测中：

- Posting 级；
- 不携带 SKU；
- 不携带 quantity；
- 不携带 seller_price。

### Type 67 — RfbsGlobalDelivery

当前多 SKU rFBS 实测中：

- Posting 级；
- 不携带 SKU；
- 不携带 quantity；
- 不携带 seller_price。

因此：

Type 66 / 67 不能静默归属某个 SKU。

SKU 利润需要由 `METRICS.md` 定义明确的 Posting → SKU 分摊规则。

### Type 29 / 32

当前仅有单 SKU 样本。

多 SKU 粒度尚未验证，不得外推。

---

## 16.5 Finance 币种

当前测试账期中，Finance Accrual v1 所观察到的 Money.currency 均为 RUB。

但这只是当前真实样本。

O3Pilot 必须：

- 保存 API 实际返回的 currency；
- 不使用 Shop settlement currency 替代 Finance currency；
- 不硬编码 Finance 永远为 RUB。

---

## 16.6 Finance v3

以下接口：

- `/v3/finance/transaction/totals`；
- `/v3/finance/transaction/list`。

状态：`DEPRECATED`

Ozon 公告关闭日期：

2026-09-08。

用途仅限：

- 历史迁移；
- 新旧 Finance 对账；
- 已有历史数据解释。

新 O3Pilot 不得建立新的持续运行依赖。

当前验证已经证明，在既有测试账期内，新 Accrual v1 可以与旧 v3 主流水完成记录 ID 与金额级对账。

---

## 16.7 Finance 当前待验证边界

仍需保留：

- `compensation_amount` 非零场景映射未验证；
- `/by-day` 非空 `last_id` 第二页未验证；
- Type 29 / 32 多 SKU 粒度未验证；
- `container_fees` 当前缺少非空真实样本。

这些未知项不能通过推测补齐。

---

## 16.8 Settlement / Payout

Ozon Finance Accrual 与卖家实际收到的结算付款不是同一层事实。

当前已确认 Seller Center 财务体系存在：

- 结算周期；
- 付款；
- 计划 / 实际付款状态；
- 相互结算报告；
- 外币支付报告；
- 对账单；
- 修正文件。

当前没有在本基线中验证一个可以完整替代这些报告的稳定只读 Seller API Contract。

因此首版定义：

```text
Finance Accrual = 自动主链路
Settlement / Payout = Ozon 官方财务报告 MANUAL + Accrual Reconciliation
```

不得根据 Accrual 金额自行推断：

```text
已付款
实际付款日期
实际到账金额
```

这些必须来自明确来源。

---

# 17. Analytics

## 17.1 `/v1/analytics/data`

状态：`VERIFIED`

定位：

O3Pilot 日级经营 Analytics 的主要来源。

当前已验证：

- `sku + day` 粒度；
- Revenue；
- Ordered Units；
- Search Views；
- PDP Views；
- Total Views；
- Add-to-cart；
- Sessions；
- Returns；
- Cancellations；
- Delivered Units。

分页：

`limit + offset`。

当前验证使用 `limit=1000` 并完成多页拉取。

关键解析规则：

响应：

```text
data[].metrics[i]
```

与“API 实际接受的 metrics 顺序”对应。

如果请求中包含未知 metric，API 可能静默忽略该 metric 并缩短返回数组，而不是返回 400。

因此实现必须：

- 只使用已验证 metric allowlist；
- 校验响应 metric 数量；
- 禁止盲目根据原请求下标绑定字段。

历史：

支持指定日期区间。

当前测试中观察到同日数据，但不能将其定义成永久实时 SLA。

---

## 17.2 `/v1/analytics/product-queries`

状态：`VERIFIED`

定位：

SKU 级搜索表现汇总。

已验证：

- SKU；
- 搜索用户；
- 浏览用户；
- 平均搜索位置；
- 浏览转化；
- GMV；
- Currency；
- 商品信息。

请求要求：

- `date_from` / `date_to` 使用 ISO 8601 / RFC3339 Timestamp；
- `skus` 必须非空；
- page 从 `0` 开始。

无数据 SKU：

可能从结果中直接省略，而不是返回全 0 对象。

因此：

“对象缺失”不能解释成“所有指标等于 0”。

当前测试观察到最新可用搜索数据为 T-1，但不能固化成永久 SLA。

---

## 17.3 `/v1/analytics/product-queries/details`

状态：`VERIFIED`

定位：

搜索词级分析。

最细已验证粒度：

```text
SKU + Query + Analytics Period
```

可以获得：

- query；
- SKU；
- Search Users；
- View Users；
- Position；
- Conversion；
- Order Count；
- GMV；
- Currency。

分页从 `page=0` 开始。

当前同样观察到搜索数据存在一定计算延迟，但具体 SLA 以实际返回为准。

---

## 17.4 `/v1/analytics/turnover/stocks`

状态：`VERIFIED`

详见库存周转章节。

---

## 17.5 `/v1/analytics/stocks`

状态：`PARTIAL`

详见当前库存章节。

在取得真实非空样本前，不作为正式库存依赖。

---

# 18. Performance API

## 18.1 定位

Performance API 是 O3Pilot 广告活动和广告效果数据的主要自动化来源。

Seller API 凭证与 Performance API 凭证必须分开管理。

---

## 18.2 Host

当前使用：

```text
https://api-performance.ozon.ru
```

旧 Host：

```text
performance.ozon.ru
```

已停止使用，不得作为新系统配置。

---

## 18.3 认证

Token：

```text
POST /api/client/token
```

使用 Client Credentials。

当前实测 Token 响应包含：

- token_type；
- access_token；
- expires_in。

当前样本 `expires_in = 1800` 秒。

实现应根据实际返回的过期时间管理 Token，不硬编码固定生命周期。

---

## 18.4 请求额度

Performance API 官方当前说明：

每个 Performance Account 每天最多发送 100,000 个请求。

因此广告采集必须：

- 控制请求数量；
- 优先批量接口；
- 记录限流；
- 避免无意义重复采集。

具体调度策略进入 `ARCHITECTURE.md`。

---

# 19. Performance Campaign

## 19.1 `/api/client/campaign`

状态：`VERIFIED`

用途：

Campaign 主数据。

当前已验证：

- Campaign ID；
- Title；
- State；
- advObjectType；
- Placement；
- Budget 配置；
- Strategy；
- PaymentType；
- Create / Update 时间。

分页：

当前实测 page 从 `1` 开始。

实现应显式分页，不依赖“无参数请求当前恰好返回全部数据”。

一致性：

- `list` 是实际 Campaign 数据；
- `total` 用于一致性校验。

如果 `total != len(list)` 或一次同步突然返回空集合，应视为需要复核的数据异常。

---

## 19.2 Campaign 空结果保护

当前历史测试曾真实观察到一次：

```json
{"list":[],"total":0}
```

随后大量重复验证又稳定恢复正常，最终根因无法确认。

因此：

如果本地已有历史 Campaign，而一次全量同步突然返回 0：

1. 不立即删除现有 Campaign；
2. 重新获取 Token；
3. 重新采样；
4. 检查分页与过滤；
5. 仍持续为空后再进入异常状态；
6. 不把一次空响应当作“全部 Campaign 已删除”。

---

## 19.3 Performance 字段兼容

当前实测 Campaign JSON 字段为：

```text
PaymentType
```

而不是只存在：

```text
paymentType
payment_type
```

解析器必须以原始响应为准，并对字段大小写变化保持兼容。

`placement` 当前是数组，必须保留数组语义。

---

# 20. Performance 广告统计

## 20.1 `/api/client/statistics/campaign/product/json`

状态：`VERIFIED`

粒度：

```text
Campaign + 查询区间
```

当前已验证可以获得：

- Views；
- Clicks；
- Money Spent；
- Orders；
- Orders Money；
- To Cart；
- CTR；
- Click Price；
- DRR；
- Strategy 等。

适合作为：

Campaign 区间大盘和交叉核验来源。

---

## 20.2 `/api/client/statistics/daily/json`

状态：`VERIFIED`

粒度：

```text
Campaign + Day
```

适合作为：

- 广告历史日事实；
- 趋势；
- Campaign 区间数据重建。

当前实测中，Daily 聚合与 Campaign 区间统计高度一致。

无数据日期通常会被省略，并不保证返回全 0 行。

因此缺失日期必须区分：

- 无数据；
- 0；
- 未成功同步。

---

## 20.3 `/api/client/statistics/products/sku`

状态：`VERIFIED`，但存在关键历史限制

粒度：

```text
Campaign + Day + SKU
```

当前可以获取：

- Views；
- Clicks；
- Spend；
- Orders；
- Sales / Revenue；
- CTR；
- CPC；
- DRR 等。

关键限制：

当前实测只允许查询：

- 今天；
- 昨天。

历史日期返回 400。

因此这是 O3Pilot 当前最重要的“不可补回数据源”之一。

同步要求：

- 必须持续每日采集；
- 必须把成功结果持久化；
- 不能假设未来可以重新拉取数月前的 SKU 广告数据；
- 应至少覆盖今天和昨天，以降低单次漏采风险。

具体执行时间由 `ARCHITECTURE.md` 定义。

---

## 20.4 `/api/client/campaign/{campaignId}/objects`

状态：`PARTIAL`

用途：

Campaign → Object 发现。

当前有效样本中 `Object.id` 与 SKU Statistics 中 SKU 一致。

但：

- 部分 Campaign 自然返回 404；
- 当前字段较少；
- 不能把 `Object.id == SKU` 固化为永久业务规则。

应与 SKU Statistics 或其他商品数据交叉验证。

---

## 20.5 Performance Phrases

接口：

- `/api/client/statistics/phrases`；
- `/api/client/statistics/phrases/json`；
- 对应异步状态和报表下载链路。

状态：`PARTIAL`

已验证：

- 可以提交任务；
- 可以轮询任务；
- 可以完成任务；
- 可以下载结果。

但当前真实任务仍未取得非空 `rows`。

因此暂时不能确认：

- 搜索词行级字段；
- SKU 关联；
- Impressions；
- Clicks；
- Spend；
- Orders；
- Revenue 等真实行结构。

结论：

Performance Phrases 当前不能作为 O3Pilot 正式广告搜索词功能的数据依赖。

如果未来获得非空真实样本，重新验证后再升级。

---

## 20.6 Performance 金额 Scale

不同 Performance Endpoint 的金额单位并不统一。

当前已验证：

- Campaign List 中部分预算字段使用百万分之一 RUB 的 Scale；
- Statistics 的 `moneySpent`、`ordersMoney`、`expense`、`sales`、`clickPrice`、`avgCpc` 等使用普通 RUB 数值表示。

因此：

禁止给所有 Performance 金额字段统一套用 `/ 1,000,000`。

每个 Endpoint / Field 必须具有独立解析规则。

---

## 20.7 Performance 比例字段

当前不同统计接口中，同名或相似比例字段的 Scale 也可能不同。

例如当前实测：

- Campaign Product CTR 为 0～1 比例；
- SKU CTR 为 0～100 百分比。

因此指标标准化前必须保留原始值和来源。

统一口径进入 `METRICS.md`。

---

# 21. Questions 与 Answers

当前 Questions / Answers 是 O3Pilot 已验证可使用的客户声音数据源。

## 21.1 `/v1/question/count`

状态：`VERIFIED`

获得：

- all；
- new；
- viewed；
- processed；
- unprocessed。

当前验证发现：

```text
all = new + viewed + processed
```

`unprocessed` 不能再作为第四个互斥状态重复加入总数。

具体状态集合语义不能仅靠计数继续推断。

---

## 21.2 `/v1/question/list`

状态：`VERIFIED`

获得：

- Question；
- SKU；
- 正文；
- 回答数；
- 状态；
- 商品链接等。

分页：

`last_id`。

已真实触发非空游标续页。

---

## 21.3 `/v1/question/info`

状态：`VERIFIED`

用于单条 Question Detail。

当前实测中不负责返回 Answer 正文，因此不能替代 Answer List。

---

## 21.4 `/v1/question/top-sku`

状态：`VERIFIED`

当前响应主要提供按问题热度排序的 SKU 列表。

当前没有返回 question_count。

因此：

它适合作为热点发现来源，但不能替代 Question List 的时间范围聚合。

---

## 21.5 `/v1/question/answer/list`

状态：`VERIFIED`

获得：

- Answer ID；
- Answer 正文；
- 作者；
- 发布时间；
- status_publication。

分页：

`last_id`。

当前已经真实验证：

- 非空 `last_id`；
- 下一页可能返回 0 条后游标结束。

因此不能把“非空 last_id”理解为“下一页一定存在记录”。

---

## 21.6 Questions 隐私边界

Question / Answer 中可能包含：

- `author_name`；
- 用户文本。

O3Pilot 应最小化存储、日志输出和界面暴露。

详细隐私和安全要求进入 `SECURITY.md`。

---

# 22. Reviews

## 22.1 `/v2/review/count`

状态：`ACCESS_DEPENDENT`

当前验证店铺均因未开通对应“评价管理”能力或 Premium Pro 返回订阅权限 403。

这属于正常权限限制，不是 API 故障。

---

## 22.2 `/v2/review/list`

状态：`ACCESS_DEPENDENT / NOT TESTED`

当前没有继续调用，因此不能写成“已验证 403”。

真实分页和响应字段仍未验证。

---

## 22.3 `/v2/review/info`

状态：`ACCESS_DEPENDENT / NOT TESTED`

当前没有真实 Review Detail 样本。

---

## 22.4 Review 数据规则

当前 O3Pilot 不得把 Review 数据展示为：

```text
0 Reviews
0 Low Ratings
```

正确状态应为：

```text
Unavailable / N/A
```

直到店铺开通订阅并重新验证读取能力。

---

# 23. Chat

Seller API 官方能力包含聊天管理。

但当前 DATA_SOURCES v1.0 尚没有把聊天历史读取建立为经过当前店铺完整验证的正式数据 Contract。

状态：

`DOCUMENTED`

因此：

- 可以保留为未来只读数据源；
- 当前不能依赖聊天数据完成核心 VOC 功能；
- 任何发送消息能力属于 `PROHIBITED`。

Questions / Answers 当前已经是可实现的客户声音主链路。

---

# 24. Rating 与店铺健康

## 24.1 `/v1/rating/summary`

状态：`VERIFIED`

用途：

当前店铺 Rating 总览。

当前已验证：

- Rating 分组；
- 当前值；
- 历史比较值；
- Rating code；
- 方向；
- Status；
- Premium 状态；
- Localization Index 等。

Rating code 是业务事实，应原样保存。

O3Pilot 的中文显示名称属于标准化层。

---

## 24.2 `/v1/rating/history`

状态：`VERIFIED`

用途：

Rating 历史。

当前测试得到：

- 多个 Rating 的日粒度历史；
- threshold；
- warning / danger / premium 状态结构。

历史窗口：

当前已验证 33 个日粒度点。

不能从单次测试推断永久最大保留天数。

---

## 24.3 Rating 风险状态边界

当前测试期间没有真实出现：

```text
danger = true
warning = true
```

因此：

- 字段结构已经验证；
- 真实触发场景尚缺样本。

`premium_scores` 结构已验证，但当前没有非空真实罚分样本。

---

## 24.4 `/v1/rating/index/fbs/info`

状态：`VERIFIED`

用途：

FBS Error Index 概览。

当前已验证：

- 当前 Index；
- Processing Cost；
- 按日 defects 历史。

---

## 24.5 `/v1/rating/index/fbs/posting/list`

状态：`PARTIAL`

接口当前返回 200，但当前真实样本：

```text
errors = []
```

因此尚未验证：

- 非空 Error Posting 行结构；
- 非零金额；
- 与历史 defects 的完整重建关系。

即使历史 defects 曾非零，也不能假设当前 Posting List 可以回溯整个历史。

---

# 25. Push Webhook

## 25.1 定位

Push Webhook 是事件数据源。

它负责：

- 提高数据变化发现速度；
- 触发后续同步；
- 保存事件时间线。

它不替代 Seller API 的完整状态同步。

原则：

```text
Webhook = Change Signal / Event
Seller API = State Reconciliation
```

---

## 25.2 `/v1/notification/list`

状态：`VERIFIED`

只读用途：

- 查询当前已注册 Endpoint；
- 是否启用；
- 已订阅 Event Types；
- Availability Status；
- 配置健康信息。

O3Pilot 可以读取。

O3Pilot 不负责注册、修改、启停或删除订阅。

---

## 25.3 `/v1/notification/push-type/list`

状态：`VERIFIED`

当前验证得到 22 个可用 Push Type。

覆盖的事件类别包括：

- Posting；
- Order；
- FBO Posting；
- Stocks；
- 商品创建 / 更新；
- Category Tree；
- Chat / Message。

规则：

Push Type capability 可以保存。

但 Push Type 名称相似不代表语义相同。

例如：

- `TYPE_NEW_POSTING`；
- `TYPE_ORDER_NEW`；
- `TYPE_FBO_POSTING_NEW`

当前 API 中可以并存，不能未经 Payload 验证就合并成一个事件类型。

---

## 25.4 Webhook Payload

状态：`PARTIAL / NOT VERIFIED`

当前已经验证：

- Push Type 列表；
- Endpoint 绑定信息。

当前尚未通过自然业务事件完整验证：

- 所有 Event 的真实 Payload；
- 事件延迟；
- 重试间隔；
- 重复推送；
- 事件顺序；
- 各取消时间字段；
- 不同 Posting / Order Event 之间的字段差异。

因此：

以下描述不能进入正式 Contract：

```text
1～3 秒送达
绝对秒级实时
某字段一定存在于某类 Event
Webhook 永不重复
Webhook 一定按顺序到达
```

---

## 25.5 Webhook 同步策略

每个 Webhook Event 应：

1. 保存原始 Payload；
2. 记录接收时间；
3. 尽可能提取 Ozon Event Time；
4. 幂等处理；
5. 触发对应业务对象 API 回读；
6. 用 API 当前状态进行最终一致性校验。

因为重复推送行为尚未验证，接收器必须按“事件可能重复”设计。

因为事件顺序尚未验证，接收器不能依赖到达顺序作为唯一状态机依据。

---

## 25.6 Webhook 管理写接口

以下类型操作不进入 O3Pilot 运行时 allowlist：

- Set；
- Update；
- Enable / Disable；
- Delete；
- 其他修改订阅的操作。

Webhook 配置由用户在 Ozon 外部完成。

O3Pilot 只读取配置并接收事件。

---

# 26. Ozon 官方报告

## 26.1 定位

Ozon 官方报告属于：

`MANUAL`

它们由用户从 Ozon Seller Center 人工导出后主动上传给 O3Pilot。

它们不是自动 API 的替代品。

日常核心功能不能要求用户持续上传报告才能工作。

---

## 26.2 主要用途

Ozon 官方报告主要用于：

- API 数据对账；
- 历史补录；
- 财务结算核验；
- API 暂未覆盖数据补充；
- 数据异常调查；
- 某些 Seller Center 业务指标的核对。

---

## 26.3 分析报告

Ozon Seller Center 当前可以导出多类参考报告。

报告字段根据报告类型变化。

已知业务类型包括：

### 商品报告

可能包含：

- Ozon Product ID；
- 不同 Ozon SKU；
- 商品状态；
- 价格；
- 佣金；
- 包装尺寸；
- 库存；
- Reserved；
- Recommended Price；
- 推荐价格来源链接等。

### 订单报告

可能包含：

- Shipment ID；
- Status；
- Cost；
- Ozon ID；
- Seller Product Code；
- Price；
- Quantity。

当前官方规则中，自有仓订单报告的单个时间范围最大为一个月。

需要多月数据时，应分别生成多个报告。

### 退货报告

可能包含：

- Returned Item ID；
- Shipment ID；
- Status；
- Product ID；
- Ozon ID；
- Return Reason；
- Placement Cost；
- Ready for Pickup 时间；
- Free Placement 截止时间；
- 退回商家时间；
- Location；
- Package Opened；
- Referral Fee 等。

部分退货报告只适用于特定履约场景。

### 应计与佣金报告

可能包含：

- 报告期；
- 期初 / 期末余额；
- 订单总额；
- 退款总额；
- 佣金；
- 附加服务；
- VAT；
- 交易级应计数据。

### 质量控制报告

包括：

- 取消率相关报告；
- 逾期交货相关报告。

这些报告还可能包含 Ozon 计算分子 / 分母的标志，可用于核对官方 Rating 口径。

---

## 26.4 财务报告文件

Seller Center 财务区域还提供多种结算和参考文件，例如：

- 卖家完工单；
- 商品销售报告；
- 每订单销售报告；
- 奖金 / 积分报告；
- 国际物流重新提供服务报告；
- 转账报告；
- 相互结算报告；
- 外币支付报告；
- 服务费用和销售开销报告；
- 赔偿报告；
- 罚款报告；
- 对账单。

部分月度报告在次月生成。

报告还可能发生修正。

因此导入系统必须支持：

- 报告期间；
- Report Type；
- 导入时间；
- 文件 Hash；
- 版本；
- 修正文件；
- 重复导入检测。

不得只按“月份 + 文件名”覆盖旧版本。

其中：

- 相互结算报告；
- 外币支付报告；
- 对账单；
- 修正文件；

可以成为 `Settlement / Payout` 域的人工事实来源。

但其字段和生成周期仍以具体报告类型为准，不能用统一 Schema 强行合并。

---

## 26.5 官方报告格式

不同报告的实际导出格式可能不同。

O3Pilot 不应设计一个假定“所有官方报告都是同一种 CSV Schema”的通用解析器。

每一种正式支持的报告类型应拥有：

- Report Type；
- Parser Version；
- Header / Schema 校验；
- 原始文件保存；
- 导入结果；
- 错误报告。

---

## 26.6 官方报告与 API 的关系

官方报告不能静默覆盖 API。

正确处理：

```text
API Fact
Report Fact
↓
Reconciliation
↓
Match / Difference / Unknown
```

如果存在差异，应展示和记录差异。

只有在明确的数据契约中，某项报告字段才能被定义为某项业务的补充主来源。

---

# 27. 卖家自有数据

## 27.1 定位

卖家自有数据属于 O3Pilot 经营分析所需、但 Ozon 不负责提供的数据。

属于：

`MANUAL`、ERP Import、物流商文件导入或用户维护数据。

Google Drive 中保存的开发样本本身不属于运行时数据源。

O3Pilot 运行时只处理用户主动导入或配置的数据。

---

## 27.2 当前主要类别

包括：

- 商品采购成本；
- 国内物流成本；
- 国际物流成本；
- 包装成本；
- SKU / 包裹实测重量；
- SKU / 包裹实测体积；
- 物流商实际计费重量；
- 供应商；
- 采购周期；
- 备货周期；
- 在途库存；
- 内部商品名称；
- 内部商品分类；
- 跨店商品映射；
- 其他经营成本和参数。

---

## 27.3 马帮 ERP 成本 Adapter

状态：`MANUAL`

样本验证：`VERIFIED FORMAT SAMPLE`

当前开发样本已经确认马帮 ERP 导出包含：

```text
订单编号
平台SKU
平台SKU数量
平台SKU单个成本
汇率(原币)
平台链接
```

这些字段可以作为订单商品采购成本导入的首个正式 Adapter 基础。

当前样本支持的处理原则：

- `订单编号` 用于关联 Ozon Posting；
- `平台SKU` 是 ERP 自身字段名，不能因名称包含 SKU 就直接等同 Ozon 数字 `sku`；
- `平台SKU单个成本` 与 `汇率(原币)` 必须原样保存；
- 当前样本支持 `transaction_cost_amount = cost_basis_amount / erp_exchange_rate` 的 Adapter 计算；
- 具体目标币种必须结合对应 Ozon 订单 / 店铺真实币种和 Adapter Profile 决定；
- 不得硬编码 `6.9 = USD`；
- 同一商品不同订单可以拥有不同历史采购成本。

因此：

历史 Profit 优先使用订单商品实际采购成本，而不是今天 SKU 的最新参考成本。

---

## 27.4 卖家采购成本汇率

ERP / Seller Cost FX 与 Ozon Business FX 完全分开。

```text
Seller Cost FX
!=
Ozon Exchange Rate
```

Ozon Exchange Rate 不得自动用于采购成本换算。

ERP 成本汇率也不得用于改写 Ozon Finance 原始金额。

---

## 27.5 物流商导出与 Seller Logistics Cost

状态：`MANUAL`

Adapter 状态：`FORMAT TO BE ADAPTERIZED`

开发参考资料中已经存在真实物流商导出样本，因此 O3Pilot 数据模型必须为以下 Seller-Owned Facts 预留正式位置：

- Posting / Shipment 关联；
- 物流商；
- 国内 / 国际 / 退回等费用；
- 实际重量；
- 体积重量；
- 计费重量；
- 账单时间；
- 原始币种；
- 原始行。

正式运行时不读取 Drive 中的样本文件。

首版实现应采用：

```text
User Upload
→ Logistics Provider Adapter
→ Raw Import Row
→ Seller Logistics Fact
```

具体字段映射在实现 Adapter 前以真实导出 Schema 再验证。

不能把：

```text
Ozon Finance Logistics Fee
```

与：

```text
Seller / Logistics Provider Actual Cost
```

合并为同一个原始字段。

---

## 27.6 包裹测量数据

卖家 / 物流商可能提供：

- 包裹实际重量；
- 长宽高；
- 体积；
- 体积重量；
- 最终计费重量。

这些属于 Package / Shipment Fact。

它们不能覆盖商品卡片 Product Attribute 中的重量和尺寸。

如果同一包裹存在多个来源测量结果，应全部保留来源并允许对账。

---

## 27.7 在途库存

在途库存属于独立于当前 Ozon 可售库存的事实。

来源可以包括：

- 卖家手工维护；
- ERP；
- 物流商；
- Ozon 可读取的交货 / 入库事实；
- Ozon 官方报告。

当前 Ozon Seller API 基线尚未验证一个足以覆盖所有 FBP 在途库存生命周期的完整只读 Contract。

因此：

```text
Current Inventory
!=
In-transit Inventory
```

首版不能为了补货计算方便，把“计划发货数量”直接加进当前库存字段。

---

## 27.8 跨店 Seller Catalog Mapping

同一真实商品可能存在于多个 Ozon Shop，并拥有不同：

- Ozon Product ID；
- SKU；
- offer_id。

跨店合并属于 Seller-Owned Mapping。

允许用户显式建立 Seller Catalog Item 与各 Shop Product 的映射。

不得只根据相同 `offer_id`、商品名或 SKU 自动认定为同一商品。

---

## 27.9 规则

卖家数据只能覆盖“卖家拥有定义权”的字段。

例如：

卖家可以定义：

- 采购成本；
- 供应商；
- Lead Time；
- 物流商实际费用；
- 包裹实际测量；
- 跨店内部商品映射。

卖家数据不能覆盖：

- Ozon 原始订单状态；
- Ozon Finance Accrual；
- Ozon SKU；
- Ozon Rating 原始值。

卖家修改数据后，应保留更新时间。

对于会影响历史利润或预测的字段，应支持有效期或历史版本，具体模型由 `DATA_MODEL.md` 定义。

---

# 28. 业务域数据覆盖矩阵

| 业务域 | 主来源 | 补充来源 | 当前状态 | 主要限制 |
|---|---|---|---|---|
| 店铺主体 | Seller API | — | VERIFIED | 当前状态为主 |
| API 权限 | Seller API | — | VERIFIED | 凭证权限不等于产品权限 |
| 商品目录 | Seller API | 官方报告 | VERIFIED | 历史需本地快照 |
| 商品详情 | Seller API | 官方报告 | VERIFIED | 字段随类目变化 |
| 商品内容质量 | Seller API | — | VERIFIED | 评分规则可能变化 |
| 当前价格 | Seller API | 官方报告 | VERIFIED | 历史需本地快照 |
| 佣金费率 | Seller API | Finance / 报告核验 | VERIFIED | 费率不等于实际订单佣金 |
| 当前库存 | Seller API | 官方报告 | VERIFIED | Analytics Stocks 暂不可作为主来源 |
| FBO 仓级库存 | Seller API | — | VERIFIED | 仅对应 FBO 仓级覆盖 |
| 库存周转 | Analytics | — | VERIFIED | Ozon 自有滚动指标 |
| 订单 | Seller API | Webhook / 报告 | VERIFIED | 技术 schema ≠ 最终业务模式 |
| 配送时效 | Seller API | — | VERIFIED | 部分节点需要 Detail |
| 取消 | Orders + Analytics | 官方质量报告 | VERIFIED | 公式在 METRICS |
| rFBS 退货 | Seller API | 官方报告 | VERIFIED | List + Detail 组合 |
| 综合退货 / WHD | Seller API | 官方报告 | VERIFIED | 不等同普通首次履约 |
| 产品退货率 | Orders + Returns | Analytics 核验 | DERIVED | 公式在 METRICS |
| Finance | Accrual v1 | 官方财务报告 | VERIFIED | Posting / SKU 粒度不一致 |
| Finance v3 | 旧 Seller API | — | DEPRECATED | 2026-09-08 关闭 |
| 日级经营 Analytics | Seller API | Orders 核验 | VERIFIED | metric 必须 allowlist |
| SKU 搜索表现 | Seller API | — | VERIFIED | 当前观察有计算延迟 |
| 搜索词 | Seller API Analytics | — | VERIFIED | SKU + Query + Period |
| 广告 Campaign | Performance API | — | VERIFIED | 必须防单次空集合 |
| Campaign 日统计 | Performance API | — | VERIFIED | 无数据日可能省略 |
| Campaign SKU 日统计 | Performance API | — | VERIFIED | 仅今天 / 昨天，可补回性差 |
| Performance Phrases | Performance API | — | PARTIAL | 未取得非空 rows |
| Questions | Seller API | — | VERIFIED | 游标分页 |
| Answers | Seller API | — | VERIFIED | 游标非空不保证下一页有记录 |
| Reviews | Seller API | — | ACCESS_DEPENDENT | 当前订阅无权限 |
| Chat | Seller API | Webhook | DOCUMENTED | 当前未形成完整读取 Contract |
| Rating 当前值 | Seller API | Seller Info | VERIFIED | Ozon Rating 规则会变化 |
| Rating 历史 | Seller API | — | VERIFIED | 永久最大保留范围未确认 |
| FBS Error Index | Seller API | — | VERIFIED | 当前 Index 可读 |
| FBS Error Posting | Seller API | — | PARTIAL | 当前无真实 Error 行 |
| Webhook Type | Seller API | — | VERIFIED | Payload 不可由 Type 名称推断 |
| Webhook Payload | Webhook | Seller API 回读 | PARTIAL | 真实事件 Contract 尚未完整验证 |
| Ozon 业务汇率 | Ozon Exchange Rate API | 官方财务规则 / 报告 | VERIFIED | Undocumented XAPI；需持久化历史并容忍接口变化 |
| 官方报告 | 用户上传 | — | MANUAL | 低频、可能存在修正版 |
| Settlement / Payout | 官方财务报告 | Finance Accrual 对账 | MANUAL | 当前无完整自动 API Contract |
| 采购成本 | ERP / 卖家自有数据 | — | MANUAL | 历史订单成本需版本化 |
| Seller Cost FX | ERP / 卖家自有数据 | — | MANUAL | 不得与 Ozon FX 混用 |
| Seller Logistics Cost | 物流商 / 卖家自有数据 | Orders 关联 | MANUAL | Adapter Schema 需逐类验证 |
| Package Measurement | 物流商 / 卖家自有数据 | Product / Orders 核验 | MANUAL | Product Weight ≠ Package Weight |
| In-transit Inventory | 卖家数据 / Ozon 可得交货事实 | 官方报告 | MANUAL | 当前无完整自动覆盖 Contract；自动来源待验证 |
| Seller Catalog Mapping | 卖家自有数据 | Product | MANUAL | 必须显式映射 |

---

# 29. 历史回溯能力

不同数据源的历史可恢复能力不同。

| 数据 | 历史回溯能力 | O3Pilot 要求 |
|---|---|---|
| 商品目录 | API 当前快照 | 周期保存快照 |
| 商品属性 | API 当前快照 | 需要历史时保存版本 |
| 价格 | API 当前快照 | 周期保存快照 |
| 当前库存 | API 当前快照 | 高频保存库存快照 |
| 订单 | 支持日期范围查询 | 持续同步 + 可回补 |
| FBO Detail | 按 Posting 查询 | 需要时补充 |
| 退货 | 支持列表 / Detail | 持续同步 + 可回补 |
| Finance Accrual | 按日查询 | 逐日回补 |
| Analytics Data | 支持日期区间 | 可回补历史 |
| Search Analytics | 支持 Period | 可回补已保留区间 |
| Turnover | 当前 / 滚动 Analytics | 周期保存快照 |
| Campaign | 当前 Campaign 集合 | 保存状态历史 |
| Campaign Interval Stats | 区间查询 | 可用于历史聚合 |
| Campaign Daily | 日期数据 | 持久化日事实 |
| Campaign SKU Daily | 仅今天 / 昨天 | 必须每天持续采集，不能依赖未来回补 |
| Phrases | 异步报表 | 当前不可作为历史主来源 |
| Rating History | 日期范围 | 可获取当前可用历史区间 |
| Webhook | 无历史回放保证 | 必须用 Seller API 对账补漏 |
| Ozon Exchange Rate | 支持 Period 查询；长期稳定性未承诺 | 持久化每日 / 区间历史 |
| 官方报告 | 取决于 Seller Center 可生成范围 | 人工导入并保留文件版本 |
| Settlement / Payout | 依赖 Seller Center 报告保留范围 | 导入后长期保存 |
| ERP 订单实际采购成本 | 取决于用户 ERP 导出 | 按订单商品长期保存 |
| Seller Logistics Cost | 取决于物流商账单 / 导出 | 按 Posting / Package 长期保存 |
| Package Measurement | 取决于物流商 / Seller Data | 保存来源与测量时间 |
| In-transit Inventory | 来源能力不同 | 保存状态历史，不覆盖当前库存 |
| Seller Catalog Mapping | 用户维护 | 保存有效期与映射版本 |

---

# 30. 同步优先级分类

DATA_SOURCES 不规定具体 Cron 时间，但规定数据源的同步性质。

## 30.1 Event

Push Webhook。

特点：

- 事件驱动；
- 低延迟目标；
- 不能承担最终完整性。

---

## 30.2 Current State

例如：

- Orders；
- Stocks；
- Prices；
- Campaign。

特点：

- API 返回当前状态；
- O3Pilot 通过周期同步建立变化历史。

---

## 30.3 Historical Fact

例如：

- Finance Accrual；
- Analytics SKU × Day；
- Campaign Daily。

特点：

- 事实一旦发生需要持久化；
- 可以在 API 支持范围内回补。

---

## 30.4 Expiring Window

Performance SKU Statistics。

特点：

- 只允许今天 / 昨天；
- 数据窗口会快速失效；
- 漏采后无法依赖当前接口补回。

这是 O3Pilot 最高采集可靠性要求之一。

---

## 30.5 Low Frequency

例如：

- Seller Info；
- Roles；
- Product Attributes；
- Product Content Rating。

变化频率相对较低。

---

## 30.6 Manual

包括：

- Ozon 官方报告；
- 卖家自有数据。

必须显示最后导入 / 更新时间。

---

# 31. 分页策略

Ozon 不同 Endpoint 使用不同分页协议。

禁止建立一个“所有接口统一 page=1”的假设。

当前已验证分页模式包括：

| 模式 | 代表接口 |
|---|---|
| Cursor | FBS / FBO Posting、商品库存、FBO 仓级库存等 |
| `limit + offset` | `/v1/analytics/data` |
| Page 从 0 开始 | Search Analytics |
| Page 从 1 开始 | Performance Campaign |
| `last_id` | rFBS Returns、Question、Answer、Finance 部分接口 |
| 异步任务 | Performance Phrases |

每个 Endpoint 的分页行为必须独立实现和测试。

分页结束条件必须来自该 Endpoint Contract，不能复用其他接口规则。

---

# 32. 空集合处理

API 返回空数组不一定代表业务事实为 0。

当前已经存在以下真实案例：

- Performance Campaign 曾出现一次不可复现的全空响应；
- Analytics Stocks 长期返回空 `items`，即使传入真实 FBO 正库存 SKU；
- FBS Error Posting 当前 `errors=[]`，但历史 defects 曾非零；
- Search Analytics 对无数据 SKU 可能直接省略对象；
- Performance Daily 对无数据日期可能直接省略行。

因此空集合必须根据来源语义处理。

对“本地已有数据 → API 突然返回全空”的高风险场景，应优先：

- 重试；
- 重新认证；
- 校验分页；
- 校验过滤条件；
- 标记异常；

而不是立即执行全量删除。

---

# 33. 限流与错误分类

O3Pilot 必须区分：

## 可重试

例如：

- 429；
- 临时网络失败；
- 部分 5xx。

重试应有：

- 退避；
- 最大次数；
- 日志；
- 不重复写入。

## 不应盲目重试

例如：

- 400 参数验证错误；
- 403 订阅权限不足；
- 明确的历史日期不支持；
- 业务对象自然不存在。

例如 Review 403 属于权限状态，不应当成临时网络故障无限重试。

---

# 34. 数据幂等

任何同步任务都必须支持重复执行。

同一个：

- Product；
- Posting；
- Return；
- Accrual；
- Campaign；
- Analytics Row；
- Question；
- Answer；
- Webhook Event；

重复进入系统不能自动产生重复业务事实。

具体唯一键和 Upsert 策略由 `DATA_MODEL.md` 定义。

---

# 35. Schema 变化

Ozon API 的：

- 新字段；
- 可选字段；
- 字段缺失；
- Enum 新值；
- 大小写变化；
- Array / Object 变化；

都必须被视为正常演进风险。

已存在的真实例子包括：

- Performance `PaymentType` 字段大小写；
- Webhook `seller_endpoint` 未绑定时字段整体缺失；
- Question Answer 非空游标但下一页 0 行；
- 不同商品类目的 Attributes 集合完全不同。

因此：

- 原始 JSON 必须保留；
- 解析器对未知字段应容忍；
- Enum 不应通过封闭映射导致同步崩溃；
- 关键 Contract 变化应产生数据质量告警。

---

# 36. 货币与汇率规则

## 36.1 Money 原则

所有 Money 数据必须优先保存：

- 原始金额；
- 原始币种。

不能通过 Shop Currency 推断 API Money Currency。

当前真实数据已经证明：

- 店铺结算货币可以不同；
- 商品价格可以使用店铺相关币种；
- 买家相关价格和佣金可以出现 RUB；
- Finance Accrual 当前测试账期 Money 为 RUB；
- Performance Statistics 使用 RUB；
- Price Index 的比较价格可以拥有独立币种。

因此币种是字段级事实，不是店铺级全局常量。

---

## 36.2 Ozon Exchange Rate API

状态：`VERIFIED`

接口性质：`Undocumented Ozon XAPI`

当前已验证接口：

```text
GET https://xapi.ozon.ru/exchange-rates/sellers/exchange-rate/by-period
```

该接口来自 Ozon 官方帮助页面实际使用的 `xapi.ozon.ru` 数据请求。

当前验证特点：

- 不属于 Seller API / Performance API 正式公开 Contract；
- 当前请求无需 Seller API Key / Cookie；
- 支持按时间段查询；
- 可以获得多个币种相对 RUB 的汇率时间线；
- 已确认存在 `rate` 与 `rateWithAdjustment` 两套值。

当前页面语义：

```text
rate
→ 服务和罚款

rateWithAdjustment
→ 销售
```

因此 O3Pilot 将其作为正式运行时辅助数据源，但稳定性级别低于公开 Seller API Contract。

必须：

- 保存 Raw Response；
- 保存 Fetch Time；
- 持久化历史区间；
- 允许 Schema / Endpoint 变化；
- 不依赖页面实时调用才能解释历史 Profit。

---

## 36.3 Ozon Business FX 与 Seller Cost FX

必须严格区分：

```text
Ozon Business FX
!=
Seller Cost FX
```

Ozon Business FX 用于解释 Ozon 自己的业务换算。

Seller Cost FX 来自：

- ERP；
- 卖家维护；
- 其他 Seller-Owned Cost Source。

Ozon FX 不得自动用于采购成本。

Seller Cost FX 不得改写 Ozon Finance 原始金额。

---

## 36.4 汇率适用时间依据

不能假设所有 Ozon 业务金额都使用“订单创建日汇率”。

当前 Ozon 业务规则已经证明不同费用存在不同时间依据，例如：

```text
ORDER_CREATED_AT
SERVICE_PROVIDED_AT
ACCRUAL_DATE
```

销售及部分配送 / 转售费用可能使用订单创建时间。

仓储、逆向物流、销毁、取货点处理等服务可能使用服务提供时间。

因此每次 Money Conversion 必须保存：

```text
rate_basis_type
rate_basis_time
rate_policy_version
```

而不是只保存一个笼统 `business_time`。

---

## 36.5 Exchange Rate Timeline

Ozon Exchange Rate 返回的是时间区间，而不是简单的：

```text
YYYY-MM-DD = rate
```

当前实测时间边界以 UTC 21:00 切分，对应莫斯科自然日 00:00。

正确匹配原则：

```text
valid_from <= rate_basis_time < valid_to
```

不能简单按北京时间日期 Join。

---

# 37. 时间规则

原始 API 时间必须保留。

O3Pilot 业务展示和默认统计使用：

```text
Asia/Shanghai
UTC+8
```

但不能覆盖 API 原始时间。

不同 API 还存在不同日期格式要求：

- 有些接受 `YYYY-MM-DD`；
- Search Analytics 要求 RFC3339 / ISO Timestamp；
- Rating 某些查询要求 RFC3339；
- Performance Daily 返回日期但不明确声明最终统计时区。

因此每个来源必须独立处理时间 Contract。

不能假设所有 Ozon API 都使用相同日期边界。

---

# 38. 数据新鲜度

每项数据必须能够展示其新鲜度。

至少区分：

- Source Event Time；
- Source Business Date；
- Fetched At；
- Coverage Start；
- Coverage End；
- Imported At。

对于：

- Analytics；
- Search；
- Advertising；
- Finance；

“API 请求刚刚成功”不等于“业务数据已经完整到现在”。

O3Pilot 应展示实际覆盖到的业务日期，而不是只展示最后同步时间。

---

# 39. 当前明确不可作为核心依赖的数据

以下数据在 DATA_SOURCES v1.0 中不能成为核心功能的必要前置：

## Analytics Stocks

原因：

持续没有非空真实样本。

## Performance Phrases

原因：

异步链路成功，但没有非空真实行。

## Reviews

原因：

当前店铺订阅权限不足。

## FBS Error Posting Detail Dataset

原因：

当前只有空 `errors` 样本。

## Webhook Payload 具体字段

原因：

Push Type 已验证，但真实自然事件 Payload Contract 尚未完整验证。

## Chat History

原因：

官方能力存在，但当前验证基线没有建立正式读取 Contract。

---

# 40. 当前禁止的数据操作

即使 Ozon API 提供能力，以下操作不属于数据源，也不得进入 O3Pilot：

- 创建或修改商品；
- 修改商品属性；
- 修改价格；
- 修改库存；
- 创建、取消或处理订单；
- 执行发货；
- 修改仓库和物流配置；
- 创建或修改广告；
- 修改广告商品；
- 修改预算；
- 修改出价；
- 回复 Questions / Reviews；
- 发送 Chat Message；
- 注册 Webhook；
- 修改 Webhook；
- 启停 Webhook；
- 删除 Webhook；
- 其他改变 Ozon 服务端状态的调用。

实现安全规则：

```text
Default Deny
+
Explicit Read Allowlist
```

---

# 41. 当前已验证只读 Endpoint Registry

以下 Registry 是 DATA_SOURCES v1.0 的当前验证基线。

它不是 Ozon API 全量列表。

## 41.1 Seller / Product / Inventory / Order / Returns / Finance / Analytics

| 业务域 | Endpoint | 状态 | 备注 |
|---|---|---|---|
| Shop | `/v1/seller/info` | VERIFIED | 店铺主体、币种、订阅、部分 Rating |
| Permission | `/v1/roles` | VERIFIED | API Key Role / Methods |
| Product | `/v3/product/list` | VERIFIED | 商品目录 |
| Product | `/v3/product/info/list` | VERIFIED | 商品主数据批量详情 |
| Product | `/v4/product/info/attributes` | VERIFIED | 属性、尺寸、重量、图片等 |
| Product | `/v1/product/rating-by-sku` | VERIFIED | 商品内容质量 |
| Price | `/v5/product/info/prices` | VERIFIED | 价格、佣金、营销、Price Index |
| Inventory | `/v4/product/info/stocks` | VERIFIED | 全渠道当前库存 |
| Inventory | `/v1/product/info/stocks-by-warehouse/fbo` | VERIFIED | FBO 仓级库存 |
| Inventory Analytics | `/v1/analytics/stocks` | PARTIAL | 当前始终空 items |
| Turnover | `/v1/analytics/turnover/stocks` | VERIFIED | 库存周转 |
| Order | `/v4/posting/fbs/list` | VERIFIED | FBS 技术订单 |
| Order | `/v3/posting/fbo/list` | VERIFIED | FBO 技术订单 |
| Order Detail | `/v2/posting/fbo/get` | VERIFIED | FBO Detail |
| Order Detail | `/v1/posting/fbp/get` | VERIFIED / BETA | FBP Detail |
| Return | `/v2/returns/rfbs/list` | VERIFIED | rFBS Returns |
| Return Detail | `/v2/returns/rfbs/get` | VERIFIED | rFBS Return Detail |
| Return | `/v1/returns/list` | VERIFIED | 综合退货 / WHD |
| Finance | `/v1/finance/accrual/types` | VERIFIED | Accrual 字典 |
| Finance | `/v1/finance/accrual/by-day` | VERIFIED | 每日 Accrual；非空游标未触发 |
| Finance | `/v1/finance/accrual/postings` | VERIFIED | Posting Accrual |
| Analytics | `/v1/analytics/data` | VERIFIED | SKU × Day |
| Search | `/v1/analytics/product-queries` | VERIFIED | SKU Search Period |
| Search | `/v1/analytics/product-queries/details` | VERIFIED | SKU × Query × Period |
| Question | `/v1/question/count` | VERIFIED | Question Count |
| Question | `/v1/question/list` | VERIFIED | Question List |
| Question | `/v1/question/info` | VERIFIED | Question Detail |
| Question | `/v1/question/top-sku` | VERIFIED | Hot SKU |
| Answer | `/v1/question/answer/list` | VERIFIED | Answer Text |
| Review | `/v2/review/count` | ACCESS_DEPENDENT | 当前订阅 403 |
| Review | `/v2/review/list` | ACCESS_DEPENDENT / NOT TESTED | 当前未调用 |
| Review | `/v2/review/info` | ACCESS_DEPENDENT / NOT TESTED | 当前未调用 |
| Rating | `/v1/rating/summary` | VERIFIED | Current Rating |
| Rating | `/v1/rating/history` | VERIFIED | Rating History |
| Rating | `/v1/rating/index/fbs/info` | VERIFIED | FBS Error Index |
| Rating | `/v1/rating/index/fbs/posting/list` | PARTIAL | 当前 errors=[] |
| Webhook Config | `/v1/notification/list` | VERIFIED | 只读配置 |
| Webhook Types | `/v1/notification/push-type/list` | VERIFIED | 22 Push Types |

---

## 41.2 Finance Deprecated

| Endpoint | 状态 | 规则 |
|---|---|---|
| `/v3/finance/transaction/totals` | DEPRECATED | 仅迁移 / 历史对账 |
| `/v3/finance/transaction/list` | DEPRECATED | 仅迁移 / 历史对账 |

关闭基线日期：

2026-09-08。

新系统不得依赖。

---

## 41.3 Performance

| Endpoint | 状态 | 备注 |
|---|---|---|
| `/api/client/token` | VERIFIED | Client Credentials Token |
| `/api/client/campaign` | VERIFIED | Campaign List |
| `/api/client/statistics/campaign/product/json` | VERIFIED | Campaign Period Stats |
| `/api/client/statistics/daily/json` | VERIFIED | Campaign × Day |
| `/api/client/statistics/products/sku` | VERIFIED | Campaign × Day × SKU；仅今天/昨天 |
| `/api/client/statistics/phrases` | PARTIAL | 异步工作流，未取得非空 Rows |
| `/api/client/statistics/phrases/json` | PARTIAL | 同上 |
| `/api/client/campaign/{campaignId}/objects` | PARTIAL | 部分自然 404 |

---

## 41.4 Ozon Exchange Rate XAPI

| Endpoint | 状态 | 备注 |
|---|---|---|
| `GET https://xapi.ozon.ru/exchange-rates/sellers/exchange-rate/by-period` | VERIFIED | Undocumented Ozon XAPI；官方帮助页面实际使用；销售 / 服务两套汇率；必须持久化历史 |

该 Endpoint 不属于 Seller API allowlist。

它应进入独立的 Ozon XAPI Read Allowlist，并保持与 Seller API / Performance API 凭证体系分离。

---

# 42. 待验证清单

DATA_SOURCES v1.0 当前正式保留以下待验证项目：

## Finance

- compensation 非零场景；
- `/by-day` 非空 `last_id` 续页；
- Type 29 / 32 多 SKU 粒度；
- `container_fees` 非空样本。

## Analytics

- `/v1/analytics/stocks` 非空样本及真实适用范围。

## Performance

- Phrases 非空 Rows；
- Campaign Objects 更完整真实样本；
- Campaign 偶发全空响应根因。

## Reviews

- 订阅开通后的 Count；
- List；
- Detail。

## Rating

- danger / warning 真实触发样本；
- Premium Score 非空样本；
- FBS Error Posting 非空样本。

## Webhook

- 真实 Payload；
- 重复事件；
- Retry；
- Event Ordering；
- End-to-End Latency；
- 各 Push Type 真实字段。

## Chat

- 正式只读历史数据接口和当前店铺真实返回。

## Exchange Rate XAPI

- Endpoint 长期稳定性；
- 历史区间是否可能被 Ozon 修订；
- Marketplace / Currency 覆盖范围变化；
- 页面调用 Contract 变化检测。

## Seller Logistics

- 各物流商导出 Schema；
- Posting / Package 稳定关联键；
- 实际重量 / 体积重量 / 计费重量字段语义；
- 退货物流费用的独立粒度。

## Inbound Supply

- 是否存在稳定且完整的只读 Seller API Contract；
- 各状态与在途数量的可靠定义；
- 验收入库数量的最终来源。

## Settlement / Payout

- 是否存在可替代人工财务报告的稳定只读 API；
- 实际付款日期 / 状态 / 到账金额的自动来源。

---

# 43. Phase 0 数据源验收标准

在进入稳定业务功能开发前，Data Foundation 至少需要满足：

1. Seller API 和 Performance API 凭证严格分离；
2. 每个 Shop 数据隔离；
3. Ozon Read Allowlist 生效；
4. 所有非 allowlist Ozon 调用默认拒绝；
5. 商品目录可以全量同步；
6. 商品属性可以同步；
7. 当前价格可以同步并建立快照；
8. 当前库存可以同步并建立快照；
9. FBS / FBO / FBP 已验证订单来源可以同步；
10. 退货 List / Detail 可以同步；
11. Finance Accrual v1 可以逐日同步；
12. Finance v3 不进入新运行时主链路；
13. Analytics SKU × Day 可以分页完整同步；
14. Search Analytics 可以正确处理 page=0 和对象省略；
15. Performance Campaign 可以显式分页；
16. Performance Daily 可以持续持久化；
17. Performance SKU Daily 可以在有效窗口内持续采集；
18. 单次 Campaign 空响应不会触发灾难性删除；
19. Questions / Answers 可以完整分页；
20. Review 权限不足显示为 N/A，而不是 0；
21. Rating Summary / History 可以同步；
22. Webhook Type / Config 可以只读同步；
23. Webhook Event 可以保存 Raw Payload；
24. Webhook 失败不能破坏 API 最终一致性；
25. 所有金额保存原始币种；
26. 所有时间保存原始时间；
27. 所有重要数据具有 source lineage；
28. 所有分页策略按 Endpoint 独立实现；
29. 429 和临时错误可以安全重试；
30. 400 / 403 等确定性错误不会无限重试；
31. 空集合不会被无条件解释为业务 0；
32. Ozon 官方报告导入具有 Report Type 和版本信息；
33. 卖家自有数据与 Ozon 原始事实物理或逻辑区分；
34. Ozon Exchange Rate XAPI 具有独立 Read Allowlist、Raw 保存和历史持久化；
35. Ozon Business FX 与 Seller Cost FX 分离；
36. 每次汇率换算保存 Rate Basis Type / Time；
37. 马帮 ERP 订单商品成本可以原样导入并关联 Posting Item；
38. 历史订单实际采购成本不会被当前 SKU 成本覆盖；
39. Seller Logistics Cost 与 Ozon Finance Logistics Fee 分离；
40. Product Weight、Package Weight、Chargeable Weight 分离；
41. 物流商导入可以保存 Raw Row 和 Mapping Status；
42. 当前库存与在途库存分离；
43. Settlement / Payout 不由 Finance Accrual 推断伪造；
44. 跨店商品映射必须显式可追溯；
45. Posting 发货截止和承诺配送窗口允许保存历史变化；
46. 同一同步任务重复执行不产生重复业务记录。

---

# 44. 与其他文档的关系

## PRODUCT.md

定义：

O3Pilot 应该具备哪些产品能力。

DATA_SOURCES 负责回答：

这些能力的数据从哪里来，以及当前能可靠获得到什么程度。

---

## DATA_MODEL.md

已经定义：

- 原始对象如何保存；
- 标准化实体；
- ID；
- 主键；
- 关联关系；
- Shop 隔离；
- Product；
- Order；
- Posting；
- Inventory；
- Return；
- Finance；
- Advertising；
- Rating；
- Question / Answer；
- Source Lineage；
- Seller Catalog；
- Package；
- Seller Logistics Cost；
- Inbound Supply；
- Settlement / Payout；
- Ozon FX / Seller Cost FX。

DATA_MODEL 可以为已明确存在但当前只能人工导入的业务事实建立实体，但不能把未验证的 API 字段伪装成自动来源。

---

## METRICS.md

定义：

- 销售；
- 转化；
- 库存；
- 物流；
- 取消率；
- 产品退货率；
- Finance 聚合；
- Profit；
- Advertising；
- Shop Health；
- Prediction Metrics。

METRICS 中所有指标必须能够回溯到 DATA_SOURCES 中已经登记的数据。

---

## ARCHITECTURE.md

定义：

- 实际同步周期；
- Scheduler；
- Retry；
- Backoff；
- Queue / Worker；
- Storage；
- Raw Data；
- Snapshot；
- Monitoring。

DATA_SOURCES 只定义数据源约束和同步性质，不提前锁定具体技术实现。

---

# 45. 核心数据原则

**没有来源的数据，不进入正式业务事实。**

**没有验证的数据，不伪装成已验证数据。**

**缺失不等于 0。**

**空集合不等于业务为 0。**

**API 技术模式不等于业务模式。**

**Shop Currency 不等于所有 Money Currency。**

**Ozon Business FX 不等于 Seller Cost FX。**

**汇率换算必须保存适用时间依据。**

**Product Weight 不等于 Package Weight，也不等于 Chargeable Weight。**

**Ozon Finance Logistics Fee 不等于 Seller Logistics Cost。**

**当前库存不等于在途库存。**

**Webhook 负责变化发现，API 负责最终状态。**

**Finance Accrual v1 是新系统 Finance 主链路。**

**人工报告用于对账和补充，也可以承载当前无自动 API 的 Settlement / Payout 事实。**

**跨店商品只能通过显式 Seller Catalog Mapping 合并。**

**时间窗口会失效的数据必须及时持久化。**

**所有来源都必须可追溯。**

**O3Pilot 永远只读 Ozon。**
