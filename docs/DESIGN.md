# O3Pilot — DESIGN.md

> Version: 1.0  
> Status: Draft for Review  
> Updated: 2026-09-04  
> Applies to: O3Pilot  
> B.3 Remediation: 2026-09-05 — Session Revoked projection/QA aligned to `SECURITY.md` v1.1 multi-session semantics; scope limited to §4.11 and §37. Parent baseline: `cd8f5d8aa91a586ca73b290e79b67d24d27dc487bfcaa7fc65fef57e98db3ad0`.

# 1. 文档目的

`DESIGN.md` 定义 O3Pilot 的正式产品界面、信息架构、页面信息优先级、视觉系统、交互模式、数据表达、动效、响应式、可访问性和前端体验验收标准。

`DESIGN.md` 可以引用 `PRODUCT.md`、`DATA_SOURCES.md`、`DATA_MODEL.md`、`METRICS.md` 等上游 Contract 中已经定义的业务能力、实体、指标和数据状态，但不得为了界面实现便利而重新定义、弱化或改变其语义。

本文件负责回答：

- O3Pilot 的页面和导航如何组织；
- 不同产品能力如何映射到稳定的信息架构；
- 页面布局、排版、颜色、间距、圆角、Surface、图标如何统一；
- 数据状态、来源、新鲜度、覆盖率、估算、未知、无权限、无分母等如何正确展示；
- 多店铺、多币种、多时间口径如何在 UI 中避免误导；
- Dashboard、Sales、Products、Orders、Inventory、Fulfillment、Returns、Advertising、Customer Voice、Finance、Profit、Shop Health、Alerts、Recommendations、Data Center、Settings 等界面如何组织；
- 左侧导航如何展开、收缩和反馈；
- Lucide 与 Morphicons 如何进入稳定的图标抽象层；
- 动效在什么情况下允许存在、如何保证至少 120Hz 高刷新率环境下的流畅度；
- Loading、Empty、Unavailable、Error、Partial、Stale 等状态如何区分；
- Desktop、Compact、Narrow 屏幕如何适配；
- 键盘、Focus、Reduced Motion、对比度和图表辅助信息如何满足可访问性；
- UI 设计如何在未来版本中演进而不破坏产品一致性。

本文件不重新定义：

- O3Pilot 的产品边界；
- Ozon API 可读取能力；
- 数据源验证状态；
- 数据库 Schema；
- 业务实体主键；
- Metric 公式；
- Profit / Forecast 最终计算逻辑；
- Scheduler / Worker / Job System；
- 安装、升级、备份和恢复流程；
- 密码、Session、CSRF、Secret 等服务端安全实现。

对于认证、Session、Credential、Secret 等安全相关能力，`DESIGN.md` 只定义用户可见状态和交互表现，不定义认证、加密、授权等服务端安全机制。

上述内容分别由：

- `PRODUCT.md`
- `DATA_SOURCES.md`
- `DATA_MODEL.md`
- `METRICS.md`
- `ARCHITECTURE.md`
- `SECURITY.md`
- `DEPLOYMENT.md`

定义。

---

# 2. 设计来源与优先级

## 2.1 强制项目约束

O3Pilot 设计必须满足以下强制要求：

1. 全局统一使用 `MiSans Global`；
2. 中文、英文、俄语、数字和常用符号均使用同一字体体系；
3. 只使用 `400 / 500 / 600 / 700` 四档字重；
4. 视觉语言参考 Apple `DESIGN.md`，但只吸收适合 O3Pilot 数据产品的视觉原则，不继承其营销页面布局、字体、信息密度或动效规则；
5. UI / UX 方法可参考 UI UX Pro Max，主要用于可访问性、响应式、信息组织、交互完整性与生产级 UI 检查；其推荐的视觉风格、字体、色板和动效不自动进入 O3Pilot；
6. 动效规范使用 Emil Kowalski `skills` 作为专项外部参考；Apple DESIGN 与 UI UX Pro Max 不进入 O3Pilot Motion 决策链；
7. 图标包使用 Lucide；
8. Morphicons 可以用于图标状态变化；
9. 左侧导航栏支持收缩，收缩后保留图标；
10. 导航项点击后，图标 Morph 为 Check，再恢复原图标；
11. 动效必须按至少 120Hz 环境的体验基线设计；
12. 如果 CSS / HTML / TypeScript 足够实现，不因动效而额外引入库；
13. 当前已经实现的前端代码和页面结构不是 `DESIGN.md` 的约束，`DESIGN.md` 定义目标设计。

## 2.2 Contract 与设计决策优先级

设计决策必须先满足 O3Pilot 已经正式定义的上游 Contract。

这些 Contract 决定业务事实、产品边界、数据语义、指标口径、安全边界和运行约束：

```text
PRODUCT.md
DATA_SOURCES.md
DATA_MODEL.md
METRICS.md
ARCHITECTURE.md
SECURITY.md
DEPLOYMENT.md
```

`DESIGN.md` 可以决定这些事实如何被组织、呈现和交互，但不能覆盖或重新解释这些事实。

例如：

- 不能为了页面简洁把 `UNAVAILABLE` 显示成 `0`；
- 不能为了减少控件而把不同币种直接相加；
- 不能为了更强的 CTA 引入 Ozon 写操作；
- 不能为了视觉统一把 Ozon 官方指标与 O3Pilot 自算指标合并成同一语义。

在不违反上游 Contract 的前提下，O3Pilot 设计决策优先级为：

```text
用户明确确认的 O3Pilot 设计决策
↓
DESIGN.md 已正式采用的 O3Pilot 规则
↓
指定专项参考
↓
通用外部参考
↓
已有代码
```

其中：

- 用户后续明确确认的新设计决策可以修改 `DESIGN.md`；
- 修改必须显式写回 `DESIGN.md`，不能只存在于实现代码或口头约定中；
- 已有代码永远不能反向定义正式设计规范；
- 外部参考不能绕过已经确认的 O3Pilot 设计规则。

## 2.3 外部参考采用与冻结规则

O3Pilot 的外部设计资料只作为制定、审查和更新 `DESIGN.md` 的参考，不是产品运行时依赖，也不是自动更新源。

正式关系：

```text
External Reference
≠ Runtime Dependency
≠ Automatic Design Update
```

规则：

1. O3Pilot 正式落地规则以当前 `DESIGN.md` 已写入内容为准；
2. 外部仓库、文章或 Skill 后续更新，不得自动改变 O3Pilot 的设计；
3. 需要采用外部参考的新版本时，必须人工 Review；
4. 只有 Review 后明确接受的变化，才能进入 `DESIGN.md`；
5. 外部参考与 O3Pilot 已确认设计发生冲突时，外部参考不得自动覆盖 O3Pilot；
6. 对会显著影响设计实现的参考版本，应记录 Review Date 和可追溯版本标识；
7. 外部参考不存在、仓库改名或内容删除时，不影响已经冻结在 `DESIGN.md` 中的正式规则。

v1.0 初始参考基线：

```text
Apple DESIGN.md
Repository: VoltAgent/awesome-design-md
Reference SHA: 0d37e1e5f6b621ccbdc666f6d9c82f2ada26208b
Reviewed: 2026-09-04

Emil Kowalski animate/SKILL.md
Repository: emilkowalski/skills
Reference SHA: 159fe0753228ab9d43fce274bd2c24433f3c4771
Reviewed: 2026-09-04

UI UX Pro Max
Repository: nextlevelbuilder/ui-ux-pro-max-skill
Review Basis: README / design-system methodology available on 2026-09-04
Reviewed: 2026-09-04
```

这些标识用于说明 v1.0 制定时参考了什么，不表示 O3Pilot 必须长期固定使用这些版本。

## 2.4 Motion 专项优先级

Motion 单独使用以下决策链：

```text
用户明确确认的 Motion 要求
↓
O3Pilot 120Hz / Accessibility / Performance Contract
↓
DESIGN.md 已正式采用的 O3Pilot Motion Tokens / Patterns
↓
Emil Kowalski Motion Rules
↓
已有实现
```

Apple DESIGN 与 UI UX Pro Max 不进入 Motion 决策链。

任何外部 Motion 建议如果与以下 O3Pilot 正式约束冲突：

```text
120Hz 性能目标
Reduced Motion
高频操作最小动画
数据阅读稳定性
Read-only UX
```

均不得采用。

---

# 3. Design Principles

O3Pilot 的正式 Design Principles：

```text
Quiet
Precise
Data-first
Decision-oriented
Traceable
Efficient Density
Read-only by Design
```

这些原则定义 O3Pilot 界面在视觉、信息组织和交互上的长期取向。

`Consistency / Responsiveness / Accessibility / Performance` 不作为独立视觉风格原则，而作为所有设计都必须满足的全局 Quality Attributes，由后续专项章节定义其验收标准。

## 3.1 Quiet

UI 本身不应压过业务数据。

避免：

- 大面积装饰渐变；
- 无业务意义的阴影；
- 多套品牌色；
- 过度玻璃拟态；
- 巨型 Hero；
- 大量卡片套卡片；
- 高频动画；
- 为“高级感”牺牲信息效率。

## 3.2 Precise

任何一个数字、状态和操作都必须能准确解释其含义。

页面不得为了简洁：

- 把未知显示为 0；
- 把无权限显示为空；
- 把估算显示成事实；
- 把不同币种直接相加；
- 把不同 Metric Contract 都叫“销售额”；
- 把 Ozon 官方指标和 O3Pilot 自算指标混成一个指标；
- 把当前状态与历史事实混为一体。

## 3.3 Data-first

空间优先级：

```text
Business Information
>
Controls
>
Decoration
```

分析页允许较高信息密度。

O3Pilot 不采用 Apple 营销页的超低密度布局。

## 3.4 Decision-oriented

O3Pilot 不是单纯的数据展示后台。

核心分析体验应尽可能帮助用户完成以下判断过程：

```text
State
↓
Change / Cause
↓
Risk
↓
Evidence
↓
Recommended Attention
```

对应产品层面要回答的问题：

```text
发生了什么？
为什么？
是否存在风险或异常？
接下来应该关注什么？
```

不是每个页面都必须机械展示五个固定区块，但 Dashboard、分析页和诊断页必须优先帮助用户完成判断，而不是只排列 KPI、Chart 和 Table。

页面信息层级应服务决策路径，而不是服务组件数量。

## 3.5 Traceable

当一个数字会影响经营决策时，用户应该能够继续向下看到：

```text
Value
↓
Metric Definition
↓
Status
↓
Coverage
↓
As-of Time
↓
Time Basis
↓
Source / Origin
↓
Related Entity / Raw Lineage
```

不要求所有信息默认同时展开，但必须能够追溯。

## 3.6 Efficient Density

O3Pilot 允许高信息密度，但密度必须由任务决定，而不是由视觉风格决定。

正式原则：

```text
Density follows task
not visual style
```

典型取向：

```text
Orders / Products / Inventory Table
→ High Density

Dashboard / Overview
→ Medium Density

Profit Diagnosis / Recommendation Detail
→ Medium to Low Density with stronger hierarchy

Settings / Explanatory Surfaces
→ Low to Medium Density
```

比较、筛选、批量判断需要更高密度；理解、诊断、解释和决策需要更强的信息层级与呼吸空间。

禁止为了“专业后台感”把所有页面统一压缩成极小字号、极窄行高或持续高认知负荷的布局。

## 3.7 Read-only by Design

界面必须明确体现：

> O3Pilot 提供数据、分析、模拟和建议，但不主动修改 Ozon。

正式边界：

```text
Ozon Server State
→ Read-only

O3Pilot Local State
→ Writable when the product requires it
```

因此：

```text
Read-only by Design
≠ O3Pilot UI is read-only
```

O3Pilot 可以并需要管理自身本地状态，例如：

- O3Pilot 设置；
- Seller Data / 成本数据；
- 数据导入结果；
- Alert Ack / 本地处理状态；
- Recommendation 本地状态；
- 商品内部映射；
- Forecast 参数；
- 本地标签或备注；
- Backup 配置；
- 数据源 Credential 配置。

但禁止出现会暗示直接写入 Ozon 的正式操作，例如：

```text
应用价格
立即补货
修改 Ozon 库存
调整广告预算
更新出价
发送到 Ozon
自动回复评价
自动创建促销
执行发货
```

允许：

```text
查看依据
复制建议
导出
生成本地草稿
模拟
标记 O3Pilot 本地状态
保存 O3Pilot 本地配置
```

草稿、模拟结果和建议不得通过按钮名称、成功状态或交互流程让用户误以为内容已经自动发送或应用到 Ozon。

## 3.8 Global Quality Attributes

以下属性适用于所有页面、组件和交互：

```text
Consistency
Responsiveness
Accessibility
Performance
```

它们属于全局质量门槛，而不是可选视觉风格。

后续对应章节负责定义具体尺寸、断点、Focus、Contrast、Reduced Motion、120Hz 性能目标和组件一致性规则，本章不重复这些实现细则。

---

# 4. Product UX Invariants

以下属于 UI 不可突破的数据与交互不变量。

这些规则定义 O3Pilot 在任何页面、组件、图表、表格、详情和状态展示中都不得破坏的基础语义。

## 4.1 Shop First

所有核心业务页必须存在明确 Shop Context。

支持：

```text
全部店铺
单一店铺
明确的店铺集合（未来）
```

任何 Shop-scoped 数据都不能在用户不知道当前 Shop Context 的情况下展示。

同时：

```text
All Shops
≠ Blind Aggregation
```

`全部店铺` 只是一个 Shop Context，不自动表示所有数据都可以直接跨店聚合。

例如：

- 不同币种未经 Reporting Currency 处理时不得直接相加；
- 某店铺缺失某项 Capability 时不得按 `0` 参与聚合；
- 某个 Metric Contract 不支持跨店聚合时，应按店铺分别展示；
- 某些状态类、排名类或分布类指标应保持其原有业务粒度，而不是为了总览强行求和。

## 4.2 Money Always Has Context

任何金额至少具备：

```text
amount
currency
```

跨币种聚合时必须存在：

```text
reporting_currency
```

如果没有 Reporting Currency：

```text
USD 100
CNY 500
RUB 1,000
```

必须按币种分组展示。

禁止：

```text
1600
```

跨币种转换后的金额必须能够继续追溯至少以下语义：

```text
source_amount
source_currency
reporting_currency
rate
rate_basis
rate_time
conversion_policy
```

这些信息不要求全部常驻主界面，但必须能通过 Money Details、Metric Details 或等价详情路径查看。

转换后的金额不得抹掉原始金额语义。

例如允许：

```text
¥ 12,430
Converted from ₽ 152,000
```

而不是只剩下一个无法追溯来源的换算金额。

## 4.3 Metric Origin Is Discoverable

正式 Origin：

```text
OZON_SOURCE
OZON_REFERENCE
O3P_DERIVED
O3P_ESTIMATE
O3P_FORECAST
```

展示名称：

```text
Ozon 官方
Ozon 参考口径
O3Pilot 计算
O3Pilot 估算
O3Pilot 预测
```

正式原则：

```text
Origin must always be discoverable.
Estimate / Forecast / Unverified nature must be immediately visible.
```

普通 `OZON_SOURCE / OZON_REFERENCE / O3P_DERIVED` 不要求在每个 KPI 或表格单元格旁持续显示 Badge，可通过 Tooltip、Metric Details 或其他明确详情路径保证可追溯。

以下性质会直接影响用户如何理解数字，因此必须在主要阅读层立即可识别：

```text
O3P_ESTIMATE
O3P_FORECAST
UNVERIFIED
```

不得把估算、预测或尚未验证的数据仅隐藏在深层详情中。

## 4.4 Metric Status Is Not Hidden

至少支持：

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

UI 不得将上述状态统一转换成：

```text
0
-
暂无数据
```

状态表达遵循：

```text
VALID
→ Default Quiet

PARTIAL / UNAVAILABLE / UNVERIFIED / STALE /
ESTIMATED / NO_DENOMINATOR / NO_RECENT_DEMAND /
OPEN_COHORT / NOT_APPLICABLE
→ Must Be Identifiable
```

正常数据不应因为大量绿色 `VALID` Badge 产生状态噪音；异常、限制和不完整状态则必须能够被用户直接识别。

## 4.5 Unknown ≠ Zero

实体字段与业务数据中的未知状态必须与实际数值 `0` 明确区分。

至少区分：

```text
KNOWN
NULL
UNAVAILABLE
NOT_APPLICABLE
UNMATCHED
UNVERIFIED
```

以及真实数值：

```text
0
```

例如：

```text
库存 0
```

与以下状态不是同一事实：

```text
库存未知
库存数据源不可用
该对象不适用库存
商品未匹配
数据尚未验证
```

不得为了表格整洁把这些不同状态全部显示成同一个 `—`。

当空间受限时，可以使用简短状态文本、图标或状态符号，但必须存在可访问的完整解释路径。

## 4.6 Freshness Is Part of the Value

对时效敏感的数据，数值本身与其新鲜度共同构成用户所理解的事实。

正式原则：

```text
Value without relevant freshness context
can be misleading.
```

例如：

```text
库存 37
```

如果该值是 5 分钟前获取，与 3 天前最后一次成功同步，不应在 UI 中呈现为同等可信的“当前库存”。

规则：

- 正常且新鲜的数据可以保持视觉安静；
- `STALE` 必须明确可识别；
- 对库存、价格、订单状态、广告状态等时间敏感数据，用户必须能够看到或继续查看 `as_of_time`；
- 同步失败后继续保留最后有效值时，必须明确这是 Last Known Value，而不能伪装为当前值；
- 数据源长期未更新、覆盖断档或同步失败，应通过统一 Freshness / Coverage 表达暴露，而不是由各页面自行猜测。

## 4.7 No Silent Fallback

数据源替代、缓存回退、估算、降级或其他 fallback 不得静默发生。

正式原则：

```text
Fallback
≠ Same Truth
```

例如禁止：

```text
Performance API 无数据
→ 静默改用其他来源并继续显示为同一指标

API 同步失败
→ 静默显示旧缓存并伪装成当前数据

Ozon 官方报告未导入
→ 静默改用估算值
```

允许 fallback 的前提是：

- 上游 Contract 已允许该 fallback；
- fallback 的来源、状态或性质可识别；
- 不改变 Metric Contract；
- 不把估算或替代来源伪装成原始事实；
- 用户能够继续追溯实际使用的数据来源。

## 4.8 Current State ≠ Historical Fact

当前状态与历史事实必须保持明确区分。

例如：

```text
当前售价
当前商品标题
当前库存
当前佣金规则
当前物流状态
```

不能被 UI 误表达为：

```text
历史订单成交价
历史订单商品快照
历史库存快照
历史应计费率
某个历史时间点的物流状态
```

只要一个页面同时涉及 Current 与 Historical 数据，就必须通过字段命名、标签、时间、区域或信息层级明确区分。

禁止因为实体已经更新，就用当前值覆盖或替代历史展示中的原始事实。

## 4.9 Time Basis Must Be Discoverable

时间分析必须知道使用的是哪种业务时间。

例如：

```text
下单时间
发货时间
签收时间
退货创建时间
财务应计日期
广告业务日期
Delivery Cohort
```

常规页面可将 Time Basis 放在 Tooltip / Metric Details，但不可完全丢失。

当同一页面同时使用多个 Time Basis 时，必须避免让用户误以为所有图表、KPI 和表格都使用同一日期口径。

## 4.10 Default Display Timezone

默认前端展示时区：

```text
Asia/Shanghai
UTC+8
```

如果展示来源业务日，应明确保留其业务日期语义。

特别是 `BUSINESS_DATE`、广告业务日期、财务业务日期等 date-only 值，不应被错误当作 UTC Timestamp 再进行时区转换，从而导致日期跨天。

正式区分：

```text
Timestamp
→ 可按 Display Timezone 转换

Business Date / Date-only Semantic
→ 保留来源业务日期语义
```

## 4.11 Session Revoked

当当前 Session 按 `SECURITY.md` 定义的 Session 语义失效或被撤销：

- 不弹普通 Error Toast；
- 不保留半可操作页面；
- 立即进入 Session Revoked 状态；
- 引导重新登录；
- 不泄漏原页面敏感数据。

Session 撤销原因由 `SECURITY.md` 定义；`DESIGN.md` 只负责失效后的 UI 投影。

---

# 5. Information Architecture

Capability Map 不等于页面清单。

一级导航只对应长期稳定的用户任务和业务域。

O3Pilot 的正式 IA 使用稳定的 Navigation Semantic ID 表达产品结构，界面显示名称只是本地化 Label。

正式关系：

```text
Navigation Semantic ID
≠ Localized Display Label
```

因此，中文、英文、俄语界面可以使用不同显示名称，但不能因为翻译变化而改变 IA 本身。

正式 IA：

```text
O3Pilot

overview → 总览

经营
├── sales       → 销售
├── products    → 商品
├── orders      → 订单
├── inventory   → 库存
├── fulfillment → 履约
└── returns     → 退货

增长
├── advertising    → 广告
└── buyer-feedback → 买家反馈

财务
├── settlements → 结算
└── profit      → 利润

风险
├── shop-health → 店铺健康
└── alerts      → 预警

决策
└── recommendations → 经营建议

数据
└── data-center → 数据中心

系统
└── settings → 系统设置
```

Semantic ID 应保持稳定；显示名称可以随着本地化、术语优化或文案调整而变化。

例如：

```text
alerts
→ zh-CN: 预警
→ en: Alerts
→ ru: Оповещения
```

其中 `settlements → 结算` 是财务工作空间的用户可见名称，不限制其内部只展示狭义结算记录。该工作空间仍可覆盖应计、平台费用、结算、Payout 与相关财务事实，但不应使用重复的 `财务 > 财务` 导航命名。

`buyer-feedback → 买家反馈` 用于承载评价、评分、反馈趋势、问题归因和本地回复草稿等买家反馈分析能力；不表示 O3Pilot 可以向 Ozon 自动发送回复。

## 5.1 导航层级

O3Pilot 明确区分三种导航层级：

```text
Primary Navigation
→ 稳定业务工作空间

Page-local Navigation
→ Tabs / Views / Filters / Detail

Global Utilities
→ Search / Data Status / Alerts Entry / Shop Context
```

### Primary Navigation

左侧 Sidebar 只承载长期稳定的业务工作空间。

它回答：

> 用户现在正在处理哪个业务领域？

Primary Navigation 不用于承载单一分析维度、技术任务、一次性工作流或实体详情。

### Page-local Navigation

页面内部的 Tabs、Views、Filters、Detail、Drawer 等负责承载业务域内的细分任务。

例如：

```text
库存
├── 当前库存
├── Forecast
├── 补货
└── 清库存分析
```

上述能力可以属于同一 `inventory` 工作空间，而不需要分别成为 Sidebar 一级页面。

### Global Utilities

Global Utilities 是跨业务域的全局工具，不作为第二套主导航。

例如：

```text
Shop Context
Reporting Currency
Global Search
Data Status
Alerts Entry
```

其中 `Alerts Entry` 可以作为全局快速入口，但正式业务工作空间仍然是 `alerts`。

## 5.2 一级导航准入规则

新增能力只有在同时满足以下条件时，才应考虑成为独立一级页面：

1. 对应独立且高频或持续存在的用户任务；
2. 具有稳定、清晰的业务心智模型；
3. 内容规模和交互复杂度足以形成独立工作空间；
4. 不只是某个实体详情、过滤视图、报表维度或单一指标；
5. 不只是数据源、同步任务、导入任务或技术基础设施；
6. 放入现有业务域会明显损害可发现性、理解成本或任务效率。

如果不满足，应优先采用：

```text
现有页面 Tab
→ View / Filter
→ Detail
→ Drawer / Modal
→ Data Center
```

而不是新增 Sidebar 项。

任何新增一级导航都必须同时更新：

- Navigation Semantic ID；
- 本地化 Label；
- Page Template；
- Sidebar Icon Mapping；
- Global Search / Command Palette；
- Responsive 行为；
- 权限或 Capability 不可用时的状态表达。

## 5.3 不建立独立一级页面的能力

以下能力默认不是一级导航：

```text
取消
价格
佣金
促销
搜索词
Forecast
补货
清库存
广告建议
定价建议
商品 SEO 建议
数据导入
Jobs
Data Quality
Lineage
Webhook Event
```

它们分别进入已有稳定业务域。

例如：

```text
取消 → 订单 / 店铺健康 / 预警
价格 → 商品详情
Forecast → 库存 / 经营建议
数据导入 → 数据中心 > Imports
Jobs → 数据中心 > Jobs
Data Quality → 数据中心 > Quality
Lineage → 数据中心 > Lineage
Webhook Event → 数据中心 / 系统设置中的诊断视图
```

这些映射是信息架构归属，不重新定义相关业务能力或数据 Contract。

---

# 6. Global Application Shell

O3Pilot 的全局 Shell 由四个长期稳定的区域组成：

```text
App Shell
├── Sidebar
├── Global Context Bar
├── Page Header
└── Page Content
```

职责边界：

```text
Sidebar
→ Primary Navigation

Global Context Bar
→ 全局 Context、全局状态与全局工具

Page Header
→ 当前页面标题、时间范围、比较、页面级 Filter 与页面级操作

Page Content
→ KPI / Chart / Table / Detail / Analysis
```

Desktop：

```text
┌────────────────┬─────────────────────────────────────────────────────┐
│                │ Global Context Bar                                  │
│                ├─────────────────────────────────────────────────────┤
│                │ Page Header                                         │
│    Sidebar     ├─────────────────────────────────────────────────────┤
│    248px       │                                                     │
│                │                  Page Content                       │
│                │                                                     │
│                │                                                     │
└────────────────┴─────────────────────────────────────────────────────┘
```

Collapsed：

```text
┌──────┬───────────────────────────────────────────────────────────────┐
│      │ Global Context Bar                                            │
│ 72px ├───────────────────────────────────────────────────────────────┤
│      │ Page Header                                                   │
│ Icon ├───────────────────────────────────────────────────────────────┤
│ Rail │                       Page Content                            │
│      │                                                               │
└──────┴───────────────────────────────────────────────────────────────┘
```

## 6.1 Shell Regions

四个区域必须保持明确职责，不通过临时页面需求互相侵占。

正式原则：

```text
Global Context
≠ Page Context
≠ Page Content
```

普通页面不得为了节省垂直空间，把页面级 Filter、Export、Refresh、Breadcrumb 或业务操作长期塞入 Global Context Bar。

同样，Shop Context、Reporting Currency、Global Search、Data Status 等全局能力不应在每个页面重复实现一套独立控件。

## 6.2 Global Context Bar

职责仅限全局 Context、全局状态和全局工具，不作为第二套主导航。

正式高度：

```text
64px
```

与 Sidebar Top Area 形成同一水平基线。

正式结构：

```text
Left
[Shop Context ▼] [Reporting Currency ▼]

Right
[Global Search] [Data Status] [Alerts]
```

`Reporting Currency` 至少应支持：

```text
Original currencies / 不转换
CNY
USD
RUB
...
```

当选择 `Original currencies / 不转换` 时，多币种金额继续按原始币种分别显示，不允许为了形成单一总数而自动换算或相加。

Global Context Bar 不承载普通页面级：

```text
Date Range
Comparison
Page Filter
Export
Refresh
Breadcrumb
Page Action
```

除非未来存在真正影响整个应用的全局时间 Context，否则 v1 的 `Date Range / Comparison` 一律属于 Page Header 或页面内部分析控件。

窄屏时可以压缩控件表现，例如：

```text
Shop Context    → compact label / icon
Currency        → compact selector
Global Search   → icon trigger
Data Status     → icon / status indicator
Alerts          → icon
```

但功能和可发现性不能消失。

## 6.3 Page Header

Page Header 管理当前任务 Context。

典型内容：

```text
Page Title
Optional Subtitle / Context Description
Date Range
Comparison
Page Filters
Page Tabs
Page-local Actions
```

不是每个页面都必须出现全部内容。

Page Header 应根据页面复杂度保持紧凑，不定义所有页面统一固定高度。

常规顺序优先为：

```text
Title / Context
↓
Primary Page Controls
↓
Tabs / Secondary Controls when needed
↓
Page Content
```

如果某项控制只影响当前页面、当前数据集或当前分析视图，默认属于 Page Header 或 Page Content，而不是 Global Context Bar。

## 6.4 Scroll Model

正式滚动模型：

```text
Sidebar
→ fixed / independently stable

Global Context Bar
→ sticky

Page Header
→ sticky only when the page benefits from persistent controls

Page Content
→ primary vertical scroll surface
```

普通页面应尽量只有一个主要纵向滚动容器。

禁止无明确需求地形成：

```text
Browser Scroll
+
Page Scroll
+
Card Scroll
+
Table Body Scroll
```

这种多层纵向滚动结构。

允许：

- 大型表格横向滚动；
- Drawer / Modal 自身必要的独立滚动；
- 数据规模要求下的虚拟列表；
- 明确具有独立阅读语义的局部滚动区域。

但这些都不能破坏主要页面滚动路径和键盘导航顺序。

## 6.5 Content Layout Modes

页面不应各自随意定义最大宽度，而应选择正式 Layout Mode。

### Data Canvas

用于高密度分析、宽表格、矩阵和需要完整横向空间的页面：

```text
width: fluid
max-width: none
```

典型：

```text
Orders
Products
Inventory
Advertising analytics
Finance tables
```

### Standard Content

用于设置、配置、常规表单和中等复杂度工作区：

```text
max-width: 1200px
```

### Reading / Detail

用于长文本解释、诊断详情和更强调阅读层级的页面：

```text
max-width: 960px
```

页面的 Layout Mode 应由任务和信息结构决定，而不是为了模仿某种视觉风格。

Table / Chart 不应因为父级采用较窄模式而被不合理压缩；需要高密度数据时，应切换到 `Data Canvas` 或使用合理的局部宽屏结构。

禁止为了 Apple 风格把数据表强制限制在窄列中。

## 6.6 Responsive Shell Contract

第 31 章定义完整响应式规则；本节只定义 Shell 不变量。

```text
Desktop
→ Sidebar expanded or collapsed

Compact
→ Sidebar normally collapsed to icon rail

Narrow
→ Sidebar remains visible as narrow icon rail
→ Global controls use compact representation
→ Page Content becomes one primary column where the task permits
```

正式规则：

```text
Narrow
≠ Hamburger-only Navigation
```

即使在窄屏，Primary Navigation 仍保留图标 Rail，不允许完全隐藏成只能通过 Hamburger 打开的临时导航。

响应式可以改变：

- Label 是否常驻；
- 控件宽度；
- Page Header 换行；
- Chart / Table 布局；
- Detail 是否使用全屏层；

但不得改变：

- 当前 Shop Context 的可发现性；
- Reporting Currency Context；
- Primary Navigation 可访问性；
- Global Search 入口；
- Data Status / Alerts 的关键状态可发现性。

---

# 7. Sidebar

Sidebar 是 O3Pilot 的 Primary Navigation Surface。

它必须长期保持位置稳定、语义稳定和交互稳定，不因店铺权限、数据缺失、同步状态或局部页面状态而随意改变结构。

## 7.1 Dimensions

```text
Expanded width       248px
Collapsed width        72px
Narrow icon rail       60px
Top area height         64px
Desktop item height     40px
Touch / Narrow target ≥ 44px
Icon size               18–20px
```

桌面环境优先保持高信息密度；Touch / Narrow 环境必须保证可可靠触发的点击目标，不因压缩布局而降低可操作性。

## 7.2 Structure and Groups

Sidebar 的正式结构：

```text
Brand / Product Area
↓
Primary Navigation Groups
↓
Collapse Control
```

Primary Navigation Groups 使用第 5 章定义的稳定业务分组，例如：

```text
经营
  销售
  商品
  订单
  库存
  履约
  退货

增长
  广告
  买家反馈

...
```

Group Label 是结构标签，不是交互入口。

正式原则：

```text
Primary Navigation Groups
→ structural labels
→ not interaction gates
```

因此默认不使用 Accordion / Disclosure 将整个业务分组折叠隐藏。

一级页面数量不足以形成明显信息过载时，不得为了视觉整洁增加一次无业务价值的展开点击。

## 7.3 Expanded State

Expanded 状态显示：

```text
[Icon]  商品
```

要求：

- 图标与标签同时可见；
- Label 使用当前 Locale 的正式显示名称；
- Group Label 可见；
- Active 状态清晰但克制；
- 不使用高饱和大面积背景制造导航噪音。

Active 推荐使用浅 Surface / Ink 变化 / 适度字重差异。

不使用：

- 左侧粗彩色条 + 高饱和背景的双重强调；
- 强阴影；
- 渐变；
- 大范围 Glow。

## 7.4 Collapsed State

Desktop Collapsed：

```text
[Icon]
```

必须：

- 宽度保持 `72px`；
- 保留每个 Primary Navigation Item；
- 保留 Active 状态；
- Hover / Focus 显示 Tooltip；
- Tooltip 包含完整本地化名称；
- 支持键盘 Focus；
- Group Label 隐藏，但保留足够分组间距形成结构记忆；
- Collapse 后不得变成不可发现的 Hamburger-only Navigation。

## 7.5 Narrow / Touch State

Narrow 环境保留永久 `60px` Icon Rail。

正式关系：

```text
Narrow
≠ Hamburger-only Navigation
```

基础行为：

```text
60px permanent rail
→ icon tap navigates directly
```

允许提供 Expand Control，将 Sidebar 临时展开为覆盖 Page Content 的 Label View，以提升 Touch 环境下的图标可发现性。

但临时展开不得改变以下事实：

```text
Base 60px Rail
→ always exists in Narrow shell
```

Narrow 状态下：

- 导航项点击目标至少 `44px` 高；
- 不依赖 Hover 才能知道导航项含义；
- 临时展开层关闭后恢复 Rail；
- 页面内容不因临时展开而发生复杂重排动画。

## 7.6 Active Route State

任一时刻只能有一个 Primary Navigation Item 处于 Active 状态。

```text
Only one primary nav item
may be active at a time.
```

Active 由当前 Primary Workspace 决定，而不是只由当前 URL 的最末级路径决定。

例如：

```text
/products
/products/123
/products/123/history
```

均应保持：

```text
products → 商品 → Active
```

打开 Detail Drawer、子路由或 Page-local Tab 时，不应丢失父级 Primary Navigation Context。

语义实现应使用普通网页导航模式，例如：

```html
<nav>
  <a aria-current="page">...</a>
</nav>
```

Sidebar 不应被实现为桌面应用式 `menu / menuitem` 控件，除非真实交互语义发生变化。

## 7.7 Navigation Stability

Primary Navigation 的存在由正式 IA 决定，不由当前店铺是否有数据决定。

正式原则：

```text
Capability Availability
≠ Navigation Existence
```

例如某个 Shop 暂无买家反馈权限时：

```text
买家反馈
→ 仍存在于 Sidebar
→ 页面显示 UNAVAILABLE / ACCESS_DEPENDENT / PARTIAL 等正式状态
```

禁止：

```text
切换 Shop
→ 一级导航项静默消失 / 出现
```

以下情况不应静默改变 Sidebar IA：

- Shop Capability 不可用；
- API 暂无权限；
- 数据尚未同步；
- Source 暂时失败；
- 当前结果为空；
- 某个 Metric 不适用。

只有正式产品版本改变 IA 时，才允许增删 Primary Navigation Item。

## 7.8 Collapse Preference

用户主动选择的 Desktop Sidebar 状态属于 O3Pilot 本地 UI Preference。

```text
User preference
→ remembered
```

例如：

```text
Expanded
Collapsed
```

Responsive Constraint 可以临时覆盖当前呈现方式，但不得破坏已保存偏好：

```text
Desktop preference: Expanded
↓
Window becomes Compact
↓
Temporarily present Collapsed Rail
↓
Window returns to Desktop
↓
Restore Expanded
```

Preference 属于 O3Pilot Local State，不涉及 Ozon 写操作。

## 7.9 Nav Click Morph Feedback

用户明确触发 Primary Navigation 后，图标使用以下视觉反馈：

```text
Original Icon
↓
Morph to Check
↓
Brief confirmation
↓
Morph back to Original Icon
```

目标总时长：

```text
约 180–220ms
```

该动画只用于确认用户操作已被接受。

最重要的行为规则：

```text
Navigation
→ immediate

Morph
→ non-blocking visual feedback
```

即：

- 路由切换不得等待 Morph 完成；
- 不等待页面数据加载完成；
- 不与全页 Slide Transition 叠加；
- Programmatic Navigation 不执行该 Morph；
- Keyboard Activation 可以显示反馈，但不得延迟导航；
- 连续快速导航时，前一次 Morph 允许取消并进入当前目标状态；
- Rapid repeated activation 不要求排队播放全部动画；
- `prefers-reduced-motion` 时使用短暂 Check / opacity 状态，不执行明显空间 Morph。

## 7.10 Accessibility and Semantics

Sidebar 必须满足：

- Primary Navigation 使用正确 `<nav>` landmark；
- Active Link 使用 `aria-current="page"` 或等效网页导航语义；
- 所有图标型 Navigation Item 有可访问名称；
- Collapsed 状态不能只依赖 Tooltip 为辅助技术提供名称；
- Keyboard Focus 顺序与视觉顺序一致；
- Focus Ring 不得被容器裁剪；
- Group Label 不应进入不必要的 Tab 顺序；
- Tooltip 不能取代 link 的 accessible name；
- Motion Feedback 不得成为理解当前 Active State 的唯一方式。

## 7.11 Sidebar Scroll Model

Sidebar 采用固定上下区域 + 中间 Navigation Scroll Region：

```text
┌────────────────────┐
│ Brand / Logo 64px  │
├────────────────────┤
│                    │
│ Navigation         │
│ primary scroll     │
│                    │
├────────────────────┤
│ Collapse Control   │
└────────────────────┘
```

只有 Navigation Region 在内容高度不足时允许独立纵向滚动。

Brand / Product Area 与 Collapse Control 不应因为 Navigation 内容增长而被滚出视图。

Sidebar 不承载：

- 同步日志；
- 数据源详情；
- Job 列表；
- 大量业务数字 Badge；
- 快捷操作面板；
- 数据质量详情；
- 页面级 Filter。

这些内容分别属于 Global Context Bar、Data Center、Page Header 或 Page Content。

---

# 8. Typography

Typography 是 O3Pilot 的信息结构基础，不只是字体和字号配置。

正式排版系统必须同时满足：

```text
Chinese
English / Latin
Russian / Cyrillic
Numeric data
Currency
Long identifiers
Dense analytical UI
Accessibility
```

任何页面不得为了塞入更多信息而绕过本章 Token、偷偷缩小字号或依赖非正式字体 fallback。

## 8.1 Font Family and Local Delivery

正式字体：

```css
font-family:
  "MiSans Global",
  system-ui,
  -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  sans-serif;
```

`MiSans Global` 是 O3Pilot 中文、英文、俄语、数字和常用符号的正式字体体系。

`system-ui` 等只作为字体资源加载失败时的安全 fallback，不作为正常界面的第二套字体设计语言。

正式字体资源：

- 随 O3Pilot Release 本地提供；
- 不使用 Google Fonts；
- 不依赖 CDN；
- 不执行 Runtime Font Fetch；
- 不因为系统已安装某个字体就跳过 O3Pilot 自带字体资源；
- 不允许页面或 Feature 私自指定另一套 UI Font。

正常加载路径必须避免明显 FOUT / Layout Shift；字体加载失败时界面仍应可用，但应被视为可诊断的资源异常，而不是正常视觉状态。

## 8.2 Font Weight Contract

只允许：

```text
400 Regular
500 Medium
600 Semibold
700 Bold
```

禁止：

```text
300
800
900
未经定义的可变字重
```

请求的 Font Weight 必须映射到真实提供的 MiSans Global 字重文件，不依赖浏览器生成伪粗体或伪斜体。

正式规则：

```text
Requested Weight
→ Actual supported MiSans Global weight

Missing Weight
→ Safe fallback / diagnosable failure
→ Never synthetic bold or italic
```

全局实现应禁止 Synthetic Font Styling，例如：

```css
font-synthesis: none;
```

字重表达层级，而不是装饰：

```text
400 → Regular reading
500 → Control / moderate emphasis
600 → Heading / Metric emphasis
700 → Rare strong emphasis only
```

`700` 不作为普通标题、普通按钮或表格内容的默认字重。

## 8.3 Glyph Coverage

正式字体包必须验证至少覆盖：

```text
Simplified Chinese
English / Latin
Russian / Cyrillic
Arabic numerals
Common punctuation
O3Pilot commonly used currency symbols
Common analytical symbols
```

至少验证以下代表性字符：

```text
中文测试
O3Pilot Seller Analytics
Данные магазина
0123456789
¥ $ ₽
% ±
→ ↑ ↓
…
/ - _ . , : ; ( ) [ ]
```

如果正式界面中的常用字符在正常字体加载情况下偷偷 fallback 到另一字体，应视为 Typography QA Failure。

Emoji 不属于 O3Pilot 正式 UI Icon System，不通过字体 Emoji 替代 Lucide / Semantic Icon Registry。

## 8.4 Semantic Type Scale

O3Pilot 使用 Semantic Typography Token，不把 HTML Heading Level 与视觉字号机械绑定。

正式关系：

```text
HTML semantics
≠ Visual typography token
```

页面仍必须使用正确的 `h1 / h2 / h3...` 文档层级和可访问性语义，但视觉表现由以下正式 Token 决定：

| Token | Size | Weight | Line Height | Use |
|---|---:|---:|---:|---|
| `page-title` | 28px | 600 | 1.25 | 页面主标题 |
| `section-title` | 22px | 600 | 1.30 | 主分析区块标题 |
| `panel-title` | 17px | 600 | 1.35 | Panel / Card / Drawer 区块标题 |
| `body` | 15px | 400 | 1.50 | 常规正文与主要说明 |
| `body-medium` | 15px | 500 | 1.45 | 控件、强调、重要 Label |
| `small` | 13px | 400/500 | 1.45 | 辅助文本、次级 Label |
| `micro` | 12px | 400/500 | 1.40 | Source / Timestamp / Meta |
| `metric-xl` | 32px | 600 | 1.15 | 少量 Dashboard 核心 KPI |
| `metric-lg` | 24px | 600 | 1.20 | 常规 KPI / 重要金额 |
| `metric-md` | 17px | 600 | 1.25 | Dense KPI / Table Summary |

原则：

- `body` 的正式基础字号保持 `15px`；
- 不通过把全站正文压到 `14px` 或更小来获得“专业后台感”；
- `micro 12px` 只用于 Source、Timestamp、Meta 等辅助信息；
- 主要业务数据、Form Value、状态解释和正文不得默认使用 `micro`；
- `metric-xl` 必须克制，只用于极少数真正需要最高视觉优先级的指标；
- 同一页面不应同时滥用多个超大 Metric 层级。

信息密度优先通过以下手段控制：

```text
Row height
Spacing
Column layout
Information hierarchy
Progressive disclosure
```

而不是持续缩小字体。

## 8.5 Numeric Typography

用于比较的金额、百分比、数量、Duration、时间序列数值和表格数值默认使用 Tabular Numerals：

```css
font-variant-numeric: tabular-nums;
```

需要时可补充：

```css
font-feature-settings: "tnum";
```

正式原则：

```text
Comparable numeric columns
→ right aligned
→ tabular nums

Standalone KPI
→ alignment follows layout
→ still uses stable numeric glyphs where useful
```

数字对齐规则：

- 可逐行比较的 Numeric Column 默认右对齐；
- 金额列的 Currency / Amount 排版必须保持可扫描性；
- Percentage / Rate / Count 使用一致的小数精度策略；
- 单独 KPI 不因为“是数字”就强制右对齐；
- 时间戳在表格中的对齐方式应全列一致；
- 负数、正数和零不得因比例字体导致明显水平跳动。

数值格式与 Locale Format 可以变化，但不得改变底层业务值和 Metric Contract。

## 8.6 Identifier Typography

正式区分：

```text
Numeric Value
≠ Numeric-looking Identifier
```

以下属于 Identifier，而不是 Numeric Metric：

```text
order_number
posting_number
SKU
sku
offer_id
product_id
Campaign ID
Import Batch ID
Job ID
```

例如：

```text
0155167697-0113   → 母订单号
0155167697-0113-1 → 订单号
```

Identifier 必须保留原始文本语义。

禁止：

- 删除前导 `0`；
- 添加千位分隔符；
- 按 Locale 改写格式；
- 当成金额或普通数字格式化；
- 因看起来像数字而进行 Numeric Arithmetic；
- 使用会改变字符内容的自动格式化。

Identifier 的列排序必须遵守对应实体的正式排序语义；不能默认假设 Numeric Sort 就是正确业务顺序。

Identifier 可以使用 Tabular Numerals 提高扫描稳定性，但这不改变它的字符串身份。

## 8.7 Wrapping and Truncation

O3Pilot 正式支持中文、英文和俄语，因此文本长度变化必须由布局吸收，而不是通过临时缩小字体解决。

正式原则：

```text
Long text
→ wrap or truncate intentionally

Never
→ silently shrink typography below token
```

### 默认不得语义截断

以下内容正常情况下不得因为布局便利而只留下不可理解的截断文本：

```text
Primary Metric Value
Critical Status
Form Error
Alert Severity / Critical Message
Page Title
Security / Session State
```

### 允许密集布局中单行省略

例如：

```text
Long Product Name in dense table
Long Shop Name
Secondary Description
Non-critical secondary label
```

允许：

```text
single line
+
ellipsis
+
full value discoverable by Tooltip / Detail / accessible name
```

### Identifier

`posting_number / order_number / SKU / offer_id / product_id` 等 Identifier 优先保持：

```text
single line
no semantic truncation when space permits
```

如果空间确实不足而必须截断：

- 必须提供查看完整值的路径；
- 必须能够复制完整原始值；
- Accessible Name / Title 不得只包含截断后的字符串；
- 不应截掉后让多个不同 Identifier 在视觉上变得不可区分。

俄罗斯语或英文长词不得通过把字号压低到 Token 以下“硬塞”进固定宽度组件。

必要时应优先：

```text
Grow control
Wrap
Reflow
Use tooltip / detail
Use responsive layout
```

## 8.8 Typography Accessibility / QA

正式 Typography QA 至少覆盖：

```text
Chinese
English
Russian
Mixed CN + EN + Numbers
Mixed Cyrillic + Numbers
Currency symbols
Percent / signed values
Long product names
Long shop names
Long identifiers
Dense tables
200% browser zoom
```

验收要求：

- MiSans Global 实际加载；
- 只使用正式字重；
- 无 Synthetic Weight；
- 无常用字符意外 fallback；
- 200% Browser Zoom 下主要信息仍可读取和操作；
- 长英文 / 俄语不因固定高度被裁切；
- 字体放大时不出现正文互相覆盖；
- `micro` 不承担主要业务信息；
- 表格 Numeric Column 仍保持可比较性；
- Identifier 保持完整业务语义；
- 文字颜色对比度遵守 Accessibility Contract。

---

# 9. Color System

O3Pilot 的颜色系统用于建立层级、表达交互和传递经过业务语义确认的状态，不承担装饰性品牌渲染。

正式原则：

```text
Action Color ≠ Decoration
Direction ≠ Outcome
Status Palette ≠ Chart Palette
Normal → Quiet
Attention → Color
```

颜色永远不能单独承担完整业务语义；状态、Origin、趋势方向和风险必须同时存在文字、Icon、Label 或其他可访问表达。

## 9.1 Color Principles

颜色使用遵循以下优先级：

```text
Structure
→ Neutral / Surface / Border

Interaction
→ Action Color

Business Attention
→ Semantic Status Color

Data Series
→ Chart Palette
```

禁止为了“重要”而随意把 KPI、标题、Card、数字或大面积背景改成 Action Blue。

正式关系：

```text
Importance
≠ Blue

Normal
≠ Green everywhere

Difference
≠ Different semantic color
```

## 9.2 Interaction Colors

Light Theme 正式 Interaction Tokens：

```text
Action                 #0066CC
Action Hover           #0071E3
Action Pressed         #0055B3
Action On Dark         #2997FF
On Action              #FFFFFF
Focus Ring             #0071E3
Selection Subtle       rgba(0,102,204,0.08)
```

全局只有一套主要交互色。

`Action Blue` 仅用于真实可交互元素、Focus、Selection 和必要的交互强调，例如：

- Link；
- Primary Action；
- Selected Control；
- Focus Ring；
- Active interactive affordance。

不用于：

- 普通 KPI 数字；
- 装饰性标题；
- 无交互 Card；
- 普通 Chart Series；
- 仅因为内容“重要”而着色。

Dark Theme 中，普通实心 Primary Action 仍应使用能够与前景文本满足对比度的正式 Action Token；`#2997FF` 主要用于 Dark Surface 上的 Link / Focus / 高可见交互强调，不自动作为所有实心按钮背景。

## 9.3 Neutral Colors

### Light Theme

```text
Canvas                #FFFFFF
Canvas Secondary      #F5F5F7
Surface               #FFFFFF
Surface Subtle        #FAFAFC
Ink                   #1D1D1F
Text Secondary        #6E6E73
Text Tertiary         #86868B
Hairline              rgba(0,0,0,0.08)
Divider Soft          rgba(0,0,0,0.05)
Control Hover         rgba(0,0,0,0.04)
Selected Surface      rgba(0,102,204,0.08)
Disabled Text         rgba(29,29,31,0.38)
Disabled Surface      rgba(0,0,0,0.04)
Overlay / Scrim       rgba(0,0,0,0.28)
```

### Dark Theme

```text
Canvas                #000000
Canvas Secondary      #1D1D1F
Surface               #1C1C1E
Surface Subtle        #242426
Ink                   #F5F5F7
Text Secondary        #A1A1A6
Text Tertiary         #8E8E93
Hairline              rgba(255,255,255,0.12)
Divider Soft          rgba(255,255,255,0.08)
Control Hover         rgba(255,255,255,0.06)
Selected Surface      rgba(41,151,255,0.14)
Disabled Text         rgba(245,245,247,0.38)
Disabled Surface      rgba(255,255,255,0.06)
Overlay / Scrim       rgba(0,0,0,0.48)
```

Neutral Token 应保持数量克制。

正式原则：

```text
Few neutral levels
+
clear semantic roles
```

不得为了细微视觉差异建立大量难以区分、缺乏明确职责的灰色 Token。

## 9.4 Semantic Status Colors

Semantic Color 表达已经确定的业务状态，不是品牌色，也不是普通分类色。

正式 Namespace：

```text
color.status.success.*
color.status.warning.*
color.status.danger.*
color.status.information.*
color.status.neutral.*
```

每个状态至少定义：

```text
Foreground
Subtle Background
Border / Indicator
```

### Light Theme

| Semantic | Foreground | Subtle Background | Border / Indicator |
|---|---|---|---|
| `Success` | `#1F7A3D` | `rgba(31,122,61,0.10)` | `rgba(31,122,61,0.28)` |
| `Warning` | `#946200` | `rgba(148,98,0,0.10)` | `rgba(148,98,0,0.28)` |
| `Danger` | `#B42318` | `rgba(180,35,24,0.10)` | `rgba(180,35,24,0.28)` |
| `Information` | `#0066CC` | `rgba(0,102,204,0.08)` | `rgba(0,102,204,0.24)` |
| `Neutral` | `#6E6E73` | `rgba(110,110,115,0.08)` | `rgba(110,110,115,0.22)` |

### Dark Theme

| Semantic | Foreground | Subtle Background | Border / Indicator |
|---|---|---|---|
| `Success` | `#6CCB82` | `rgba(108,203,130,0.14)` | `rgba(108,203,130,0.34)` |
| `Warning` | `#FFD166` | `rgba(255,209,102,0.14)` | `rgba(255,209,102,0.34)` |
| `Danger` | `#FF7B72` | `rgba(255,123,114,0.14)` | `rgba(255,123,114,0.34)` |
| `Information` | `#5AC8FA` | `rgba(90,200,250,0.14)` | `rgba(90,200,250,0.34)` |
| `Neutral` | `#A1A1A6` | `rgba(161,161,166,0.12)` | `rgba(161,161,166,0.28)` |

规则：

- 同一语义全站使用同一 Token；
- 不使用颜色单独传递信息；
- Status 同时提供文字、Icon 或其他明确语义；
- `Danger` 保留给真实错误、风险和负向业务状态，不作为普通强调色；
- `VALID / Healthy / Normal` 默认使用安静的 Neutral 表达，不因“正常”而在全站铺满 Success Green；
- Success Green 主要用于明确确认成功、Connection Test、Health Checklist 等真正需要确认成功的场景。

正式原则：

```text
Normal
→ Quiet

Attention
→ Semantic Color
```

## 9.5 Direction vs Business Outcome

趋势方向与业务结果不是同一语义。

正式原则：

```text
Direction
≠ Outcome
```

必须区分：

```text
Change Direction
→ ↑ / ↓ / + / -

Business Meaning
→ Positive / Negative / Neutral
```

例如：

```text
销售额 ↑
→ 可能 Positive

产品退货率 ↑
→ Negative

取消率 ↑
→ Negative

库存 ↑
→ 可能 Positive / Negative / Neutral

广告花费 ↓
→ 不能仅凭方向判断
```

因此禁止建立全局规则：

```text
↑ = Green
↓ = Red
```

只有 Metric Contract、业务规则或明确分析上下文已经确定变化的业务意义时，才能将趋势变化映射到 `Success / Danger / Warning / Neutral`。

箭头表达方向，Semantic Color 表达经过判断后的业务含义。

## 9.6 Origin Color Rules

Metric Origin 与 Semantic Status 是不同维度。

以下 Origin：

```text
OZON_SOURCE
OZON_REFERENCE
O3P_DERIVED
O3P_ESTIMATE
O3P_FORECAST
```

不得机械映射成：

```text
Success
Warning
Danger
```

默认规则：

- `OZON_SOURCE / OZON_REFERENCE / O3P_DERIVED` 使用 Neutral 为主；
- `O3P_ESTIMATE / O3P_FORECAST` 必须依靠明确 Label 表达性质；
- 如果具体 Metric Status 同时要求 Warning / Danger，则由 Status 决定 Semantic Color；
- 不因为“Ozon 官方”而使用 Success Green；
- 不因为“预测”而默认使用 Warning Yellow。

正式原则：

```text
Color
≠ Origin Label
```

Color 不能替代 Origin 文本或可访问名称。

## 9.7 Chart Color Separation

Chart Palette 与 Semantic Status Palette 必须完全分层管理。

正式 Namespace：

```text
color.chart.*
≠
color.status.*
```

例如普通产品、店铺、渠道、Campaign Series 不得直接使用：

```text
Success Green
Warning Yellow
Danger Red
```

作为无语义的分类颜色。

否则用户会把普通 Series 误解为成功、风险或错误。

`Danger` 红色尤其保留给真实负向状态和需要注意的异常。

第 22 章负责定义图表具体 Palette、Series 数量和可访问性；本节只冻结 Status 与 Chart 的职责边界。

## 9.8 Light / Dark Theme Contract

v1 正式支持：

```text
Appearance
├── System
├── Light
└── Dark
```

其中：

```text
System
→ follow OS / browser color-scheme preference
```

该选择属于 O3Pilot 本地 Appearance Preference。

正式关系：

```text
Dark
≠ Inverted Light
```

禁止：

```css
filter: invert();
```

或通过机械数学反转 Light Token 生成 Dark Theme。

Dark Theme 必须使用独立正式 Token 覆盖至少：

```text
Canvas
Surface
Text
Border
Action
Status
Chart
Focus
Overlay
```

Theme 切换只能改变表现层，不得改变：

- Metric Status；
- Origin；
- Risk Severity；
- 数据含义；
- Read-only Boundary；
- Chart Series 的逻辑身份。

## 9.9 Contrast and Accessibility

正式最低目标：

```text
Normal text        ≥ 4.5:1
Large text         ≥ 3:1
UI boundaries      ≥ 3:1 when required
Focus indication   clearly distinguishable
```

尤其需要验证：

```text
Text Tertiary
12px Micro
Status Badge
Chart Label
Focus Ring
Disabled State
Light / Dark Theme
```

Disabled Control 的颜色不能被误认为仍可操作；同时不能依靠低对比度作为唯一禁用表达。

不得为了模仿浅灰视觉风格，使主要业务文本、Source、Timestamp、Metric Meta 或状态说明低于正式可读性要求。

## 9.10 Color QA

正式 Color QA 至少覆盖：

```text
Light Theme
Dark Theme
System Theme switch
Normal / Hover / Pressed / Focus / Disabled
Success / Warning / Danger / Information / Neutral
VALID / PARTIAL / STALE / UNAVAILABLE / ESTIMATED
Positive / Negative / Neutral metric changes
Origin labels
Charts with semantic states nearby
Color-vision deficiency checks
WCAG AA contrast
```

验收时必须确认：

- 去掉颜色后，用户仍能理解主要状态；
- 趋势方向没有被机械映射成好坏；
- Chart Series 没有错误借用 Status Color；
- Normal 状态不会形成大面积绿色噪音；
- Light / Dark 下业务语义保持一致；
- Focus、Error 和 Risk 在两个 Theme 中均清晰可识别。

---

# 10. Surfaces

O3Pilot 不使用“所有内容都是卡片”的 Dashboard 模式。

Surface 的目的不是制造视觉层级本身，而是在 Spacing、Section、Divider 已不足以表达结构时，为一组具有独立语义的内容建立边界。

正式原则：

```text
Section ≠ Card

Metric Tile ≠ Mandatory Card

Spacing before Surface
Surface before Shadow
```

页面结构优先级：

```text
Canvas
↓
Section
↓
Bounded / Interactive Surface when needed
↓
Control
↓
Overlay only when interaction requires it
```

## 10.1 Surface Principles

O3Pilot 优先通过以下顺序建立信息结构：

```text
Spacing
>
Background Difference
>
Hairline / Divider
>
Bounded Surface
>
Elevation / Shadow
```

Shadow 是最后一级视觉手段，而不是默认 Panel 样式。

正式规则：

- Section 默认不创建 Card；
- KPI 默认不各自创建 Card；
- 数据密集页优先保持连续阅读面；
- Surface 必须对应真实分组、交互或层级关系；
- 不为“高级感”增加无业务意义的边框、阴影、浮层或卡片嵌套；
- 同一页面的 Surface 数量由信息结构决定，不由组件数量决定。

## 10.2 Surface Roles

O3Pilot 正式区分以下 Surface Role：

```text
Canvas
Section
Bounded Surface
Interactive Surface
Overlay Surface
```

### Canvas

页面或工作区的基础背景。

Canvas：

- 承载页面整体；
- 不表达业务状态；
- 不因为某个 Metric 正常、异常或高价值而改变背景色；
- 使用正式 Theme Token。

### Section

Section 是页面中的语义区域，不是 Card。

主要通过：

```text
Heading
Spacing
Divider when needed
```

形成结构。

Section 默认：

```text
No Border
No Radius
No Shadow
```

### Bounded Surface

仅在一组内容确实需要独立边界时使用。

典型场景：

- 独立 KPI Group；
- 诊断摘要；
- Import Preview；
- Settings Group；
- 需要明确与周围内容分离的数据区域；
- 明确独立的状态或解释区域。

默认可以使用：

```text
Surface Background
+
Hairline
+
Radius
```

但默认无 Shadow。

### Interactive Surface

只有整个区域本身存在明确交互时，才属于 Interactive Surface。

例如：

- 可进入详情的实体结果；
- 可选择的数据对象；
- 可展开的独立对象；
- Command / Search Result。

Interactive Surface 不通过大幅抬升、强阴影或 Hover 缩放表达可点击性。

### Overlay Surface

Overlay Surface 位于主内容层之上。

包括：

```text
Tooltip
Popover
Dropdown
Command Palette
Drawer
Modal
```

Overlay 可以使用 Elevation / Shadow / Scrim，因为它们确实具有独立视觉层级。

## 10.3 When to Create a Surface

创建新 Surface 前，应按以下顺序判断：

```text
Need grouping?
↓
Can spacing solve it?
→ Yes: use spacing

Need stronger separation?
↓
Can background difference / divider solve it?
→ Yes: use them

Need an independently bounded object?
→ Use Bounded Surface

Need content above the normal document layer?
→ Use Overlay Surface
```

因此：

```text
One component
≠ One Surface

One Section
≠ One Card
```

不得因为使用了一个 React / Vue Component，就自动赋予独立 Card 外观。

## 10.4 Surface Nesting

可见的 Boxed Surface 嵌套应保持极少。

正式原则：

```text
Visible boxed surface nesting
→ normally max 2 levels
```

常规结构：

```text
Canvas
└── Bounded Surface
    └── Controls / Content
```

必要时可以：

```text
Canvas
└── Bounded Surface
    └── Subtle bounded group
```

但避免：

```text
Card
└── Card
    └── Card
        └── KPI Card
```

当需要第三层视觉区分时，优先改用：

- Divider；
- Section Heading；
- Spacing；
- Subtle Background；
- Typography hierarchy。

## 10.5 Metric / Data Surfaces

正式原则：

```text
Metric Tile
≠ Mandatory Card
```

一组 KPI 可以共享一个 Section 或一个 Bounded Surface，例如：

```text
Revenue     Orders     Profit     Return Rate
¥...        ...        ...        ...
```

不要求每个 Metric 都拥有：

- 独立 Border；
- 独立 Radius；
- 独立 Background；
- 独立 Shadow。

只有当某个 Metric 自身具备以下独立语义时，才考虑独立 Surface：

- 独立交互；
- 独立告警或诊断状态；
- 明确不同于周围 Metric 的内容结构；
- 需要承载更复杂的详情摘要。

纯数值比较应优先保证共同基线和阅读效率，而不是 Card 数量。

## 10.6 Interactive Surfaces

交互 affordance 只能出现在真实可交互 Surface 上。

正式原则：

```text
Interactive affordance
→ only when interaction exists
```

禁止纯展示数据区域因为视觉模板而出现：

- `cursor: pointer`；
- Hover Action Blue；
- Hover 上浮；
- Hover Scale；
- 强 Shadow；
- 看似按钮但没有操作的视觉状态。

推荐 Interactive Surface 状态：

```text
Default
→ neutral surface

Hover
→ subtle background / border change

Selected
→ selected surface / border

Keyboard Focus
→ formal focus ring
```

不使用普通 Card Hover：

```css
transform: translateY(-2px);
box-shadow: ...;
```

作为默认可点击反馈。

## 10.7 Borders and Dividers

Border / Divider 用于结构，不用于装饰。

优先：

```text
Hairline
Divider Soft
```

规则：

- Table Row 不因每一行都创建 Card Border；
- Section 之间不需要同时存在 Background Difference + Border + Shadow 三重分隔；
- Selected / Focus / Error Border 必须使用对应正式语义 Token；
- 普通静态 Surface 不使用高对比粗边框；
- Dark Theme 必须使用独立 Dark Border Token，不简单反转 Light Border。

## 10.8 Elevation and Shadow

普通 Dashboard Panel、Metric、Table、Section 默认：

```text
elevation-0
```

正式 Elevation Role：

```text
elevation-0
→ Canvas / Section / Bounded Surface / normal Panel

elevation-1
→ Tooltip / Small Popover / Dropdown

elevation-2
→ Drawer / Command Palette / stronger Overlay

elevation-3
→ Modal / Blocking Overlay
```

规则：

1. 所有 Shadow 必须使用正式 Elevation Token；
2. Feature 不得自行创建任意 `box-shadow`；
3. 普通 Panel 不使用 Shadow；
4. Sticky Header / Global Context Bar 优先使用 Hairline 或 Surface Difference；
5. 只有发生真实视觉重叠时，Sticky Element 才允许轻量 Elevation Indication；
6. Drawer / Modal 的 Shadow 不得比内容层级本身更抢眼；
7. Light / Dark Theme 分别使用适配的 Shadow Token。

允许 Shadow 的典型对象：

- Tooltip；
- Popover；
- Dropdown；
- Command Palette；
- Drawer；
- Modal；
- 必要的 Floating Control。

不允许：

- 每个 KPI 一层 Shadow；
- 表格每行 Shadow；
- 普通 Section Shadow；
- Card-in-Card 每层 Shadow。

## 10.9 Overlay Surfaces

Overlay Surface 必须通过 Surface、Scrim、Focus 和 z-index 形成真实层级，而不是依赖夸张 Shadow。

典型结构：

```text
Page Content
↓
Scrim when blocking is required
↓
Overlay Surface
```

规则：

- Tooltip / Dropdown 不使用 Blocking Scrim；
- Drawer 是否使用 Scrim 由交互模式决定；
- Modal 必须明确阻塞背景交互；
- Overlay 必须保持与第 27 章 Focus Management 一致；
- Overlay 层级不得通过 Feature 私自增加随机 z-index 解决；
- Surface Visual Elevation 与 Interaction Blocking 是两个概念，不得混淆。

## 10.10 Responsive Surface Behavior

Responsive 不是把 Desktop Card 等比例缩小。

正式原则：

```text
Responsive
≠ shrink every card
```

Narrow 环境中，大型数据 Surface 可以：

```text
Become edge-to-edge
Reduce outer margin
Reduce or remove outer radius when appropriate
Preserve internal hierarchy
```

因此 Desktop 上的：

```text
Bounded Surface
```

在 Narrow 上可以转化为：

```text
Section
```

只要：

- 信息语义不改变；
- Header / Controls / Data 层级仍然清晰；
- Focus / Selection / Error 状态仍可识别；
- 不因为去 Card 化而丢失真实交互边界。

不得为了保留 Desktop Card 外观，在 390px 屏幕上叠加过多：

```text
Page Margin
+
Card Margin
+
Card Padding
+
Nested Card Padding
```

导致业务数据可用宽度被无意义压缩。

## 10.11 Surface QA

正式 Surface QA 至少检查：

```text
Dashboard with many metrics
Dense data table
Settings form
Interactive search result
Popover / Dropdown
Drawer
Modal
Light Theme
Dark Theme
390 Narrow
```

必须确认：

- Section 没有被机械 Card 化；
- Metric 没有被机械拆成独立 Card；
- Visible Boxed Nesting 通常不超过 2 层；
- 普通 Panel 无 Shadow；
- Interactive Affordance 只存在于真实可交互对象；
- Keyboard Focus 与 Hover 能明确区分；
- Dark Theme Surface / Border / Shadow 仍保持层级；
- Narrow 不因保留 Card 外壳而浪费主要内容宽度。

---

# 11. Shape

O3Pilot 使用有限、稳定、按组件角色分配的圆角系统。Shape 用于建立组件层级与一致性，不承担业务状态、交互重要度或可点击性的表达。

正式原则：

```text
Radius follows component role
not developer preference
```

Feature 不得因为局部视觉偏好自行引入新的圆角值。

## 11.1 Radius Tokens

v1.0 正式 Radius Tokens：

```text
radius-xs       6px
radius-sm       8px
radius-md      10px
radius-lg      12px
radius-xl      16px
radius-pill   999px
```

正式实现只能优先从上述 Token 选择，不应在 Feature 内新增例如：

```text
7px
13px
20px
24px
```

等未定义值。

避免无区别地对所有组件使用 20–24px 超大圆角。

## 11.2 Role Mapping

推荐的正式角色映射：

| Token | 主要用途 |
|---|---|
| `radius-xs` / `6px` | 极小型内部元素、紧凑 Meta Surface |
| `radius-sm` / `8px` | Button、Input、Select、普通 Control |
| `radius-md` / `10px` | 较宽 Control、Interactive Surface、小型独立对象 |
| `radius-lg` / `12px` | Bounded Surface、Popover、普通 Panel |
| `radius-xl` / `16px` | 大型分析 Surface、Drawer、Modal 等大型容器 |
| `radius-pill` / `999px` | Status、Tag、Filter Chip 等真正 Pill 形态 |

正式原则：

```text
Component Role
→ Radius Token
```

而不是：

```text
Feature Preference
→ Arbitrary Radius
```

同类组件应保持同一 Shape Contract。

## 11.3 Pill Usage

`radius-pill` 是受限形态，不是默认圆角。

正式原则：

```text
Pill
≠ Default rounded component
```

适合 Pill 的典型场景：

- Status；
- Tag；
- Filter Chip；
- 少量紧凑 Segmented Control；
- 紧凑数值 Badge。

不应默认使用 Pill 的组件：

- Primary Button；
- Secondary Button；
- Input；
- Search Box；
- Table；
- Card / Surface；
- Modal；
- Drawer。

避免把 O3Pilot 变成由大量胶囊按钮和胶囊容器组成的消费型 App 视觉语言。

## 11.4 Component Shape Consistency

Shape 不负责表达组件的重要程度、风险等级或业务语义。

禁止通过不同几何形状表达：

```text
Primary
Secondary
Warning
Danger
Success
```

例如不得采用：

```text
Primary Button   → Pill
Secondary Button → 8px
Danger Button    → Sharp Corners
```

正式原则：

```text
Semantic importance
≠ Different corner geometry
```

同一种 Button Component 应保持一致 Shape；Primary / Secondary / Danger 等差异由：

- Color；
- Border；
- Typography；
- Icon；
- Label；

表达。

同样：

```text
Clickable
≠ More rounded
```

圆角本身不用于表示可点击性。

## 11.5 Nested Radius

当存在可见的嵌套边界时，内层 Shape 应与外层保持视觉连续。

默认关系：

```text
Inner Radius
≤
Parent Radius
```

例如：

```text
16px Large Surface
└── 12px Bounded Group
    └── 8px Controls
```

避免：

```text
12px Surface
└── 20px Input
```

导致内部组件比外层容器更膨胀的视觉冲突。

该规则不是要求实现机械的数学同心圆计算；当父子元素之间不存在可见边界关系时，不需要强行推导所谓 perfect concentric radius。

## 11.6 Data Surface Shape

高密度数据界面必须优先保持连续阅读面。

正式规则：

```text
Table rows
→ no individual radius
```

允许：

```text
Table outer bounded surface
→ radius-lg
```

不允许把 Table 处理为：

```text
Every Row → Rounded Card
Every Cell → Rounded Box
Hover Row → Floating Rounded Card
```

Chart 同样不因为是 Chart 就自动获得独立圆角。

正式关系：

```text
Chart
→ follows containing Surface
```

只有 Chart 所在 Bounded Surface 需要边界时，才由外层 Surface 使用正式 Radius Token。

Metric Tile 也遵循第 10 章规则：

```text
Metric Tile
≠ Mandatory rounded Card
```

## 11.7 Responsive Shape Behavior

Responsive 可以移除圆角，但不创建新的圆角体系。

正式原则：

```text
Responsive may remove a radius
but does not invent new radii
```

例如 Narrow 模式下，大型 Bounded Surface 可以转为 edge-to-edge Section：

```text
Desktop
16px bounded surface

Narrow
→ edge-to-edge
→ viewport-facing corners may become 0px
```

全屏 Drawer / Modal 在 Narrow 下贴合 Viewport 边缘时，对应边可以取消圆角。

这属于已有 Shape 的响应式降级，不意味着建立独立 Mobile Radius System。

禁止为了“移动端更柔和”额外引入一套例如：

```text
18px
20px
24px
```

的 Mobile-only Radius Tokens。

## 11.8 Shape QA

正式 Shape QA 至少检查：

```text
Button / Input / Select
Popover / Dropdown
Bounded Surface
Large Analysis Surface
Modal / Drawer
Status / Tag / Filter Chip
Dense Table
Metric Group
Nested Surface
Narrow edge-to-edge Surface
```

验收重点：

- 同类组件是否使用稳定一致的 Radius Token；
- 是否出现 Feature 自行定义的任意圆角值；
- Pill 是否只用于真正适合 Pill 的组件；
- Primary / Danger 等语义是否错误地通过 Shape 区分；
- 可点击性是否错误地依赖更大圆角表达；
- 可见嵌套中是否基本满足 `Inner Radius ≤ Parent Radius`；
- Table Row / Cell 是否被错误地卡片化；
- Narrow 是否通过移除外层圆角节省空间，而不是发明新的 Radius System。

Shape 核心不变量：

```text
Radius follows role

Pill ≠ default component shape

Semantic importance ≠ different geometry

Inner Radius ≤ Parent Radius

Table rows → no individual radius

Responsive may remove radius
but does not invent new radii
```

---

# 12. Spacing

O3Pilot 正式采用 `4px` 基础间距网格。

Spacing 的目标不是让所有页面拥有相同空白，而是用有限、稳定的 Token 表达信息之间的语义距离，并在 Desktop、Compact、Narrow 和不同 Density 下保持一致。

正式原则：

```text
Semantic distance
→ Spatial distance

Spacing
→ Use formal token

Arbitrary visual spacing
→ Prohibited by default
```

---

## 12.1 Base Grid and Tokens

正式 Spacing Tokens：

```text
space-1     4px
space-2     8px
space-3    12px
space-4    16px
space-5    20px
space-6    24px
space-8    32px
space-10   40px
space-12   48px
space-16   64px
```

正式关系：

```text
4px
→ base grid

8 / 12 / 16 / 20 / 24 / 32 / 40 / 48 / 64
→ approved spacing values
```

Feature / Page 不得为了局部视觉调整自行引入新的视觉间距，例如：

```text
6px
10px
14px
18px
22px
28px
30px
```

除非该数值属于组件几何计算结果，而不是用于表达视觉间距。

例如：

```text
Calculated geometry
→ allowed when technically required

Visual spacing decision
→ must use token
```

不得把“4px 基础网格”理解为任何 4 的倍数都自动成为正式 Token。

---

## 12.2 Spacing Semantics

Spacing 按语义关系而不是组件类型随机选择。

推荐语义：

```text
4px
→ micro relation
→ Source / Timestamp / tightly related metadata

8px
→ tight relation
→ Icon + Label / closely related inline items

12px
→ control relation
→ controls in one group / compact field relation

16px
→ group relation
→ content groups inside one section

24px
→ related section separation
→ closely related analytical blocks

32px
→ major section separation
→ primary page structure

40px / 48px / 64px
→ exceptional large separation only
```

正式原则：

```text
More semantic distance
→ More spatial distance
```

但不得为了制造“高级感”在普通数据页面频繁使用：

```text
48px
64px
```

的大面积空白。

O3Pilot 是高信息密度数据产品，不采用营销页面式的大跨度留白作为默认节奏。

---

## 12.3 Page Gutters

页面水平 Gutter 按 Breakpoint 使用确定值，不使用区间。

正式值：

```text
Desktop
→ 32px

Compact
→ 24px

Narrow
→ 16px
```

正式原则：

```text
Breakpoint
→ Deterministic gutter
```

禁止使用：

```text
28–32px
20–24px
12–16px
```

这类把最终布局决定留给各 Feature 的范围值。

如果页面需要不同宽度行为，应通过第 6 章定义的 Layout Mode 或明确的 Responsive Contract 处理，而不是自行修改 Page Gutter。

例如：

```text
Data Canvas
→ may use the full available width

Standard Content
→ uses formal page gutter

Reading / Detail
→ uses formal content width + page gutter
```

页面默认顶部和底部主要内容间距：

```text
32px
```

特殊全屏、Edge-to-edge 或 Overlay 场景可以由其自身 Pattern 覆盖。

---

## 12.4 Section Rhythm

页面垂直结构主要采用：

```text
Major Section Gap
→ 32px

Related Section Gap
→ 24px

Content Group Gap
→ 16px
```

示例：

```text
Page Header
↓ 32px
Primary Analysis Section

Primary Analysis Section
↓ 32px
Risk Section

Chart
↓ 24px
Related Breakdown

Panel Title
↓ 16px
Panel Content
```

`Section Gap` 不使用模糊的 `24–32px` 范围。

页面应先判断内容之间的语义关系，再选择正式档位。

禁止为了填满视觉空间而：

```text
随机增加 Margin
反复使用 40 / 48 / 64px
对不同页面使用不同但近似的 Section Gap
```

---

## 12.5 Component Padding and Gaps

组件内部正式使用以下常见关系：

```text
Standard Bounded Surface padding
→ 24px

Compact / Dense Surface padding
→ 16px

Control Group gap
→ 12px

Inline Control gap
→ 8px

Icon + Label gap
→ 8px

Tight Metadata gap
→ 4px
```

正式原则：

```text
Density
→ Chooses a defined spacing mode

Component
→ Does not invent its own near-identical padding
```

例如普通 Surface 不应分别出现：

```text
18px
20px
22px
```

三套近似 Padding。

### Padding / Gap / Margin

三者职责：

```text
Padding
→ boundary to content

Gap
→ spacing between sibling items

Margin
→ exceptional relationship between external layout objects
```

组件内部优先：

```text
Parent
→ owns child spacing
```

即优先通过：

```text
padding
gap
```

管理布局，而不是让 Child 大量携带：

```text
margin-top
margin-right
margin-bottom
margin-left
```

用于修补组合关系。

这样组件被放入不同 Context 时不会携带意外外边距。

---

## 12.6 Data / Table Spacing

高密度业务数据必须使用独立的 Data / Table Contract，不由普通 Component Padding 随意推导。

表格水平 Cell Padding：

```text
Standard Table
→ 16px

Compact Table
→ 12px
```

表格垂直密度由第 23 章的 Table Contract 定义，例如：

```text
Header Height
Row Height
Compact Row Height
```

正式原则：

```text
Table row height
→ Table Contract

General spacing token
→ must not accidentally redefine table density
```

不得通过 Feature 自行增加或减少 Cell Vertical Padding 来绕过正式 Row Height。

对于图表、Metric、Dense List 等 Data Surface：

- 优先保证可比较值的扫描效率；
- 不为视觉留白牺牲有效数据宽度；
- 不因为外围 Surface 使用 `24px` Padding，就要求所有内部数据组件继续叠加同等级 Padding。

---

## 12.7 Density Modes

O3Pilot 正式允许三种 Density Mode：

```text
Comfortable
Standard
Compact
```

它们表达任务密度，不是全局按比例缩放。

正式原则：

```text
Compact
≠ Everything smaller
```

典型关系：

```text
Page Gutter
→ primarily breakpoint-driven

Table Density
→ task-driven

Surface Padding
→ Standard or Compact

Touch Target
→ accessibility-driven
→ must not shrink below minimum
```

例如：

```text
Compact Table
→ may use 12px horizontal cell padding

Compact
≠ 12px body text
≠ smaller focus target
≠ smaller Narrow touch target
```

`Comfortable` 只应用于解释、诊断、设置或阅读型 Context，不应把数据表格扩张成低密度展示页面。

---

## 12.8 Responsive Spacing

Responsive 的目标是保护可用内容宽度，而不是机械缩小全部 Token。

正式原则：

```text
Preserve usable content width
not accumulated container padding
```

尤其 Narrow 下需要避免：

```text
Page Gutter
+
Surface Padding
+
Nested Surface Padding
```

不断叠加。

例如不推荐：

```text
Viewport
└── 16px Page Gutter
    └── Surface
        └── 24px Padding
            └── Nested Surface
                └── 16px Padding
```

在约 `390px` 的 Narrow 屏幕中会无意义压缩数据空间。

结合第 10 章的 Responsive Surface Behavior：

```text
Narrow

Large Bounded Surface
→ may become edge-to-edge Section

Section content
→ keeps required 16px effective horizontal content inset
```

因此：

```text
Page gutter + Surface padding
≠ Mandatory double gutter
```

当 Surface 已去 Card 化或 Edge-to-edge 时，应重新计算有效内容 Inset，而不是继续累加外层 Padding。

Responsive 可以：

- 移除冗余容器 Padding；
- 将 Bounded Surface 转为 Section；
- 减少同一层级的重复空白；

但不得：

- 创建新的 Mobile-only Spacing Token；
- 把 Touch Target 缩小；
- 把主要正文或数据压缩到不可读密度。

---

## 12.9 Implementation Rules

正式实现规则：

1. 所有通用 Spacing 必须来自正式 Token；
2. CSS / Component API 优先暴露语义或 Token，而不是任意像素值；
3. Feature 不得自行创建 near-duplicate spacing；
4. Parent 默认负责 sibling spacing；
5. Layout 优先使用 `gap` 和 `padding`；
6. 外部 `margin` 仅用于明确的跨组件关系；
7. 不使用空白 DOM、不可见文本或重复 `<br>` 制造间距；
8. 不使用绝对定位作为普通排版间距工具；
9. Responsive 调整必须保持同一 Spacing Language；
10. 任何非 Token 视觉间距必须在 Design System 中有明确例外依据。

推荐关系：

```text
Design Token
↓
Layout Primitive / Component
↓
Feature
```

而不是：

```text
Feature
→ arbitrary px
→ local visual fix
```

---

## 12.10 Spacing QA

正式 Spacing QA 至少检查：

```text
1440px Desktop
1200px Desktop
1024px Compact
768px Compact / Narrow boundary
390px Narrow

Dashboard
Dense Orders / Products / Inventory Table
Standard Settings Page
Reading / Detail Page
Drawer / Modal
Nested Surface
Long CN / EN / RU text
```

验收重点：

- 不存在 `28px` 等未批准的视觉间距；
- Page Gutter 在相同 Breakpoint 下保持一致；
- Major / Related / Group 关系能从空间距离中辨认；
- Surface 不出现多个近似但无语义差异的 Padding；
- Parent 正确拥有 sibling spacing；
- Compact Mode 没有把全部控件机械缩小；
- Table Row Height 未被普通 Spacing 覆盖；
- Narrow 没有出现 Page + Surface + Nested Surface 的无意义 Padding 累积；
- 390px 下主要业务内容仍保有合理有效宽度；
- 40 / 48 / 64px 没有在普通数据页被过度使用。

核心规则：

```text
4px base grid

Breakpoint
→ deterministic gutter

Semantic distance
→ spatial distance

Parent owns child spacing

Compact
≠ everything smaller

Avoid accumulated container padding

No arbitrary visual spacing values
```

---

# 13. Icons

O3Pilot 的功能图标系统必须保持单一、稳定、可维护，并且与业务语义解耦。

正式原则：

```text
Lucide
→ only general-purpose functional icon source

Brand Mark
≠ Functional Icon

Semantic ID
≠ Lucide icon name
```

图标用于提高扫描效率、识别交互和辅助状态理解，但不得替代文字、业务状态或可访问语义。

## 13.1 Icon Principles

O3Pilot 图标系统遵循：

```text
Consistent
Semantic
Quiet
Accessible
Maintainable
```

正式要求：

1. 通用功能图标只使用 Lucide；
2. Feature / Page 不直接以 Lucide 图标名称表达业务语义；
3. 图标的尺寸、Stroke 和交互状态必须使用统一 Contract；
4. Active、Primary、Danger 等重要程度不得通过切换图标家族、增粗 Stroke 或装饰 Shadow 表达；
5. 图标不能成为业务状态、操作含义或可访问名称的唯一载体；
6. Morph 是例外反馈模式，不是默认图标行为；
7. 不为了视觉差异引入第二套通用图标库。

正式关系：

```text
Importance
≠ thicker icon

Active
≠ filled icon variant

Status Icon
≠ Status itself

Space saving
≠ sufficient reason for Icon-only
```

## 13.2 Sources and Exceptions

### Functional UI Icons

正式通用图标源：

```text
Lucide
```

Lucide Outline 为 O3Pilot canonical functional icon language。

不得建立：

```text
Outline Icons
+
Filled Icons
```

两套并行状态系统。

### Morphing Icons

允许使用：

```text
Morphicons
```

但 Morphicons 只承担已批准的状态过渡反馈，不成为第二套静态通用图标库。

### Brand / Special-purpose Assets

以下内容不属于 Lucide Functional Icon Contract：

```text
O3Pilot Brand Mark
External Platform Brand Mark
Approved special-purpose product symbol
```

自定义 SVG 仅允许用于：

1. Brand Mark；
2. Lucide 确实不存在、且业务上需要稳定专用符号的产品语义。

这些资源必须进入集中 Asset / Icon Layer，不允许 Feature 临时粘贴 Inline SVG 作为日常图标实现。

正式关系：

```text
Brand Logo / Platform Mark
→ Brand Asset

Lucide
→ Functional UI Icon
```

不得为了“只能用 Lucide”而使用近似通用图标冒充正式品牌标识。

## 13.3 Semantic Icon Registry

业务代码不得大量直接写：

```ts
import { Package, ShoppingCart, ... } from "lucide-*";
```

正式层级：

```text
Feature / Page
↓
O3Icon
↓
Semantic Icon Registry
↓
Lucide / Approved Asset
```

Registry 按语义角色组织：

```text
nav.*
action.*
entity.*
status.*
utility.*
```

示例：

```text
nav.inventory

action.search
action.copy
action.export
action.filter
action.refresh
action.close
action.more

entity.shop
entity.product
entity.order
entity.campaign

status.success
status.warning
status.danger
status.info
status.unavailable
status.stale

utility.currency
utility.data-status
utility.calendar
```

业务代码依赖的是 Semantic Registry Key，而不是上游图标名称。

正式关系：

```text
Semantic Registry Key
→ stable product contract

Lucide Icon Name
→ replaceable implementation detail
```

## 13.4 Navigation Icon Mapping

导航 Icon Key 必须直接对应第 5 章正式 Navigation Semantic ID。

```text
overview
→ nav.overview

sales
→ nav.sales

products
→ nav.products

orders
→ nav.orders

inventory
→ nav.inventory

fulfillment
→ nav.fulfillment

returns
→ nav.returns

advertising
→ nav.advertising

buyer-feedback
→ nav.buyer-feedback

settlements
→ nav.settlements

profit
→ nav.profit

shop-health
→ nav.shop-health

alerts
→ nav.alerts

recommendations
→ nav.recommendations

data-center
→ nav.data-center

settings
→ nav.settings
```

正式关系：

```text
Navigation Semantic ID
→ stable

Navigation Icon Registry Key
→ derived from semantic role

Lucide Icon Name
→ implementation detail
```

不得重新引入：

```text
customerVoice
finance
health
alert
recommendation
data
```

等与当前正式 IA 不一致的旧 Registry 业务语义。

## 13.5 Size and Stroke Contract

正式 Icon Size Tokens：

| Token | Size | Typical Use |
|---|---:|---|
| `icon-sm` | 16px | Inline / Metadata |
| `icon-md` | 18px | Dense Control / Table Action |
| `icon-lg` | 20px | Standard Control / Sidebar |
| `icon-xl` | 24px | Empty / Status / Larger Explanatory Context |

高频 UI 默认使用：

```text
16 / 18 / 20 / 24px
```

大型 Empty State 或专项说明可以由其 Component Pattern 定义更大的展示尺寸，但不得因此持续新增全局 Icon Size Token。

Lucide 默认 Stroke：

```text
stroke-width: 2
```

Feature 不得自行使用：

```text
1.5
2.25
2.5
```

等 Stroke Width 制造局部视觉风格。

不得通过：

```text
Fill
Thicker Stroke
Filter
Shadow
```

使某个普通功能图标显得“更重要”。

正式原则：

```text
Importance
≠ thicker icon
```

重要程度由 Color、Typography、Surface、Position 和 Label 表达。

## 13.6 Icon + Label / Icon-only Rules

默认优先：

```text
Icon + Label
```

Icon-only 只适用于高熟悉度、低歧义、重复使用的操作。

典型允许：

```text
Search
Close
More
Copy
Compact Filter
```

重要、低频或语义不明确的操作优先：

```text
Icon + Label
```

例如：

```text
导入官方报告
开始利润回测
重新映射商品
恢复备份
```

不得仅通过一个没有明确文字的图标表达。

正式原则：

```text
Space saving
≠ sufficient reason for Icon-only
```

Icon-only Control 必须至少具备：

```text
Accessible Name
Tooltip
Focus State
44×44 minimum touch target where touch interaction applies
```

视觉 Icon 可以小于触控目标。

## 13.7 Accessibility

Icon-only Button 的可访问名称属于 Control 自身，而不是 Tooltip。

推荐语义：

```html
<button aria-label="复制订单号">
  <svg aria-hidden="true">...</svg>
</button>
```

正式关系：

```text
Accessible Name
→ control semantics

Tooltip
→ supplemental explanation only
```

不得依赖：

- SVG `<title>`；
- Hover Tooltip；
- 图标文件名；

作为 Icon-only Control 的唯一 Accessible Name。

当控件已经拥有可见文字 Label 时，装饰性图标默认：

```text
aria-hidden="true"
```

避免辅助技术重复朗读。

所有可交互图标必须具备可见 Focus State。

颜色、形状和图标均不得成为唯一的状态识别方式。

## 13.8 Status Icon Rules

Status Icon 用于提高扫描效率，但不能替代正式状态本身。

推荐结构：

```text
Status Icon
+
Status Label
+
Semantic State
```

例如 O3Pilot 正式数据状态：

```text
VALID
PARTIAL
UNAVAILABLE
UNVERIFIED
STALE
```

不能仅显示：

```text
✓
!
?
```

让用户自行推断业务含义。

同时遵守第 9 章 Color Contract：

```text
Status Icon Color
→ follows semantic status

Direction Icon
≠ business outcome by itself
```

正常状态默认保持 Quiet，不为了“正常”而在整个界面铺满绿色 Check Icon。

## 13.9 Morphicons Contract

静态图标是默认模式：

```text
Static Icon
→ default

Morph
→ exception
```

Morph 只在状态连续性能够明显改善反馈时使用。

已正式批准的 Sidebar Pattern：

```text
Original
→ Check
→ Original
```

适合的其他本地反馈示例：

```text
Copy
→ Check
```

禁止因为 Morphicons 已存在，就让以下高频图标普遍 Morph：

- Search；
- Settings；
- Dashboard / Overview；
- Filter；
- 普通 Hover；
- 普通 Active State。

Morph 不得阻塞操作完成或路由跳转。

具体 Duration、Easing、取消、并发行为由第 14 章 Motion System 统一定义。

`prefers-reduced-motion` 下：

```text
Static state replacement
→ sufficient
```

不要求保留完整 Morph Path Animation。

## 13.10 Lucide Dependency and Update Contract

Lucide 频繁更新不得让业务代码直接承受破坏。

正式规则：

1. 使用 exact dependency version；
2. Lockfile 必须提交；
3. 不使用运行时 CDN；
4. 升级单独执行，不随无关依赖自动升级；
5. Semantic Icon Registry 中统一处理 renamed / removed icon；
6. 建立 Icon Registry Test；
7. 每次升级检查所有正式 Semantic Icon；
8. Morphicons Wrapper 同样隔离上游 API；
9. 业务代码不得依赖某个 Lucide 图标名称作为业务语义；
10. Upstream icon 改名时，不修改 Feature API；
11. Missing Semantic Icon 必须显式失败，不允许 Silent Fallback；
12. Registry 使用 explicit imports，并保持 tree-shakeable；
13. 不通过整个 Lucide Namespace 的运行时字符串查找实现动态图标发现。

禁止：

```ts
import * as Icons from "lucide-*";
```

再以业务字符串动态读取整个图标 Namespace 作为正式 Registry 实现。

推荐：

```text
Lucide
→ build-time dependency
→ explicit registry imports
→ tree-shakeable bundle
```

如果升级后某个正式图标不存在：

```text
Build / Registry Test
→ fail explicitly
```

不得静默统一替换为通用 Question / Help Icon 后继续发布。

## 13.11 Icon QA

正式 Icon QA 至少检查：

```text
Expanded Sidebar
Collapsed Sidebar
Narrow Icon Rail
Global Context Bar
Dense Table Actions
Forms
Status Labels
Alerts
Empty States
Command Palette
Light Theme
Dark Theme
Keyboard Focus
Touch Targets
Reduced Motion
CN / EN / RU Labels
```

验收要求：

- 所有 Navigation Semantic ID 均存在唯一正式 `nav.*` Registry Key；
- 不存在旧 IA 语义残留导致的第二套 Icon Key；
- Feature 不直接依赖 Lucide Icon Name 作为业务 API；
- 不存在第二套通用功能图标库；
- Lucide Size / Stroke 统一；
- Active State 不依赖 Filled Variant；
- Icon-only 操作均具有 Accessible Name；
- Tooltip 不承担唯一可访问名称；
- Icon + Visible Label 不发生重复朗读；
- Status Icon 不替代 Status Label；
- Morph 只出现在正式允许的反馈模式；
- Missing Registry Icon 会显式失败；
- 依赖实现可 Tree-shake，不打包整个图标 Namespace；
- Light / Dark 下图标对比度和状态语义保持一致。

---

# 14. Motion System

动效只遵循 O3Pilot 本节与 Emil Kowalski Motion Principles。

Motion 的默认立场不是“让界面更有动感”，而是：

```text
Motion must have a purpose
```

正式原则：

```text
Motion budget follows interaction frequency

User intent
>
animation completion

Data change
≠ animation event

Visual continuity
<
responsiveness

Decorative motion
→ off by default
```

---

## 14.1 Motion Principles

O3Pilot Motion 用于：

```text
Feedback
Spatial consistency
State indication
Preventing a jarring change
Explanation
Limited delight（仅低频、非业务关键场景）
```

无法明确说明动效目的时：

```text
Do not animate
```

Motion 不负责：

- 营造营销感；
- 让静态数据“更活”；
- 装饰性强调普通页面；
- 替代清晰的信息层级；
- 掩盖慢响应；
- 延迟用户实际操作。

---

## 14.2 Animation Gate

是否使用 Motion 按交互频率与任务性质判断，不使用“每天大约点击多少次”的主观计数。

正式分类：

```text
Continuous / Repetitive Interaction
→ no decorative motion

High-frequency Action
→ no motion or near-imperceptible functional feedback

Routine State Transition
→ short functional motion when it improves understanding

Low-frequency Spatial Transition
→ standard spatial motion

Rare / Onboarding Context
→ limited delight may be considered
```

典型 High-frequency Action：

```text
Navigation
Search typing
Table sorting
Pagination
Filter toggling
Copy
Row selection
Tabs
```

这些场景的 Motion 必须非常短，且不能阻塞下一次操作。

典型 Low-frequency Spatial Transition：

```text
Drawer open / close
Modal open / close
Large contextual popover
```

正式原则：

```text
Motion budget follows interaction frequency
not estimated daily counts
```

---

## 14.3 Motion Purposes and Categories

O3Pilot 将 Motion 分为四类。

### Feedback Motion

用于确认用户输入已经被接受。

例如：

```text
Button press
Copy → Check
Nav Morph
Toggle state
```

要求：

- 立即反馈；
- 时长短；
- 可中断；
- 不等待后台任务或页面数据加载完成。

### Spatial Motion

用于解释界面对象从哪里出现、到哪里消失，或空间关系如何变化。

例如：

```text
Popover
Dropdown
Drawer
Modal
```

Spatial Motion 必须服务空间理解，不得只是装饰。

### State Transition

用于帮助用户理解可见状态变化。

例如：

```text
Loading → Result
Collapsed → Expanded
Selected → Unselected
Visible → Hidden
```

只有 Motion 能明显改善理解时才使用。

### Decorative Motion

正式默认：

```text
Decorative Motion
→ Off
```

只允许在低频、非业务关键、不会干扰数据阅读或操作效率的场景中，经明确设计后使用。

---

## 14.4 Implementation Order

正式实现优先级：

```text
CSS Transition
↓
CSS @starting-style
↓
CSS Animation
↓
WAAPI
↓
Motion Library
```

CSS / HTML / TypeScript 足够实现时，不因 Motion 引入额外运行时库。

只有以下情况才考虑 Motion Library：

- Spring；
- 无简单 CSS 等价实现、且确实需要的 Layout Animation；
- Exit Animation；
- Gesture-driven Motion；
- 复杂可中断的空间交互。

Motion Library：

- 必须按需加载；
- 不作为所有控件的默认运行时依赖；
- 不得成为绕过本章 Motion Contract 的理由。

---

## 14.5 Animated Properties

优先动画：

```text
transform
opacity
```

高频 Motion 默认禁止直接动画：

```text
width
height
margin
padding
top
left
```

原因：这些属性容易触发布局、重排和大面积 Paint。

Accordion、真实内容高度展开等不存在合理 Transform 等价方案的场景可以作为明确例外，但必须：

- 证明 Motion 对理解有价值；
- 不造成明显 Layout Thrashing；
- 满足第 15 章 120Hz Performance Contract。

---

## 14.6 Easing Tokens

正式 Easing Tokens：

```css
--ease-ui: cubic-bezier(0.25, 0.1, 0.25, 1);
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```

映射：

```text
Color / Subtle State    → ease-ui
Enter / Exit            → ease-out
Move / Morph            → ease-in-out
Drawer                   → ease-drawer
Constant Progress        → linear
```

普通 UI 禁止使用 `ease-in` 作为进入或退出曲线。

禁止 Feature 自行定义未进入 Token System 的随机 cubic-bezier。

---

## 14.7 Duration Tokens

正式 Duration Tokens：

```text
motion-instant     80ms
motion-fast       120ms
motion-standard   180ms
motion-medium     220ms
motion-slow       280ms
```

普通 UI 原则：

```text
≤ 280ms
```

组件默认映射：

```text
Button visual feedback    80–120ms
Tooltip                  120ms
Small Popover            180ms
Dropdown                 180ms
Tabs indicator           180ms
Nav Morph                180–220ms total
Toast                    180ms
Drawer                   220ms
Modal                    220ms
```

其中 `Nav Morph` 因包含完整 Morph Path，允许 `180–220ms` 的窄范围；其他组件优先直接使用单一 Duration Token。

不允许不同 Feature 为同一种组件自行选择任意 Duration，例如：

```text
Dropdown A → 150ms
Dropdown B → 237ms
Dropdown C → 250ms
```

除非正式 Component Contract 已定义差异。

---

## 14.8 Interaction Patterns

### Navigation

正式行为：

```text
Navigation
→ immediate

Morph
→ non-blocking visual feedback
```

Sidebar 用户点击成功导航时：

```text
Original Icon
→ Check
→ Original Icon
```

目标总时长：

```text
180–220ms
```

Motion 不得延迟 Route Change，也不等待页面数据加载完成。

程序化导航默认不触发 Morph。

Keyboard activation 可以保留反馈，但同样不得延迟导航。

### Button Press

Button Press 默认使用：

```text
Surface / Color / Border state
```

可以使用极轻微、不可感知为“弹跳”的 Transform，但不得把明显 Scale 作为默认反馈。

禁止普通按钮默认使用：

```css
transform: scale(0.95);
```

### Copy Feedback

允许：

```text
Copy
→ Check
```

作为短暂本地成功反馈。

不等待 Clipboard 之外的远程任务。

### Tooltip / Popover / Dropdown

进入与退出必须：

- 响应立即；
- 不通过长 Motion 延迟内容可读；
- 空间位移保持短距离；
- 高频打开场景优先更弱 Motion。

### Drawer / Modal

允许标准 Spatial Motion，但：

- 不阻塞关闭；
- 用户反向操作时立即响应；
- 不因动画未完成拒绝新的 Intent。

### Keyboard-triggered High-frequency Surface

例如 Command Palette：

```text
⌘K / Ctrl+K
→ immediately visible
```

最多允许不可感知的轻微 Opacity Treatment。

正式原则：

```text
Shortcut response
must feel immediate
```

这不等于“所有键盘触发的 UI 永远禁止 Motion”；禁止的是可感知的入口延迟。

---

## 14.9 Interruptibility

所有用户驱动 Motion 默认必须可中断。

正式原则：

```text
User intent
>
animation completion
```

例如：

```text
Open Drawer
↓
Animation still running
↓
User closes Drawer
↓
Close immediately supersedes open motion
```

禁止：

- Queue 用户交互动画；
- 等 Exit Animation 完成后才响应新的关闭 / 打开 Intent；
- 快速连续点击后播放完整历史 Morph 队列；
- 因 Motion 状态锁住导航、Tabs 或核心控件。

新的用户操作可以：

- 取消前一个 Motion；
- 从当前视觉状态反向；
- 直接跳到新的目标状态。

---

## 14.10 Data Motion Rules

业务数据变化默认不是动画事件。

正式关系：

```text
Data change
≠ animation event
```

例如：

```text
38,241
↓
40,918
```

默认直接更新。

禁止默认使用：

- Count-up；
- 数字滚动；
- 翻牌；
- 弹跳；
- 闪烁；
- 每次刷新都重新播放 KPI 入场动画。

实时或同步后的数据更新如需表达“数据已更新”，优先通过：

- As-of Time；
- Freshness State；
- 极轻的局部状态提示；

而不是让数值本身持续运动。

Chart 数据同样不得因为刷新、筛选或时间范围变化而做装饰性弹跳或大规模 Entrance Animation。

---

## 14.11 Layout Motion Rules

Layout Animation 的采用门槛高于普通 Opacity / Transform Motion。

正式决策：

```text
Need Layout Animation?
↓
Can layout snap + transform / opacity explain the change?
→ Yes: use that

No
↓
Does motion materially explain spatial change?
→ Yes: consider Layout Animation

Otherwise
→ no Layout Animation
```

正式原则：

```text
Visual continuity
<
Responsiveness
```

Sidebar `248px → 72px` 是典型案例：

如果持续 Width Animation 会明显影响数据表、图表或页面重排性能，正式实现优先：

```text
Layout width instant switch
+
label opacity
+
icon transform / feedback
```

不为“连续感”牺牲交互响应。

---

## 14.12 Never Ship

禁止：

```css
transition: all;
```

禁止普通 UI：

```text
scale(0) entrance
noticeable button bounce / scale
`ease-in` entrance / exit
500ms Dropdown
可感知延迟的 Keyboard Shortcut entrance
Chart 数据装饰性跳动
KPI count-up by default
高频 hover 大缩放
Motion queue blocking user intent
Decorative full-page route slide
```

同时禁止：

- 用 Motion 掩盖慢请求；
- 每次页面加载都播放整页 Entrance Sequence；
- 数据更新触发大面积闪动；
- Feature 自行定义未进入正式 Token 的 Easing / Duration；
- 为了动效额外引入并长期常驻不必要的运行时库。

---

## 14.13 Motion QA

正式 Motion QA 至少检查：

```text
Mouse
Keyboard
Touch
Rapid repeated interaction
Interrupted open / close
Route changes
Tabs
Filters
Copy feedback
Drawer / Modal
Command Palette
Large tables
Charts
Metric refresh
Light Theme
Dark Theme
prefers-reduced-motion
120Hz-capable display where available
```

验收要求：

- Navigation 先发生，Morph 不阻塞；
- Sidebar Nav Morph 为 `180–220ms`；
- 高频操作没有明显 Entrance / Exit Delay；
- 连续快速操作不会产生 Motion Queue；
- 用户反向操作可以立即打断当前 Motion；
- 普通 UI 不超过 `motion-slow = 280ms`；
- 数据刷新不会触发 Count-up、Bounce 或装饰性跳动；
- Layout Motion 不造成明显掉帧或输入延迟；
- 没有 `transition: all`；
- 没有未登记 Easing / Duration；
- Motion Library 只在必要场景按需加载；
- Reduced Motion 行为符合第 16 章；
- Performance 行为符合第 15 章。

---

# 15. 120Hz Performance Contract

O3Pilot 的交互性能以 120Hz-capable 环境为高刷新率设计基线。

120Hz 意味着：

```text
one new frame opportunity
→ every ~8.33ms
```

但：

```text
8.33ms
≠ JavaScript budget
≠ Animation duration
```

一帧时间由浏览器完整渲染管线共同使用：

```text
Input
JavaScript
Style
Layout
Paint
Composite
Browser overhead
```

正式原则：

```text
Frame budget is shared
```

`DESIGN.md` 不承诺在任何设备、任何系统负载、任何浏览器状态下都绝对达到 120fps；正式目标是在支持 120Hz 的代表性环境与代表性业务数据下，高频交互不出现系统性掉帧或明显 Jank。

同时：

```text
120Hz optimization
must not break 60Hz behavior
```

---

## 15.1 Performance Principles

O3Pilot 的性能优先级：

```text
Input responsiveness
>
motion continuity
>
decorative richness
```

正式规则：

1. 用户输入必须优先得到立即反馈；
2. 不为了完整播放动画延迟新的用户意图；
3. 不为了更丰富的 Motion 牺牲交互稳定性；
4. 不把后台同步、解析或计算工作放进高频交互关键路径；
5. 简单、稳定、流畅的交互优先于复杂但掉帧的视觉效果；
6. 性能降级只能减少 Motion 或视觉复杂度，不得改变功能、业务结果、信息完整性或安全边界。

正式关系：

```text
Simpler smooth interaction
>
richer janky interaction
```

---

## 15.2 Frame Budget

`~8.33ms / frame` 是整个浏览器渲染周期的共享预算，而不是业务 JavaScript 可以独占的执行时间。

因此禁止以如下方式理解 120Hz：

```text
JavaScript runs for 8.33ms
then browser renders separately
```

高频交互路径应尽量留下足够空间给 Style / Layout / Paint / Composite 与浏览器自身工作。

在性能审查中，单次明显阻塞主线程的长任务属于重点问题。

正式警示线：

```text
Long Task ≥ 50ms
on an interaction-critical path
→ performance defect
```

这不表示任何业务任务都不得超过 50ms，而是表示长计算不得同步阻塞高频用户交互。

---

## 15.3 Scope

120Hz Performance Contract 主要约束用户直接感知的实时交互：

```text
Scrolling
Navigation feedback
Hover / Focus / Press
Tabs
Dropdown / Popover
Drawer / Modal
Sidebar collapse
Table interaction
Chart interaction
Drag / gesture if introduced
```

以下属于任务完成延迟，不直接使用 FPS 衡量：

```text
API waiting
Background sync
Import parsing
Forecast computation
Large local calculation
```

但这些任务仍不得无必要阻塞主线程或让 Shell、Navigation、输入控件失去响应。

正式关系：

```text
Rendering smoothness
≠ Task completion latency
```

---

## 15.4 Input Responsiveness

高频操作包括但不限于：

```text
Navigation
Filter
Tab
Copy
Sort
Pagination
Search
Shop Context switching
Reporting Currency switching
```

正式行为：

```text
User input
→ immediate visible feedback
→ heavy work separately when needed
```

禁止：

```text
click
→ synchronous heavy calculation
→ visible freeze
→ feedback appears later
```

后台工作可以持续，但当前 Shell、Navigation、Focus、输入状态与取消/切换操作必须保持可响应。

---

## 15.5 Layout / Style / Paint

简单动效优先使用 CSS；Predetermined Motion 优先 CSS Animation。

只有 CSS 无法合理完成且确实需要逐帧控制时，才考虑 JavaScript Animation。

正式规则：

1. 不使用 JS `requestAnimationFrame` 实现可由 CSS 完成的动效；
2. 动画元素优先 `transform / opacity`；
3. 不在动画过程中持续交错读写 Layout；
4. 避免 Forced Synchronous Layout；
5. 避免高频触发大面积 Paint；
6. Sticky / Overlay / Drawer 的视觉效果不得依赖高成本持续重绘；
7. 大型阴影、模糊与 Backdrop Filter 不进入默认交互路径。

Layout Read / Write 正式顺序：

```text
Measure
↓
Batch state calculation
↓
Mutate
```

原则：

```text
Layout reads
and
layout writes
→ grouped
```

禁止反复执行：

```text
read layout
→ write style
→ read layout
→ write style
→ ...
```

---

## 15.6 Compositor and Layer Rules

Compositor Promotion 必须是有目的的，而不是全局“性能优化”。

正式原则：

```text
Compositor promotion
→ intentional
not universal
```

`will-change`：

```text
→ targeted
→ temporary where practical
```

禁止：

```css
* {
  will-change: transform;
}
```

也不得为了所谓“开启 GPU”而普遍加入：

```css
transform: translateZ(0);
```

大型动画 Blur、`backdrop-filter`、持续变化的大面积 Shadow 默认禁止。

Overlay 优先：

```text
simple scrim opacity
+
surface transform / opacity
```

---

## 15.7 Motion Concurrency

高性能不只取决于单个动画是否简单，也取决于同一时间有多少区域发生变化。

正式原则：

```text
Motion should be localized
```

一次用户操作只应让与当前意图直接相关的 Surface 发生必要 Motion。

禁止典型整页编舞：

```text
Dashboard load
→ KPI entrance
→ chart entrance
→ row stagger
→ sidebar motion
→ decorative motion
```

同时发生。

正式关系：

```text
Active interaction
→ animate the relevant surface

Whole-page choreography
→ prohibited
```

---

## 15.8 Data-intensive Tables

O3Pilot 的 Orders、Products、Inventory 等页面可能长期承载高密度数据。

正式原则：

```text
Large Table
→ scrolling and interaction performance are primary
```

禁止：

- 大表 Scroll Handler 每帧执行大量业务计算；
- 每行绑定持续 Motion；
- 每个 Cell 创建不必要的复杂 Observer；
- Hover 导致整表 Reflow；
- 单个数据变化导致整表无必要重建；
- 为动画效果关闭合理的 Pagination / Virtualization；
- 在滚动路径加入高成本 Blur、Shadow 或同步测量。

分页、Virtualization 或其他数据窗口策略由具体数据规模和技术方案决定，但最终结果必须满足：

```text
Large dataset
→ remains interactive
```

Table 的正式视觉与交互规则由第 23 章定义。

---

## 15.9 Chart Performance

Chart 不做大规模首屏 Entrance Animation，也不为了视觉连续性强制对所有数据点进行动画过渡。

正式原则：

```text
Chart data complexity
→ must not become unbounded DOM complexity
```

大量数据点时，可以根据具体 Chart 实现采用：

- aggregation；
- sampling / downsampling；
- 更适合的大数据 Renderer；
- 减少不可见 DOM；
- 降低无业务价值的点级交互。

但任何性能优化都不得改变 Metric Contract 或制造错误业务含义。

正式关系：

```text
Data fidelity
≠ render every raw point simultaneously
```

禁止为了“完整展示”默认要求：

```text
10,000 raw points
+
10,000 DOM / SVG nodes
+
animated transition
```

---

## 15.10 Data Refresh Performance

数据刷新应尽量局部更新受影响区域。

正式原则：

```text
Data refresh
→ update affected regions
```

避免：

```text
one metric changes
→ rebuild entire page
```

以下全局 Context 变化可能触发较大范围的数据重新查询或计算，必须纳入 Performance QA：

```text
Shop Context
Reporting Currency
Date Range
```

即使业务内容正在重新计算：

- Sidebar 不应冻结；
- Global Context Bar 不应冻结；
- 当前 Focus 不应无理由丢失；
- 用户应可以继续执行导航或取消/改变当前选择；
- 无关页面结构不应因数据刷新产生明显 Layout Jank。

---

## 15.11 Sidebar and Layout Exception

Sidebar 的正式宽度状态：

```text
248px → 72px
```

如果持续 Width Animation 在真实数据页面造成 Layout Churn 或无法稳定高刷，则正式实现可以降级为：

```text
Layout width instant switch
+
label opacity
+
icon transform / local feedback
```

优先保证响应，而不是强制动画 Layout。

这一原则不只适用于 Sidebar。

任何 Layout Motion 都应遵循：

```text
Can compositor-only motion explain the change?
→ Yes: prefer it

No?
↓
Does layout animation materially improve comprehension?
→ Yes: consider it

Otherwise
→ snap layout
```

---

## 15.12 Performance Degradation Strategy

正式降级层级：

```text
Full intended motion
↓
Reduced compositor-only motion
↓
Instant layout + functional feedback
```

触发降级时：

- 功能必须保持不变；
- 用户意图必须立即生效；
- 业务数据不得缺失；
- Accessibility 不得下降；
- 不得因为“性能优化”隐藏状态、错误、风险或数据来源。

正式原则：

```text
Visual continuity
<
Responsiveness
```

---

## 15.13 Implementation Anti-patterns

禁止：

```text
setInterval(..., 8.33)
setTimeout(..., 8)
manual fixed-120fps loop
```

浏览器负责适配真实显示刷新率。

确实需要逐帧交互时才允许使用：

```text
requestAnimationFrame
```

但仍遵循：

```text
CSS can do it
→ do not use rAF
```

正式关系：

```text
120Hz support
≠ manually request 120 updates per second
```

另外禁止：

- `transition: all`；
- 动画过程重复读取 `getBoundingClientRect()` 后立即写 Layout；
- 全局 `will-change`；
- 普遍 `translateZ(0)`；
- Scroll 期间同步执行重业务逻辑；
- 大量 Element 同时 Entrance Animation；
- 大表逐 Cell / Row 的持续动画；
- 大型 Blur / Backdrop Filter 跟随高频 Motion；
- 仅为了 Motion 引入并常驻不必要的 Runtime Library。

---

## 15.14 Performance QA

Performance QA 必须使用：

```text
Production build
+
representative dataset
+
representative page complexity
```

不能只在：

```text
10 rows
1 KPI
empty chart
Development build
```

条件下宣称性能通过。

至少覆盖：

```text
Dashboard
Large Orders Table
Products
Inventory
Multi-series Chart
Sidebar Collapse
Drawer
Global Search
Rapid Filter Switching
Shop Context Switching
Reporting Currency Switching
```

同时验证：

- 120Hz-capable 环境下没有系统性 Frame Miss；
- 代表性交互没有肉眼可见的持续 Jank；
- 60Hz 环境行为正常；
- Long Task 不出现在高频交互关键路径；
- Scroll 不被持续业务计算阻塞；
- 大表、大图表仍可交互；
- Motion Degradation 不改变业务功能或信息；
- Background Sync / Parsing / Calculation 不让 Shell 失去响应；
- Reduced Motion 模式下性能至少不劣于标准 Motion 模式。

正式验收关系：

```text
120Hz-capable environment
→ no systematic frame misses
→ no visible interaction jank in representative flows
```

---

# 16. Reduced Motion

所有正式 Motion Pattern 都必须同时定义 Reduced Motion 行为。

Reduced Motion 的目标不是删除所有视觉反馈，而是在不损失状态理解、操作确认和可访问性的前提下，移除不必要的空间运动、持续运动和装饰性运动。

正式原则：

```text
Reduced Motion
≠ No Feedback
≠ Disable Every Transition
```

以及：

```text
Keep State Clarity
Remove Unnecessary Spatial Movement
Remove Continuous Motion
Remove Decorative Motion
```

---

## 16.1 Principles

Reduced Motion 必须服从以下优先级：

```text
State Comprehension
>
Interaction Feedback
>
Spatial Continuity
>
Decorative Motion
```

因此：

```text
State clarity
>
spatial continuity
```

在 Reduced Motion 下仍必须清楚表达：

- 当前选中状态；
- 点击或键盘操作是否被接受；
- Copy / Save Local State 等本地操作是否成功；
- Drawer / Modal / Popover 是否已经打开或关闭；
- Loading / Error / Success / Warning 等状态；
- Focus 所在位置；
- 数据是否正在更新、已更新或不可用。

Reduced Motion 只改变表现方式，不改变：

- 功能；
- 路由；
- 数据；
- 业务结果；
- 控件可用性；
- O3Pilot Read-only 边界。

---

## 16.2 Preference Source

v1 正式使用操作系统提供的：

```css
@media (prefers-reduced-motion: reduce) {
  /* 每个正式 Motion Pattern 使用其 Reduced Motion 替代方案。 */
}
```

正式关系：

```text
OS Accessibility Preference
→ O3Pilot Motion Mode
```

v1 不额外提供独立的 O3Pilot “减少动态效果”开关。

未来如果加入应用级 Motion Preference：

```text
User explicitly requests less motion
→ allowed

OS requests reduced motion
→ app must not override with more motion
```

即：

```text
OS Reduce
→ never overridden with more motion
```

---

## 16.3 Reduction Hierarchy

Reduced Motion 按以下顺序处理：

```text
Decorative motion
→ remove

Continuous motion
→ remove or replace with static state

Large spatial movement
→ replace with final-position appearance

Scale / bounce / spring
→ remove

Path morph / rotation used only for delight
→ remove

Small functional opacity / color feedback
→ may remain

Instant state change
→ always acceptable
```

不允许只把大型位移动画“加速播放”来假装完成 Reduced Motion。

例如：

```text
220ms large slide
→ 50ms large slide
```

仍然属于大幅空间移动，不是合格替代。

正式目标：

```text
Reduce travel distance first
then reduce duration if needed
```

---

## 16.4 Navigation and Morphicons

Sidebar 正常模式：

```text
Original
→ Check
→ Original
```

总时长约：

```text
180–220ms
```

Reduced Motion 下：

```text
Original
→ instant Check state
→ brief confirmation hold
→ Original
```

允许：

- 极短的 opacity；
- color change；
- 静态 Check 状态替换。

禁止：

```text
Path Morph
Rotation
Scale
Displacement
Bounce
```

最重要的行为保持不变：

```text
Navigation
→ immediate

Reduced Motion feedback
→ non-blocking
```

用户快速连续点击时，前一次反馈必须立即被新状态覆盖，不排队播放。

同样规则适用于：

```text
Copy
→ Check
```

等 Morphicons 本地成功反馈。

---

## 16.5 Overlay Motion

### Drawer

正常模式可使用：

```text
translate + opacity
```

Reduced Motion：

```text
final position immediately
+ optional short opacity
```

不得保留明显水平旅行距离。

### Modal

Reduced Motion：

```text
final position immediately
+ optional short opacity
```

禁止：

- zoom；
- spring；
- bounce；
- 大尺度 scale。

### Popover / Dropdown / Tooltip

Reduced Motion：

```text
no scale
no displacement
optional short opacity
```

### Command Palette

`⌘K / Ctrl+K` 必须立即可用。

Reduced Motion 下默认直接显示最终位置；不得因动画增加可感知输入延迟。

---

## 16.6 Loading and Continuous Motion

持续 Motion 在 Reduced Motion 下默认移除。

包括：

```text
Spinner
Skeleton Shimmer
Infinite Pulse
Rotating Sync Icon
Indeterminate Decorative Loop
```

正式规则：

```text
Standard Mode
→ subtle progress motion may be used when appropriate

Reduced Motion
→ static loading indicator
+ text / real progress state
```

优先显示真实状态，例如：

```text
加载中…
正在同步…
正在生成预测…
63%
```

Skeleton 可以保留静态占位结构，但：

```text
Skeleton
→ no shimmer in Reduced Motion
```

如果任务具有真实进度，真实 Progress 优先于无限循环动画。

---

## 16.7 Data and Chart Motion

业务数据在 Reduced Motion 下保持稳定。

正式规则：

```text
Business data
→ stable
```

Chart：

```text
Initial load
→ no entrance animation

Data refresh
→ redraw directly

Series visibility toggle
→ immediate or minimal non-spatial feedback
```

禁止：

- Line draw animation；
- Bar grow animation；
- Pie rotation；
- Point travel；
- 数字 Count-up；
- 翻牌；
- 数据点 Bounce；
- 从 0 动画到实际业务值。

数据变化本身不是需要动画庆祝的事件：

```text
Data change
≠ animation event
```

需要表达“刚刚更新”时，应使用状态文本、时间戳或轻量非空间反馈，而不是让数据值本身持续运动。

---

## 16.8 Scroll Behavior

Reduced Motion 下禁止程序化 Smooth Scroll。

例如：

```text
Global Search result location
Anchor navigation
Form validation error location
Detail location
Jump-to-related-record
```

正常模式若使用：

```js
scrollIntoView({ behavior: "smooth" })
```

Reduced Motion 必须改为：

```text
auto / instant
```

正式规则：

```text
Reduced Motion
→ no programmatic smooth scrolling
```

用户自己的原生滚动行为不由 O3Pilot 强制改写。

---

## 16.9 Hover / Focus / Feedback

Reduced Motion 不删除必要的 Hover / Focus / Selected 反馈。

可以继续使用：

```text
Background
Border
Color
Focus Ring
Selected Surface
Static Icon State
```

应移除：

```text
translateY
scale
bounce
rotation
morph used only for visual delight
```

正式关系：

```text
State feedback
→ preserved

Motion feedback
→ reduced
```

Focus 尤其不得被削弱：

```text
Reduced Motion
must not reduce focus visibility
```

Focus Ring 的存在不得依赖动画完成后才可见。

---

## 16.10 Toast and Notifications

正常模式 Toast 可以使用轻量进入 / 退出 Motion。

Reduced Motion：

```text
Toast
→ appear in final position
→ optional short opacity
```

禁止从视口边缘进行明显长距离滑入或弹跳。

同时必须区分：

```text
Toast display duration
≠ animation duration
```

Reduced Motion 不得缩短用户阅读 Toast 的可用时间。

Alert / Notification 的业务严重程度、内容和可访问语义保持不变。

---

## 16.11 Implementation Rules

O3Pilot 不使用全局 `0.01ms` Hack 作为 Reduced Motion 的正式实现策略。

例如不把以下模式作为设计 Contract：

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

原因包括：

- 可能破坏依赖 Transition / Animation Event 的实现；
- 可能一并删除本应保留的 Color / Focus Feedback；
- 容易掩盖组件本身未定义 Reduced Motion 的问题；
- 无法表达不同 Motion Pattern 应采取的不同替代方式。

正式原则：

```text
Reduced Motion
→ component-defined alternative

not
→ global timing hack
```

可以存在有限的全局安全防线，但每一个进入正式 Design System 的 Motion Pattern 仍必须显式定义 Reduced Motion 行为。

Feature 不得自行绕过该 Contract。

---

## 16.12 Reduced Motion QA

正式 QA 至少覆盖：

```text
Sidebar navigation
Nav Original → Check → Original
Copy → Check
Tabs
Dropdown
Popover
Tooltip
Drawer
Modal
Command Palette
Toast
Loading
Spinner
Skeleton
Chart initial load
Chart refresh
Series toggle
Programmatic scroll locations
Keyboard Focus
```

同时测试：

```text
Mouse
Keyboard
120Hz-capable display
60Hz display
Light
Dark
CN
EN
RU
```

必须确认：

- 大幅空间运动已经移除或替换；
- Continuous Motion 已移除或替换为静态状态；
- 业务状态仍然清晰；
- Focus 仍清楚可见；
- 导航、点击和键盘反馈仍立即响应；
- Business Data 不因 Reduced Motion 改变数值、语义或更新逻辑；
- Reduced Motion 不造成新的布局跳动；
- OS `prefers-reduced-motion: reduce` 能覆盖所有正式 Motion Pattern；
- 不依赖全局 `0.01ms` Hack 才能通过验收。

正式验收原则：

```text
Reduced Motion ≠ No Feedback

State clarity > spatial continuity

Continuous motion → remove

Large movement → replace

Business data → stable

Focus visibility → always preserved

OS Reduce
→ never overridden with more motion

Component-defined alternative
>
global timing hack
```

---

# 17. Data State Language

Data State Language 是 O3Pilot 的核心设计特征之一。

本章不重新定义数据状态或指标状态，只定义上游 Contract 中已经存在的状态如何进入 UI。

正式关系：

```text
DATA_MODEL.md
→ Value / Availability State Truth

METRICS.md
→ Metric Status Truth

DESIGN.md
→ User-facing Presentation Mapping
```

因此：

```text
Value State
≠ Metric Status
≠ Retrieval / System State
```

也必须始终满足：

```text
0
≠ Missing
≠ Unknown
≠ Unavailable
```

---

## 17.1 State Model

O3Pilot UI 至少区分以下三个层级：

```text
Value State
Metric Status
Retrieval / System State
```

三者可以同时存在。

例如：

```text
利润
¥18,482

Metric Status: PARTIAL
Coverage: 96.7%
Retrieval State: current
```

也可能出现：

```text
广告 SKU 数据
12,842 RUB

Metric Status: STALE
Retrieval State: latest refresh failed
Last valid: 昨日 23:59
```

禁止为了界面简洁，把不同层级状态合并成一个不明确的：

```text
NO_DATA
异常
不可用
```

---

## 17.2 Known Zero

`0` 是有效业务值，不属于缺失状态。

例如：

```text
退货数
0
```

必须明确区分：

```text
0
→ Known Zero

—
→ No numeric value available
```

正式原则：

```text
Numeric Value
≠ Data State
```

以及：

```text
Known Zero
→ valid business value
→ never rendered as missing
```

不得将：

```text
0
0.00
0%
```

用作以下任何状态的视觉替代：

```text
NULL
UNAVAILABLE
NOT_APPLICABLE
UNMATCHED
UNVERIFIED
NO_DENOMINATOR
```

---

## 17.3 Value State

Value State 直接复用 `DATA_MODEL.md` 已正式定义的 taxonomy：

```text
KNOWN
NULL
UNAVAILABLE
NOT_APPLICABLE
UNMATCHED
UNVERIFIED
```

以及真实业务值：

```text
0
```

`DESIGN.md` 不增加第三套 Value State。

### KNOWN

存在明确有效值。

UI：

```text
→ 正常显示业务值
```

### NULL

当前没有可展示值。

内部可以是：

```text
NULL
```

但最终用户界面默认不得直接显示字符串：

```text
NULL
```

UI 应显示：

```text
—
```

并在原因已知时提供解释。

### UNAVAILABLE

该数据当前不可获得，例如能力、权限或来源不可用。

典型 UI：

```text
Reviews
N/A
当前店铺无此数据权限
```

`UNAVAILABLE` 不等于 `0`，也不等于当前筛选范围没有记录。

### NOT_APPLICABLE

该字段或指标对当前对象不适用。

典型 UI：

```text
N/A
不适用于当前对象
```

不得把 `NOT_APPLICABLE` 当成错误或风险状态。

### UNMATCHED

`UNMATCHED` 是 Entity / Source Relationship State。

典型场景：

- Seller-owned Cost 无法匹配商品；
- 导入记录无法匹配 SKU；
- 外部记录无法匹配订单。

典型 UI：

```text
成本记录未匹配商品
SKU: ABC-123
```

`UNMATCHED` 不作为任意 KPI 的通用状态。

### UNVERIFIED

数据或映射尚未完成验证。

典型 UI：

```text
指标暂未验证
当前不作为正式决策指标
```

UI 是否显示具体值，由对应上游 Contract 决定；若显示，必须同时保留 `UNVERIFIED` 语义。

---

## 17.4 Metric Status

Metric Status 直接复用 `METRICS.md` 已正式定义的 taxonomy：

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

`DESIGN.md` 不增加、合并或重命名这些正式状态。

### VALID

指标按当前 Metric Contract 有效。

默认视觉：

```text
Quiet
```

不要求每个有效指标都显示绿色 `VALID` Badge。

### PARTIAL

指标存在可展示结果，但源数据覆盖不完整。

典型 UI：

```text
利润
¥18,482

数据覆盖 96.7%
部分订单缺少成本
```

如果有 `coverage_ratio`，应优先展示。

### UNAVAILABLE

指标当前不可获得。

UI 必须尽可能显示已知原因，例如：

```text
N/A
当前店铺无此数据权限
```

### NOT_APPLICABLE

指标对当前对象、场景或时间窗口不适用。

不应使用 Danger / Warning 语义。

### UNVERIFIED

指标尚未完成正式验证。

UI 必须明确：

```text
未经验证
```

不能以普通正式指标样式无差别呈现。

### STALE

当前显示的是 Last Known Value，而不是当前最新事实。

典型 UI：

```text
广告 SKU 数据
12,842 RUB

数据已过期
最后有效：昨日 23:59
```

必须同时满足：

```text
Last Known Value
→ remains visible when useful
→ remains identifiable as stale
```

### ESTIMATED

指标依赖假设、近似或缺失数据补全。

典型 UI：

```text
估算利润
¥18,482
估算
```

`ESTIMATED` 不自动等于 Warning。

### NO_DENOMINATOR

对应 `METRICS.md` 中：

```text
denominator = 0
→ value = NULL
→ status = NO_DENOMINATOR
```

推荐显示：

```text
退货率
—
无可计算样本
```

不得显示：

```text
0%
```

### NO_RECENT_DEMAND

当前窗口不存在足够的近期有效需求事实。

UI 使用对应 Metric Contract 允许的值或 `—`，并明确原因：

```text
—
近期无有效需求数据
```

不得直接解释为需求等于 `0`，除非该指标 Contract 明确如此定义。

### OPEN_COHORT

Cohort 尚未闭合，当前结果仍会随时间变化。

UI 应明确：

```text
当前 Cohort 尚未闭合
```

如果 Metric Contract 允许展示当前结果，可显示值，但不得伪装成最终稳定结果。

---

## 17.5 Retrieval / System State

业务数据状态与 Retrieval / System State 必须分离。

例如：

```text
Business Data State
≠ Request Error
≠ Sync Error
≠ Parser Error
```

请求失败时，如果存在 Last Known Value，可以显示：

```text
利润
¥18,482

数据更新失败
最后有效：10:28
```

禁止将：

```text
ERROR
```

直接解释成：

```text
0
NULL
UNAVAILABLE
```

也禁止因为刷新失败而静默清除最后有效值。

正式原则：

```text
Error
≠ Zero
≠ Missing
```

错误文案应说明当前失败发生在哪一层：

```text
加载失败
同步失败
解析失败
计算失败
```

而不是统一显示：

```text
暂无数据
```

---

## 17.6 Empty Result

`NO_DATA` 不作为 O3Pilot 的通用正式业务状态。

因为“没有数据”可能代表完全不同的情况：

```text
当前筛选条件没有记录
当前时间窗口没有业务事件
没有权限
来源不可用
同步尚未完成
同步失败
没有分母
没有近期需求
Entity 未匹配
```

UI 必须尽可能表达真实原因。

例如：

```text
当前筛选条件下没有订单
```

属于 Empty Result。

而：

```text
当前店铺无 Reviews 数据权限
```

属于 `UNAVAILABLE`。

两者不得使用完全相同的状态文案。

正式原则：

```text
NO_DATA
≠ universal state
```

---

## 17.7 Last Known Value

当最新刷新失败、数据过期或来源暂时不可用，但历史有效值仍有业务价值时，可以保留 Last Known Value。

必须同时显示：

```text
Value
Status / Reason
Last Valid Time
```

例如：

```text
广告 SKU 数据
12,842 RUB

数据已过期
最后有效：昨日 23:59
```

禁止：

```text
旧值
→ 继续按当前值样式展示
→ 不显示 Stale / Last Valid Time
```

也禁止在已有 Last Known Value 时，因为一次 Retrieval Error 就将值替换为 `0`。

---

## 17.8 User-facing Display Mapping

内部状态代码与最终用户文案分离。

正式关系：

```text
Internal State Code
≠ User-facing Copy
```

推荐基础映射：

| Formal State | Primary Value | Supporting Copy |
|---|---|---|
| `VALID` | 正常显示 | 通常无需状态 Badge |
| `NULL` | `—` | 根据已知原因解释 |
| `UNAVAILABLE` | `N/A` 或 `—` | 明确权限 / 能力 / 来源不可用原因 |
| `NOT_APPLICABLE` | `N/A` | 说明不适用于当前对象 |
| `UNMATCHED` | 保留原记录 | 明确未匹配对象与原因 |
| `UNVERIFIED` | 按 Contract 决定是否显示值 | 明确未经验证 |
| `PARTIAL` | 显示可用值 | Coverage + 缺失原因 |
| `STALE` | 显示 Last Known Value | 最后有效时间 |
| `ESTIMATED` | 显示估算值 | 明确“估算” |
| `NO_DENOMINATOR` | `—` | 无可计算样本 |
| `NO_RECENT_DEMAND` | 按 Contract 显示值或 `—` | 近期无有效需求数据 |
| `OPEN_COHORT` | 按 Contract 展示当前结果 | Cohort 尚未闭合 |

`N/A` 不是万能占位符。

推荐语义：

```text
—
→ no numeric value available

N/A
→ genuinely unavailable / not applicable
```

因此以下状态不能全部无差别显示为 `N/A`：

```text
Retrieval Error
NO_DENOMINATOR
NO_RECENT_DEMAND
UNMATCHED
STALE
```

---

## 17.9 Status Composition

一个最终展示状态通常不是单个 Badge，而是多个信息的组合。

正式结构：

```text
Value
+
Formal Status
+
Reason when known
+
Coverage when relevant
+
As-of / Last Valid when relevant
+
Origin when relevant
```

例如：

```text
利润
¥18,482

部分数据 · Coverage 96.7%
部分订单缺少成本
截至 10:20
O3Pilot 计算
```

禁止把所有状态信息压缩成一个：

```text
Warning
```

或：

```text
异常
```

Status Composition 的视觉层级由第 18 章 Metric Presentation 继续定义。

---

## 17.10 Color and Icon Rules

状态代码本身不直接决定严重程度。

正式关系：

```text
State
≠ Severity
```

例如：

```text
VALID
NOT_APPLICABLE
ESTIMATED
OPEN_COHORT
```

不应机械映射为：

```text
Green
Gray
Yellow
Yellow
```

颜色必须遵守第 9 章：

```text
Normal
→ Quiet

Attention
→ Color
```

Icon 同样不能成为唯一状态表达。

必须满足：

```text
Status Icon
+
Status Label / Accessible Meaning
```

不得只依赖：

```text
✓
!
?
```

来表达正式业务状态。

---

## 17.11 Localization

状态本地化必须保留正式语义。

例如：

```text
PARTIAL
```

在中文、英文、俄语中可以使用不同显示文案，但不能因为翻译而改变为：

```text
ERROR
WARNING
UNKNOWN
```

内部 Formal State Code 保持稳定。

用户可见 Copy 必须适合当前语言，但至少满足：

- 不把未知翻译成 `0`；
- 不把无权限翻译成“没有数据”；
- 不把 `ESTIMATED` 翻译成“已确认”；
- 不把 `OPEN_COHORT` 翻译成“最终结果”；
- 不把 `NOT_APPLICABLE` 翻译成错误；
- `UNMATCHED` 必须保留对象关系未匹配的语义。

---

## 17.12 Data State QA

正式 Data State QA 至少覆盖：

```text
Known positive value
Known zero
NULL
UNAVAILABLE
NOT_APPLICABLE
UNMATCHED
UNVERIFIED
VALID
PARTIAL
STALE
ESTIMATED
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
Empty Result
Retrieval Error
Last Known Value
```

同时至少检查：

```text
Metric Tile
Table Cell
Table Row
Chart Label / Tooltip
Detail Drawer
Data Center
Import Diagnostics
Global Data Status
```

QA 必须验证：

1. `0` 从未被显示成缺失；
2. 缺失、无权限、不适用、未匹配、未验证互不混淆；
3. `NO_DENOMINATOR` 不显示 `0%`；
4. Retrieval Error 不会静默覆盖业务状态；
5. Last Known Value 始终可识别为非当前值；
6. `PARTIAL` 在存在 Coverage 时可发现覆盖率；
7. `ESTIMATED` 与 `O3P_ESTIMATE` Origin 不被混成同一概念；
8. `O3P_FORECAST` Origin 不自动映射为 Warning；
9. Internal State Code 与用户文案分离；
10. 中文、英文、俄语都保持相同正式语义；
11. 状态不只依赖颜色或 Icon；
12. 不存在通用 `NO_DATA` 占位符吞掉真实原因。

核心验收原则：

```text
0 ≠ Missing

Value State
≠ Metric Status
≠ Retrieval State

Internal Code
≠ User-facing Copy

NO_DATA
≠ universal state

Error
≠ Zero
≠ Missing

Origin
≠ Status

Last Known Value
→ must remain identifiable as stale

Reason
→ shown whenever known
```

---

# 18. Metric Presentation

Metric Presentation 定义一个正式指标如何在 O3Pilot UI 中被阅读、比较、追溯和判断可信度。

本章定义的是信息结构，不要求每个指标都使用独立 Card。

正式关系：

```text
Metric Presentation
≠ Mandatory Card
```

所有指标展示都必须继续服从 `METRICS.md` 的正式 Metric Contract，以及第 17 章的数据状态语言。

---

## 18.1 Metric Presentation Principles

正式原则：

```text
Primary Value
>
Status requiring attention
>
Comparison
>
Origin / Coverage / As-of Metadata
```

但该顺序不是机械排版顺序，而是视觉优先级。

必须始终满足：

```text
Origin
≠ Status

Direction
≠ Outcome

pp
≠ %

Traceable
≠ Everything visible by default
```

Metric Presentation 可以存在于：

- Dashboard KPI 区；
- 普通分析 Section；
- Bounded Surface；
- Table；
- Detail Drawer；
- Recommendation / Diagnostic Context。

不要求每个指标拥有：

- 独立背景；
- 独立圆角；
- 独立 Shadow；
- 独立 Card Border。

---

## 18.2 Metric Anatomy

标准 Metric Anatomy：

```text
Metric Name
Primary Value
Comparison when valid
Status Context when relevant
Origin
Coverage when relevant
As-of / Last Valid when relevant
```

例如：

```text
产品退货率
4.82%
较上一周期 +0.4pp

O3Pilot 计算 · 截至 01:43
```

如果指标为 `PARTIAL`：

```text
利润
¥18,482

部分数据 · Coverage 96.7%
部分订单缺少成本
O3Pilot 计算 · 截至 10:20
```

如果指标为 `STALE`：

```text
广告收入
12,842 RUB

数据已过期
最后有效：昨日 23:59
Ozon 官方
```

`VALID` 默认不要求显式显示。

正式原则：

```text
VALID
→ normally implicit
```

---

## 18.3 Primary Value

Primary Value 必须优先表达指标的真实值与单位语义。

### Number

普通数字应使用第 8 章定义的 Numeric Typography。

例如：

```text
1,284
```

### Ratio / Percentage

展示为百分比时必须保留明确 Scale：

```text
4.82%
```

内部 0–1 与来源 0–100 的标准化规则仍由 `METRICS.md` 定义。

### Money

不得只显示缺少币种语义的：

```text
18,482
```

必须明确：

```text
¥18,482
18,482 RUB
$18,482
```

Reporting Currency 与跨币种规则由第 20 章继续定义。

### Unit

单位不是显而易见时必须可见，例如：

```text
12.4 天
3.8 kg
428 ms
96.7%
```

正式原则：

```text
Value without unit context
→ prohibited when unit is not self-evident
```

### Missing / Non-computable Value

无有效数值时必须遵守第 17 章，不得伪造 `0`。

例如：

```text
退货率
—
无可计算样本
```

---

## 18.4 Comparison

Comparison 必须说明比较基准。

正式结构：

```text
Comparison
=
Direction
+
Magnitude
+
Comparison Basis
```

推荐：

```text
较上一周期 +8.2%
较前 7 天 -3.1%
较上一周期 +0.4pp
```

不推荐：

```text
↑ 8.2%
↓ 3.1%
```

因为缺少 Comparison Basis。

### Percentage Point vs Relative Percent

例如：

```text
4.0%
→ 4.4%
```

百分点变化：

```text
+0.4pp
```

相对变化：

```text
+10%
```

两者不得混用。

正式原则：

```text
Percentage-point change
≠ Relative percent change
```

### Direction vs Business Outcome

箭头或正负号只表达方向。

```text
↑ / ↓ / + / -
→ Direction
```

业务含义必须由 Metric Contract / Context 决定。

例如：

```text
销售额 ↑
→ may be positive

退货率 ↑
→ may be negative
```

禁止全局实现：

```text
↑ = Success
↓ = Danger
```

### Comparable Basis Required

只有在两个结果具有可比的：

```text
Metric Contract
Time Basis
Window
Scope
Status
```

时才显示 Comparison。

尤其 `STALE`、`PARTIAL` 或来源覆盖不一致时，必须确认 Comparison 仍然成立。

正式原则：

```text
Comparable basis
→ required before showing comparison
```

Primary Value 当前不可比较时：

```text
Comparison
→ suppressed
```

不得为了保持 KPI 版式完整而显示无意义 Delta。

---

## 18.5 Status Presentation

Status 直接使用第 17 章与 `METRICS.md` 的正式状态语义。

### VALID

默认安静，不要求 Badge：

```text
VALID
→ Quiet
→ normally implicit
```

### PARTIAL

存在 Coverage 时，应在 Primary Metric 附近可发现：

```text
部分数据 · Coverage 96.7%
```

不能只藏在 Detail 中。

### STALE

必须使 Last Known Value 可识别：

```text
数据已过期
最后有效：昨日 23:59
```

### ESTIMATED

必须明确：

```text
估算
```

但 `ESTIMATED` 不自动映射 Warning。

### UNVERIFIED

必须明确未经验证，不能按普通 `VALID` Metric 无差别呈现。

### OPEN_COHORT

如果允许展示当前值，必须说明 Cohort 尚未闭合。

Status 不得被压缩成只有颜色、Icon 或一个模糊的：

```text
异常
Warning
```

---

## 18.6 Origin Presentation

Origin 与 Status 分离。

正式 Origin 映射直接对应 `METRICS.md`：

```text
OZON_SOURCE
→ Ozon 官方

OZON_REFERENCE
→ Ozon 参考

O3P_DERIVED
→ O3Pilot 计算

O3P_ESTIMATE
→ O3Pilot 估算

O3P_FORECAST
→ O3Pilot 预测
```

正式关系：

```text
Metric Origin
≠ Metric Status
```

例如：

```text
O3P_ESTIMATE
→ Origin

ESTIMATED
→ Status
```

两者可能同时出现，但不得合并成同一字段或同一个不明确的 Badge。

Origin 默认使用弱强调 Metadata：

- neutral；
- small；
- text 或 text + icon；
- 不使用大面积状态色。

继续保持：

```text
Ozon 官方
≠ Green
≠ Certification Checkmark

O3Pilot 预测
≠ Warning by default
```

`Ozon 官方` 不使用会暗示品牌认证、授权或 endorsement 的“认证蓝勾”。

---

## 18.7 Coverage and As-of

### Coverage

`METRICS.md` 中 Metric Result 可包含：

```text
value
status
sample_count
eligible_count
coverage_ratio
as_of_time
```

Coverage 的主展示规则：

```text
VALID + complete / expected coverage
→ may remain quiet

PARTIAL
→ coverage visible near the value when available
```

例如：

```text
利润
¥18,482
部分数据 · Coverage 96.7%
```

用户不应必须打开 Detail 才发现指标不完整。

### As-of

正常且新鲜时，可作为轻量 Metadata：

```text
截至 10:28
```

如果状态为 `STALE`，必须升级为明确可信度信息：

```text
数据已过期
最后有效：昨日 23:59
```

正式原则：

```text
As-of
≠ decorative timestamp
```

Data Freshness 的全局规则由第 19 章继续定义。

---

## 18.8 Metric Density Variants

O3Pilot 正式使用三种 Metric Density Variant：

```text
Prominent Metric
Standard Metric
Inline Metric
```

### Prominent Metric

用于：

- Dashboard 关键 KPI；
- 页面首要业务状态；
- 需要较强视觉层级的决策指标。

特点：

- Primary Value 更大；
- Comparison 可直接显示；
- 需要关注的 Status 直接显示；
- Origin / As-of 保持弱强调但可发现。

### Standard Metric

用于：

- 普通分析 Surface；
- Section 内指标；
- Detail Summary。

使用标准字号与 Metadata 层级。

### Inline Metric

用于：

- Table；
- Detail Row；
- Compact Context。

优先：

```text
Value
+
必要 Status
```

Origin / Detail 按需披露。

三个 Variant 只改变视觉密度，不改变：

```text
Metric Contract
Status Semantics
Origin Semantics
Comparison Semantics
```

正式原则：

```text
Visual density changes
≠ semantic changes
```

---

## 18.9 Metric Detail Disclosure

Metric Detail 使用统一 Disclosure Pattern，而不是只限定为 Popover。

推荐触发：

```text
Metric Name + explicit Info Control
```

不要默认让普通 Metric Name 本身暗示可点击。

Desktop 简单指标可以使用：

```text
Popover
```

复杂指标可以使用：

```text
Detail Drawer
```

Narrow 环境可以使用适合窄屏的 Detail Surface。

如果整个 Metric Surface 具有 Drill-down 行为，必须使用第 10 章定义的明确 Interactive Affordance。

### Detail 内容

正式 Metric Detail 至少支持：

```text
Metric Name
Metric Code

Value
Unit / Currency

Origin
Status

Definition
Time Basis
Window
Grain / Scope

Sample Count
Eligible Count
Coverage

As-of
Metric Version
```

Ratio 类指标在有解释价值时，可继续显示：

```text
Numerator
Denominator
```

不要求每个简单指标默认展开所有技术字段。

正式原则：

```text
Traceable
≠ Everything visible by default
```

---

## 18.10 Unavailable / Non-comparable Metrics

如果 Primary Value 不可计算、不可获得或不适用，主展示必须继续保留原因。

例如：

```text
退货率
—
无可计算样本
```

```text
Reviews
N/A
当前店铺无此数据权限
```

```text
预测需求
—
近期无有效需求数据
```

不得继续显示无效 Comparison。

正式规则：

```text
Primary Value not comparable
→ comparison suppressed
```

对于 `UNVERIFIED`，是否显示具体值由上游 Contract 决定；若显示，必须同时保留未经验证语义。

对于 `STALE`，只有比较双方仍满足相同可比 Basis 时，才允许显示 Comparison。

---

## 18.11 Responsive and Accessibility

Metric Presentation 在 Desktop / Compact / Narrow 下可以改变布局，但不能删除关键语义。

Narrow 下可：

- Comparison 换行；
- Origin / As-of 移到下一行；
- Detail 使用 Drawer / 全宽 Surface；
- Prominent Metric 降为更紧凑布局。

但不得因为窄屏：

- 隐藏 `PARTIAL`；
- 隐藏 `STALE`；
- 隐藏 `ESTIMATED`；
- 隐藏 Currency / Unit；
- 将未知显示为 `0`。

Accessibility：

1. Comparison 不只依赖颜色；
2. Direction 使用文字、符号或 Accessible Label；
3. Status 不只依赖 Icon；
4. Info Control 必须有 Accessible Name；
5. Tooltip 不是唯一可访问解释来源；
6. 数值、单位、状态在 Screen Reader 顺序中应保持可理解关系；
7. Detail Disclosure 必须可键盘操作。

---

## 18.12 Metric Presentation QA

正式 QA 至少覆盖：

```text
Known positive value
Known zero
Money
Ratio
Percentage-point comparison
Relative percent comparison
PARTIAL
STALE
ESTIMATED
UNVERIFIED
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
UNAVAILABLE
NOT_APPLICABLE
```

至少检查以下 Metric Origin：

```text
OZON_SOURCE
OZON_REFERENCE
O3P_DERIVED
O3P_ESTIMATE
O3P_FORECAST
```

至少检查以下 Surface：

```text
Dashboard
Analysis Section
Table
Detail Drawer
Diagnostic Surface
Narrow Layout
```

QA 必须验证：

1. Metric Presentation 不被实现成强制独立 Card；
2. `Origin` 与 `Status` 从未混成同一字段；
3. `VALID` 默认 Quiet；
4. Comparison 始终有明确 Basis；
5. `pp` 与相对 `%` 从未混用；
6. ↑ / ↓ 不机械绑定 Success / Danger；
7. `PARTIAL` 在存在 Coverage 时可直接发现 Coverage；
8. `STALE` 显示 Last Valid Time；
9. 无有效值时不显示虚假 Comparison；
10. Money 保留 Currency Context；
11. Origin Label 与正式 `metric_origin` 一一映射；
12. Ozon 官方不使用认证蓝勾；
13. Detail 可追溯到 Metric Code、Definition、Time Basis、Coverage、As-of 与 Version；
14. Narrow 下关键 Status / Currency / Unit 不被隐藏；
15. 中文、英文、俄语均保持相同 Metric Semantics；
16. Status / Direction / Comparison 不只依赖颜色。

核心验收原则：

```text
Metric Presentation
≠ Mandatory Card

Origin
≠ Status

VALID
→ Quiet

Comparison
→ always has a basis

pp
≠ %

Direction
≠ Outcome

PARTIAL
→ Coverage visible

STALE
→ Last Valid visible

Traceable
≠ Everything visible by default
```

---

# 19. Data Freshness

Data Freshness 是 O3Pilot 数据可信度表达的一部分，用于回答：

```text
这份数据最近何时被取得？
它实际覆盖到什么时间？
是否存在已知缺口？
当前展示的是最新有效状态，还是 Last Known Value？
```

Freshness 只负责呈现上游数据 Contract、同步系统和 Metric Result 已经确定的时效与覆盖事实，不在 `DESIGN.md` 中重新定义任何数据源 SLA、同步周期或业务状态语义。

正式原则：

```text
Latest Timestamp
≠ Freshness

Fetch Time
≠ Coverage Time

Freshness
≠ Coverage
≠ Availability
≠ Retrieval State

Historical Age
≠ Staleness
```

---

## 19.1 Freshness Principles

O3Pilot 的 Freshness 表达遵循：

```text
Current data
→ quiet when healthy

Delayed / Stale / Gap / Failed
→ visible when relevant

Reason
→ discoverable whenever known

Last Known Value
→ never presented as current without qualification
```

Freshness 不得通过单一“最后更新时间”替代完整的数据可信度信息。

以下事实可以同时成立：

```text
最近同步成功
+
存在历史覆盖缺口
```

也可以：

```text
最近一次同步失败
+
仍有可用 Last Known Value
```

因此 UI 必须允许状态组合，而不是把所有情况压成单一绿色 / 黄色 / 红色灯。

---

## 19.2 Freshness vs Coverage / Availability

必须区分：

### Freshness

当前数据是否满足该 Dataset 自己的时效要求。

### Coverage

目标时间范围是否已经取得预期数据，以及是否存在 Gap。

### Availability

当前 Shop / Capability 是否能够取得该类数据。

### Retrieval State

最近一次请求、同步、导入或计算是否成功。

正式关系：

```text
Fresh
≠ Complete

Available
≠ Fresh

Failed Refresh
≠ No Data
```

示例：

```text
Seller API
最后成功：10:28
状态：CURRENT

Coverage Gap
09:00–09:30
```

这里不能因为最后同步时间很新，就隐藏已经确认的 Gap。

---

## 19.3 Time Semantics

Freshness UI 至少区分以下时间语义：

```text
Observed / Synced At
Coverage Through
Business Time
```

### Observed / Synced At

O3Pilot 实际取得、导入或成功同步该数据的时间。

对应上游 `source_observed_at` 等事实。

### Coverage Through

当前已取得的数据实际覆盖到什么业务时间或区间。

例如：

```text
Performance API
最后同步：10:28
数据覆盖至：今日 10:00
```

### Business Time

来源自身表达的业务发生时间。

它不能被 Fetch Time 替代。

正式原则：

```text
Fetch Time
≠ Data Coverage Time
≠ Source Business Time
```

如果当前 UI 只展示一个时间，必须让用户能够明确知道它是哪一种时间语义。

---

## 19.4 Freshness Presentation States

本节定义的是 **Freshness Result 的 UI Presentation State**，不是新的业务数据 taxonomy，也不替代 `DATA_MODEL.md`、`METRICS.md` 或同步系统中的正式状态。

允许的呈现状态：

```text
CURRENT
DELAYED
STALE
GAP
FAILED
UNKNOWN
```

### `CURRENT`

满足该 Dataset 已定义的 freshness policy。

默认表现：

```text
Quiet
Neutral
No success banner
```

### `DELAYED`

数据仍可使用，但更新晚于该 Dataset 的正常预期。

必须允许查看：

- Last Success；
- Coverage Through；
- Delay Reason（若已知）。

### `STALE`

当前展示的是已经过期的 Last Known Data。

必须明确：

```text
数据已过期
最后有效：<time>
```

不能把 Last Known Value 静默呈现成 Current Value。

### `GAP`

已经确认目标覆盖范围中存在缺失区间或缺失批次。

必须允许查看：

- Gap Range；
- Affected Dataset；
- Backfill Capability；
- 是否可能永久缺失。

### `FAILED`

最近一次 Refresh / Sync / Import / Retrieval 失败。

如果 Last Known Value 仍可使用：

```text
Value remains visible
+
FAILED context
+
Last Success
```

不得自动清空为 `—`，更不得变成 `0`。

### `UNKNOWN`

当前无法可靠判断 Freshness。

`UNKNOWN` 不能静默映射为 `CURRENT`。

---

## 19.5 Source-specific Freshness Policy

O3Pilot 不定义一个适用于所有数据源的固定 Freshness Threshold。

禁止：

```text
All sources
> 5 minutes
→ STALE
```

正式关系：

```text
Source / Dataset
→ own freshness policy
```

具体阈值、同步周期、预期延迟和最终性由：

- `DATA_SOURCES.md`；
- Scheduler / Sync Contract；
- 对应 Dataset / Metric Contract；

决定。

`DESIGN.md` 只定义这些状态如何被用户正确理解。

因此：

```text
Seller API
Performance API
Webhook
Official Report
Seller-owned Data
Exchange Rate
```

不得伪装成拥有相同实时性、回补能力或最终性。

---

## 19.6 Coverage Gaps and Backfill Risk

`GAP` 是一等数据健康状态。

正式原则：

```text
Coverage Gap
>
simple timestamp age
```

对于回补能力有限或窗口快速失效的数据源，Gap 必须比普通 `DELAYED` 更显著。

例如：

```text
Performance · Campaign SKU Daily

GAP
最后成功：10:28
覆盖至：今日 10:00
缺失：昨日 14:00–18:00
回补能力：受限
```

如果上游 Contract 已知：

```text
Missing historical window
→ cannot reliably backfill
```

UI 必须保留这一风险，不得只显示“当前同步正常”。

禁止：

```text
Latest sync succeeded
→ hide historical gap
```

---

## 19.7 Webhook and State Reconciliation

Webhook Freshness 与业务状态 Freshness 必须分开。

正式关系：

```text
Webhook
→ Change Signal / Event

Seller API Readback
→ Reconciled State
```

因此：

```text
Webhook event is fresh
≠ reconciled business state is fresh
```

允许同时展示：

```text
Webhook
最后事件：10:31

Seller API Readback
最后对账：10:32
```

禁止仅因为最新 Webhook Event 很新，就把 Orders / Returns / Product 等最终状态标记为 `CURRENT`。

Webhook 也不能在 UI 中宣称未经正式 Contract 保证的：

- 固定 1–3 秒送达；
- 绝对秒级实时；
- 永不重复；
- 严格顺序；
- 永不漏事件。

---

## 19.8 Manual Source Freshness

人工导入来源必须区分：

```text
Import Time
Coverage Through
Freshness State
```

示例：

```text
Seller Cost
手动数据

最近导入：Sep 03 14:20
数据覆盖至：Aug 31
```

正式原则：

```text
Manual
≠ Stale
```

人工数据是否过期，仍由对应 Dataset 的业务 Freshness Policy 判断。

例如：

- 月度物流成本可以对一个已闭合月份完全有效；
- 同样年龄的库存快照可能已经不适合当前库存判断。

如果存在 Import Batch，Source Detail 应允许继续定位到对应批次。

---

## 19.9 Derived Metric Freshness

派生指标不能只使用计算时间表示 Freshness。

例如：

```text
Finance
+
Advertising
+
Seller Cost
+
FX
↓
Profit
```

即使：

```text
Profit calculated at 10:32
```

只要必要依赖中的某一项覆盖不足，就不能把最终指标简单显示成：

```text
数据截至 10:32
```

正式原则：

```text
Derived Metric Freshness
→ follows required dependency coverage
not calculation timestamp alone
```

推荐表现：

```text
利润
¥18,482

计算时间：10:32
有效数据覆盖至：Sep 03 23:59
物流成本覆盖率：96.7%
```

具体 Metric Status 仍由 `METRICS.md` 决定；Freshness UI 只负责让依赖时效和覆盖事实可理解。

---

## 19.10 Current vs Historical Data

历史数据“时间在过去”本身不构成 `STALE`。

正式原则：

```text
Historical Age
≠ Staleness
```

### Current-state Data

Freshness 主要回答：

```text
How recent is the current state?
```

例如：

- 当前库存；
- 当前价格；
- 当前 Campaign 状态；
- 当前 Shop Rating。

### Historical / Closed-period Data

Freshness 主要回答：

```text
Is the requested period complete?
Has the expected latest revision been obtained?
```

例如查看：

```text
Aug 01–Aug 31 Finance
```

不能因为数据业务时间停留在 Aug 31 就自动显示 `STALE`。

应判断：

- 月度区间是否完整；
- 是否等待后续结算 / 修订；
- 是否已获得预期最终版本。

---

## 19.11 Global Data Status Indicator

Global Data Status 位于第 6 章定义的 `Global Context Bar`。

正式关系：

```text
Global Context Bar
└── Data Status
    ├── Overall freshness summary
    └── Source detail disclosure
```

不要求每一个分析页面重复创建一套独立 Global Freshness Widget。

### Collapsed

Collapsed 状态只表达整体情况。

正常示例：

```text
数据正常 · 更新于 10:28
```

需要关注时：

```text
2 个数据源需要关注
```

不建议在 Collapsed 状态持续列出：

```text
Seller 10:28 · Performance 10:23 · Webhook 10:31 · FX ...
```

正式原则：

```text
Collapsed
→ overall state

Expanded
→ source detail
```

### Overall Status

Overall Status 不采用简单多数表决。

禁止：

```text
5 normal
1 critical gap
→ Overall Normal
```

同时也禁止：

```text
one irrelevant source failed
→ every page becomes critical
```

正式层级：

```text
Global Data Health
Page-relevant Data Health
Metric-specific Data Health
```

当前页面和当前指标依赖的关键来源必须优先影响用户可见状态。

---

## 19.12 Source Detail Disclosure

展开后的每一个 Source / Dataset 至少支持展示：

```text
Source / Dataset
Status

Last Attempt
Last Success

Coverage Through
Known Gap

Mode
Automatic / Manual

Relevant Shop
```

按需继续显示：

```text
Endpoint
Sync Run
Import Batch
Failure Reason
Backfill Capability
```

示例：

```text
Performance · Campaign SKU Daily

GAP
最后尝试：10:30
最后成功：10:28
覆盖至：今日 10:00
缺失：昨日 14:00–18:00
回补能力：受限
```

Source Detail 应优先显示业务可理解信息，不要求顶层暴露内部技术 ID。

---

## 19.13 Multi-shop Freshness

Freshness 必须遵循 Shop First。

禁止把：

```text
Shop A → CURRENT
Shop B → STALE
```

合并显示成：

```text
Seller API → Normal
```

### 单店 Context

只显示当前 Shop 相关的 Freshness。

### 跨店分析

允许聚合摘要：

```text
5 个店铺
4 正常
1 延迟
```

但必须允许继续定位到具体 Shop。

正式原则：

```text
Cross-shop summary
→ discoverable per-shop state
```

---

## 19.14 Visual Priority

正常状态保持安静。

推荐层级：

```text
CURRENT
→ Neutral / Quiet

DELAYED
→ Subtle Attention

STALE / GAP / FAILED
→ Stronger visibility when relevant

UNKNOWN
→ explicit but not automatically Danger
```

禁止：

- 为 `CURRENT` 使用巨大绿色 Banner；
- 把 `Manual` 自动显示为 Warning；
- 把所有 Source Status 都用彩色 Pill 堆叠；
- 只依赖颜色区分 Freshness；
- 用脉冲、闪烁或持续 Motion 表示同步状态。

状态应通过：

```text
Label
+
Reason
+
Timestamp / Coverage
+
Semantic Color when warranted
```

共同表达。

---

## 19.15 Freshness QA

正式 Freshness QA 至少覆盖：

```text
Current automatic source
Delayed automatic source
Stale Last Known Value
Confirmed coverage gap
Failed latest sync with valid historical value
Unknown freshness
Manual source
Historical closed period
Webhook event + API reconciliation
Derived multi-source metric
Cross-shop mixed freshness
```

必须检查：

1. `Fetch Time` 与 `Coverage Through` 不被混用；
2. `Freshness`、`Coverage`、`Availability`、`Retrieval State` 不被合并成一个状态；
3. 历史数据不会仅因时间较早被标记为 `STALE`；
4. `Manual` 不自动等于 `STALE`；
5. Webhook Event 不被当成最终 Reconciled State；
6. Gap 不会因为最新一次同步成功而被隐藏；
7. Derived Metric Freshness 能反映必要依赖的覆盖事实；
8. Refresh Failure 不会把 Last Known Value 变成 `0`；
9. Global Summary 与 Page / Metric Relevant Status 不互相矛盾；
10. Multi-shop 状态可追溯到具体 Shop；
11. Normal 状态保持 Quiet；
12. Light / Dark / Keyboard / Screen Reader 下状态都可理解。

最终必须满足：

```text
Fetch Time
≠ Coverage Time

Freshness
≠ Coverage
≠ Availability
≠ Retrieval State

Historical Age
≠ Staleness

Webhook Event
≠ Reconciled State

Manual
≠ Stale

Derived Freshness
→ follows required dependencies

Gap
→ first-class state

Normal
→ Quiet

Reason / Last Success / Coverage
→ discoverable
```

---

# 20. Reporting Currency UX

Reporting Currency 是 O3Pilot 的金额分析与跨币种聚合 Context。

它只改变允许转换的 Money Presentation / Aggregation，不改变来源事实、Metric Contract、Shop Context、业务状态或 Ozon Server State。

正式原则：

```text
Reporting Currency
→ Display / Aggregation Context

Reporting Currency
≠ Source Currency
≠ Settlement Currency
≠ Rewrite of Original Money
```

所有正式行为继续遵守 `METRICS.md` 的 Money / FX Contract。

---

## 20.1 Reporting Currency Principles

O3Pilot 必须始终保留原始 Money：

```text
amount
currency
```

Reporting Currency 只能在需要跨币种分析时建立新的展示与聚合 Context。

例如：

```text
Source Fact
100 USD

Reporting Currency
CNY

Displayed Converted Value
¥xxx.xx CNY
```

其中底层来源事实仍然是：

```text
100 USD
```

正式规则：

```text
Converted Value
≠ Original Fact
```

因此：

- 切换 Reporting Currency 不得覆盖 Source Currency；
- Money Detail / Metric Detail 必须能够继续看到原始金额与原始币种；
- Export / Lineage 不得因为当前页面选择了 Reporting Currency，就把原始 Money 伪装成转换后的币种事实；
- Reporting Currency 不得改变指标来源 `metric_origin`；
- Reporting Currency 不得改变任何非金额指标的业务语义。

禁止：

```text
Locale
→ infer Reporting Currency

Shop Settlement Currency
→ infer every Money currency
```

正式原则：

```text
Locale
≠ Reporting Currency

Settlement Currency
≠ Universal Currency
```

只有用户明确选择、或未来由正式产品设置明确保存的 Reporting Currency Preference，才可以成为默认分析 Context。

---

## 20.2 Reporting Currency Context

第 6 章 Global Context Bar 中的：

```text
[Reporting Currency ▼]
```

是 Reporting Currency 的唯一全局 Context Control。

第 6 章中的：

```text
Original currencies / 不转换
```

在正式用户界面中使用更直接的文案：

```text
按原币种
```

语义保持：

```text
按原币种
→ no cross-currency conversion
→ currencies remain separate
```

推荐 Control：

```text
汇总币种
[ 按原币种 ▾ ]
```

可选项至少支持当前业务数据实际需要的：

```text
按原币种
CNY
USD
RUB
...
```

Currency List 不应因为 UI Locale 而固定成一组与实际业务无关的币种。

正式原则：

```text
Reporting Currency options
→ follow supported business currencies / FX capability
```

---

## 20.3 Original Currencies Mode

当：

```text
汇总币种 = 按原币种
```

多币种 Money 必须分别展示。

例如：

```text
销售金额

USD   10,423.18
CNY   42,330.00
RUB  843,219.40
```

允许：

- 按 Currency 分组；
- 在 Table 每行明确 Currency；
- 在同一 Metric Surface 中展示多个独立币种结果。

禁止：

```text
USD 100
+
CNY 500
+
RUB 1,000
=
1,600
```

正式关系：

```text
No Reporting Currency
→ No cross-currency total
```

如果当前 Scope 只有单一币种，可以正常显示单一 Money Value，但仍不得因此推断其他来源 Money 也使用同一 Currency。

---

## 20.4 Selected Reporting Currency

当用户明确选择：

```text
Reporting Currency = CNY
```

且对应 Metric Contract / FX Contract 支持转换时，页面才可以形成跨币种汇总值。

推荐表达：

```text
销售金额
¥57,882.31 CNY

汇总币种：CNY
```

在空间受限时允许：

```text
¥57,882.31
CNY · 已折算
```

但不能只依赖：

```text
¥
$
₽
```

作为完整 Currency Identity。

正式原则：

```text
Currency Symbol
≠ complete currency identity
```

重要跨币种分析场景优先保留 Currency Code。

### Converted ≠ Estimated

正常 FX Conversion 不应使用：

```text
≈ ¥57,882
```

作为默认表达。

`≈` 很容易被理解成 Metric `ESTIMATED`。

正式区分：

```text
Converted
→ Currency Context

ESTIMATED
→ Metric Status / Origin semantics
```

两者不得混用。

---

## 20.5 Scope and Propagation

Reporting Currency 只影响正式支持 Currency Conversion 的 Money Presentation / Aggregation。

切换：

```text
CNY
→ USD
```

不得改变：

```text
Orders
Units
CTR
Return Rate
P50 / P90
Days of Cover
Rating
Status
Coverage
Metric Origin
Time Basis
```

正式原则：

```text
Reporting Currency
→ Money presentation / aggregation

not
→ business metric semantics
```

### Page Relevance

Global Context Bar 可以保留用户当前选择，但页面只有在确实存在 Monetary Context 时才让该选择影响内容。

正式规则：

```text
No monetary context
→ selector visually receded / no page effect

Single-currency context
→ original currency normally sufficient

Multi-currency aggregation context
→ selector becomes materially relevant
```

普通 Feature 不得在页面内部再实现第二套 Reporting Currency Selector。

### Context Preservation

Reporting Currency Change 必须保持其他 Context：

```text
Shop Context
Date Range
Comparison
Filters
Selected Entities
Table position when practical
```

不应因为 Currency 变化而重置页面分析状态。

---

## 20.6 Converted Metric Presentation

Converted Money 继续使用第 18 章 Metric Presentation Contract。

至少保证：

```text
Metric Name
Converted Value
Reporting Currency
Metric Status when relevant
Origin
Coverage / As-of when relevant
```

例如：

```text
利润
¥18,482.31 CNY

O3Pilot 计算
汇总币种：CNY
```

如果转换存在 `PARTIAL`：

```text
利润
¥18,482.31 CNY

部分数据 · FX Coverage 96.7%
```

不能为了得到单一 Currency Value 而弱化第 17、18 章的状态表达。

### Metric Origin

FX Transformation 不改变原始 Metric Identity。

例如：

```text
Ozon Analytics Revenue
metric_origin = OZON_SOURCE
```

即使页面折算为 CNY，也不能因为 FX Conversion 就把该 Metric Origin 改成：

```text
O3Pilot 计算
```

正式原则：

```text
Metric Origin
≠ FX Transformation
```

如果上游 Metric Contract 本身定义了新的派生指标，则按上游 Contract 处理；DESIGN 不重新定义 Origin。

---

## 20.7 Original Currency Breakdown

任何跨币种 Converted Total 都必须保留 Original Currency Breakdown 的可发现路径。

标准追溯：

```text
Converted Total
↓
Original Currency Breakdown
↓
FX Basis / Policy Detail
```

例如：

```text
汇总
¥57,882.31 CNY

原币种
USD   10,423.18
CNY   42,330.00
RUB  843,219.40
```

必要时再继续查看：

```text
Source Amount
Source Currency
Exchange Rate
Rate Basis
Rate Time / Period
Conversion Policy
Policy Version
```

正式原则：

```text
Converted Total
must never destroy original currency visibility
```

Original Breakdown 不要求始终常驻主界面，但必须可通过 Metric Detail、Money Detail、Drawer 或等价 Detail Disclosure 查看。

---

## 20.8 FX Basis and Traceability

每次跨币种转换的 UI 必须能够继续追溯 `METRICS.md` 已定义的 FX 语义：

```text
exchange_rate
exchange_rate_type
rate_basis_type
rate_basis_time
rate_policy_version
```

具体字段命名和计算规则由上游 Contract 决定。

DESIGN 负责确保用户能够理解：

```text
Value
↓
Reporting Currency
↓
Original Money
↓
FX Policy / Basis
↓
Rate Time / Period
```

### Historical Period

对于跨日期区间的 Money，不能为了 UI 简单而伪造一条“代表汇率”。

如果实际计算使用：

```text
multiple dates
multiple rates
multiple rate periods
```

Detail 应表达：

```text
Multiple rates used
```

并允许继续查看对应 Policy / Period Detail。

禁止：

```text
30-day converted result
→ display one arbitrary spot rate as if it explains the total
```

正式原则：

```text
Complex FX calculation
≠ fake single representative rate
```

---

## 20.9 Mixed FX Policies

O3Pilot 必须区分不同 Money Component 的 FX Provenance。

上游 Contract 已定义：

```text
Ozon Business FX
≠ Seller Cost FX
```

因此 Profit 等综合指标可能出现：

```text
Revenue Component
→ Ozon Business FX

Purchase Cost Component
→ Seller Cost FX
```

UI 不得把整个综合指标简单标成：

```text
汇率：Ozon Rate
```

如果其内部实际使用多套 FX Policy。

推荐 Detail：

```text
收入转换
Ozon Business FX

采购成本转换
Seller Cost FX
```

正式原则：

```text
Metric / Component
→ own FX provenance
```

综合指标可以提供统一 Reporting Currency，但不能抹掉不同 Component 的 FX Basis。

---

## 20.10 Partial / Unavailable Conversion

### Missing FX

例如：

```text
USD → CNY   available
RUB → CNY   available
XYZ → CNY   unavailable
```

禁止：

```text
Converted Total
= USD + RUB
```

然后静默忽略 XYZ。

必须按 Metric Contract 进入：

```text
PARTIAL
```

或：

```text
UNAVAILABLE
```

并说明原因。

例如：

```text
¥48,230.12 CNY

部分数据
FX Coverage 92.4%
1 个币种无法折算
```

正式原则：

```text
Missing FX
≠ Zero
≠ silent exclusion
```

### Unknown Currency

如果来源事实为：

```text
amount = 100
currency = UNKNOWN
```

禁止：

```text
assume Shop settlement currency
→ convert
```

正确表达至少为：

```text
100
币种未知
```

并让相关聚合进入上游 Contract 所定义的 Partial / Unavailable 状态。

正式原则：

```text
Unknown Source Currency
→ not convertible
```

### No Silent Fallback

当转换失败时，不得自动回退到：

- Shop Settlement Currency；
- Browser Locale Currency；
- Last Used Rate without policy support；
- 任意固定汇率；
- `1:1`。

---

## 20.11 Comparison Rules

Money Comparison 必须首先满足可比较的 Currency Context。

正式原则：

```text
Comparable Currency Context
→ required before Comparison
```

禁止：

```text
Current
→ CNY converted

Previous
→ USD original

Delta
→ +8.2%
```

当 Reporting Currency 已选择时：

```text
Current Period
→ converted under formal FX policy

Previous Period
→ converted under formal FX policy
```

两者必须满足 `METRICS.md` 的 Comparison / FX Contract 后，才能展示 Delta。

如果无法形成一致可比较结果：

```text
Comparison
→ suppressed / unavailable
```

不得为了保持 KPI 版式完整而强行计算。

### Display Precision

正式区分：

```text
Display precision
≠ calculation precision
```

UI 可以显示：

```text
¥57,882.31
```

但显示时的 Round / Format 不能成为后续聚合或 Comparison 的输入。

FX 精度、计算精度和财务舍入方式仍由上游 Contract 定义。

---

## 20.12 Charts and Tables

Reporting Currency 必须在可比较的视觉中保持明确。

### Chart

Converted Chart 至少在 Title、Axis、Legend 或等价上下文中明确 Currency：

```text
Revenue · CNY
```

或：

```text
Y-axis
CNY
```

禁止：

```text
USD / CNY / RUB raw values
→ one shared numeric axis
→ visually compare heights as if same unit
```

在 `按原币种` 模式下，多币种 Chart 应：

- 按 Currency 分组；
- 使用 Small Multiples；
- 或采用其他不会制造单位混淆的方式。

### Table

Converted Column 应明确：

```text
销售金额 (CNY)
```

或在每个 Money Cell 中明确 Currency。

在 `按原币种` 模式下：

```text
Mixed currencies
→ explicit per-row currency or grouped presentation
```

用户不应必须记住页面顶部当前币种，才能理解 Table / Chart 中的金额单位。

---

## 20.13 Multi-shop Currency UX

`全部店铺` Context 下，Reporting Currency 不能隐藏 Shop 的原始货币环境。

例如：

```text
Shop A → USD
Shop B → CNY
Shop C → RUB
```

当：

```text
Reporting Currency = CNY
```

且 Metric Contract 支持跨店聚合时，可以显示：

```text
All Shops Total
¥xxx.xx CNY
```

但 Detail 必须继续允许查看：

```text
Shop A   USD
Shop B   CNY
Shop C   RUB
```

以及必要的 FX Provenance。

正式原则：

```text
Reporting Currency
≠ loss of Shop currency context
```

如果某 Shop 无法转换：

- 不得按 `0` 参与；
- 不得静默排除；
- 必须进入对应 Partial / Unavailable 状态。

---

## 20.14 Local State and Persistence

Reporting Currency 是 O3Pilot 本地分析 Context。

切换 Reporting Currency：

```text
CNY
→ USD
```

本质是：

```text
Local UI / Analysis Context Change
```

不是：

```text
Ozon Data Mutation
Data Source Sync
Ozon Refresh
```

因此：

```text
Currency switch
≠ Data refresh
≠ Source sync
```

如果需要重新计算 Converted Result，可以使用：

```text
重新计算…
```

或等价局部 Loading 状态。

禁止使用：

```text
正在同步 Ozon…
```

来表达纯本地 Currency Conversion。

### Persistence

如果 v1 保存 Reporting Currency Preference：

- 它属于 O3Pilot Local State；
- 不能写回 Ozon；
- 不能覆盖来源 Money；
- 用户可以随时切回 `按原币种`。

未来即使存在自动建议，也不得在没有明确产品 Contract 的情况下，根据 Locale、Shop Settlement Currency 或当前页面自动静默改变用户已经选择的 Reporting Currency。

---

## 20.15 Reporting Currency QA

正式 QA 至少覆盖：

```text
Single-currency scope
Multi-currency scope
Original currencies / 按原币种
CNY reporting currency
USD reporting currency
RUB reporting currency
Unknown source currency
Missing FX
Partial FX coverage
Unavailable conversion
Historical multi-rate conversion
Mixed Ozon Business FX + Seller Cost FX
Current vs previous period comparison
Multi-shop mixed currencies
Chart
Table
Metric Detail
Money Detail
Currency switching
```

必须同时验证：

1. 未选择 Reporting Currency 时不产生跨币种总数；
2. Converted Value 不覆盖 Original Money；
3. Currency Symbol 不作为唯一 Currency Identity；
4. Converted 不被误标为 `ESTIMATED`；
5. Original Currency Breakdown 可发现；
6. FX Basis / Rate Time / Policy 可追溯；
7. 历史多汇率结果不伪装成单一代表 Rate；
8. Ozon Business FX 与 Seller Cost FX 不混淆；
9. Missing FX 不静默排除；
10. Unknown Currency 不通过 Shop Currency 推断；
11. Display Rounding 不进入计算；
12. Comparison 使用一致可比较的 Currency Context；
13. Table / Chart 明确 Currency；
14. Multi-shop Conversion 不隐藏 Shop 原始币种；
15. Currency Switch 不被表现成 Ozon Sync；
16. Reporting Currency 改变不影响非 Money Metric；
17. Reporting Currency 不改变 Metric Origin；
18. Light / Dark、Desktop / Compact / Narrow 下 Currency Context 均可理解；
19. CN / EN / RU 中 Currency Code、Amount 与状态关系保持可读；
20. Keyboard 与 Screen Reader 可以理解 Selector 当前值和 Converted Money Context。

核心 Contract：

```text
Reporting Currency
≠ Source Currency

Locale
≠ Reporting Currency

Settlement Currency
≠ Universal Currency

Conversion
≠ Estimate

Converted Total
→ Original Breakdown discoverable

FX Detail
→ true policy / basis / time

Missing FX
≠ silent exclusion

Display rounding
≠ calculation rounding

Comparable Currency Context
→ required for Comparison

Metric Origin
≠ FX Transformation
```

---

# 21. Date / Time UX

Date / Time UX 定义 O3Pilot 页面如何表达日期范围、时间口径、展示时区、来源业务日期、Period Comparison 与数据可用范围。

Date Control 只能改变查询 / 分析时间 Context，不得重新定义 `METRICS.md` 已规定的 `time_basis`，也不得为了前端统一而重写来源业务日期语义。

正式原则：

```text
Date Range
→ Page Context

Preset
→ deterministic exact range

Timestamp
≠ Business Date

Display Timezone
≠ Business Date rewrite

Relative Date
→ convenience

Exact Date
→ traceability
```

---

## 21.1 Date / Time Principles

O3Pilot 必须同时区分：

```text
Date Range
Time Basis
Display Timezone
Business Date / Date-only Value
Timestamp
Comparison Window
Source Coverage
```

这些概念不得压缩为一个“日期”字段。

完整的时间分析 Context 至少是：

```text
Selected Range
+
Time Basis
+
Display Timezone when timestamps are involved
+
Comparison Context when enabled
```

正式规则：

```text
One date label
≠ complete time semantics
```

页面可以对低频技术信息采用渐进披露，但用户必须能够发现当前时间口径与最终实际日期范围。

---

## 21.2 Page-level Date Context

Date Range 属于 Page Context，不属于 Global Application Context。

正式关系：

```text
Global Context
→ Shop
→ Reporting Currency

Page Context
→ Date Range
→ Comparison
```

因此 v1 不在 Global Context Bar 增加全局日期选择器。

推荐 Page Header 结构：

```text
销售

[ 7D ▾ ] [ 较上一周期 ▾ ]
2026-08-29 — 2026-09-04
```

不同页面可以拥有不同的有效 Date Range 与 Time Basis。

切换页面时，不得假设上一页面的 Date Range 对新页面一定有效；若产品保留用户最近使用的范围，新页面仍必须通过自己的数据能力 Contract 验证该范围是否可用。

正式原则：

```text
Date Context
→ scoped to page capability
```

---

## 21.3 Date Range Presets

v1 正式高频 Preset：

```text
Today
7D
30D
90D
Custom
```

暂不扩展为大量快捷项，例如：

```text
WTD
MTD
QTD
YTD
Last Month
Last Quarter
```

除非未来具体页面存在明确、高频且已确认的业务需求。

正式原则：

```text
Preset
→ high-frequency analysis window
```

是否允许某个 Preset，必须由当前页面 / Dataset 的实际数据能力决定。

Unsupported Preset：

```text
→ disabled / unavailable
→ reason discoverable
```

不得允许用户选择一个已知不支持的范围后，再把空响应表现成业务上“没有数据”。

例如：

```text
90D
不可用

该数据源当前仅支持今天和昨天
```

---

## 21.4 Exact Range Semantics

### 21.4.1 Deterministic Presets

`7D / 30D / 90D` 必须拥有固定、可测试的语义。

正式定义：

```text
7D
→ 7 calendar dates ending on the active end date

30D
→ 30 calendar dates ending on the active end date

90D
→ 90 calendar dates ending on the active end date
```

例如：

```text
7D

2026-08-29
—
2026-09-04
```

包含 7 个 Calendar Dates。

正式原则：

```text
Preset label
→ deterministic resolved range
```

展开 Date Control 或进入 Detail 时，必须能够看到最终解析后的：

```text
start_date
end_date
```

相同 Preset 不得在不同 Feature 中被实现为不同长度。

### 21.4.2 Today

`Today` 必须跟随当前页面实际 Date Semantics。

对于普通 Timestamp-based 页面：

```text
Today
→ current date in Display Timezone
```

v1 默认 Display Timezone：

```text
Asia/Shanghai
UTC+8
```

对于正式 Date-only / Business-Date 语义，例如：

```text
BUSINESS_DATE
CAMPAIGN_DATE
ACCRUAL_DATE
```

`Today` 必须遵循该来源 / Metric Contract 的 Business-Date Semantics。

禁止把 Date-only Value 先转换为 Midnight Timestamp，再通过 Display Timezone 重新划日。

正式关系：

```text
Timestamp Today
≠ automatically Business-Date Today
```

因此：

```text
Today
→ follows active date semantics
```

---

## 21.5 Time Basis

任何 Date Range 都必须能够追溯当前 `time_basis`。

`METRICS.md` 已定义正式 Time Basis，例如：

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

DESIGN 不重新定义这些业务含义，只定义其展示方式。

用户可见示例：

```text
按下单时间
按签收时间
按应计日期
按 Delivery Cohort
```

正式原则：

```text
Date Range
+
Time Basis
→ complete time context
```

默认可通过以下一种或多种方式发现：

- Page Header 次级 Metadata；
- Date Control Detail；
- Metric Info；
- Tooltip / Popover；
- Detail Drawer。

不要求内部 Code 长期常驻主界面，但语义必须可发现。

### 21.5.1 Multi-basis Page

一个页面可以包含多个 Time Basis。

例如 Dashboard：

```text
Sales
→ ORDER_CREATED

Returns
→ DELIVERY_COHORT

Finance
→ ACCRUAL_DATE
```

因此：

```text
One page
≠ necessarily one Time Basis
```

页面可以存在 Primary Time Context，但任何使用不同 Time Basis 的 Metric 必须在其附近或 Metric Detail 中明确说明。

不得仅显示“过去 30 天”就让用户误以为页面上所有指标完全使用同一时间归属。

---

## 21.6 Display Timezone

v1 默认业务展示时区：

```text
Asia/Shanghai
UTC+8
```

主要适用于 Timestamp，例如：

- Order Timestamp；
- Webhook Timestamp；
- Sync Time；
- Job Time；
- Import Time；
- As-of Time；
- Created / Updated Time。

正式关系：

```text
Timestamp
→ Display Timezone conversion allowed
```

但：

```text
Display Timezone
≠ Source Business Date Rewrite
```

v1 不要求在每个页面提供独立 Timezone Selector。

未来如允许用户修改 Display Timezone，必须作为显式设置进入正式 Product / Settings Contract，而不是由页面临时决定。

### 21.6.1 Timezone Discoverability

主界面可简洁显示：

```text
2026-09-04 11:35
```

Detail 必须能够发现：

```text
Asia/Shanghai
UTC+08:00
```

如果来源 Timestamp 包含原始 Offset，需要时还应能够追溯：

```text
Original Timestamp
Source Offset
```

正式原则：

```text
Displayed Time
→ timezone discoverable
```

无需在每一个 Table Cell 后重复 `UTC+8`。

---

## 21.7 Timestamp vs Business Date

Date-only Value 与 Timestamp 必须彻底区分。

Timestamp 示例：

```text
created_at
2026-09-04T02:00:00Z
```

在 v1 Display Timezone 下可以展示为：

```text
2026-09-04 10:00
```

而来源 Date-only 示例：

```text
Performance Date
2026-09-04
```

必须继续保持：

```text
2026-09-04
```

不能因为时区换算变成前一天或后一天。

正式原则：

```text
Date-only value
≠ midnight timestamp
```

禁止：

```text
Ozon Analytics Day
Ozon Finance Date
Ozon Exchange Rate Interval Date
Performance Date
```

为了前端“统一时间处理”而先转成 Timestamp 再重新划日。

---

## 21.8 Period Comparison

v1 Comparison Mode：

```text
Off
Previous period
Previous year
```

`Previous year` 只有在存在可比较历史、且 Metric Contract 支持时可用。

### 21.8.1 Previous Period

正式定义：

```text
Previous Period
→ immediately preceding
→ equal-length
→ non-overlapping
```

例如：

```text
Current
2026-08-29 — 2026-09-04
7 dates

Previous Period
2026-08-22 — 2026-08-28
7 dates
```

### 21.8.2 Previous Year

Previous Year 必须保持：

```text
same intended calendar period
+
same Metric Contract
+
same Time Basis
```

Leap Year、来源边界和复杂 Calendar Alignment 由正式 Comparison Logic / Metric Contract 处理，DESIGN 不重新定义算法。

UI 必须显示最终解析后的实际比较日期，例如：

```text
较去年同期
2025-08-29 — 2025-09-04
```

正式原则：

```text
Comparison label
≠ hidden comparison range
```

### 21.8.3 Comparison Compatibility

正式 Comparison 至少要求：

```text
same metric
same time basis
comparable window
compatible currency context when Money
```

因此：

```text
Comparable Time Basis
→ required before Comparison
```

不可比较时：

```text
Comparison
→ suppressed / unavailable
```

不得为了保持 KPI 布局完整而展示误导性的 Delta。

---

## 21.9 Open / Incomplete Periods

O3Pilot 必须区分：

```text
Open Period
Closed Period
```

例如用户在当前日尚未结束时查看：

```text
Today
```

页面应表达：

```text
Today · 截至 11:35
```

或：

```text
当前周期尚未结束
```

禁止把未闭合 Current Period 与完整 Historical Period 静默比较。

例如禁止：

```text
Today until 11:35
vs
Yesterday full day
→ -52%
```

却不给任何说明。

正式原则：

```text
Incomplete Current Period
≠ Full Historical Period
```

Comparison 必须按 Metric Contract 使用：

```text
matched-as-of comparison
```

或：

```text
comparison unavailable
```

不能由 UI 自己发明临时算法。

对于 Cohort 类指标，如果 Cohort 尚未闭合，应继续使用上游正式状态，例如：

```text
OPEN_COHORT
```

而不是将其当成普通 Date Range 问题隐藏。

---

## 21.10 Source Range and Coverage

Date Picker 必须尊重数据源真实可查询范围与已持久化覆盖范围。

### 21.10.1 Unsupported Range

已知不支持的范围：

```text
→ disabled / unavailable
→ reason discoverable
```

不得：

```text
Selectable
→ empty response
→ presented as business no-data
```

### 21.10.2 Requested Range vs Available Coverage

禁止静默缩短用户请求范围。

例如：

```text
Selected Range
2026-08-01 — 2026-08-31

Available Coverage
2026-08-15 — 2026-08-31
```

UI 必须保留 Selected Range，并根据 Metric Contract 显示：

```text
PARTIAL
```

或其他正式不可用状态。

正式原则：

```text
Requested Range
≠ silently shortened range
```

Coverage 可以通过：

- Metric Metadata；
- Data Freshness Detail；
- Page-level Data Status；

共同表达。

### 21.10.3 Multi-source Page

当页面 Range 为：

```text
30D
```

而不同来源支持范围不同：

```text
Seller Data
→ 30D available

Performance Dataset
→ only partial coverage
```

Page Date Context 仍保持：

```text
30D
```

受限 Metric 显式显示自己的 Coverage / Status。

禁止：

```text
one widget
→ silently switches to 2D
```

却继续处在 30D 页面 Context 中。

正式原则：

```text
Page Date Context
→ stable

Metric limitations
→ explicit
```

---

## 21.11 Multi-source / Multi-shop Time Context

### 21.11.1 Multi-source

多个来源不能通过静默改变 Range 来制造“完整一致”的假象。

如果某来源缺失：

```text
Selected Range remains stable
+
Affected Metric gets explicit coverage / status
```

### 21.11.2 Multi-shop

跨店铺分析同样必须保持 Shop First + Time Coverage。

例如：

```text
Shop A coverage
2026-08-01 — 2026-09-04

Shop B coverage
2026-08-15 — 2026-09-04
```

用户选择：

```text
2026-08-01 — 2026-09-04
```

不得：

- 把 Shop B 缺失部分当 `0`；
- 静默缩短 All Shops Range；
- 静默从聚合中删除 Shop B。

应显示类似：

```text
PARTIAL
4 / 5 shops complete
```

或对应正式 Coverage 结果。

正式原则：

```text
Shop First
+
Time Coverage
→ both preserved
```

---

## 21.12 Date and Time Formatting

### 21.12.1 Exact Date

正式分析界面优先使用无歧义格式：

```text
YYYY-MM-DD
```

例如：

```text
2026-09-04
```

日期范围：

```text
2026-08-29 — 2026-09-04
```

禁止把以下纯数字格式作为统一正式格式：

```text
09/04/2026
```

因为 CN / EN / RU 用户可能产生 Month / Day 顺序歧义。

用户友好型长日期可以本地化，例如：

```text
2026年9月4日
Sep 4, 2026
4 сент. 2026 г.
```

但不得引入数字顺序歧义。

### 21.12.2 Relative Date

以下可以用于高频交互：

```text
Today
Yesterday
7D
30D
```

但：

```text
Relative Date
→ convenience

Exact Date
→ traceability
```

Metric Detail、Tooltip、Export、Screenshot / Audit 等需要长期可解释的场景必须能够看到 Exact Date。

### 21.12.3 Time

v1 正式采用 24 小时制：

```text
HH:mm
```

例如：

```text
11:35
23:59
```

普通分析场景默认不展示秒。

秒可以用于：

- Webhook；
- Job；
- Event；
- Technical Detail；
- Debug / Diagnostic。

毫秒只进入确实需要的技术诊断。

正式原则：

```text
Time precision
→ follows task
```

不能为了“更精确”在普通数据表中机械显示毫秒。

---

## 21.13 Forecast / Future Dates

Future Date 不得被全局 DatePicker 一刀切禁止。

正式规则：

```text
Historical Analytics
→ past / current supported dates

Forecast
→ future dates when product / metric contract supports them
```

因此：

```text
Date availability
→ follows product capability
```

而不是所有页面共享：

```text
future disabled = true
```

Forecast 的 Actual / Forecast Boundary 必须可辨识，未来日期不能被视觉上伪装成已经发生的事实。

---

## 21.14 Responsive and Accessibility

### Desktop / Compact

优先结构：

```text
Preset
Comparison
Exact Range
Time Basis / Detail
```

### Narrow

Date Controls 可以压缩为单一入口，例如：

```text
[ 7D · 比较开启 ▾ ]
```

打开后展示完整：

```text
Preset
Exact Range
Comparison Mode
Comparison Exact Range
Time Basis
Coverage limitation when relevant
```

不得因为窄屏隐藏当前实际 Range 或 Comparison Basis。

### Accessibility

必须满足：

1. Date Picker 可完整键盘操作；
2. 选中日期不能只靠颜色表达；
3. 当前日期、选中范围起止点、Disabled Date 需要可访问语义；
4. Calendar Grid 使用正确的 Grid / Button Semantics；
5. Date Range 的 Screen Reader 文案必须说明 Start / End；
6. Comparison Mode 的当前状态必须可访问；
7. Time Basis Info Control 必须拥有 Accessible Name；
8. Disabled Range / Preset 必须能发现原因，而不是只表现为灰色不可点。

---

## 21.15 Date / Time QA

正式 QA 至少覆盖：

```text
Today
7D
30D
90D
Custom
Previous Period
Previous Year
Open Current Day
Closed Historical Period
Timestamp
Business Date
Date-only Source Value
Multi-basis Dashboard
Unsupported Source Range
Partial Coverage
Multi-source Coverage Mismatch
Multi-shop Coverage Mismatch
Forecast Future Range
```

必须同时验证：

```text
Asia/Shanghai display timezone
CN
EN
RU
Desktop
Compact
Narrow 390px
Keyboard-only
Screen Reader semantics
Light
Dark
```

验收检查：

1. Preset 必须解析为确定的 Exact Range；
2. `Today` 必须遵循当前页面 Date Semantics；
3. Timestamp 与 Business Date 不得互相转换语义；
4. Display Timezone 不得重写来源 Business Date；
5. Time Basis 必须可发现；
6. Comparison 必须显示可追溯的实际窗口；
7. Previous Period 必须等长、紧邻且不重叠；
8. Money Comparison 同时满足第 20 章 Currency Context；
9. Open Period 不得静默与完整 Closed Period 比较；
10. Requested Range 不得被静默裁剪；
11. Coverage Gap 不得被显示为 `0` 或普通 No Data；
12. Relative Date 必须能够追溯到 Exact Date；
13. 数字日期格式不得产生 CN / EN / RU 顺序歧义；
14. 普通时间采用 24 小时制；
15. Narrow 下 Date Context 仍必须完整可理解；
16. Date Control 的键盘与辅助技术路径必须完整。

正式冻结：

```text
Date Range
→ Page Context

Preset
→ deterministic exact range

Today
→ follows active date semantics

Timestamp
≠ Business Date

Display Timezone
→ Asia/Shanghai

Display Timezone
≠ Business Date rewrite

Comparison
→ equal and compatible time context

Incomplete Period
≠ Closed Period

Requested Range
≠ silently shortened range

Relative Date
→ convenience

Exact Date
→ traceability
```

---

# 22. Charts

O3Pilot 的 Chart 是分析证据，不是页面装饰。

正式关系：

```text
Chart
→ Analytical Evidence

Chart
≠ Decoration
≠ Dashboard Filler
```

任何 Chart 都必须先回答一个明确的分析问题，再决定是否需要图表以及使用哪一种图型。

---

## 22.1 Chart Principles

正式原则：

```text
Question
→ Chart Type

Component Slot
↛ Chart
```

典型问题包括：

```text
趋势如何变化？
哪些对象差异最大？
构成如何变化？
金额如何从 Gross 走到 Net？
漏斗在哪一阶段损失？
什么时间 × 维度出现异常？
```

如果 Metric 或 Table 能更准确、更高效地回答问题，则优先使用 Metric 或 Table，不为了“视觉丰富”增加 Chart。

Chart 必须继承并遵守此前已经正式定义的：

```text
Metric Contract
Data State Language
Metric Presentation
Data Freshness
Reporting Currency UX
Date / Time UX
Motion System
Reduced Motion
120Hz Performance Contract
```

因此：

```text
Missing
≠ Zero

Forecast
≠ Actual

Direction
≠ Outcome

Origin
≠ Status

Historical age
≠ Staleness
```

这些语义不能因为进入 Chart 而被弱化。

---

## 22.2 Chart Selection

正式优先关系：

| Analytical Question | Preferred Presentation |
|---|---|
| 单一关键值 | Metric |
| 时间趋势 | Line |
| 分类精确比较 | Bar |
| 绝对构成 | Stacked Bar |
| 比例构成 | 100% Stacked Bar |
| Signed additive bridge | Waterfall |
| 同 Cohort 的有序转化阶段 | Funnel |
| 时间 × 维度强弱分布 | Heatmap |
| 高精度、多字段比较 | Table |

避免默认使用：

```text
3D
Radar
Gauge
Decorative donut
无意义 Pie
大面积渐变 Area
Dual Y-axis
无业务含义的动效
```

Pie / Donut 不是 v1 默认图型。

只有同时满足：

```text
true part-to-whole
small category count
whole is meaningful
precise ranking is not primary task
```

才可以考虑使用。

否则优先：

```text
Bar
Table
```

正式原则：

```text
Chart choice
→ follows analytical encoding
not visual novelty
```

---

## 22.3 Line Charts

Line Chart 主要用于连续时间趋势。

默认规则：

```text
X-axis
→ ordered time

Y-axis
→ one comparable unit / metric context
```

Line 默认使用直线段：

```text
straight segment
```

不默认使用视觉平滑曲线。

原因：

```text
Interpolation
must not invent business values
```

禁止使用会产生 Overshoot 的曲线，使视觉峰值或谷值超过真实数据。

例如：

```text
Actual max = 100
Rendered curve visually peaks at 108
→ prohibited
```

Line Chart 可以不从 `0` 开始，因为其主要编码是位置与变化趋势，但实际 Axis Domain 必须可理解，不得通过极端截断制造误导。

### 多 Series

Primary Visible Line Series 正常情况下：

```text
≤ 5
```

超过时优先：

```text
Explicit Series Selection
Top-N
Small Multiples
Table
```

如果使用 Top-N，必须显式说明，例如：

```text
Top 5 by Revenue
```

禁止静默隐藏其余 Series。

正式原则：

```text
Hidden Series
→ Never Silent
```

---

## 22.4 Bar / Stacked Bar

Bar 主要用于分类比较。

正式基线：

```text
Bar Axis
→ Starts at Zero
```

Bar Length 直接编码 Magnitude，因此禁止通过截断 Axis 夸大差异。

### Stacked Bar

只在 Component 能形成有意义的可加总整体时使用：

```text
Stacked
→ Components of a Meaningful Total
```

禁止将仅仅“相关”的不同指标堆叠，例如：

```text
Orders
CTR
Revenue
Return Rate
```

### 100% Stacked Bar

用于表达比例构成：

```text
100% Stacked Bar
→ Part-to-whole Ratio Composition
```

不得与绝对金额 / 数量的 Stacked Bar 混为同一种语义。

`Others` 只有在剩余类别可以被合法聚合时才能存在。

如果使用：

```text
Others
```

必须能够继续查看其构成，不得成为无法解释的黑盒聚合。

---

## 22.5 Waterfall

Waterfall 只用于：

```text
Signed Additive Components
→ Reconcile Start / End Total
```

例如：

```text
Gross Sales
- Commission
- Logistics
- Advertising
- Returns
+ Adjustments
= Net / Profit
```

正式规则：

```text
Waterfall
→ Must Reconcile
```

禁止在同一 Waterfall 中混入：

- Ratio；
- 不同 Currency 且未经正式转换的 Money；
- 不同不可比 Period；
- 不可相加 Metric；
- 重复费用；
- 不同 Metric Contract 下的金额。

如果结果为：

```text
PARTIAL
```

必须在图表附近展示：

```text
Coverage
Missing Component / Reason
```

不能让 Waterfall 在视觉上表现成完整闭合但实际缺少部分成本。

---

## 22.6 Funnel

Funnel 只在以下条件同时成立时使用：

```text
Ordered Stages
+
Same Eligible Population / Cohort Logic
+
Meaningful Stage-to-stage Progression
```

例如某个正式 Metric Contract 支持：

```text
Views
→ Add to Cart
→ Orders
```

才可以使用 Funnel。

禁止将：

```text
Revenue
Orders
Return Rate
Advertising Spend
```

这种不存在连续人口关系的指标排成 Funnel。

Funnel 默认应支持：

```text
Absolute Count
Stage Conversion
Overall Conversion
```

具体资格、分母、分子与 Conversion Formula 由 `METRICS.md` 定义，`DESIGN.md` 不重新定义。

正式原则：

```text
Funnel Shape
≠ Proof of Funnel Semantics
```

---

## 22.7 Heatmap

Heatmap 用于：

```text
Dimension A
×
Dimension B
→ Intensity / Magnitude Pattern
```

典型场景：

```text
Hour × Weekday
Warehouse × Date
SKU × Period
```

Heatmap 必须具有：

- 明确 Legend；
- 可解释的 Scale；
- 非颜色唯一的信息访问方式；
- Tooltip / Detail 可读取精确值。

禁止：

```text
Color intensity
→ undefined metric meaning
```

如果 Scale 使用分位、阈值、对数等非线性方式，必须能够发现该规则。

缺失 Cell：

```text
Missing
```

必须与真实：

```text
0
```

具有不同视觉语义。

---

## 22.8 Axis / Scale / Unit Rules

Chart 必须明确当前：

```text
Metric
Unit
Currency when Money
```

例如：

```text
销售金额 · CNY
```

或者：

```text
Y-axis
CNY
```

### Bar

```text
Bar Length
→ Zero Baseline
```

### Line

```text
Line Trend
→ Contextual Domain Allowed
```

但不能使用误导性的极端 Domain。

### Dual Y-axis

v1 正式规则：

```text
Dual Y-axis
→ Prohibited by Default
```

例如：

```text
Revenue
→ Left Axis

CTR
→ Right Axis
```

这种设计容易通过 Scale 人为制造视觉相关性。

不同 Unit 需要同时观察时优先：

```text
Aligned Separate Charts
Small Multiples
Metric + Chart
```

而不是第二根 Y Axis。

### Currency

未选择 Reporting Currency 时：

```text
USD
CNY
RUB
```

不得放进一个统一数值 Axis 直接比较高度。

优先：

```text
Separate Currency Groups
Small Multiples
Table
```

或先显式选择 Reporting Currency。

正式原则：

```text
Value without Unit / Currency Context
→ Prohibited
```

---

## 22.9 Series and Color

Chart Series 使用独立 Chart Palette：

```text
color.chart.*
```

不得直接复用：

```text
color.status.*
```

正式关系：

```text
Series Identity
→ Chart Palette

Business Warning / Danger / Success
→ Semantic Status Palette when warranted
```

普通 Series 不能因为顺序、排名或视觉需要就自动使用 Danger Red / Success Green。

同一 Chart 内：

```text
Series Identity
→ Stable
```

Filter、Hover、Selection 过程中不得随机改变 Series Color Identity。

同时：

```text
Ozon Official
≠ Automatically Blue Series

O3Pilot Forecast
≠ Automatically Yellow Series

Metric Origin
≠ Chart Color
```

### 非颜色唯一编码

需要区分关键状态时必须同时使用：

```text
Label
Line Style
Marker / Pattern where appropriate
Legend Text
```

不能只依赖颜色。

---

## 22.10 Missing / Partial / Stale Data

Chart 必须严格继承第 17、19 章的数据状态语义。

### Known Zero

```text
0
→ Valid Plotted Value
```

### Missing / NULL

```text
Missing
→ No Value
```

对于时间序列：

```text
10
12
missing
15
```

禁止自动连成：

```text
10 → 12 → 15
```

默认：

```text
Missing Interval
→ Visible Break
→ Optional Gap Annotation
```

正式规则：

```text
Missing
≠ Zero
≠ Interpolation
```

### Gap

已确认存在 Coverage Gap 时：

```text
Gap
→ Explicitly Visible
```

且必须能够查看 Gap 原因 / 区间。

### PARTIAL

如果 Chart 数据只覆盖部分请求范围：

```text
PARTIAL
→ Coverage discoverable near Chart
```

禁止用未覆盖日期填 `0`。

### STALE

如果存在 Last Known Series：

```text
Chart
→ Keep Last Known Data

Status
→ STALE / Update Failed
→ Last Valid Time
```

禁止因为最近一次 Retrieval Failure 就清空 Chart。

正式原则：

```text
Retrieval Failure
≠ Erase Last Known Chart
```

### Empty Result

只有真正没有符合当前查询条件的记录时进入 Empty State。

如果所有值都是真实：

```text
0, 0, 0, 0
```

必须绘制零值数据，而不是显示“暂无数据”。

---

## 22.11 Comparison / Forecast

### Comparison

Comparison Chart 必须继承第 18、20、21 章的可比性要求。

至少满足：

```text
Same Metric
Same Time Basis
Comparable Range
Compatible Currency Context when Money
```

Tooltip / Detail 必须可以查看实际比较日期，例如：

```text
Current
2026-09-04

Previous
2026-08-28
```

不能只依赖：

```text
Blue Line
Gray Line
```

表达 Current / Previous。

如果 Current Period 尚未闭合，继续遵守：

```text
Incomplete Period
≠ Closed Period
```

的 Comparison Contract。

### Forecast

Actual 与 Forecast 必须明确区分。

推荐语义：

```text
Actual
────────

Forecast
- - - - -
```

但 Dash 不能成为唯一的区分方式，仍必须有 Label / Legend。

Forecast Boundary 必须可见：

```text
Actual
│
Forecast Boundary
│
Forecast
```

如果模型没有正式 Confidence Interval：

```text
No Formal Interval
→ No Confidence Band
```

禁止为了“预测感”自行创建半透明置信区间。

同样：

```text
ESTIMATED
≠ Warning

Forecast
≠ Actual
```

---

## 22.12 Tooltip and Interaction

Tooltip 是精确 Detail 的补充，不是唯一信息载体。

正式：

```text
Tooltip
→ Supplemental Exact Detail
```

关键语义不能只有 Hover 后才知道，例如：

- Unit；
- Currency；
- Series Identity；
- Forecast / Actual；
- Comparison Basis；
- Critical Status。

Tooltip 默认至少包含：

```text
Exact Date / Category
Series
Value
Unit / Currency
```

必要时再增加：

```text
Status
Coverage
Comparison Value
```

不把完整 Metric Contract 全部塞入 Tooltip。

Tooltip 必须即时响应，不因 Entrance Motion 延迟显示。

### Legend Interaction

如果 Legend 支持隐藏 / 显示 Series，则 Legend Item 必须是真正的交互控件，并明确表达：

```text
Visible
Hidden
```

状态。

关闭某个 Series：

```text
→ Local Analysis State
```

不得改变底层数据或 Ozon 状态。

---

## 22.13 Motion

Chart Motion 必须服从第 14、16 章。

正式原则：

```text
Business Data
→ Stable
```

默认：

- 首次加载不做逐点生长；
- Bar 不从 `0` 生长；
- Line 不做路径绘制动画；
- 数字不 Count-up；
- Funnel 不弹跳重排；
- 数据点不“飞”向新位置；
- Tooltip 即时响应。

Filter / State 切换如确实需要轻量连续性，可使用：

```text
Optional Chart Crossfade
→ motion-standard
→ 180ms
→ opacity only
```

但：

```text
Data Update
→ Direct Redraw by Default
```

Reduced Motion：

```text
→ Immediate
→ No Chart Transition
```

禁止重新创造 `120–180ms` 等自由范围。

---

## 22.14 Performance

Chart 必须继承第 15 章 Performance Contract。

正式原则：

```text
Data Fidelity
≠ Render Every Raw Point Simultaneously
```

大量数据允许根据技术方案使用：

```text
Aggregation
Downsampling
Viewport-aware Rendering
Appropriate Renderer
```

但这些技术处理不得改变 Metric Contract。

禁止为了“展示全部原始数据”：

```text
10,000 points
→ 10,000 SVG nodes
→ Animated Transition
```

造成可见卡顿。

任何 Aggregation / Sampling 必须：

- 保持业务语义；
- 不改变总量 / 比率 Contract；
- 不静默隐藏影响决策的重要峰值、Gap 或异常；
- 需要时可继续访问更高精度数据或 Table。

正式关系：

```text
Performance Optimization
≠ Metric Re-definition
```

---

## 22.15 Responsive

Responsive Chart 不等于把 Desktop Chart 等比缩小。

正式：

```text
Responsive Chart
→ Preserve Analytical Question
not Desktop Geometry
```

Narrow 可以：

- 减少 Tick Density；
- 调整 Legend 位置；
- 使用 Series Selector；
- 将 Small Multiples 纵向排列；
- 将适合的 Vertical Bar 切换为 Horizontal Bar；
- 提供更直接的“查看数据”。

禁止：

- 静默删除 Series；
- 静默缩短 Date Range；
- 隐藏 Unit / Currency；
- 隐藏 Status；
- 改变 Metric；
- 因屏幕变窄而改变实际业务聚合口径。

如果 Chart 已无法在 Narrow 保持可读性，宁可：

```text
Simpler Chart
+
Accessible Table / Detail
```

也不维持复杂 Desktop Geometry。

---

## 22.16 Accessibility

O3Pilot Chart 使用双通道原则：

```text
Visual Chart
+
Accessible Data Representation
```

至少支持：

- 明确 Chart Title；
- 简短 Analytical Description；
- Legend；
- 非仅颜色编码；
- Tooltip / Detail；
- 可以访问精确底层数据或“查看数据”；
- 高价值交互控件支持 Keyboard；
- Focus 可见；
- CN / EN / RU 文案可读。

对于大量数据：

```text
Accessible
≠ Every Mark Is a Tab Stop
```

例如：

```text
10,000 points
```

不应制造 10,000 个 Tab Stop。

更合理的结构：

```text
Summary
+
Series Controls
+
Accessible Data Table
```

如果 Legend Item 可控制 Series，则必须：

- 可键盘操作；
- 拥有 Accessible Name；
- 表达 selected / hidden state；
- 不依赖颜色表达状态。

Chart 的 Bottom-line Insight 不能只存在于视觉形状中；关键业务结论应在需要时能以文字或数据形式被获取。

---

## 22.17 Chart QA

正式 Chart QA 至少覆盖：

```text
Single Metric
Line Trend
Bar Comparison
Stacked Bar
100% Stacked Bar
Waterfall
Funnel
Heatmap
Comparison Chart
Forecast Chart
```

数据状态必须覆盖：

```text
Known Zero
Missing
Gap
PARTIAL
STALE
Empty Result
Retrieval Error with Last Known Data
```

Context 必须覆盖：

```text
Single Currency
Original Multi-currency
Reporting Currency Selected
Current Period
Previous Period
Open Period
Historical Closed Period
Different Time Basis
```

交互与可访问性必须覆盖：

```text
Mouse
Keyboard
Hover-capable Pointer
Touch / Narrow
Reduced Motion
120Hz
60Hz
Light
Dark
CN
EN
RU
```

性能至少覆盖：

```text
Large Time Series
Many Categories
Rapid Filter Switching
Series Toggle
Resize
Narrow Layout
```

必须验证：

1. Chart 能明确回答其设计问题；
2. Bar 使用 Zero Baseline；
3. Dual Y-axis 未被默认引入；
4. Stacked 数据可合法加总；
5. Waterfall 可 Reconcile；
6. Funnel 使用正式 Cohort / Stage Contract；
7. Missing 没有被显示为 `0` 或自动插值；
8. Gap 可被识别；
9. PARTIAL / STALE 状态没有被 Chart 隐藏；
10. Forecast 与 Actual 不会混淆；
11. 没有正式 Confidence Interval 时没有伪造 Band；
12. Top-N / Hidden Series 不会静默发生；
13. Currency / Unit 始终明确；
14. Comparison 使用相同 Time Basis 与可比窗口；
15. Tooltip 不是唯一的关键语义来源；
16. Reduced Motion 下不存在 Chart Motion；
17. Chart 不依赖 Entrance Animation 才能表达数据；
18. 大数据情况下仍保持可交互；
19. Narrow 不静默改变数据口径；
20. 可访问数据表示完整存在。

核心规则：

```text
Chart
→ Analytical Evidence

Missing
≠ Zero
≠ Interpolation

Bar
→ Zero Baseline

Dual Y-axis
→ Prohibited by Default

Stacked
→ Meaningful Additive Composition Only

Funnel
→ Same Ordered Population Logic

Waterfall
→ Must Reconcile

Forecast
≠ Actual

No Formal Confidence Interval
→ No Confidence Band

Hidden / Top-N Data
→ Never Silent

Data Fidelity
≠ Render Every Raw Point

Accessible
≠ Every Mark Is a Tab Stop
```

---

# 23. Tables

O3Pilot 大量核心分析最终以 Table 表达。Table 的首要任务是支持精确比较、扫描、查找和诊断，而不是模拟 Spreadsheet 或把每一行包装成 Card。

正式原则：

```text
Table
→ precise comparison
→ scanning
→ lookup
→ operational diagnosis

Table
≠ spreadsheet by default
≠ collection of cards
```

当任务需要精确比值、多字段比较、订单 / SKU 查询、风险扫描或异常定位时，应优先使用 Table，而不是为了视觉变化强行改成 Chart 或 Card。

---

## 23.1 Table Principles

O3Pilot Table 必须遵守：

```text
Precision before decoration
Semantic density before visual novelty
Stable reading before live movement
Query truth before convenient UI fiction
```

规则：

1. Table 是正式数据表达组件，不是装饰性列表；
2. Table Density 不得通过丢失状态、单位、币种、时间或标识符语义实现；
3. 排序、筛选、搜索、分页必须共同描述同一个 Query State；
4. Table 不得暗示不存在的 Total、Page Count 或完整 Coverage；
5. Business Fact Table 默认 Read-only；
6. Table 中任何操作不得修改 Ozon Server State；
7. Responsive 只调整显示优先级，不改变数据模型或业务语义。

---

## 23.2 Density and Geometry

正式尺寸：

```text
Header height       40px
Row Standard        44px
Row Compact         36px
```

与 Section 12 Spacing Contract 对齐：

```text
Standard horizontal cell padding   16px
Compact horizontal cell padding    12px
```

正式 Density：

```text
Standard
→ normal analytical tables

Compact
→ high-density scanning / comparison
```

规则：

1. Feature 不得自行创造 38px、40px、42px 等第三套 Row Density；
2. `Compact ≠ tiny typography`；
3. 字号、行高和字重继续遵循 Typography Contract；
4. Density 不得降低 Focus、Readable State 或 Accessible Target 的可用性；
5. Header 默认单行；
6. Header Label 优先通过更准确的短名称、列宽和 Tooltip / Info 解决长度问题；
7. 只有业务术语确实无法合理缩短时，才使用明确的 Multi-line Header Pattern。

正式原则：

```text
Column Label
→ concise but semantically complete
```

不得为了单行显示，把正式业务名称缩写成不可理解文本。

---

## 23.3 Column Types and Alignment

正式 Alignment：

```text
Text / Name
→ left

Identifier
→ left

Numeric Measure
→ right

Money
→ right

Ratio / Percentage
→ right

Date / Time
→ consistent alignment

Status
→ left / compact semantic treatment

Actions
→ right
```

最重要的区分：

```text
Identifier
≠ Numeric Measure
```

以下字段即使只包含数字，也属于字符串标识符：

```text
order_number
posting_number
sku
offer_id
product_id
campaign_id
```

因此：

- 左对齐；
- 保留前导零；
- 不使用千位分隔；
- 不使用科学计数法；
- 不参与数值型格式化。

---

## 23.4 Numeric / Money / Identifier Cells

可比较 Numeric 必须：

```text
right aligned
+
tabular-nums
```

金额继续遵循 Reporting Currency Contract：

```text
Amount
+
Currency Context
```

禁止只显示：

```text
18,482
```

而让用户无法判断 Currency。

比例、百分比和 Percentage Point 必须遵循 Metric Contract：

```text
%
≠
pp
```

标识符优先完整可读并支持完整 Copy。

当空间不足而必须视觉截断时：

```text
Rendered text
→ may truncate

Copied value
→ full original value

Accessible / Detail value
→ full original value
```

禁止 Copy 结果本身也是：

```text
015516...-1
```

---

## 23.5 Data State Presentation

Table 必须继承 Section 17 Data State Language。

正式区分：

```text
0
→ valid zero

—
→ no numeric value

N/A
→ genuinely unavailable / not applicable
```

以下状态不得因为 Table 密度高而被静默抹除：

```text
PARTIAL
STALE
UNVERIFIED
ESTIMATED
UNAVAILABLE
NOT_APPLICABLE
```

必要时可以使用紧凑表达：

```text
Value
+
Status Marker / Label
```

例如：

```text
¥18,482
部分数据
```

或者：

```text
382
数据已过期
```

正式原则：

```text
Table density
≠ semantic loss
```

`NULL / UNAVAILABLE / ERROR / STALE` 均不得显示成 `0`。

---

## 23.6 Sorting

Sorting 必须针对：

```text
entire filtered result set
```

而不是当前已经加载的一页。

正式关系：

```text
Sort
+
Filter
+
Search
+
Pagination
→ one consistent query state
```

禁止：

```text
Page 3
→ sort only visible 50 rows
```

并让用户误以为这是全局排序结果。

Applied Sort 必须可见：

```text
Revenue ↓
```

规则：

1. 后端存在 Default Sort 时，前端必须显示对应排序状态；
2. 不支持合法排序的列，不显示 Sort Affordance；
3. Sort Direction 不能只依赖极弱颜色变化；
4. Keyboard / Screen Reader 必须能够读取当前 Sort State；
5. 多列排序只有在产品确实需要时引入，不作为 v1 默认复杂度。

正式原则：

```text
Applied Sort
→ visible
```

---

## 23.7 Filtering and Search

### Filtering

Filtered Empty 必须与 Dataset Empty 分开。

例如：

```text
当前筛选条件下没有订单
[清除筛选]
```

而不是：

```text
暂无订单
```

如果当前有 Active Filter，应能发现当前筛选状态。

正式原则：

```text
Filtered Empty
≠ Dataset Empty
```

### Search

Search 必须明确 Scope。

例如订单表：

```text
搜索母订单号、订单号、SKU
```

用户必须能够理解搜索是针对：

- 当前页；
- 当前 Filter 后的完整 Dataset；
- 哪些字段；
- 是否支持精确 / 模糊匹配。

O3Pilot 默认业务 Table Search 应针对完整 Query Scope，而不是只搜索浏览器当前已加载页。

如果某个功能确实只是当前页查找，必须明确写成：

```text
当前页查找
```

正式原则：

```text
Search Scope
→ explicit
```

---

## 23.8 Pagination

O3Pilot 正式默认：

```text
Pagination / Cursor Pagination
→ preferred

Infinite Scroll
→ not default
```

原因：

- 分析位置更容易理解；
- 查询状态更容易复现；
- Back / Forward 更稳定；
- 批量扫描更可控；
- 大数据不持续堆积 DOM。

Pagination UI 必须反映真实后端语义。

例如 Cursor API 不知道完整 Total 时，不得伪造：

```text
Page 7 of 42
```

可以使用：

```text
上一页
下一页
```

或：

```text
已显示 50 条
还有更多
```

不知道 Total 时，不得推测或显示为 0。

正式原则：

```text
Pagination UI
→ follows backend semantics

Unknown Total
→ never fabricated
```

如果后端提供可靠 Total，则可以显示精确总数和页码。

---

## 23.9 Sticky Header / Key Column

### Sticky Header

对超过一个 Viewport 高度的核心分析表：

```text
Sticky Header
→ default
```

Sticky Header 必须：

- 保持不透明或足够可读的 Surface；
- 不遮挡第一行；
- 不覆盖 Focus Ring；
- 使用 Hairline / Surface Difference 分层；
- 不依赖明显 Shadow。

### Sticky Key Column

仅当其主要作用是保留 Row Identity 时使用。

典型：

```text
Product
SKU
Order Identifier
Campaign
```

正式原则：

```text
Sticky Key Column
→ preserve row identity
```

禁止把前 3–4 列机械全部 Sticky，导致有效数据区域被挤压。

Frozen 与 Scrollable 区域使用：

```text
hairline / subtle surface boundary
```

而不是大面积阴影。

---

## 23.10 Horizontal Scrolling

对于宽分析表：

```text
Horizontal Scroll
→ acceptable
```

O3Pilot 不为了消灭横向滚动而：

- 删除关键列；
- 把多列表格 Card 化；
- 缩小字体到不可读；
- 把关键金额和标识符强制省略。

正式关系：

```text
Wide analytical table
→ horizontal scroll allowed

Key identity
→ preserved

Critical columns
→ prioritized
```

Horizontal Scroll 必须：

- 可通过 Trackpad / Mouse / Keyboard 正常操作；
- 不阻断 Focus Navigation；
- Sticky Key Column 不遮挡滚动内容；
- Narrow 屏幕仍保持可用。

---

## 23.11 Column Visibility and Layout State

复杂 Table 可以支持：

```text
Column Visibility
```

其本质是：

```text
local presentation state
```

不得修改：

- Query Data；
- Metric Calculation；
- Ozon Server State；
- 原始业务事实。

正式列角色：

```text
Required Identity Column
Core Analytical Column
Optional Column
```

Required Identity Column 不应被配置到让 Row Identity 完全消失。

如果未来支持 Column Order：

```text
Column Order
→ O3Pilot local preference
```

但 v1 不要求为了“可定制”强制实现 Drag-and-drop。

正式原则：

```text
Customization
→ only when it improves repeated analytical work
```

Export 与 Column Visibility 语义必须分开：

```text
Visible Columns
≠ Export Scope by default
```

只有用户明确选择“仅导出可见列”时，Export 才按当前 Visibility 处理。

---

## 23.12 Row Interaction / Detail / Copy

### Row Interaction

Table Row 默认不是 Button。

建议优先：

```text
Primary Identifier
→ opens detail

Explicit Detail Control
→ opens detail
```

只有具体 Table Contract 明确需要整行打开 Detail 时，才允许 Row Click，并必须：

- 有明确 Hover / Focus Affordance；
- Keyboard 可访问；
- 不干扰文本选择；
- 不干扰 Copy；
- 不干扰 Checkbox / Link / Action；
- 不把展示型 Row 视觉 Card 化。

正式原则：

```text
Row
≠ Button by default
```

### Copy

高频 Identifier 在 Hover / Focus 时可以暴露 Copy Action。

反馈：

```text
Copy
→ Check
```

允许复用 Section 13 / 14 的 Morph Feedback。

但：

```text
Copy
≠ row navigation
```

点击 Copy 不得同时打开 Detail。

---

## 23.13 Selection and Actions

### Row Actions

Action Column：

```text
right aligned
```

默认避免每行常驻多个文字按钮。

建议：

```text
one primary high-frequency action
+
More menu when needed
```

允许的典型操作包括：

- Copy；
- Open Detail；
- Export local data；
- O3Pilot Local State Actions。

任何 Row Action 均不得：

```text
mutate Ozon server state
```

### Selection

Checkbox Selection 只有在存在真实 Multi-row Action 时才出现：

```text
Row Selection
→ only when a real multi-row action exists
```

如果没有 Bulk Action：

```text
→ no checkbox column
```

Selection 不得演化为：

- 批量改价；
- 批量改库存；
- 批量上下架；
- 批量修改广告；
- 其他 Ozon 写操作。

正式原则：

```text
Selection
→ purpose-driven
not table decoration
```

---

## 23.14 Loading / Refresh / Stale

### Initial Loading

首次加载可以使用 Row Skeleton，但必须尽量保持 Table Geometry 稳定：

```text
Skeleton
→ preserves row / column geometry
```

Reduced Motion：

```text
no shimmer
```

### Refresh

禁止：

```text
Filter / Refresh
→ entire table disappears
→ spinner
→ table reappears
```

如果已有数据，应尽量保留已有结构和 Last Known Rows，并使用轻量 Loading State。

当 Refresh 失败但存在 Last Known Data：

```text
keep table
+
show failure / stale state
+
show last valid time when relevant
```

正式原则：

```text
Retrieval Failure
≠ Empty Table
```

### Stable Reading

背景数据变化不应让 Table 持续重排。

```text
Background data update
→ stable reading first
```

当前 Sort 的确要求重新排序时，可以在正式 Refresh / Query Update 后重新排序；但不要在用户正在 Copy、选择、阅读时每秒重新排列 Row。

---

## 23.15 Responsive Tables

Narrow 不自动 Cardify Table。

正式 Pattern：

```text
Narrow
├── Preserve table semantics
├── Key columns remain visible
├── Horizontal scroll where needed
└── Row Detail for secondary fields
```

允许：

- 减少默认可见的 Optional Columns；
- 优先保留 Identity / Critical Metrics；
- Horizontal Scroll；
- 通过 Detail 查看 Secondary Fields。

禁止：

- 静默删除重要列；
- 把 12 列数据自动变成 12 层 Card；
- 为了“移动端适配”改变 Metric；
- 静默缩短 Date Range；
- 隐藏 Status / Currency / Unit。

正式原则：

```text
Responsive
→ prioritizes columns
not changes data model
```

Hidden Column 必须仍能通过 Detail / Column Control 被访问。

---

## 23.16 Performance

大型 Table 必须继承 Section 15 Performance Contract：

```text
Large dataset
→ remains interactive
```

禁止：

- 每个 Cell 建复杂 Observer；
- Hover 导致全表 Reflow；
- 每行持续 Animation；
- 一条数据更新重建整个 Table；
- 为了“全部展示”一次性创建无限 DOM；
- Scroll Handler 中执行大量同步业务计算。

允许根据数据规模采用：

```text
Server Pagination
Virtualization
Incremental Rendering
```

但：

```text
Performance technique
≠ UX semantics
```

如果使用 Virtualization，必须继续保证：

- Keyboard Navigation 正常；
- Sticky Header / Key Column 正常；
- Copy 正常；
- Scroll Position 稳定；
- Row Height Contract 可预测；
- Detail 返回后能够恢复合理位置；
- Screen Reader 语义不因虚拟化而失真。

---

## 23.17 Accessibility

默认使用真正的：

```text
semantic table
```

如果主要交互是：

```text
read
sort
filter
open detail
copy
```

则不应因为使用复杂 Table Library 就自动变成：

```text
role="grid"
```

只有真正存在 Cell-level Keyboard Interaction / Editable Grid 交互模型时，才考虑 ARIA Grid。

正式原则：

```text
Semantic Table
→ default

ARIA Grid
→ only when interaction model requires it
```

Accessibility Contract：

1. Header Sort 可通过 Keyboard 操作；
2. 当前 Sort Direction 可被 Screen Reader 识别；
3. Filter Control 有 Accessible Label；
4. Search 有明确 Scope 和 Accessible Name；
5. Column Visibility 有 Accessible Name；
6. Copy 与 Row Action 可通过 Keyboard 使用；
7. Horizontal Scroll 不阻断 Focus；
8. Sticky Header / Column 不遮挡 Focus Ring；
9. Status 不能只靠颜色表达；
10. Hidden / Truncated Identifier 必须能够获取完整值；
11. Loading / Empty / Error / Stale 状态必须有文字语义；
12. Checkbox Selection 必须正确表达 Selected State。

---

## 23.18 Table QA

正式 Table QA 至少覆盖：

```text
Standard Density
Compact Density

Text Column
Identifier Column
Numeric Column
Money Column
Ratio / Percentage
Date / Time
Status
Actions

Known Zero
Missing
Unavailable
Partial
Stale
Estimated

Ascending Sort
Descending Sort
Default Sort
Server-side Sort

Filter Active
Filtered Empty
Dataset Empty

Search
Search Scope
Exact Identifier Search

Known Total Pagination
Unknown Total / Cursor Pagination

Sticky Header
Sticky Key Column
Horizontal Scroll

Column Visibility
Required Identity Column

Copy Identifier
Open Detail
Row Actions
Selection with valid bulk action
No-selection table

Initial Loading
Refresh
Refresh Failure with Last Known Data

Large Dataset
Virtualized implementation when used

1440px
1200px
1024px
768px
390px

Mouse
Keyboard
Screen Reader semantics
200% zoom

Light
Dark
CN
EN
RU
```

验收必须确认：

1. `40 / 44 / 36px` Geometry Contract 未被 Feature 自行修改；
2. Identifier 未被当成 Numeric Measure；
3. Comparable Numeric 使用 Tabular Numbers；
4. `0 / — / N/A` 语义正确；
5. PARTIAL / STALE 等状态没有因 Table 密度被隐藏；
6. Sort 作用于完整 Filtered Result Set；
7. Applied Sort 可见；
8. Filtered Empty 与 Dataset Empty 区分；
9. Search Scope 明确；
10. Pagination UI 与 Backend Semantics 一致；
11. Unknown Total 未被伪造；
12. Wide Table 在 Narrow 下仍保留 Table Semantics；
13. Row 默认不是 Button；
14. Selection 只有在有真实 Multi-row Action 时出现；
15. Business Fact Table 保持 Read-only；
16. 没有任何 Table Action 修改 Ozon Server State；
17. Retrieval Failure 未被显示成 Empty Table；
18. Large Dataset 仍保持交互可用；
19. Accessibility 默认使用 Semantic Table，未无理由套用 ARIA Grid；
20. Sticky / Scroll / Focus 在 Keyboard 与 200% Zoom 下正常。

核心 Contract：

```text
Table
≠ Spreadsheet by default

Identifier
≠ Numeric Measure

Sort
→ entire filtered result set

Applied Sort
→ visible

Filtered Empty
≠ Dataset Empty

Pagination UI
→ follows backend semantics

Unknown Total
→ never fabricated

Wide Table
→ horizontal scroll allowed

Responsive
→ prioritizes columns
not changes data model

Row
≠ Button by default

Selection
→ only when real batch action exists

Business Fact Table
→ read-only

Retrieval Failure
≠ Empty Table

Semantic Table
→ default

ARIA Grid
→ only when required
```

---

# 24. Forms

O3Pilot 的 Form 用于编辑 O3Pilot 自身拥有的可写状态、提交本地配置与业务输入，以及收集 Credential 或 Seller-owned Data。

正式边界：

```text
Form
→ edits O3Pilot-owned state
→ submits local configuration / input
→ collects credentials or seller-owned data

Form
≠ permission to mutate Ozon
```

因此：

```text
O3Pilot Local State
→ writable when product requires it

Ozon Server State
→ never writable
```

Form 的存在不能被解释为 Ozon 写能力。商品、价格、库存、订单、广告、Webhook Subscription 等 Ozon Server State 均不得因为使用 Form Pattern 而获得修改入口。

---

## 24.1 Form Principles

正式原则：

```text
Label
→ always persistent

Placeholder
≠ Label

Read-only
≠ Disabled

Validation
→ timely
not premature

Submission Failure
→ preserve input

Commit Model
→ explicit

Unit / Currency
→ persistent context
```

同时遵循：

```text
Clear ownership
Clear commit behavior
Recoverable errors
Accessible interaction
No hidden destructive default
```

Form 不用于把普通 Business Fact 伪装成 Disabled Input，也不以输入控件数量制造“后台系统感”。

---

## 24.2 Form Ownership and Write Boundary

允许通过 Form 修改的对象必须属于 O3Pilot 正式可写范围，例如：

- O3Pilot Settings；
- Seller-owned Cost；
- Local Mapping；
- Import Parameters；
- Backup Configuration；
- Alert / Notification 的 O3Pilot 本地配置；
- Appearance / View Preference；
- Credential Replacement。

禁止出现任何可直接修改 Ozon Server State 的 Form，例如：

```text
修改 Ozon 商品价格
修改 Ozon 库存
修改 Ozon 商品内容
修改 Ozon 广告出价
修改 Ozon 订单状态
修改 Ozon Webhook Subscription
```

正式关系：

```text
Form Ownership
→ discoverable from context

Local Write
≠ Ozon Write
```

如果一个页面同时展示 Ozon Fact 与 O3Pilot Local Setting，二者必须通过信息结构清晰区分，不能让用户误以为保存本地设置会修改 Ozon。

---

## 24.3 Field Anatomy

标准 Field：

```text
Field

Label
↓
Control
↓
Helper / Validation Message
```

必要时 Label 行可以包含：

```text
Field Name
+
Required / Optional State
```

规则：

1. Label 始终可见；
2. Placeholder 只用于 Example / Hint；
3. Placeholder 不承担字段唯一语义；
4. Helper Text 只在能减少歧义或解释格式时使用；
5. Error / Validation Message 位于对应 Field 附近；
6. Unit、Currency、格式约束等重要上下文不能只存在于 Placeholder；
7. Label 点击应聚焦对应 Control。

禁止：

```text
[ 请输入 Client ID ]
```

作为唯一字段说明。

应使用：

```text
Client ID
[________________]
```

---

## 24.4 Geometry and Spacing

单行 Control 正式高度：

```text
Desktop
→ 40px

Touch / Narrow
→ ≥ 44px interactive target
```

不再定义 `41 / 42 / 43px` 等 Feature-specific 单行输入高度。

Checkbox、Radio、Icon 等视觉图形可以小于 44px，但 Touch / Narrow 下完整 Interactive Target 必须满足至少 44px。

Field Spacing 复用第 12 章 Token：

```text
Label → Control          8px
Control → Helper/Error   4px
Field → Field           16px
Form Group → Group      24px
```

正式原则：

```text
Spacing Tokens
→ reused

Feature-specific arbitrary spacing
→ prohibited
```

---

## 24.5 Layout and Grouping

Settings、Credentials、Configuration 等 Form 默认使用单列结构：

```text
Single Column
→ default
```

原因：

- Label 与 Control 关系更清晰；
- CN / EN / RU 长文案更稳定；
- Validation Message 不会破坏相邻 Field；
- Narrow 可自然降级。

双列仅用于具有明确强关系的字段：

```text
Two Columns
→ semantic relationship required
```

不能仅因为桌面存在空白就机械使用双列。

Form Group 优先通过：

```text
Section Heading
+
Spacing
+
Optional Divider / Surface when needed
```

组织。

禁止：

```text
Card
└── Card
    └── Card
```

式的表单嵌套。

---

## 24.6 Required / Optional Fields

Required / Optional State 必须同时具备视觉与程序语义。

正式规则：

```text
Required
≠ color only
≠ asterisk only
```

允许使用 `*` 作为视觉辅助，但辅助技术必须能够识别 Field 为 Required。

复杂 Form 可以显式标记：

```text
可选
```

但同一 Form 内必须保持一致的 Required / Optional 表达策略。

不能依靠用户提交后才发现哪些 Field 属于必填项。

---

## 24.7 Disabled / Read-only States

正式区分：

```text
Read-only
→ value exists
→ inspectable
→ often selectable / copyable
→ not editable

Disabled
→ control is currently unavailable
→ not operable
```

因此：

```text
Read-only
≠ Disabled
```

普通 Ozon Business Fact 不应通过 Disabled Input 展示。

例如：

```text
[ 18,482 RUB ] disabled
```

不作为普通金额事实的展示方式。

应使用正常 Display / Metric / Detail Presentation。

如果 Disabled 原因不明显：

```text
Disabled Reason
→ discoverable
```

不能只使用降低透明度表达不可操作。

---

## 24.8 Validation

Validation Lifecycle：

```text
User typing
→ do not aggressively interrupt

Field blur after interaction
→ field validation may appear

Form submit
→ validate complete form

Server response
→ server / integration validation
```

正式原则：

```text
Validation
→ timely

not
→ premature
```

用户输入尚未完成时，不应持续触发明显错误反馈。

例如 URL Field 在用户只输入到 `htt` 时，不应立即显示强烈的 `Invalid URL` 错误。

Client-side Validation 不替代 Server-side Validation。

---

## 24.9 Errors and Recovery

Error 必须：

```text
Specific
Near the affected field when applicable
Actionable when the system knows the remedy
Non-color-only
```

禁止仅显示：

```text
Invalid value
Error 400
Validation failed
```

如果系统知道原因，应使用可修复文案，例如：

```text
Client ID 不能为空
```

或：

```text
文件格式不受支持，仅支持 CSV / XLS / XLSX
```

正式区分：

```text
Field Validation Error
Form Validation Error
Integration Error
System Error
```

不能把 Integration Error 伪装成输入格式错误。

提交失败时：

```text
Submission Failure
→ preserve user input
```

不能因为一个字段失败而清空其他已填写内容。

多错误 Form 提交失败后：

```text
Focus / announce
→ first relevant error
```

长 Form 可以额外提供错误 Summary，但 Summary 不替代 Field-level Error。

---

## 24.10 Commit Models and Dirty State

每个正式 Form 必须明确属于一种 Commit Model：

```text
Explicit Save
Immediate Local Apply
Preview / Draft
```

### Explicit Save

适用于：

- Credentials；
- Seller Cost；
- Backup Configuration；
- Mapping；
- 关键 O3Pilot Settings。

流程：

```text
Edit
↓
Save
↓
Confirmed State
```

### Immediate Local Apply

只适用于：

```text
low-risk
reversible
O3Pilot-local preference
```

例如适合立即生效的 Appearance / View Preference。

不得用于 Secret 或高风险配置。

### Preview / Draft

适用于需要先检查再提交本地结果的复杂工作流，例如某些 Import / Mapping Review。

正式原则：

```text
Commit Model
→ explicit

One form group
→ must not silently mix commit semantics
```

Explicit Save Form 必须能够区分：

```text
Saved
Edited / Unsaved
Saving
Save Failed
```

有实质性未保存内容时，离开页面是否需要 Navigation Protection 由丢失成本决定；不得为所有微小本地设置机械弹确认框。

---

## 24.11 Async Submission

异步提交中：

```text
Save
→ Saving…
```

规则：

1. 防止重复提交；
2. 不使整页内容消失；
3. 用户必须知道当前操作仍在进行；
4. Failure 后保留输入并允许 Retry；
5. Success 只在服务端确认成功后显示；
6. Critical Configuration / Credential 不使用未经确认的 Optimistic Success；
7. Loading Feedback 遵循第 14、16、29 章 Motion / Reduced Motion / Loading Contract。

正式关系：

```text
Saving
≠ Saved
```

以及：

```text
Cannot submit
≠ silently disabled button
```

Disabled Submit 只有在不可提交原因明显或可发现时才允许作为主要策略；多数 Validation 应通过明确错误帮助用户修复。

---

## 24.12 Credentials and Secrets

Credential Form 必须服从 `SECURITY.md` 的 Secret 分类与处理 Contract。

正式 UX：

```text
Saved Secret
→ never redisplayed as plaintext
```

保存后的 Credential 默认展示：

```text
Status
Necessary mask when Security Contract allows it
Last validation state / time when available
```

例如：

```text
Seller API Key
已配置
••••••••A7F2
最后验证：10:28
```

修改 Secret 使用：

```text
Replace Credential
```

而不是：

```text
Reveal Existing Secret
```

正式规则：

```text
Stored Secret
→ status / mask only
→ replace, not reveal
→ no Copy by default
```

是否允许掩码显示、哪些 Identifier 属于 Secret，必须服从 Security Contract；DESIGN 不自行降低安全分类。

Password / Secret Field：

```text
→ must not block paste
→ must not block password manager
```

Credential Test 可以存在，但必须：

```text
Verify Credential
→ explicit read-only allowlisted endpoint
```

禁止以“测试连接”为名触发任何 Ozon Business Write。

---

## 24.13 Numeric / Money / Unit Inputs

数值输入必须保留永久 Unit Context。

例如：

```text
采购成本
[ 120.00 ] CNY
```

而不是仅在 Placeholder 中写入 Currency。

正式原则：

```text
Unit / Currency
→ persistent context
```

适用于：

- Money / Currency；
- Weight；
- Percentage；
- Days；
- 其他有单位的 Seller-owned Input。

如果允许选择 Currency，则 Currency Selector 明确属于该值本身。

输入过程中：

```text
Typing
→ stable raw interaction

Blur / Commit
→ display formatting may apply
```

禁止持续格式化造成 Caret Jump，例如用户输入 `1234` 时立即反复改写成 `1,234.00`。

正式关系：

```text
Display Formatting
≠ Calculation / Storage Precision
```

具体 Parsing、Rounding 与业务精度由上游 Contract 定义。

---

## 24.14 Checkbox / Radio / Select / Switch

控件语义：

```text
Checkbox
→ independent boolean / multi-select

Radio
→ one choice among a small mutually exclusive set

Select
→ larger option set

Switch
→ immediate reversible local setting
```

因此：

```text
Switch
+
Save button
```

通常属于 Commit Semantic 冲突。

如果 Boolean 必须等用户显式 Save 后才能提交，优先使用 Checkbox / Form Control，而不是表现为立即生效的 Switch。

控件选择必须由交互语义决定，而不是仅由视觉偏好决定。

---

## 24.15 File Inputs

Official Report / Seller-owned Data 等文件导入 Form 必须将文件视为不可信输入。

UX 至少区分：

```text
Selected
Validated
Parsed
Imported
```

正式关系：

```text
Selected File
≠ Validated File
≠ Parsed File
≠ Imported File
```

需要显示的状态包括：

- Selected File；
- File Type；
- File Size when useful；
- Validation State；
- Parsing State；
- Import Result；
- Failure Reason when known。

Drag & Drop 可以作为增强交互，但必须同时存在普通 File Picker；不得成为唯一文件输入方式。

用户选择文件后，不能在尚未完成验证和解析前显示“导入成功”。

---

## 24.16 Responsive Forms

Narrow 下：

```text
Multi-column Form
→ Single Column
```

字段内部顺序保持：

```text
Label
Control
Helper / Error
```

禁止：

- 缩小字体以塞入多列；
- 隐藏 Label；
- 将必要 Helper / Error 改成 Tooltip-only；
- 横向挤压两个长输入框；
- 截断关键错误文本；
- 因 Narrow 改变字段语义或 Commit Model。

Touch / Narrow 的 Interactive Target 必须满足至少 44px。

---

## 24.17 Accessibility

Form 必须满足：

```text
Label
→ programmatically associated

Error
→ associated with field

Required
→ programmatically discoverable

Focus
→ visible

Tab Order
→ logical
```

正式规则：

1. Label 点击可聚焦对应 Control；
2. Error 不只依赖颜色；
3. Helper / Error 能被辅助技术与 Field 关联；
4. Keyboard 可以完成完整 Form；
5. 200% Zoom 不丢失输入、Label、Helper 和 Error；
6. Disabled Reason 在必要时可发现；
7. Touch / Narrow Interactive Target ≥ 44px；
8. Password / Secret 允许 Paste 与 Password Manager；
9. Placeholder 不承担唯一语义；
10. CN / EN / RU 长 Label、Helper、Error 必须正常换行；
11. Async Submission 状态必须可被辅助技术发现；
12. Focus 不因 Sticky / Overlay / Validation 更新而被无意义重置。

---

## 24.18 Form QA

正式 Form QA 至少覆盖：

```text
Standard text input
Long text input
Number input
Money + currency
Unit input
Required / Optional
Helper text
Field validation
Multiple errors
Server / integration error
Submission retry
Read-only
Disabled + reason
Explicit Save
Unsaved state
Immediate local preference
Credential configured state
Credential replacement
Password manager / paste
Checkbox
Radio
Select
Switch
File picker
Drag & drop
File validation / parsing / import states
Desktop
Narrow
Keyboard-only
200% zoom
Light
Dark
CN
EN
RU
```

同时必须验证：

```text
Form
→ O3Pilot-owned writable state

Form
≠ Ozon write capability
```

并验证：

```text
Desktop single-line control
→ 40px

Touch / Narrow
→ ≥ 44px target

Stored Secret
→ status / mask only
→ replace, not reveal

Selected File
≠ Imported File
```

---

# 25. Buttons

O3Pilot Button 用于执行明确动作，不承担导航语义，也不能突破产品与安全边界。

正式关系：

```text
Hierarchy
≠ Intent
≠ Form Factor
```

Button 的三个独立维度：

```text
Prominence
├── Primary
├── Secondary
└── Ghost

Intent
├── Default
└── Destructive

Form
├── Label
├── Icon + Label
└── Icon-only
```

因此一个 Button 可以同时是：

```text
Secondary + Destructive + Icon + Label
```

而不是在 `Secondary` 与 `Destructive` 之间二选一。

## 25.1 Button Principles

正式原则：

```text
Button
→ performs an action

Link
→ navigates

Primary
→ expresses local decision priority

Button prominence
≠ permission

Important
≠ Destructive

No meaningful Primary Action
→ no Primary Button
```

O3Pilot 不因为页面需要“视觉焦点”而人为制造 CTA。

分析页可以完全没有 Primary Button。

Button 视觉层级必须服务真实操作优先级，而不是成为装饰。

## 25.2 Action Boundary

所有 Button 都必须服从 O3Pilot 的只读产品边界：

```text
Ozon Server State
→ Read-only

O3Pilot Local State
→ Writable when product requires it
```

因此任何 Button，无论视觉强调程度，都不得执行或暗示以下 Ozon 写操作：

```text
修改 Ozon 商品内容
修改 Ozon 价格
修改 Ozon 库存
修改 Ozon 订单
修改 Ozon 广告 / 出价 / 推广配置
修改 Ozon Webhook 配置 / 订阅
其他 Ozon 业务状态写入
```

允许的典型动作包括：

- 打开详情；
- 复制；
- 搜索；
- 筛选；
- 导入本地数据；
- 保存 O3Pilot 设置；
- 更新 Seller-owned Data；
- 保存本地 Mapping；
- 导出；
- 执行本地计算、模拟、回测或重新验证；
- 对 O3Pilot Local State 执行明确允许的写操作。

正式原则：

```text
Primary
Secondary
Ghost
Destructive
→ all remain inside O3Pilot product boundary
```

## 25.3 Prominence

正式 Prominence：

```text
Primary
Secondary
Ghost
```

### Primary

Primary 表示当前行动上下文中最主要的完成动作。

正式规则：

```text
One action context
→ one dominant Primary
```

典型：

```text
Page-level action group
→ ≤ 1 Primary

Form
→ ≤ 1 Primary submit

Modal / Drawer footer
→ ≤ 1 Primary completion action
```

不同、彼此独立的 Form Section 可以各自拥有自己的 Primary；不要求整个长页面全局只能有一个 Primary。

例如：

```text
Seller API Credential
→ 保存凭证

Backup Configuration
→ 保存备份设置
```

两者属于不同 Action Context，可以各自有 Primary。

分析页如果没有真实的完成动作：

```text
→ no Primary Button
```

禁止为了“页面完整”增加巨型蓝色 CTA。

### Secondary

Secondary 用于支持当前主要任务但优先级较低的动作，例如：

- 取消；
- 重新验证；
- 打开次级配置；
- 导出；
- 其他不应与 Primary 竞争注意力的操作。

### Ghost

Ghost 用于低强调、上下文明确的 Utility Action，例如：

- 查看来源；
- 查看定义；
- 清除筛选；
- 低频辅助入口。

Ghost 不应被用于隐藏实际重要的完成动作。

## 25.4 Intent

正式 Intent：

```text
Default
Destructive
```

`Destructive` 描述动作后果，而不是视觉层级。

真正 Destructive 的典型对象是 O3Pilot Local State，例如：

- 删除本地配置；
- 删除本地 Mapping；
- 清除本地导入结果；
- 恢复 / 替换本地状态；
- 删除明确允许删除的 O3Pilot 本地记录。

以下并不因为“重要”而自动成为 Destructive：

```text
重新计算利润
重新同步
重新验证 Credential
重新生成预测
导出数据
```

正式关系：

```text
Important
≠ Destructive

Warning
≠ Destructive
```

Destructive Intent 可以与 Secondary / Ghost / Primary Prominence 组合。

推荐：

```text
Routine destructive entry
→ Secondary / Ghost + Destructive Intent

Final destructive confirmation
→ stronger Destructive treatment may be used
```

Danger Color 的强度应与真实破坏性决策点成比例，而不是在页面入口就高强度强调。

## 25.5 Form Factors

正式 Form Factor：

```text
Label
Icon + Label
Icon-only
```

### Label

纯文字 Button 适合文案已经足够明确的动作。

### Icon + Label

当 Icon 能辅助扫描，但文字仍承担主要语义时使用。

### Icon-only

Icon-only 不是按钮层级，只是 Form Factor。

必须遵守第 13 章 Icon Contract：

```text
Icon-only
→ familiar
→ low ambiguity
→ repeated utility action
```

典型适用：

- Search；
- Close；
- More；
- Copy；
- 高密度 Toolbar 中语义稳定的 Filter / Utility Action。

不适合：

- 导入官方报告；
- 恢复备份；
- 修改复杂 Mapping；
- 执行利润回测；
- 其他依赖文字才能准确理解的业务动作。

所有 Icon-only Button 必须具备：

```text
Accessible Name
Tooltip
Visible Focus
Touch Target when applicable
```

正式原则：

```text
Icon-only
≠ save space at any cost
```

## 25.6 Sizes and Geometry

正式尺寸：

```text
button-compact    32px
button-standard   40px
touch-target     ≥44px
```

### Standard Desktop

```text
height = 40px
```

用于：

- Form Submit；
- Settings Action；
- 普通页面级动作；
- Modal / Drawer Footer Action；
- 其他常规 Button。

### Compact Desktop

```text
height = 32px
```

只用于：

- Dense Toolbar；
- Table Utility；
- 非主要 Inline Action；
- 明确需要高密度的 Desktop Context。

Compact 不用于：

- 关键 Save；
- Credential Submit；
- Destructive Confirmation；
- Narrow Touch UI。

### Touch / Narrow

Interactive Target：

```text
≥ 44px
```

视觉 Button 高度与 Hit Target 必须保证触控可用性。

禁止 Feature 自行引入：

```text
34px
38px
42px
```

等新的 Button 尺寸层级。

Geometry 遵守第 11 章 Shape Contract；Button 不因为更重要而额外变圆。

## 25.7 Labels

Button Label 应描述执行后的具体动作。

优先：

```text
保存设置
导入文件
复制建议
重新验证
导出报告
删除本地配置
```

避免脱离上下文后含义不明确的：

```text
确定
提交
执行
继续
```

Dialog / Confirmation 中尤其应使用明确动作：

```text
是否删除本地配置？

[取消] [删除本地配置]
```

优于：

```text
[否] [确定]
```

正式原则：

```text
Button Label
→ describes the resulting action
```

关键操作文案不得通过 Ellipsis 截断到改变动作理解。

CN / EN / RU 长文案应通过合理宽度、换行策略或 Narrow Full-width 保持完整语义。

## 25.8 States

正式 Button State：

```text
Default
Hover
Pressed
Focus
Disabled
Loading
```

### Hover

Hover 是 Mouse 环境增强反馈，不是唯一交互反馈。

### Pressed

Press Feedback 应：

```text
immediate
subtle
non-blocking
```

优先：

- Background；
- Border；
- Color；
- Subtle Opacity。

普通 Button 默认禁止：

```text
scale(0.95)
noticeable bounce
lift
spring
```

### Focus

Focus 必须可见，且不能因为 Hover / Loading / Motion 状态被削弱。

### Touch

Touch 不能依赖 Hover 才表达可操作性。

正式关系：

```text
Hover
→ enhancement

Focus
→ required

Pressed
→ required interaction feedback
```

## 25.9 Loading and Feedback

执行异步动作时：

```text
Action
↓
Loading
↓
Confirmed Success / Failure
```

例如：

```text
保存设置
↓
正在保存…
↓
已保存
```

正式规则：

1. Loading 必须防止同一动作重复提交；
2. Loading 不冻结整个页面，除非业务操作本身要求；
3. Button Geometry 应保持稳定，避免状态文案变化导致周围布局跳动；
4. Action Meaning 必须保留，不能只剩一个无语义 Spinner；
5. `Saving` 不等于 `Saved`；
6. 只有确认成功后才能显示成功状态；
7. Failure 后必须能够理解失败并重试；
8. Focus 不应因 Loading 状态被无意义移走。

正式原则：

```text
Loading
→ preserve button geometry
→ preserve action meaning
```

Spinner 不是 Loading 必需品。

Reduced Motion 下可以只使用：

```text
正在保存…
```

等文字 / 静态状态，不需要持续旋转。

正式关系：

```text
Loading comprehension
>
continuous animation
```

### Success Feedback

Morphicons 只作为已经批准的例外反馈模式使用。

允许：

```text
Copy
→ Check
```

以及 Sidebar 已定义的：

```text
Original → Check → Original
```

普通：

```text
保存
搜索
筛选
打开
取消
```

不因为成功就默认 Morph 为 Check。

正式原则：

```text
Button morph
→ exception
not default success pattern
```

## 25.10 Disabled Actions

Disabled Button 继续遵守第 24 章 Form Contract。

正式原则：

```text
Cannot submit
≠ silently disabled button
```

当 Disabled 原因天然清晰，例如尚未选择文件，可以 Disabled。

当原因不明显：

```text
Reason
→ discoverable
```

不能只显示一个灰色 Button，让用户无法知道为什么不能操作。

Disabled State 不能代替 Validation Error。

Disabled 也不能只依赖低透明度表达；必须在视觉和语义上保持可辨识。

## 25.11 Action Groups

同一操作组应建立明确层级：

```text
1 Primary
+ 1–2 Supporting Actions
```

更多低频动作可以进入：

```text
More
```

但不能把高风险或关键业务动作隐藏到无法发现的位置。

禁止多个同级强 Primary 互相竞争：

```text
[Blue] [Blue] [Blue] [Red] [Blue]
```

一个 Action Context 中必须能快速判断主要完成动作是什么。

## 25.12 Destructive Actions

Destructive Action 应尽量与普通动作保持空间分隔。

例如：

```text
保存设置        取消

----------------

删除本地配置
```

优于：

```text
保存  删除  取消
```

尤其适用于：

- Settings；
- Backup；
- Credential；
- Data Center；
- 本地 Mapping / Import Management。

正式原则：

```text
Destructive Action
→ separated when practical
```

严重 Destructive Action 的确认方式由对应 Confirmation / Modal Pattern 定义，但无论如何都不得引入 Ozon 写操作。

## 25.13 Modal / Drawer Actions

Modal / Drawer Footer 的常规完成模式：

```text
[Secondary / Cancel] [Primary Completion]
```

O3Pilot v1 的 CN / EN / RU 均为 LTR，Desktop Footer 统一靠右排列：

```text
[取消] [保存]
```

Destructive Confirmation：

```text
[取消] [删除本地配置]
```

其中最终危险动作使用 Destructive Intent。

不同页面不得随意反转 Action Order。

如果 Drawer / Modal 只是查看详情而没有提交动作，则不强行制造 Primary Footer Button。

## 25.14 Responsive Buttons

Desktop 默认：

```text
Button
→ intrinsic width
```

Narrow 可以在有利于完成任务和触控清晰度时使用 Full-width，例如：

- Form Submit；
- 独立完成动作；
- 长 CN / EN / RU Label；
- 需要明确触控目标的确认操作。

但 Narrow 不要求所有 Button 都：

```text
width: 100%
```

例如：

- Copy；
- Filter；
- More；
- Table Utility；

仍可以保持紧凑。

正式原则：

```text
Responsive Button
→ preserves action hierarchy
not blindly width: 100%
```

关键 Action Label 不通过 Ellipsis 隐藏风险或结果语义。

## 25.15 Accessibility

正式 Accessibility Contract：

1. 优先使用原生 `button` 语义；
2. Button 必须支持 Keyboard 操作；
3. `Enter / Space` 行为符合平台语义；
4. Focus 必须清晰可见；
5. Icon-only 必须有 Accessible Name；
6. Disabled State 必须可被辅助技术理解；
7. Loading State 必须可被辅助技术发现；
8. Loading 时不得让辅助技术误以为仍可重复触发；
9. Focus 不因 Loading / Success Feedback 被无意义移走；
10. Touch / Narrow Target ≥44px；
11. Destructive 不只靠红色表达；
12. Disabled 不只靠降低透明度表达；
13. 200% Zoom 下 Label、Focus 和 State 不丢失；
14. CN / EN / RU 长 Label 可完整理解；
15. Button 与 Link 使用与行为一致的语义元素；
16. 禁止使用普通 `div` + click handler 冒充 Button。

正式关系：

```text
Button
→ action

Link
→ navigation
```

即使 Navigation 使用 Button-like Visual Appearance，语义仍必须保持 Link。

## 25.16 Button QA

正式 Button QA 至少覆盖：

```text
Primary
Secondary
Ghost
Default Intent
Destructive Intent
Label
Icon + Label
Icon-only
Compact 32px
Standard 40px
Touch ≥44px
Hover
Pressed
Focus
Disabled
Loading
Success
Failure
Copy → Check
Action Group
Destructive Confirmation
Modal Footer
Drawer Footer
Desktop
Narrow
Light
Dark
Reduced Motion
Keyboard
Touch
200% Zoom
CN
EN
RU
```

必须确认：

1. 一个 Action Context 不出现多个竞争性的 Primary；
2. 无真实 Primary Action 的分析页不制造 CTA；
3. `Destructive` 不被错误当成视觉层级；
4. Icon-only 只有在低歧义、高熟悉度场景使用；
5. Button 不执行或暗示任何 Ozon 写操作；
6. Loading 不改变 Button Geometry；
7. Loading 不提前显示 Success；
8. Disabled 原因在需要时可发现；
9. Destructive Action 在可行时与普通操作分隔；
10. Button Label 明确描述动作结果；
11. CN / EN / RU 长文案不因截断失去动作语义；
12. Keyboard / Touch / Screen Reader 行为与 Mouse 一致可用；
13. Motion 不成为 Button 响应前置条件；
14. Button Morph 只用于批准的例外场景；
15. Button / Link 语义与实际行为一致。

核心规则：

```text
Hierarchy
≠ Intent
≠ Form Factor

One action context
→ one dominant Primary

No meaningful Primary
→ no Primary Button

Primary
≠ Ozon write permission

Important
≠ Destructive

Button
→ action

Link
→ navigation

Desktop Standard
→ 40px

Compact Desktop
→ 32px

Touch / Narrow
→ ≥44px

Loading
→ preserve geometry

Button morph
→ exception

Destructive Action
→ separated when practical
```

---

# 26. Tabs / Filters / Segmented Controls

Tabs、Filters 与 Segmented Controls 都会改变当前界面内容，但三者表达的产品语义不同，不能因为视觉形态相近而互相替代。

正式原则：

```text
Tab
≠ Filter
≠ Segmented Control
```

判断顺序：

```text
Does the user move to another stable peer content domain?
→ Tab

Does the user narrow / constrain the current dataset?
→ Filter

Does the user choose one of a few mutually exclusive modes for the same subject?
→ Segmented Control
```

同一种关系必须长期使用同一种组件语义，不能在不同页面中一会儿用 Tab、一会儿用 Chip、一会儿用 Segmented Control 表达同一个状态。

## 26.1 Component Principles

O3Pilot 使用三类正式语义：

```text
Tabs
→ stable peer information domain

Filters
→ subset / constraint of current dataset

Segmented Control
→ small mutually exclusive presentation / mode choice
```

组件选择先由业务关系决定，再决定视觉形式。

正式关系：

```text
Semantic relationship
→ Component type
→ Visual treatment
```

禁止：

```text
Visual preference
→ choose component
→ reinterpret business relationship
```

例如同样是“胶囊形状”并不能证明某个控件就是 Filter Chip 或 Segmented Control。

同时：

```text
Context
≠ Filter
```

以下已经由前文定义的上下文不得被普通 Filter 重新包装：

- Shop Context；
- Reporting Currency；
- Page-level Date Range；
- Comparison Context；
- Metric Contract 中的正式 Time Basis。

Filter 负责在这些 Context 下继续缩小当前数据集合。

## 26.2 Semantic Decision Rules

### Stable Peer Domain → Tab

Tabs 用于稳定的同级信息域。

例如：

```text
商品详情
[概览] [销售] [库存] [退货]
```

各 Tab：

- 仍属于同一个 Product Detail Context；
- 表达长期稳定的同级信息域；
- 不只是改变同一张表的结果子集。

### Same Dataset + Subset Change → Filter

如果变化只意味着：

```text
same entity type
+
same analytical surface
+
same result structure
+
subset changes
```

则使用 Filter。

例如：

```text
订单
[全部] [异常] [已取消]
```

如果三者只是同一 Orders Dataset 的状态筛选，则它们属于 Filter，而不是 Tabs。

正式原则：

```text
Same dataset + subset change
→ Filter

not
→ Tab
```

不能为了视觉上“像后台导航”而把 Status Filter 伪装成 Tabs。

### Few Mutually Exclusive Modes → Segmented Control

Segmented Control 只用于少量、互斥、同一上下文下的 Mode。

典型：

```text
[数量] [金额]

[按日] [按周]

[图表] [表格]
```

正式范围：

```text
Segmented Control
→ normally 2–4 options
→ same context
→ same subject
→ reversible low-risk mode
```

如果选项数量继续增加，或开始表达不同业务信息域，应改用 Tabs、Select 或其他更合适的 Navigation / Selection Pattern。

禁止：

```text
[销售] [库存] [广告] [利润] [退货] [履约]
```

使用 Segmented Control 表达完整页面结构。

也禁止：

```text
[保留数据] [删除数据]
```

使用 Segmented Control 承担 Destructive Decision。

## 26.3 Tabs

Tabs 属于 Page-local Navigation，不替代产品级 IA。

正式关系：

```text
Global IA
→ Sidebar

Page-local peer content
→ Tabs
```

因此 Sidebar 已经存在的：

```text
销售
商品
订单
库存
履约
退货
```

不能机械复制成页面顶部 Tabs。

### Active State

一个 Tab Group：

```text
exactly one active tab
```

Active State 必须明显且可访问，但不依赖：

- Filled Icon Variant；
- 只有颜色差异；
- 过大的 Pill Surface；
- 强阴影或 Hover Lift。

如果 Tab 使用 Icon，仍遵守 Lucide 的统一 Outline Icon Contract。

### Nested Tabs

正式默认：

```text
One visible tab level
```

Tabs inside Tabs 只作为例外。

当页面出现第二层、第三层同类导航时，优先重新判断是否应该使用：

- Section；
- Filter；
- Segmented Control；
- Detail Drawer；
- Secondary page-local navigation。

正式原则：

```text
Nested Tabs
→ exception
```

## 26.4 Tab State and Navigation

Tab Selection 属于高频交互。

正式规则：

```text
User intent
→ content state changes immediately

Visual indicator
→ follows without blocking
```

不能：

```text
click
→ wait for indicator animation
→ change content
```

Tab Indicator 如需 Motion：

```text
motion-standard
180ms
```

仅允许轻量、局部的 Transform / Opacity / Surface Treatment。

不建议默认使用跨多个 Tab 的长距离“滑动胶囊”动画。

正式原则：

```text
Tab motion
→ localized

not
→ decorative spatial travel
```

Reduced Motion：

```text
Tab state change
→ immediate
→ no spatial travel
```

### State Preservation

Tab 切换不得无理由重置仍兼容的分析上下文。

例如：

```text
库存 → 销售
```

如果两个 Tab 均支持当前：

- Shop；
- Reporting Currency；
- Date Range；
- Comparison；

则应继续保留。

只有业务 Contract 明确不兼容时才移除或改变，并让用户能够理解原因。

### Restorable State

当 Tab Selection 对分析结果有长期意义时，它应进入可恢复的 Page State。

目标行为：

```text
Back
Forward
Refresh
Deep Link
→ selected peer domain remains understandable
```

实现可以使用 URL Query、Route Segment 或等价的前端路由状态；`DESIGN.md` 不绑定具体 Router Library。

## 26.5 Filters

Filter 用于缩小当前 Dataset 或 Entity Set。

典型：

- Status；
- SKU / Product；
- Warehouse / Fulfillment Type；
- Campaign；
- Inventory Risk；
- Data State；
- Numeric Range；
- Entity Attribute。

Filter 不重新定义：

- Shop Context；
- Reporting Currency；
- Date Context；
- Metric Definition；
- Time Basis。

正式：

```text
Context
→ defines analysis frame

Filter
→ narrows entities / records inside that frame
```

### Applied Filter Visibility

Active Filtering 必须可发现。

不能：

```text
6 active filters
+
button only says “筛选”
```

而用户看不到当前为什么只剩少量结果。

至少支持：

```text
筛选 6
```

并能展开或直接查看已应用条件。

当出现 Filtered Empty：

```text
当前筛选条件下没有订单
```

用户必须能够快速看到当前 Applied Filters 并清除或修改。

## 26.6 Quick / Applied Filter Chips

O3Pilot 区分两种 Chip 语义。

### Quick Filter Chip

用于直接触发简单、高频筛选：

```text
[缺货]
[异常]
[有广告]
```

Quick Filter：

```text
→ invokes / toggles filtering
```

必须有明确 Selected State。

### Applied Filter Chip

用于表达已经生效的 Filter State：

```text
状态：已取消  ×
店铺属性：FBP  ×
库存：< 5      ×
```

Applied Filter Chip：

```text
→ represents active state
→ may provide remove action
```

两种 Chip 可以共享视觉语言，但交互语义不能模糊。

### Complex Filters

复杂条件例如：

```text
销售额 1,000–5,000
利润率 < 10%
多个 Warehouse + 多个 Status
```

可以在 Applied State 中压缩显示为 Chip，但复杂编辑必须进入：

```text
Filter Popover / Panel / Drawer
```

正式：

```text
Chip
→ compact state / simple action

Complex filter editing
→ dedicated control surface
```

不能在狭窄 Chip 内直接塞入复杂输入 Form。

## 26.7 Filter Apply Models

O3Pilot 允许两种正式 Filter Apply Model：

```text
Immediate Filter
Explicit Apply Filter
```

### Immediate Filter

适用于：

- Quick Filter Chip；
- 单个 Status；
- 简单 Select；
- 低成本、结果快速的单条件筛选。

行为：

```text
selection
→ applied immediately
→ query state updates
```

### Explicit Apply Filter

适用于：

- Advanced Filter；
- 多字段组合；
- 多个 Numeric / Date-like Range；
- 一次改变会触发较高成本 Query 的 Filter Panel。

行为：

```text
edit draft filters
↓
Apply
↓
query state changes
```

正式原则：

```text
One filter surface
→ one clear apply model
```

同一 Filter Surface 不能在没有说明的情况下同时出现：

- 某些字段立即生效；
- 某些字段必须按 Apply；

导致用户无法判断当前结果到底用了哪一组条件。

### Draft vs Applied

Explicit Apply 模式必须区分：

```text
Draft Filter State
≠ Applied Filter State
```

用户在 Panel 内编辑但尚未 Apply 时，当前页面数据仍然对应 Applied State。

关闭 Panel 后的 Draft：

- 可以按明确规则丢弃；
- 或暂时保留用于再次打开；

但不能偷偷 Apply。

## 26.8 Query State

对于 Table 与分析结果，以下状态应构成一致的 Query State：

```text
Search
+
Filters
+
Sort
+
Date Range
+
Pagination
```

并与适用的：

```text
Shop Context
Reporting Currency
Comparison
Selected Tab / Mode
```

共同决定最终页面结果。

正式原则：

```text
Query controls
→ one coherent state
```

不能让 Search、Filter、Sort、Pagination 各自维护互相矛盾的结果集。

### Filter Change and Pagination

Filter 改变后，如果当前 Pagination Position 已不再有效：

```text
Pagination
→ reset to a valid position
```

但其他仍兼容的：

- Search；
- Sort；
- Date Range；
- Other Filters；

默认保持。

具体 Query Semantics 由页面和后端能力决定，但 UI 不能只排序 / 筛选当前已经加载的某一页，却让用户误以为操作作用于完整结果集。

### Restorable Analytical State

具有分析意义的状态应尽可能可恢复：

```text
filters
search
sort
date range
comparison
selected tab / mode when appropriate
```

目标：

```text
Back / Forward / Refresh / Deep Link
→ analysis remains reproducible
```

以下内容不得进入 URL 或可分享 Query State：

- Secret；
- Password；
- 临时 Credential 明文；
- Form Draft 中的敏感值。

### Detail Relationship

Filter 改变后，如果当前已打开的 Row / Entity Detail 已不属于新结果集：

```text
Filter change
→ must not create contradictory selection state
```

可以：

- 保留 Detail，但明确说明其已不属于当前筛选结果；
- 或按该页面正式 Detail Pattern 关闭 Detail。

具体行为由第 54 章 Page-specific Detail Pattern 继续定义。

## 26.9 Clear / Reset Rules

当存在 Applied Filters 时，应提供可发现的：

```text
清除筛选
```

但：

```text
Clear Filters
≠ Reset Page Context
```

清除筛选不得顺便重置：

- Shop；
- Reporting Currency；
- Date Range；
- Comparison；

除非这些值明确属于当前 Filter Group，而不是前文定义的 Context。

如果页面另有完整：

```text
Reset View
```

则必须明确它会重置哪些状态，不能与“清除筛选”使用相同文案表达不同范围。

## 26.10 Segmented Controls

Segmented Control 表达少量互斥 Mode，不表达完整 Dataset Summary。

推荐：

```text
[数量] [金额]
[按日] [按周]
[图表] [表格]
```

不推荐：

```text
[全部 18,428]
[异常 128]
[利润低 47]
[缺货 22]
```

后者本质上属于 Filter / Dataset State，而不是 View Mode。

正式：

```text
Segmented Control
→ mode

not
→ dataset summary
```

### Immediate Apply

Segmented Control 必须立即生效：

```text
selection
→ immediate
```

不使用：

- Save；
- Apply；
- Confirm。

如果切换需要 Apply，通常说明该选择不应该使用 Segmented Control。

### Context Preservation

例如：

```text
图表 → 表格
```

应继续保留：

```text
same metric
same filter
same date range
same comparison
same currency context
```

正式：

```text
Presentation mode change
→ preserves analytical context
```

除非目标模式在业务上无法表示当前 Context，此时必须显式说明差异。

## 26.11 Data Availability and Counts

### Result Count

Filter Surface 可以显示结果数量，但只有在系统确实知道精确结果数时才显示。

例如：

```text
应用筛选 · 37 条
```

如果 Backend / Cursor Query 不知道精确 Total：

```text
→ do not fabricate count
```

与 Table 的 Unknown Total Contract 一致。

### Option Count

例如：

```text
已取消 (42)
```

只有当 `42` 的统计 Scope 明确且与当前 Context 一致时才允许显示。

禁止把：

- 当前页 Count；
- 全库 Count；
- 其他日期范围 Count；

作为当前 Filter Option Count 而不说明。

如果 Count Contract 不明确：

```text
→ omit count
```

### Unavailable vs Zero

Filter Option 必须继续遵守：

```text
Unavailable
≠ Zero
```

如果某能力在当前 Shop 不可用：

```text
Unavailable
→ disabled / explained
```

如果某合法 Filter Condition 当前确实匹配 0 条：

```text
Zero result
→ valid zero
```

多店铺切换后，稳定 Filter Option 不应因为某店铺无能力就无提示消失；优先显示不可用状态和原因，使 Capability Difference 可理解。

## 26.12 Motion and Loading

Tabs、Filters 与 Segmented Controls 都属于高频交互。

正式 Motion 原则：

```text
State response
→ immediate

Motion
→ localized and non-blocking
```

禁止：

- Filter Chip 大幅 Scale；
- Tab 长距离弹性滑动；
- Segmented Control Spring Bounce；
- Filter Apply 后整页 Route-like Slide；
- 数据结果通过大量行/卡片入场动画重新出现。

数据更新：

```text
Data change
≠ animation event
```

### Query Loading

用户改变 Filter：

```text
Filter control state
→ updates immediately

Result area
→ loading / refreshing state
```

Filter Bar 本身通常仍保持可操作。

避免：

```text
Filter changed
→ entire page disabled
```

Text Search / Text Filter 如果需要 Server Query：

```text
typing
→ local input updates immediately
→ query may debounce
```

具体 Debounce 时间属于实现，不在 DESIGN 固定毫秒值。

正式：

```text
Input responsiveness
>
query completion
```

当用户快速连续改变查询条件：

```text
Latest user intent
→ wins
```

旧请求或旧动画不能覆盖较新的 Query State。

### Background Data Updates

后台数据同步导致 Count / Option Metadata 改变时：

- 可以更新数字；
- 不自动改变 Filter 排序；
- 不移动 Active Control；
- 不丢失 Focus；
- 不把数据变化包装成装饰动画。

## 26.13 Responsive

三种组件在 Narrow 下采用不同策略。

### Tabs

Narrow 默认：

```text
Tabs
→ preserve semantic tabs
→ horizontal scroll when needed
```

不通过缩小字体或截断关键标签强行塞入一行。

不默认自动改成 Select；只有 Tab 数量、标签长度和页面语义确实要求时才使用其他 Navigation Pattern。

### Filters

Narrow 推荐：

```text
Primary Quick Filters
+
Filter button
↓
Advanced Filter Drawer / Sheet-like surface
```

Applied Filter State 仍必须可发现。

不能为了节省空间把所有已应用条件隐藏到一个无 Count、无状态的图标后面。

### Segmented Control

如果 2–4 个短选项可以完整显示：

```text
→ retain Segmented Control
```

如果 CN / EN / RU 文案变长后无法合理显示：

```text
→ use another semantically appropriate control
```

不能通过：

- 极小字体；
- 关键文字省略；
- 过窄 Touch Target；

保留表面上的 Segmented Shape。

Responsive：

```text
preserves semantic relationship
not component geometry at any cost
```

## 26.14 Accessibility

视觉形态不能决定 Accessibility Role。

正式原则：

```text
Visual shape
≠ accessibility role
```

### Tabs

使用真正的 Tabs Pattern：

```text
tablist
tab
tabpanel
```

并正确表达 Active State，例如：

```text
aria-selected
```

Keyboard 必须遵守合理的 Tabs Interaction Pattern。

Tab 切换后：

```text
Focus
→ remains logically on selected Tab
```

不能因为内容刷新跳回页面顶部。

### Filters

Filter 继续使用正确的基础控件语义：

- Button；
- Checkbox；
- Radio；
- Select；
- Input。

视觉做成 Chip 不意味着失去原始交互语义。

Applied Filter Remove Action 必须具有明确 Accessible Name，例如：

```text
移除筛选：状态 = 已取消
```

而不是只有一个无名称的 `×`。

### Segmented Control

根据实际语义优先使用：

```text
radio group
```

或等价的单选模式语义。

必须表达：

- Group Label；
- Current Selection；
- Keyboard Selection；
- Focus State。

### General Accessibility

1. Focus 永远可见；
2. Active / Selected 不能只靠颜色；
3. Touch / Narrow Target ≥ 44px；
4. 200% Zoom 下标签、Applied Filter 与 Clear Action 不丢失；
5. CN / EN / RU 长标签必须可读；
6. Filter Count 不应成为唯一状态说明；
7. Loading / Updating State 必须可被辅助技术理解；
8. 查询结果刷新不能无意义重置 Focus；
9. Disabled / Unavailable Filter Option 必须能够发现原因；
10. Keyboard 用户可以完成 Tab、Filter、Segmented 的完整工作流。

## 26.15 Component QA

正式 QA 至少覆盖：

```text
Tabs
Quick Filters
Applied Filter Chips
Advanced Filter Panel
Immediate Apply
Explicit Apply
Draft vs Applied
Clear Filters
Segmented Controls
Search + Filter + Sort + Pagination
Unavailable Filter Options
Zero-result Filter Options
Known / Unknown Counts
Loading / Rapid Re-filtering
Back / Forward / Refresh / Deep Link
Desktop
Compact
Narrow
Mouse
Keyboard
Touch
Reduced Motion
Light
Dark
CN
EN
RU
200% Zoom
```

必须验证：

1. Tab、Filter、Segmented Control 的语义没有互相替代；
2. 同一 Dataset 的 Subset Change 使用 Filter，而不是伪装成 Tab；
3. Sidebar IA 没有被页面 Tabs 重复复制；
4. 一个 Tab Group 始终只有一个 Active Tab；
5. Nested Tabs 不是默认页面结构；
6. Immediate / Explicit Apply Model 在每个 Filter Surface 内一致；
7. Draft Filter 不会在未 Apply 时偷偷改变结果；
8. Applied Filters 始终可发现；
9. Clear Filters 不会重置 Shop / Currency / Date 等 Context；
10. Search / Filter / Sort / Pagination 构成一致 Query State；
11. Back / Forward / Refresh 后具有分析意义的状态可恢复；
12. Secret / Password / Credential Draft 不进入可分享 URL State；
13. Unknown Result Count 不被伪造；
14. Unavailable 与 Zero-result Filter Option 明确区分；
15. Segmented Control 只表达少量互斥 Mode；
16. View Mode 切换保留兼容的分析上下文；
17. Filter 变化不会产生与当前 Detail 冲突的 Selection State；
18. 快速连续输入和筛选以最新用户意图为准；
19. Loading 不锁死整个 Filter Bar；
20. 数据刷新不会导致 Control 抖动、失去 Focus 或产生装饰动画；
21. Narrow 下 Tabs、Filters、Segmented Control 仍保留正确语义；
22. Accessibility Role 与视觉 Shape 不混淆；
23. Keyboard 可以完整操作 Tabs、Filters 与 Segmented Controls；
24. Reduced Motion 下状态变化仍然立即、清晰、无长距离位移。

核心规则：

```text
Tab
≠ Filter
≠ Segmented Control

Same dataset + subset change
→ Filter

Stable peer information domain
→ Tab

Few mutually exclusive modes
→ Segmented Control

Context
≠ Filter

Draft Filter
≠ Applied Filter

Clear Filters
≠ Reset Page Context

Meaningful analytical state
→ restorable

Latest user intent
→ wins

Presentation mode change
→ preserves analytical context

Visual shape
≠ accessibility role
```

---

# 27. Drawer / Modal / Popover

Drawer、Modal 与 Popover 都属于 Overlay，但承担不同的交互职责。

正式关系：

```text
Drawer
→ detailed secondary context
→ preserves visible relationship to current page

Modal
→ blocking focused decision / short task

Popover
→ lightweight anchored contextual interaction
```

核心原则：

```text
Drawer
≠ Modal
≠ Popover

Detail context
→ Drawer

Blocking focused task
→ Modal

Anchored lightweight interaction
→ Popover

Deep analysis / complex workflow
→ Full Page
```

Overlay 不能因为视觉层级更高而突破 O3Pilot 的产品边界、安全边界或 Read-only Contract。

---

## 27.1 Overlay Principles

Overlay 用于临时提高当前任务的一部分信息或交互优先级，而不是替代稳定的信息架构。

正式原则：

```text
Overlay
→ temporary focused context

Overlay
≠ new application shell
≠ hidden page hierarchy
```

规则：

1. 先判断用户任务，再选择 Overlay 类型；
2. 不为了“高级感”把普通页面内容放进 Overlay；
3. 不使用 Overlay 隐藏应当常驻的重要业务状态；
4. Overlay 打开后，用户必须仍然理解当前实体、当前页面与当前任务之间的关系；
5. Overlay 关闭后，应尽量返回打开前的分析位置与上下文；
6. Overlay 中的操作继续遵守 Section 24 Forms 与 Section 25 Buttons；
7. 所有 Overlay 必须遵守 Section 14–16 的 Motion、Performance 与 Reduced Motion Contract；
8. Overlay 不能通过确认、按钮层级或视觉强调引入 Ozon Server State 写操作。

---

## 27.2 Container Decision Rules

正式决策顺序：

```text
Need detailed entity context
without leaving current page?
→ Drawer

Need a blocking decision
or focused short task?
→ Modal

Need lightweight controls / contextual information
anchored to a trigger?
→ Popover

Need deep analysis / complex workflow?
→ Full Page
```

典型映射：

```text
Order Detail
→ Drawer

Product Quick Detail
→ Drawer

Alert Detail
→ Drawer

Finance Accrual Detail
→ Drawer

Campaign Detail
→ Drawer

Data Quality Issue
→ Drawer

Delete local configuration confirmation
→ Modal

Short credential replacement form
→ Modal

Simple file import confirmation
→ Modal

Filter controls
→ Popover / responsive promoted container

Column Visibility
→ Popover

Metric Info
→ Popover / Detail Drawer when complex

Date Picker
→ Popover

Comparison Selector
→ Popover

Reporting Currency Selector
→ Popover
```

如果一个 Overlay 的内容持续增加，必须重新评估容器，而不是无限扩大当前 Overlay。

---

## 27.3 Detail Drawer

Detail Drawer 用于在保留当前页面视觉关系的前提下查看实体或问题的详细上下文。

正式关系：

```text
List / Table
↓
Entity Detail Drawer
↓
Deep Analysis when needed
↓
Full Page
```

Detail Drawer 适合：

- Order Detail；
- Product Quick Detail；
- Alert Detail；
- Finance Accrual Detail；
- Campaign Detail；
- Data Quality Issue；
- Source / Lineage 诊断；
- 其他需要保持列表或分析页上下文的次级详情。

Drawer 不是完整业务页面的缩小版本。

正式原则：

```text
Drawer
→ secondary detail context

Drawer
≠ mini page
```

如果用户需要长时间分析、多图表、多表格、多步骤任务或复杂导航，应提供完整页面。

---

## 27.4 Drawer Anatomy and Sizes

Desktop 只允许两种正式宽度：

```text
drawer-standard   480px
drawer-wide       640px
```

用途：

```text
480px Standard
→ normal entity detail
→ order quick detail
→ alert
→ data quality issue

640px Wide
→ multi-column detail
→ timeline
→ finance detail
→ complex source / lineage evidence
```

禁止 Feature 自行创造：

```text
512px
560px
600px
```

等中间宽度。

正式原则：

```text
Drawer width
→ follows content role
not arbitrary feature preference
```

标准 Drawer Anatomy：

```text
┌────────────────────────────────┐
│ Title / Identity           ×   │
│ Primary Identifier             │
│ Status when relevant           │
├────────────────────────────────┤
│                                │
│ Primary Detail                 │
│                                │
│ Supporting Evidence            │
│                                │
├────────────────────────────────┤
│ Origin / As-of / Lineage       │
│ when relevant                  │
└────────────────────────────────┘
```

Header 至少支持：

```text
Title
Primary Identifier
Status when relevant
Close
```

关键 Identifier：

- 不得截断到无法辨认；
- 必须保留完整原始值；
- 在产品允许时支持 Copy；
- 复制结果必须是完整值，而不是视觉截断文本。

Drawer 不要求固定 Tabs。

```text
Drawer
≠ mandatory tabs
```

只有真正存在稳定同级信息域时才使用 Tabs，并继续遵守 Section 26。

---

## 27.5 Drawer Navigation and Depth

默认只允许一层 Drawer。

正式规则：

```text
One active Drawer layer
→ default

Nested Drawer
→ prohibited by default
```

禁止：

```text
Drawer
↓
Drawer
↓
Drawer
```

如果当前 Drawer 内需要查看另一个深层实体，优先：

```text
replace current detail
```

或：

```text
open full page
```

必要时可以提供清晰的 Back / History Pattern，但不能无限叠加 Overlay。

深层详情应保持明确的实体 Identity 与导航关系。

---

## 27.6 Modal

Modal 用于需要暂时阻塞底层工作流的明确决策或短任务。

正式定义：

```text
Modal
→ temporarily blocks underlying workflow
```

适合：

- 明确确认；
- Destructive local confirmation；
- 短表单；
- Credential replacement；
- 简短导入确认；
- Restore / Replace 等需要用户明确决策的 O3Pilot 本地流程。

不适合：

- 查看订单详情；
- 长报表；
- 大型 Table；
- 大型 Chart；
- 复杂诊断；
- 多信息域分析。

正式原则：

```text
Modal
→ focused short task

Modal
≠ general detail container
```

---

## 27.7 Modal Complexity Boundary

如果 Modal 开始出现：

- 多个 Tabs；
- 大 Table；
- 大 Chart；
- 多阶段复杂工作流；
- 长篇 Source / Lineage；
- 大量滚动 Section；

必须重新判断是否应升级为 Drawer 或 Full Page。

正式关系：

```text
Task complexity grows
↓
Modal becomes long / highly scrollable / multi-domain
↓
Drawer or Full Page
```

核心规则：

```text
Modal
≠ mini application page
```

对于导入：

```text
Select File
→ Validate
→ Confirm
```

这类短流程可以使用 Modal。

但：

```text
Upload
→ Mapping
→ Validation
→ Conflict Resolution
→ Preview
→ Import Result
```

属于复杂 Workflow，应优先使用 Dedicated Page 或 Drawer Workflow。

正式原则：

```text
Short focused sequence
→ Modal may be used

Complex workflow
→ Page
```

---

## 27.8 Popover

Popover 用于与某个 Trigger 明确关联的轻量上下文信息或交互。

正式关系：

```text
Trigger
↓
Anchored Popover
↓
Lightweight contextual content
```

Popover：

```text
→ anchored to trigger
→ no page-level blocking
→ no scrim
```

典型用途：

- Filter；
- Column Visibility；
- Metric Info；
- Date Picker；
- Comparison Selector；
- Reporting Currency Selector；
- 简单 Contextual Action；
- 小型 Select-like Composite Control。

Popover 必须明确属于触发它的 Control，不应在页面中形成新的独立导航层级。

---

## 27.9 Popover Complexity Boundary

Popover 应保持：

```text
short
contextual
locally actionable
```

如果内容开始包含：

- 大量 Form Fields；
- 长错误诊断；
- 大型数据表；
- Entity Detail；
- 长帮助文章；
- 复杂多阶段操作；

应升级为：

```text
Drawer
Modal
Full Page
```

正式原则：

```text
Popover complexity grows
→ promote container
```

Popover 与 Tooltip 必须区分：

```text
Tooltip
→ brief explanatory text
→ no meaningful interactive workflow

Popover
→ interactive contextual surface
```

如果 Metric Info 仅是非常短的辅助说明，可以使用 Tooltip；如果需要 Definition、Time Basis、Coverage、Origin 等多项信息，应使用 Metric Detail Popover 或更合适的 Detail Surface。

---

## 27.10 Overlay Hierarchy and Scrim

Overlay 层级直接继承 Section 10：

```text
Popover / Dropdown
→ Elevation 1

Drawer / Command Palette
→ Elevation 2

Modal
→ Elevation 3
```

正式原则：

```text
Overlay hierarchy
→ semantic
not arbitrary z-index competition
```

禁止 Feature 通过：

```text
z-index: 999999
```

等方式自行竞争层级。

Scrim 规则：

```text
Popover
→ no scrim

Drawer
→ subtle scrim by default

Modal
→ scrim required
```

Drawer 打开后：

```text
underlying page
→ remains visually visible
→ interaction becomes inert
```

即：

```text
Visible page context
≠ simultaneous background interaction
```

如果未来某种高级分析需要真正 Side-by-side 同时操作，应设计 Dedicated Split View，而不是让普通 Drawer 同时承担 Overlay 与双栏工作区两套语义。

Scrim 优先使用简单透明度，不通过大范围 Blur / Backdrop Filter 制造层级。

```text
simple scrim opacity
>
animated backdrop blur
```

---

## 27.11 Focus and Dismissal

Drawer / Modal 打开时：

```text
Trigger
↓
Overlay opens
↓
Focus enters meaningful first control / heading
```

打开期间：

```text
Focus
→ remains inside active overlay
```

关闭后：

```text
Focus
→ returns to logical trigger
```

如果原 Trigger 已不存在，Focus 应返回合理的附近上下文，而不是跳到页面顶部。

Popover 的 Focus 行为按内容决定：

- 纯阅读型 Popover 可以保留 Focus 在 Trigger；
- 交互型 Popover 应进入可操作内容；
- 关闭后恢复到触发上下文。

默认关闭规则：

```text
Escape
→ closes topmost dismissible overlay
```

多 Overlay 情况下，只关闭当前最高层可关闭 Overlay，不能一次关闭所有层级。

Outside Click / Scrim Click：

```text
Popover
→ outside click normally closes

Drawer
→ scrim click may close
   when no material unsaved state

Modal
→ dismissal follows task risk
```

Critical Confirmation、Restore、有未保存关键输入等流程不应通过误触 Scrim 静默关闭。

正式原则：

```text
Dismiss behavior
→ follows task risk
```

Close Control：

```text
Drawer / Modal Header
→ Close at top-right
```

使用 Lucide `X`，并满足：

- Accessible Name；
- Visible Focus；
- Touch / Narrow Target ≥ 44px；
- Icon-only 规则继承 Section 13 与 Section 25。

---

## 27.12 Unsaved State and Actions

当 Overlay 中存在有价值的未保存更改：

```text
Overlay contains material unsaved changes
+
user attempts close
→ protect against accidental loss when appropriate
```

触发路径包括：

- Close Button；
- Escape；
- Scrim Click；
- Route Change。

但不要所有 Overlay 都机械弹出关闭确认。

正式原则：

```text
No meaningful unsaved state
→ close normally

Material unsaved state
→ protect when loss matters
```

Modal Footer 直接继承 Section 25：

```text
[Cancel] [Primary Completion]
```

Destructive Confirmation：

```text
[Cancel] [Destructive Action]
```

CN / EN / RU 都使用 LTR，因此 Desktop Action Footer 保持统一靠右排列。

Detail Drawer 不强制拥有 Footer。

```text
Detail Drawer
→ no mandatory footer
```

只有确实存在 O3Pilot 本地允许操作时才显示 Action，例如：

- Copy；
- Export；
- Acknowledge local alert；
- 其他 O3Pilot-owned local action。

不得为了“Drawer 完整”常驻无意义的：

```text
Cancel / Confirm
```

Overlay 中的确认不能突破产品边界：

```text
Modal confirmation
≠ permission to mutate Ozon
```

---

## 27.13 Motion and Reduced Motion

Overlay Motion 直接复用 Section 14 正式 Token：

```text
Popover
→ motion-standard
→ 180ms
→ opacity + very subtle transform

Drawer
→ motion-medium
→ 220ms
→ transform + scrim opacity

Modal
→ motion-medium
→ 220ms
→ opacity + minimal spatial treatment
```

不允许 Feature 自行引入：

```text
250ms
300ms
350ms
```

等新的 Overlay 时长。

核心规则：

```text
User intent
>
animation completion
```

关闭、重新打开或快速切换 Overlay 时不能排动画队列。

Reduced Motion 直接继承 Section 16：

```text
Drawer
→ final position immediately
→ optional short opacity only

Modal
→ final position immediately
→ optional short opacity only

Popover
→ no scale / displacement
→ optional short opacity only
```

Reduced Motion 不创建另一套功能语义，只减少空间移动。

---

## 27.14 Responsive

### Drawer

Desktop：

```text
480px Standard
640px Wide
```

Narrow：

```text
→ full-width detail surface
```

不能把固定 480px Drawer 挤进 390px Viewport。

### Modal

Narrow：

```text
→ width follows viewport gutters
```

如果 Modal 在 Narrow 上接近完整页面，并承载复杂内容，应重新判断是否应该改为 Full Page / Drawer。

### Popover

空间不足时允许：

```text
reposition
resize within viewport
```

复杂 Filter Popover 在 Narrow 可以升级为 Drawer / Sheet-like Surface。

正式原则：

```text
Responsive container may change
while interaction semantics remain
```

即：容器视觉形态可以根据 Viewport 调整，但任务语义、状态与可访问性不能改变。

---

## 27.15 Route / State Preservation

Detail Overlay 必须尽量保留打开前的分析状态。

例如：

```text
Open Detail
→ inspect
→ close
→ return to same analytical position
```

至少保持适用的：

- Scroll Position；
- Search；
- Sort；
- Filters；
- Pagination / Cursor Position；
- Date Context；
- Selected Tab / Mode；
- Shop Context；
- Reporting Currency。

重要 Entity Detail 可以进入可恢复的 Route / Query State，例如：

```text
Orders
+
filters
+
posting detail
```

使 Refresh / Back / Forward 后仍能理解当前状态。

但短暂 Overlay Visibility 不进入 Route State，例如：

- Tooltip；
- Filter Popover Open；
- More Menu；
- Date Picker Open。

正式原则：

```text
Meaningful detail state
→ may be restorable

Transient overlay visibility
→ not route state
```

---

## 27.16 Accessibility

Drawer / Modal 必须满足：

1. 使用与行为匹配的 Dialog / Modal Semantic；
2. Overlay 有明确 Accessible Name；
3. Screen Reader 能感知 Overlay 已打开；
4. Focus 进入 Overlay 后不能逃逸到底层 Inert 内容；
5. 关闭后 Focus 返回合理触发上下文；
6. `Escape` 对 Dismissible Overlay 可用；
7. Close Control 有 Accessible Name；
8. Focus Ring 不被 Overlay Surface、Sticky Header 或 Scrim 裁切；
9. Background 在 Modal / Drawer 激活时按 Contract Inert；
10. 200% Zoom 下内容、Action 与关闭入口仍可访问。

Popover 必须满足：

1. Trigger 表达 Expanded State；
2. Trigger 与 Popup 关系可被辅助技术理解；
3. Keyboard 可打开与关闭；
4. 不能依赖 Hover 作为唯一入口；
5. 交互型 Popover 中所有 Control 可正常键盘操作；
6. 关闭后 Focus 恢复合理位置。

正式原则：

```text
Visual overlay
≠ div with high z-index
```

Overlay 的视觉位置不能替代正式的语义、Focus 与键盘行为。

---

## 27.17 Overlay QA

正式 QA 至少覆盖：

```text
Drawer Standard 480px
Drawer Wide 640px
Modal
Popover
Tooltip vs Popover boundary
Entity detail
Short form
Destructive local confirmation
Simple import confirmation
Filter Popover
Metric Info
Date Picker
Column Visibility
```

交互状态至少覆盖：

```text
Open
Close
Escape
Outside Click
Scrim Click
Focus Enter
Focus Trap
Focus Return
Unsaved State
Loading
Error
Rapid reopen
Back / Forward
```

响应式至少覆盖：

```text
1440px
1200px
1024px
768px
390px
```

同时验证：

```text
Mouse
Keyboard
Touch
Light
Dark
Reduced Motion
120Hz
60Hz
200% Zoom
CN
EN
RU
```

必须确认：

1. Drawer / Modal / Popover 使用正确容器语义；
2. 不出现默认 Nested Drawer；
3. Modal 不承担复杂页面工作流；
4. Popover 不无限扩张；
5. Drawer 打开时底层页面可见但不可交互；
6. Focus 不泄漏到底层页面；
7. Escape 只关闭最高层可关闭 Overlay；
8. Unsaved State 不被静默丢失；
9. Close 后返回原分析位置；
10. Overlay Motion 不阻塞用户输入；
11. Reduced Motion 下不保留大范围空间移动；
12. Narrow 下容器变化不改变业务语义；
13. Modal Confirmation 不产生 Ozon Server State 写能力；
14. Long CN / EN / RU 文案不破坏关闭入口或 Footer；
15. Overlay 不使用任意 z-index 或大范围 Blur 竞争层级。

核心 Contract：

```text
Drawer
≠ Modal
≠ Popover

Detail context
→ Drawer

Blocking focused task
→ Modal

Anchored lightweight interaction
→ Popover

Deep analysis / complex workflow
→ Full Page

drawer-standard
→ 480px

drawer-wide
→ 640px

Nested Drawer
→ prohibited by default

Modal
≠ mini application page

Popover complexity grows
→ promote container

Overlay hierarchy
→ semantic

Drawer open
→ background visually preserved
→ interaction inert

Escape
→ closes topmost dismissible overlay

Dismiss behavior
→ follows task risk

Open Detail
→ inspect
→ close
→ return to same analytical position
```

---

# 28. Toast / Notification

本章定义 O3Pilot 的短暂操作反馈、持久消息与业务预警之间的正式呈现边界。

核心关系：

```text
Toast
≠ Notification
≠ Business Alert
≠ Inline Status
```

其中：

```text
Toast
→ transient feedback

Notification / Notice
→ persistent or discoverable message presentation

Business Alert
→ domain object defined by O3Pilot alert capability

Inline Status
→ current page / component state
```

`Notification` 仅表示消息的投递或呈现方式，不在本章创建新的业务对象、导航节点或第二套预警体系。

## 28.1 Feedback Principles

O3Pilot 的反馈体系遵循：

```text
Transient Feedback
→ Toast

Contextual Persistent Message
→ Inline Notice / Status

Business Risk / Attention
→ Alert
```

正式原则：

```text
Toast
→ feedback

not
→ primary state container
```

如果用户错过 Toast 后就无法理解页面当前真实状态，说明该信息不应只使用 Toast。

同时：

```text
Visible result
>
redundant success Toast
```

界面自身已经明确表达结果的操作，不默认再叠加成功 Toast。

例如以下操作通常不需要 Toast：

- Tab 切换；
- Filter 生效；
- Drawer 打开；
- Segmented Control 切换；
- Appearance 改变且结果立即可见。

## 28.2 Toast vs Notification vs Alert

正式语义：

### Toast

Toast 是：

```text
transient
non-blocking
recent-action feedback
```

典型用途：

- 设置已保存；
- 文件已下载；
- 本地记录已更新；
- 轻量操作失败；
- 在其他反馈不足时确认 Copy / Export / Retry 等操作结果。

### Notification / Notice

Notification / Notice 是用户之后仍可能需要看到、理解或据此行动的消息呈现。

例如：

- 存在数据源缺口；
- Backup 尚未配置；
- Credential 当前不可用；
- 长任务存在需要用户查看的结果。

正式关系：

```text
Future action depends on message
→ persistent / discoverable
```

本章中的 Notification Presentation：

```text
≠ new navigation destination
≠ Notification Center requirement
```

v1 不因为本章而新增独立 Notification Center。

### Business Alert

Business Alert 是风险、异常或经营关注项的正式 Domain Object。

例如：

```text
库存风险
→ Alert

“3 个新预警已生成”
→ Notification
```

正式规则：

```text
Alert
≠ Notification
```

Alert 的生命周期、Ack、状态和页面由正式 Alert Contract 定义，不由 Toast / Notification Presentation 重定义。

## 28.3 Toast Use Cases

Toast 适合：

```text
recent action
+
result is useful to confirm
+
message is non-critical
+
missing the Toast does not hide persistent business truth
```

典型：

```text
设置已保存
文件已下载
重新计算已开始
本地记录已更新
```

成功反馈不是所有 Action 的默认后续步骤。

正式：

```text
Self-evident state change
→ no Toast required
```

对于 Copy，若 `Copy → Check` 已经形成充分反馈，则不默认再叠加：

```text
Toast “复制成功”
```

正式：

```text
One sufficient feedback channel
→ preferred
```

对于 Explicit Save，如果 Form 内已经持续显示：

```text
Saving
Saved
Save Failed
```

也不默认同时再显示重复 Toast。

## 28.4 When Toast Is Not Enough

Toast 不适合作为以下状态的唯一载体：

- 重要 Data Error；
- `STALE / GAP / FAILED` 等持久 Data State；
- Security-critical State；
- Session Revoked / Authentication Expired；
- Backup / Restore 关键失败；
- 数据库或持久化完整性问题；
- 关键 Import 部分失败；
- 需要用户后续持续关注的问题。

正式：

```text
Transient notification
≠ persistent data truth
```

例如数据刷新失败且仍有 Last Known Value：

```text
Metric / Table
→ keep Last Known Data
→ show STALE / FAILED state

Global Data Status
→ reflect source problem

Toast
→ optional immediate notice only
```

不能：

```text
Toast “同步失败”
↓
Toast disappears
↓
page still looks VALID
```

Security / Session：

```text
Security-critical state
→ persistent explicit surface
```

Toast 最多作为补充。

## 28.5 Message Content

Toast 文案必须描述真实动作或结果。

禁止笼统文案：

```text
成功
操作成功
已完成
失败
发生错误
```

优先：

```text
设置已保存
订单号已复制
导入文件解析失败
无法保存成本数据
```

如果系统已经知道可修复原因，应直接说明：

```text
无法保存成本数据：采购币种不能为空
```

但：

```text
Toast
≠ validation surface
```

Field Validation 继续由 Form 在字段附近表达；Toast 不代替用户定位具体输入错误。

文案需要满足：

- CN / EN / RU 均可自然表达；
- 不暴露 Credential / Secret；
- 不把内部异常堆栈直接作为用户文案；
- 不用模糊“未知错误”掩盖已知原因；
- 不把 Warning、Error、Business Alert 语义混成一个红色 Toast。

## 28.6 Actions and Undo

Toast 可以包含：

```text
0 or 1 action
```

允许的简单 Action 示例：

```text
撤销
重试
查看
```

正式：

```text
Toast actions
→ 0 or 1
```

复杂决策、多个 Action、长表单或风险确认应升级为：

```text
Inline Notice
Drawer
Modal
Full Page
```

### Undo

Undo 只允许用于确实满足以下条件的 O3Pilot 本地操作：

```text
local
reversible
safe
```

例如某些可原子恢复的本地 Presentation State。

禁止使用 Undo 模式包装：

- Restore；
- 删除关键本地业务数据；
- 不可逆操作；
- 任何 Ozon Server State mutation。

正式：

```text
Undo
→ local reversible safe action only
```

## 28.7 Duration and Dismissal

O3Pilot 不在 DESIGN 中规定所有 Toast 必须固定为 `3000ms / 5000ms / 7000ms`。

正式原则：

```text
Reading time
→ follows content and interaction need
```

分类：

```text
Brief informational Toast
→ auto-dismiss allowed

Actionable Toast
→ remains long enough to act

Critical information
→ not Toast-only
```

如果 Toast 包含 Action，用户 Hover 或 Keyboard Focus 时：

```text
Auto-dismiss timer
→ pauses
```

离开后可以继续。

Dismiss Control 不是所有短暂无 Action Toast 的必需元素。

较长或 Actionable Toast 可以提供 Close；若提供：

- 使用 Lucide `X`；
- 有 Accessible Name；
- Keyboard 可操作；
- Touch / Narrow Target 满足全局规则。

正式：

```text
Dismissal
≠ Resolution
```

关闭一条 Notice 只改变 Presentation State，不自动改变对应业务对象的 Domain State。

## 28.8 Stacking and Aggregation

Toast Stack 必须受控。

正式：

```text
Visible Toast Stack
→ small and bounded
```

具体最大可见数量由实现根据 Viewport 决定，但不允许 Toast 持续堆叠遮挡主要业务内容。

同类高频消息：

```text
Repeated equivalent messages
→ coalesce / update
```

例如连续多次 Copy 不产生多个重复“已复制” Toast。

后台批量事件：

```text
Background batch events
→ aggregate
```

例如：

```text
12 个数据集已更新
```

但普通正常后台运行默认保持：

```text
Background normal operation
→ Quiet
```

第 19 章已有 Global Data Status 时，不需要再用成功 Toast 持续播报常规 Sync。

## 28.9 Persistent Notifications / Notices

当信息在 Toast 消失之后仍然影响后续判断或操作，应使用持久 / 可发现 Message Pattern。

例如：

```text
存在 1 个数据源缺口
Backup 尚未配置
Credential 当前验证失败
Import 有 3 条记录未匹配
```

根据语义可以落在：

- Page-level Inline Notice；
- Section-level Status；
- Global Data Status；
- Settings Status；
- Alert；
- Job / Import Result。

正式：

```text
Future action depends on message
→ persistent / discoverable
```

Notification Presentation 本身不创建独立业务生命周期。

## 28.10 Data / Security / Session States

### Data

Data 状态继续服从第 17–19 章：

```text
Data State
→ persistent where business truth requires it
```

Toast 只能提供即时补充反馈。

### Security

对于 Credential、Security Boundary、敏感配置错误：

- 不暴露 Secret；
- 不仅依赖 Toast；
- 错误原因在正式安全允许范围内持久可见；
- 不把完整凭据回显进 Message。

### Session

```text
Session Revoked / Authentication Expired
→ dedicated persistent flow
```

不能依赖自动消失 Toast。

## 28.11 Alert Delivery and Acknowledgement

Business Alert 可以通过 Notification / Toast 告知“存在新预警”，但 Delivery 不改变 Alert Domain State。

正式：

```text
Toast shown
≠ Alert acknowledged
≠ Alert resolved
```

Alert Ack / Resolution 只能通过正式的 O3Pilot Local State Action 改变。

同样：

```text
Notification dismissed
≠ Alert dismissed
≠ Alert resolved
```

除非 Alert Contract 明确定义对应本地动作。

## 28.12 Position and Layout

Desktop 默认 Toast Region：

```text
Top-right
below / clear of global chrome
```

必须避免遮挡：

- Global Context Bar 关键 Control；
- Global Search；
- Modal / Drawer Primary Action；
- Page-level Critical Notice；
- Table / Chart 核心读数。

Narrow：

```text
Top region
within viewport gutters
```

Toast：

```text
→ overlay
→ no document reflow
```

不能因为出现 Toast 让 Page Content 整体向下跳动。

Toast Layer 不得与 Modal / Drawer 形成任意 `z-index` 竞争；Overlay 层级继续遵守第 10、27 章的正式语义。

## 28.13 Motion and Reduced Motion

Toast Motion 直接复用正式 Token：

```text
Enter / Exit
→ motion-standard
→ 180ms
```

仅使用：

- opacity；
- 必要时非常小的局部 transform。

禁止：

- 从屏幕外长距离飞入；
- bounce；
- spring；
- scale pop；
- 多条 Toast 连续编排表演。

正式：

```text
Toast
→ appears
not performs
```

Reduced Motion：

```text
Toast
→ appear in final position
→ optional short opacity only
```

并且：

```text
Toast reading duration
≠ motion duration
```

Reduced Motion 不缩短可阅读时间。

## 28.14 Long-running Tasks

长任务的过程状态不应依赖长期驻留 Toast。

正式：

```text
Long-running task
→ persistent task status

Toast
→ completion / lightweight transition feedback
```

例如：

```text
Import 47%
```

应显示在 Import / Job Surface，而不是让 Toast 在页面角落持续数分钟。

进度优先：

```text
Control-local / task-local progress
→ near the task
```

完成后如有价值，可以使用：

```text
导入完成
```

作为短暂 Toast。

同样：

```text
Control-local progress
→ near control

Completion feedback
→ Toast when useful
```

不默认同时产生“正在保存…”Toast 与“保存成功”Toast 两次播报。

## 28.15 Accessibility

Toast / Notification 必须满足：

1. 普通成功 / 信息使用非打断式 Announcement；
2. Announcement Priority 由实际紧迫性决定；
3. 不把所有 Toast 都设为 Assertive；
4. Security / Session Critical State 本来就不能只靠 Live Region；
5. Toast 出现时不自动抢走 Keyboard Focus；
6. Toast Action Keyboard 可达；
7. Toast Action 获得 Hover / Focus 时自动消失计时暂停；
8. Action / Close 有 Visible Focus；
9. Action 有明确 Accessible Name；
10. Toast 消失不能让当前 Focus 无意义丢失；
11. 颜色不是 Success / Error / Warning 的唯一表达；
12. CN / EN / RU 长文案可换行且不遮挡关键 UI；
13. 200% Zoom 下 Toast 内容和 Action 可读；
14. Reduced Motion 不影响消息理解和阅读时间。

正式：

```text
Toast announcement
≠ focus steal
```

## 28.16 Toast / Notification QA

正式 QA 至少覆盖：

```text
Success feedback
Lightweight failure
Copy feedback
Save feedback
Export feedback
Retry action
Undo-safe local action
Duplicate Toast coalescing
Batch event aggregation
Long-running task completion
Persistent data error
Security-critical state
Session revoked
Alert delivery without Ack
Dismissal vs Resolution
Desktop top-right region
Narrow layout
Modal / Drawer coexistence
Keyboard
Screen Reader announcement
200% Zoom
CN
EN
RU
Light
Dark
Reduced Motion
```

验收必须确认：

1. 自解释的状态变化不会机械弹 Success Toast；
2. `Copy → Check` 足够时不叠加重复 Toast；
3. Field Validation 不被 Toast 取代；
4. `STALE / GAP / FAILED` 不会因 Toast 消失而失去持久表达；
5. Security / Session 状态存在明确持久 Surface；
6. Alert Delivery 不会自动 Ack / Resolve；
7. 同类高频消息不会形成 Toast Storm；
8. Actionable Toast 在 Focus 时不会自动消失；
9. Toast 不导致 Page Reflow；
10. Toast 不遮挡关键 Overlay Action；
11. Long-running Task 使用 Persistent Task Status；
12. Toast Motion 使用正式 Token；
13. Reduced Motion 不降低可读性；
14. Toast 出现不会抢走当前 Keyboard Focus。

核心规则：

```text
Toast
≠ Notification
≠ Business Alert

Toast
→ transient feedback

Persistent business truth
→ persistent surface

Alert
→ domain object

Notification
→ delivery / presentation

Self-evident state change
→ no Toast required

Toast
≠ validation surface

Transient notification
≠ persistent data truth

Security-critical state
→ persistent explicit surface

Toast actions
→ 0 or 1

Dismissal
≠ Resolution

Toast shown
≠ Alert acknowledged

Background normal operation
→ Quiet

One sufficient feedback channel
→ preferred

Toast announcement
≠ focus steal
```

---

# 29. Loading

Loading 只描述“系统正在取得、计算、提交或刷新结果”的过程状态，不得替代正式的数据状态、错误状态或持久任务状态。

正式原则：

```text
Initial Load
≠ Refresh
≠ Local Action Progress
≠ Background Job
≠ Lazy / Deferred Load
```

核心优先级保持为：

```text
Existing data + refreshing state
>
Skeleton
>
Spinner
```

只要已有仍可使用的数据，优先保留数据和阅读上下文，而不是重新把整个区域退回 Initial Loading。

---

## 29.1 Loading Principles

Loading 设计必须满足：

```text
Usable existing content
→ preserve it

Predictable initial geometry
→ Skeleton may be used

No predictable content geometry
→ quiet loading indicator / text

Known progress
→ determinate progress

Long-running work
→ persistent task status
```

正式关系：

```text
Keep useful content
>
replace it with loading chrome
```

Loading 不得：

- 把已知业务值清空为占位符；
- 把 Refresh 伪装成首次加载；
- 用假百分比制造进度感；
- 因单个局部请求阻塞整个页面；
- 让正常后台同步持续抢占用户注意力；
- 在失败后无限停留在 Loading；
- 通过动画暗示业务数据本身正在“变化”。

Loading 的目标是让用户理解：

```text
What is waiting?
What remains usable?
Is progress known?
Can I continue working?
What happens if it fails?
```

---

## 29.2 Loading Types

O3Pilot 正式区分以下 Loading 类型。

### Initial Load

当前区域尚无可展示结果，正在取得首次内容。

例如：

```text
首次进入订单页
首次加载某个 Detail Drawer
首次取得某张 Chart 数据
```

### Refresh

当前已经存在可用内容，系统正在尝试取得更新版本。

```text
Existing Data
+
Refreshing State
```

### Local Action Progress

用户刚触发一个局部操作，进度应靠近触发上下文。

例如：

```text
Save
Retry
Recalculate
Validate Credential
Export
```

### Background Job

任务可以持续较长时间，并可能脱离当前页面继续执行。

```text
Job Created
↓
Progress / State
↓
Result
```

其状态语义服从 `ARCHITECTURE.md` 的 Persistent Job System；`DESIGN.md` 不重新定义后端 Job 状态机。

### Lazy / Deferred Load

非首要内容在主内容可用后继续按需加载。

例如：

```text
Secondary Detail
Additional Lineage
Low-priority supporting data
```

正式原则：

```text
Primary content availability
≠ every secondary request completed
```

---

## 29.3 Existing Data and Refresh

如果已有旧数据：

```text
保留旧数据
+
显示 Refreshing
```

不要让整个页面白屏，也不要重新播放 Initial Skeleton。

例如：

```text
库存
382

正在更新…
```

而不是：

```text
库存
██████
```

当旧值仍然符合上游 Contract 的可展示条件时，用户应继续能够：

- 阅读；
- 滚动；
- 复制；
- 切换 Detail；
- 修改兼容的筛选条件；
- 查看其他不受影响的区域。

正式关系：

```text
Refresh
≠ Initial Load
```

```text
Preserved content
≠ pretend refresh succeeded
```

刷新中保留的内容仍属于刷新前的已知数据；如果最终刷新失败，必须转入第 17、19、30 章定义的 `STALE / FAILED / Error` 等正式状态，并在存在 Last Known Data 时保留其可识别性。

---

## 29.4 Initial Loading

Initial Loading 只在当前区域确实没有可展示内容时使用。

优先顺序：

```text
Predictable final geometry
→ Skeleton

Simple unknown wait
→ concise loading indicator / text

Known measurable work
→ determinate progress
```

Initial Loading 应尽可能保持最终布局稳定，包括：

- Page Header；
- Section 位置；
- Table Header；
- Chart Container；
- Drawer Header；
- Form Label / Control Geometry。

不要为了 Loading 隐藏已经可以确定的页面结构。

正式原则：

```text
Loading state
→ preserve layout intent
```

---

## 29.5 Skeleton

Skeleton 只用于最终结构高度可预测的区域。

适合：

```text
Metric placeholder
Table rows
Known detail fields
Stable content blocks
```

不适合：

```text
Unknown free-form content
Complex variable-length analysis
Arbitrary decorative placeholder blocks
```

Skeleton 必须近似最终几何，避免内容到达后产生明显 Layout Shift。

正式原则：

```text
Skeleton
→ communicates expected structure

not
→ decorative gray rectangles
```

Skeleton 不得模拟不存在的信息架构。

例如：

```text
Actual page has 3 KPI
→ do not render 8 KPI skeletons
```

```text
Unknown dynamic table columns
→ do not invent many placeholder columns
```

Skeleton 只表示“这里将出现已知结构的内容”，不表示任何业务值、数量或状态。

---

## 29.6 Indeterminate Loading

Indeterminate Loading 适用于系统正在工作，但当前无法提供可信完成比例的短暂等待。

```text
Spinner
→ indeterminate progress
```

但 Spinner 不是 Loading 的默认视觉装饰。

如果文字已经足够说明状态，可以使用：

```text
正在加载…
正在更新…
正在验证…
正在重新计算…
```

而不必叠加持续旋转动画。

禁止长时间只显示：

```text
Loading…
```

而不说明当前区域正在等待什么。

正式原则：

```text
Loading comprehension
>
continuous animation
```

---

## 29.7 Determinate Progress

如果系统拥有可信的已完成工作量，必须优先显示真实 Progress。

```text
Known progress
→ determinate

Unknown progress
→ indeterminate
```

例如：

```text
正在解析文件
47%
```

或：

```text
已处理 47 / 100 条
```

前提是该数值来自真实任务状态。

禁止人为制造：

```text
0%
20%
60%
90%
99%
```

来模拟“看起来正在推进”的进度。

正式原则：

```text
Unknown Progress
≠ Fake Percentage
```

如果只能知道阶段，可以显示真实阶段：

```text
验证文件
↓
解析数据
↓
写入结构化结果
```

但阶段也必须来自真实工作流，而不是视觉动画脚本。

---

## 29.8 Local Action Progress

局部操作的进度应尽量留在触发操作附近。

例如：

```text
[保存设置]
↓
[正在保存…]
```

```text
重新计算利润
↓
相关结果区域显示正在重新计算
```

而不是：

```text
one local action
→ entire page blocked
```

正式原则：

```text
Control-local progress
→ near the control
```

执行中必须：

- 防止无意义重复提交；
- 保留用户对当前动作的理解；
- 不把整个页面冻结为不可操作状态；
- 只在真实成功后显示成功；
- 失败后进入可恢复状态。

具体 Button / Form 状态继续服从第 24、25 章。

---

## 29.9 Query / Data Refresh

Search、Filter、Sort、Date Range 等分析 Query 变化时：

```text
Input
→ updates immediately

Result Region
→ may update asynchronously
```

Query 请求不能反向阻塞输入响应。

正式原则：

```text
Input responsiveness
>
query completion
```

如果连续发生多个 Query：

```text
Latest user intent
→ wins
```

旧请求晚返回时，不能覆盖已经更新后的 Query State。

Query 更新期间：

- Filter / Search Control 通常保持可操作；
- 当前结果可在合适情况下继续保留；
- Pagination / Sort / Filter 仍遵守第 23、26 章的一致 Query Contract；
- 不因每一次字符输入重置整个 Page Shell。

如果当前结果与新 Query 已不再具有可解释关系，可以对 Result Region 使用明确 Updating 状态，但仍不应影响其他页面区域。

---

## 29.10 Tables and Charts

### Table

Initial Load：

```text
Table Header
→ retained when known

Rows
→ geometry-matched skeleton / loading state
```

Refresh：

```text
Existing Rows
→ retained

Refresh status
→ subtle and discoverable
```

禁止 Refresh 时清空所有 Rows 后重新播放 Skeleton。

### Chart

Initial Load：

```text
Chart Container Geometry
→ retained
```

Refresh：

```text
Existing Series
→ retained when still valid for display

Updating State
→ quiet
```

数据更新默认直接重绘，不重新播放 Chart Entrance Animation。

正式原则：

```text
Refresh
≠ replay entrance
```

Missing / Gap / Stale 等业务状态继续服从第 17、19、22 章，不能由 Loading 表现替代。

---

## 29.11 Persistent Jobs

长任务不得依赖当前页面是否持续打开来表达任务生命期。

正式关系：

```text
Page visibility
≠ Job lifetime
```

当架构层已经创建 Persistent Job：

```text
Job Created
↓
Progress / State
↓
Result
```

UI 应能够表达：

- 任务已创建；
- 当前状态；
- 已知 Progress；
- 最后更新时间；
- 成功结果；
- 失败原因；
- 可恢复或重试能力（如果正式 Contract 支持）。

用户离开页面不得被误导为 Job 自动取消。

如果任务可在后台继续执行，返回相关页面后应能够恢复任务上下文，而不是重新从零播放一个假 Loading。

正常后台同步如果没有需要用户行动的状态，应遵守：

```text
Background normal operation
→ Quiet
```

不得制造持续的全局 Loading 噪音。

---

## 29.12 Loading Failure and State Transition

Loading 必须最终转换到明确结果：

```text
Loading
→ Success
→ Empty
→ Unavailable
→ Partial / Stale
→ Error
```

具体可用状态由上游 Contract 和第 17、30 章决定。

禁止：

```text
Request failed
→ spinner keeps spinning forever
```

正式原则：

```text
Loading failure
→ explicit state
not infinite loading
```

如果存在 Last Known Data：

```text
Failure
+
Last Known Data
→ keep data
→ show stale / failed context
```

如果不存在任何可展示结果：

```text
Failure
→ explicit Error / Unavailable state
```

Loading 本身不承担 Error Copy，也不能把失败显示成 Empty。

---

## 29.13 Motion and Reduced Motion

Loading Motion 必须服从第 14–16 章。

普通 Motion 模式下：

- Skeleton 不需要明显位移动效；
- Spinner 只在确有必要时连续运动；
- Progress 的数值更新不使用装饰性 Count-up；
- Refresh 不重新播放内容 Entrance；
- Loading Indicator 不使用 Bounce / Spring；
- 不使用 `transition: all`。

Reduced Motion：

```text
Skeleton
→ static
→ no shimmer
```

```text
Continuous loader
→ static indicator + text when possible
```

```text
Determinate progress
→ preserve accurate progress
→ no decorative travel
```

正式原则：

```text
Reduced Motion
→ no shimmer / continuous decorative motion
```

Reduced Motion 不能减少用户对真实任务状态的理解，也不能让任务更快自动消失。

---

## 29.14 Accessibility

Loading State 必须可被辅助技术理解，但不得成为高频干扰源。

要求：

1. 需要时使用正确的 Busy / Progress 语义；
2. 普通轻量 Refresh 不反复通过 Assertive Announcement 打断用户；
3. Determinate Progress 应暴露真实 Progress；
4. Indeterminate Progress 应表达正在处理的任务；
5. Loading 不自动抢走 Focus；
6. Loading 完成后不无意义把 Focus 移到 Page Top；
7. Query Refresh 保留用户当前输入与操作 Focus；
8. Local Action Loading 防止重复提交，但 Focus 不被无意义移走；
9. Reduced Motion 下仍然能够理解任务状态；
10. Loading 最终进入 Error / Empty / Unavailable 时，状态变化必须可被发现。

正式原则：

```text
Loading announcement
≠ focus steal
```

大量后台 Job 更新也不能逐条通过 Live Region 制造通知风暴。

---

## 29.15 Loading QA

正式 QA 至少覆盖：

```text
Initial Load
Refresh with existing data
Refresh failure with last known data
No existing data + failure
Table initial load
Table refresh
Chart initial load
Chart refresh
Local Save / Submit
Credential validation
Filter / Search rapid changes
Out-of-order requests
Known progress
Unknown progress
Persistent Job
Background quiet sync
Lazy detail load
Reduced Motion
Keyboard
Screen Reader
120Hz
60Hz
Desktop
Compact
Narrow
CN / EN / RU
```

必须验证：

1. `Initial Load ≠ Refresh ≠ Background Job`；
2. Existing Data Refresh 不清空旧值；
3. Refresh 不重新播放 Initial Skeleton；
4. Skeleton 只用于可预测结构；
5. Skeleton Geometry 与最终内容基本一致；
6. Unknown Progress 不伪造 Percentage；
7. Known Progress 使用真实数据；
8. 局部 Loading 不冻结无关区域；
9. Query 输入在请求期间保持响应；
10. 最新 Query 不会被旧请求结果覆盖；
11. Table / Chart Refresh 保持阅读位置和上下文；
12. Persistent Job 不依赖当前页面是否打开；
13. Failure 不会无限 Loading；
14. Last Known Data 在失败时仍可识别为非当前结果；
15. Reduced Motion 无 Shimmer / Decorative Continuous Motion；
16. Loading 不无意义抢夺 Focus；
17. Slow Request、Fast Request 与瞬时结果都不会产生明显闪烁；
18. 390px 下 Loading 状态不会造成布局溢出或隐藏任务语义。

核心验收关系：

```text
Initial Load
≠ Refresh
≠ Background Job

Usable existing content
→ preserve it

Refresh
≠ replay initial loading

Skeleton
→ predictable structure only

Unknown Progress
≠ Fake Percentage

Known Progress
→ determinate

Control-local progress
→ near the control

Affected region
→ loading state

Page visibility
≠ Job lifetime

Latest user intent
→ wins

Reduced Motion
→ no shimmer / continuous decorative motion

Loading failure
→ explicit state
not infinite loading
```

---

# 30. Empty / Error / Unavailable

本章定义 O3Pilot 在页面、Section、Metric、Chart、Table 与 Detail Surface 上如何把正式数据状态、指标状态与系统状态转换成用户可理解的界面。

本章不新增 `DATA_MODEL.md` 或 `METRICS.md` 的状态码。

正式关系：

```text
Formal Data / Metric / Retrieval State
↓
Presentation Decision
↓
Empty / Unavailable / Error / Not Yet Ready Surface
```

核心原则：

```text
Presentation Family
≠ Formal Data State

Empty
≠ Unavailable
≠ Error
≠ Unknown

Known Zero
≠ Empty
```

## 30.1 State Surface Principles

O3Pilot 正式采用四类用户可见 Presentation Family：

```text
Empty
Unavailable
Error
Not Yet Ready
```

这些名称只定义界面表达，不作为新的业务状态枚举。

任何 Surface 先读取正式状态，再决定：

```text
Preserve existing content?
Replace affected surface?
Show reason?
Show coverage / last valid time?
Offer recovery action?
```

不得先选一个视觉模板，再倒推业务状态。

## 30.2 Formal State vs Presentation Family

正式状态继续来自：

```text
DATA_MODEL.md
→ KNOWN / NULL / UNAVAILABLE / NOT_APPLICABLE / UNMATCHED / UNVERIFIED

METRICS.md
→ VALID / PARTIAL / UNAVAILABLE / NOT_APPLICABLE / UNVERIFIED /
   STALE / ESTIMATED / NO_DENOMINATOR / NO_RECENT_DEMAND / OPEN_COHORT

Retrieval / System State
→ request / sync / parser / integration / job result
```

例如：

```text
NO_DENOMINATOR
→ Metric may show “— · 无可计算样本”
→ does not create a new NOT_YET_READY metric status
```

正式原则：

```text
Internal status code
→ preserved

User-facing surface
→ selected from context
```

## 30.3 State Scope

状态必须限制在最小受影响范围：

```text
Component
Section
Page
Application
```

例如：

```text
Advertising Chart fails
→ Chart-level Error
→ Dashboard remains usable
```

```text
Buyer Feedback capability unavailable for selected Shop
→ Buyer Feedback Section / Page Unavailable
→ Sidebar remains stable
```

```text
Authentication invalid
→ Application-level security surface
```

正式原则：

```text
State scope
→ smallest affected surface
```

一个接口失败不得默认升级为全页错误。

## 30.4 Empty

`Empty` 只表示：

```text
retrieval succeeded
+
capability is available
+
query is valid
+
zero eligible records
```

示例：

```text
当前周期没有退货
```

```text
当前筛选条件下没有订单
```

禁止把以下情况显示为 Empty：

```text
API failure
permission unavailable
unknown capability
sync failure
parser failure
missing required source
```

正式原则：

```text
Empty
→ valid absence
```

## 30.5 Dataset / Filtered / Search Empty

Empty 至少区分：

### Dataset Empty

```text
当前周期没有退货
```

表示当前业务范围内确实没有记录。

### Filtered Empty

```text
当前筛选条件下没有订单

[清除筛选]
```

表示数据集存在，但当前 Filter 没有匹配结果。

### Search Empty

```text
未找到与 “015516...” 匹配的订单

[清除搜索]
```

正式关系：

```text
Filtered Empty
≠ Dataset Empty

Search No Result
→ Filtered Empty family
```

`Clear Filters` 不得重置 Shop、Reporting Currency、Date Range 等非 Filter Context。

## 30.6 Unavailable

`Unavailable` 表示本应存在或产品支持的能力，在当前 Shop / Source / Permission / Required Input 条件下无法提供。

示例：

```text
买家反馈不可用

当前店铺订阅不包含该数据。
```

```text
利润不可用

尚未导入必要的卖家成本数据。
```

`Unavailable` 不是 Error，也不是业务上的 0。

正式原则：

```text
Unavailable
≠ Empty
≠ Error
≠ Zero
```

当 Reason 已知时必须展示。

## 30.7 Not Applicable / Unsupported

`NOT_APPLICABLE` 与 `UNAVAILABLE` 不能使用完全相同的解释。

```text
UNAVAILABLE
→ this value would be meaningful, but cannot currently be obtained

NOT_APPLICABLE
→ this value does not apply to the current entity / mode
```

例如：

```text
N/A
该指标不适用于此履约模式
```

而不是：

```text
数据不可用
```

对于 Source Contract 不支持的条件，例如时间范围：

```text
当前数据源不支持此时间范围
```

不得显示成：

```text
暂无数据
同步失败
```

正式原则：

```text
Unavailable Surface
→ reason identifies capability boundary
```

## 30.8 Error

`Error` 表示一个本应能够成功的系统操作、查询、同步、解析或任务失败。

典型：

```text
无法加载订单
最近一次查询失败
[重试]
```

```text
同步失败
最后成功：10:28
[查看诊断]
```

Error 主文案表达：

```text
What failed
What is affected
What the user can do next
```

普通主界面不直接暴露：

```text
Stack Trace
SQLITE_BUSY
JSONDecodeError
HTTP response dump
Secret-bearing request details
```

技术细节进入 Data Center / Diagnostic Detail。

## 30.9 Last Known Data and Retained Error

存在可信 Last Known Data 时，Error 默认不清空内容。

正式模式：

```text
Last Known Value / Rows / Series
+
Failure State
+
Last Valid / Last Success Time
+
Recovery / Diagnosis
```

示例：

```text
库存
382

数据更新失败
最后有效：10:28
```

而不是：

```text
库存
—
```

Error 因此分为：

```text
Retained Error
Replacement Error
```

`Replacement Error` 只在没有任何可信内容可展示时使用。

正式原则：

```text
Error + Last Known Data
→ preserve content
```

## 30.10 Recovery Actions

Recovery Action 必须针对失败类型，并尽量限制在最小合理范围。

例如：

```text
Query failure
→ 重试

Sync failure
→ 重新同步该数据集

Credential validation failure
→ 检查凭证

Import validation failure
→ 重新选择 / 修正文件

Filtered Empty
→ 清除筛选
```

禁止所有错误统一使用：

```text
[刷新页面]
```

正式原则：

```text
Recovery action
→ follows failure type
→ narrowest meaningful scope
```

任何 Recovery Action 均不得执行 Ozon Server State 写操作。

## 30.11 Not Yet Ready

`Not Yet Ready` 是用户可见 Presentation Family，不是新的 Metric Status。

适用于：

```text
历史样本尚不足以形成趋势
预测模型尚无足够训练 / 回测输入
必要的本地数据尚未建立
整个分析 Surface 当前没有有意义的主结果
```

示例：

```text
历史数据不足
继续采集后才能生成趋势
```

它可以映射自正式状态或条件，例如：

```text
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
insufficient historical coverage
missing required local input
```

但正式状态必须仍可追溯。

## 30.12 Partial / Stale / Unmatched Boundaries

以下状态通常仍有可用内容，因此不得默认转成 Replacement Surface：

### PARTIAL

```text
Value remains visible
+
Coverage / Missing Reason
```

### STALE

```text
Last Known Value remains visible
+
Last Valid Time
```

### UNMATCHED

```text
Matched records remain usable
+
Unmatched records are identifiable
+
Mapping reason / action is available
```

### OPEN_COHORT

如果 Metric Contract 允许当前结果：

```text
Current Value
+
Cohort 尚未闭合
```

不能为了统一状态模板而把所有非 `VALID` 状态都替换成大面积空白。

正式原则：

```text
PARTIAL / STALE
→ usually retained-content states
```

## 30.13 Copy and Visual Treatment

State Surface 的标准信息结构：

```text
Optional semantic icon
Title
Explanation / Reason
Relevant context
Recovery / Next Step
```

示例：

```text
无法加载订单

最近一次同步失败。
最后成功：10:28。

[重试] [查看诊断]
```

视觉原则：

```text
Empty         → Neutral
Unavailable   → Neutral / subtle informational
Error         → Danger only when true error
Not Yet Ready → Neutral / informational
```

颜色和 Icon 只作为辅助。

禁止：

```text
Oops!
糟糕！
出了点问题 😢
```

优先精确说明事实。

正式原则：

```text
Reason
→ shown whenever known

State
≠ Color
```

## 30.14 Responsive and Accessibility

Narrow 下 State Surface：

- 保持同一语义；
- 不因为空间不足删除 Reason；
- Recovery Action 可以纵向排列；
- 不使用巨大 Illustration 挤占有效空间；
- 关键 Action 保持 ≥44px Touch Target。

Accessibility：

- Title / Reason 不能只由 Icon 表达；
- Error 不能只靠红色；
- 状态改变应按紧急程度使用合适的 Live Region；
- 普通 Empty 不使用 Assertive Announcement；
- Error 出现默认不抢走用户 Focus；
- Retry 后 Focus 保持合理；
- 200% Zoom 下文案和 Action 不丢失；
- CN / EN / RU 长文案可以正常换行。

## 30.15 State Surface QA

正式 QA 至少覆盖：

```text
Known Zero
Dataset Empty
Filtered Empty
Search Empty
UNAVAILABLE
NOT_APPLICABLE
Unsupported Range
Error without previous data
Error with Last Known Data
PARTIAL
STALE
UNMATCHED
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
Not Yet Ready
```

同时验证：

```text
Component scope
Section scope
Page scope
Desktop
Compact
Narrow
Light
Dark
Keyboard
Screen Reader
200% Zoom
CN / EN / RU
```

核心验收：

```text
Presentation Family
≠ Formal Data State

Known Zero
≠ Empty

Error + Last Known Data
→ preserve content

State scope
→ smallest affected surface

不得统一叫“暂无数据”
```

---

# 31. Responsive

O3Pilot 的 Responsive 目标不是把 Desktop 界面等比例缩小，而是在不同可用宽度下保持同一个业务问题、同一套数据语义和同一导航模型。

核心原则：

```text
Responsive
→ preserve task and semantics

Responsive
≠ shrink everything
≠ silently remove data
≠ mobile cardification
```

## 31.1 Breakpoints

正式 Breakpoints：

```text
≥ 1440px       Full Desktop
1200–1439px    Desktop
768–1199px     Compact
< 768px        Narrow
```

边界必须确定：

```text
768px
→ Compact
```

Feature 不得自行发明另一套 Breakpoint。

Breakpoint 只决定表现策略，不改变：

- Metric Contract；
- Shop Context；
- Reporting Currency；
- Date / Time Basis；
- Filter Semantics；
- Read-only Boundary。

## 31.2 Responsive Principles

正式优先级：

```text
Business meaning
>
Primary task
>
Interaction clarity
>
Visual symmetry
```

空间不足时优先：

1. 减少非必要并列；
2. 调整 Column Priority；
3. 使用 Horizontal Scroll；
4. 将 Secondary Detail 移入 Drawer / Detail；
5. 调整 Legend / Label 密度；
6. 最后才考虑改变容器形式。

禁止：

```text
Narrow
→ smaller fonts everywhere
→ hide status
→ hide currency
→ shorten date range silently
```

## 31.3 Shell and Sidebar

Sidebar 正式尺寸继续使用第 7 章：

```text
Full Desktop / Desktop
→ 248px Expanded or 72px Collapsed

Compact
→ 72px default rail

Narrow
→ permanent 60px icon rail
```

Narrow：

```text
60px rail
→ always present
```

允许临时展开 Label View 覆盖 Page Content，但不移除 Rail 这一基础导航模型。

Responsive Constraint 可以临时覆盖 Desktop Sidebar Preference，但恢复 Desktop 后必须恢复用户保存的 Expanded / Collapsed 偏好。

## 31.4 Page Gutters and Content Width

Page Gutter 继续使用第 12 章：

```text
Full Desktop / Desktop   32px
Compact                  24px
Narrow                   16px
```

Data Canvas：

```text
Fluid Analytical Canvas
Standard Content   ≈ 1200px max when reading benefits from constraint
Reading / Detail   ≈ 960px max when long-form comprehension benefits
```

宽度限制是内容策略，不是全站强制居中窄栏。

高密度 Table / Chart 可以使用 Fluid Canvas。

## 31.5 Information Priority

Responsive 不得静默删除重要信息。

页面应为字段和区块定义：

```text
Primary
Secondary
Detail-only
```

例如 Product Table Narrow：

```text
Primary
→ Product / SKU / critical status / primary metric

Secondary
→ additional columns via horizontal scroll or row detail
```

不得把 Secondary 直接从产品中消失。

正式原则：

```text
Hidden in viewport
≠ unavailable in product
```

## 31.6 Tables

Narrow Table 正式策略：

```text
Preserve table semantics
+
Key identity columns
+
Horizontal scroll
+
Row Detail for secondary information when useful
```

不强制 Card 化。

禁止：

- 将 12 列拆成 12 层 Card；
- 把 Identifier 截断到不可复制；
- 缩小 Body Typography；
- 静默删除 Status / Currency / Coverage；
- 改变 Sort / Filter Scope。

Sticky Key Column 在 Narrow 仍遵循第 23 章的使用条件。

## 31.7 Charts

Responsive Chart：

```text
preserve analytical question
not desktop geometry
```

可以：

- 减少 Axis Tick Density；
- 将 Legend 移到图下；
- 使用明确 Series Selector；
- 将多个并列 Chart 改为纵向；
- 在分类比较中改用 Horizontal Bar。

禁止：

- 静默删除 Series；
- 静默缩短日期范围；
- 隐藏 Unit / Currency；
- 改变 Metric；
- 因空间不足自动使用双 Y Axis。

如果隐藏 Series，必须明确显示当前 Selection / Top-N Scope。

## 31.8 Forms and Controls

Narrow Form：

```text
Multi-column
→ single column
```

单行 Control：

```text
Desktop 40px
Touch / Narrow ≥44px
```

Label、Helper、Error 仍然完整可见。

禁止：

- Placeholder-only Label；
- 隐藏 Helper Text；
- 横向强塞长 Label + Input；
- 为节省空间降低可触达目标。

Button 在 Narrow 可以 Full-width，但只有在提升任务完成度时使用，不全局 `width: 100%`。

## 31.9 Overlays

Drawer：

```text
Desktop
→ 480px Standard / 640px Wide

Narrow
→ full-width detail surface
```

Modal：

```text
Narrow
→ viewport width minus 16px gutters
```

复杂 Modal 如果接近完整应用页面，应升级为 Drawer / Full Page。

Popover：

```text
reposition / constrain within viewport
```

复杂 Filter Popover 在 Narrow 可以升级为 Drawer / Sheet-like Surface，但交互语义不变。

正式原则：

```text
Responsive container may change
while interaction semantics remain
```

## 31.10 Tabs, Filters and Search

Tabs Narrow：

```text
horizontal scroll
```

优先于缩小字体。

Filters：

```text
High-frequency Quick Filters
+
Filter Button
→ advanced filters in Drawer when needed
```

Applied Filters 仍须可发现。

Segmented Control：

- 2–4 个短 Label 可继续保留；
- 语言增长导致不可读时改用合适 Control；
- 不通过缩小文字硬塞。

Global Search / Command Palette Narrow 使用接近全宽 Surface，但仍保留原 Search Scope 和 Keyboard / Focus Contract。

## 31.11 Metrics and Summary Regions

Metric Presentation 在 Narrow：

- Primary Value 不缩到 Micro Typography；
- Unit / Currency 保持可见；
- PARTIAL / STALE 等关键 Status 保持可发现；
- Comparison Basis 不省略成只有箭头；
- 多个 Metric 可以从 4 列改为 2 列 / 1 列；
- 不为了保持 4-column Dashboard 而挤压内容。

```text
Metric Grid Columns
→ responsive

Metric semantics
→ stable
```

## 31.12 State Preservation

Breakpoint 变化不得无意义重置：

```text
Shop
Reporting Currency
Date Range
Comparison
Filters
Sort
Search
Selected Tab / Mode
Table position when practical
Detail route state when meaningful
```

例如：

```text
Desktop Filter Popover open
↓
viewport becomes Narrow
↓
convert to appropriate Narrow filter surface
```

不得把 Draft Filter 静默 Apply 或清空。

## 31.13 Touch, Pointer and Input Modes

Responsive Width 与 Input Mode 不是同一个概念。

```text
Narrow
≠ always Touch
Desktop
≠ always Mouse
```

Hover Motion 必须通过 Hover / Pointer capability gate。

Touch / coarse pointer 环境：

- Target ≥44px；
- 不依赖 Hover 才可理解；
- Tooltip 不能是唯一说明；
- Horizontal Scroll 必须可直接触摸操作。

Keyboard 在所有 Breakpoint 都必须有效。

## 31.14 Responsive QA

每个正式页面至少检查：

```text
1440px Full Desktop
1200px Desktop
1024px Compact
768px Compact boundary
390px Narrow
```

并额外验证：

```text
Sidebar preference restore
Long CN / EN / RU labels
Wide table
Many filters
Long identifier
Long currency amount
Partial / Stale status
Drawer Standard / Wide
Modal
Popover
Keyboard
Touch
200% Zoom
```

核心验收：

```text
Narrow
→ permanent 60px rail

Wide analytical table
→ horizontal scroll allowed

Responsive
→ preserves semantics
not removes difficult content
```

---

# 32. Accessibility

O3Pilot 正式 Accessibility 目标：

```text
WCAG 2.2 AA
```

Accessibility 是全局 Quality Attribute，不是上线前的补充修饰。

核心原则：

```text
Semantic HTML first
Keyboard complete
Focus visible
State not color-only
Motion optional
Data remains understandable without precision pointing
```

## 32.1 Accessibility Scope

正式验收至少覆盖：

- App Shell；
- Sidebar；
- Global Context Bar；
- Page Header；
- Tables；
- Charts；
- Forms；
- Tabs / Filters / Segmented Controls；
- Drawer / Modal / Popover；
- Toast / Notification；
- Loading / Empty / Error；
- Login / Session Revoked；
- Global Search；
- Data Center；
- Settings。

业务页面不得因为数据密度高而降低 Accessibility Contract。

## 32.2 Semantic Structure and Landmarks

优先使用原生 HTML 语义：

```text
header
nav
main
section
form
table
button
a
input
```

正式要求：

- Primary Sidebar 使用 `<nav>`；
- 主内容有唯一主要 `<main>`；
- 页面拥有明确 H1；
- Heading Level 反映内容结构；
- Link 用于 Navigation；
- Button 用于 Action；
- 只读数据不要伪装成 Disabled Input；
- Table 默认使用 Semantic Table，不无理由使用 ARIA Grid。

正式原则：

```text
Visual shape
≠ Accessibility role
```

## 32.3 Keyboard Interaction

主要产品路径必须能够只用键盘完成。

至少覆盖：

```text
Primary navigation
Global Search
Tabs
Filters
Search
Sort
Table actions
Open / Close Detail
Forms
Save / Cancel
Popover / Modal / Drawer
Retry / Recovery
Settings
```

要求：

- Tab Order 与视觉顺序一致；
- 无 Keyboard Trap，除了符合 Dialog Contract 的受控 Focus Trap；
- `Enter / Space / Arrow / Escape` 按对应标准 Pattern 工作；
- Hover-only Action 必须有 Focus / keyboard equivalent；
- 快捷键不能成为唯一入口。

## 32.4 Focus

Focus Ring 不得移除或通过低对比度弱化到不可见。

正式要求：

```text
Focus
→ always discoverable
```

- Focus Ring 不被 `overflow: hidden` 裁剪；
- Sticky Header / Column 不遮挡 Focus；
- Overlay 关闭后 Focus 返回合理 Trigger；
- Loading / Refresh 不无意义移动 Focus；
- Toast 出现不抢 Focus；
- Route Change 后 Focus 根据页面语义进入 Page Heading / retained context；
- Session Revoked 等安全状态进入明确页面焦点上下文。

## 32.5 Touch and Pointer Targets

正式区分：

```text
Desktop dense visual control
≠ Touch target requirement
```

Touch / Narrow / coarse pointer：

```text
Interactive Target ≥44×44px
```

视觉 Icon 可以是 16 / 18 / 20px，但可操作区域必须满足目标尺寸。

Compact 32px Button 只用于明确的 Desktop Dense Context，不作为 Touch UI 基线。

## 32.6 Color and Contrast

所有文本、Control、Focus、Status 必须满足 AA 对比度要求。

颜色不得成为唯一信息渠道。

例如：

```text
PARTIAL
→ label / text + optional color

Danger
→ icon / text + color
```

禁止：

```text
green number = good
red number = bad
```

却没有业务语义说明。

Direction 继续遵守：

```text
↑ / ↓
≠ automatic positive / negative
```

## 32.7 Text Resize, Zoom and Reflow

正式测试：

```text
Browser Zoom 200%
```

要求：

- 主内容仍可读取；
- 关键 Action 不被裁剪；
- Label 与 Input 关系不丢失；
- Modal / Drawer 仍可滚动和关闭；
- 表格可以水平滚动，而不是把字缩小；
- 长 CN / EN / RU 文本可以换行；
- 关键 Identifier 仍可完整获取 / 复制。

不通过固定高度裁切长文本。

## 32.8 Forms

Form Accessibility 继续使用第 24 章：

- Label programmatically associated；
- Required 可被程序识别；
- Helper / Error 与 Field 关联；
- Error 不只靠颜色；
- Password / Secret 允许 Paste 与 Password Manager；
- Disabled Reason 可发现；
- Submit Failure 保留输入。

Field Error 出现时不依赖 Toast 作为唯一提示。

## 32.9 Overlays

Drawer / Modal：

- Dialog / Modal semantic 正确；
- Accessible Name；
- Background inert；
- Focus 进入 Overlay；
- Focus 保持在 Active Overlay；
- Escape 行为一致；
- Close Button 有 accessible name；
- 关闭后 Focus Return。

Popover：

- Trigger 表达 expanded state；
- Popup 与 Trigger 有正式关联；
- Keyboard 可打开和关闭；
- 不只依赖 Hover。

## 32.10 Tables

Semantic Table 默认。

要求：

- Header 与 Cell 关系正确；
- Sort Control 可键盘操作并表达排序状态；
- Row Action 可 Focus；
- Horizontal Scroll 不阻断 Focus；
- Sticky UI 不覆盖 Focus；
- Pagination 语义真实；
- Virtualization 不破坏 Keyboard / Screen Reader / Scroll Position；
- Identifier 可获取完整值。

不要求每个 Cell 都成为 Tab Stop。

## 32.11 Charts

Chart 使用双通道：

```text
Visual Chart
+
Accessible Data Representation
```

至少提供：

- Chart Title；
- 简短分析说明；
- Legend；
- 非颜色唯一的 Series 区分；
- Accessible exact data / Table；
- Forecast / Comparison 状态文字说明。

大量数据点不全部进入 Tab Order。

```text
Accessible
≠ every mark is a Tab stop
```

## 32.12 Dynamic Updates and Live Regions

动态信息必须按紧急程度选择 Announcement。

```text
Routine success / small update
→ polite or no announcement when visual state is self-evident

Critical interaction failure
→ stronger announcement when necessary
```

禁止：

- 每个数据刷新都朗读；
- 每个后台 Job 进度都朗读；
- 所有 Toast 使用 assertive；
- 结果区域每次 Filter 输入都打断 Screen Reader。

正式原则：

```text
Dynamic update
≠ automatic interruption
```

## 32.13 Reduced Motion

Accessibility 必须包含第 16 章完整 Reduced Motion Contract。

至少：

```text
No shimmer
No long spatial overlay movement
No decorative chart animation
No programmatic smooth scroll
No path morph displacement when reduced
Focus visibility preserved
```

Reduced Motion 不得移除反馈和状态理解。

## 32.14 Localization Accessibility

Accessibility Label、Error、Status、Tooltip 与 Screen Reader Copy 均需支持：

```text
中文
English
Русский
```

禁止只翻译视觉文本，却保留错误语言的 `aria-label`。

技术 Code 可以不翻译，但需搭配人类可理解的本地化名称。

## 32.15 Accessibility QA

正式 QA 至少包含：

```text
Keyboard-only
Visible Focus
Screen Reader smoke test
200% Zoom
390px Narrow
High contrast-sensitive review
Reduced Motion
Light / Dark
CN / EN / RU
Long error text
Long identifiers
Wide tables
Interactive charts
Overlays
Forms
Session Revoked
```

核心验收：

```text
Semantic HTML first
Focus never invisible
Status never color-only
Hover never sole access path
Primary task keyboard-complete
```

---

# 33. Localization

O3Pilot 的设计目标必须从一开始支持：

```text
中文
English
Русский
```

本章定义 Layout、Formatting、Terminology 和 UI Copy 的多语言 Contract。

本章不承诺某个具体 Release 一定同时开放全部 Locale；正式 Release 支持范围由产品版本决定，但任何 Layout 都不得依赖“中文一定短”。

## 33.1 Localization Principles

```text
Semantic ID
≠ Localized Label

Meaning
→ stable

Copy
→ locale-specific
```

页面、导航、Metric、Status、Action 的内部语义 ID 必须稳定。

切换语言不能改变：

- Metric Code；
- Origin；
- Status；
- Shop identity；
- Data query；
- Read-only Boundary。

## 33.2 Navigation and Terminology

第 5 章定义的 Navigation Semantic ID 为正式内部身份。

例如：

```text
buyer-feedback
→ 买家反馈
→ Buyer Feedback
→ Обратная связь покупателей
```

正式显示文案进入 Locale Dictionary，不把中文字符串作为逻辑 Key。

核心业务术语必须统一维护，例如：

```text
母订单号
订单号
商品代码（货号）
SKU
Ozon Product ID
结算
预警
```

不同页面不能自行创造同义词。

## 33.3 Locale Architecture

正式 UI Copy 使用稳定 Message Key：

```text
nav.sales
nav.products
metric.return_rate
status.partial
action.retry
```

禁止：

```text
if (locale === 'ru')
  use a feature-specific hard-coded sentence
```

业务字段值与用户输入不通过 UI Translation Dictionary 修改。

## 33.4 Typography and Glyph Coverage

全 Locale 统一使用：

```text
MiSans Global
```

正式覆盖：

- 中文；
- Latin；
- Cyrillic；
- 数字；
- 常用货币符号；
- 常用标点与单位。

不得因俄语字符进入系统字体 fallback，造成：

- Baseline 变化；
- Numeric alignment 变化；
- Button 高度变化；
- 字重不一致。

## 33.5 Text Expansion and Wrapping

设计时至少预留：

```text
Button / Control label
→ 30–50% text expansion tolerance when practical
```

但不是通过固定加宽所有 Button 实现。

长文本策略：

```text
Grow
Wrap
Reflow
Use full-width on Narrow when useful
```

优先于：

```text
Shrink font
Ellipsis critical action
Hide reason
```

Sidebar Expanded、Tabs、Form Labels、State Messages 必须验证俄语长度。

## 33.6 Numbers and Currency

Numeric Value 的业务值不因 Locale 改变。

Display Formatting 可以按 Locale 使用：

- Group separator；
- Decimal separator；
- Currency display convention。

但 Reporting Currency 身份必须保持明确。

正式原则：

```text
Localized number formatting
≠ different calculation
```

Identifier 不使用 Locale Number Formatting。

例如：

```text
0155167697-0113
```

永远按字符串显示。

## 33.7 Date and Time

相对日期可以本地化：

```text
今天 / Today / Сегодня
```

但 Traceability 场景必须能够查看精确日期。

技术 / 分析界面优先避免纯数字顺序歧义：

```text
2026-09-04
```

普通用户友好文案可按 Locale 格式化，但不能把 Business Date 转成另一个时区日期。

时间默认 24 小时制：

```text
HH:mm
```

保持跨 Locale 扫描一致性。

## 33.8 Technical Codes and Identifiers

以下内容默认不翻译：

```text
Metric Code
Source Code
API Endpoint
Status internal code when shown in diagnostics
SKU
offer_id
product_id
order_number
posting_number
Job ID
Sync Run ID
```

但技术 Code 旁应提供本地化人类名称，除非当前 Surface 本身就是诊断工具。

例如：

```text
部分数据
PARTIAL
```

而不是只展示 `PARTIAL`。

## 33.9 Truncation and Tooltips

关键业务语义不能因为某种语言较长而被截断到不可理解。

允许 Ellipsis 的普通长文本必须有：

```text
Tooltip / Detail / accessible full value
```

关键 Action、Error、Status Reason、Currency、Identifier 不依赖 Tooltip 才能理解。

## 33.10 Grammar, Plural and Count

计数文案必须使用 Locale-aware Message Pattern。

例如英文 / 俄语复数规则不能通过：

```text
count + " items"
```

硬拼。

Count 不可知时不得为了满足文案模板伪造 `0`。

## 33.11 Missing Translation and Fallback

正式 Release 不允许在普通 UI 中静默显示：

```text
translation.key.name
undefined
[object Object]
```

Fallback 必须：

- 可预测；
- 可测试；
- 不改变业务语义；
- 不把技术 Key 当正式用户文案。

缺失翻译属于 QA Defect。

## 33.12 Localization QA

每个正式页面至少验证：

```text
中文
English
Русский
```

测试数据包括：

```text
Long sidebar labels
Long tabs
Long filter chips
Long button labels
Long status reason
Large currency amount
Negative amount
Long product name
Long identifiers
Plural counts
Date ranges
Error messages
```

同时验证：

```text
Desktop
Compact
390px Narrow
200% Zoom
Light / Dark
```

核心验收：

```text
Locale change
→ copy changes
→ semantics do not
```

---

# 34. Global Search / Command Palette

Global Search / Command Palette 是 O3Pilot 的全局查找与快速导航入口。

它必须优先帮助用户定位实体、页面和已有分析对象，而不是演变成隐藏的 Ozon 管理终端。

## 34.1 Entry

正式入口：

```text
macOS     ⌘K
Linux     Ctrl+K
```

Global Context Bar 同时提供可见 Search Entry。

快捷键是加速路径，不是唯一入口。

因为属于高频动作：

```text
Open / Close
→ immediate or near-imperceptible
```

不做明显全屏动效。

## 34.2 Search Scope

v1 Search 至少支持正式可定位实体：

```text
Product
SKU
offer_id
Ozon Product ID
Mother Order / order_number
Posting / posting_number
Campaign
Alert
```

当对应实体已经进入正式产品和索引能力时，可扩展：

```text
Return Case
Finance Accrual
Data Quality Issue
Recommendation
```

Search Scope 必须与实际已建立的索引 / 查询能力一致，不得显示不可工作的入口。

## 34.3 Query Semantics

Global Search 同时支持：

```text
Exact identifier lookup
Prefix / keyword lookup
Localized name lookup where supported
```

Identifier Search 优先：

```text
Exact Match
>
Prefix Match
>
Fuzzy / Text Match
```

避免产品名称的模糊结果覆盖一个精确 `posting_number`。

Search 不修改数据。

## 34.4 Result Grouping

结果按实体分组：

```text
商品
订单
广告
预警
...
```

每个 Result 至少表达：

```text
Primary identity
Secondary context
Shop when needed
Relevant status when useful
```

例如：

```text
0155167697-0113-1
订单 · Shop A · 已签收
```

不要在 Search Result 中堆完整业务详情。

## 34.5 Ranking and Ambiguity

跨店铺存在相同 SKU / offer_id 时，结果不得静默合并。

```text
Same identifier across shops
→ separate results
→ Shop context visible
```

跨店 Seller Catalog 等价关系只使用正式显式 Mapping，不通过相同名称自动合并。

如果 Query 有多个合理解释，结果分组而不是猜一个。

## 34.6 Navigation and Commands

Command Palette 可以提供：

```text
Go to Page
Open Settings section
Open Data Center
Open selected entity detail
Copy identifier where useful
```

不提供：

```text
Modify Ozon price
Update inventory
Change bid / budget
Send reply
Cancel order
Modify webhook subscription
```

正式原则：

```text
Command Palette
≠ hidden write console
```

高风险 O3Pilot Local State 变更也不通过模糊文本 Command 隐式执行；仍使用正式页面 / Dialog Contract。

## 34.7 Shop Context

默认 Search 在当前 Shop Context 下优先排序。

跨店能力允许时：

```text
Current Shop results
→ first

Other Shop results
→ clearly labeled
```

如果用户当前处于跨店 Context，Result 必须显示 Shop Identity。

Search 不能改变 Shop Context 而不让用户知道。

## 34.8 Keyboard and Focus

打开后：

- Focus 进入 Search Input；
- Arrow Keys 移动 Result Selection；
- Enter 打开；
- Escape 关闭；
- Tab 行为遵守正式可访问 Combobox / Dialog Pattern；
- 关闭后 Focus 返回 Trigger / previous context。

Result Selection 必须同时有视觉和可访问状态。

## 34.9 Loading, Empty and Error

Search Input 本地字符反馈立即发生。

远程 / Server Query 可以 Debounce，但：

```text
Input responsiveness
>
query completion
```

Loading 不锁死 Input。

No Result：

```text
未找到匹配结果
```

如果 Search 本身失败：

```text
无法搜索
[重试]
```

不得把 Search Error 表现成“没有结果”。

## 34.10 Responsive

Desktop 使用 Command Palette Overlay。

Narrow：

```text
near-full-width search surface
within 16px viewport gutters
```

Result Group 纵向布局。

不因为 Narrow：

- 减少 Search Scope；
- 隐藏 Shop；
- 截断精确 Identifier；
- 改成只支持当前页查找。

## 34.11 Privacy and Security

Search Result 只显示当前有效 Session 有权访问的 O3Pilot 业务数据。

Login / Session Revoked 状态：

```text
Global Search
→ unavailable
```

Secret、Password、Credential 明文、Recovery Key 不进入 Global Search Index。

## 34.12 Global Search QA

正式 QA 至少覆盖：

```text
Exact posting_number
Exact order_number
SKU
offer_id
Ozon Product ID
Product name
Same SKU across shops
No result
Search failure
Rapid typing / stale response race
Keyboard-only
Escape / Focus return
390px Narrow
CN / EN / RU
Long result labels
Session Revoked
```

核心验收：

```text
Exact identifier
→ high priority

Search Error
≠ Empty Result

Command Palette
≠ Ozon write surface
```

---

# 35. Page Template

本章定义 O3Pilot 页面如何组织稳定的信息层级。

Page Template 是结构 Contract，不是要求每个页面机械拥有相同数量的 KPI、Chart 或 Card。

核心原则：

```text
Page structure
→ follows user question

Not
→ fill component slots
```

## 35.1 Page Families

O3Pilot 正式页面主要分为：

```text
Analytical Page
Entity List Page
Entity Detail Page
Settings / Configuration Page
System / Data Operations Page
Security State Page
```

不同 Page Family 共享 Shell、Typography、State、Responsive 和 Accessibility Contract，但内容结构可以不同。

## 35.2 Relationship to App Shell

所有已登录业务页面复用第 6 章：

```text
App Shell
├── Sidebar
├── Global Context Bar
├── Page Header
└── Page Content
```

Page 不自行创建第二套：

- Global Shop Selector；
- Global Reporting Currency；
- Global Search；
- Primary Navigation。

## 35.3 Page Header Anatomy

标准 Page Header：

```text
Page Title
Optional one-line description
Page Context / controls
Optional local actions
```

Analytical Page 还可包含：

```text
Date Range
Comparison
Primary Time Basis when useful
```

Data Freshness 的 Global Summary 位于 Global Context Bar；页面只在当前 Dataset 有特殊 Freshness / Gap 时增加 Local State。

正式原则：

```text
Global health
→ Global Context Bar

Page-specific data limitation
→ Page / affected section
```

## 35.4 Analytical Page Template

参考结构：

```text
┌────────────────────────────────────────────────────────────┐
│ Page Title                       Date / Compare / Actions    │
│ Optional description / time basis                          │
├────────────────────────────────────────────────────────────┤
│ Primary Metrics / Summary                                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│                    Primary Analysis                        │
│                                                            │
├──────────────────────────────┬─────────────────────────────┤
│ Supporting Analysis          │ Supporting Analysis         │
├──────────────────────────────┴─────────────────────────────┤
│ Filters / Search / Controls                                │
│ Data Table / Detailed Evidence                             │
└────────────────────────────────────────────────────────────┘
```

不是所有页面都必须：

- 4 个 KPI；
- 2 张 Secondary Chart；
- Table。

没有业务价值就不放。

## 35.5 Entity List Page Template

适用于：

```text
Products
Orders
Campaigns
Alerts
Imports
Quality Issues
```

标准结构：

```text
Page Header
↓
Quick Summary when useful
↓
Search / Filters / Sort / View controls
↓
Result summary
↓
Table / List
↓
Detail route / Drawer
```

Applied Filter / Sort 必须可发现。

Filtered Empty 与 Dataset Empty 分开。

## 35.6 Entity Detail Page Template

适用于复杂实体 Full Detail。

标准结构：

```text
Identity
Current State
Primary Facts
Stable peer information domains
History / Timeline
Related Entities
Metrics / Analysis
Source / Lineage
```

简单详情优先 Drawer；复杂、深度分析或需要可分享 Route 时使用 Full Page。

具体复用第 54 章。

## 35.7 Settings / Configuration Template

标准结构：

```text
Settings navigation
↓
Section title + explanation
↓
Form / current status
↓
Save / local action when required
↓
Validation / verification state
```

默认单列 Form。

不使用 Card-in-Card 组织每两个字段。

O3Pilot Local State 可写，但任何设置页不得突破 Ozon Read-only Boundary。

## 35.8 Data Operations Template

Data Center、Import、Backup 等页面可包含：

```text
System / Data Status
↓
Operation controls
↓
Persistent Job / Run status
↓
History / Diagnostics
```

长任务使用 Persistent Job，不以全页 Spinner 代替。

## 35.9 Density and Surface Strategy

Page Family 决定 Density：

```text
List / comparison
→ higher density

Analysis
→ medium density

Detail / diagnosis
→ medium to lower density

Settings
→ lower to medium density
```

Section 不等于 Card。

遵守：

```text
Spacing > Background Difference > Hairline > Surface > Shadow
```

## 35.10 Responsive Page Template

Desktop 可以使用并列 Supporting Analysis。

Compact / Narrow：

```text
parallel sections
→ vertical sequence
```

优先级顺序必须保留。

Page Header Actions 在 Narrow 可以换行或进入 More，但关键 Context 不得隐藏。

Table / Chart / Overlay 继续使用各专项 Responsive Contract。

## 35.11 Page Template QA

每个页面必须能够明确回答：

```text
What question does this page answer?
What is primary evidence?
What is secondary evidence?
What context controls the data?
What states can invalidate or weaken the result?
Where can the user drill down?
What local actions exist?
```

核心验收：

```text
No business value
→ no component

Page-specific status
→ near affected content

Page Template
≠ mandatory dashboard grid
```

---

# 36. 登录页

登录页是 O3Pilot 的 Owner Authentication 入口。

目标：

```text
Minimal
Private
Unambiguous
Password-manager friendly
No business data leakage
```

登录页不是营销页，也不是 Ozon Credential 配置页。

## 36.1 Security and Product Boundary

登录前不得显示：

- 店铺名称；
- 销售数字；
- Alert 数量；
- 数据同步状态；
- Ozon Client ID；
- Credential 状态；
- Backup 详情；
- 最近访问实体。

只展示完成 Owner Login 所需信息。

Ozon Credential 不在登录页输入。

## 36.2 Layout

正式 Wireframe：

```text
┌────────────────────────────────────────────────────────────┐
│                                                            │
│                         O3Pilot                             │
│                                                            │
│                请输入实例所有者密码                        │
│                                                            │
│              ┌────────────────────────────┐                │
│              │ Password                   │                │
│              └────────────────────────────┘                │
│                                                            │
│                       [ 登录 ]                             │
│                                                            │
│                 Single-user instance                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

不使用：

- Hero Illustration；
- 新闻 / 更新内容；
- 商业 Marketing Copy；
- Ozon 品牌认证暗示；
- 第三方社交登录。

## 36.3 Login Form

Form：

```text
Password Label
Password Input
Submit
Inline Error when needed
```

Password Label 必须持久可见。

允许：

```text
Password Manager
Paste
Unicode
Spaces when Security Contract allows
```

不得阻止 Paste。

DESIGN 不重新定义密码长度和 Hash；具体规则由 `SECURITY.md` 决定。

## 36.4 Authentication Error

错误文案应精确但避免泄漏不必要安全细节。

例如：

```text
密码不正确
```

或在安全策略需要时使用更一般的：

```text
无法登录，请检查密码后重试
```

Rate Limit：

```text
登录尝试过多，请稍后重试
```

不把内部 Security Counter / Hash / Session Detail 暴露给匿名用户。

Error 靠近 Form，不依赖 Toast-only。

## 36.5 Loading and Submission

点击登录后：

```text
登录
→ 正在登录…
```

Button 保持几何稳定，防止重复提交。

整个页面不需要 Skeleton。

登录成功：

```text
Authenticated App Shell
→ directly available
```

不插入装饰性成功动画。

## 36.6 Existing / Revoked Session

如果当前浏览器已有有效 Session，登录入口不应重复展示业务敏感登录页状态；应用直接进入已认证 Shell。

如果 Session 被撤销，则使用第 37 章 Session Revoked Surface，而不是把旧业务页面后面盖一个小 Toast。

## 36.7 Responsive

登录 Form 使用 Reading-width Surface，不因 Desktop 超宽而无限拉伸。

Narrow：

- 16px Viewport Gutter；
- Input Touch Target ≥44px；
- Submit 可使用 Full-width；
- 不缩小 Label / Error；
- Virtual Keyboard 出现时仍能看到当前 Input / Error / Submit。

## 36.8 Accessibility

要求：

- 页面 H1 / Product Name 语义明确；
- Password Input 有 Label；
- Password Manager 可识别；
- Error 与 Input 关联；
- Focus Ring 可见；
- Enter 可以提交；
- Loading 状态可被识别；
- 不自动朗读安全无关信息；
- 200% Zoom 正常；
- CN / EN / RU Copy 正常换行。

## 36.9 Login QA

至少覆盖：

```text
Fresh unauthenticated visit
Correct password
Incorrect password
Rapid repeated submit
Rate limited state
Password manager fill
Paste
Keyboard-only
200% Zoom
390px Narrow
Light / Dark
CN / EN / RU
Existing valid session
Session revoked path
```

核心验收：

```text
Login page
→ no business data leakage
→ no Ozon credential input
→ no marketing surface
```

---

# 37. Session Revoked

O3Pilot v1 使用 Single User / Multiple Active Server-side Sessions。

Session 的建立、撤销与失效语义由 `SECURITY.md` 定义；`DESIGN.md` 不定义撤销原因，只投影当前 Session 已失效后的界面状态。

Session Revoked 是 Security State Page，不是普通 Error Toast。

## 37.1 Purpose

本页必须明确表达：

```text
Current session is no longer valid
Business data is no longer available in this session
User must authenticate again to continue
```

不尝试推断或展示另一台设备的硬件身份。

## 37.2 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│                                                            │
│                    当前会话已结束                          │
│                                                            │
│       当前会话已不再有效。                                │
│       为保护业务数据，请重新登录后继续。                  │
│                                                            │
│                       [ 重新登录 ]                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

如 `SECURITY.md` 提供可安全展示的具体 Security Reason，文案可以据此调整；`DESIGN.md` 不自行推断或定义撤销原因，也不能显示不可信的设备推断。

## 37.3 Business Data Visibility

Session Revoked 后：

```text
Previously rendered business content
→ no longer visible
```

不得：

- 在背景保留可读 Dashboard；
- 继续允许 Table Copy；
- 继续允许 Drawer 浏览；
- 继续让 Global Search 返回缓存业务实体。

正式原则：

```text
Revoked session
→ no readable business surface
```

## 37.4 Cached / In-memory State

UI 必须进入明确的未认证状态。

表现层不得依赖“API 以后都会 401”而继续保留旧数据。

以下临时 UI State 可以在安全允许的范围内丢弃：

```text
Open Drawer
Popover
Draft query
Transient selection
```

Secret / Credential / Recovery Material 不保留在可恢复 UI 中。

## 37.5 Actions

主操作仅需要：

```text
重新登录
```

必要时可以提供：

```text
返回登录页
```

不需要：

- Reload；
- Retry Business API；
- Continue Offline；
- Keep Reading Cached Data。

## 37.6 Motion and Transition

进入 Session Revoked：

```text
Security state change
→ immediate
```

不等待 Page Exit Animation。

Reduced Motion 与 Standard Motion 行为都应快速进入最终状态。

## 37.7 Responsive and Accessibility

页面使用简单 Reading-width Layout。

要求：

- H1 明确；
- Reason 可读；
- Primary Action Keyboard 可达；
- Focus 进入新安全页面的主要上下文；
- 不依赖颜色说明会话失效；
- 390px / 200% Zoom 正常；
- CN / EN / RU 长文案可换行。

## 37.8 Session Revoked QA

至少覆盖两个独立场景。

### Scenario A — Multiple Active Sessions

```text
Login A active
Login B succeeds
A remains valid
B remains valid
```

该场景只验证 `SECURITY.md` 已定义的多 Session 语义，不由 DESIGN 新增 Session 生命周期规则。

### Scenario B — Revoked-state UI

使用一个 `SECURITY.md` 已定义的 Session invalidation event 作为测试 fixture，例如 Password Change：

```text
Session active
SECURITY-defined invalidation event occurs
Current Session becomes invalid
Session Revoked page shown
Business content disappears
Global Search unavailable
Open Drawer removed
Keyboard focus moves to revoked page
Re-login succeeds
390px Narrow
Light / Dark
CN / EN / RU
```

核心验收：

```text
Session Revoked
→ persistent security page
→ no previous business data remains readable
```

---
# 38. 总览 Dashboard

总览是 O3Pilot 的经营入口，不是所有功能的缩略图集合。

它回答两个问题：

```text
现在发生了什么？
最值得关注什么？
```

正式信息路径：

```text
State
↓
Change
↓
Risk / Attention
↓
Evidence
↓
Recommended Attention
```

## 38.1 Context

Dashboard 使用：

```text
Global Shop Context
Global Reporting Currency
Page Date Range
Page Comparison
```

默认 Display Timezone 继续使用 `Asia/Shanghai`，但各 Metric 的正式 Time Basis 仍由 Metric Contract 决定。

Dashboard 同一页面可以存在多个 Time Basis；当某个 Metric 与页面主要时间口径不同，必须在 Metric / Detail 中明确。

## 38.2 Information Hierarchy

正式顺序：

```text
Primary business state
↓
Meaningful change
↓
Important risk / anomaly
↓
Supporting domain summaries
↓
Recommendations
```

禁止按 Sidebar 顺序机械展示每个模块一个 Card。

Dashboard 不承担完整诊断；详细证据进入对应页面或 Detail。

## 38.3 Primary Metrics

Dashboard 只展示少量真正跨域核心 Metric。

每个 Metric 必须引用正式 `metric_code / display_name`，例如销售金额不得统一写成模糊“销售额”。

正式要求：

```text
Metric Name
Primary Value
Comparison when valid
Status when attention is required
Origin / As-of discoverable
```

`VALID` 默认 Quiet。

`PARTIAL / STALE / ESTIMATED / OPEN_COHORT` 等影响判断时必须可见。

Metric 数量由经营问题决定，不要求固定 4 个。

## 38.4 Core Trend

Primary Analysis 优先展示一张核心经营趋势，而不是多个小型 Decorative Chart。

趋势必须明确：

```text
Metric
Unit / Currency
Date Range
Time Basis
Comparison when enabled
```

多币种未选择 Reporting Currency 时不得绘制无意义合计曲线。

Missing / Gap 使用断点和状态说明，不自动插值。

## 38.5 Risk and Attention

Dashboard 的 Risk / Attention 区域聚合当前最重要的：

- 库存风险；
- 履约异常；
- 退货异常；
- 财务 / 利润异常；
- 广告异常；
- 店铺健康风险；
- 数据质量 / Freshness 问题。

这些内容来自正式 Alert / Metric / Data Quality 结果，不在 Dashboard 重新定义阈值。

正式原则：

```text
Dashboard attention
→ references domain evidence
not creates a second alert model
```

## 38.6 Supporting Domain Summaries

可以使用 2–4 个 Supporting Summary，例如：

```text
Inventory Risk
Fulfillment / Returns
Advertising / Shop Health
Profit / Cost Coverage
```

每个 Summary 只回答一个明确问题，并提供进入对应页面的 Link。

不要把完整 Table、完整 P&L、完整 Campaign 管理器塞进 Dashboard。

## 38.7 Recommendations

Dashboard 可以显示少量高优先级 Recommendation 摘要：

```text
Entity
Recommendation summary
Reason
Priority / Confidence when formally available
Coverage / Status when material
[查看依据]
```

允许：

```text
查看
复制建议
打开经营建议页
```

禁止：

```text
立即执行
应用到 Ozon
修改价格
修改库存
修改广告
```

## 38.8 Multi-shop

跨店 Dashboard 只在当前 Shop Context 明确为跨店范围时进行跨店聚合。

要求：

- 每个事实仍归属 Shop；
- Money 必须遵守 Reporting Currency；
- Ratio 重聚合分子 / 分母；
- 部分 Shop 数据不完整时显示 Coverage；
- 不能把缺失 Shop 当 0；
- 能够 Drill-down 到具体 Shop。

例如：

```text
5 个店铺
4 完整 · 1 延迟
```

## 38.9 Dashboard States

不同 Section 独立进入：

```text
Loading
Empty
Unavailable
Partial
Stale
Error
```

一个广告数据源失败不应让整个 Dashboard 进入 Error。

如果已有 Last Known Data，保留内容并显示 Stale / Failure Context。

## 38.10 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 总览                           7D · 较上一周期              │
│ 当前经营状态与最值得关注的问题                            │
├────────────────────────────────────────────────────────────┤
│ Metric        Metric        Metric        Metric            │
│ value         value         value         value             │
│ compare       status        compare       coverage          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│                    核心经营趋势                            │
│                                                            │
├──────────────────────────────┬─────────────────────────────┤
│ 当前重要风险                 │ 库存风险                    │
│ Alert / Evidence             │ Stockout / DOC              │
├──────────────────────────────┼─────────────────────────────┤
│ 履约 / 退货                  │ 广告 / 店铺健康             │
├──────────────────────────────┴─────────────────────────────┤
│ 经营建议摘要                                               │
│ [查看依据] [复制建议]                                      │
└────────────────────────────────────────────────────────────┘
```

Global Data Status 不在页面重复造一套大 Banner；只有异常 Dataset 在局部升级表达。

## 38.11 Responsive

Desktop：Metric 与 Supporting Summary 可以并列。

Compact / Narrow：

```text
Primary Metrics
→ 2 columns / 1 column as needed

Supporting Summary
→ vertical
```

Narrow 不隐藏：

- Currency；
- Status；
- Comparison Basis；
- Alert Reason。

Chart 继续保持同一 Metric 和 Date Context。

## 38.12 Dashboard QA

至少覆盖：

```text
Single Shop
Multi-shop
Single currency
Mixed currency without Reporting Currency
Reporting Currency selected
All VALID
PARTIAL metric
STALE metric
One source FAILED
No alerts
Many alerts
No recommendations
Forecast / Estimated result
390px Narrow
CN / EN / RU
```

核心验收：

```text
Dashboard
→ prioritizes attention
not component count
```

---

# 39. 销售

销售页回答：

```text
卖得怎么样？
增长 / 下滑来自哪里？
流量与转化哪里发生变化？
```

它聚焦销售、流量、搜索和商品贡献，不把 Finance、Advertising Attribution 与 Profit 的不同金额概念混为同一个“销售额”。

## 39.1 Context

页面至少拥有：

```text
Shop Context
Date Range
Comparison
Reporting Currency when needed
Primary Time Basis
```

具体 Metric 使用正式 `time_basis`。

当同页不同 Metric 来自不同业务日期语义时，差异必须可发现。

## 39.2 Summary Metrics

Summary 只选经过正式定义并可解释的销售核心指标，例如：

```text
Ordered Units
Order Count
Ozon Analytics Revenue when available
Conversion Metric when formally available
```

这些是页面示例，不改变 `METRICS.md` 的 Metric Contract。

页面必须使用正式 Display Name，不把：

```text
order_gross_value
buyer_paid_value
ozon.analytics.revenue
performance_orders_money
```

统一改名成“销售额”。

## 39.3 Sales Trend

Primary Trend：

```text
Selected Sales Metric
by active Date Range / Time Basis
```

支持 Comparison 时：

```text
Current Period
Previous Period / Previous Year
```

必须满足第 18、20、21 章的 Comparison Contract。

Open Current Period 不与完整 Previous Period 静默比较。

## 39.4 Traffic, Search and Conversion

流量 / 搜索 / 转化能力只在正式来源可用时展示。

可以形成：

```text
Traffic / Search visibility
↓
Product view / related event
↓
Order / conversion
```

只有相同 Cohort / Eligible Population Contract 成立时才使用 Funnel。

如果无法建立真实 Funnel：

```text
Metric + Trend + Table
```

优先于伪 Funnel。

## 39.5 Product / SKU Contribution

Contribution Analysis 用于回答：

```text
增长主要来自哪些 Product / SKU？
```

适合：

```text
Horizontal Bar
Contribution Table
Top-N with explicit scope
```

如果使用 Top-N：

```text
Top 10 by <formal metric>
```

必须显式显示，不能静默隐藏其他 SKU。

## 39.6 SKU Performance Table

标准列根据正式数据能力选择，例如：

```text
Product / SKU
offer_id
Ordered Units
Orders
Formal Revenue Metric
Conversion
Comparison
Status / Coverage when material
```

要求：

- Identifier 左对齐；
- Numeric 右对齐；
- Money 显示 Currency；
- Sort 作用于完整 Filtered Result；
- Search Scope 明确；
- Filtered Empty 可清除筛选。

## 39.7 Search and Filters

Filter 用于：

```text
Product / SKU
Category when formally available
Sales change / risk conditions
Other supported dimensions
```

Filter 不承担：

- Shop Context；
- Date Range；
- Reporting Currency。

Search Placeholder 应明确 Scope，例如：

```text
搜索商品、SKU、商品代码
```

## 39.8 Data Source and State

Seller Analytics / other Seller API 数据与 Performance 数据拥有不同 Freshness。

页面局部必须表达：

```text
Current coverage
Gap when material
PARTIAL / STALE
```

不能因为都在“销售页”就表现为同一更新时间。

## 39.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 销售分析                         30D · 较上一周期           │
│ Time Basis / special data state when needed                │
├────────────────────────────────────────────────────────────┤
│ Metric          Metric          Metric          Metric      │
├────────────────────────────────────────────────────────────┤
│                     销售趋势                               │
│ Metric · Unit / Currency                                   │
├──────────────────────────────┬─────────────────────────────┤
│ 流量 / 搜索 / 转化           │ Product / SKU 贡献          │
├──────────────────────────────┴─────────────────────────────┤
│ Search / Filters                                            │
│ SKU Performance Table                                      │
└────────────────────────────────────────────────────────────┘
```

## 39.10 Responsive

Narrow：

- Trend 全宽；
- Supporting Analysis 纵向；
- SKU Table 保持 Table + Horizontal Scroll；
- Applied Filters 可发现；
- 不缩短 Date Range；
- Money / Currency 不隐藏。

## 39.11 Sales QA

至少覆盖：

```text
Revenue source variants
Single / mixed currency
Comparison
Open period
Traffic unavailable
Performance gap
PARTIAL
No sales = known zero / empty records distinction
Top-N scope
Large SKU table
390px Narrow
CN / EN / RU
```

核心验收：

```text
One label
→ one formal metric meaning
```

---

# 40. 商品

商品页回答：

```text
商品当前是什么状态？
价格、内容、库存、销售、退货、广告、利润之间有什么关系？
```

`Product` 是 Shop-scoped Ozon 商品；跨店真实商品等价关系必须通过正式 Seller Catalog Mapping 建立。

## 40.1 Product List Context

页面默认在当前 Shop Context 下列出商品。

Header：

```text
商品
搜索商品 / SKU / 商品代码 / Ozon Product ID
```

跨店 Context 存在时，Shop 必须成为结果的一等维度。

## 40.2 Filters

以下是 Filter，不是 Tabs：

```text
状态
风险
内容问题
价格风险
库存风险
其他正式可筛选状态
```

Quick Filter 可以显示：

```text
[风险]
[内容问题]
[低库存]
```

Applied State 必须可发现和清除。

正式原则：

```text
Same Product Dataset + subset change
→ Filter
not Tab
```

## 40.3 Product Table

典型 Column Groups：

```text
Identity
→ Product / SKU / offer_id / Ozon Product ID

Current State
→ status / availability

Business Evidence
→ price / inventory / sales / return / advertising / profit
```

具体 Column 只在数据可用时出现；Capability 不可用不得用 0 填充。

`status`、`availability`、`inventory` 必须分别显示，不能合成未经定义的：

```text
is_sellable
```

作为 Ozon 原始事实。

## 40.4 Identity

主要 Identity 推荐：

```text
Product Name
SKU
商品代码（offer_id）
Ozon Product ID
```

Identifier：

- 左对齐；
- 不 Locale-number-format；
- 保留完整 Copy Value；
- 多店同值不自动合并。

## 40.5 Product State Presentation

Ozon Source State 与 O3Pilot Derived Risk 分开。

例如：

```text
Ozon 商品状态
→ source / normalized fact

低库存风险
→ O3Pilot derived result
```

不能用一个颜色 Badge 同时表达二者。

`UNAVAILABLE / UNVERIFIED / PARTIAL` 继续使用正式 Data State Language。

## 40.6 Product Detail

复杂 Product Full Detail 使用稳定同级信息域：

```text
概览
价格
内容
库存
销售
退货
广告
利润
买家反馈
历史
来源
```

这些是 Page-local Tabs / Detail Domains，不进入 Sidebar。

如果某个能力不可用：

```text
Tab remains stable
→ content shows Unavailable / Not Applicable
```

不随 Shop Capability 静默删除整个 Domain。

简单快速检查优先 480px / 640px Detail Drawer；深度分析使用 Full Page。

## 40.7 Cross-shop Seller Catalog Mapping

跨店聚合只使用正式：

```text
seller_catalog_item
+
seller_catalog_product_mapping
```

页面可以显示：

```text
已映射到卖家商品
未匹配
规则匹配
待验证
```

但不得通过：

```text
same name
same SKU
same offer_id
```

自动宣称两个 Shop Product 一定相同。

Mapping 修改属于 O3Pilot Local State，可以进入正式 Mapping Flow。

## 40.8 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 商品                   [搜索商品、SKU、商品代码、Product ID]│
├────────────────────────────────────────────────────────────┤
│ [风险] [内容问题] [低库存]        筛选 3 · 清除筛选        │
├────────────────────────────────────────────────────────────┤
│ 商品 │ SKU │ 状态 │ 可用性 │ 价格 │ 库存 │ 销售 │ 退货... │
│ ---------------------------------------------------------  │
│ ...                                                        │
└────────────────────────────────────────────────────────────┘
```

Full Detail：

```text
Product Identity
[概览][价格][内容][库存][销售][退货][广告][利润][买家反馈]...
Current State / Analysis / History / Source
```

## 40.9 Responsive

Narrow：

- Search 全宽；
- Quick Filters 横向滚动或进入 Filter Surface；
- Table 使用 Key Columns + Horizontal Scroll；
- Product Name 可换行；
- Identifier Copy 保持完整；
- Detail Drawer 升级为 Full-width Surface；
- Detail Tabs 可横向滚动。

## 40.10 Product QA

至少覆盖：

```text
Long product name
Same SKU across shops
Changed offer_id history
UNAVAILABLE capability
UNMATCHED catalog mapping
PARTIAL profit
Zero inventory
Inbound inventory
No sales
Many columns
Product detail domains
390px Narrow
CN / EN / RU
```

核心验收：

```text
Product identity
→ precise

Source fact
≠ Derived risk

Cross-shop equivalence
→ explicit mapping only
```

---

# 41. 订单

订单页回答：

```text
这个订单发生了什么？
当前处于哪个生命周期状态？
取消、履约、退货与财务事实如何关联？
```

订单术语正式固定：

```text
order_number
→ 母订单号

posting_number
→ 订单号
```

示例：

```text
0155167697-0113
→ 母订单号

0155167697-0113-1
→ 订单号
```

不得反转。

## 41.1 Search and Query

主 Search：

```text
搜索母订单号、订单号、SKU
```

Search Scope 必须作用于完整 Query Scope，不仅是当前已加载 Page。

Quick Filters 可以覆盖：

```text
Lifecycle state
Cancelled
Delayed / risk
Fulfillment mode
Other formally supported fields
```

这些是 Filter，不是稳定页面 Tabs。

## 41.2 Order Table

核心列：

```text
母订单号
订单号
商品 / SKU
正式金额 Metric / Currency
Lifecycle State
Fulfillment
Created Time
Relevant risk / status
```

Identifier 左对齐且可复制。

Money 不使用“金额”掩盖不同正式定义；Column Header 必须说明对应 Money Contract。

Sort / Filter / Pagination 使用统一 Query State。

## 41.3 Lifecycle State

UI 展示标准化业务状态时，必须能够追溯 Source Status / Mapping Version。

Timeline 不根据 UI Guess 构造不存在的节点。

可以显示：

```text
Created
Processing
Handover / Shipped
Delivered
Cancelled
Return events
```

具体事件只在事实存在时出现。

正式原则：

```text
Timeline
→ observed / normalized events
not decorative inferred milestones
```

## 41.4 Cancellation

取消是独立经营行为，不与退货混为同一状态。

订单页负责提供取消记录与订单生命周期上下文，例如：

```text
Cancelled state
Cancellation time
Reason when available
Seller responsibility when available
Pre-shipment / post-shipment classification when formally established
```

聚合取消分析可以作为订单页的 Secondary Analysis / Filtered View，并链接到相关履约风险；不得把取消率并入退货率。

## 41.5 Order Detail

标准 Detail：

```text
Identity
↓
Order / Posting Summary
↓
Lifecycle Timeline
↓
Items
↓
Fulfillment / Logistics
↓
Cancellation / Return when applicable
↓
Finance / Accrual Relations
↓
Source / Lineage
```

Order Detail 优先 Drawer；复杂跨域诊断可以 Full Page。

`Mother Order` 与 `Posting` 层级必须在视觉结构中明确。

## 41.6 Related Entities

Detail 可以跳转：

```text
Product
Return Case
Finance Accrual
Alert
Related Recommendation
```

跳转保持当前 Primary Navigation Context 或进入明确的新 Primary Workspace。

禁止无限 Nested Drawer。

## 41.7 Time and Money

时间：

- Timestamp 转换到 Display Timezone；
- Source Business Date 保持 Date-only 语义；
- Timeline 可查看原始时间 / Source when needed。

Money：

- 保留 Currency；
- 不自动使用 Shop Settlement Currency 替代未知币种；
- Reporting Currency Conversion 与 Source Money 分层。

## 41.8 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 订单                         [搜索母订单号、订单号、SKU]    │
├────────────────────────────────────────────────────────────┤
│ Quick Filters                    筛选 n · Sort              │
├────────────────────────────────────────────────────────────┤
│ 母订单号 │ 订单号 │ 商品 │ 金额/币种 │ 状态 │ 履约 │ 时间   │
│ ---------------------------------------------------------  │
│ ...                                                        │
└────────────────────────────────────────────────────────────┘
```

Detail：

```text
0155167697-0113-1
Status
Timeline
Items
Fulfillment
Cancellation / Return
Finance
Source
```

## 41.9 Responsive

Narrow：

- Search 全宽；
- Order Table 保持 Table；
- 母订单号 / 订单号至少一个 Key Identity Sticky / visible；
- 次要字段进入 Horizontal Scroll / Detail；
- Timeline 纵向；
- Drawer 全宽。

## 41.10 Orders QA

至少覆盖：

```text
Mother Order with one Posting
Mother Order with multiple Postings
Leading-zero identifier
Long SKU
Pre-shipment cancellation
Post-shipment cancellation
Delivered + Return
Unknown currency
Partial finance relation
Search exact / no result
Large result set
390px Narrow
CN / EN / RU
```

核心验收：

```text
order_number
→ 母订单号

posting_number
→ 订单号

Cancellation
≠ Return
```

---

# 42. 库存

库存页回答：

```text
现在有多少库存？
哪些库存正在进入？
多久可能缺货？
哪些 SKU 需要关注或补货？
```

当前库存事实、在途库存与预测必须视觉和语义分离。

## 42.1 Primary Domains

页面可以使用稳定同级 Tabs：

```text
当前库存
历史
预测
```

因为三者属于同一库存主题下的不同稳定信息域。

切换 Tab 保持：

```text
Shop
Relevant SKU filter
Reporting Currency not applicable to pure quantity metrics
```

但每个 Domain 使用自己的 Date / Time Context。

## 42.2 Current Stock

Current Stock 展示 API / normalized inventory facts，例如：

```text
Sellable
Reserved when available
Warehouse / type breakdown
Observed at
```

当前值必须带 Snapshot / As-of 语义。

禁止把：

```text
Sellable + Inbound
```

显示成一个没有定义的“总可用库存”。

## 42.3 Inbound / In-transit

Inbound 是独立 Inventory Component。

正式原则：

```text
Current Stock
≠ Inbound Stock
```

页面应使用独立 Label / Column / Summary。

如果 Inbound 来自 Seller-owned Data 或其他来源，显示其独立 Freshness / Origin。

## 42.4 Inventory Risk and Forecast

派生能力可以包括：

```text
Current Days of Cover
Low Stock
Stockout Risk
Forecast Demand
Future Stockout Date
Forecasted Days of Cover
```

所有 Forecast：

```text
Origin = O3P_FORECAST
```

并明确与 Actual / Current Fact 分开。

没有足够近期需求时，使用正式 `NO_RECENT_DEMAND` 等状态，不把预测日销当 0。

## 42.5 Replenishment Recommendation

补货建议属于 Recommendation，不是 Inventory Fact。

可以显示：

```text
预计断货日期
建议补货时间
建议补货数量
Lead Time evidence
Coverage / model status
```

允许：

```text
查看依据
复制建议
模拟参数 when product contract supports local simulation
```

禁止：

```text
创建 Ozon 补货
修改 Ozon 库存
```

## 42.6 Inventory Table

典型列：

```text
Product / SKU
Current Sellable
Reserved
Inbound
Current DOC
Forecast Demand
Forecast Stockout Date
Risk / Status
As-of
```

具体列只在 Metric / Source Contract 支持时出现。

Numeric 右对齐，单位明确。

## 42.7 Historical Inventory

History 使用 Snapshot Time，不把当前值复制成历史事实。

Chart：

```text
Actual inventory snapshots
→ line / step-like treatment when appropriate
```

不得通过平滑曲线制造不存在的库存中间值。

Gap 必须可见。

## 42.8 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 库存                                                       │
│ [当前库存] [历史] [预测]                                   │
├────────────────────────────────────────────────────────────┤
│ 当前可售        低库存 SKU       缺货风险       在途库存     │
├────────────────────────────────────────────────────────────┤
│                    Inventory Coverage                      │
├──────────────────────────────┬─────────────────────────────┤
│ 缺货风险                     │ 库存结构                    │
├──────────────────────────────┴─────────────────────────────┤
│ Product / SKU │ Current │ Inbound │ DOC │ Forecast │ Status │
└────────────────────────────────────────────────────────────┘
```

## 42.9 Responsive

Narrow：

- Tabs 横向滚动；
- Summary 2/1 列；
- Chart 全宽；
- Inventory Table 横向滚动；
- `Current / Inbound / Forecast` 不因空间小合并；
- Forecast Label 保持可见。

## 42.10 Inventory QA

至少覆盖：

```text
Current zero stock
Inbound > 0
Reserved unavailable
Multiple warehouses
No recent demand
Forecast available / unavailable
Forecast gap
Recommendation available
Same SKU across shops
Large table
390px Narrow
```

核心验收：

```text
Current
≠ Inbound
≠ Forecast
≠ Recommendation
```

---

# 43. 履约

履约页回答：

```text
订单发货和配送是否及时？
延误发生在哪个环节、履约模式、仓库或物流渠道？
```

履约页面重点分析交付过程，不重新定义订单生命周期或取消率公式。

## 43.1 Fulfillment Scope

正式业务模式来自标准化 Mapping，可包括：

```text
FBP
realFBS
OGL
other verified modes
```

原始技术字段例如：

```text
FBO / FBS
schema
integration_type_flow
stock.type
```

仍保留为 Source Fact。

UI 不能仅根据 Endpoint Name 猜最终 Fulfillment Mode。

## 43.2 Timeliness Metrics

时效类 Metric 默认组合：

```text
P50
P90
Average
Sample Count
```

P50 为主要观察值。

典型分析：

```text
Created → Handover / Shipped
Handover / Shipped → Delivered
Order → Delivered total duration
On-time / Delayed when formally defined
```

每个 Metric 必须明确 Time Basis 与 Eligibility。

## 43.3 Primary Analysis

推荐：

```text
Timeliness Trend
```

并支持按：

```text
Fulfillment Mode
Warehouse
Logistics Channel / Provider
Product / SKU
```

进行分解。

不同分组比较必须有足够 Sample Count；低样本状态不得假装稳定结论。

## 43.4 Delay Diagnosis

延误诊断优先回答：

```text
Where is delay concentrated?
Which entities contribute most?
What stage is slow?
```

可以使用：

- Bar；
- Heatmap；
- Table；
- Order Detail。

不使用 Decorative Gauge。

## 43.5 Delayed Orders

Delayed Orders Table 可以包含：

```text
posting_number
Product / SKU
Fulfillment Mode
Warehouse / Channel
Relevant timestamps
Delay metric / state
Current lifecycle state
```

点击 Identifier 进入 Order Detail。

## 43.6 Cancellation Relationship

取消继续保持独立业务概念。

履约页可以展示与履约相关的取消风险 / Seller-responsible cancellation evidence，当正式数据支持时：

```text
Fulfillment risk
↔ Cancellation evidence
```

但：

```text
Cancellation Rate
≠ Delivery Delay Rate
≠ Return Rate
```

取消记录的主生命周期查看仍进入订单页。

## 43.7 Data Quality and Coverage

Timeliness 依赖多个时间点。

必须能够显示：

```text
Sample Count
Eligible Count
Coverage
Missing timestamp reason
PARTIAL
```

缺少签收时间不能静默按 0 小时计算。

Source Gap / Freshness 必须影响结果状态。

## 43.8 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 履约与物流                         30D · Comparison          │
├────────────────────────────────────────────────────────────┤
│ 发货 P50       配送 P50       P90       On-time Metric      │
│ Sample / Coverage                                           │
├────────────────────────────────────────────────────────────┤
│                       时效趋势                             │
├──────────────────────────────┬─────────────────────────────┤
│ Fulfillment Mode             │ Warehouse / Channel         │
├──────────────────────────────┴─────────────────────────────┤
│ Delayed Orders                                              │
└────────────────────────────────────────────────────────────┘
```

## 43.9 Responsive

Narrow：

- Timeliness Metric 2/1 列；
- Mode / Warehouse 分析纵向；
- Delayed Orders Table 保持 Horizontal Scroll；
- Timestamp 不压缩为不可理解短词；
- Sample / Coverage 保持可见。

## 43.10 Fulfillment QA

至少覆盖：

```text
FBP
realFBS
OGL
Unknown / unverified mode
Missing shipped time
Missing delivered time
Low sample
PARTIAL coverage
P50 / P90 correctness presentation
Delayed order detail
Cancellation relation
390px Narrow
```

核心验收：

```text
Technical mode
≠ Business fulfillment mode

Missing timestamp
≠ zero duration
```

---

# 44. 退货

退货页回答：

```text
哪些商品在退？
为什么退？
退货进展到哪里？
退货对库存、财务和商品质量意味着什么？
```

退货与取消严格分开。

## 44.1 Return Summary

Summary 可以包含正式 Metric：

```text
Return Count / Units
Product Return Rate
Return Amount when formally defined
Open / pending return cases
```

如果某项金额不具备可靠归属，不为了版式补一个估算数字。

## 44.2 Product Return Rate

产品退货率必须使用正式 Cohort Metric Contract。

页面必须显示：

```text
Value
Cohort / Time Basis discoverable
Status
Coverage when material
```

`OPEN_COHORT`：

```text
Current result may remain visible when contract permits
+
Cohort 尚未闭合
```

不能伪装成最终成熟率。

`NO_DENOMINATOR` 不能显示 0%。

## 44.3 Return Reasons

Return Reason Analysis：

- 使用来源可获得的原因；
- 标准化映射必须可追溯；
- Unknown / Missing Reason 与真实“其他”分开；
- 不把未分类原因静默归入 Others。

适合：

```text
Bar
Table
Topic / category breakdown
```

## 44.4 Reverse Logistics Lifecycle

正式生命周期关系：

```text
Original Order / Posting
↓
Return Case
↓
Reverse Logistics / Warehouse State
↓
Disposal / Resale / WHD when observed
↓
Possible secondary sale relation
```

WHD 不作为与 FBP / realFBS / OGL 平级的首次销售 Fulfillment Mode。

Timeline 只展示实际可获得事件。

## 44.5 Product / SKU Diagnosis

高退货 Product / SKU 分析可结合：

```text
Return Rate
Return Count
Reason structure
Buyer Feedback
Product content / quality evidence
Profit impact
```

这些是相关证据，不意味着因果关系已经被证明。

界面使用：

```text
Associated evidence
```

而不是武断文案：

```text
“差评导致退货”
```

除非正式分析 Contract 能支持。

## 44.6 Financial and Inventory Impact

退货对：

- Finance Accrual；
- Refund / return cost；
- Inventory return state；
- Profit；

的影响只展示已有正式关联。

不能把未匹配费用平均摊到 SKU 后表现成 Ozon 原始事实。

## 44.7 Return Table and Detail

典型列：

```text
Return ID / Case
Order / Posting
Product / SKU
Return State
Reason
Created / Completed time
Relevant amount / currency
Status / source
```

Detail：

```text
Identity
Current State
Timeline
Product / Order
Reason
Reverse Logistics
Finance / Inventory Relations
Source / Lineage
```

## 44.8 Data States

重点覆盖：

```text
OPEN_COHORT
PARTIAL
UNMATCHED order/product
Unavailable reason
Stale return state
Empty valid return period
```

不得统一叫“暂无退货数据”。

## 44.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 退货与逆向物流                       30D / Cohort Context    │
├────────────────────────────────────────────────────────────┤
│ 退货量       产品退货率       Open Cases       财务影响      │
├────────────────────────────────────────────────────────────┤
│                       Return Trend                         │
├──────────────────────────────┬─────────────────────────────┤
│ 退货原因                     │ 高退货 Product / SKU        │
├──────────────────────────────┴─────────────────────────────┤
│ Return / Reverse Logistics Table                          │
└────────────────────────────────────────────────────────────┘
```

## 44.10 Returns QA

至少覆盖：

```text
Closed cohort
Open cohort
No denominator
Return reason missing
Return-product unmatched
Return-order unmatched
WHD lifecycle relation
Finance relation partial
No returns = valid empty / zero distinction
Large table
390px Narrow
```

核心验收：

```text
Return
≠ Cancellation

Open Cohort
≠ final mature rate
```

---

# 45. 广告

广告页回答：

```text
广告花费产生了什么结果？
哪些 Campaign / SKU 存在效率或利润问题？
Performance 数据本身是否完整？
```

主要自动化来源为 Performance API。

## 45.1 Performance Freshness First

Performance 数据存在特定时间窗口和回补限制，因此页面必须把：

```text
Freshness
Coverage
Gap
Recoverability
```

作为一等信息。

如果 Campaign SKU Daily 等数据存在不可补回 / 回补受限 Gap：

```text
GAP
→ prominent near affected analysis
```

不能因为最新同步成功就显示成“广告数据正常”。

## 45.2 Summary Metrics

典型正式 Metric：

```text
Spend
Performance-attributed Orders / Revenue
ROAS
ACOS
CTR / other supported Performance metrics
```

每个值必须使用正式来源、币种、Time Basis 和 Scale。

`Performance Revenue` 不等于 Seller Analytics Revenue 或 Finance Gross Sales。

## 45.3 Trend

Advertising Trend 可以同时查看：

```text
Spend
Attributed result
Efficiency metric
```

不同 Unit 不使用 Dual Y Axis 默认叠加。

优先：

```text
aligned separate charts
small multiples
metric + chart
```

## 45.4 Campaign and SKU Analysis

页面支持两个核心 Grain：

```text
Campaign
Campaign + SKU
```

可以通过 Tabs / Segmented / Filter 选择真正合适的模式：

- 若切换的是同一数据主题的 Grain / View Mode，可使用 Segmented Control；
- 若进入稳定的 Campaign vs SKU Peer Analysis Domain，可使用 Tabs。

具体实现只能选择一种清晰语义，不能同一页面混用两套入口。

## 45.5 Campaign Table

典型列：

```text
Campaign
Status (read-only source state)
Spend
Orders
Attributed Revenue
ROAS
ACOS
CTR
Coverage / Gap
```

允许：

```text
Search
Filter
Sort
Open Detail
```

禁止：

```text
Edit bid
Edit budget
Pause campaign
Add product
```

## 45.6 Profit Relationship

结合 Profit 数据时，可以展示：

```text
Advertising Cost
Ad-attributed result
Ad-after-profit result
Profit ROAS / related formal derived metric
```

但必须明确：

```text
Performance attribution truth
≠ Finance settlement truth
≠ Profit derivation
```

缺少 Seller Cost / Finance 时，Profit-related result 使用 `PARTIAL / ESTIMATED / UNAVAILABLE` 等正式状态。

## 45.7 Recommendations

可以生成：

```text
低效率 Campaign
高 ACOS
低 ROAS
亏损 SKU
预算 / 出价建议
```

允许：

```text
查看依据
复制建议
模拟影响 when formally supported
```

禁止：

```text
应用预算
应用出价
暂停广告
```

## 45.8 Campaign Detail

Detail：

```text
Identity
Read-only campaign state
Time series
SKU contribution
Efficiency metrics
Profit relation
Coverage / Gap
Source / Lineage
```

来源数据不可用时保持 Campaign Identity 和其他可用信息，不整块清空。

## 45.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 广告                              Campaign / SKU · Date     │
│ Performance：Coverage / Gap when material                  │
├────────────────────────────────────────────────────────────┤
│ Spend          Result Metric      ROAS          ACOS        │
├────────────────────────────────────────────────────────────┤
│                    Advertising Trend                       │
├──────────────────────────────┬─────────────────────────────┤
│ Campaign Performance         │ SKU Performance             │
├──────────────────────────────┴─────────────────────────────┤
│ Search / Filters                                            │
│ Campaign Table                                               │
└────────────────────────────────────────────────────────────┘
```

## 45.10 Responsive

Narrow：

- Freshness / Gap 不隐藏；
- Trend 纵向；
- Campaign / SKU 控件保持明确；
- Table Horizontal Scroll；
- Money / Currency / Status 保留。

## 45.11 Advertising QA

至少覆盖：

```text
Campaign level
Campaign SKU level
Today / yesterday-limited dataset
Coverage gap
Latest sync success + historical gap
Mixed currency
Performance metric scale
Profit unavailable / estimated
Recommendation present
Read-only action audit
390px Narrow
```

核心验收：

```text
Latest sync success
≠ complete Performance coverage

Advertising analysis
→ read-only
```

---
# 46. 买家反馈

买家反馈页回答：

```text
买家在反馈什么？
哪些 Product / SKU 出现重复问题？
反馈是否与退货、评分或商品质量风险相关？
```

本章正式采用第 5 章 IA Label：

```text
buyer-feedback
→ 买家反馈
```

不再使用旧页面标题“客户声音”。

## 46.1 Capability and Availability

买家反馈能力依赖当前 Shop 的 Seller API 权限 / 订阅能力。

页面必须先表达：

```text
Available
Partial
Unavailable
```

如果 Reviews / Feedback 能力不可用：

```text
买家反馈不可用
当前店铺订阅或权限不包含该数据
```

不能显示：

```text
Reviews = 0
```

Sidebar 仍保持 `买家反馈` 导航项稳定存在。

## 46.2 Summary

在数据可用时，可以展示：

```text
Rating / Rating-related formal metric
Review Count
Negative Feedback Count / Rate when formally defined
Affected Products
```

每个 Metric 必须保留正式来源与状态。

Ozon Rating Source Fact 与 O3Pilot Derived Topic / Sentiment 分开。

## 46.3 Rating and Feedback Trend

Trend 可以分析：

```text
Rating change
Review / feedback volume
Negative feedback trend
```

Date / Time Basis 必须来自 Source / Metric Contract。

Rating 的历史事实不能被当前最新 Rating 覆盖。

## 46.4 Topics and Sentiment

O3Pilot 可以派生：

```text
High-frequency issues
Feedback topics
Quality problem clusters
Sentiment analysis
```

这些属于 O3Pilot Derived / Estimate 结果时必须明确 Origin / Status。

Topic / Sentiment 不是 Ozon 官方分类，不能通过视觉样式伪装成 Ozon Source Fact。

## 46.5 Feedback Records

根据当前可读取能力，Record Surface 可以包含：

```text
Reviews
Questions / Answers when available
Readable chat / conversation history when available
```

不同能力的 Availability 必须分别判断。

不能因为其中一个 API 无权限，就把其他已经可读的反馈数据一起隐藏。

记录至少显示：

```text
Date / Time
Product / SKU relation
Rating / category when available
Text
Source / status when material
```

## 46.6 Product Diagnosis

页面可以将反馈与：

```text
Product Return Rate
Return Reasons
Product Content Quality
Sales / Conversion
```

并列为诊断证据。

正式原则：

```text
Correlation / associated evidence
≠ proven causality
```

不得仅因同一商品同时差评高、退货高，就在 UI 直接写“差评导致退货”。

## 46.7 Reply Draft

允许 O3Pilot 生成：

```text
回复草稿
```

正式交互：

```text
Feedback record
↓
Generate Draft
↓
Review locally
↓
Copy
```

禁止：

```text
Send to Ozon
Auto Reply
Auto-submit review response
```

按钮文案必须是：

```text
生成回复草稿
复制草稿
```

而不是：

```text
回复
发送
```

以避免产生错误执行预期。

## 46.8 Data State

重点状态：

```text
UNAVAILABLE permission
PARTIAL coverage
STALE feedback snapshot
UNMATCHED product
Empty valid period
```

`UNMATCHED Product` 保留反馈记录本身，并显示映射问题，不把原始记录丢弃。

## 46.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 买家反馈                                                   │
│ Capability / Coverage when material                        │
├────────────────────────────────────────────────────────────┤
│ Rating        Reviews        Negative        Risk Products │
├──────────────────────────────┬─────────────────────────────┤
│ Rating / Feedback Trend      │ Feedback Topics             │
├──────────────────────────────┴─────────────────────────────┤
│ Search / Filters                                            │
│ Feedback Records                                            │
│ ...                                    [生成回复草稿]      │
└────────────────────────────────────────────────────────────┘
```

## 46.10 Buyer Feedback QA

至少覆盖：

```text
Capability available
Capability unavailable
Partial reviews
Review with unmatched product
Long RU feedback text
Negative / neutral / positive derived sentiment
Return-rate correlation view
Draft generation
Copy draft
No send action anywhere
390px Narrow
```

核心验收：

```text
Unavailable
≠ 0 reviews

Draft
→ local only
→ never auto-send to Ozon
```

---

# 47. 结算

结算页回答：

```text
Ozon 为什么产生这笔收入或费用？
当前结算由哪些 Finance Accrual 构成？
API、官方报告与付款事实是否能够对账？
```

本章正式采用第 5 章 IA Label：

```text
settlements
→ 结算
```

页面内部可以使用“财务事件”“应计”“费用”等业务术语，但不再以“财务”作为 Primary Navigation Label。

## 47.1 Money Contract

结算页必须严格区分：

```text
finance_gross_sales
finance_net_accrual
payout_amount
order_gross_value
buyer_paid_value
ozon.analytics.revenue
performance_orders_money
```

这些都不是同一个“销售额”。

每个 Money：

```text
amount
+
currency
```

跨币种汇总遵守第 20 章 Reporting Currency。

## 47.2 Settlement Summary

Summary 可以使用正式财务 Metric，例如：

```text
Finance Gross Sales
Finance Net Accrual
Commission
Logistics / Service Cost
Payout Amount when available
```

实际展示集合由可用数据与正式 Metric 决定。

不得为了页面固定四个 KPI 而用近似值补齐。

## 47.3 Finance Trend

趋势按正式：

```text
ACCRUAL_DATE / ACCRUAL_PERIOD / relevant Finance time basis
```

显示。

如果官方报告采用独立 Period / Business Date，保持其来源日期语义，不强行按北京时间重新划日。

## 47.4 Accrual Events

Finance Accrual Events 是结算页的核心证据表。

典型列：

```text
Accrual ID
Date / Period
Finance Type
Signed Amount
Currency
Order / Posting relation
Product / SKU relation when reliable
Source
Status
```

正负号必须保留正式 Signed Amount 语义。

费用与收入不通过颜色替代正负号和文本。

## 47.5 Income / Cost Structure

可以展示：

```text
Income Structure
Cost Structure
```

只将可以合法相加的 Finance Components 组合。

Waterfall 使用：

```text
signed additive components
→ reconcile to formal total
```

如果数据 `PARTIAL`，不能让 Waterfall 看起来完整闭合而不说明 Coverage。

## 47.6 Reconciliation

用户导入 Ozon Official Report 后，结算页可以提供：

```text
API fact
vs
Official Report fact
```

的 Reconciliation。

正式表达：

```text
Matched
Difference
Missing on API side
Missing on report side
Unverified
```

具体 Reconciliation 状态由数据模型 /算法 Contract 决定；DESIGN 只定义可追溯表达。

报告导入时间与报告覆盖 Period 分开显示。

## 47.7 Payout

如果数据源提供正式付款 / 计划付款事实，可以显示：

```text
payout_amount
payment / settlement period
source
```

但：

```text
Finance Net Accrual
≠ Payout Amount
```

不能把应计净额当成实际到账金额。

能力不可用时不伪造 Payout。

## 47.8 Allocation and Attribution

当 Ozon 只提供 Order-level Finance Cost，而没有可靠 SKU Attribution：

```text
Order-level fact
→ remains source truth
```

任何 O3Pilot SKU Allocation：

```text
→ derived / estimated
→ allocation rule discoverable
→ version discoverable
```

不得表现成 Ozon Source Fact。

## 47.9 Detail

Finance Accrual Detail：

```text
Identity
Signed Amount + Currency
Finance Type
Date / Period
Order / Posting relation
Product relation when available
Classification / Allocation when derived
Official report reconciliation when available
Source / Lineage
```

复杂 Source / Lineage 可以使用 640px Drawer 或 Full Page。

## 47.10 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 结算                              Date / Reporting Currency │
├────────────────────────────────────────────────────────────┤
│ Gross Sales     Net Accrual     Commission     Logistics   │
├────────────────────────────────────────────────────────────┤
│                       Finance Trend                        │
├──────────────────────────────┬─────────────────────────────┤
│ 收入结构                     │ 费用结构                    │
├──────────────────────────────┴─────────────────────────────┤
│ Reconciliation / Official Report State                    │
├────────────────────────────────────────────────────────────┤
│ Finance Accrual Events                                     │
└────────────────────────────────────────────────────────────┘
```

## 47.11 Settlement QA

至少覆盖：

```text
Positive / negative accrual
Multiple currencies
No Reporting Currency
Official report imported
Report / API mismatch
Order-level cost without SKU attribution
Derived allocation
PARTIAL coverage
Payout available / unavailable
Long finance type
390px Narrow
```

核心验收：

```text
Accrual
≠ Payout

Source fact
≠ derived allocation

Money name
→ formal contract only
```

---

# 48. 利润

利润页回答：

```text
真正赚了多少？
成本是否完整？
哪些 Product / SKU / Order 在贡献或侵蚀利润？
当前利润结果可信到什么程度？
```

Profit 是 O3Pilot Derived Capability，不是 Ozon 原始字段。

## 48.1 Trust and Coverage First

利润结果必须把数据完整度作为一等信息。

Page Header / Primary Summary 应能够显示：

```text
Coverage
Status
As-of
```

例如：

```text
成本覆盖 96.7%
PARTIAL
```

不能只显示一个精确到分的净利润，让用户误以为全部成本已完整归属。

## 48.2 Profit Summary

典型正式 Metric：

```text
Profit Revenue / formal revenue basis
Gross Profit
Net Profit
Profit Margin
Cost Coverage
```

具体定义完全服从 `METRICS.md`。

Profit Revenue 不与 Ozon Analytics Revenue、Finance Gross Sales 等混用名称。

## 48.3 P&L Waterfall

P&L Waterfall 必须：

```text
start from formal revenue basis
↓
apply signed additive cost / adjustment components
↓
reconcile to formal profit result
```

示例组件只在 Contract 支持时出现：

```text
Revenue
- Ozon Commission
- Logistics
- Advertising
- Seller Purchase Cost
- Other Costs
± Adjustments
= Profit
```

禁止：

- 重复计费；
- 不同币种未转换直接相加；
- 把 Ratio 放入 Waterfall；
- 让 Partial Waterfall 假装完整闭合。

## 48.4 Profit Trend

Trend 明确：

```text
Metric
Reporting Currency
Time Basis
Cost / FX Policy Version discoverable
```

历史 Profit 必须能够按正式版本重算，不能使用当前最新 Seller Cost 覆盖历史真实成本语义。

## 48.5 Cost Structure

Cost Structure 用于回答：

```text
利润主要被哪些成本消耗？
```

可按：

```text
Ozon charges
Advertising
Seller purchase cost
Seller logistics / other owned cost
```

分组，但每类来源与 FX Policy 必须可追溯。

`Ozon Business FX` 与 `Seller Cost FX` 分开。

## 48.6 Entity Profitability

支持 Grain：

```text
Shop
Seller Catalog Item
Product / SKU
Order / Posting / Item when reliable
Period
```

Table 典型列：

```text
Entity
Revenue Basis
Ozon Cost
Seller Cost
Ad Cost
Profit
Margin
Coverage
Status
```

不能把没有可靠 SKU Allocation 的 Order-level Finance Cost 静默平均分配后显示成 Source Fact。

## 48.7 Estimated Profit

当使用：

```text
reference cost
fallback cost
estimated logistics
incomplete finance allocation
```

结果必须进入正式：

```text
ESTIMATED / PARTIAL / other applicable status
```

`Estimated` 不自动等于 Warning，但必须可见。

Detail 应说明：

```text
Which component is estimated
Fallback rule
Coverage
Policy version
```

## 48.8 Missing Cost

缺少 Seller Cost 时：

```text
Missing Cost
≠ Zero Cost
```

利润不能静默变高。

页面应：

- 显示受影响 Entity；
- 显示 Cost Coverage；
- 提供进入 Seller Data / Mapping 的本地修复入口；
- 不把利润显示成完整 `VALID`。

## 48.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 利润                       Coverage 96.7% · Reporting CNY   │
├────────────────────────────────────────────────────────────┤
│ Revenue Basis    Gross Profit    Net Profit    Margin       │
├────────────────────────────────────────────────────────────┤
│                        P&L Waterfall                        │
├──────────────────────────────┬─────────────────────────────┤
│ Profit Trend                 │ Cost Structure              │
├──────────────────────────────┴─────────────────────────────┤
│ Search / Filters                                            │
│ Product / SKU / Order Profitability                        │
└────────────────────────────────────────────────────────────┘
```

## 48.10 Profit QA

至少覆盖：

```text
Full cost coverage
PARTIAL coverage
Estimated seller cost
Missing seller cost
Mixed currency
Ozon Business FX + Seller Cost FX
Historical cost version
Order-level finance without SKU allocation
Loss-making SKU
Zero profit
Negative profit
390px Narrow
```

核心验收：

```text
Missing Cost
≠ Zero Cost

Profit precision
≠ Profit trust

Coverage
→ first-class
```

---

# 49. 店铺健康

店铺健康页回答：

```text
哪些店铺指标正在接近风险？
风险来自哪里？
哪些是 Ozon 官方状态，哪些是 O3Pilot 自算信号？
```

## 49.1 Official vs Derived

必须严格区分：

```text
Ozon Official / Source Rating or Health Fact
≠ O3Pilot Derived Risk Metric
```

页面可以并列，但 Origin 必须可发现。

不能为了统一 Dashboard 把二者做成同一个“健康分”。

## 49.2 Summary

Summary 可以包含正式可用的：

```text
Service / Rating Metric
Delivery / Timeliness Risk
Cancellation Risk
Product Rating / Quality Risk
Other verified shop-health metrics
```

具体字段由当前 Ozon 数据能力决定。

不创建未经验证的综合 Score。

## 49.3 Thresholds

如果 Ozon Source /正式 O3Pilot Rule 提供阈值：

```text
Current Value
Threshold
Distance to Threshold
Origin
As-of
```

可以展示。

DESIGN 不定义新的业务阈值。

禁止为了视觉使用 Gauge 自行产生“安全区 / 危险区”。

## 49.4 Trend

Health Trend 必须区分：

```text
Historical Source Fact
Derived Metric History
```

当前值不可覆盖历史事实。

Stale / Gap 必须可见。

## 49.5 Risk Drivers

Risk Driver 可以链接：

```text
Fulfillment Mode
Warehouse / Logistics
Cancellation
Product / SKU
Buyer Feedback
Return
```

只显示已建立正式关联的证据。

## 49.6 Multi-shop

跨店 Health View 可以展示：

```text
Shop
Metric
Value
Status
Origin
As-of
```

不同 Shop Capability 不可用时不从比较表静默删除。

```text
UNAVAILABLE
→ remains visible with reason
```

## 49.7 Data State

重点状态：

```text
UNAVAILABLE
UNVERIFIED
PARTIAL
STALE
Known healthy / known risk
```

`VALID` 正常状态保持 Quiet；不把整个页面铺成绿色。

## 49.8 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 店铺健康                                                   │
├────────────────────────────────────────────────────────────┤
│ Service      Delivery      Cancellation      Rating         │
│ Origin / Status when needed                                │
├────────────────────────────────────────────────────────────┤
│                       Health Trend                         │
├──────────────────────────────┬─────────────────────────────┤
│ 接近阈值                     │ 风险来源                    │
├──────────────────────────────┴─────────────────────────────┤
│ Related Products / Fulfillment / Logistics Evidence        │
└────────────────────────────────────────────────────────────┘
```

## 49.9 Shop Health QA

至少覆盖：

```text
Official metric
O3Pilot derived metric
Near threshold
No formal threshold
UNAVAILABLE shop capability
STALE source
Multi-shop comparison
Normal all-clear state
Risk state
390px Narrow
```

核心验收：

```text
Official
≠ Derived

No validated threshold
→ no invented gauge boundary
```

---

# 50. 预警

预警页是 O3Pilot 的经营 Attention Inbox。

本章正式采用第 5 章 IA Label：

```text
alerts
→ 预警
```

它回答：

```text
什么事情现在值得关注？
为什么触发？
相关证据是什么？
当前是否已经处理 / 解决？
```

## 50.1 Alert Domain Boundary

Alert 是正式 Derived Entity，引用其他域事实，不复制或重新定义业务事实。

正式关系：

```text
Metric / Data Quality / Business Fact
↓
Alert Rule
↓
Alert
```

DESIGN 不定义 Alert Threshold 或 Rule Formula。

## 50.2 Inbox Layout

预警页优先使用 List / Table + Detail Pattern，而不是大量 Alert Card 墙。

标准结构：

```text
Summary / counts
↓
Search / Filters
↓
Alert Inbox
↓
Detail
```

Alert Count 必须明确当前 Query / Status Scope。

Unknown Total 时不伪造精确数量。

## 50.3 Priority and Severity

Alert Entity 可以拥有 `severity`。

UI 将正式 Severity 映射到 Semantic Status Color，但：

```text
Severity code
→ comes from Alert Contract

Color
→ presentation only
```

如果后端没有某种 Severity，不由 DESIGN 另行创造 `Critical / Warning / Info` 业务枚举。

正常普通记录不使用 Danger Red。

## 50.4 Filters

可按正式字段过滤：

```text
Status
Severity
Domain / alert_type
Shop
Entity type
Time
```

`Shop` 若已经是 Global Context，则不重复作为普通 Filter；跨店 Inbox 才显示 Shop Dimension。

Applied Filter 必须可发现。

## 50.5 Alert Detail

统一结构：

```text
Alert Identity / Type
Current Status
Affected Entity
Triggered At
Why / Explanation
Metric Value
Threshold Value when formally defined
Evidence
Related Entities
Rule Version
Source / Lineage
```

`Why` 必须基于 Alert `details / explanation`，不能 UI 临时猜原因。

## 50.6 Acknowledge / Local State

允许改变：

```text
O3Pilot Alert local state
```

例如：

```text
标记已处理
确认已查看
```

前提是这些状态存在于正式 Alert Contract。

禁止：

```text
Acknowledge alert
→ mutate Ozon
```

并且：

```text
Notification shown
≠ Alert acknowledged

Dismiss UI
≠ Alert resolved
```

## 50.7 Resolved vs Acknowledged

如果 Alert Domain 区分：

```text
acknowledged
resolved
```

UI 必须分别表达。

例如：

```text
已处理
→ user workflow state

已解决
→ underlying condition no longer active / formal alert state
```

DESIGN 不自行合并这两个语义。

## 50.8 Notifications

新 Alert 可以通过 O3Pilot 自身 Notification Delivery 提醒，但：

```text
Alert
≠ Notification
```

Notification 只负责 Delivery / Presentation。

Alert 仍在预警 Inbox 中保持正式状态。

## 50.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 预警                              未处理 / 当前 Filter Scope│
├────────────────────────────────────────────────────────────┤
│ Search / Severity / Domain / Status Filters                │
├─────────────────────────┬──────────────────────────────────┤
│ Alert Inbox             │ Alert Detail                     │
│                         │                                  │
│ Inventory Risk          │ Affected Entity                  │
│ Returns                 │ Why                              │
│ Finance                 │ Metric / Threshold               │
│ Advertising             │ Evidence                         │
│ Data Quality            │ Related Entities                 │
│                         │                                  │
│                         │ [标记已处理]                     │
└─────────────────────────┴──────────────────────────────────┘
```

Compact / Narrow 使用 List → Full-width Detail，不强制双栏。

## 50.10 Alerts QA

至少覆盖：

```text
Multiple severities
Unknown severity mapping
Unacknowledged
Acknowledged
Resolved
Alert with threshold
Alert without threshold
Alert with stale evidence
Data-quality alert
Same entity multiple alerts
Multi-shop
Notification shown but not acknowledged
390px Narrow
Keyboard
```

核心验收：

```text
Alert
→ business attention object

Notification
→ delivery only

Local acknowledgement
→ never Ozon mutation
```

---

# 51. 经营建议

经营建议页回答：

```text
接下来最值得考虑什么经营动作？
为什么？
证据和可信度如何？
```

O3Pilot 提供 Recommendation、Forecast、Simulation 和 Copy，不直接执行 Ozon 写操作。

## 51.1 Recommendation Domains

正式 Recommendation 类型可以包括：

```text
补货
清库存
定价
广告
商品优化
```

内部对应正式 Recommendation Type，例如：

```text
REPLENISHMENT
CLEARANCE
PRICE
AD_BID / AD_BUDGET
PRODUCT_OPTIMIZATION
```

页面可使用本地化 Tabs：

```text
[补货] [清库存] [定价] [广告] [商品优化] [预测]
```

其中：

```text
预测
→ Forecast evidence domain
→ not another Recommendation type by default
```

避免把 Forecast Entity 与 Recommendation Entity 混成一个状态模型。

## 51.2 Ranking and Priority

Recommendation 可以按正式 `priority`、Impact 或其他已定义字段排序。

DESIGN 不自行计算业务优先级。

如果 Priority 不可用：

```text
→ do not invent High / Medium / Low
```

排序依据必须可发现。

## 51.3 Recommendation Presentation

标准信息结构：

```text
Entity
Recommendation summary
Expected / relevant outcome when formally available
Why
Evidence
Status / Coverage
Priority when formal
Generated At / Valid Until when relevant
[查看依据] [复制建议]
```

Recommendation Surface 不要求大 Card；高密度场景可以使用 List / Table + Detail。

## 51.4 Evidence and Traceability

用户必须能够从 Recommendation 继续查看：

```text
Input snapshot
Related metrics
Coverage
Source / Origin
Model / rule version
Generated At
Valid Until when defined
```

正式原则：

```text
Recommendation
→ explainable
→ reproducible to its input snapshot
```

不能只显示：

```text
建议补货 42 件
```

而没有依据入口。

## 51.5 Forecast as Evidence

补货、缺货、清库存等建议可以使用 Forecast，但必须明确：

```text
Actual
≠ Forecast
≠ Recommendation
```

Forecast View 展示：

- Forecast Demand；
- Future Stockout Date；
- Backtest / Accuracy when formally available；
- Model Version；
- Status / Coverage。

没有正式 Confidence Interval 时不画假的 Confidence Band。

## 51.6 Simulation

如果产品正式支持本地 Simulation，可以允许用户修改：

```text
Lead Time
Target Coverage
Price scenario
Cost assumption
Ad assumption
```

用于 O3Pilot 本地分析。

Simulation 必须清楚标记：

```text
Scenario / simulated
≠ current Ozon state
```

Simulation 结果不得自动保存成 Ozon State 或 Source Fact。

## 51.7 Copy

Copy 应输出可复用建议文本或关键参数。

Copy 完成可以使用：

```text
Copy → Check
```

单一反馈足够时不再额外 Toast。

复制内容必须保留：

- Entity identity；
- Recommendation；
- 关键数值 / Unit；
- 必要的状态 /限制。

## 51.8 Read-only Boundary

正式禁止：

```text
执行建议
立即补货
应用价格
修改预算
修改出价
修改库存
自动优化商品
```

允许：

```text
查看依据
复制建议
模拟
打开相关页面
```

正式原则：

```text
Recommendation
→ decision support
not execution surface
```

## 51.9 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 经营建议                                                   │
├────────────────────────────────────────────────────────────┤
│ 补货 | 清库存 | 定价 | 广告 | 商品优化 | 预测               │
├────────────────────────────────────────────────────────────┤
│ Product / SKU                                              │
│ 预计 9 天后断货                                            │
│ 建议补货 42 件                                             │
│ Coverage / Status / Priority                               │
│ Why / Evidence                                             │
│                              [查看依据] [复制建议]          │
├────────────────────────────────────────────────────────────┤
│ More recommendations / evidence                            │
└────────────────────────────────────────────────────────────┘
```

## 51.10 Recommendations QA

至少覆盖：

```text
Replenishment
Clearance
Price
Advertising
Product optimization
Forecast evidence
Recommendation without priority
PARTIAL input
ESTIMATED input
Expired recommendation when valid_until exists
Simulation
Copy
Read-only action audit
390px Narrow
```

核心验收：

```text
Actual
≠ Forecast
≠ Recommendation
≠ Simulation

No Execute-to-Ozon action
```

---

# 52. 数据中心

数据中心回答：

```text
O3Pilot 的数据现在是否可信？
数据从哪里来？
哪里存在 Coverage Gap？
后台任务在做什么？
导入与 Lineage 是否可追溯？
```

数据中心是 Data Operations / Trust Workspace，不是普通业务 Dashboard。

## 52.1 Stable Domains

正式 Tabs：

```text
来源
覆盖
质量
任务
导入
血缘
```

内部 Semantic ID：

```text
Sources
Coverage
Quality
Jobs
Imports
Lineage
```

这些是稳定同级信息域，适合 Tabs。

## 52.2 Sources

Sources 展示 Dataset / Source 的：

```text
Source / Dataset
Shop
Freshness Presentation State
Last Attempt
Last Success
Coverage Through
Known Gap
Mode: Automatic / Manual
```

需要时进一步查看：

```text
Endpoint
Sync Run
Backfill capability
Failure reason
```

使用第 19 章：

```text
CURRENT
DELAYED
STALE
GAP
FAILED
UNKNOWN
```

作为 UI Freshness Presentation，不把 `Healthy` 当新的正式统一数据状态。

## 52.3 Coverage

Coverage Table：

```text
Dataset
Shop
Requested / Expected Window when formal
Actual Coverage From
Actual Coverage To
Gap
Recoverability
Last Success
```

重点：

```text
Latest sync
≠ complete historical coverage
```

不可补回 / 回补受限 Gap 提升视觉优先级。

Manual Source 显示：

```text
Import Time
Coverage Through
```

而不是只有文件日期。

## 52.4 Quality

Quality 展示正式 `data_quality_issue`：

```text
Issue Type
Severity
Entity
Source
Detected At
Status
Resolved At when applicable
```

典型 Issue Type 可直接使用 Data Model，例如：

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

主界面提供本地化名称，技术 Code 在 Detail 可查看。

## 52.5 Jobs

Jobs 直接服从 Persistent Job Contract。

正式状态至少可能包括：

```text
PENDING
RUNNING
RETRY_WAIT
SUCCEEDED
FAILED
CANCELLED
INTERRUPTED / ORPHANED
```

UI 不自行改写成另一套长期 Job 状态机。

Job Row 至少可查看：

```text
Job Type
Shop
Status
Priority Class
Created / Scheduled
Started / Finished
Attempt
Next Retry when relevant
Error when failed
Related Run / Result
```

页面离开不改变 Job Lifetime。

## 52.6 Imports

Imports 支持：

```text
Ozon 官方报告
卖家自有数据
```

标准 UX：

```text
Select File
↓
Detect / Parse
↓
Validate
↓
Mapping when required
↓
Preview
↓
Confirm Import
↓
Persist structured facts / metadata / errors
↓
Delete original upload according to security / deployment contract
```

正式原则：

```text
Selected
≠ Validated
≠ Parsed
≠ Imported
```

Preview 必须明确显示：

- 识别的数据类型；
- 预计写入 / 更新的 O3Pilot Local Data Scope；
- Invalid Rows / Mapping Issues；
- Currency / Date / Identifier Problems；
- 是否会覆盖 O3Pilot 自有现有记录（如果该 Import Contract 允许）。

Import 只写 O3Pilot Local State，不修改 Ozon。

## 52.7 Import History

Import History 显示：

```text
Import Batch
Type
Imported At
Coverage
Rows / result summary
Status
Errors
```

成功导入后 UI 不暗示原始 CSV / XLS / XLSX 长期保存。

原文件 Download / Re-open 按钮不得存在，除非未来 Deployment Contract 明确改变文件保留策略。

## 52.8 Lineage

Lineage 用于从 Derived Result 追溯：

```text
Metric / Forecast / Profit / Alert / Recommendation
↓
Calculation / Rule / Model Version
↓
Normalized Facts
↓
Source Object / Raw Payload
↓
Sync / Import Run
```

UI 不要求一次展示完整 Graph。

可以使用：

```text
progressive disclosure
related entity links
structured source detail
```

复杂 Lineage 使用 Full Page 或 Wide Detail，不塞进小 Tooltip。

## 52.9 Multi-shop

Data Center 默认尊重当前 Shop Context，并支持跨店 Overview。

跨店 Overview：

```text
Shop
Source / Dataset
State
Coverage
Gap
```

一个 Shop 的 Gap 不得被其他 Shop 的正常状态平均掉。

## 52.10 Data Center Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 数据中心                                                   │
├────────────────────────────────────────────────────────────┤
│ 来源 | 覆盖 | 质量 | 任务 | 导入 | 血缘                     │
├────────────────────────────────────────────────────────────┤
│ Dataset / Source       State       Coverage      Last Success│
│ Seller API             CURRENT     ...           10:28       │
│ Performance Dataset    GAP         ...           10:22       │
│ Webhook                CURRENT     ...           10:31       │
│ Seller Cost            Manual      Aug 31        Sep 03      │
└────────────────────────────────────────────────────────────┘
```

Imports：

```text
[Ozon 官方报告] [卖家数据]

Select / Drop File
↓
Validate
↓
Preview
↓
Import

Import History
```

## 52.11 Responsive and Performance

Narrow：

- Tabs 横向滚动；
- Sources / Coverage / Jobs 使用 Table Horizontal Scroll；
- Job / Issue Detail 全宽；
- Import Step 纵向；
- 不隐藏 Error Code / Identifier 的完整获取能力。

Jobs / Logs / Issues 大数据使用 Pagination / Virtualization 等技术时，保持正式 Table / Accessibility Contract。

## 52.12 Data Center QA

至少覆盖：

```text
CURRENT source
DELAYED source
GAP source
FAILED source
Manual source
Unknown source state
Multi-shop mixed health
Quality issues
Job pending/running/retry/failed/interrupted
Import valid
Import invalid
Partial row errors
Original file removed after import
Lineage from metric to source
390px Narrow
```

核心验收：

```text
Data Center
→ trust and operations

Drive development references
≠ runtime data source

Import file
→ temporary input
```

---

# 53. 系统设置

系统设置管理 O3Pilot 自身的 Local Configuration、Integration Credential、Backup、Appearance 与 System Status。

它不是 Ozon Seller Center 的远程控制面板。

正式结构：

```text
Shops
Credentials
Webhook
Notifications
Backup
Appearance
System
```

## 53.1 Settings Architecture

Desktop 可以使用 Settings Local Navigation：

```text
┌──────────────────┬─────────────────────────────────────────┐
│ Shops            │ Section Title                           │
│ Credentials      │ Description                             │
│ Webhook          │                                         │
│ Notifications    │ Form / Status                           │
│ Backup           │                                         │
│ Appearance       │                                         │
│ System           │                                         │
└──────────────────┴─────────────────────────────────────────┘
```

这是 Page-local Settings Navigation，不进入 Primary Sidebar。

每个 Section 默认独立 Form / Commit Model。

## 53.2 Shops

Shops 显示：

```text
Display Name
Client / Shop identifier when safe
Country / settlement metadata when available
Capability summary
Connection / data state
```

允许管理 O3Pilot 对 Shop 的本地连接配置。

新增 / 删除 Shop Configuration 只影响 O3Pilot Local State，不调用 Ozon 创建 / 删除 Seller Account。

删除本地 Shop Configuration 属于高风险 Local Destructive Action，必须使用明确 Confirmation，并说明本地数据影响范围。

## 53.3 Credentials

Credentials 按 Provider / Shop 分组。

显示：

```text
Configured
Valid / Invalid / Unverified
Last Verified
Expires when known
Capability / Provider
Masked identifier when safe
O3Pilot access: READ ONLY
```

Stored Secret：

```text
status / mask only
→ replace, not reveal
→ no Copy
```

Credential Test：

```text
→ approved read-only allowlisted endpoint only
```

即使 Credential 拥有 Ozon Write Permission：

```text
O3Pilot access
→ READ ONLY
```

## 53.4 Webhook

Webhook Settings 可以显示：

```text
O3Pilot Webhook Endpoint
Local Webhook Secret configured state
Last received event
Authentication / validation state
Readback / Reconciliation state
Manual setup instructions
```

允许 O3Pilot 管理自身：

```text
Webhook Endpoint Secret
```

的生成 / Rotation Flow，服从 Security Contract。

但：

```text
O3Pilot
→ never creates / updates / enables / deletes Ozon Webhook Subscription
```

如果 Secret Rotation 需要用户同步更新 Ozon Seller-side 配置，UI 只提供明确人工步骤。

## 53.5 Notifications

Notifications 只管理 O3Pilot 自身提醒 Delivery。

可配置的 Channel 必须来自产品正式支持能力，例如未来：

```text
DingTalk
Email
Outbound Webhook
```

未实现的 Channel 不以 Disabled Placeholder 冒充已支持功能。

Notification Setting 可以决定：

```text
Channel
Enabled state
Scope / alert types when supported
Delivery status
```

通知内容不得暴露 Secret / Recovery Material。

## 53.6 Backup

Backup 显示：

```text
Local Backup State
Last Successful Backup
Last Verified Restore Point
Backup Repository State
R2 Replica Configured / Not Configured
R2 Last Replica Result
Restore Readiness
```

正式关系：

```text
Local Backup Success
≠ R2 Replica Success
```

R2 未配置：

```text
→ valid supported state
→ not an error
```

Backup / Restore 属于 Persistent Job / Security-sensitive Workflow。

普通 Settings 页面不显示完整 Recovery Key。

## 53.7 Recovery Setup

如果 Security Contract 要求 Recovery Key Setup：

```text
Dedicated sensitive flow
→ explicit re-auth when required
→ show full Recovery Key only at allowed setup / rotation moment
→ require user to save outside host
```

普通 Backup Summary 只显示：

```text
Recovery readiness / configured state
```

而不是 Key 本身。

## 53.8 Appearance

正式支持：

```text
System
Light
Dark
```

Appearance 属于 O3Pilot Local Preference。

Theme 切换立即、可逆、低风险，可以使用 Immediate Local Apply。

Theme 不改变：

- Metric Status；
- Risk Severity；
- Chart meaning；
- Read-only Boundary。

未来其他纯表现偏好只有在改善重复工作时增加。

## 53.9 System

System 显示可诊断但不泄密的信息，例如：

```text
O3Pilot Version
Platform
Supported / unsupported deployment state when known
Service Status
Database / Raw / Backup size summary
Log size summary
Update availability only when formally implemented
```

可以链接：

```text
Data Center
Logs / Diagnostics
Deployment instructions
```

不能把：

```text
Cloudflare Tunnel Config
systemd / launchd raw credential
Secret file content
```

直接暴露在普通页面。

`cloudflared` 是独立外部服务；O3Pilot 可以显示 Remote Access 状态 / 指引，但不负责其完整配置生命周期。

## 53.10 Write and Security Boundary

系统设置可以写：

```text
O3Pilot Local State
Integration Secret Store
Local Notification Config
Backup Config
Appearance
```

但永远不写：

```text
Ozon product
price
inventory
order
fulfillment
advertising
promotion
Ozon webhook subscription
buyer communication
```

State-changing Settings Form 必须有 CSRF / Session 等 Security Contract 对应实现，DESIGN 只负责可见交互。

## 53.11 Wireframe

```text
┌────────────────────────────────────────────────────────────┐
│ 系统设置                                                   │
├──────────────────┬─────────────────────────────────────────┤
│ Shops            │ Credentials                             │
│ Credentials      │ Seller API                              │
│ Webhook          │ 已配置 · Last Verified 10:28           │
│ Notifications    │ O3Pilot access: READ ONLY              │
│ Backup           │                                         │
│ Appearance       │ [替换凭证] [验证连接]                   │
│ System           │                                         │
└──────────────────┴─────────────────────────────────────────┘
```

## 53.12 Responsive and Accessibility

Narrow：

```text
Settings local nav
→ compact list / select-like navigation with explicit current section

Form
→ single column
```

要求：

- Secret Field 可 Password Manager / Paste；
- Stored Secret 不 Reveal；
- Validation 靠近 Field；
- Destructive Local Action 与 Save 分离；
- 44px Touch Target；
- 200% Zoom；
- CN / EN / RU 长说明正常。

## 53.13 Settings QA

至少覆盖：

```text
Multiple shops
Credential configured / invalid / unverified
Credential replace
Read-only validation call
Webhook secret configured / rotated
No Ozon subscription mutation
Notifications unavailable / configured
Local backup success
R2 not configured
R2 replica failed but local backup succeeded
Recovery readiness
System / Light / Dark
Supported macOS / Linux metadata
390px Narrow
Keyboard
```

核心验收：

```text
Settings
→ O3Pilot control plane
not Ozon control plane
```

---
# 54. Page-specific Detail Pattern

所有核心实体详情优先复用统一 Detail Contract。

适用：

```text
Product
Order / Posting
Campaign
Alert
Finance Accrual
Return Case
Recommendation
Data Quality Issue
Import Batch
Job / Run
```

核心原则：

```text
Entity changes
→ content changes

Detail interaction model
→ stays consistent
```

## 54.1 Detail Structure

统一结构：

```text
Identity
↓
Current State
↓
Primary Facts
↓
Timeline / History when meaningful
↓
Related Entities
↓
Metrics / Analysis
↓
Source / Lineage
```

不是每个实体必须机械拥有全部区块；不存在业务意义的区块不展示。

## 54.2 Identity

Header 至少能够明确：

```text
Entity Type
Primary Identifier / Name
Shop when needed
Current high-level status when meaningful
```

关键 Identifier：

- 不因窄屏丢失完整获取能力；
- 可复制；
- 不 Locale-number-format；
- 多店同值时 Shop Context 可见。

## 54.3 Current State

Current State 只展示正式 Source / Normalized / Derived 状态。

例如：

```text
Ozon source state
O3Pilot risk state
Metric status
Freshness
```

必须分层，不把所有状态压成一个 Badge。

`Last Known Data` 要明确其 Last Valid Time。

## 54.4 Timeline and History

Timeline 只展示真实观察到或正式标准化的事件。

```text
Observed event
→ timeline node

Expected but unobserved event
→ not fabricated
```

历史快照与当前状态分开。

复杂历史可以使用专用 Tab / Section。

## 54.5 Related Entities

Related Entity Link 用于跨域追溯，例如：

```text
Order → Product
Order → Return
Order → Finance Accrual
Product → Alert
Product → Recommendation
Campaign → SKU
Quality Issue → Source / Entity
```

链接必须保留 Primary Navigation Context 的可理解性。

不得无限打开 Nested Drawer。

## 54.6 Metrics and Analysis

Detail 中的 Metric 继续使用第 18–22 章：

```text
Formal Metric Name
Value / Unit / Currency
Status
Coverage
As-of
Origin
Time Basis
```

Traceability 可以更深，但不要求默认铺满技术字段。

## 54.7 Source and Lineage

Source / Lineage 至少能够继续追溯：

```text
Source System
Endpoint / Dataset when relevant
Source Object Key
Observed At
Business Time
Sync / Import Run
Raw Payload reference when allowed
Parser / Mapping / Metric Version when relevant
```

Secret / Credential 明文永远不进入 Lineage UI。

## 54.8 Container Selection

容器选择：

```text
Quick detailed inspection
→ 480px Drawer

Complex multi-column / lineage inspection
→ 640px Drawer

Deep analysis / shareable / multi-domain workflow
→ Full Page
```

Popover 不用于 Entity Full Detail。

Modal 不用于只读长详情。

## 54.9 Route and Back State

重要 Detail State 可以进入可恢复 Route / Query State。

目标：

```text
List
→ open detail
→ inspect
→ close / back
→ same analytical position
```

尽量保留：

- Filter；
- Sort；
- Search；
- Pagination；
- Scroll Position；
- Page Date Context；
- Shop Context。

Transient Popover / Tooltip 不进入 Route State。

## 54.10 Actions

Detail 中允许：

```text
Copy
Open related entity
Export local view when supported
O3Pilot local state action when formally allowed
```

禁止：

```text
Ozon write action
```

例如 Order Detail 不出现 Cancel Order；Campaign Detail 不出现 Edit Bid；Buyer Feedback Detail 不出现 Send Reply。

## 54.11 Responsive and Accessibility

Narrow：

```text
Drawer
→ full-width detail surface
```

要求：

- Identity 始终可见；
- Detail Tabs 可横向滚动；
- Table / Timeline 使用自己的 Responsive Contract；
- Focus 进入并返回正确；
- Heading 结构清晰；
- Copy Action 可键盘；
- 200% Zoom；
- CN / EN / RU 长内容。

## 54.12 Detail QA

至少抽样：

```text
Product
Order
Campaign
Alert
Finance Accrual
Return Case
Data Quality Issue
```

检查：

```text
Identity
Current State
History
Related Entities
Metrics
Source / Lineage
Back-state preservation
Drawer / Full Page decision
Read-only boundary
390px Narrow
Keyboard
```

核心验收：

```text
One detail interaction language
across business domains
```

---

# 55. Design Tokens

Design Token 是 O3Pilot 设计 Contract 的实现层接口。

Token 必须表达语义，不把任意 CSS 值散落到 Feature。

正式原则：

```text
Semantic token
>
raw value in feature code
```

如果本章示例与前面专项章节发生冲突，以对应专项章节的正式规则为准，并必须在同一版本中修正冲突，不允许长期双重定义。

## 55.1 Token Namespaces

正式 Namespace 至少包括：

```text
font.*
type.*
color.*
space.*
radius.*
size.*
border.*
shadow.*
motion.*
ease.*
z.*
chart.*
status.*
```

Feature 级代码应通过 Semantic Alias 使用，而不是依赖“第 3 个蓝色”。

## 55.2 Typography Tokens

正式字体：

```css
--font-sans: "MiSans Global", system-ui, sans-serif;
```

字重：

```css
--font-weight-regular: 400;
--font-weight-medium: 500;
--font-weight-semibold: 600;
--font-weight-bold: 700;
```

正式 Type Scale：

```css
--type-page-title: 28px;
--type-section-title: 22px;
--type-panel-title: 17px;
--type-body: 15px;
--type-small: 13px;
--type-micro: 12px;
--type-metric-xl: 32px;
--type-metric-lg: 24px;
--type-metric-md: 17px;
```

实际 Line Height / Weight 使用第 8 章语义映射，不通过 Feature 重定义。

Numeric Comparison 使用：

```css
font-variant-numeric: tabular-nums;
```

Identifier 不使用 locale numeric formatting。

## 55.3 Color Tokens

Light Theme 核心 Token：

```css
--color-action: #0066CC;
--color-action-hover: #0071E3;
--color-action-pressed: #0055B3;
--color-action-on-dark: #2997FF;
--color-on-action: #FFFFFF;
--color-focus: #0071E3;

--color-canvas: #FFFFFF;
--color-canvas-secondary: #F5F5F7;
--color-surface: #FFFFFF;
--color-surface-subtle: #FAFAFC;
--color-ink: #1D1D1F;
--color-text-secondary: #6E6E73;
--color-text-tertiary: #86868B;
--color-hairline: rgba(0,0,0,0.08);
--color-divider-soft: rgba(0,0,0,0.05);
--color-control-hover: rgba(0,0,0,0.04);
--color-selected-surface: rgba(0,102,204,0.08);
--color-disabled-text: rgba(29,29,31,0.38);
--color-disabled-surface: rgba(0,0,0,0.04);
--color-overlay: rgba(0,0,0,0.28);
```

Dark Theme：

```css
--color-canvas: #000000;
--color-canvas-secondary: #1D1D1F;
--color-surface: #1C1C1E;
--color-surface-subtle: #242426;
--color-ink: #F5F5F7;
--color-text-secondary: #A1A1A6;
--color-text-tertiary: #8E8E93;
--color-hairline: rgba(255,255,255,0.12);
--color-divider-soft: rgba(255,255,255,0.08);
--color-control-hover: rgba(255,255,255,0.06);
--color-selected-surface: rgba(41,151,255,0.14);
--color-disabled-text: rgba(245,245,247,0.38);
--color-disabled-surface: rgba(255,255,255,0.06);
--color-overlay: rgba(0,0,0,0.48);
```

Dark Action / Status 的具体映射继续使用第 9 章，不通过数学反转生成。

## 55.4 Semantic Status Tokens

正式 Namespace：

```text
color.status.success.*
color.status.warning.*
color.status.danger.*
color.status.information.*
color.status.neutral.*
```

每个至少包含：

```text
foreground
subtle-background
indicator / border
```

Light / Dark 的精确值使用第 9.4 节。

正式原则：

```text
Normal
→ neutral / quiet

Attention
→ semantic color
```

不得为 `VALID` 建立全站高饱和绿色默认样式。

## 55.5 Spacing Tokens

正式 Spacing：

```css
--space-1: 4px;
--space-2: 8px;
--space-3: 12px;
--space-4: 16px;
--space-5: 20px;
--space-6: 24px;
--space-8: 32px;
--space-10: 40px;
--space-12: 48px;
--space-16: 64px;
```

禁止 Feature 创建未批准视觉 Spacing：

```text
6px / 10px / 14px / 18px / 22px / 28px / 30px
```

除非未来正式 Token Contract 修改。

## 55.6 Radius Tokens

正式：

```css
--radius-xs: 6px;
--radius-sm: 8px;
--radius-md: 10px;
--radius-lg: 12px;
--radius-xl: 16px;
--radius-pill: 999px;
```

Role Mapping 使用第 11 章。

不建立：

```text
7px / 13px / 20px / 24px
```

等任意 Radius。

## 55.7 Size Tokens

核心 Layout / Control Size：

```css
--size-context-bar: 64px;

--size-sidebar-expanded: 248px;
--size-sidebar-collapsed: 72px;
--size-sidebar-narrow: 60px;

--size-control-compact: 32px;
--size-control-standard: 40px;
--size-touch-target: 44px;

--size-table-header: 40px;
--size-table-row-standard: 44px;
--size-table-row-compact: 36px;

--size-drawer-standard: 480px;
--size-drawer-wide: 640px;
```

Icon：

```css
--size-icon-sm: 16px;
--size-icon-md: 18px;
--size-icon-lg: 20px;
--size-icon-xl: 24px;
```

Responsive 仍允许容器成为 Fluid / Full-width；Token 定义的是正式角色基线。

## 55.8 Border and Shadow

Border：

```css
--border-hairline: 1px solid var(--color-hairline);
--border-divider: 1px solid var(--color-divider-soft);
```

Normal Surface 默认：

```text
shadow-none
```

Overlay 才允许使用克制 Shadow Token，例如：

```text
shadow-popover
shadow-drawer
shadow-modal
```

具体实现必须：

- Light / Dark 分别审查；
- 不制造 Glow；
- 不取代 Surface / Hairline 分层；
- 不用于普通 KPI / Card。

正式优先级：

```text
Spacing
>
Surface difference
>
Hairline
>
Shadow
```

## 55.9 Motion Tokens

正式 Duration：

```css
--motion-instant: 80ms;
--motion-fast: 120ms;
--motion-standard: 180ms;
--motion-medium: 220ms;
--motion-slow: 280ms;
```

正式 Easing：

```css
--ease-ui: cubic-bezier(0.25, 0.1, 0.25, 1);
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```

正式映射示例：

```text
Button feedback   80–120ms role mapping
Tooltip           120ms
Popover           180ms
Tabs indicator    180ms
Toast             180ms
Drawer            220ms
Modal             220ms
```

Nav Morph 允许完整 `180–220ms` Contract。

旧示例中的 `motion-slow: 240ms` 不再成立。

## 55.10 Z-index Tokens

正式层级使用语义 Token：

```css
--z-base: 0;
--z-sticky: 10;
--z-popover: 30;
--z-drawer: 40;
--z-command: 40;
--z-modal: 50;
--z-toast: 60;
```

Scrim 与对应 Overlay 绑定，不用全局任意 `99999`。

如果未来增加新 Overlay Role，必须先定义语义层级。

正式原则：

```text
z-index
→ semantic hierarchy
not local competition
```

## 55.11 Chart Tokens

Chart Namespace 与 Status 完全分开：

```text
color.chart.*
≠ color.status.*
```

v1 Categorical Chart Palette 正式固定为独立冷色 / 中性色系：

### Light

```css
--chart-series-1: #5E5CE6;
--chart-series-2: #0086A0;
--chart-series-3: #AF52DE;
--chart-series-4: #A2845E;
--chart-series-5: #667085;
--chart-series-6: #C2417A;
```

### Dark

```css
--chart-series-1: #7D7AFF;
--chart-series-2: #64D2FF;
--chart-series-3: #BF5AF2;
--chart-series-4: #D4B483;
--chart-series-5: #A1A1A6;
--chart-series-6: #FF8AD8;
```

规则：

- Series Identity 在同一图中稳定；
- 普通 Series 不使用 Danger / Success 语义色；
- Forecast / Comparison 继续通过 Dash / Label / Boundary 等非颜色通道区分；
- 颜色不是唯一 Series 区分方式；
- 具体图表需通过 Accessibility QA。

## 55.12 Semantic Aliases

组件不应直接依赖基础 Token 名称表达业务角色。

推荐：

```text
button.primary.background
input.border.focus
table.row.hover
status.danger.foreground
overlay.scrim
chart.axis.text
```

Implementation 可以使用 CSS Variables、TypeScript Token Object 或编译期 Design Token Pipeline，但语义映射必须唯一。

## 55.13 Token Versioning

Token 修改遵守第 59 章。

破坏性 Token Change 包括：

- 删除公开 Semantic Token；
- 改变同名 Token 业务语义；
- 重新解释 Status Color；
- 重新定义 Breakpoint / Size Contract。

不得因为 Feature 急需一个值就直接在本地 CSS 中永久绕过 Token System。

## 55.14 Token QA

正式检查：

```text
No arbitrary spacing
No arbitrary radius
No arbitrary z-index
No unregistered color
No old 240ms motion token
No feature-specific icon size
No chart/status palette mixing
Light / Dark token completeness
CN / EN / RU typography
```

核心验收：

```text
Semantic token
→ stable role
→ centralized change
```

---

# 56. Anti-patterns

本章集中列出 O3Pilot 正式禁止的设计与实现模式。

任何页面即使“看起来更漂亮”，只要违反这些规则，都不能通过正式 Design QA。

## 56.1 Visual

禁止：

```text
Decorative gradients
Glassmorphism everywhere
Every card has shadow
Every row is a card
Every metric has a different color
Huge marketing hero inside app
Nested cards without need
Excessive borders
Random radius
Random spacing
Glow / neon data UI
Large decorative blur
```

正式取向：

```text
Quiet
Precise
Surface before shadow
Data before decoration
```

## 56.2 Data Semantics

禁止：

```text
Unknown → 0
Unavailable → 0
No denominator → 0%
Missing → interpolation
Missing cost → zero cost
Estimated → fact
Forecast → actual
Open cohort → final result
Historical age → stale automatically
Current state → historical fact
```

## 56.3 Money and Metrics

禁止：

```text
Different currencies → silent sum
Unknown currency → assume shop currency
All revenue concepts → “销售额”
Display rounding → calculation rounding
FX conversion → estimate label automatically
Metric origin → hidden
PARTIAL → shown as complete VALID
```

## 56.4 Navigation

禁止：

```text
One capability = one sidebar item
Sidebar disappears completely when collapsed
Hamburger-only Narrow navigation
Multiple competing primary nav systems
Shop capability changes → nav item silently disappears
Global IA copied again as page tabs
Nested tabs by default
Random page-local icon semantics
```

## 56.5 Tabs / Filters / Controls

禁止：

```text
Filter disguised as tab
Segmented control with many domains
Context hidden inside ordinary filters
Draft filter silently applied
Clear filters resets date/shop/currency
Applied filters invisible
Filter count fabricated
Unavailable option silently removed
```

## 56.6 Tables

禁止：

```text
Current page-only sort presented as global sort
Unknown total fabricated
Infinite scroll by default
Identifier treated as number
Leading zero removed
Scientific notation for IDs
Wide table cardified on mobile
Every row clickable by default
Checkbox column without batch action
Business fact table turned into spreadsheet
```

## 56.7 Charts

禁止：

```text
3D chart
Radar chart by default
Decorative gauge
Meaningless pie / donut
Dual Y-axis by default
Bar chart with misleading non-zero baseline
Smooth curve that overshoots actual values
Gap auto-connected
Fake confidence interval
Unlabeled Top-N truncation
Decorative chart entrance animation
```

## 56.8 Forms and Buttons

禁止：

```text
Placeholder-only label
Disabled input used to display business fact
Premature validation on first keystroke
Submit failure clears form
Saved secret revealed again
Secret copy button
Credential test through write endpoint
Switch + Save with ambiguous commit semantics
Multiple competing Primary buttons in one action context
Important = destructive
```

## 56.9 Overlays

禁止：

```text
Drawer inside drawer inside drawer
Modal used as mini application page
Popover used for long complex workflow
Background interactive while modal/drawer focus is trapped
Arbitrary z-index competition
Backdrop blur as primary elevation mechanism
Unsaved material changes silently discarded
```

## 56.10 Loading and States

禁止：

```text
Refresh → blank page → skeleton
Fake percentage progress
Infinite loading after request failed
Loading one widget → disable whole page
Every success → toast
Every background event → toast storm
Error → empty state
Error with Last Known Data → erase content
All states → “暂无数据”
```

## 56.11 Motion

禁止：

```css
transition: all;
```

以及：

```text
UI ease-in as default
scale(0)
Button bounce / spring by default
500ms dropdown
Animated keyboard shortcut
Layout-heavy motion
Decorative moving KPI numbers
Chart bars growing from zero on refresh
Motion queue that blocks latest user intent
No reduced motion
Manual 120fps loop
```

## 56.12 Accessibility

禁止：

```text
Focus ring removed
Hover-only critical information
Color-only status
Icon-only action without name
Every chart point as Tab stop
ARIA grid without grid interaction model
Keyboard shortcut as only path
Touch target too small on Narrow
Tooltip as only accessible label
Screen reader interrupted on every refresh
```

## 56.13 Localization

禁止：

```text
Chinese string as logical key
Hard-coded English inside feature
Critical button ellipsis because RU is long
Date format 09/04/2026 without locale clarity
Identifier locale-formatted
Translation key visible to user
Font fallback breaking Cyrillic metrics
```

## 56.14 Security and Ozon Boundary

正式禁止任何主动 Ozon 写操作 CTA：

```text
Create / edit / delete product
Modify product content
Modify price
Modify inventory
Create / cancel order
Change order status
Fulfill / ship
Modify warehouse / logistics
Create / modify campaign
Change bid / budget
Modify promotion
Create / update / enable / delete Ozon webhook subscription
Send Ozon chat / reply
Auto reply review
```

也禁止：

```text
Secret in logs
Stored secret reveal
Recovery Key in ordinary settings
Business data visible after Session Revoked
```

核心原则：

```text
Capability of credential
≠ capability of O3Pilot
```

---

# 57. Acceptance Criteria

`DESIGN.md` 定义正式页面进入实现完成状态前必须满足的设计验收门槛。

某页面满足视觉稿，不代表满足 Acceptance Criteria。

## 57.1 Design System

- [ ] MiSans Global 正确加载；
- [ ] 只使用 400 / 500 / 600 / 700；
- [ ] Type Scale 使用正式 Token；
- [ ] Spacing 使用正式 Token；
- [ ] Radius 使用正式 Token；
- [ ] 无未注册视觉色；
- [ ] Light / Dark 使用正式 Theme Token；
- [ ] Lucide 通过 Semantic Icon Registry；
- [ ] Morphicons 只用于批准场景；
- [ ] Chart Palette 与 Status Palette 分离。

## 57.2 Shell and Navigation

- [ ] Sidebar IA 与第 5 章一致；
- [ ] Expanded 248px；
- [ ] Collapsed 72px；
- [ ] Narrow 60px Rail 永久存在；
- [ ] Active Primary Nav 唯一；
- [ ] Shop Capability 不导致 Nav 静默增删；
- [ ] Global Context Bar 不被页面重复实现；
- [ ] Page Header Context 正确；
- [ ] Back / Detail 保留合理 Query State。

## 57.3 Data Semantics

- [ ] 0 与 Missing 正确区分；
- [ ] NULL / UNAVAILABLE / NOT_APPLICABLE / UNMATCHED / UNVERIFIED 正确表达；
- [ ] Metric Status 可追溯；
- [ ] `VALID` 默认 Quiet；
- [ ] PARTIAL 显示 Coverage；
- [ ] STALE 显示 Last Valid；
- [ ] NO_DENOMINATOR 不显示 0%；
- [ ] OPEN_COHORT 不伪装最终值；
- [ ] Origin 可发现；
- [ ] As-of / Freshness 可发现；
- [ ] Time Basis 可发现；
- [ ] Historical Fact 不被 Current State 覆盖。

## 57.4 Money and Comparison

- [ ] Money 始终具有 Currency Context；
- [ ] Multi-currency 未选择 Reporting Currency 时不静默相加；
- [ ] Reporting Currency 不覆盖 Source Currency；
- [ ] FX Basis 可追溯；
- [ ] Missing FX 不静默删除币种；
- [ ] Ozon Business FX 与 Seller Cost FX 分开；
- [ ] Comparison 有明确 Basis；
- [ ] `pp` 与 `%` 区分；
- [ ] Comparable Time / Currency Context 成立才显示 Comparison；
- [ ] Open Period 不与 Closed Period 静默比较。

## 57.5 Interaction

- [ ] Button / Link 语义正确；
- [ ] 一个 Action Context 只有一个 Dominant Primary；
- [ ] Filter / Tab / Segmented Control 语义不混用；
- [ ] Applied Filter 可发现；
- [ ] Draft Filter 与 Applied Filter 分开；
- [ ] Sort 作用于完整 Filtered Result；
- [ ] Search Scope 明确；
- [ ] Pagination 与 Backend Semantics 一致；
- [ ] Drawer / Modal / Popover 选择正确；
- [ ] Focus / Dismissal / Unsaved State 正确；
- [ ] Toast 不重复自明反馈。

## 57.6 Loading and State Surfaces

- [ ] Initial Load 与 Refresh 分开；
- [ ] 有旧数据时 Refresh 保留旧数据；
- [ ] Skeleton 只用于可预测结构；
- [ ] 不显示 Fake Progress；
- [ ] Long-running Task 使用 Persistent Job；
- [ ] Error 不伪装 Empty；
- [ ] Error + Last Known Data 保留内容；
- [ ] Filtered Empty 与 Dataset Empty 分开；
- [ ] Recovery Action 匹配 Failure Type；
- [ ] 状态 Scope 限制在最小受影响 Surface。

## 57.7 Responsive

- [ ] 1440 / 1200 / 1024 / 768 / 390 均可完成主要任务；
- [ ] Narrow 不隐藏关键语义；
- [ ] Wide Table 使用 Horizontal Scroll；
- [ ] Chart 不因 Narrow 改 Metric / Date Range；
- [ ] Form Narrow 单列；
- [ ] Touch Target ≥44px；
- [ ] Drawer Narrow 全宽；
- [ ] Tabs / Filters 长文本可处理；
- [ ] Breakpoint 变化不重置业务 Context。

## 57.8 Accessibility

- [ ] WCAG 2.2 AA；
- [ ] Semantic HTML / Landmark；
- [ ] 主要路径 Keyboard-complete；
- [ ] Focus Ring 可见；
- [ ] 状态不只靠颜色；
- [ ] Icon-only 有 Accessible Name；
- [ ] Overlay Focus 正确；
- [ ] Form Error 可识别；
- [ ] Chart 有 Accessible Data Representation；
- [ ] Reduced Motion 完整；
- [ ] 200% Zoom 可用；
- [ ] Live Region 不制造通知风暴。

## 57.9 Motion and Performance

- [ ] Motion 有明确 Purpose；
- [ ] 高频操作无明显动画；
- [ ] 普通 UI 使用正式 80 / 120 / 180 / 220 / 280ms Token；
- [ ] 无 `transition: all`；
- [ ] 无默认 `ease-in` UI；
- [ ] 优先 transform / opacity；
- [ ] Latest User Intent 可中断旧动画 / 请求；
- [ ] Production Build + representative data 下无系统性明显掉帧；
- [ ] 120Hz 优化不破坏 60Hz；
- [ ] Large Table / Chart 保持交互。

## 57.10 Localization

- [ ] CN / EN / RU Layout 验证；
- [ ] Cyrillic 使用 MiSans Global；
- [ ] 长 Button / Tabs / Filters 可读；
- [ ] Identifier 不被 Locale Formatting；
- [ ] Currency / Number 按 Locale 展示但不改变计算；
- [ ] Date 不产生歧义；
- [ ] 技术 Code 与本地化 Label 分层；
- [ ] 无 Translation Key 泄漏。

## 57.11 Page Completeness

每个正式页面至少定义：

- [ ] 页面目标问题；
- [ ] Context；
- [ ] Primary Evidence；
- [ ] Supporting Evidence；
- [ ] Search / Filter / Table when needed；
- [ ] Loading；
- [ ] Empty；
- [ ] Unavailable；
- [ ] Partial；
- [ ] Stale；
- [ ] Error；
- [ ] Detail / Drill-down；
- [ ] Responsive；
- [ ] Accessibility；
- [ ] Wireframe / Layout Reference。

## 57.12 Read-only and Security

- [ ] 页面无 Ozon 写操作 CTA；
- [ ] Recommendation 只有查看 / 复制 / 模拟；
- [ ] 买家反馈不发送 Ozon 回复；
- [ ] Advertising 不修改 Campaign；
- [ ] Inventory 不修改 Ozon 库存；
- [ ] Orders 不取消 Ozon 订单；
- [ ] Webhook 设置不修改 Ozon Subscription；
- [ ] Credential 页面明确 `O3Pilot access: READ ONLY`；
- [ ] Stored Secret 不 Reveal / Copy；
- [ ] Recovery Key 不在普通 Settings 展示；
- [ ] Session Revoked 后无业务数据可读。

## 57.13 Acceptance Gate

正式页面只有在以下都通过后才视为 Design-complete：

```text
Semantics
+
Responsive
+
Accessibility
+
State coverage
+
Performance
+
Localization
+
Read-only boundary
```

视觉接近参考图但其中任一 Contract 不满足：

```text
→ not accepted
```

---

# 58. Design QA

Design QA 使用固定矩阵验证 Design Contract 是否在真实页面、真实数据量和真实交互中成立。

## 58.1 QA Levels

正式分三层：

```text
Component QA
Page QA
Cross-product QA
```

### Component QA

检查 Button、Form、Table、Chart、Overlay、State 等局部 Contract。

### Page QA

检查页面信息层级、Context、数据语义和 Responsive。

### Cross-product QA

检查同一语义在不同页面是否一致，例如：

```text
PARTIAL
Currency
Date Range
Drawer
Copy
Alert
```

不能每个 Feature 各有一套表现。

## 58.2 Viewport Matrix

每个正式页面至少：

```text
1440 Desktop
1200 Desktop
1024 Compact
768 Compact boundary
390 Narrow
```

必要时增加：

```text
very wide desktop
short viewport height
mobile landscape-like width
```

但不得用额外 Viewport 代替正式矩阵。

## 58.3 Theme Matrix

必须验证：

```text
Light
Dark
System behavior
```

Dark 不是 Light Invert。

重点：

- Text Contrast；
- Hairline；
- Selected Surface；
- Status；
- Chart；
- Focus；
- Overlay；
- Disabled State。

## 58.4 Locale Matrix

至少：

```text
中文
English
Русский
```

每种语言包含 Long-string Dataset。

截图 / Review 不只使用最短中文文案。

## 58.5 State Matrix

至少覆盖：

```text
Loading initial
Refreshing
Known zero
Dataset Empty
Filtered Empty
UNAVAILABLE
NOT_APPLICABLE
UNVERIFIED
PARTIAL
STALE
ESTIMATED
NO_DENOMINATOR
NO_RECENT_DEMAND
OPEN_COHORT
Error without data
Error with Last Known Data
GAP
Job Running / Failed
Session Revoked
```

## 58.6 Input Matrix

至少：

```text
Mouse
Trackpad
Keyboard
Touch / coarse pointer where applicable
```

重点验证：

- Hover / Focus parity；
- 44px Target；
- Horizontal Scroll；
- Overlay dismissal；
- Search / Filter rapid interaction；
- Copy；
- Form paste。

## 58.7 Data Volume Matrix

不能只用 3 行 Demo Data。

至少验证：

```text
Empty
Small
Representative
Large
```

Large 场景覆盖：

- 长 Table；
- 多页 Cursor；
- 大量 Chart Points；
- 多 SKU；
- 多 Shop；
- 多 Currency；
- 多 Alert；
- 多 Job / Import History。

Top-N / Sampling / Pagination 必须明确，不静默丢数据。

## 58.8 Performance QA

使用 Production Build 和代表性数据。

重点：

```text
120Hz interaction
60Hz compatibility
Sidebar collapse
Rapid filter/search
Large table scroll
Chart hover
Drawer open/close
Theme switch
Data refresh
```

Long Task / Frame Miss 作为诊断信号。

不通过手工 120fps Loop 模拟“高刷支持”。

## 58.9 Accessibility QA

至少执行：

```text
Keyboard-only pass
Focus visibility review
Screen reader smoke test
200% zoom
Reduced Motion
Contrast review
```

复杂 Table / Chart / Overlay 必须单独验证。

## 58.10 Regression QA

当修改全局：

```text
Typography
Color
Spacing
Radius
Icon Registry
Motion
Breakpoint
Metric Presentation
State Language
```

必须进行 Cross-product Regression，而不是只检查修改页面。

例如修改 `PARTIAL` 样式：

```text
Dashboard
Sales
Inventory
Advertising
Profit
Data Center
```

都必须检查。

## 58.11 Design QA Evidence

正式实现验收应保留足够证据，例如：

```text
Screenshots / visual snapshots
Interaction recording when motion matters
Keyboard path notes
Accessibility findings
Performance trace when performance matters
State fixtures / representative data
```

不规定具体测试工具，但结果必须可重复检查。

## 58.12 QA Failure Rule

如果出现：

```text
Semantics wrong
Read-only boundary broken
Business data hidden
Accessibility blocker
Persistent jank
Security-sensitive leakage
```

则即使视觉完成度很高，也属于 Blocking Design Defect。

---

# 59. Versioning

`DESIGN.md` 使用语义化版本思路管理正式设计 Contract。

当前：

```text
Version 1.0
Status: Draft for Review
```

只有用户明确完成整体 Review / Freeze 后，才应修改 Document Status；本轮后半段补全本身不自动把 Draft 改成 Final。

## 59.1 Major

发生不兼容的核心设计变化时升级 Major，例如：

```text
Primary IA redesign
Navigation model replacement
Core design language replacement
Metric Presentation Contract incompatible change
Data State Language incompatible change
Breakpoint model replacement
Core Token semantic replacement
Read-only interaction model change
```

## 59.2 Minor

向后兼容地增加正式能力：

```text
New page
New component pattern
New supported state presentation
New detail domain
New responsive pattern
New chart type with defined contract
```

## 59.3 Patch

不改变现有 Contract 的修正：

```text
Copy correction
Example correction
Clarification
Typo
Token value micro-adjustment that preserves semantic role
QA wording improvement
```

如果所谓“微调”实际改变交互尺寸、Semantic Color 或 Business Meaning，则不能降级成 Patch。

## 59.4 Token and Component Compatibility

以下也需要版本意识：

```text
Design Tokens
Semantic Icon Registry
Component API
Locale Dictionary keys
Navigation Semantic IDs
Metric display mappings
```

禁止：

```text
rename semantic token silently
remove icon semantic ID silently
reuse old token name for new meaning
```

## 59.5 External Reference Updates

外部 Reference 更新：

```text
New external commit
≠ automatic O3Pilot design change
```

必须：

```text
Review
↓
Accept selected change
↓
Update DESIGN.md
↓
Version accordingly
```

## 59.6 Change Review

影响多个页面的 Design Change 至少检查：

```text
Affected sections
Token / component impact
Responsive impact
Accessibility impact
Localization impact
Motion / performance impact
Read-only / security impact
```

如果改变上游业务语义，必须先修改对应 `PRODUCT / DATA_MODEL / METRICS / SECURITY / ARCHITECTURE`，不能从 DESIGN 反向覆盖。

## 59.7 Version QA

每次发布 Design Revision 前确认：

- Version 与 Change Scope 匹配；
- Updated Date 更新；
- Reference Baseline 有变化时记录；
- 无同名 Token 双重语义；
- 无旧组件 Contract 残留；
- 文档内交叉引用仍正确。

核心原则：

```text
Design evolves deliberately
not by implementation drift
```

---

# 60. Reference Baseline

O3Pilot 的正式设计首先由自己的产品与数据 Contract 决定，外部资料只用于设计方法与交互参考。

## 60.1 O3Pilot Contract Baseline

优先级最高的正式来源：

```text
PRODUCT.md
DATA_SOURCES.md
DATA_MODEL.md
METRICS.md
ARCHITECTURE.md
SECURITY.md
DEPLOYMENT.md
DESIGN.md
```

这些文件定义：

- 产品边界；
- Ozon Read-only 能力；
- Runtime Data Source；
- Data State；
- Metric；
- Architecture；
- Security；
- Deployment；
- UI / UX。

正式关系：

```text
Business / Data Contract
→ DESIGN presents it

DESIGN
↛ silently redefines it
```

## 60.2 Apple DESIGN Reference

Reference：

```text
Repository: VoltAgent/awesome-design-md
Path: design-md/apple/DESIGN.md
Reference SHA: 0d37e1e5f6b621ccbdc666f6d9c82f2ada26208b
Reviewed: 2026-09-04
```

URL：

```text
https://raw.githubusercontent.com/VoltAgent/awesome-design-md/refs/heads/main/design-md/apple/DESIGN.md
```

O3Pilot 主要吸收：

- Quiet visual hierarchy；
- restrained surfaces；
- disciplined typography；
- subtle interaction color；
- minimal decorative effects。

不继承：

- SF Pro；
- Apple Marketing Page density；
- Apple-specific product motion；
- Apple-specific component implementation。

## 60.3 UI UX Pro Max Reference

Reference：

```text
Repository: nextlevelbuilder/ui-ux-pro-max-skill
Review Basis: README / design-system methodology available on 2026-09-04
Reviewed: 2026-09-04
```

URL：

```text
https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
```

主要用于：

```text
Accessibility checks
Responsive methodology
Dashboard / chart production checks
Information organization
Design system completeness
```

其推荐字体、颜色、视觉风格和 Motion 不自动进入 O3Pilot。

## 60.4 Emil Kowalski Motion Reference

Reference：

```text
Repository: emilkowalski/skills
Animate Skill Reference SHA: 159fe0753228ab9d43fce274bd2c24433f3c4771
Reviewed: 2026-09-04
```

URLs：

```text
https://github.com/emilkowalski/skills
https://github.com/emilkowalski/skills/blob/main/skills/animate/SKILL.md
```

Emil 是 O3Pilot Motion 的专项外部参考。

Motion 决策链保持：

```text
User-confirmed O3Pilot requirements
↓
O3Pilot 120Hz / Accessibility / Performance Contract
↓
DESIGN.md Motion Tokens / Patterns
↓
Emil Kowalski Motion Rules
↓
Existing implementation
```

Apple DESIGN 与 UI UX Pro Max 不进入 Motion 专项决策链。

## 60.5 Reference Freeze

正式关系：

```text
External Reference
≠ Runtime Dependency
≠ Automatic Design Update
```

外部仓库后续变化：

```text
→ no automatic effect on O3Pilot
```

需要采用时必须重新 Review 并写入 `DESIGN.md`。

## 60.6 Development References

Google Drive 中的：

```text
O3Pilot开发参考资料
```

以及其中的：

- Ozon API 验证资料；
- Ozon Global 卖家知识库镜像；
- 平台 / ERP / 物流商导出示例；

只用于 O3Pilot 的设计、验证和开发参考。

正式关系：

```text
Development Reference
≠ Runtime Data Source
```

O3Pilot 运行时数据源只由 `DATA_SOURCES.md` 定义。

历史仓库也只作为实现经验，不作为新项目设计约束。

## 60.7 Final Precedence

最终设计优先级：

```text
User-confirmed O3Pilot decisions
↓
O3Pilot upstream Contracts
↓
Current DESIGN.md formal rules
↓
Approved specialized references
↓
General external references
↓
Existing implementation
```

其中已有代码永远不能反向成为正式设计规范。

最终原则：

```text
O3Pilot semantics
→ owned by O3Pilot

External references
→ inform quality
not product truth
```

---
