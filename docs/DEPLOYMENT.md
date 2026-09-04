# O3Pilot — DEPLOYMENT.md

> Version: 1.0  
> Status: Frozen Deployment & Operations Baseline  
> Updated: 2026-09-04  
> Applies to: O3Pilot

# 1. 文档目的

`DEPLOYMENT.md` 定义 O3Pilot 的正式安装、运行、升级、备份、恢复、迁移、卸载和日常运维规范。

本文件负责回答：

- O3Pilot 正式支持部署在哪些操作系统和 CPU 架构；
- Application、Database、Raw、Backup、Config、Secret、State、Temp 与 Logs 如何落盘；
- macOS 与 Linux 如何做到 Single Application Root，避免在系统中散落 O3Pilot 自有文件；
- O3Pilot 如何作为长期运行的系统服务启动；
- O3Pilot 如何只监听 Loopback，并通过 Cloudflare Tunnel 提供公网访问；
- Cloudflare R2 如何作为可选 Off-device Backup Replica；
- 首次安装如何建立 Owner、Instance 与 Recovery Key；
- 正式 Release 如何安装、升级、回滚；
- Local Backup、Portable Backup 与 Restore 如何执行；
- 卸载时如何区分删除应用与彻底删除实例。

本文件不重新定义：

- 产品能力与业务边界；
- Ozon API 数据范围与数据源验证状态；
- 业务实体、字段、主键和数据库 Schema；
- 指标、Profit、Forecast 公式；
- 密码 Hash、Backup Cipher、Key Derivation 等密码学算法；
- UI、导航和页面设计。

这些内容分别由 `PRODUCT.md`、`DATA_SOURCES.md`、`DATA_MODEL.md`、`METRICS.md`、`ARCHITECTURE.md`、`SECURITY.md`、`DESIGN.md` 等文档定义。

---

# 2. Deployment Baseline

O3Pilot v1 正式部署基线：

```text
Native Installation
macOS Apple Silicon only
Linux
Single Machine
Single O3Pilot Primary Instance
Unprivileged Runtime
Local Persistent SQLite
Local Raw Store
Local Backup Repository
Cloudflare Tunnel for Remote Access
Optional Cloudflare R2 Backup Replica
Persistent Data Separated from Replaceable Application Releases
Pre-upgrade Backup
Compatibility-aware Upgrade
Recoverable Rollback
No Docker
No Public Application Port
No Remote Primary Database
```

正式运行拓扑：

```text
Browser
   │ HTTPS
   ▼
Cloudflare
   │
   ▼
Cloudflare Tunnel
   │ local origin
   ▼
127.0.0.1:<o3pilot-port>
   │
   ▼
O3Pilot
├── API Runtime
├── Worker Runtime
└── Scheduler Runtime
   │
   ├── SQLite Primary Database
   ├── Local Raw Store
   ├── Local Backup Repository
   └── Encrypted Secret Store

Local Backup Repository
   │
   └── optional encrypted replica
          ▼
     Cloudflare R2
```

`cloudflared` 与 O3Pilot 是两个独立系统服务。

R2 不参与正常数据库读写。

---

# 3. 正式支持范围

O3Pilot 的 `Production Supported` 只表示已经进入正式构建、安装、启动、升级、Backup、Restore 与 Acceptance Matrix 的平台，不等同于“理论上可能运行”。

## 3.1 macOS

O3Pilot v1 正式支持：

```text
Minimum macOS: macOS 26 Tahoe
Architecture: Apple Silicon / arm64 only
Installer condition: Darwin + arm64 + macOS major version >= 26
```

支持规则：

```text
macOS 26 Tahoe
    → Production Supported minimum baseline

Newer macOS
    → allowed by default
    → Production Acceptance 仍以经过验证的正式系统版本为准

Beta macOS
    → installation allowed
    → not required in Production Release Acceptance Matrix
```

明确不支持：

```text
macOS 25 and earlier
Intel Mac
macOS x86_64
Rosetta 2 Production Deployment
```

不得通过 Rosetta 2 将 x86_64 构建作为正式 macOS 兼容方案。

## 3.2 Linux

O3Pilot v1 正式支持矩阵固定为：

| Distribution | Version | x86_64 | arm64 |
|---|---:|---:|---:|
| Ubuntu LTS | 24.04 | Supported | Supported |
| Ubuntu LTS | 26.04 | Supported | Supported |
| Debian | 13 | Supported | Supported |

Linux Host 还必须满足：

```text
64-bit OS
systemd
standard glibc-based userspace
local persistent filesystem
```

明确不进入 v1 Production Support：

```text
Ubuntu 22.04 and earlier
Ubuntu non-LTS
Debian 12 and earlier
Fedora
RHEL
Rocky Linux
AlmaLinux
Arch Linux
Manjaro
openSUSE
Linux Mint
Alpine Linux / musl-only Linux
OpenRC-only environment
WSL
32-bit Linux
```

可能在其他 Linux 环境中运行不等于 `Production Supported`。Installer 对明确识别为不支持的平台必须：

```text
UNSUPPORTED PLATFORM
→ ABORT INSTALL
```

不得在不支持的平台上继续“尽力安装”。

## 3.3 不属于 v1 正式部署

```text
Intel Mac
macOS x86_64
Rosetta 2 Production Deployment
Windows Native
WSL
Docker
Docker Compose
Kubernetes
NAS Container
Serverless Runtime
Cloudflare Workers Runtime
Remote SQLite
SQLite over NFS / SMB / WebDAV / R2 Mount
Shared Multi-host Database
```

PostgreSQL、Cloudflare D1、Hyperdrive 等均不是 O3Pilot v1 运行依赖。

---

# 4. Runtime Identity

正式 O3Pilot Runtime 必须以非 root OS Identity 运行。

```text
O3Pilot Runtime != root
```

安装器可以在创建系统目录、设置权限、注册 `launchd` / `systemd` 等必要阶段请求管理员权限，但管理员权限不能成为 O3Pilot 长期 Runtime 权限。

O3Pilot Web Owner 与 OS Runtime Identity 是不同概念：

```text
OS Runtime Identity
!=
O3Pilot Owner
```

---

# 5. macOS Single Application Root Contract

## 5.1 正式根目录

macOS 的 O3Pilot 正式根目录固定为：

```text
O3PILOT_ROOT=/Library/O3Pilot
```

除 macOS 强制要求位于系统固定位置的 Service Definition 外，O3Pilot 自己创建、下载、生成、维护的所有长期文件和运行文件必须全部位于：

```text
/Library/O3Pilot/
```

O3Pilot 不得在以下位置创建长期持久化数据：

```text
~/Library
~/Desktop
~/Documents
/tmp
/Library/Application Support/O3Pilot
/Library/Logs/O3Pilot
其他与 O3Pilot Root 无关的目录
```

目标是：

> 用户能够通过一个根目录明确知道 O3Pilot 自己管理了哪些文件，并能够进行容量检查、迁移、诊断和彻底清理。

## 5.2 macOS 目录结构

正式逻辑结构：

```text
/Library/O3Pilot/
├── bin/
│   └── o3pilotctl -> ../app/current/bin/o3pilotctl
├── app/
│   ├── releases/
│   │   ├── <version-a>/
│   │   ├── <version-b>/
│   │   └── ...
│   └── current -> releases/<active-version>/
│
├── data/
│   ├── o3pilot.db
│   └── raw/
│       ├── api/
│       └── webhook/
│
├── backups/
│   └── repository/
│
├── config/
│
├── secrets/
│
├── state/
│   ├── run/
│   ├── staging/
│   └── tmp/
│
└── logs/
```

其中：

```text
bin       → Stable local Operator CLI entry point
app       → 可替换 Application Release
data      → SQLite 与自动来源 Raw Data
backups   → Local Backup Repository
config    → Portable / Runtime Configuration
secrets   → 本机受保护 Secret State
state     → Lock、Staging、Temporary Runtime State
logs      → O3Pilot 自有运行日志
```

## 5.3 唯一允许的 macOS 外部文件

如果采用系统级 `launchd`，macOS 要求 Service Definition 位于系统规定位置，例如：

```text
/Library/LaunchDaemons/com.o3pilot.app.plist
```

因此正式 macOS 安装原则为：

```text
/Library/O3Pilot/
    → O3Pilot 自己管理的全部 Application / Data / Config / Secret / Backup / State / Temp / Logs

/Library/LaunchDaemons/com.o3pilot.app.plist
    → macOS 强制位置的 Service Definition
```

不得因为 Service Definition 必须位于 `/Library/LaunchDaemons`，就继续在其他目录散落 O3Pilot 自有数据。

`cloudflared` 自己的程序和配置属于 Cloudflare 软件自身，不属于 O3Pilot Single Root Contract；O3Pilot 不接管 cloudflared 的文件体系。

---

# 6. Linux Single Application Root Contract

## 6.1 正式根目录

Linux 的 O3Pilot 正式根目录固定为：

```text
O3PILOT_ROOT=/opt/O3Pilot
```

除 Linux 强制要求位于系统固定位置的 Service Definition 外，O3Pilot 自己创建、下载、生成、维护的所有长期文件和运行文件必须全部位于：

```text
/opt/O3Pilot/
```

O3Pilot 不得把自己的长期 Application、Database、Raw、Backup、Config、Secret、State、Temporary Files 或 Logs 分散到：

```text
/var/lib/o3pilot
/etc/o3pilot
/var/log/o3pilot
用户 Home
/tmp
其他与 O3Pilot Root 无关的目录
```

目标与 macOS 相同：用户能够通过一个根目录明确知道 O3Pilot 自己管理了哪些文件，并能够进行容量检查、迁移、诊断和彻底清理。

## 6.2 Linux 目录结构

正式逻辑结构：

```text
/opt/O3Pilot/
├── bin/
│   └── o3pilotctl -> ../app/current/bin/o3pilotctl
├── app/
│   ├── releases/
│   │   ├── <version-a>/
│   │   ├── <version-b>/
│   │   └── ...
│   └── current -> releases/<active-version>/
│
├── data/
│   ├── o3pilot.db
│   └── raw/
│       ├── api/
│       └── webhook/
│
├── backups/
│   └── repository/
│
├── config/
│
├── secrets/
│
├── state/
│   ├── run/
│   ├── staging/
│   └── tmp/
│
└── logs/
```

其中目录职责与 macOS Single Application Root 完全一致。

## 6.3 唯一允许的 Linux 外部文件

正式 Linux 使用 `systemd` 管理 O3Pilot 服务时，Service Definition 位于系统规定位置，例如：

```text
/etc/systemd/system/o3pilot.service
```

因此正式 Linux 安装原则为：

```text
/opt/O3Pilot/
    → O3Pilot 自己管理的全部 Application / Data / Config / Secret / Backup / State / Temp / Logs

/etc/systemd/system/o3pilot.service
    → Linux systemd Service Definition
```

不得因为 systemd Service Definition 必须位于系统目录，就继续在 `/var/lib`、`/etc`、`/var/log` 或其他目录散落 O3Pilot 自有数据。

`systemd` 自身维护的 Journal、PID、cgroup 等 Runtime State 属于操作系统服务管理体系，不属于 O3Pilot Single Root Contract。

`cloudflared` 自己的程序和配置属于 Cloudflare 软件自身，不属于 O3Pilot Single Root Contract；O3Pilot 不接管 cloudflared 的文件体系。

---

# 7. File Permission Contract

默认权限原则：

```text
Default Deny
Runtime User Read/Write Only where possible
No Secret World-readable
No Business Data World-readable
```

macOS `/Library/O3Pilot` 与 Linux `/opt/O3Pilot` 应由 O3Pilot Runtime Identity 与必要管理员控制。

Secret 文件应使用等价于：

```text
0600
```

Secret / Data 等敏感目录应采用等价于：

```text
0700
```

的权限基线。

正式生产主机建议启用：

```text
macOS → FileVault
Linux → LUKS 或等效全盘加密
```

---

# 8. Manual Upload Staging

CSV、XLS、XLSX 等用户上传文件只允许进入临时 Staging。

平台对应 Staging 目录：

```text
macOS → /Library/O3Pilot/state/staging/
Linux → /opt/O3Pilot/state/staging/
```

处理流程：

```text
Upload
↓
Validate
↓
Parse
↓
Persist allowed structured facts + lineage
↓
Delete source file
```

人工导入文件只是一次性 Import Medium，不属于 O3Pilot 长期 Raw Store。

不得长期保存用户上传原文件，也不得把这些文件复制到 R2 Backup。

---

# 9. Local Storage Requirement

Primary SQLite 与 Local Raw Store 必须位于本机持久化文件系统。

正式禁止：

```text
NFS
SMB
WebDAV
R2 Mount
FUSE-backed remote object storage
Cloud Drive synchronized directory
```

作为 O3Pilot Primary Storage。

尤其不得把正式 `data/` 放入：

```text
iCloud Drive
Dropbox
OneDrive
Google Drive
Synology Drive Sync Folder
```

---

# 10. Local Port Contract

O3Pilot 默认只监听：

```text
127.0.0.1
```

禁止默认监听：

```text
0.0.0.0
::
LAN IP
Public IP
```

v1 默认 Application Port：

```text
38652
```

安装时必须显式检查端口占用。

端口冲突时不得静默寻找随机新端口；必须停止安装或要求显式指定替代值。

---

# 11. Application Release Model

正式生产 O3Pilot 不使用：

```text
git clone
git pull
git checkout main
```

作为 Production Release 管理机制。

## 11.1 Production Artifact Contract

O3Pilot v1 Production Artifact 固定为平台 / 架构独立构建：

```text
o3pilot-<version>-macos-arm64.tar.gz
o3pilot-<version>-linux-x86_64.tar.gz
o3pilot-<version>-linux-arm64.tar.gz
```

不提供 Universal Artifact。

Production Artifact 必须满足：

```text
Platform-specific
Architecture-specific
Immutable after publication
Self-contained
Pre-built
No source build on target host
No external Python / Node runtime requirement
No Instance Data
.tar.gz transport format
One Release = one immutable release directory
Frontend + Backend from the same Release
```

最终用户主机不得为了运行正式 Artifact 额外安装：

```text
Python
pip
virtualenv
Node.js
npm / pnpm / yarn
Git
compiler toolchain
frontend build tooling
```

具体内部打包技术可以演进，但正式 Artifact 对目标主机暴露的行为合同不得改变。

一个 Release 的概念结构可以为：

```text
o3pilot/
├── bin/
│   ├── o3pilot
│   └── o3pilotctl
├── runtime/
├── app/
├── web/
├── migrations/
├── resources/
├── release-manifest.json
└── LICENSE
```

Artifact 不得包含：

```text
.git
.github
development environment
tests / test caches
.env
secrets / credentials
user config
SQLite database
Raw Data
backups
logs
uploaded source files
mock production data
IDE configuration
historical releases
```

## 11.2 Release Manifest

每个 Release 必须包含 `release-manifest.json`，至少记录：

```text
application
application_version
build_id
source_revision
build_time
platform
architecture
release_channel
artifact_format_version
persistent_format
minimum_supported_persistent_format
compatibility_manifest
artifact_checksum
```

Manifest Identity 与实际下载 Artifact、目标 OS / Architecture 或 Active Instance 不一致时必须 Fail Closed。

## 11.3 Canonical Release Distribution

O3Pilot Production Release 的唯一 Canonical Source 为：

```text
GitHub Releases
Repository: LI5ee3/O3Pilot
```

GitHub `main` / branch / working tree 不是正式 Release Source。

Release Tag 固定采用：

```text
v<semver>
```

例如：

```text
v1.2.0
```

每个 Production GitHub Release 至少包含：

```text
o3pilot-<version>-macos-arm64.tar.gz
o3pilot-<version>-linux-x86_64.tar.gz
o3pilot-<version>-linux-arm64.tar.gz
release-manifest.json
SHA256SUMS
```

正常 Stable Installer / Updater 只选择：

```text
non-Draft
non-Pre-release
formal GitHub Release
```

已经发布的 Tag / Release Artifact 不得静默覆盖；需要修复时必须发布新版本。

`o3pilot.app/install.sh`、`update.sh`、`uninstall.sh` 仍是用户生命周期入口，但 Production Artifact 实际来源为上述 GitHub Releases。

## 11.4 Release Authenticity & Immutability

O3Pilot v1 的 Production Release 必须同时采用：

```text
GitHub Release Immutability
GitHub Artifact Attestation
SHA-256
```

三者职责不同：

```text
GitHub Release Immutability
    → 发布后的 Tag / Release Assets 不得被静默替换、修改或删除

GitHub Artifact Attestation
    → 证明 Production Artifact 来自 LI5ee3/O3Pilot 的正式 GitHub Actions 构建流程，并关联 Source Revision / Workflow / Build Provenance

SHA-256
    → 验证最终下载 Artifact 的字节完整性
```

Production Release 的完整 Build / Publish 顺序由 `11.5 Release Trigger & Publication Contract` 定义。Authenticity / Immutability 阶段必须保持以下顺序语义：

```text
Explicit Production Release Build confirmation
↓
Freeze exact Source Revision
↓
Build platform artifacts in GitHub Actions
↓
Generate Artifact Attestations
↓
Generate SHA256SUMS
↓
Validate Release completeness / identity / provenance
↓
Create or populate Draft Release
↓
Explicit Publish confirmation
↓
Revalidate Draft Release
↓
Publish Release
↓
Release becomes immutable
```

任何已正式发布的 Production Release 都不得通过覆盖同一 Tag、替换同名 Artifact 或重新上传内容的方式修复。需要修复时必须发布新的 SemVer 版本。

Artifact Attestation 必须至少绑定：

```text
Repository: LI5ee3/O3Pilot
Artifact digest
Source revision
GitHub Actions workflow identity
Build provenance
```

Production Release Acceptance 必须验证 Attestation 与 Release Artifact 一致。v1 不要求最终用户主机安装 `gh` CLI，也不把 Attestation Verification 作为 Installer 的额外 Host Dependency；Installer / Updater 仍必须执行 SHA-256、Release Manifest 与 Artifact Identity 校验。

O3Pilot v1 不引入独立自定义签名信任根：

```text
GPG signing key        → Not Required
Minisign signing key   → Not Required
Cosign custom key      → Not Required
Custom O3Pilot PKI     → Not Required
```

如未来需要建立独立于 GitHub 的 O3Pilot Signing Identity，必须通过新的 Security / Deployment Contract 单独设计 Key Generation、Offline Storage、Rotation、Revocation、Recovery 与 Installer Trust Pinning，不得在实现阶段临时加入。


## 11.5 Release Trigger & Publication Contract

O3Pilot Production Release 采用双重显式人工确认。普通开发活动不得自动产生正式 Production Release。

以下事件均不得直接触发 Production Release Build 或 Publish：

```text
push to main
merge pull request
Codex commit / push
ordinary tag push
normal CI success
```

第一阶段必须由独立的 `Production Release Build` GitHub Actions Workflow 通过 `workflow_dispatch` 显式人工触发。该 Workflow 是 Production Build 的唯一正式触发入口；不得由 `push`、PR merge、tag push 或其他普通 CI 事件复用触发。触发前至少必须满足：

```text
Target code already merged into main
Normal CI passes
Target SemVer is explicitly chosen
No known Release-blocking issue remains
Operator explicitly requests Production Release Build
```

第一阶段的人工确认必须至少输入 / 确认：

```text
Version: <semver>
Source: main
Confirmation: RELEASE
```

Workflow 一旦开始，必须立即冻结并记录本次 Release Candidate 的精确 `source_revision`。后续构建、Acceptance、Manifest、Checksum 与 Attestation 都必须绑定同一个 Source Revision，不得在 Workflow 运行过程中漂移到更新的 `main`。

正式 Build 流程：

```text
Explicit Release Build confirmation
↓
Capture exact Source Revision
↓
Run full Release Acceptance Gate
↓
Build all Production Platform Artifacts
↓
Generate release-manifest.json
↓
Generate SHA256SUMS
↓
Generate Artifact Attestations
↓
Verify Artifact / Manifest / Provenance completeness
↓
Create or populate GitHub Draft Release
↓
Draft Release = READY FOR REVIEW
```

第一阶段成功只允许得到 `Draft Release`。Draft 不属于 Stable Release，正常 `install.sh` / `update.sh` 不得发现、选择或安装 Draft Release。

Production Release 的最终公开发布必须经过第二次独立显式人工确认，并且必须由独立的 `Publish Release` GitHub Actions Workflow 通过 `workflow_dispatch` 执行。该 Workflow 不得承担 Build 职责，不重新构建 Artifact，只验证已经完成的 Draft Release 并执行 Publish。`Production Release Build` 与 `Publish Release` 必须是两个独立 Workflow，不得合并为一个可连续自动 Build-and-Publish 的 Workflow。

第二阶段至少必须输入 / 确认：

```text
Version: <semver>
Confirmation: PUBLISH
```

Publish Workflow 必须重新验证：

```text
Draft Release exists
Version matches requested SemVer
Source Revision matches Release Candidate
All required platform artifacts exist
release-manifest.json exists and matches artifacts
SHA256SUMS verifies successfully
Artifact Attestations verify successfully
Deployment Acceptance Gate = PASS
No required Release Asset is missing
```

任一检查失败：

```text
PUBLISH BLOCKED
```

不得降级为 warning，也不得通过隐藏 `--force`、跳过 Attestation、跳过 Checksum 或跳过 Acceptance 的方式发布。

第二阶段成功后：

```text
Draft Release
↓
Explicit Publish confirmation
↓
Revalidation
↓
Publish
↓
GitHub Immutable Production Release
↓
Eligible for normal install.sh / update.sh discovery
```

Release Build 与 Release Publish 必须是两个独立权限边界：

```text
Build confirmation != Publish confirmation
```

Codex、普通 CI、代码提交任务或自动化修复可以 commit / push，但不得因为代码任务完成而自动触发 Production Release Build 或最终 Publish。

如果 Draft Release 在最终 Publish 前发现问题，应修复代码并重新执行新的 Release Build；不得把已经验证过的旧 Source Revision 与新的 Artifact 混合。若目标 SemVer 已正式 Publish，则任何修复必须使用新的 SemVer Release。

## 11.6 Local Release Retention Contract

O3Pilot 本机 `app/releases/` 不得无限累积历史 Release，也不得为了节省空间破坏可验证回滚能力。

默认自动保留集合固定为：

```text
Active Release
+
latest 3 previously active Releases
```

其中 `previously active Release` 指曾经实际成为 `app/current` 指向目标并通过对应版本启动 / Readiness 验证的正式 Production Release；仅下载失败、解压失败、未激活 Staging 或未完成安装的版本不进入 Retention Count。

除上述默认集合外，以下 Release 进入 `Protected Release Set`，不得被自动清理：

```text
Any Release required by a retained VERIFIED PRE_UPGRADE_BACKUP
Any Release required by an in-progress Rollback
Any Release required by an in-progress Restore / Recovery operation
Active Release referenced by app/current
```

因此实际保留规则为：

```text
Retained Releases
=
Active Release
+ latest 3 previously active Releases
+ Protected Release Set
```

普通 Upgrade 完成后，旧 Release 清理只能在以下条件全部成立后执行：

```text
New Release switched active successfully
Post-upgrade Readiness = PASS
Upgrade state = SUCCEEDED
Retention / Protection references recalculated
Target old Release is outside Retained Releases
```

Cleanup 必须采用“先判定、后删除”的受控方式，不得通过目录年龄、文件修改时间或简单 `find ... -mtime` 规则删除 Release。

以下行为正式禁止：

```text
Delete app/current target
Delete a Release required by retained PRE_UPGRADE_BACKUP
Delete previous Release before upgrade validation succeeds
Delete Release during active Rollback / Restore
Auto-delete protected Releases merely because disk is low
Treat Release directory as Backup data and upload it to R2 by default
```

磁盘空间不足时，可以优先清理已经满足 Cleanup Eligibility 的非保护历史 Release；如果清理后仍不足，则必须按 Disk Capacity Contract 阻断高风险操作，而不是突破 Release Protection Boundary。

Local Release Retention 只管理本机可替换 Application Releases，不改变 Backup Retention，也不把 Application Release 纳入 Portable Instance Backup 的默认内容。跨主机 Restore 后所需 Application Release 应按目标 Restore / Compatibility Contract 从正式 Release Source 获取，不依赖复制原主机整个 `app/releases/`。

## 11.7 Release Installation Layout

正式 Release 安装到：

```text
macOS → /Library/O3Pilot/app/releases/<version>/
Linux → /opt/O3Pilot/app/releases/<version>/
```

当前版本通过：

```text
macOS → /Library/O3Pilot/app/current
Linux → /opt/O3Pilot/app/current
```

指向 Active Release。

Application Upgrade 不得覆盖：

```text
data/
backups/
config/
secrets/
state/
logs/
```

`O3PILOT_ROOT/bin/o3pilotctl` 是稳定本机 Operator Entry Point，推荐实现为指向 Active Release 中 `o3pilotctl` 的受控 symlink / launcher；它属于可替换 Application Support，不属于 Persistent Business Data。

## 11.8 Temporary Transfer Artifact Lifecycle

`.tar.gz` 只允许作为 Temporary Transfer Artifact，永远不属于长期持久化状态。

正式生命周期：

```text
Download .tar.gz to Staging
↓
Verify Artifact Digest / Manifest
↓
Extract to new Release Staging Directory
↓
Verify extracted Release
↓
Install / move to app/releases/<version>/
↓
Delete downloaded .tar.gz
↓
Delete temporary extraction staging
↓
Switch app/current when appropriate
```

失败时必须 best-effort 清理：

```text
partial download
.tar.gz
incomplete extraction directory
```

但 Cleanup Failure 永远不得导致删除已有有效旧 Release。

Artifact Integrity 校验失败必须 Fail Closed。

---

# 12. Unified Official Installation Contract

## 12.1 唯一正式安装入口

O3Pilot v1 在 macOS 与 Linux 上统一采用官方 Shell Installer。

唯一正式安装命令固定为：

```bash
curl -fsSL https://o3pilot.app/install.sh | sudo bash
```

该命令不是示例，而是 O3Pilot v1 的正式用户安装入口。

v1 不把以下方式作为正式安装方式：

```text
.pkg
.dmg
Homebrew
.deb
.rpm
Snap
Flatpak
Docker
git clone + source install
manual pip / npm build
```

Fresh Install 与安全卸载后的 Retained Instance Reinstall 以 `install.sh` 为基线；Existing Instance Upgrade 以 `update.sh` 为基线。`install.sh` 不得作为运行中实例的升级机制。

## 12.2 Installer Host Requirements

正式 Host 只需要：

```text
Supported OS / Architecture
sudo / administrator capability
working HTTPS connectivity
system-provided Bash
curl
tar + gzip
standard Unix utilities
platform-native SHA-256 utility
launchd on macOS
systemd on Linux
```

允许依赖的基础命令包括：

```text
bash
curl
tar
gzip
uname
grep
sed
awk
cut
printf
mkdir
rm
mv
cp
ln
chmod
chown
id
mktemp
date
```

Checksum：

```text
macOS → shasum -a 256
Linux → sha256sum
```

Linux 可以使用系统提供的账户管理能力，例如 `useradd`；macOS 可以使用系统提供的 `launchctl` 与账户管理能力。

正式安装明确不要求：

```text
Python
pip
Node.js
npm
Git
GitHub CLI
jq
Homebrew
GNU coreutils
compiler / build toolchain
Xcode Command Line Tools
Rust / Go
Docker
```

Installer / Updater / Uninstaller 必须兼容受支持系统自带 Bash，不得要求 Homebrew Bash、Bash 4/5-only feature 或 zsh-specific syntax。

O3Pilot 不自动安装 Homebrew、Git、Python、Node.js、jq、systemd 等系统 / 开发依赖。缺少正式 Host Requirement 时必须清晰失败，而不是静默修改主机环境。

HTTPS / CA Validation 必须正常。正式脚本禁止：

```text
curl -k
--insecure
skip TLS certificate verification
```

TLS 或 CA 验证失败必须 Fail Closed。

## 12.3 Installer 权限边界

`install.sh` 通过 `sudo bash` 获得安装阶段所需的管理员权限，用于：

```text
Create O3PILOT_ROOT
Create dedicated Runtime Identity
Set filesystem ownership and permissions
Install immutable Release Artifact
Register launchd / systemd Service Definition
Enable and start the service
```

管理员权限只属于 Installer。

安装完成后：

```text
O3Pilot Runtime != root
```

O3Pilot Runtime 必须以专用非 root OS Identity 运行。

## 12.4 Platform Detection

同一个 `install.sh` 必须自动识别 OS、OS Version、CPU Architecture 与 Linux Distribution，并选择对应 Release Artifact。

正式识别矩阵：

```text
Darwin + arm64 + macOS >= 26
    → macOS Apple Silicon
    → O3PILOT_ROOT=/Library/O3Pilot
    → launchd

Ubuntu 24.04 LTS + x86_64
Ubuntu 24.04 LTS + arm64
Ubuntu 26.04 LTS + x86_64
Ubuntu 26.04 LTS + arm64
Debian 13 + x86_64
Debian 13 + arm64
    → O3PILOT_ROOT=/opt/O3Pilot
    → systemd
```

以下情况必须直接 Fail Closed：

```text
macOS < 26
Intel Mac / macOS x86_64
Unsupported Linux Distribution / Version
Unsupported Linux Architecture
Non-systemd Linux
Unsupported OS
Unsupported Release / Platform Combination
```

## 12.5 Release Selection 与完整性校验

Installer 默认安装 `LI5ee3/O3Pilot` GitHub Releases 中与当前平台兼容的最新 Stable 正式 Release。

Installer 至少必须确认：

```text
release version
operating system
cpu architecture
artifact identity
artifact checksum
release manifest
compatibility manifest
```

正常 Bootstrap 不依赖 `jq` 或 GitHub CLI。可以通过 HTTPS 与 GitHub Stable Release redirect / asset naming contract 解析目标版本和 Artifact。

Release Artifact 下载完成后必须执行 SHA-256 校验；校验失败必须删除无效 Temporary Transfer Artifact 并停止安装。

不得提供：

```text
--skip-checksum
arbitrary --repo
arbitrary artifact download URL
hidden release-source override
```

`install.sh` 只负责 Bootstrap 与受控安装，不得在目标机器上执行 Production Source Build。

## 12.6 macOS 安装流程

当检测到受支持 macOS 时：

```text
Verify macOS >= 26 / Apple Silicon
↓
Verify Host Requirements
↓
Check CPU / RAM / Disk / Port / Existing Instance
↓
Create minimal /Library/O3Pilot/state/staging
↓
Resolve Stable GitHub Release
↓
Download macOS arm64 .tar.gz to Staging
↓
Verify SHA-256 / Manifest / Artifact identity
↓
Extract + verify new Release staging
↓
Create complete directory structure + permissions
↓
Create dedicated non-root Runtime Identity
↓
Install Release into /Library/O3Pilot/app/releases/<version>/
↓
Create / refresh stable o3pilotctl launcher
↓
Delete .tar.gz + temporary extraction staging
↓
Set /Library/O3Pilot/app/current
↓
Register /Library/LaunchDaemons/com.o3pilot.app.plist
↓
Start O3Pilot through launchd
↓
Verify 127.0.0.1:38652
↓
Run Readiness Check
↓
Output trusted local initialization URL
```

## 12.7 Linux 安装流程

当检测到受支持 Linux 时：

```text
Verify supported Distribution / Version / Architecture / systemd
↓
Verify Host Requirements
↓
Check CPU / RAM / Disk / Port / Existing Instance
↓
Create minimal /opt/O3Pilot/state/staging
↓
Resolve Stable GitHub Release
↓
Download matching Linux .tar.gz to Staging
↓
Verify SHA-256 / Manifest / Artifact identity
↓
Extract + verify new Release staging
↓
Create complete directory structure + permissions
↓
Create dedicated non-root Runtime Identity
↓
Install Release into /opt/O3Pilot/app/releases/<version>/
↓
Create / refresh stable o3pilotctl launcher
↓
Delete .tar.gz + temporary extraction staging
↓
Set /opt/O3Pilot/app/current
↓
Register /etc/systemd/system/o3pilot.service
↓
systemctl daemon-reload
↓
Enable and start O3Pilot through systemd
↓
Verify 127.0.0.1:38652
↓
Run Readiness Check
↓
Output Local / Headless initialization instructions
```

Linux v1 不要求最终用户选择 `.deb`、`.rpm` 或发行版专属安装包。

## 12.8 最终用户不承担 Build 责任

最终用户不应该为了安装正式 Release 而手动：

```text
git clone
install Python dependencies
create .venv
install Node.js
npm install
npm build
run uvicorn manually
```

这些属于 Build / Release 阶段责任，而不是最终用户安装责任。

## 12.9 Installer 失败原则

Installer 在任何关键步骤失败时必须：

- 明确输出失败阶段；
- 不把不完整 Release 标记为 Active；
- 不破坏已有可运行实例；
- 不删除已有 Persistent Data；
- best-effort 删除 partial download、`.tar.gz` 和 incomplete extraction staging；
- Cleanup Failure 不得删除已有有效 Release；
- 不静默更换端口、目录或 Release Channel；
- 对完整性、兼容性、权限、资源不足或平台识别异常采取 Fail Closed。

---

# 13. Fresh Installation Flow

正式 Fresh Install：

```text
1. Verify supported OS / Architecture / Host Requirements
2. Run CPU / RAM / Disk / Port Preflight
3. Create minimal O3PILOT_ROOT staging area
4. Resolve and verify Production Release Artifact
5. Create Runtime Identity
6. Create complete filesystem structure
7. Apply file permissions
8. Install immutable Release
9. Delete Temporary Transfer Artifact / extraction staging
10. Initialize empty Data Root
11. Generate Instance Security Material
12. Register OS Service
13. Start O3Pilot on Loopback
14. Run Local Readiness Check
15. Local-only Owner Initialization
16. Generate Recovery Key
17. Require Recovery Key acknowledgement
18. Complete Owner Login
19. Configure Ozon Integrations
20. Configure Cloudflare Tunnel if remote access is needed
21. Configure R2 if off-device replica is desired
22. Create and verify first NORMAL_BACKUP
23. Mark Deployment READY
```

任何中间失败必须能够解释失败阶段、已经产生的状态以及下一步恢复方式。

---

# 14. Local-only Owner Initialization

首次 Owner 创建只能通过本机受信任初始化流程执行。

未初始化实例：

```text
UNINITIALIZED
```

不得提供 Internet 上任意人都可以抢占管理员的 Public Setup 页面。

初始化至少建立：

```text
Instance ID
Owner Credential
Instance Master Key
Backup Repository Identity
Backup Repository Key
Recovery Key
Config Version
```

Recovery Key 必须保存在 O3Pilot 主机之外。

## 14.1 Local Desktop Initialization

本机有浏览器时，可以由安装程序生成短生命周期、一次性、本机限定的初始化能力，例如：

```text
http://127.0.0.1:38652/initialize?<one-time-token>
```

具体 Token 格式不是 Deployment Contract；实现必须遵守 `SECURITY.md`。

## 14.2 Linux Headless Initialization

无桌面 Linux 服务器正式使用 SSH Local Port Forwarding，不为首次初始化开放 LAN / Public Application Port。

服务器端 O3Pilot 始终保持：

```text
127.0.0.1:38652
```

用户在自己的本地电脑执行：

```bash
ssh -N -L 38652:127.0.0.1:38652 <user>@<server>
```

然后本地浏览器访问：

```text
http://127.0.0.1:38652
```

本地端口冲突时只改变 SSH local-side port，例如：

```bash
ssh -N -L 38653:127.0.0.1:38652 <user>@<server>
```

浏览器改为：

```text
http://127.0.0.1:38653
```

服务器 O3Pilot Port 不因此改变。

Linux Installer 在实例尚未初始化时必须输出 Headless SSH Port Forwarding 模板。

## 14.3 Public Initialization Forbidden

Owner Initialization 不得通过 Cloudflare Tunnel 作为正式初始化通道。

顺序固定为：

```text
Install
↓
Start O3Pilot on Loopback
↓
Local browser / SSH Port Forward
↓
Create Owner
↓
Complete Recovery Key setup
↓
Instance INITIALIZED
↓
Configure / enable Cloudflare Tunnel
```

初始化请求必须验证 Local Context。正式实现应只接受 `localhost` / Loopback Host Context，并拒绝明显经公网反向代理进入的 Owner Bootstrap Request，例如带有 Cloudflare / Forwarded Client IP Header 的请求。

## 14.4 One-time Bootstrap

Owner 创建并完成 Recovery Key 初始化后：

```text
UNINITIALIZED
→ INITIALIZED
```

首次 Owner Bootstrap Endpoint 必须永久失效。

以下行为不得重新打开 Owner Claim：

```text
restart
upgrade
application reinstall over retained instance
```

只有真正 Fresh Instance 才允许首次 Owner Initialization。

---

# 15. OS Service Model

正式平台：

```text
macOS → launchd
Linux → systemd
```

Service Manager 必须负责：

- 开机自动启动；
- 异常退出后的安全重启；
- 固定 Runtime Identity；
- 正确 Working Directory / Config；
- Loopback Binding；
- 正常 Shutdown Signal；
- 日志接入；
- Crash-loop protection；
- 不在 Service Definition 中写明文业务 Secret。

macOS Production 不依赖用户手动 Terminal 长期开着运行进程。

## 15.1 Service Recovery Policy

普通 Unexpected Process Failure 可以自动恢复：

```text
Unexpected Crash
↓
Wait 10 seconds
↓
Restart same Release
```

默认 Recovery 参数：

```text
Restart Delay = 10 seconds
Crash-loop threshold = 5 unexpected failures within 5 minutes
Graceful Shutdown Timeout = 60 seconds
```

达到 Crash-loop threshold 后：

```text
CRASH_LOOP
→ suspend automatic restart
→ require operator intervention
```

Planned Stop 不得被解释为 Crash：

```text
update.sh
uninstall.sh
operator explicit stop
OS shutdown / reboot
controlled maintenance
```

Updater / Operator 必须能够受控停止服务，而不是被 Service Manager 立即拉回旧 Runtime。

## 15.2 Failure Classification

可以自动重启：

```text
Unexpected process crash
Unhandled fatal process exit
One-time OOM termination
Unexpected SIGKILL
```

必须 Fail Closed、不得 hot-loop：

```text
SQLite integrity failure
Unsupported schema / persistent format
Secret store cannot decrypt
Critical configuration corruption
Missing active Release
Artifact integrity failure
Instance identity conflict
Persistent directory permission violation
```

Service Recovery 只恢复进程，不执行：

```text
Automatic Backup Restore
Automatic Release Rollback
Automatic Database Rebuild
Automatic SQLite replacement
Automatic Raw deletion
```

Ozon API、Performance API、Exchange Rate API、R2、DNS、Internet 或 Cloudflare Tunnel 暂时不可用不触发 O3Pilot Service Restart。

单个 Background Job Failure 应进入 Job FAILED / RETRY 状态，不应通过崩溃整个 O3Pilot Runtime 来恢复。

## 15.3 Platform Behavior Mapping

Linux `systemd` 语义至少等价于：

```text
Restart on unexpected failure
Restart delay 10 seconds
5 failures / 5 minutes crash-loop protection
60 seconds graceful stop timeout
Boot auto-start
Dedicated non-root Runtime User
```

macOS `launchd` 必须实现等价行为合同，不要求配置字段与 systemd 一致。

至少持久化或可诊断：

```text
last_start_time
last_exit_time
last_exit_reason
last_exit_code
restart_count
crash_loop_state
active_release
```

不得在这些诊断状态中记录 Secret。

---

# 16. Single Primary Instance

同一个 Data Directory 同一时刻只允许一个 O3Pilot Primary Process。

启动必须获得：

```text
OS-level Exclusive Instance Lock
```

第二实例检测到有效 Lock 后必须：

```text
REFUSE START
```

不得依赖 SQLite 最后发生写冲突才发现重复实例。

---

# 17. Startup & Shutdown

正式启动顺序：

```text
Resolve Paths
↓
Acquire Instance Lock
↓
Load Config / Secrets
↓
Open SQLite
↓
Verify DB Compatibility / Integrity
↓
Resolve Migration State
↓
Verify Raw Store / Disk
↓
Recover Interrupted Jobs
↓
Start API / Worker / Scheduler
↓
READY
```

关键完整性或兼容性失败：

```text
RECOVERY_REQUIRED
```

正常停止：

```text
Stop accepting new requests that create new work
↓
Stop dispatching new scheduled work
↓
Stop claiming new Jobs
↓
Allow active work to finish or reach safe checkpoint
↓
Reach safe SQLite transaction boundary
↓
Commit or safely rollback transactions
↓
Persist recovery state
↓
Flush logs / state
↓
Close SQLite
↓
Release Instance Lock
↓
STOPPED
```

Graceful Shutdown 默认最大等待：

```text
60 seconds
```

超过超时后允许 Service Manager 强制终止，但必须记录：

```text
FORCED_SHUTDOWN
```

正式升级不得默认以 `kill -9` 替代正常停止流程。

---

# 18. Cloudflare Tunnel

Cloudflare Tunnel 是 O3Pilot v1 正式远程访问方式：

```text
Public Hostname
↓
Cloudflare
↓
Cloudflare Tunnel
↓
http://127.0.0.1:<o3pilot-port>
```

## 18.1 Lifecycle Ownership

正式定义：

```text
O3Pilot != cloudflared owner
```

`cloudflared` 是 Cloudflare 提供、独立安装、独立升级、独立配置、独立卸载的外部系统服务。

因此：

```text
install.sh
update.sh
uninstall.sh
```

均不得直接管理 `cloudflared` 生命周期。

缺少 `cloudflared` 不得导致 O3Pilot Fresh Install 或 Local Runtime 失败。

```text
Local Access = Core Capability
Cloudflare Tunnel = Remote Access Capability
```

## 18.2 Recommended Tunnel Model

O3Pilot v1 推荐：

```text
Cloudflare Remotely-managed Tunnel
```

Origin 固定指向：

```text
http://127.0.0.1:38652
```

O3Pilot 不负责生成或长期维护 Cloudflare Tunnel 配置文件，也不负责 Cloudflare Dashboard / DNS / Access Policy 的生命周期。

## 18.3 Credential Boundary

Tunnel Token / Connector Credential 由 cloudflared / Cloudflare 管理。

O3Pilot：

```text
不读取
不复制
不备份
不迁移
不上传 R2
不显示
不轮换
```

Cloudflare Tunnel Credential。

因此 Portable O3Pilot Backup 不包含 cloudflared Credential；Cross-host Restore 后如需远程访问，必须在目标主机重新建立 cloudflared Connector。

## 18.4 Upgrade / Uninstall Boundary

O3Pilot Upgrade 不得：

```text
upgrade cloudflared
restart cloudflared
modify Tunnel
modify DNS
modify Cloudflare configuration
```

即使 `uninstall.sh --purge` 也不得：

```text
uninstall cloudflared
delete cloudflared service
delete Tunnel
delete DNS
delete Cloudflare Connector
delete Cloudflare Access Policy
```

因为 cloudflared 可能同时服务于主机上的其他应用。

## 18.5 Failure Isolation

Tunnel 不可用时：

```text
Cloudflare Tunnel Down
!=
O3Pilot Down
```

以下情况都只能影响 Remote Access：

```text
cloudflared stopped
Cloudflare outage
Tunnel disconnected
DNS failure
```

本地 O3Pilot、Scheduler、Worker 与 Local Backup 必须继续正常工作。

O3Pilot 可以观察并显示 cloudflared / Tunnel Health，但不得因状态检测而静默安装或修改 Cloudflare 资源。

Cloudflare Access 可以作为额外防护，但不能替代 O3Pilot 应用自己的 Authentication / Session / CSRF。

---

# 19. Cloudflare R2

Cloudflare R2 是：

```text
Optional Off-device Backup Replica
```

不是：

```text
Primary Database
Transactional Database
SQLite Filesystem
Primary Raw Store
Mandatory Runtime Dependency
```

未配置 R2 时，O3Pilot 必须完整支持：

```text
Normal Operation
Local Backup
Portable Export
Local Restore
Cross-host Restore using transferred backup
```

R2 Bucket 应保持 Private。

Backup 在上传 R2 前必须由 O3Pilot Backup Encryption 保护。

正式关系：

```text
Local Backup Success
!=
R2 Replica Success
```

R2 故障不得破坏已经成功的 Local Backup。

---

# 20. Backup Classes

O3Pilot 至少区分：

```text
NORMAL_BACKUP
PRE_UPGRADE_BACKUP
PORTABLE_MIGRATION_BACKUP
```

一个可恢复的 Instance Backup 至少关联：

```text
Consistent SQLite Snapshot
Required Local Raw Objects
Portable Config
Portable Encrypted Secret State
Backup Manifest
Compatibility Metadata
Integrity Metadata
```

Backup 只有在创建并完成完整性验证后才能标记为：

```text
VERIFIED
```

默认正式安装应启用至少每日一次 Local `NORMAL_BACKUP`。

## 20.1 NORMAL_BACKUP Default Retention

默认分层保留：

```text
Daily    → 14 days
Weekly   → 8 weeks
Monthly  → 12 months
```

Retention 的目标是保留合理的 Restore Point Density，不要求保存每一天的完整独立对象副本。

## 20.2 PRE_UPGRADE_BACKUP Retention

默认保留：

```text
latest 3 VERIFIED PRE_UPGRADE_BACKUP restore points
```

并且当前 Active Release 所需的最近一次有效 PRE_UPGRADE_BACKUP 不得因普通 Retention 自动删除。

## 20.3 PORTABLE_MIGRATION_BACKUP Retention

```text
Automatic Retention = NONE
```

O3Pilot 不自动删除用户主动创建的 Portable Migration Backup；只有显式用户删除或未来单独定义的正式归档策略才能移除。

## 20.4 Retention Safety

Backup Retention 删除的是 Restore Point / Manifest 可见性，不是按目录年龄暴力删除文件。

正式清理流程：

```text
Retire Backup Manifest
↓
Calculate Object Reachability
↓
Only unreferenced objects become GC candidates
```

只要 Object 仍被任何有效 Backup Manifest 引用，就不得删除。

---

# 21. Backup Repository

Local Backup Repository 固定在各平台的 `O3PILOT_ROOT` 内：

```text
macOS → /Library/O3Pilot/backups/repository/
Linux → /opt/O3Pilot/backups/repository/
```

Local Backup 与 Primary Data 位于同一主机时，不能防御整机或整盘丢失。

因此长期正式实例建议额外采用：

```text
Cloudflare R2
or
Periodic Portable Backup to separate physical storage
```

R2 仍属于 Optional Capability。

Backup Retention 必须根据 Manifest 与 Object Reachability 执行，不得简单按文件年龄删除仍被有效 Manifest 引用的对象。

如果启用 R2：

```text
Local Backup Manifest
= Retention Source of Truth

R2
= encrypted replica of retained repository state
```

v1 不再建立独立于 Local Repository 的第二套 R2 Retention Policy。R2 Object GC 可以晚于 Local Manifest Retirement 执行，但不得删除仍被有效 Manifest 引用的对象。

---

# 22. PRE_UPGRADE_BACKUP

任何可能改变持久化格式的升级前必须成功创建并验证：

```text
PRE_UPGRADE_BACKUP
```

覆盖可能变化的：

```text
Database Schema
Config Schema
Raw Store Format
Secret Format
Backup Format
other persistent state
```

Backup 创建或验证失败：

```text
ABORT UPGRADE
```

---

# 23. Official Update Contract

## 23.1 唯一正式更新入口

O3Pilot v1 在 macOS 与 Linux 上统一采用官方 Shell Updater。

唯一正式手动更新命令固定为：

```bash
curl -fsSL https://o3pilot.app/update.sh | sudo bash
```

生产升级明确不使用：

```text
git pull
git checkout
source tree overwrite
manual pip / npm update
re-run install.sh as an upgrade mechanism
```

`install.sh` 与 `update.sh` 职责必须分离：

```text
install.sh
    → Fresh Install
    → Retained Instance Reinstall after APPLICATION_UNINSTALL
    → never upgrade an active installed instance

update.sh
    → Existing active Instance Upgrade only
```

`update.sh` 找不到有效现有 O3Pilot Instance 时必须停止并提示使用 `install.sh`，不得静默转为 Fresh Install。

O3Pilot v1 默认不执行无人值守自动升级。升级必须由用户显式触发。

## 23.2 Updater 权限边界

`update.sh` 通过 `sudo bash` 获得升级阶段所需的管理员权限，用于下载和安装新 Release、维护 `O3PILOT_ROOT/app/`、执行受控 Migration、切换 Active Release，以及重启 `launchd` / `systemd` 服务。

管理员权限只属于 Updater。

升级完成后仍必须满足：

```text
O3Pilot Runtime != root
```

## 23.3 Upgrade Compatibility Window

同一个 Major Version 内：

```text
Any older Stable → latest Stable directly
```

O3Pilot v1 做出以下兼容保证：

```text
All v1 Persistent Formats remain directly upgradable
minimum_supported_persistent_format = 1
Skipped Application Versions = Supported
Intermediate Application Releases = Not Required
```

例如：

```text
1.0.0 → 1.8.0  Supported
1.2.3 → 1.8.0  Supported
1.7.1 → 1.8.0  Supported
```

Migration Runner 必须根据 Persistent Format 顺序执行所有缺失 Migration，例如：

```text
Current Persistent Format = 2
Target Persistent Format = 6

2 → 3 → 4 → 5 → 6
```

Application Version 与 Persistent Format 必须分离管理；不是每个 Application Release 都必须改变 Persistent Format。

每个 Target Release Manifest 必须明确声明：

```text
persistent_format
minimum_supported_persistent_format
```

Cross-major Upgrade 只有在目标 Major Release 明确声明兼容时才允许，不自动继承 v1 的全窗口保证。

Downgrade 不属于 `update.sh` 能力：

```text
Target Version < Current Version
→ ABORT
```

相同版本：

```text
Installed = Latest Stable
→ NO UPDATE AVAILABLE
→ EXIT SUCCESSFULLY
```

Draft / Pre-release 不进入默认 Stable Update Path。

## 23.4 Version Check 与 Release Selection

Updater 必须先读取：

```text
current application version
current persistent format
current platform / architecture
```

再读取 `LI5ee3/O3Pilot` 最新 Stable Release 的 Manifest。

兼容性检查必须在停止服务、创建 PRE_UPGRADE_BACKUP 或修改 Instance 之前完成。

如果 Target Manifest 不支持当前 Persistent Format：

```text
UPGRADE NOT COMPATIBLE
→ FAIL BEFORE MODIFYING INSTANCE
```

如果当前版本已经是最新兼容 Stable Release：

```text
NO UPDATE AVAILABLE
→ EXIT SUCCESSFULLY
```

发现新版本后至少确认：

```text
current application version
target application version
operating system
cpu architecture
artifact identity
artifact checksum
compatibility manifest
persistent format compatibility
```

## 23.5 Formal Upgrade Pipeline

正式更新流程：

```text
Validate Existing Instance
↓
Read Current Version + Persistent Format
↓
Resolve Latest Stable Release Manifest
↓
Compatibility Preflight
↓
No New Version → Exit Successfully
↓
Capacity + SQLite / Persistent State Integrity Preflight
↓
Download .tar.gz to Staging
↓
Verify SHA-256 / Manifest / Artifact identity
↓
Extract + verify New Release Staging
↓
Quiesce Scheduler / Worker
↓
Acquire Exclusive Instance Lock
↓
Create + Verify PRE_UPGRADE_BACKUP
↓
Install Release into app/releases/<new-version>/
↓
Run all required Versioned Migrations
↓
Verify migrated persistent state
↓
Delete .tar.gz + temporary extraction staging
↓
Switch app/current → new Release
↓
Restart O3Pilot through launchd / systemd
↓
Readiness + Post-upgrade Validation
↓
Resume Scheduler / Worker
↓
Mark Upgrade SUCCEEDED
```

Application Release 安装和切换不得覆盖：

```text
data/
backups/
config/
secrets/
state/
logs/
```

Migration 必须 Versioned、Auditable、Crash-aware，并且 Idempotent 或具有明确 Resume Strategy。

## 23.6 Platform Mapping

同一个 `update.sh` 必须自动识别现有实例的平台布局：

```text
macOS 26+ Apple Silicon
    → O3PILOT_ROOT=/Library/O3Pilot
    → Release Root=/Library/O3Pilot/app/
    → Service=launchd

Supported Ubuntu / Debian x86_64 / arm64
    → O3PILOT_ROOT=/opt/O3Pilot
    → Release Root=/opt/O3Pilot/app/
    → Service=systemd
```

Updater 不得把一个平台的 Artifact 安装到另一平台，也不得跨 Architecture 复用 Release。

## 23.7 Upgrade Failure Boundary

任何升级阶段失败都必须留下明确的失败状态，并禁止在未确认 Persistent State 兼容性的情况下继续启动旧或新 Release。

如果 Failure 发生在 Persistent Migration 之前，可保持或切回 Previous Application Release。

如果 Migration 已经改变旧 Release 无法理解的 Persistent Format，则必须进入第 24 章定义的 Backup-based Rollback。

不得因 Crash Loop 自动执行 Release Rollback。

---

# 24. Rollback

如果持久化 Migration 尚未发生，新 Release 启动失败，可以直接切回旧 Application Release。

如果 Migration 已经改变旧 Release 无法理解的持久化结构：

```text
Stop New Release
↓
Restore PRE_UPGRADE_BACKUP
↓
Switch current → previous Release
↓
Start Previous Release
↓
Verify
```

旧 Application 不得打开高于自身支持范围的新数据库或 Config Format 并尝试继续运行。

O3Pilot 不以每个 Migration 都实现通用 Reverse Migration 作为回滚基础。

---

# 25. Restore

Restore 原则：

```text
Fail Closed
Restore to Staging
Verify Before Promotion
Never Partially Restore Into Live Instance
Compatibility First
Integrity First
Old Sessions Invalid
```

Restore 不得边读取 Backup 边覆盖当前生产数据库。

正式 Local Restore：

```text
Stop O3Pilot
↓
Acquire Restore Lock
↓
Select + Verify Backup Manifest
↓
Verify Recovery Authorization
↓
Verify Application Compatibility
↓
Restore into isolated Staging
↓
Materialize DB / Raw / Config / Encrypted Secret State
↓
Validate SQLite / Object Integrity
↓
Reset Runtime-specific State
↓
Invalidate old Sessions
↓
Run supported Migration if required
↓
Atomic Promotion
↓
Start O3Pilot
↓
Readiness Check
```

错误 Recovery Key、对象篡改、缺失 Object、Manifest 不一致、SQLite Integrity 失败都必须 Fail Closed。

---

# 26. Cross-host Restore

Portable Backup 必须能够在正式支持的平台之间恢复：

```text
Apple Silicon Mac → Apple Silicon Mac
Apple Silicon Mac → supported Linux
supported Linux → Apple Silicon Mac
supported Linux → supported Linux
```

不承诺 Intel Mac 相关迁移，因为 Intel Mac 不属于 O3Pilot v1 正式支持平台。

Portable Backup 不得依赖：

- 原主机 Application 绝对路径；
- Python virtualenv 绝对路径；
- UID / PID；
- launchd / systemd Runtime State；
- cloudflared Tunnel Credential / Connector Runtime State；
- 临时目录；
- 浏览器 Session。

Persistent Business Identity 可以保留，Runtime Identity 必须在目标主机重新建立。

Cross-host Restore 后如需要 Cloudflare Remote Access，必须在目标主机单独重新建立 cloudflared Connector。

---

# 27. Logs & Temporary Files

所有 O3Pilot 自有日志固定进入各平台 `O3PILOT_ROOT/logs/`：

```text
macOS → /Library/O3Pilot/logs/
Linux → /opt/O3Pilot/logs/
```

不得额外写入 `/Library/Logs/O3Pilot`、`/var/log/o3pilot` 等位置形成第二套持久化日志目录。

所有 O3Pilot Temporary / Staging 文件必须位于各平台 `O3PILOT_ROOT/state/`：

```text
macOS → /Library/O3Pilot/state/tmp/
macOS → /Library/O3Pilot/state/staging/
Linux → /opt/O3Pilot/state/tmp/
Linux → /opt/O3Pilot/state/staging/
```

不得在 `/tmp` 或用户 Home 长期遗留 O3Pilot 文件。

## 27.1 Runtime Log Rotation

默认 Log Contract：

```text
Retention: 30 days
Maximum total O3Pilot log storage: 1 GiB
Rotate: daily OR 20 MiB per active log file
```

Age Limit 或 Total Size Limit 任一达到即可从最旧 Rotated Log 开始清理；旧日志允许压缩。

不得出现无限增长的单个 Active Log。

ERROR / CRITICAL Runtime Logs 同样受上述 Retention / Size Cap 约束；真正需要长期保留的业务 / 运维事实必须进入结构化持久层，而不是依赖日志文件永久存在。

## 27.2 Structured History Boundary

以下结构化历史如果存储在 SQLite：

```text
Job History
Backup Manifest
Import History
Alert History
Sync Status
Migration History
Security / Operation Events
```

不属于 Runtime Log，不受 `30 days / 1 GiB` Log Retention 直接删除。

## 27.3 Secret Redaction

Logs 不得记录：

```text
API Key
Client Secret
Access Token
Session Token
Recovery Key
R2 Secret
Instance Master Key
Backup Repository Key
complete Authorization Header
cloudflared Tunnel Token
```

---

# 28. Disk & SQLite Operations

持续观察：

```text
Free Disk Space
SQLite Size
Raw Store Size
Backup Repository Size
Log Size
Temporary Usage
```

## 28.1 Minimum Resource Baseline

正式最低资源：

```text
Minimum CPU: 2 logical cores
Recommended CPU: 4+ logical cores

Minimum RAM: 2 GiB
Recommended RAM: 4 GiB+
Large datasets / frequent imports: 8 GiB+ recommended

Fresh Install Minimum Free Disk: 5 GiB
```

低于 `2 logical cores`、`2 GiB RAM` 或 Fresh Install `5 GiB` 可用磁盘时不得作为正式受支持安装继续。

## 28.2 Dynamic Capacity Preflight

Capacity Check 不是 Installer 一次性检查，至少必须发生在：

```text
Fresh Install
Startup
Backup
Upgrade
Restore
Large Import
```

Upgrade 必须动态估算：

```text
Artifact
+ Extracted Release
+ PRE_UPGRADE_BACKUP requirement
+ Migration workspace
+ Safety reserve
```

Backup / Restore / Large Import 也必须在执行前估算自身需要的可用空间。

空间不足必须在高风险写入开始前失败，不能把磁盘写满后才发现。

## 28.3 Runtime Disk Thresholds

默认：

```text
DISK_SPACE_WARNING
    → Free Disk < 10 GiB OR < 10%

DISK_SPACE_CRITICAL
    → Free Disk < 2 GiB
```

`DISK_SPACE_CRITICAL` 时应阻止或暂停高增长 / 高风险操作，例如：

```text
Manual File Import
Large historical backfill
New backup creation
Nonessential bulk Raw ingestion
Upgrade
```

但必须尽可能保持：

```text
UI
Authentication
Diagnostics
Existing data reads
Cleanup / maintenance operations
```

可用。

磁盘压力下不得自动删除：

```text
SQLite business data
IRREPLACEABLE Raw
valid Backup restore points
Objects referenced by active Backup Manifest
```

自动删除 Raw / Backup / DB 不能作为“救磁盘”机制。

## 28.4 SQLite Safety

运行中的 SQLite 不得通过普通 `cp o3pilot.db backup.db` 被当作正式一致性 Backup。

正式 Backup 必须使用 SQLite 支持的一致性 Snapshot 机制。

生产数据库应优先通过正式 Migration / Maintenance / Recovery / Reprocess 能力操作，不以手工 SQL 修改作为正常运维方式。

---

# 29. Runtime Health

至少区分：

```text
Liveness
Readiness
External Integration Health
Backup Health
Tunnel Health
R2 Replica Health
Disk Health
Crash-loop State
```

Ozon、R2、Internet 或 Cloudflare 暂时不可用不自动意味着本地 O3Pilot `NOT_READY`。

Readiness 至少要求：

```text
Instance Lock valid
SQLite readable and compatible
Critical Migration complete
Required Config valid
Raw Store usable
No fatal integrity condition
```

Integration Health 与 Process Health 必须分离；外部系统失败不能通过不断重启 O3Pilot Runtime 来“恢复”。

---

# 30. Unified Official Uninstall Contract

## 30.1 唯一正式卸载入口

O3Pilot v1 在 macOS 与 Linux 上统一采用官方 Shell Uninstaller。

默认安全卸载命令固定为：

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash
```

该命令不是示例，而是 O3Pilot v1 的正式 `APPLICATION_UNINSTALL` 入口。

彻底删除整个本机 O3Pilot Instance 的正式命令固定为：

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash -s -- --purge
```

该命令对应：

```text
FULL_INSTANCE_PURGE
```

默认卸载与 Full Purge 必须是两个语义明确、不可混淆的操作。

`uninstall.sh` 不得把默认卸载静默提升为 `--purge`。

## 30.2 Uninstaller 权限边界

`uninstall.sh` 通过 `sudo bash` 获得卸载阶段所需的管理员权限，用于：

```text
Stop O3Pilot Service
Disable / Unregister launchd or systemd Service
Remove replaceable Application files when applicable
Remove Service Definition when applicable
Perform explicitly confirmed Full Purge
Remove dedicated Runtime Identity only during Full Purge
```

管理员权限只属于 Uninstaller。

Uninstaller 不得利用 root 权限扩大删除范围，不得删除未属于 O3Pilot 的本机文件，也不得主动修改任何 Ozon 服务端业务状态。

## 30.3 APPLICATION_UNINSTALL — 默认安全卸载

执行：

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash
```

默认只执行安全卸载。

流程：

```text
Validate Existing O3Pilot Installation
↓
Stop O3Pilot
↓
Disable / Unregister OS Service
↓
Remove O3Pilot Service Definition
↓
Remove replaceable Application Releases + O3PILOT_ROOT/bin launcher
↓
Preserve Persistent Instance State
↓
Mark Instance as APPLICATION_UNINSTALLED / RETAINED
↓
Exit Successfully
```

默认安全卸载必须保留：

```text
data/
backups/
config/
secrets/
必要的 instance / compatibility state
```

默认安全卸载不得删除：

```text
SQLite Primary Database
Local Raw Store
Local Backup Repository
Portable Backup
Runtime Integration Secrets
Recovery-related local metadata
Instance Identity
```

为了保留文件所有权和后续恢复能力，默认安全卸载也不得删除专用 O3Pilot Runtime Identity。

默认安全卸载后，实例不再运行，但长期数据仍然存在。

重新安装 Application 使用正式安装入口：

```bash
curl -fsSL https://o3pilot.app/install.sh | sudo bash
```

此时 `install.sh` 必须识别 `APPLICATION_UNINSTALLED / RETAINED` 状态，并执行 `RETAINED_INSTANCE_REINSTALL`：

```text
Reuse existing Instance Identity
Preserve data / backups / config / secrets
Install a compatible Stable Release
Restore O3PILOT_ROOT/bin/o3pilotctl launcher
Restore OS Service Definition
Start O3Pilot
Run Readiness + Compatibility Validation
```

该流程属于 Retained Instance Reinstall，不属于 Existing Instance Upgrade；运行中的已安装实例升级仍必须使用 `update.sh`。

## 30.4 FULL_INSTANCE_PURGE — 彻底删除

只有用户显式执行：

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash -s -- --purge
```

才允许进入 Full Purge。

Full Purge 属于不可逆的本机数据删除操作。

执行前必须：

```text
Detect Existing Instance
↓
Display exact deletion scope
↓
Recommend creating PORTABLE_MIGRATION_BACKUP
↓
Require explicit interactive destructive confirmation
↓
Only then continue
```

由于 `uninstall.sh` 通过 Pipe 执行，破坏性确认必须直接从 `/dev/tty` 或等效真实交互终端读取，不得把普通 STDIN Pipe 内容视为确认。

确认文本应使用明确、难以误触的固定短语，例如：

```text
DELETE O3PILOT
```

没有获得完全匹配的交互式确认时必须：

```text
ABORT PURGE
```

不得通过默认值、超时、`yes` Pipe 或非交互 STDIN 自动确认 Full Purge。

确认后可删除：

```text
macOS:
/Library/O3Pilot/
/Library/LaunchDaemons/com.o3pilot.app.plist
Dedicated O3Pilot Runtime Identity

Linux:
/opt/O3Pilot/
/etc/systemd/system/o3pilot.service
Dedicated O3Pilot Runtime Identity
```

Linux 删除 Service Definition 后必须执行必要的 `systemd daemon-reload`，使系统服务状态与文件状态一致。

Full Purge 必须在删除前停止 O3Pilot，并确保 Primary Instance Lock 已释放。

## 30.5 Full Purge 删除边界

`--purge` 只删除当前主机上属于 O3Pilot 本机实例的资源。

即使执行 Full Purge，O3Pilot 也不得自动：

- 删除 Cloudflare Tunnel；
- 删除 Cloudflare DNS Record；
- 删除 Cloudflare R2 Bucket；
- 删除已经上传到 R2 的 Backup Replica；
- 删除或修改 Ozon Webhook Subscription；
- 修改 Ozon 商品；
- 修改 Ozon 价格；
- 修改 Ozon 库存；
- 修改 Ozon 订单；
- 修改 Ozon 推广；
- 修改任何其他 Ozon 配置。

这些外部资源的生命周期独立于本机 O3Pilot Instance。

O3Pilot 永远不主动修改 Ozon。

## 30.6 卸载失败原则

Uninstaller 必须 Fail Closed。

如果无法明确判断目标路径、Instance Identity、Service Identity 或删除范围：

```text
ABORT UNINSTALL / PURGE
```

尤其对于 Full Purge：

- 不得使用模糊路径匹配；
- 不得递归删除未经验证的 Parent Directory；
- 不得跟随可能逃逸 `O3PILOT_ROOT` 的危险 Symlink 执行删除；
- 不得因为部分删除失败而继续扩大清理范围；
- 必须输出已经完成和未完成的删除项。

正式 Release Acceptance 必须分别测试：

```text
APPLICATION_UNINSTALL
RETAINED_INSTANCE_REINSTALL
FULL_INSTANCE_PURGE
macOS launchd cleanup
Linux systemd cleanup
Persistent data preservation
External resource non-deletion
```

---

# 31. Operator Command Contract

O3Pilot v1 提供统一的本机 Operator CLI：

```text
macOS → /Library/O3Pilot/bin/o3pilotctl
Linux → /opt/O3Pilot/bin/o3pilotctl
```

推荐由该稳定 Entry Point 受控指向 `app/current` 中同版本实现，从而保持 Single Application Root，同时随 Active Release 一致切换。

## 31.1 Lifecycle Boundary

```text
install.sh / update.sh / uninstall.sh
→ Application Lifecycle

o3pilotctl
→ Installed Instance Operations
```

因此 v1 不提供：

```text
o3pilotctl install
o3pilotctl update
o3pilotctl uninstall
```

## 31.2 Service Commands

```text
o3pilotctl status
o3pilotctl start
o3pilotctl stop
o3pilotctl restart
```

`status` 至少显示：

```text
Service State
PID
Active Release
Uptime
Listen Address / Port
Crash-loop State
```

## 31.3 Inspection Commands

```text
o3pilotctl health
o3pilotctl version
o3pilotctl logs
o3pilotctl logs --follow
o3pilotctl logs --since <duration>
o3pilotctl diagnostics
```

`health` 用于当前组件健康；`diagnostics` 用于深度环境 / 完整性检查。v1 不再额外增加语义重复的 `doctor` 命令。

`version` 至少输出：

```text
Application Version
Build ID
Source Revision
Persistent Format
Artifact Format
Platform / Architecture
```

`diagnostics` 至少检查：

```text
OS / Architecture
CPU / RAM
Disk
Application Version
Release Integrity
Service State
Port
SQLite integrity
Persistent Format
Directory permissions
Instance lock
Backup repository
Last backup
Crash-loop state
Log state
Ozon integration connectivity state
R2 state
```

并以 `PASS / WARN / FAIL` 等稳定状态表达结果，不得输出 Secret。

## 31.4 Backup Commands

```text
o3pilotctl backup create
o3pilotctl backup create --portable
o3pilotctl backup list
o3pilotctl backup verify <backup-id>
```

`backup create` 默认创建 `NORMAL_BACKUP`。

`backup create --portable` 创建 `PORTABLE_MIGRATION_BACKUP`。

正式 Backup Create 必须执行：

```text
Capacity Preflight
Consistent Snapshot
Integrity Verification
```

只有 VERIFIED 后才能成功返回。

`backup list` 至少显示：

```text
Backup ID
Type
Created At
Application Version
Persistent Format
Size
Verification State
R2 Replica State
```

`backup verify` 只验证，不 Restore、不修改当前 Instance。

## 31.5 Restore Commands

正式 Restore 入口：

```text
o3pilotctl restore <backup-id>
o3pilotctl restore --from <portable-backup>
```

Restore 必须：

```text
Capacity / Compatibility Preflight
↓
Verify Backup + Integrity
↓
Show Restore Target
↓
Require interactive confirmation from a real terminal
↓
Stop Service
↓
Acquire Exclusive Restore Lock
↓
Restore to Staging
↓
Verify
↓
Promote
↓
Start
↓
Readiness Check
```

高风险 Restore 应要求显式确认文本，例如：

```text
RESTORE O3PILOT
```

不得依赖普通 `yes | ...` 即可绕过确认。

禁止提供：

```text
--force-without-verification
--ignore-integrity
--ignore-compatibility
```

## 31.6 Offline Operation

除真正需要访问外部服务的 Health Check 外，下列 Core Operator Command 必须在 Internet / GitHub / Cloudflare / Ozon 不可用时仍能执行：

```text
status
start
stop
restart
version
backup
restore
logs
diagnostics
```

`o3pilotctl` 必须随本地安装存在，不得设计成在线下载脚本才能进行日常运维。

## 31.7 Privilege Boundary

正式文档可以统一使用：

```bash
sudo <O3PILOT_ROOT>/bin/o3pilotctl <command>
```

CLI 可以为 Service Manager、Backup、Restore、Permissions 等必要操作获得管理员权限，但不得因此把长期 O3Pilot Runtime 以 root 启动。

## 31.8 Dangerous Escape Hatches Forbidden

正式 CLI 不提供：

```text
arbitrary SQL shell
arbitrary application shell
raw-delete
force-schema-version
unsafe config file mutation
force unlock that bypasses active instance safety
integrity / compatibility bypass
```

如果未来需要特殊 Recovery Capability，必须单独定义正式 Recovery Contract。

---

# 32. Platform Acceptance Matrix

| Platform | Fresh Install | Start/Stop | Upgrade | Rollback | Backup | Restore | Tunnel |
|---|---:|---:|---:|---:|---:|---:|---:|
| macOS 26+ Apple Silicon / arm64 | Required | Required | Required | Required | Required | Required | Required |
| Ubuntu 24.04 LTS x86_64 | Required | Required | Required | Required | Required | Required | Required |
| Ubuntu 24.04 LTS arm64 | Required | Required | Required | Required | Required | Required | Required |
| Ubuntu 26.04 LTS x86_64 | Required | Required | Required | Required | Required | Required | Required |
| Ubuntu 26.04 LTS arm64 | Required | Required | Required | Required | Required | Required | Required |
| Debian 13 x86_64 | Required | Required | Required | Required | Required | Required | Required |
| Debian 13 arm64 | Required | Required | Required | Required | Required | Required | Required |

Beta macOS `>= 26` 可以安装，但不要求进入 Production Release Acceptance Matrix。

以下平台不进入 Acceptance Matrix：

```text
macOS < 26
macOS x86_64
Intel Mac
Rosetta 2 Production Deployment
Ubuntu <= 22.04
Debian <= 12
non-Ubuntu / Debian Linux
non-systemd Linux
Alpine / musl
WSL
32-bit Linux
```

---

# 33. Deployment Acceptance Gate

一个正式 Release 至少必须通过：

```text
Production Artifact Contract Validation
Canonical GitHub Release Source Validation
GitHub Release Immutability Validation
Artifact Attestation Validation
Release Trigger Contract Validation
Draft-to-Publish Revalidation
Temporary Transfer Artifact Cleanup
Local Release Retention Validation
Artifact Integrity
Compatibility Manifest Validation
Security Tests
Ozon Read-only Boundary Tests
Fresh Install
Upgrade
Rollback
Backup
Restore
Cross-host Restore
Second-instance Rejection
Crash Recovery
Crash-loop Protection
Disk Failure Paths
Cloudflare Tunnel Deployment
R2 Outage without Runtime Failure
Backup Plaintext Scan
Secret Plaintext Scan
Clean Supported OS Host Validation
No Undeclared Host Dependency
Minimum Resource Environment Validation
Oldest Supported Upgrade Path Validation
Operator CLI Contract Validation
```

其中 `Clean Supported OS Host Validation` 必须在未额外预装开发工具的干净受支持 OS 环境中执行。Installer、Updater 与 Uninstaller 不得因为开发者机器上碰巧存在 Homebrew、GNU coreutils、jq、Python、Node.js、Git、GitHub CLI 或其他未声明工具而产生隐式依赖。

`Minimum Resource Environment Validation` 必须至少在 `2 logical CPU cores + 2 GiB RAM` 的最低资源环境中完成 Fresh Install、Start、基础数据同步、Backup 与 Upgrade 验证。若某个 Production Release 无法在该最低环境稳定通过，则该 Release 不得继续宣称 `2 Core / 2 GiB RAM` 为最低资源基线，必须在发布前显式提高 Minimum Resource Requirement。

`Oldest Supported Upgrade Path Validation` 必须验证每个 Production Release 从该 Major Version 最早仍受支持的 Persistent State 直接升级到目标 Release 的完整 Migration Chain。对于 O3Pilot v1，至少必须覆盖最早 v1 Persistent Format → Target Release，以及 Latest Previous Stable → Target Release；任一链路失败都必须 Block Release。

`Operator CLI Contract Validation` 至少必须验证 Service、Health、Version、Backup、Restore、Logs、Diagnostics 的正式命令行为、Offline Availability 与 Privilege Boundary，不得出现依赖开发机工具或绕过 Integrity / Compatibility 的隐藏入口。

`GitHub Release Immutability Validation` 必须确认正式 Production Release 已启用并满足 GitHub Release Immutability；已经发布的 Tag 与 Release Assets 不得允许静默替换，同版本修复必须通过新的 SemVer Release 完成。

`Artifact Attestation Validation` 必须验证所有 Production Platform Artifacts 的 Attestation 与 `LI5ee3/O3Pilot`、目标 Source Revision、GitHub Actions Workflow Identity 和 Artifact Digest 一致；任一 Artifact 缺失、来源不符或验证失败都必须 Block Release。该 Acceptance Validation 不得转化为最终用户 Host 必须安装 `gh` CLI 的前置条件。

`Release Trigger Contract Validation` 必须确认普通 `push to main`、PR merge、Codex commit/push、普通 tag push 与 Normal CI Success 均不会自动产生 Production Release；正式 Release Build 只能由独立的 `Production Release Build` Workflow 通过 `workflow_dispatch` 显式人工触发，并且必须冻结目标 SemVer 与精确 Source Revision。

`Draft-to-Publish Revalidation` 必须确认 Release Build 成功后只产生 Draft Release，最终 Publish 需要第二次独立显式人工确认，并只能由独立的 `Publish Release` Workflow 执行；两个 Workflow 不得合并成自动连续 Build-and-Publish。Publish 前必须重新验证 Source Revision、全部 Platform Artifacts、Manifest、SHA-256、Artifact Attestations 与完整 Deployment Acceptance 结果。Build Confirmation 不得等同于 Publish Confirmation。

`Local Release Retention Validation` 必须验证默认保留 `Active Release + latest 3 previously active Releases`，并确保仍被有效 PRE_UPGRADE_BACKUP、Rollback、Restore / Recovery 或 `app/current` 引用的 Protected Release 不会被自动清理。历史 Release Cleanup 只能在 Upgrade 成功、Post-upgrade Readiness 通过并重新计算保护引用后执行。

Critical Deployment / Security Failure 必须 Block Release。

---

# 34. 当前实现边界

本文档定义的是 O3Pilot v1 正式 Deployment Contract。

当前项目处于新架构开发阶段，因此文档中的：

```text
installer
production release package
migration runner
backup / restore implementation
o3pilotctl
platform service adapter
```

描述的是后续实现必须满足的行为合同，不表示这些能力已经完成实现或 Production Acceptance。

正式实现必须采用本文已经固定的用户入口：

```text
install.sh
update.sh
uninstall.sh
<O3PILOT_ROOT>/bin/o3pilotctl
```

历史 O3Pilot / oPanel 的 macOS `launchd`、Loopback Binding、端口检查、运行前测试和启动后验证经验可以作为参考，但历史项目中的 Production `git pull`、项目目录混合 Runtime State、macOS Intel 兼容或 macOS-only Installer 不构成当前项目约束。

---

# 35. 核心原则

**macOS 最低正式版本为 macOS 26 Tahoe，只支持 Apple Silicon / arm64，不支持 Intel Mac。**

**Linux v1 正式支持 Ubuntu 24.04 LTS、Ubuntu 26.04 LTS、Debian 13 的 x86_64 / arm64 systemd 环境。**

**macOS 采用 Single Application Root：除系统强制位置的 launchd Service Definition 外，O3Pilot 自有文件全部位于 `/Library/O3Pilot/`。**

**Linux 采用 Single Application Root：除 systemd Service Definition 等操作系统强制位置外，O3Pilot 自有文件全部位于 `/opt/O3Pilot/`。**

**Production Artifact 是平台 / 架构独立、预构建、自包含的 `.tar.gz`；`.tar.gz` 只是 Temporary Transfer Artifact，验证与安装完成后必须删除。**

**GitHub Releases `LI5ee3/O3Pilot` 是 Production Release Canonical Source；Production 不直接从 Git Working Tree 更新。**

**Production Release 必须启用 GitHub Release Immutability，并为正式平台 Artifact 生成和验证 GitHub Artifact Attestation；v1 不引入 GPG / Minisign / Cosign 等自定义签名体系。**

**Production Release Build 与 Publish 必须由两个独立的手动 GitHub Actions Workflow 执行；Build 只能生成 Draft，第二次人工确认后才能 Publish。**

**本机 Release 默认保留 Active Release + 最近 3 个曾激活 Release；任何受有效 PRE_UPGRADE_BACKUP、Rollback / Restore 或 `app/current` 保护的 Release 不得自动清理。**

**最终用户主机不承担 Python / Node / Git / jq / Homebrew / Compiler 等开发环境责任。**

**Application 可以替换，Persistent Data 不能随 Release 被覆盖。**

**SQLite 永远运行在本机持久化文件系统。**

**正式 O3Pilot 只监听 Loopback。Linux Headless 首次初始化使用 SSH Local Port Forwarding，不开放 Public Setup。**

**公网访问通过独立 cloudflared / Cloudflare Tunnel，而不是开放 Application Port；O3Pilot 不接管 cloudflared 生命周期或 Credential。**

**Cloudflare R2 是 Backup Replica，不是数据库。没有 R2，O3Pilot 仍必须完整运行、备份和恢复。**

**Database Backup 不等于 Portable Instance Backup；Recovery Key 必须与 Backup 分离。**

**Backup Retention 必须 Manifest-aware / Reachability-aware，不能按文件年龄暴力删除仍被引用对象。**

**服务可以从普通进程 Crash 自动恢复，但 Persistent Integrity Failure 必须 Fail Closed；禁止自动 Restore、自动 Rollback 或自动重建数据库。**

**O3Pilot v1 同 Major 内保证旧 Stable / Persistent Format 可以直接升级到最新 Stable；升级前先证明兼容并证明可以恢复，再修改持久化状态。**

**运行日志有明确 Rotation / Retention / Size Cap；结构化业务和运维历史不能被当成 Runtime Log 一起清理。**

**日常实例运维统一通过 `<O3PILOT_ROOT>/bin/o3pilotctl`；安装、更新、卸载仍分别由官方 Shell Lifecycle Entry Point 负责。**

**所有部署操作都不能突破 O3Pilot 对 Ozon 的永久只读边界。**
