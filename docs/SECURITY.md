# O3Pilot — SECURITY.md

> Version: 1.0  
> Status: Security Baseline  
> Updated: 2026-09-03  
> Applies to: O3Pilot

# 1. 文档目的

`SECURITY.md` 定义 O3Pilot 的正式安全边界、安全控制和安全验收要求。

本文件负责回答：

- O3Pilot 保护哪些资产；
- 哪些入口、数据源和外部系统属于不可信边界；
- 单用户登录和单有效 Session 如何实现；
- 密码、Session、CSRF 和浏览器安全如何处理；
- Ozon Seller API、Performance API 与其他外部凭证如何保存；
- O3Pilot 如何从架构上保证永远不主动修改 Ozon；
- Webhook 如何作为公网不可信输入安全接收；
- 用户上传的 CSV / XLS / XLSX 等文件如何安全解析；
- SQLite、Raw Store、日志和本地配置如何保护；
- Portable Backup、Cloudflare R2 与 Recovery Key 如何保护；
- Restore、升级、迁移和 Secret Rotation 如何避免泄密或绕过认证；
- 哪些安全事件必须记录；
- 哪些安全测试必须在发布前通过；
- O3Pilot 明确不承诺解决哪些主机级或外部账户级风险。

本文件不重新定义：

- 产品能力；
- Ozon 数据源的业务口径；
- 数据实体和主键；
- 指标公式；
- 数据保留周期；
- Scheduler / Worker 的调度策略；
- 安装命令、systemd / launchd 配置细节；
- UI、导航和页面布局。

上述内容分别由 `PRODUCT.md`、`DATA_SOURCES.md`、`DATA_MODEL.md`、`METRICS.md`、`ARCHITECTURE.md`、未来的 `DEPLOYMENT.md` 与 `DESIGN.md` 定义。

---

# 2. 安全总原则

O3Pilot v1 的安全基线：

```text
Default Deny
Read-Only by Architecture
Single User
Single Active Session
Server-side Authorization
Least Exposure
Explicit Trust Boundaries
Secrets Separate from Business Data
Sensitive Data Minimization
Authenticated Encryption
No Secret in Logs
No Arbitrary Outbound Requests
No Untrusted Code Execution
Backup Confidentiality + Integrity
Fail Closed
Security Controls Are Testable
```

安全规则不能仅依赖：

- 前端隐藏按钮；
- 用户“不去点击”；
- Ozon API Key 恰好没有写权限；
- HTTP Method 是 GET 还是 POST；
- Cloudflare 已经挡在前面；
- Webhook Payload 看起来像 Ozon；
- 上传文件扩展名看起来正确；
- 数据来自 Ozon 就一定可信；
- “当前只有一个用户”所以无需认证。

所有核心安全边界必须由服务端执行。

---

# 3. 不可突破的安全不变量

以下规则属于 O3Pilot v1 Security Invariant。

## 3.1 Ozon 永久只读

即使用户提供：

```text
Admin API Key
拥有写接口权限的 Performance Credential
其他拥有管理能力的 Ozon Credential
```

O3Pilot 仍不得执行任何 Ozon 业务写操作。

必须同时满足：

```text
Default Deny
+
Explicit Read Allowlist
+
Typed Gateway
+
Outbound Host Allowlist
+
No Arbitrary Endpoint Client
+
Automated Security Tests
```

前端、业务模块和后台 Job 均不能绕过这一层。

## 3.2 未认证用户不能读取业务数据

除明确列入 Public Surface 的入口外：

```text
Unauthenticated Request
→ DENY
```

不能因为：

- URL 编码差异；
- 重复 `/`；
- 尾部 `/`；
- 大小写变化；
- Host Header 异常；
- Proxy Header 异常；
- SPA Fallback；
- 静态资源路由；

绕过认证。

## 3.3 Secret 不进入普通业务数据层

以下内容不得作为普通业务字段存储：

```text
Ozon API Key
Performance Client Secret
Performance Access Token
R2 Secret Access Key
Webhook Endpoint Secret
Session Token
CSRF Secret
Instance Master Key
Backup Repository Key
Recovery Key
```

业务表可以保存 Secret 的引用、状态、最后验证时间和掩码显示信息，但不能保存明文。

## 3.4 Backup 文件本身不能等于凭证

获得：

```text
Local Backup Repository
Portable Backup Package
R2 Bucket Objects
```

不得自动等价于获得全部 Ozon / R2 Credential。

## 3.5 Webhook 不能直接成为最终业务事实

Webhook 只证明：

```text
O3Pilot 收到了一条外部事件输入
```

不自动证明：

```text
事件来源一定真实
Payload 一定完整
事件一定唯一
事件顺序一定正确
业务状态已经最终确定
```

Webhook 负责时效，Seller API Readback / Reconciliation 负责最终状态校准。

---

# 4. Threat Model

## 4.1 需要保护的核心资产

### Authentication Assets

- Owner Password Hash；
- Session；
- CSRF 状态；
- 最近认证状态。

### Integration Secrets

- Seller API Client ID / API Key；
- Performance Client ID / Client Secret；
- Performance Access Token；
- R2 Access Credential；
- Webhook Endpoint Secret；
- 未来其他第三方凭证。

### Business Data

- 商品；
- 价格；
- 库存；
- 订单；
- 退货；
- Finance；
- 广告；
- Rating；
- Questions / Answers；
- 卖家采购成本；
- 物流成本；
- Profit；
- Forecast；
- Raw API / Webhook 数据。

### Personal-related Data

可能包括：

- `author_name`；
- Question / Answer 文本；
- Chat / Review 文本；
- 订单与物流接口中能够关联到买家或收货流程的信息；
- 用户主动导入文件中的个人或业务敏感字段。

### Recovery Assets

- SQLite Snapshot；
- Raw Objects；
- Portable Config；
- Encrypted Secret Package；
- Backup Repository Key；
- Recovery Key；
- Backup Manifest 与 Integrity Metadata。

## 4.2 主要攻击面

O3Pilot 至少考虑：

- Internet 对登录入口的密码猜测；
- 被盗 Session；
- CSRF；
- XSS；
- Clickjacking；
- Host Header / Proxy Header 混淆；
- Path Normalization 绕过；
- 恶意 Webhook；
- Webhook 重放和重复事件；
- 恶意或异常上传文件；
- XLSX ZIP Bomb；
- Spreadsheet Formula Injection；
- SQL Injection；
- SSRF；
- 外部 API Redirect 导致凭证泄漏；
- 日志泄密；
- Backup / R2 泄漏；
- Dependency / Supply Chain 风险；
- 主机上其他本地进程读取文件；
- 浏览器设备被恶意软件控制；
- Cloudflare / Ozon / R2 Credential 被外部泄漏。

## 4.3 不可完全防御的主机级攻击

如果攻击者已经获得：

```text
root
macOS 管理员级控制
O3Pilot 运行用户的完整本地会话
任意读取 O3Pilot 内存的能力
```

则应用无法承诺继续保护运行中的明文数据与运行时密钥。

因此 O3Pilot 的目标是：

- 降低单一数据库或单一 Backup 泄漏的影响；
- 降低普通文件权限错误的风险；
- 不把 Secret 复制到更多位置；
- 让离线 Backup 和 R2 泄漏仍保持加密；
- 明确建议生产主机开启 FileVault / LUKS 等全盘加密。

O3Pilot 不宣传“即使主机完全失陷仍可保护所有数据”。

---

# 5. Trust Boundaries

正式信任边界：

```text
Browser
  ↓ untrusted network input
Cloudflare Edge
  ↓
Cloudflare Tunnel
  ↓ local transport
O3Pilot HTTP Boundary
  ↓ authenticated / validated
Application
  ↓
SQLite / Raw Store / Secret Store
```

外部数据边界：

```text
Ozon Seller API       ─┐
Performance API       ─┤
Exchange Rate API     ─┤
Ozon Webhook          ─┼→ Untrusted External Data Boundary
Official Report File  ─┤
Seller Data File      ─┘
```

即使来源是官方 API，也必须做：

- Schema Validation；
- 类型检查；
- 长度限制；
- 枚举兼容；
- 安全输出编码；
- 错误隔离。

---

# 6. Security Data Classification

O3Pilot 使用以下安全分类。

## 6.1 `SECRET`

包括：

```text
Password-derived verifier material
Ozon API Key
Performance Client Secret
Performance Access Token
R2 Secret Access Key
Webhook Endpoint Secret
Session Token
CSRF secret material
Instance Master Key
Backup Repository Key
Recovery Key
```

规则：

- 不写普通日志；
- 不进入 Analytics；
- 不进入 Raw API Payload；
- 不进入普通 Config；
- 不明文进入 Backup；
- UI 默认只显示是否已配置和必要掩码；
- 错误信息不得包含明文。

## 6.2 `SENSITIVE`

包括：

- 订单与履约业务事实；
- Finance；
- Profit；
- 卖家成本；
- 买家沟通文本；
- `author_name`；
- Raw API / Webhook Payload；
- 用户导入的业务数据；
- 内部经营分析与预测。

规则：

- 仅认证用户可读；
- 不进入公开缓存；
- 不进入第三方错误上报正文；
- 日志只记录最小必要标识；
- Backup 必须满足本文件的加密要求。

## 6.3 `INTERNAL`

包括：

- 非敏感运行配置；
- Dataset Code；
- Metric Code；
- Schema Version；
- Job 类型；
- 已脱敏日志；
- 非敏感健康状态。

仍不得默认公开到 Internet。

## 6.4 `PUBLIC`

仅包括明确设计为公开的：

- 静态前端资源；
- 必要的登录页面壳；
- 无敏感信息的版本兼容元数据（若产品需要）。

默认分类不是 `PUBLIC`。

---

# 7. Owner Authentication

## 7.1 单用户模型

O3Pilot v1：

```text
Single User
Single Owner Credential
Single Active Session
No Multi-user RBAC
```

用户身份不依赖：

- Ozon Seller Account 登录；
- Cloudflare Account 登录；
- 本机 OS 用户名；
- 浏览器指纹。

## 7.2 首次初始化

未初始化实例：

```text
state = UNINITIALIZED
```

正式规则：

- 不允许通过公网匿名页面直接创建 Owner；
- Owner 初始凭证只能通过本机受信任初始化流程建立；
- 具体 CLI / 安装流程由 `DEPLOYMENT.md` 定义；
- 初始化完成后才允许远程登录入口进入正常工作状态。

不得提供：

```text
admin/admin
固定默认密码
首次访问任意人可抢占管理员
```

## 7.3 密码长度与规则

v1 未强制 MFA，因此 Owner Password：

```text
minimum length = 15 characters
maximum accepted length >= 64 characters
```

规则：

- 允许空格；
- 允许 Unicode；
- 不要求“大写 + 小写 + 数字 + 特殊字符”组合；
- 不静默截断；
- 允许 Password Manager 粘贴；
- 不要求无原因的周期性强制改密；
- 如果确认发生泄漏或安全事件，必须要求更换。

## 7.4 密码 Hash

Owner Password 永远不使用可逆加密保存。

正式算法：

```text
Argon2id
```

参数至少不低于当前基线：

```text
memory >= 19 MiB
iterations >= 2
parallelism >= 1
```

实现可以在目标主机基准测试后提高成本，但不得低于安全基线。

Hash 记录必须包含算法和参数版本，以支持未来 Rehash。

禁止：

```text
SHA-256(password)
MD5
SHA-1
自制 Password Hash
明文 Password
```

## 7.5 登录限流

登录入口必须同时具有：

- Source-aware Rate Limit；
- Instance-wide Rate Limit；
- Progressive Backoff；
- Authentication Failure Audit。

不能采用永久锁死 Owner 的设计。

Cloudflare Rate Limit 可以作为额外防护，但不能替代应用层限制。

## 7.6 认证错误信息

对外统一返回：

```text
Invalid credentials
```

不得通过错误差异泄漏：

- Owner 是否存在；
- Password 是否长度正确；
- Account 是否已初始化的更多内部细节。

## 7.7 Password Change

修改 Owner Password：

- 必须已有有效 Session；
- 必须再次验证当前密码；
- 成功后撤销全部既有 Session；
- 旧 Password Hash 不继续保留为有效凭证；
- 不影响 Backup Recovery Key。

## 7.8 Password Reset

v1 不提供：

- 邮件找回密码；
- 安全问题；
- 默认后门密码。

失去密码时，恢复流程依赖对本机实例的受信任 OS 级管理权限。

具体命令由 `DEPLOYMENT.md` 定义。

---

# 8. Session Security

## 8.1 Server-side Session

O3Pilot 使用服务端 Session。

Session Token：

- 使用 CSPRNG 生成；
- 至少 256 bit 随机值；
- 浏览器仅持有 Raw Token；
- 服务端只保存 Token 的不可逆摘要作为查找值；
- 不把完整 Session 状态编码成可长期离线使用的客户端 JWT。

## 8.2 Session Cookie

正式 Cookie 属性：

```text
Name = __Host-o3pilot_session
Secure
HttpOnly
SameSite=Strict
Path=/
No Domain attribute
```

不得把 Session 放在：

- URL Query；
- URL Fragment；
- LocalStorage；
- SessionStorage；
- 页面可读取 JavaScript 变量。

## 8.3 Single Active Session

新登录成功必须在一个原子安全流程中：

```text
Create New Session
+
Revoke Previous Active Session
```

旧设备下一次请求必须得到：

```text
SESSION_REVOKED / Unauthorized
```

## 8.4 Session Rotation

以下事件必须产生新 Session 或撤销旧 Session：

- 登录成功；
- Password Change；
- Owner Password Reset；
- Backup Restore；
- Session 安全状态异常；
- 明确 Logout。

## 8.5 Session Timeout

v1 默认：

```text
Idle Timeout = 24 hours
Absolute Lifetime = 7 days
```

实现允许用户缩短，但不能无限期关闭 Session Expiration。

## 8.6 Recent Authentication

以下高敏感本地操作要求最近重新认证：

- 修改 Owner Password；
- 添加 / 修改 / 删除 Integration Credential；
- 修改 R2 Credential；
- 生成 / 轮换 Webhook Endpoint Secret；
- 创建 Portable Export；
- 执行 Restore；
- 显示一次性 Recovery 相关敏感信息。

默认 Recent Authentication Window：

```text
5 minutes
```

---

# 9. CSRF 与 Browser Request Security

## 9.1 SameSite 不是唯一 CSRF 防线

即使 Session Cookie 使用：

```text
SameSite=Strict
```

所有状态改变接口仍必须具备正式 CSRF 防护。

## 9.2 CSRF Token

对基于 Cookie Session 的状态改变请求：

```text
POST
PUT
PATCH
DELETE
```

必须使用 Session-bound CSRF Token。

推荐传递方式：

```text
X-CSRF-Token
```

服务端验证 Token 与当前 Session 绑定。

## 9.3 Origin Verification

状态改变请求还应验证：

```text
Origin
```

如果浏览器场景缺少 Origin，再使用：

```text
Referer
```

目标 Origin 必须来自 O3Pilot 受信任配置，不得直接把未验证的 `Host` Header 当作信任来源。

## 9.4 Fetch Metadata

浏览器发送 `Sec-Fetch-*` 时：

- 明确拒绝不合理的 `cross-site` 状态改变请求；
- 作为纵深防御；
- 不能替代 CSRF Token 与 Origin Verification。

## 9.5 Login CSRF

登录接口本身也属于安全边界。

登录页面应使用：

- Pre-auth CSRF State；
- Origin / Referer Validation；
- 严格 Content-Type；
- 登录限流。

## 9.6 Webhook 例外

Ozon Webhook：

- 不使用浏览器 Session；
- 不使用 CSRF Token；
- 使用独立 Webhook Authentication / Validation Contract。

---

# 10. HTTP、Host 与 Cloudflare Tunnel Boundary

## 10.1 Loopback Only

O3Pilot 正式监听：

```text
127.0.0.1:<o3pilot-port>
```

不得默认监听：

```text
0.0.0.0
::
LAN Address
Public Address
```

公网访问通过 Cloudflare Tunnel。

## 10.2 Canonical External Origin

O3Pilot 必须拥有明确的：

```text
external_scheme
external_host
```

安全逻辑不得依赖任意请求中的 Host 自动推断。

## 10.3 Host Validation

允许的 Host 至少只能来自：

- 配置的 O3Pilot 公网 Host；
- 明确允许的本地访问 Host。

其他 Host：

```text
400 / 403
```

认证、CSRF、Redirect 和绝对 URL 构建不能基于未经校验的 Host。

## 10.4 Proxy Headers

`Forwarded`、`X-Forwarded-*`、`CF-Connecting-IP` 等头：

- 只有在请求确实来自预期本地 Tunnel / Proxy 边界时才可信；
- 不用于跳过认证；
- 不用于决定某 API 是否 Public；
- 来源 IP 主要用于 Rate Limit / Audit，不成为 Owner 身份。

## 10.5 Path Security

安全判断必须使用与路由器一致的规范化 Request Path 语义。

必须测试：

```text
//
trailing slash
%2F
%2E
.. segments
mixed case
query string
encoded delimiters
malformed Host
```

任何变体均不得导致受保护 Handler 绕过认证 / CSRF。

## 10.6 Cloudflare Access

Cloudflare Access 可以作为公网 UI 的第二层防护。

定位：

```text
Optional Defense in Depth
```

不是：

```text
O3Pilot 应用认证的替代品
```

即使未配置 Cloudflare Access，O3Pilot 仍必须满足完整 Authentication / Session / CSRF 安全要求。

如果启用 Access：

- Webhook 路由需要单独设计可被 Ozon 到达的策略；
- 不能为了 Webhook 将整个应用匿名放行。

---

# 11. Browser Security Headers

对 Web UI 正式启用：

```text
Content-Security-Policy
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Permissions-Policy
```

CSP 至少遵守：

```text
default-src 'self'
object-src 'none'
base-uri 'none'
frame-ancestors 'none'
```

规则：

- 禁止 `unsafe-eval`；
- JavaScript 不使用任意外部 CDN；
- 如确需 Inline Script，使用 nonce / hash，不直接开放 `unsafe-inline`；
- 页面不得被第三方站点 Frame。

对认证页面和业务 API：

```text
Cache-Control: no-store
```

Hashed Static Assets 可以使用长期 immutable cache。

公网 HTTPS 场景应启用 HSTS；具体 Domain Scope 由 `DEPLOYMENT.md` 根据部署域名确定，避免误伤同一主域下其他未使用 HTTPS 的服务。

---

# 12. CORS

O3Pilot v1 浏览器 API 默认：

```text
Same Origin Only
```

禁止：

```text
Access-Control-Allow-Origin: *
+
Credentialed Cookies
```

除非未来出现正式跨 Origin 产品需求，否则不开放通用 CORS。

---

# 13. Ozon Read-Only Security Gate

## 13.1 语义只读，不按 HTTP Method 判断

Ozon Seller API 大量读取接口使用：

```text
POST
```

因此：

```text
POST != Write
GET != Automatically Safe
```

O3Pilot 按 Endpoint Contract 的业务语义决定是否允许。

## 13.2 Endpoint Registry

Seller API、Performance API、Exchange Rate API 分别维护独立 Registry。

每个允许的外部操作至少定义：

```text
provider
operation_code
host
path
http_method
semantic_class
credential_type
response_contract
```

`semantic_class` 至少包括：

```text
READ
AUTH_BOOTSTRAP
PROHIBITED_WRITE
```

例如 Performance `client_credentials` Token 获取属于 `AUTH_BOOTSTRAP`，不是 Ozon 业务写入。

## 13.3 Default Deny

任何未注册 Endpoint：

```text
DENY
```

不能在业务代码中临时传入任意 Path 绕过 Registry。

## 13.4 禁止 Generic Arbitrary Client

不得向业务模块暴露：

```text
request(method, url, headers, payload)
request(path, arbitrary_body)
```

形式的 Ozon 万能客户端。

业务模块只能调用：

```text
Typed Read Gateway
```

## 13.5 Outbound Host Allowlist

Credential 只能发送到对应的精确 Provider Host。

例如：

- Seller Credential 只能发给 Seller API Host；
- Performance Credential 只能发给 Performance API Host；
- R2 Credential 只能发给配置的 R2 Endpoint。

外部 HTTP Client：

- 默认不跟随跨 Host Redirect；
- 如果某协议必须 Redirect，必须重新验证目标 Host；
- Credential Header 不得在 Redirect 中泄漏到其他 Host。

## 13.6 明确禁止的 Ozon 能力

至少包括：

- 商品 Create / Update / Delete；
- 商品内容修改；
- Price Update；
- Stock Update；
- Order / Posting 履约写操作；
- Warehouse / Delivery 配置修改；
- Campaign / Bid / Budget 修改；
- Promotion 修改；
- Question / Review / Chat 发送或回复；
- Webhook Subscription Set / Update / Enable / Delete；
- 任何其他改变 Ozon 服务端业务状态的接口。

Webhook `/notification/check` 等可能主动触发外部行为的管理能力在 v1 也默认禁止，除非未来重新进行产品与安全评审。

## 13.7 Credential Permission 不扩大能力

`/v1/roles` 只回答：

```text
Credential 可以访问什么
```

O3Pilot Registry 回答：

```text
O3Pilot 被允许调用什么
```

二者取交集，不取并集。

---

# 14. External Credential Storage

## 14.1 不保存在哪里

Secret 不得保存于：

- Git Repository；
- `.env` 明文文件作为长期正式 Secret Store；
- 普通 JSON / YAML Config；
- SQLite 普通业务列明文；
- Process Command Line；
- 日志；
- Crash Dump；
- Browser Storage。

环境变量也不作为 O3Pilot 正式长期 Secret Store。

## 14.2 Instance Master Key

O3Pilot 初始化时生成：

```text
instance_master_key = CSPRNG(256 bit)
```

用途：

- 加密运行时 Integration Secrets；
- 保护 Backup Repository Key 的本机副本；
- 未来保护其他需要后台自动读取的本机 Secret。

规则：

- 不进入 SQLite；
- 不进入 Git；
- 不明文进入 Portable Backup；
- 不输出到日志；
- 存储在独立 Secret File；
- Secret File Parent Directory 权限 `0700`；
- Secret File 权限 `0600`；
- 仅 O3Pilot Runtime User 可读。

macOS Keychain / Linux Secret Service 等能力可以用于额外 Wrap，但不作为跨平台运行必需条件。

## 14.3 Runtime Secret Encryption

运行时 Secret 使用：

```text
AES-256-GCM
```

每个 Secret：

- 独立随机 Nonce；
- 使用 CSPRNG；
- 使用 AAD 绑定上下文。

AAD 至少包含：

```text
instance_id
secret_type
shop_id / integration_id
secret_record_version
```

这样密文被移动到错误实体时应验证失败。

## 14.4 Secret Record

数据库只保存类似：

```text
secret_id
secret_type
integration_id
ciphertext
nonce
crypto_version
created_at
rotated_at
last_verified_at
```

不得保存明文。

## 14.5 Access Token

Performance Access Token 等短期 Token：

- 优先仅保存在内存；
- 如因 Crash Recovery 必须持久化，也使用相同 Secret Encryption；
- 到期后不得作为长期历史数据保存；
- 不能写入 Debug 日志。

## 14.6 Secret Rotation

必须支持：

- Seller API Key Rotation；
- Performance Client Secret Rotation；
- R2 Credential Rotation；
- Webhook Endpoint Secret Rotation；
- Instance Secret Crypto Version Migration。

Rotation：

```text
Validate New Secret
↓
Persist Encrypted New Version
↓
Switch Active Reference
↓
Retire Old Version
```

不得先删除唯一有效 Secret 再测试新 Secret。

---

# 15. Webhook Security

## 15.1 Webhook 是 Public Surface

Webhook 接收端是少数无需 Owner Session 的公网入口。

因此必须与普通 `/api/*` 路由隔离。

## 15.2 当前来源鉴别边界

当前 O3Pilot 基线尚未建立“经过真实验证的 Ozon Webhook Cryptographic Signature Contract”。

因此 v1 不得虚构：

- Ozon 一定发送某个签名 Header；
- 某个未验证字段可以证明来源；
- IP Allowlist 可以永久代表 Ozon 来源。

在官方签名机制被真实验证前，v1 使用独立的高熵 Webhook Endpoint Secret 作为来源鉴别的 Bearer Secret。

## 15.3 Webhook Endpoint Secret

生成：

```text
CSPRNG >= 256 bit
```

建议编码为 URL-safe Token。

规则：

- 仅首次创建时显示完整 URL；
- 服务端只持久化 Token Hash / Verifier；
- 日志永远对该 Path Segment 脱敏；
- 泄漏后必须支持 Rotation；
- Webhook Secret 与 Owner Session / CSRF / Ozon API Key 完全独立。

## 15.4 Request Validation

Webhook 接收入口至少验证：

- Secret；
- HTTP Method；
- Content-Type；
- Body Size；
- JSON Parse；
- 顶层 Schema；
- `message_type` / Event Type；
- 字段类型；
- 单字段最大长度；
- 当前允许的 Shop / Integration Mapping。

未知字段可以进入 Raw，但不能触发任意代码路径。

## 15.5 Body Size

Webhook 必须在读取完整 Body 前执行请求大小上限。

超过上限：

```text
413 Payload Too Large
```

上限由真实 Payload 样本验证后调整，但不能无限制读取。

## 15.6 Idempotency 与 Replay

因为真实重复推送行为尚未验证，O3Pilot 默认按“可能重复”设计。

至少生成：

```text
request_fingerprint
received_at
shop_scope
event_type
```

如果 Ozon 提供稳定 Event ID，则优先使用官方 Event ID；否则使用经过定义的 Fingerprint 做重复检测辅助。

重复检测不能误删合法的两次相似业务事件，因此最终状态仍由 API Readback 校准。

## 15.7 Ack 与处理分离

Webhook HTTP Handler：

```text
Authenticate
Validate
Persist Raw Event
Create / Coalesce Job
Return
```

不得在公网请求事务中执行长时间同步或大规模重算。

## 15.8 No Direct Business Mutation

Webhook 不直接：

- 覆盖订单最终事实；
- 删除业务对象；
- 改写库存真值；
- 改写 Finance 真值；
- 触发 Ozon 写操作。

---

# 16. Upload / Import Security

## 16.1 文件不是可信数据源

即使文件由用户自己上传，也按不可信输入处理。

## 16.2 Format Allowlist

每个 Import Contract 必须显式声明允许格式。

例如：

```text
CSV
XLS
XLSX
```

不能因为文件扩展名存在就自动选择任意解析器。

## 16.3 不信任 MIME 与扩展名

至少同时检查：

- Extension；
- MIME；
- Magic / File Signature（适用时）；
- Parser 是否能以预期格式打开。

## 16.4 File Size / Row / Archive Limits

每个 Import Contract 都必须定义：

```text
max_upload_bytes
max_rows
max_columns
max_cell_length
max_archive_entries
max_inflated_bytes
```

XLSX 属于 ZIP 容器，必须防止 ZIP Bomb。

## 16.5 临时文件

上传文件只进入受控 Temp Area：

- 位于 Web Root 之外；
- 使用随机内部文件名；
- 不使用用户原始文件名作为实际 Path；
- 权限仅 Runtime User 可读；
- 成功和失败都必须清理；
- 不长期复制到 R2。

这与 `DATA_SOURCES.md` 的“一次性导入介质”规则一致。

## 16.6 禁止执行文件内容

O3Pilot 永远不：

- 执行 Excel Macro；
- 执行 Embedded Script；
- 运行上传的二进制文件；
- 启动 Office / LibreOffice 来“打开”上传文件；
- 自动访问 Workbook External Link；
- 计算不受信任 Formula 作为代码。

## 16.7 Formula Injection

如果未来导出 CSV / XLSX，来自 Ozon 或用户输入的字符串以：

```text
=
+
-
@
```

等公式触发字符开始时，必须使用安全导出策略，避免文件被电子表格程序解释为恶意公式。

## 16.8 Parser Failure

解析失败：

- 不产生部分“成功”业务事实，除非 Import Contract 明确支持逐行事务；
- 错误信息不得回显整个敏感行；
- 原文件按生命周期清理。

---

# 17. Input、SQL 与 Output Safety

## 17.1 SQL

数据库访问必须使用参数化 Query。

禁止：

```text
SQL = "..." + user_input
```

动态：

- Sort Field；
- Order；
- Column；
- Filter Operator；

必须来自显式 Allowlist，不能作为绑定参数无法覆盖的 SQL 结构直接拼接用户值。

## 17.2 XSS

任何来自：

- Ozon；
- Webhook；
- Question / Answer；
- Product Name；
- 用户导入；
- 外部错误信息；

的字符串都按不可信文本处理。

默认使用框架 HTML Escape。

如未来需要渲染 HTML / Rich Content，必须使用独立 Sanitization Contract，不允许直接 `innerHTML`。

## 17.3 SSRF

O3Pilot 不自动 Fetch 外部数据中出现的任意 URL。

例如：

- 商品图片 URL；
- 用户导入 URL；
- Webhook Payload URL；
- Question Text 中的 URL。

需要服务器主动访问网络的功能必须拥有自己的 Outbound Host / Protocol Contract。

禁止请求：

- localhost；
- private network；
- link-local metadata；
- arbitrary file://；

除非属于明确内部系统设计。

---

# 18. Local Data Protection

## 18.1 Runtime User

O3Pilot 正式运行时不应以 `root` 身份运行。

安装过程如果需要更高 OS 权限，运行时仍应使用普通受限用户。

## 18.2 Persistent Directory

包含：

- SQLite；
- WAL；
- Raw Store；
- Local Backup；
- Config；
- Secret File；

的目录默认：

```text
0700
```

敏感文件默认：

```text
0600
```

## 18.3 SQLite

SQLite 仍是正式 Primary Database。

Security v1 不要求引入 SQLCipher 作为运行依赖。

原因：

- Secret 已使用应用层加密；
- 一般业务数据使用文件权限和主机全盘加密保护；
- SQLCipher 会引入新的原生依赖与恢复复杂度，应由未来独立架构决策评估。

生产环境强烈建议：

```text
macOS FileVault
Linux LUKS / equivalent full-disk encryption
```

## 18.4 Raw Store

Raw Store 可能包含比 Normalized 表更多的原始信息。

因此：

- 权限不得低于 SQLite；
- 不放在 Web Root；
- 不提供任意文件下载 Path；
- UI 读取 Raw 必须经过认证和明确业务接口；
- 日志不能复制 Raw Payload。

---

# 19. Data Minimization 与 Privacy

## 19.1 不重复制造个人数据副本

对于 `author_name`、Question / Answer Text 等字段：

- Raw Fact 可按数据架构保留；
- Normalized / Read Model 只保存业务功能真正需要的字段；
- 不为搜索、日志、告警方便而复制完整文本到更多表；
- 不默认把个人相关文本放入通知消息。

## 19.2 UI 最小暴露

页面只展示完成当前业务任务需要的信息。

例如不需要完整个人相关文本时，不因为 API 有字段就全部展示。

## 19.3 第三方通知

未来 O3Pilot 自身的 DingTalk / Email / Webhook 等提醒如果启用：

- 默认只发送必要摘要；
- 不发送 Ozon API Key；
- 不发送完整 Buyer Text；
- 不发送完整 Finance Raw Payload；
- 不发送 Recovery Secret。

---

# 20. Logging 与 Security Audit

## 20.1 永不记录的内容

普通日志和 Audit 均禁止记录：

```text
Password
Session Token
Cookie Header
CSRF Token
Ozon API Key
Performance Client Secret
Performance Access Token
R2 Secret Access Key
Webhook Endpoint Secret
Instance Master Key
Backup Repository Key
Recovery Key
完整 Authorization Header
```

## 20.2 默认不记录的业务正文

除专门诊断模式且经过额外脱敏外，默认不记录：

- Question / Answer 全文；
- Chat / Review 全文；
- 完整 Order Raw Payload；
- 完整 Finance Raw Payload；
- 上传文件完整行内容。

## 20.3 Error Sanitization

第三方 API 错误可能包含：

- Request ID；
- Path；
- Payload 片段；
- Header；

写日志前必须经过 Sanitizer。

HTTP Client Debug Mode 不得直接打印完整 Request / Response Headers。

## 20.4 Security Audit Events

至少记录：

- Login Success；
- Login Failure；
- Session Revoked；
- Logout；
- Password Changed / Reset；
- Integration Secret Created / Rotated / Removed；
- Credential Validation Failure；
- Read Allowlist Denial；
- Outbound Host Denial；
- Webhook Authentication Failure；
- Webhook Oversize / Invalid Schema；
- Import Rejected；
- Backup Started / Verified / Failed；
- R2 Replication Verified / Failed；
- Restore Started / Verified / Failed；
- Security-sensitive Config Change；
- Application Upgrade / Migration Result。

Audit 只记录：

- 内部对象 ID；
- Event Type；
- 时间；
- 结果；
- 必要 Source Metadata；

而不是 Secret。

## 20.5 Audit Integrity Boundary

v1 本地 Audit 不是不可篡改外部审计系统。

拥有主机管理员权限的人仍可能修改本地日志。

O3Pilot 不把本地 Audit 宣传成法证级不可抵赖日志。

---

# 21. Portable Backup Security

## 21.1 Backup Confidentiality

Portable Backup 不只保护 Secret。

因为 Backup 还包含：

- Finance；
- 订单；
- Raw Data；
- 成本；
- 买家相关文本；

因此 O3Pilot v1 正式要求：

```text
Portable Backup Data = Client-side Encrypted
```

不仅依赖 R2 自身 Server-side Encryption。

## 21.2 Backup Repository Key

初始化时生成：

```text
backup_repository_key = CSPRNG(256 bit)
```

用途：

- 加密 Backup DB Snapshot；
- 加密 Raw Object；
- 加密 Portable Config；
- 加密 Backup Manifest；
- 加密 Portable Secret State。

本机运行副本使用 `instance_master_key` 加密保存。

## 21.3 Recovery Key

同时生成独立：

```text
recovery_key = CSPRNG(256 bit)
```

Recovery Key：

- 与 Owner Password 独立；
- 与 Instance Master Key 独立；
- 不作为普通配置保存；
- 不上传到 R2；
- 不写日志；
- 应由用户保存在 O3Pilot 主机之外，例如 Password Manager / 离线安全介质。

O3Pilot 不承诺在 Recovery Key 丢失且原主机也不可用时还能恢复加密 Backup。

## 21.4 Recovery Envelope

为了让自动 Backup 不需要每次输入 Recovery Key：

```text
Recovery Key
   ↓ derive KEK
Encrypt Backup Repository Key
   ↓
Recovery Envelope
```

Recovery Envelope 可以安全保存在 Backup Repository。

恢复到新主机时：

```text
User supplies Recovery Key
↓
Decrypt Backup Repository Key
↓
Decrypt Backup Objects
↓
Create New Host Instance Master Key
↓
Re-encrypt Runtime Secrets for New Host
```

## 21.5 Backup Encryption

Backup Object 正式使用：

```text
AES-256-GCM
```

要求：

- 每个对象独立随机 Nonce；
- 禁止 Nonce Reuse；
- AAD 绑定 Repository / Object / Format Version；
- 任何 Authentication Tag 验证失败都必须 Fail Closed。

## 21.6 Backup Manifest

Manifest 本身包含敏感业务结构信息，因此默认同样加密。

允许保留最小无敏感 Plaintext Envelope Header，例如：

```text
backup_format_version
cipher_suite
```

不得在明文 Header 中放：

- Shop Name；
- Client ID；
- 订单信息；
- Secret Metadata；
- 文件原始路径。

## 21.7 Integrity

Backup Verification 同时验证：

- AEAD Authentication；
- Object Length；
- Object Identity；
- SQLite Integrity；
- Required Object Presence；
- Manifest Reference；
- Config / Schema Compatibility。

单独的 Checksum 不替代 AEAD Authentication。

## 21.8 不包含 Portable Secrets 的备份

如果用户明确选择不把 Integration Secrets 纳入 Portable Backup，则允许生成仅包含业务数据、Raw、Config 和必要完整性元数据的 Backup。

这种 Backup 必须显式标记：

```text
DATA_COMPLETE_BUT_CREDENTIALS_REQUIRED
```

Restore 后不得把缺失的 Integration Credential 解释为“已恢复”。

对应集成必须处于：

```text
CREDENTIALS_REQUIRED
```

直到用户重新提供并完成验证。

## 21.9 Recovery Key Derivation

`recovery_key` 是随机 256-bit Secret，不是低熵用户密码。

Recovery Envelope 的 KEK 应通过经过审查的标准 KDF 从 Recovery Key 派生，例如：

```text
HKDF-SHA-256
```

并绑定独立随机 Salt、Repository ID 与 Format Version。

不得直接把 Recovery Key 字节复用为多个不同密码学用途的 Key。

首次生成或 Rotation 后，Recovery Key 只在明确的 Recovery Setup 流程中向用户完整展示，并要求用户在主机之外保存。

---

# 22. Cloudflare R2 Security

## 22.1 R2 Role

R2 仅是：

```text
Optional Off-device Backup Replica
```

不是：

- Runtime Database；
- Live Filesystem；
- Primary Secret Store；
- Public Download Bucket。

## 22.2 Client-side Encryption First

上传 R2 前：

```text
Plain Backup Object
↓ local encryption
Ciphertext
↓ HTTPS
R2
```

R2 自身的 Encryption at Rest 是额外纵深防御，不替代 O3Pilot Client-side Encryption。

## 22.3 Bucket Access

R2 Bucket：

- 默认 Private；
- 禁止 Public Bucket / Public Custom Domain 暴露 Backup；
- Credential 只授予 O3Pilot Backup 所需最小 Scope；
- 不复用 Cloudflare Global API Key；
- 不在前端保存 R2 Credential。

R2 Credential 本身可以作为 Portable Encrypted Secret 的一部分保存，以便恢复后继续使用原有 R2 配置。

但从 R2 启动一次“新主机远程恢复”时，用户仍必须先通过独立方式提供能够读取该 Private Bucket 的 Bootstrap Credential。

```text
Recovery Key
!=
R2 Authentication Credential
```

Recovery Key 负责解锁 Backup Ciphertext，不负责取得 R2 Bucket 的访问权限。

## 22.4 R2 Credential Failure

R2 Credential 泄漏不应直接暴露明文 Backup 数据，因为 R2 对象仍应为 O3Pilot 密文。

但攻击者可能：

- 删除对象；
- 篡改对象；
- 阻止恢复。

因此本地 Verified Backup 与 R2 Replica 是独立状态，不能只保留唯一 R2 副本。

---

# 23. Restore Security

## 23.1 Restore 是高敏感操作

Restore 必须：

- 最近重新认证；
- 明确选择 Backup；
- 验证 Recovery Key；
- 验证 Backup Integrity；
- 验证 Format / Schema Compatibility；
- 在替换当前状态前保留可恢复点。

## 23.2 Restore 后 Session

Restore 后：

```text
All Previous Sessions = REVOKED
```

包括 Backup 创建时存在的旧 Session。

Session 不能作为 Portable Business State 恢复。

## 23.3 Secret Rewrap

新主机 Restore：

- 生成新的 `instance_master_key`；
- 解密 Portable Secret State；
- 使用新主机 Master Key 重新加密；
- 不把旧主机 Local Master Key 复制过来。

## 23.4 Restore Temp Data

解密后的临时文件：

- 使用受限 Temp Directory；
- 权限 `0600`；
- 不进入日志；
- 成功和失败都清理；
- 尽量缩短明文存在时间。

## 23.5 Failed Restore

错误 Recovery Key、对象篡改、Manifest 不一致或 SQLite Integrity 失败：

```text
Fail Closed
```

不得“尽量恢复一部分然后继续运行”并把状态标记为正常。

---

# 24. Backup / Recovery Key Rotation

如果 Recovery Key 疑似泄漏：

1. 在可信运行实例上生成新的 Recovery Key；
2. 使用新的 Recovery Key 重新 Wrap Backup Repository Key；
3. 验证新的 Recovery Envelope；
4. 使旧 Recovery Envelope 失效；
5. 不要求重新加密所有历史 Backup Object，除非 Backup Repository Key 本身也疑似泄漏。

如果 Backup Repository Key 泄漏：

- 必须生成新 Repository Key；
- 新 Backup Repository 使用新 Key；
- 需要保留的旧数据按迁移策略重新加密；
- 仅轮换 Recovery Envelope 不足以解决问题。

---

# 25. Security of Background Jobs

Worker / Scheduler 与前端 API 共享同一安全边界。

后台 Job：

- 不能绕过 Ozon Endpoint Registry；
- 不能从 Job Payload 注入任意 URL / SQL / Endpoint；
- Job Payload 不保存 Secret 明文；
- 重试错误不把 Secret 写入 `error_message`；
- Job Admin / Retry API 仍需要 Owner Authentication + CSRF；
- Restore 后旧 Job 不得携带可继续使用的旧 Session Token。

---

# 26. Outbound Network Security

## 26.1 Provider Isolation

每个 Gateway 拥有自己的：

- Host Allowlist；
- Credential Type；
- Timeout；
- Retry Policy；
- TLS Requirement。

## 26.2 TLS

对外 API：

```text
HTTPS Required
Certificate Verification Required
```

禁止在正式运行中：

```text
verify_tls = false
```

## 26.3 Timeout

所有外部请求必须设置：

- Connect Timeout；
- Read Timeout；
- Total / Job Deadline。

避免恶意或异常外部服务无限占用 Worker。

---

# 27. Dependency 与 Supply Chain

## 27.1 依赖最小化

不为方便引入不必要的：

- Web Framework Plugin；
- Crypto Library；
- Parser；
- Background Service；

安全相关能力优先使用成熟、广泛审计的标准实现。

## 27.2 Crypto

禁止自行实现：

- AES；
- Argon2；
- Random Generator；
- JWT / MAC；
- Password Hash；
- Signature Algorithm。

使用成熟库。

## 27.3 Version Pinning

生产依赖必须可重复解析到确定版本。

升级依赖前：

- 读取 Release / Security Notes；
- 执行 Tests；
- 执行 Security Regression；
- 不在生产启动时自动下载未知最新版本。

## 27.4 Frontend Supply Chain

前端运行时不依赖任意第三方 CDN 执行脚本。

业务页面的 JavaScript / CSS / Icon 等正式资源由 O3Pilot 自身提供。

---

# 28. Upgrade 与 Migration Security

## 28.1 Upgrade 前 Backup

涉及：

- Schema；
- Raw Format；
- Secret Format；
- Backup Format；
- Config Format；

的升级必须遵守 `ARCHITECTURE.md` 的 Pre-upgrade Backup Contract。

## 28.2 Secret Migration

Secret Migration：

- 不产生长期明文临时副本；
- Crash-safe；
- 成功验证新格式后才删除旧密文；
- 失败时保持旧版本仍可恢复。

## 28.3 Downgrade

旧版本应用不得在不理解新 Secret / Schema Format 时静默覆盖数据。

不兼容：

```text
Refuse to Start / Refuse to Migrate
```

优于猜测性降级。

---

# 29. Security Incident Response

至少支持以下事件类型。

## 29.1 Owner Password Suspected Compromise

- 修改 Password；
- 撤销当前 Session；
- 检查 Login Audit；
- 如浏览器或主机也可疑，轮换 Integration Secrets。

## 29.2 Ozon API Key Leak

- 用户在 Ozon 侧撤销 / 轮换 Key；
- O3Pilot 更新 Secret；
- 验证新 Key；
- 检查是否存在异常 API 访问；
- O3Pilot 自身不尝试调用 Ozon 写接口“修复”状态。

## 29.3 Performance Secret Leak

- 撤销旧 Client Secret；
- 配置新 Secret；
- 清除内存 Access Token；
- 重新获取 Token。

## 29.4 Webhook Secret Leak

- 生成新 Endpoint Secret；
- 用户在 Ozon 侧手工更新 Webhook URL；
- 验证自然事件或受控连通性；
- 旧 Secret 失效。

O3Pilot 不自动修改 Ozon Webhook Subscription。

## 29.5 R2 Credential Leak

- 轮换 R2 Credential；
- 检查 Bucket Object 完整性；
- 验证 Local Backup；
- 如果 Backup Repository Key 未泄漏，R2 中业务数据仍应保持 O3Pilot 密文。

## 29.6 Recovery Key Leak

按第 24 章轮换 Recovery Envelope。

---

# 30. Mandatory Security Tests

正式 Release 前至少执行以下测试。

## 30.1 Authentication

- 未登录访问所有 Protected API → 401/403；
- 新登录撤销旧 Session；
- Logout 后旧 Token 立即失效；
- Password Change 后旧 Session 失效；
- Expired Session 失效；
- Raw Session Token 不出现在数据库和日志。

## 30.2 Path / Host Regression

测试：

```text
malformed Host
duplicate slash
encoded slash
encoded dot
trailing slash
query string
mixed case
proxy header spoof
```

不能绕过 Auth / CSRF。

## 30.3 CSRF

- 缺失 CSRF Token → reject；
- 错误 Token → reject；
- Cross-origin State Change → reject；
- Valid Same-origin + Valid Token → pass；
- Webhook 不错误依赖浏览器 CSRF。

## 30.4 Ozon Read-only Gate

- 未注册 Endpoint → reject；
- Prohibited Write Endpoint → reject；
- Business Module 无法构造任意 Ozon URL；
- Seller Credential 不会发送给 Performance Host；
- Redirect 到非 Allowlist Host 时 Credential 不泄漏；
- 即使测试 Credential 返回 Admin Role，Write Endpoint 仍不可达。

## 30.5 Webhook

- 错误 Secret → reject；
- 无 Secret → reject；
- Oversize Body → 413；
- Invalid JSON → reject；
- Unknown Event → quarantined / handled safely；
- Duplicate Event 不导致重复不可逆业务动作；
- Secret 不出现在日志。

## 30.6 Upload

- 假扩展名；
- 错 MIME；
- Corrupt XLSX；
- ZIP Bomb；
- 超大 Cell；
- 超大 Row Count；
- Macro / Embedded Content；
- Formula Injection export case；
- Parser Crash 后 Temp File 清理。

## 30.7 Secret Storage

- 数据库中搜索已知 Secret 明文 → 不存在；
- Config 中搜索 → 不存在；
- Log 中搜索 → 不存在；
- 错误 `instance_master_key` 无法解密 Secret；
- 密文被篡改 → AEAD 验证失败。

## 30.8 Backup

- Backup 中搜索已知业务明文 → 不应直接出现；
- Backup 中搜索已知 Secret 明文 → 不存在；
- Wrong Recovery Key → fail；
- Tampered Object → fail；
- Missing Object → fail；
- R2 只保存 Ciphertext；
- Restore 后旧 Session 全部失效；
- Restore 到另一台支持的 Mac / Linux 后 Integration Secret 可授权恢复。

## 30.9 Logging

自动扫描日志，确认不存在：

- API Key；
- Client Secret；
- Access Token；
- Cookie；
- Recovery Key；
- Webhook Secret。

---

# 31. Security Acceptance Gate

任何下列问题存在时，不得发布为正式版本：

```text
Authentication Bypass
CSRF Bypass
Ozon Write Path Reachable
Secret Plaintext in DB / Log / Backup
Arbitrary Outbound URL with Credential
Unbounded Public Upload
Webhook Secret Leakage
Backup Integrity Not Verifiable
Restore Keeps Old Session Valid
Known Critical Dependency Vulnerability without Mitigation
```

High Severity Security Finding 默认阻止 Release。

如果因特殊原因接受残余风险，必须：

- 有明确 Finding ID；
- 说明影响；
- 说明临时缓解；
- 说明修复计划；
- 不能用“理论上没人会这样做”作为接受理由。

---

# 32. 明确禁止的实现方式

禁止：

```text
把 Ozon API Key 写进源码
把 Secret 写进 .env 后提交
把 Session 放 LocalStorage
使用明文 Password
使用 SHA-256 直接 Hash Password
信任任意 Host Header
依赖前端判断权限
把 Cloudflare Access 当作唯一认证
把 SameSite 当作唯一 CSRF 防护
使用万能 Ozon request(url, payload)
跟随任意 Redirect 并保留 Authorization
自动 Fetch 外部 Payload 中任意 URL
执行上传 Excel Macro
长期保存用户上传原文件
把 Raw Payload 全量写 Debug Log
把 Backup 明文上传 R2
把 Recovery Key 保存到同一个 Backup
Restore 后继续接受旧 Session
为了测试调用 Ozon 写接口
```

---

# 33. Residual Risk

即使满足本文件，仍存在以下残余风险。

## 33.1 Ozon Credential 本身权限过大

O3Pilot 可以保证自己不调用写接口，但如果同一个 Admin API Key 在 O3Pilot 之外泄漏，攻击者仍可能直接调用 Ozon 写接口。

因此如果 Ozon 支持更小权限 Credential，用户应优先使用最小只读权限。

但 O3Pilot 的安全不能依赖这一点。

## 33.2 Browser / Device Compromise

用户已经登录的浏览器被恶意软件完全控制时，攻击者可能以当前 Session 权限读取业务数据。

## 33.3 Host Full Compromise

拥有 Runtime User / root 内存和文件访问权限的攻击者最终可能获得运行时明文 Secret。

## 33.4 Webhook Authenticity

在 Ozon 官方 Cryptographic Signature Contract 被真实验证前，Webhook Endpoint Secret 是 Bearer Secret，不具备公钥签名级别的来源证明。

## 33.5 Cloudflare Account Compromise

Cloudflare Account 被控制可能影响：

- Tunnel；
- DNS；
- R2 可用性；
- 公网流量路由。

O3Pilot Client-side Backup Encryption 可以降低 R2 数据明文泄漏风险，但不能防止攻击者删除远程对象。

---

# 34. Security Versioning

以下变化必须更新 `SECURITY.md` 或关联 Security Contract Version：

- Password Hash Algorithm；
- Session Format；
- CSRF Strategy；
- Secret Encryption Format；
- Instance Master Key Format；
- Backup Encryption Format；
- Recovery Key / Envelope Format；
- Webhook Authentication 方式；
- Ozon Endpoint Allowlist Model；
- Cloudflare Trust Boundary；
- 支持新的公开入口；
- 支持新的文件上传格式；
- 支持新的第三方通知或 Integration。

安全格式变化必须具有明确 Migration，不允许静默重新解释旧密文。

---

# 35. 本版参考基线

本版 Security Contract 基于当前 O3Pilot 正式基线：

```text
PRODUCT.md v1.0
DATA_SOURCES.md v1.0
DATA_MODEL.md v1.0
METRICS.md v1.0
ARCHITECTURE.md v1.0
```

并结合当前开发参考资料中已经确认的事实：

- Seller API Key 可能拥有 Admin 与大量写接口权限；
- O3Pilot 仍必须永久只读；
- Seller API 不能按 GET / POST 简单判断读写；
- Performance API 使用独立 Client Credential 与短期 Bearer Token；
- Question / Answer 可能包含 `author_name` 与用户文本；
- Webhook 真实 Payload、重试、顺序和重复行为仍存在未验证边界；
- Portable Backup 必须能够跨受支持 Mac / Linux 主机恢复；
- R2 是可选 Off-device Replica，而不是 Primary Database。

安全实现参考当前主流 Web 安全与密码学实践，包括：

- Argon2id Password Hash；
- Server-side Session；
- Secure / HttpOnly / SameSite Cookie；
- CSRF Token + Origin Validation；
- CSP 与安全响应头；
- Authenticated Encryption；
- Secret / Key Separation；
- 文件上传 Allowlist 与资源限制；
- 日志 Secret Redaction。

---

# 36. 核心原则

**O3Pilot 对 Ozon 的只读边界必须由代码结构强制，而不是由用户自律。**

**API Key 的权限上限不等于 O3Pilot 的权限上限。**

**所有公网输入默认不可信。**

**所有 Secret 默认不记录、不公开、不明文备份。**

**Backup 的可恢复性不能以牺牲凭证机密性为代价。**

**R2 可以增强离机耐久性，但不能成为明文数据仓库。**

**没有经过验证的安全机制，不写成已经存在。**

**安全失败优先 Fail Closed。**

**安全规则必须能够自动测试。**
