# O3Pilot — SECURITY.md

> Version: 1.1  
> Status: B.2 Adjudicated Security Contract  
> Updated: 2026-09-05  
> Applies to: O3Pilot  
> Parent baseline: `9b0cf5a9ea1a1d4e392137606731823b84902c046fd7b8105789482bae86290c`  
> Authority freeze: `Batch_B.0.1_Authority_Coupling_Freeze.md` @ `08aa0d03a111ec54ce9a177fde3c3ae6b23ae4e2335545d8be502f347e3f524b`

# 1. Scope & Authority

`SECURITY.md` defines O3Pilot's canonical security contract:

- Owner Authentication and Session security;
- CSRF, browser, Host, proxy and HSTS security;
- Ozon permanent read-only security controls;
- Secret / credential cryptographic protection;
- canonical sensitive-data and log-redaction vocabulary;
- Webhook / upload / input security;
- Backup, Recovery Key, R2 and Restore security semantics;
- outbound, dependency, migration and incident-response security;
- executable security tests and Security Acceptance.

This document does **not** redefine:

- product capability or Feature Phase;
- Ozon endpoint reality;
- data entity / key / schema / lineage;
- metric formulas;
- runtime topology and persistent-state architecture;
- OS installation, service-manager, filesystem, backup-repository or Restore operational procedure;
- UI behavior.

Those concerns remain owned by their frozen authorities. SECURITY may state the minimum security outcome another document must implement, but does not duplicate that document's operational contract.

Platform support is not a security-compatibility authority. In particular, SECURITY does not claim WSL is technically incompatible; any v1 WSL exclusion in `DEPLOYMENT.md` is a deployment-scope statement, not security evidence.

---

# 2. Security Principles & Invariants

O3Pilot v1 security baseline:

```text
Default Deny
Read-Only by Architecture
Single User
Multiple Active Server-side Sessions
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

Security must not rely only on:

- hidden frontend controls;
- user self-restraint;
- an Ozon credential coincidentally lacking write permission;
- HTTP Method being GET or POST;
- Cloudflare being in front;
- an incoming Webhook looking legitimate;
- upload filename or MIME appearance;
- Ozon-sourced data being inherently trustworthy;
- the product having only one user.

All core boundaries are enforced server-side.

## 2.1 Ozon permanent read-only invariant

Even if the user provides credentials with administrative or write capability, O3Pilot must not perform Ozon business mutations.

The invariant requires:

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
Automated Positive + Negative Security Tests
```

Frontend, API handlers, Workers and Scheduler jobs cannot bypass it.

## 2.2 Authentication and public-surface invariant

Except for explicitly declared Public Surface:

```text
Unauthenticated Request
→ DENY
```

Routing/path/Host/proxy variants must not bypass Authentication or CSRF.

## 2.3 Secret separation invariant

SECRET material must not be stored as ordinary business fields, normal config, Raw business payload, Analytics data or plaintext Backup content.

## 2.4 Backup does not equal credential access

Possession of Local Backup, a portable Backup Artifact or R2 Backup objects must not automatically yield plaintext Ozon/R2 credentials.

## 2.5 Webhook is not final business truth

Webhook proves receipt of an external event input, not authenticity/completeness/uniqueness/order/final business state. Seller API readback/reconciliation remains the final-state correction path.

---

# 3. Threat Model & Trust Boundaries

Protected assets include:

### Authentication Assets

```text
Owner Password verifier
Session Token / Session verifier
CSRF state
server-side Recent Authentication state
operation-bound high-risk authorization state
```

### Integration / Recovery Secrets

```text
Seller API Key
Performance Client Secret
Performance Access Token
R2 Access Credential
Webhook Endpoint Secret
Instance Master Key
Backup Repository Key
Recovery Key
```

### Business / Sensitive Data

Products, prices, inventory, orders, returns, finance, advertising, ratings, Questions/Answers, seller costs, logistics costs, Profit, Forecast, Raw API/Webhook data and imported business data.

Personal-related data may include `author_name`, Question/Answer text, Chat/Review text and order/logistics fields that can relate to buyer or delivery processes.

### Recovery Assets

Verified Backup Artifacts, Recovery Envelope, Backup Repository Key, Recovery Key and the metadata required to validate/restore them.

Primary attack surfaces:

```text
Login / Session / CSRF
Public HTTPS / Cloudflare Tunnel boundary
Ozon / Performance / R2 credentials
Webhook
Upload / parser
Outbound HTTP
Logs / error paths
Backup / Restore
Upgrade / migration
Dependencies / frontend supply chain
```

Host/root compromise cannot be completely prevented by O3Pilot. Security therefore focuses on minimizing exposure, limiting plaintext lifetime, separating keys/credentials and failing closed.

Any **new public entry point** requires threat-boundary, authentication, CSRF, Host/proxy and exposure review before release.

---

# 4. Security Data Classification & Canonical Redaction Vocabulary

SECURITY is the canonical owner of security classification and redaction vocabulary.

## 4.1 `SECRET`

Includes O3Pilot-owned credential/key/token material:

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

Rules:

- never enter ordinary logs;
- never enter Analytics;
- never enter Raw business payload as a credential value;
- never enter ordinary Config in plaintext;
- never enter Backup in plaintext;
- UI exposes only configured state / necessary mask except an explicitly approved one-time reveal;
- error messages never contain plaintext.

## 4.2 `SENSITIVE`

Includes business facts, Finance/Profit/costs, buyer communication text, `author_name`, Raw API/Webhook payload, imported business data and internal forecasting/analysis.

Rules:

- authenticated access only;
- no public cache;
- no third-party error-reporting body by default;
- logs contain only minimum necessary identifiers;
- Backup uses the security protections defined here.

## 4.3 `INTERNAL`

Includes non-sensitive runtime config, Dataset/Metric code, schema/format version, Job types, sanitized logs and non-sensitive health state.

`INTERNAL` is not automatically Internet-public.

## 4.4 `PUBLIC`

Only explicitly designed public material such as static frontend resources, the necessary login shell and non-sensitive compatibility metadata.

Default classification is not `PUBLIC`.

## 4.5 Non-owned external credential material

A cloudflared / Cloudflare Tunnel connector credential is **not** reclassified as an O3Pilot-owned `SECRET` merely because O3Pilot can encounter the material.

O3Pilot must not:

```text
read for application use
persist
copy
back up
migrate
display
rotate
```

that non-owned credential.

If credential material is encountered in logs, error text, diagnostics or command output, it is still a redaction target and must not be emitted in plaintext.

## 4.6 Canonical redaction targets

At minimum, sanitize:

```text
all SECRET values
Authorization / credential-bearing headers
Session / Cookie values
CSRF secret/token values
Webhook secret values
Recovery Key
credential-bearing query/path/URL material
non-owned cloudflared/Tunnel credential material
```

A second document may reference this vocabulary but must not maintain a divergent canonical list.

Any **new third-party notification or Integration** requires classification, secret-storage, outbound-egress and trust-boundary review.

---

# 5. Owner Authentication

## 5.1 Single-user model

```text
Single User
Single Owner Credential
0..N Active Server-side Sessions
No Multi-user RBAC
No Device Identity / Trusted Device Framework
```

Owner identity does not depend on Ozon login, Cloudflare account login, OS username or browser fingerprint.

## 5.2 Local initialization and bootstrap token

An uninitialized instance has:

```text
state = UNINITIALIZED
```

Owner credential may be established only through the trusted local initialization operation defined by `DEPLOYMENT.md`.

If initialization uses a bootstrap token/capability, the canonical security contract is:

```text
CSPRNG entropy >= 256 bits
raw value delivered only to the local initialization client
server stores only an irreversible digest/verifier
accepted only while state = UNINITIALIZED
accepted only through Loopback / approved local forwarding
single-use
maximum lifetime = 15 minutes
invalidated on successful initialization
invalidated on service restart
```

The literal wire format is not frozen.

No public anonymous Owner-claim page, fixed default password, `admin/admin`, or first-visitor takeover is allowed.

A change to initialization authentication requires Authentication/Public Surface regression review.

## 5.3 Password requirements

Owner Password:

```text
minimum length = 15 characters
maximum accepted length >= 64 characters
```

Rules:

- spaces and Unicode allowed;
- no arbitrary composition rule requiring upper/lower/digit/symbol;
- no silent truncation;
- Password Manager paste allowed;
- no unjustified periodic forced change;
- confirmed compromise requires change.

## 5.4 Password verifier

Owner Password is never reversibly encrypted.

Algorithm:

```text
Argon2id
```

Baseline:

```text
memory >= 19 MiB
iterations >= 2
parallelism >= 1
```

Implementation may raise cost after host benchmarking but may not go below the baseline.

Verifier state records algorithm/parameter version to support Rehash.

Forbidden:

```text
plaintext Password
SHA-256(password)
SHA-1
MD5
custom password hashing
```

A Password Hash Algorithm change must define rehash/migration behavior and pass Authentication regression tests.

## 5.5 Login controls and error behavior

Login must have:

```text
source-aware rate limit
instance-wide rate limit
progressive backoff
authentication-failure audit
```

The design must not permanently lock out the Owner.

Cloudflare rate limiting is optional defense in depth, not a replacement.

External authentication failure response is normalized, e.g.:

```text
Invalid credentials
```

It must not disclose Owner existence, password-shape correctness or unnecessary initialization internals.

## 5.6 Password Change

Password Change requires:

```text
valid Session
+
direct verification of current Owner Password
```

Success:

```text
new verifier becomes authoritative
old verifier no longer authenticates
all active Sessions are revoked
Recovery Key is unchanged
```

## 5.7 No in-place Password Reset

O3Pilot v1 provides no in-place Password Reset capability.

No:

```text
email password reset
security questions
default/backdoor password
OS-admin command that silently replaces the current Owner verifier
```

There is no DEPLOYMENT Password Reset command delegation.

Lost-password data recovery is possible only when a suitable external recovery source exists:

```text
trusted new target instance
→ local bootstrap
→ create new target Owner Credential
→ authenticate/recently re-authenticate target Owner
→ select verified external Backup Artifact
→ provide Recovery Key
→ Restore
```

`Owner Credential State` is not Portable Business State and is not imported over the target Owner credential.

If the Owner password is lost **and no usable external Backup/recovery source exists**, O3Pilot cannot promise recovery of the protected instance data. This is an explicit residual risk, not an authentication bypass.

---

# 6. Session & Recent Authentication

## 6.1 Server-side Session

Session Token:

```text
CSPRNG >= 256 bits
```

Browser holds the raw token. Server stores only an irreversible token digest/verifier used to locate/validate the Session.

O3Pilot does not encode the complete long-lived Session state into a client JWT.

Server-side Session state includes, as needed:

```text
session identifier / token digest
created_at
last_seen_at
expires_at
revoked_at
reauthenticated_at
```

The physical database schema is owned by `DATA_MODEL.md`; this list defines required security semantics, not a table freeze.

A Session format/state change must define invalidation/migration behavior and pass Session regression tests.

## 6.2 Session Cookie

```text
Name = __Host-o3pilot_session
Secure
HttpOnly
SameSite=Strict
Path=/
No Domain attribute
```

Raw Session token must not appear in URL Query/Fragment, LocalStorage, SessionStorage or readable page variables.

## 6.3 Multiple active Sessions

Successful login:

```text
create one new server-side Session
```

It does **not** revoke unrelated valid Sessions.

Revocation rules:

```text
Logout
→ revoke current Session

Password Change
→ revoke all Sessions

Restore
→ revoke all pre-Restore Sessions

security anomaly
→ revoke affected Session or all Sessions according to anomaly scope

explicit security-wide invalidation
→ revoke all Sessions
```

No in-place Password Reset event exists in v1.

## 6.4 Session lifetime

Default:

```text
Idle Timeout = 24 hours
Absolute Lifetime = 7 days
```

User may shorten these values. Infinite Session expiration is not allowed.

## 6.5 Recent Authentication

High-risk operations include:

- Owner Password Change;
- add/change/delete Integration Credential;
- change R2 Credential;
- generate/rotate Webhook Endpoint Secret;
- create portable Backup/export;
- Restore;
- display one-time recovery-sensitive material.

Browser:

```text
successful Owner Password re-authentication
→ set server-side Session.reauthenticated_at
```

Maximum accepted age when a high-risk operation authorization is created:

```text
5 minutes
```

Implementation may require a fresher re-authentication but v1 does not permit a longer window.

For local CLI Restore, browser Session / browser Recent Authentication state is not inherited. The CLI directly verifies the current Owner Password against the authoritative verifier.

For Restore, successful re-authentication creates only operation-bound authorization state for the exact Restore invocation/source/target. It is not a reusable bearer grant and is invalidated on cancellation, failure or completion.

A long-running already-authorized Restore does not become unauthorized merely because staging/migration exceeds five minutes.

---

# 7. CSRF & Browser Request Security

SameSite is defense in depth, not the sole CSRF control.

Cookie-authenticated state-changing methods:

```text
POST
PUT
PATCH
DELETE
```

require a Session-bound CSRF token, preferably in:

```text
X-CSRF-Token
```

State-changing requests also validate `Origin`; if absent in an applicable browser scenario, validate `Referer`.

Trusted origin comes from O3Pilot configuration, not from an arbitrary unvalidated Host header.

When `Sec-Fetch-*` is present, reject unreasonable cross-site state changes as an additional control.

Login uses pre-auth CSRF state, Origin/Referer validation, strict Content-Type and login rate limiting.

Webhook endpoints do not use browser Session/CSRF and instead use their own authentication/validation contract.

Any CSRF strategy change requires CSRF regression coverage.

---

# 8. HTTP, Host, Proxy, Browser Headers & HSTS

## 8.1 Loopback and canonical origin

Application listener:

```text
127.0.0.1:<o3pilot-port>
```

not default `0.0.0.0`, `::`, LAN or Public address.

Security-sensitive origin logic uses explicitly configured:

```text
external_scheme
external_host
```

rather than arbitrary incoming Host inference.

## 8.2 Host / proxy / path validation

Allowed Host comes only from configured public Host and explicitly allowed local access Host.

Unexpected Host:

```text
400 / 403
```

Forwarded / X-Forwarded-* / CF-Connecting-IP are trusted only across the expected local Tunnel/proxy boundary and never bypass Authentication.

Request-path security uses routing-equivalent normalized semantics and tests duplicate slash, trailing slash, encoded slash/dot, `..`, mixed case, query, malformed Host and proxy spoof variants.

## 8.3 Browser headers

Web UI uses:

```text
Content-Security-Policy
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Permissions-Policy
```

CSP baseline:

```text
default-src 'self'
object-src 'none'
base-uri 'none'
frame-ancestors 'none'
```

No `unsafe-eval`. External arbitrary CDN script execution is prohibited. Inline script requires nonce/hash rather than blanket `unsafe-inline`.

Authenticated pages/business APIs:

```text
Cache-Control: no-store
```

Hashed static assets may use immutable caching.

## 8.4 HSTS

SECURITY owns HSTS policy. `DEPLOYMENT.md` supplies the actual configured public hostname/context.

For a configured production public HTTPS origin:

```text
Strict-Transport-Security: max-age=<at least 15552000>
```

`includeSubDomains` and `preload` are **off by default**. Either may be enabled only after the complete affected domain scope is separately reviewed and known to be HTTPS-safe.

A Cloudflare trust-boundary / public-host / HSTS change requires Host/proxy/HSTS security review.

## 8.5 Cloudflare Access and CORS

Cloudflare Access is optional defense in depth, never a replacement for O3Pilot Authentication/Session/CSRF.

If Access is enabled, Webhook reachability is separately designed; the entire application is not made anonymous merely for Webhook.

Browser API is Same Origin by default. `Access-Control-Allow-Origin: *` with credentialed cookies is forbidden.

---

# 9. Ozon Read-Only Security Gate

Ozon read/write permission is determined by endpoint business semantics, not HTTP method.

```text
POST != Write
GET != Automatically Safe
```

Seller API, Performance API and other providers use distinct Registries.

Allowed external operations define, at minimum:

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

Security semantic classes include:

```text
READ
AUTH_BOOTSTRAP
PROHIBITED_WRITE
```

Performance `client_credentials` token acquisition may be `AUTH_BOOTSTRAP`; it is not an Ozon business mutation.

Unregistered operation:

```text
DENY
```

Business modules receive typed approved Gateways, not an arbitrary `request(method,url,headers,payload)` escape hatch.

Credential may only be sent to the exact Provider Host assigned to that credential type. Cross-host redirects do not retain credentials unless target Host is independently revalidated.

Explicitly prohibited Ozon capability families include:

```text
product create/update/delete
product-content mutation
price mutation
stock mutation
order/posting fulfillment mutation
warehouse/delivery configuration mutation
campaign/bid/budget mutation
promotion mutation
Question/Review/Chat send/reply
Webhook subscription set/update/enable/delete
any other endpoint changing Ozon server-side business state
```

Credential permission answers what the credential *could* access; the O3Pilot Registry answers what O3Pilot *may* call. Effective ability is the intersection.

An Ozon allowlist-model change requires positive/negative read-only gate regression tests.

---

# 10. Secret & Credential Protection

## 10.1 Storage boundary

O3Pilot-owned secrets are not stored in:

```text
Git
committed .env
ordinary JSON/YAML config
plaintext ordinary business columns
process command line
logs
browser storage
crash dump by design
```

Environment variables are not the formal long-term O3Pilot Secret Store.

## 10.2 Instance Master Key

Initialize:

```text
instance_master_key = CSPRNG(256 bit)
```

It protects runtime Integration Secrets and the local copy of Backup Repository Key.

Security requirements:

- never stored in SQLite as ordinary data;
- never in Git;
- never plaintext in portable Backup;
- never logged;
- kept as independent host-local Secret material with access limited to the O3Pilot runtime identity.

Concrete filesystem location/mode/service injection is owned by `DEPLOYMENT.md`.

## 10.3 Runtime Secret encryption

```text
AES-256-GCM
```

Each Secret uses independent random nonce and AAD binding, including sufficient context such as:

```text
instance_id
secret_type
shop_id / integration_id
crypto / record version
```

Moving ciphertext into a wrong context must fail authentication.

SECURITY requires persisted encrypted Secret state to retain enough ciphertext, nonce/AEAD context and crypto/version identity for correct decryption and migration. It does not freeze the complete physical database record schema.

## 10.4 Access Token

Short-lived access token is memory-first. If crash-recovery persistence is required, use the same encrypted Secret boundary.

Expired token is not historical business data and is not logged.

## 10.5 Rotation

Support rotation for Seller API Key, Performance Client Secret, R2 Credential, Webhook Endpoint Secret and secret-crypto format as applicable.

Safe rotation:

```text
validate new material
→ persist encrypted new material
→ atomically change active reference
→ invalidate/remove old active material when safe
```

Rotation must not require plaintext historical copies or duplicate deployment lifecycle contracts.

Changes to Secret Encryption Format or Instance Master Key Format require explicit persisted-crypto migration and regression tests.

---

# 11. Webhook Security

Webhook is a Public Surface and is untrusted input.

Until an officially verified cryptographic signature contract exists, the current authentication boundary uses an independent high-entropy Webhook Endpoint Secret as bearer-secret protection.

O3Pilot does not claim unverified Ozon signature semantics.

Endpoint requirements:

```text
secret comparison performed safely
strict body-size limit
strict JSON/content validation
unknown/invalid event handled safely
idempotency/replay handling
acknowledgement separated from asynchronous processing
no direct irreversible business mutation from raw event
no Secret logging
```

Webhook readback/reconciliation remains necessary for final business state.

A Webhook Authentication mechanism change requires dedicated security regression tests.

---

# 12. Upload, Input, SQL & Output Safety

Uploaded CSV/XLS/XLSX is untrusted input.

Requirements:

```text
explicit format allowlist
do not trust extension or MIME alone
file-size / row / cell / archive limits
macro/embedded executable content never executed
parser failure fails safely
temporary upload removed after processing/failure
formula-injection-safe export behavior
```

New upload format requires parser/allowlist/resource-limit security review before support.

SQL:

- parameterized queries;
- no user-supplied raw SQL execution;
- dynamic identifiers require explicit allowlist.

XSS:

- framework/contextual escaping;
- no unsafe business-text HTML injection;
- CSP remains defense in depth.

SSRF/outbound:

- external URLs come from provider configuration/allowlist, not arbitrary payload;
- redirect target is revalidated;
- credentials are not forwarded to unapproved hosts.

---

# 13. Local Data, Privacy, Logging & Security Audit

## 13.1 Local protection outcome

Runtime identity and physical persistent-directory/file permissions are implemented by `DEPLOYMENT.md`.

SECURITY requires the result:

```text
runtime does not run as root
business data is not world-readable
Secret/sensitive files are access-restricted
primary SQLite/Raw protection is not weakened by backup/log/temp handling
```

Concrete path or POSIX mode is a Deployment implementation detail unless a security test requires a minimum equivalent protection.

## 13.2 Data minimization / privacy

Do not create unnecessary duplicate copies of personal/sensitive business text.

UI and third-party notifications expose the minimum needed data. Third-party notification payload must not silently include SECRET or unnecessary SENSITIVE content.

## 13.3 Logging / error sanitization

Never log canonical redaction targets from §4.6.

Business-body logging is minimized and off by default for SENSITIVE payloads.

Errors are sanitized before persistent logging or UI display.

## 13.4 Security Audit minimum

Audit at least security-relevant events:

```text
successful / failed Login
Session creation / revocation / expiry anomalies as needed
Password Change
read-allowlist / prohibited-endpoint denial
outbound-host denial
Integration credential add/change/delete/verification failure
Webhook secret rotation / authentication failure
upload rejection of security relevance
Backup create/verify security failure
Restore authorization / start / success / failure
Recovery Key / Recovery Envelope security operation
crypto/secret migration
security-wide Session invalidation
```

There is no Password Reset audit event in v1 because there is no in-place Password Reset capability.

The two denial axes remain semantically distinct:

```text
Ozon read/prohibited-capability denial
Outbound host/credential isolation denial
```

Audit itself must not contain raw SECRET.

---

# 14. Backup Security & Recovery Key

This section defines security semantics for the canonical Backup Artifact owned operationally by `DEPLOYMENT.md`.

## 14.1 Confidentiality and key separation

Backup must provide confidentiality and integrity before leaving the host.

Use a random:

```text
Backup Repository Key = CSPRNG(256 bit)
```

to protect Backup objects. Host runtime holds a locally protected copy under `instance_master_key`.

Independent:

```text
Recovery Key = CSPRNG(256 bit)
```

Recovery Key is:

- independent from Owner Password;
- independent from Instance Master Key;
- not normal config;
- not uploaded to R2;
- not logged;
- intended to be stored off-host by the user.

Losing both Recovery Key and usable trusted recovery material can make encrypted Backup unrecoverable.

## 14.2 Recovery Envelope

```text
Recovery Key
→ standard KDF / KEK
→ wraps Backup Repository Key
→ Recovery Envelope
```

Recovery Key is high-entropy random material, not a human password.

KDF uses a reviewed construction such as HKDF-SHA-256 with independent salt/context (Repository identity / format identity as applicable).

Do not reuse raw Recovery Key bytes directly as unrelated cryptographic keys.

## 14.3 Backup encryption

Backup object encryption:

```text
AES-256-GCM
```

Requirements:

- independent random nonce per object;
- no nonce reuse;
- AAD binds repository/object/format context;
- authentication-tag failure is Fail Closed.

Backup Manifest is encrypted because it can reveal sensitive structure.

A minimal non-sensitive plaintext envelope may expose only fields necessary to select crypto/format handling, e.g.:

```text
backup_format_version
cipher_suite
```

No Shop name, Client ID, order data, secret metadata or source-host paths in plaintext envelope.

## 14.4 Integrity

Verification covers, as applicable:

```text
AEAD authentication
object length
object identity
required object presence
Manifest references
SQLite integrity
config / schema / format compatibility
```

Checksum alone does not replace AEAD authentication.

## 14.5 Optional Integration Secret inclusion

A Backup may intentionally exclude Integration Secrets.

Such Backup is explicitly marked:

```text
DATA_COMPLETE_BUT_CREDENTIALS_REQUIRED
```

Restored integrations remain:

```text
CREDENTIALS_REQUIRED
```

until credentials are newly provided/validated.

Owner Credential State is not portable and is never included as an overwriteable restored Owner credential.

## 14.6 Recovery Key / Repository Key rotation

Recovery Key suspected leak:

```text
generate new Recovery Key
→ re-wrap Backup Repository Key
→ verify new Recovery Envelope
→ invalidate old Envelope
```

Historical Backup objects need not all be re-encrypted unless the Repository Key itself is compromised.

Repository Key compromise requires a new Repository Key and migration/re-encryption strategy for retained data.

Changes to Backup Encryption Format or Recovery Key/Envelope Format require explicit compatibility/migration behavior and Backup/Restore regression tests.

---

# 15. Cloudflare R2 Security

Operational R2 role lives in `DEPLOYMENT.md §12.2`.

SECURITY requires:

```text
R2 = optional private off-device encrypted Backup replica
not primary database/filesystem/secret store
```

Backup is client-side encrypted **before** upload. R2-at-rest encryption is only defense in depth.

Bucket:

- Private by default;
- no public Backup bucket/custom-domain exposure;
- least-privilege credential;
- no Cloudflare Global API Key reuse;
- no frontend storage of R2 credential.

R2 credential may be part of portable encrypted Integration Secret state if explicitly included. But new-host R2 recovery still requires a separately supplied Bucket-access bootstrap credential.

```text
Recovery Key != R2 Authentication Credential
```

R2 credential leak should not reveal plaintext Backup, but may enable deletion/tampering/availability attacks. Local verified Backup and remote replica are independent resilience states.

---

# 16. Restore Security

There is one canonical SECURITY Restore contract. `DEPLOYMENT.md §15` owns the operational sequence and consumes this contract.

No second umbrella Restore-authorization vocabulary is introduced.

## 16.1 Authorization and operation binding

Restore requires valid current Owner authorization on the target instance.

Browser:

```text
Owner Password re-auth
→ Session.reauthenticated_at
→ create operation-bound Restore authorization state
```

CLI:

```text
direct verification of current Owner Password
→ create process-local operation-bound Restore authorization state
```

Authorization state is bound to:

```text
Restore operation identity
selected Backup/source identity
target instance
```

It is not a reusable bearer grant and is invalidated on cancellation/failure/completion.

Recent Authentication age at creation is at most five minutes. A Restore already authorized does not expire solely because staging/migration runs longer.

## 16.2 Required independent checks

Restore security requires all of:

```text
current Owner authorization
explicit Backup/source selection
Recovery Key verification
Backup integrity verification
format/schema compatibility
required recoverable point before destructive replacement
explicit destructive confirmation in the Deployment operation
```

Roles remain distinct:

```text
Owner Password / Recent Authentication
= authorize operation

Recovery Key
= unlock Backup cryptographic material

trusted OS management privilege
= local machine control / bootstrap environment
!= Owner authentication on an initialized instance

R2 Bootstrap Credential
= access private R2 Bucket
!= Owner authentication
!= Recovery Key
```

## 16.3 Session and Owner state after Restore

All pre-Restore Sessions are revoked.

Session state is never Portable Business State.

Owner Credential State is not Portable Business State and does not overwrite the target Owner credential.

Cross-host ordering therefore is:

```text
install target
→ local bootstrap
→ create target Owner Credential
→ authenticate / recently re-authenticate target Owner
→ select Backup
→ provide Recovery Key
→ Restore
```

## 16.4 Cross-host Secret rewrap

Cross-host Restore:

```text
generate new target instance_master_key
→ decrypt portable Secret state
→ re-protect Secret state under target-host key material
```

Old host-local master key is never transplanted.

## 16.5 Decrypted temporary material

Decrypted temporary Restore material must:

```text
be access-restricted
never enter logs
be removed on success
be removed on failure
exist in plaintext for the minimum practical lifetime
```

Concrete temp path/mode is owned by `DEPLOYMENT.md`.

## 16.6 Failed Restore

Wrong Recovery Key, object tamper, Manifest inconsistency, AEAD failure or SQLite integrity failure:

```text
Fail Closed
```

Partial failed Restore must not be promoted or marked normal.

Persistent/security format incompatibility must not be guessed around or silently reinterpreted.

---

# 17. Background Jobs & Outbound Network Security

Worker/Scheduler/API share the same security boundaries.

Background Job must not:

- bypass Ozon Endpoint Registry;
- accept arbitrary URL/SQL/Endpoint injection through Job payload;
- persist plaintext SECRET in payload/error;
- bypass Owner Authentication/CSRF on administrative retry/control API;
- retain/use a pre-Restore Session after Restore.

Each external provider Gateway has its own:

```text
Host Allowlist
Credential Type
TLS Requirement
Timeout
Retry policy
```

External API requires HTTPS with certificate verification.

Production must not use:

```text
verify_tls = false
```

All external requests have connect/read/overall deadlines.

---

# 18. Dependency & Supply Chain Security

Minimize unnecessary framework plugins, crypto libraries, parsers and background services.

Do not implement custom AES, Argon2, random generator, MAC/JWT/password hash/signature primitives when mature reviewed libraries exist.

Production dependencies resolve reproducibly to known versions. Dependency upgrade includes release/security-note review, automated tests and security regression; production startup does not dynamically pull an unknown newest dependency.

Frontend runtime does not execute arbitrary third-party CDN scripts. Production JS/CSS/icon assets are served by O3Pilot unless a separately reviewed dependency contract says otherwise.

Known critical dependency vulnerability without mitigation blocks release.

---

# 19. Upgrade, Security-state Migration & Incident Response

## 19.1 Upgrade / migration

Pre-upgrade Backup operation is owned by `ARCHITECTURE.md` / `DEPLOYMENT.md`; SECURITY requires it before an upgrade that can modify security-sensitive persistent state.

Secret/security-state migration:

- no long-lived plaintext temporary copy;
- crash-aware;
- new format verified before old encrypted state is retired;
- failure leaves a recoverable state;
- old version must refuse unsupported new security/schema formats rather than silently overwrite.

**Independent migration invariant:**

> Persisted security / cryptographic format changes require an explicit Migration; old ciphertext or persisted security state must never be silently reinterpreted.

This applies to Password verifier format, Session/security state format, Secret encryption, Instance Master Key wrapping, Backup encryption, Recovery Envelope and other persisted security formats.

## 19.2 Incident response

### Owner Password suspected compromise

```text
change Owner Password
→ revoke all Sessions
→ review Login / Session audit
→ if browser/host compromise is also plausible, rotate affected Integration Secrets
```

### Ozon API Key leak

User revokes/rotates in Ozon, then updates/verifies O3Pilot credential. O3Pilot does not call Ozon write endpoints to “repair” state.

### Performance Secret leak

Revoke old Client Secret, configure new Secret, clear old Access Token and obtain a new token.

### Webhook Endpoint Secret leak

Generate new endpoint Secret; user performs any required Ozon-side Webhook URL change manually. Old Secret is invalidated. O3Pilot does not modify Ozon Webhook subscription.

### R2 Credential leak

Rotate R2 credential, verify local Backup and remote object integrity. R2 ciphertext does not imply plaintext exposure if Backup keys remain secret.

### Recovery Key leak

Rotate Recovery Envelope/Recovery Key as defined in §14.6.

### Instance-wide security incident

When incident scope cannot safely be limited to one Session, revoke **all** active Sessions. There is no single-session assumption.

---

# 20. Mandatory Security Tests

Formal Release includes at least the following.

## 20.1 Authentication / Session / initialization

- unauthenticated Protected API → reject;
- one successful login does **not** revoke unrelated valid Sessions;
- Logout invalidates current Session;
- Password Change invalidates all Sessions;
- expired Session invalidates;
- raw Session Token absent from DB/log;
- server stores token digest/verifier;
- initialization token is UNINITIALIZED-only, Loopback/local-forwarding-only, single-use, ≤15 min, digest-only server-side and invalidated on success/restart;
- no in-place Password Reset endpoint/CLI/authentication backdoor;
- Recent Authentication >5 min cannot establish a new sensitive-operation authorization;
- CLI sensitive operation does not inherit browser reauth state.

## 20.2 Path / Host / CSRF / HSTS

Test malformed Host, duplicate/encoded path variants, proxy spoof and same-origin/cross-origin CSRF behavior.

Configured public HTTPS origin emits HSTS with required minimum max-age. `includeSubDomains`/`preload` are absent unless explicitly reviewed/enabled.

A Cloudflare trust-boundary/HSTS change requires regression coverage.

## 20.3 Ozon read-only

- unregistered Endpoint → reject;
- prohibited write Endpoint → reject;
- no arbitrary Ozon URL from business module;
- Seller credential never sent to Performance host;
- redirect to unapproved host does not receive credentials;
- even Admin-role credential cannot make write path reachable.

## 20.4 Webhook

Wrong/missing Secret rejected; oversized body rejected; invalid JSON safely rejected; unknown event safely quarantined/handled; duplicate event causes no repeated irreversible mutation; Secret absent from logs.

Webhook Authentication changes require this regression suite.

## 20.5 Upload / parser

Fake extension, bad MIME/content, corrupt XLSX, archive bomb, oversized cells/rows, macro/embedded content, Formula Injection export and parser-crash temp cleanup.

Any newly supported upload format must pass the same parser/limit/threat review.

## 20.6 Secret / redaction

- known O3Pilot SECRET plaintext absent from DB ordinary fields/config/log/Backup;
- wrong Instance Master Key cannot decrypt;
- tampered ciphertext fails AEAD;
- canonical redaction catches SECRET values and credential-bearing headers/tokens/URLs;
- cloudflared/Tunnel credential material, if synthetically injected into diagnostics/error output, is redacted and not persisted/backed up.

Secret Encryption / Master Key format changes require migration regression.

## 20.7 Backup / R2 / Restore

- Backup business plaintext not directly exposed;
- SECRET plaintext absent;
- wrong Recovery Key → fail;
- tampered/missing object → fail;
- R2 stores ciphertext;
- all pre-Restore Sessions invalid after Restore;
- cross-host Restore creates new target master key and does not transplant old local key;
- Owner Credential State is not imported;
- target Owner must authenticate before Restore authorization;
- Recovery Key alone cannot authorize Restore;
- operation authorization binds exact source/target and is not reusable;
- decrypted temp material is restricted, not logged and cleaned on success/failure;
- failed Restore cannot promote partial state.

Backup Encryption / Recovery Envelope format changes require explicit compatibility/migration testing.

## 20.8 Audit / outbound / dependency

Verify both Ozon read/prohibited-capability denial and outbound-host denial remain observable.

No credential follows an unapproved cross-host redirect.

Critical dependency finding without mitigation blocks release.

---

# 21. Security Acceptance Gate

Release is blocked by any of:

```text
Authentication Bypass
CSRF Bypass
Public initialization takeover
Single-Session residue that revokes unrelated Sessions on login
Ozon Write Path Reachable
Secret Plaintext in DB / Log / Backup
Canonical Redaction Gap
Arbitrary Outbound URL with Credential
Unbounded / unsafe public upload
Webhook Secret Leakage
Backup Integrity Not Verifiable
Recovery Key accepted as Owner authentication
Restore not bound to current Owner authorization
Restore Keeps Old Session Valid
Cross-host Restore transplants old local master key
Restore temp plaintext leak / failed cleanup
Persisted security format silently reinterpreted
Known Critical Dependency Vulnerability without Mitigation
```

High Severity Security Finding blocks Release by default.

Accepted residual risk requires:

```text
Finding ID
impact
temporary mitigation
remediation plan
```

“theoretically nobody will do this” is not mitigation.

Security-impact changes must update the owning security section/tests, not a parallel security-version wrapper.

Change categories requiring explicit security regression/coverage include:

- Password Hash Algorithm;
- Session Format/state;
- CSRF Strategy;
- Secret Encryption Format;
- Instance Master Key Format;
- Backup Encryption Format;
- Recovery Key / Envelope Format;
- Webhook Authentication;
- Ozon Endpoint Allowlist Model;
- Cloudflare Trust Boundary / HSTS scope;
- any new public entry point;
- any new upload format;
- any new third-party notification / Integration.

General release/change history remains **Project Version + CHANGELOG**. There is no parallel Security Version numbering system.

---

# 22. Explicitly Prohibited Implementations

Forbidden:

```text
Ozon API Key in source code
committed plaintext Secret .env
Session token in LocalStorage
plaintext Password
direct SHA-256 password hashing
trust arbitrary Host Header
frontend-only authorization
Cloudflare Access as sole authentication
SameSite as sole CSRF defense
generic arbitrary Ozon request(url,payload)
follow arbitrary redirects while retaining Authorization
fetch arbitrary URL from external payload
execute uploaded Excel macro
retain uploaded original file indefinitely
dump full Raw payload into Debug log
upload plaintext Backup to R2
store Recovery Key inside the same Backup
treat Recovery Key as Owner authentication
Restore while accepting pre-Restore Sessions
portable Owner Credential State overwrite
bypass integrity/compatibility for Restore
test Ozon write capability by actually mutating Ozon
```

---

# 23. Residual Risk & Implementation Boundary

Even with this contract, residual risks remain.

## 23.1 Over-privileged external Ozon credential

O3Pilot can guarantee its own write path is unreachable, but cannot stop a leaked Admin credential from being used outside O3Pilot.

Use least-privilege Ozon credential where available; O3Pilot security does not depend on it.

## 23.2 Browser / device compromise

Malware fully controlling an authenticated browser/device can act with that Session's read authority and may observe business data.

## 23.3 Host full compromise

An attacker with sufficient host/root/runtime-memory control can ultimately access runtime plaintext Secret material.

## 23.4 Webhook authenticity

Until a verified Ozon cryptographic signature contract is available, Webhook Endpoint Secret is bearer-secret authentication, not public-key origin proof.

## 23.5 Cloudflare account compromise

Compromise may affect Tunnel, DNS, R2 availability and routing. Client-side Backup encryption reduces plaintext exposure but cannot prevent remote deletion/denial.

## 23.6 Lost Owner password without recovery source

Because v1 intentionally has no in-place Password Reset/backdoor:

```text
lost Owner password
+
no usable external verified Backup / recovery source
=
protected instance data may be unrecoverable through supported O3Pilot recovery
```

This is an explicit consequence of the authentication boundary.

## 23.7 Implementation boundary

This document defines the intended v1 Security Contract. Some controls may still require implementation work.

Physical installation/service/path operations belong to `DEPLOYMENT.md`; runtime/persistent-state architecture belongs to `ARCHITECTURE.md`.

A retained security rule must have a clear security reason, one canonical authority and testable value. Historical wording is not itself a reason to preserve a rule.
