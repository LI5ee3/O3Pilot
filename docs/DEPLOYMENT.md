# O3Pilot — DEPLOYMENT.md

> Version: 1.1  
> Status: B.1 Adjudicated Deployment Contract  
> Updated: 2026-09-05  
> Applies to: O3Pilot  
> Parent baseline: `3a53d4e86c9473550fa5bded1e769c0e0ba6ee042e7e199964ae6b774fb2ee07`  
> Authority freeze: `Batch_B.0.1_Authority_Coupling_Freeze.md` @ `08aa0d03a111ec54ce9a177fde3c3ae6b23ae4e2335545d8be502f347e3f524b`

# 1. Scope & Authority

`DEPLOYMENT.md` defines the operational deployment contract for O3Pilot:

- supported deployment shape and machine prerequisites;
- application root, files, service definitions and OS integration;
- release artifact installation, update and rollback operations;
- local backup repository, backup lifecycle, restore and cross-host operations;
- Cloudflare Tunnel and R2 operational roles;
- uninstall / full purge;
- stable local operator CLI;
- deployment and release acceptance orchestration.

This document does **not** redefine:

- product scope or Feature Phase;
- Ozon endpoint capabilities;
- entity / key / lineage / schema;
- metric formulas;
- runtime architecture invariants owned by `ARCHITECTURE.md`;
- Authentication / Session / Secret / security semantics owned by `SECURITY.md`;
- UI behavior owned by `DESIGN.md`.

The authority rules frozen by B.0.1 are the execution authority. This document only owns the physical / operational projection assigned to DEPLOYMENT.

O3Pilot remains permanently read-only toward Ozon. This document does not create any Ozon write capability; the canonical controls live in `ARCHITECTURE.md` and `SECURITY.md`.

---

# 2. Deployment Baseline

O3Pilot v1 uses:

```text
Native Installation
Single Machine
Single O3Pilot Primary Instance
Unprivileged Runtime
Local Persistent SQLite
Local Raw Store
Local Backup Repository
Persistent Data Separated from Replaceable Application Releases
Loopback-only Application Listener
Cloudflare Tunnel for optional Remote Access
Optional Cloudflare R2 encrypted Backup Replica
No Docker
No Public Application Port
No Remote Primary Database
```

Runtime topology:

```text
Browser
   │
   │ local access OR HTTPS through Cloudflare Tunnel
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
   └── Secret Store

Local Backup Repository
   │
   └── optional encrypted off-device replica
          ▼
     Cloudflare R2
```

`cloudflared` is an independent external service. R2 is not part of normal database reads or writes.

The following are outside the v1 deployment model:

```text
Windows Native
Docker / Docker Compose
Kubernetes
Serverless Runtime
Cloudflare Workers Runtime
Remote SQLite
SQLite over NFS / SMB / WebDAV / object-store mounts
Shared multi-host primary database
```

PostgreSQL, Cloudflare D1 and Hyperdrive are not O3Pilot v1 runtime dependencies.

---

# 3. Platform Compatibility

Platform compatibility is evidence- and capability-driven. Distribution or OS-version names are not, by themselves, the primary compatibility contract.

## 3.1 Classification

Every platform / architecture combination is classified as one of:

```text
Officially Tested
Not Qualified
Unsupported
```

Definitions:

- **Officially Tested** — the current Production Release has completed the required deployment acceptance on that platform / architecture.
- **Not Qualified** — required machine-verifiable prerequisites may be satisfied, but complete Production Acceptance evidence is absent.
- **Unsupported** — a concrete technical incompatibility exists, such as unavailable production artifact, unsupported CPU architecture, missing required service-manager capability, incompatible runtime/userspace, or non-local primary storage.

`Not Qualified` must not be presented as `Officially Tested`, but lack of qualification alone is not proof of incompatibility.

## 3.2 Machine-verifiable prerequisites

A production installation may enforce only prerequisites that are actually required by the release and can be verified on the host, including as applicable:

```text
64-bit supported CPU architecture
matching Production Artifact
required OS/runtime capability declared by the Artifact / Compatibility Manifest
launchd on macOS OR systemd on Linux
local persistent filesystem
working HTTPS + CA validation for lifecycle downloads
administrator capability for install/update/uninstall operations
```

Linux production artifacts assume the runtime/userspace declared by the release compatibility metadata. A build that requires glibc must not be installed on a musl-only host.

Installer must fail closed when a required machine-verifiable prerequisite fails. It must not reject an otherwise compatible host merely because its distribution name is absent from a static allowlist.

## 3.3 Current platform families

O3Pilot v1 publishes production artifacts for:

```text
macOS Apple Silicon / arm64
Linux x86_64
Linux arm64
```

Exact tested OS versions are release-acceptance evidence, not a permanent theoretical minimum unless a real runtime dependency requires that minimum.

The following remain unsupported for v1 because the deployment model or artifact set does not support them:

```text
Intel Mac / macOS x86_64
Rosetta 2 as Production Deployment compatibility
32-bit operating systems
non-systemd Linux where the required service contract cannot be provided
WSL
Windows Native
container-only deployment
```

---

# 4. Runtime Identity

Long-running O3Pilot runtime must use a dedicated non-root OS identity.

```text
O3Pilot Runtime != root
OS Runtime Identity != O3Pilot Owner
```

Installer, updater, uninstaller and operator commands may obtain administrator privileges for bounded OS operations, but administrator privileges must not become the long-running application identity.

---

# 5. Application Root & Storage

## 5.1 Platform root mapping

```text
macOS → O3PILOT_ROOT=/Library/O3Pilot
Linux → O3PILOT_ROOT=/opt/O3Pilot
```

Except for OS-mandated external service definitions, all O3Pilot-owned application, persistent, state and log files live under `O3PILOT_ROOT`.

Generic layout:

```text
O3PILOT_ROOT/
├── bin/
│   └── o3pilotctl
├── app/
│   ├── releases/<version>/
│   └── current
├── data/
│   ├── o3pilot.db
│   └── raw/
├── backups/
│   └── repository/
├── config/
├── secrets/
├── state/
│   ├── run/
│   ├── staging/
│   └── tmp/
└── logs/
```

`app/` and `bin/` are replaceable application state. `data/`, `backups/`, `config/`, `secrets/` and required instance compatibility state are persistent instance state.

## 5.2 OS-mandated external files

Allowed external O3Pilot service-definition locations:

```text
macOS → /Library/LaunchDaemons/com.o3pilot.app.plist
Linux → /etc/systemd/system/o3pilot.service
```

No second O3Pilot application/data/log root is created elsewhere merely for convenience.

## 5.3 Primary storage

Primary SQLite and Local Raw Store must be on a local persistent filesystem.

The following must not be used as O3Pilot primary storage:

```text
NFS
SMB
WebDAV
R2 mount
FUSE-backed remote object storage
cloud-drive synchronized directory
```

## 5.4 Permissions

DEPLOYMENT implements the physical permission boundary required by `SECURITY.md`.

At minimum:

```text
Runtime identity receives only required access
Secret files are not world-readable
Business data is not world-readable
Secret / sensitive directories are restricted
```

Where POSIX modes are applicable, the implementation baseline is equivalent to:

```text
secret file → 0600
secret / sensitive directory → 0700
```

Full-disk encryption remains a host hardening recommendation, not a substitute for application-level secret and backup protection.

---

# 6. Port, Staging & Temporary Input

O3Pilot listens on Loopback only.

Default:

```text
listen_address = 127.0.0.1
application_port = 38652
```

Default binding must not become:

```text
0.0.0.0
::
LAN IP
Public IP
```

Port occupancy is checked before activation. A conflict must not silently select a random port; an alternative requires an explicit configured value.

Manual CSV / XLS / XLSX uploads use temporary staging under:

```text
<O3PILOT_ROOT>/state/staging/
```

Lifecycle:

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

The uploaded source file is an import medium, not long-term Raw Store or R2 Backup content.

---

# 7. Release & Artifact Contract

## 7.1 Production release source

Production does not use a Git working tree as the release source.

Canonical release distribution:

```text
GitHub Releases
Repository: LI5ee3/O3Pilot
```

Normal installer / updater selects a non-Draft, non-Pre-release Stable release compatible with the target host.

Published release identity must not be silently replaced. A fix to a published version requires a new version.

## 7.2 Production artifact

A Production Artifact is:

```text
platform-specific
architecture-specific
prebuilt
self-contained for the target host
free of Instance Data
immutable after publication
installed into app/releases/<version>/
```

The target host is not required to install a Python / Node / compiler / Git development environment to run a Production Artifact.

The exact transport container, including `.tar.gz`, is an implementation choice unless a real installer/runtime consumer requires that exact format. Whatever transport is used remains temporary and is deleted after verified installation.

## 7.3 Release metadata

A release provides enough metadata to verify identity, integrity and compatibility. Runtime-consumed compatibility metadata must include:

```text
application_version
platform
architecture
artifact_checksum
persistent_format
minimum_supported_persistent_format
compatibility information required by the updater
```

Build/source identity may additionally be recorded for provenance and diagnostics.

`minimum_supported_persistent_format` is a release declaration. Its value is not permanently frozen to `1`.

## 7.4 Integrity, provenance and publication

Production release policy must preserve:

```text
artifact byte-integrity verification
published release immutability
build/source provenance evidence
explicit human approval for a Production Release
```

CI / GitHub Actions workflow names, job identities and a particular Draft/Publish implementation are not runtime Deployment Contract. They may evolve as long as the above release guarantees and Deployment Acceptance remain satisfied.

Installer / updater must verify the artifact checksum, manifest/identity and target-platform compatibility before activation. No hidden `--skip-checksum`, arbitrary repository override or arbitrary artifact URL is part of the production lifecycle surface.

## 7.5 Release layout & retention

```text
<O3PILOT_ROOT>/app/releases/<version>/
<O3PILOT_ROOT>/app/current → active release
<O3PILOT_ROOT>/bin/o3pilotctl → stable operator entry point
```

Application release changes must not overwrite persistent instance state.

Local retention protects:

```text
active release
release required by a retained VERIFIED PRE_UPGRADE restore point
release required by an in-progress rollback
release required by an in-progress restore/recovery
any other release with an active recovery reference
```

A fixed long-term count such as “latest 3 previously active releases” is not a normative compatibility guarantee. Unprotected history may be cleaned only after the new release passes post-upgrade readiness and protection references are recomputed.

Release cleanup must not use directory age alone and must not remove a protected release merely because disk is low.

---

# 8. Lifecycle Entrypoints & Shared Preconditions

Official lifecycle entrypoints:

```bash
curl -fsSL https://o3pilot.app/install.sh | sudo bash
curl -fsSL https://o3pilot.app/update.sh | sudo bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash
```

Roles:

```text
install.sh
→ Fresh Install
→ Retained Instance Reinstall

update.sh
→ Existing active Instance Upgrade

uninstall.sh
→ Application Uninstall
→ explicit Full Purge with --purge
```

`install.sh` must not silently upgrade a running installed instance. `update.sh` must not silently become a Fresh Install.

Shared lifecycle principles:

```text
detect host and existing-instance state
verify required host capabilities
resolve compatible artifact / release
verify integrity before activation
use bounded administrator privileges
preserve persistent instance state unless Full Purge was explicitly confirmed
fail closed on ambiguous identity / path / compatibility / integrity
best-effort clean incomplete staging without deleting a valid existing release
```

Host lifecycle scripts must work with system-provided shell and utilities declared by the release. Production installation must not depend on undeclared developer-machine tools such as Homebrew, Git, Python, Node.js, jq or compiler toolchains.

TLS/CA verification must not be disabled.

---

# 9. Install & Reinstall

## 9.1 Fresh Install

Canonical operational flow:

```text
Detect host / architecture / required capabilities
↓
Verify lifecycle prerequisites
↓
Capacity + port + existing-instance preflight
↓
Create minimal staging
↓
Resolve compatible Stable release
↓
Download + verify artifact / manifest / checksum
↓
Create O3PILOT_ROOT + directory structure
↓
Create dedicated non-root Runtime Identity
↓
Install release into app/releases/<version>/
↓
Create stable o3pilotctl entry point
↓
Set app/current
↓
Install platform service definition
↓
Start O3Pilot on Loopback
↓
Run readiness check
↓
Expose local-only Owner Initialization entry
```

macOS uses `launchd`; Linux uses `systemd`. Platform-specific steps branch only where the OS actually differs.

## 9.2 Retained Instance Reinstall

When `install.sh` finds an `APPLICATION_UNINSTALLED / RETAINED` instance, it performs a retained-instance reinstall:

```text
validate retained instance
preserve Instance Identity
preserve data / backups / config / secrets
resolve a compatible Stable release
restore app/current + o3pilotctl
restore OS service definition
start
run compatibility + readiness validation
```

This is not an active-instance upgrade.

---

# 10. Local Owner Initialization

DEPLOYMENT defines the local operational path only. Initialization-token security properties are owned by `SECURITY.md` under B0-C16.

An uninitialized instance is exposed only through the local Loopback origin. The deployment flow does not route first-time Owner initialization through Cloudflare Tunnel.

Desktop host:

```text
Installer starts O3Pilot on Loopback
↓
Installer presents the local initialization entry
↓
Browser opens the local origin
```

Headless Linux:

```bash
ssh -N -L <local-port>:127.0.0.1:<o3pilot-port> <user>@<server>
```

The user then opens the forwarded local address. A local-side port conflict may change only the SSH local port; the O3Pilot server port is unchanged.

If the initialization mechanism uses a bootstrap token or capability, its lifetime, replay protection, host/context checks and invalidation semantics are defined by `SECURITY.md`, not by this file.

DEPLOYMENT does not define an in-place Password Reset command. Lost-password security and supported data-recovery semantics are defined by `SECURITY.md` §5.7; any supported operational recovery uses the canonical Restore flow in §15 / §20.

---

# 11. Service & Process Projection

Platform mapping:

```text
macOS → launchd
Linux → systemd
```

The service definition implements the runtime lifecycle owned by `ARCHITECTURE.md`; DEPLOYMENT does not redefine that lifecycle.

Service-manager behavior must provide:

```text
boot auto-start
dedicated non-root runtime identity
correct working/config context
Loopback binding
graceful stop
bounded restart after unexpected process failure
crash-loop protection
no plaintext business secret embedded in the service definition
```

Unexpected process crashes may restart the same release. Planned stops, update, uninstall, maintenance and OS shutdown must not be misclassified as crashes.

Persistent integrity, unsupported persistent format, secret-store decryption failure, missing active release, instance-identity conflict and comparable fatal state must fail closed rather than enter an automatic hot loop.

Service recovery restores a process, not persistent state. It must not perform automatic:

```text
Backup Restore
Release Rollback
Database Rebuild
SQLite replacement
Raw deletion
```

Single-primary and startup/shutdown semantics remain owned by `ARCHITECTURE.md`. DEPLOYMENT implements the OS-level exclusive instance lock and service-manager projection.

---

# 12. Cloudflare Tunnel & R2

## 12.1 Cloudflare Tunnel

Cloudflare Tunnel is the optional remote-access path:

```text
Public Hostname
↓
Cloudflare
↓
cloudflared
↓
http://127.0.0.1:<o3pilot-port>
```

`cloudflared` is independently installed, configured, upgraded and removed. O3Pilot lifecycle scripts do not own it.

O3Pilot may use the configured Public Hostname as deployment context. HSTS and browser/public-edge security policy are owned by `SECURITY.md` under B0-C15.

Security classification, handling and redaction of any cloudflared / Tunnel credential material are defined canonically by `SECURITY.md` §4.5–§4.6.

Tunnel failure affects Remote Access only; it must not make local O3Pilot, Scheduler, Worker or Local Backup unavailable.

## 12.2 Cloudflare R2

R2 is:

```text
Optional Off-device Encrypted Backup Replica
```

It is not:

```text
Primary Database
SQLite Filesystem
Primary Raw Store
Mandatory Runtime Dependency
```

Local backup success does not depend on R2 replica success. R2 failure must not invalidate an already verified Local Backup.

Encryption, credential and least-privilege security semantics are owned by `SECURITY.md`. DEPLOYMENT owns upload / verify / retry / replica operations.

---

# 13. Backup Artifact, Repository & Retention

## 13.1 One Backup Artifact format

O3Pilot uses one canonical Backup Artifact format. Purpose/lifecycle does not create separate file formats.

A recoverable artifact carries or references:

```text
consistent SQLite snapshot
required Local Raw objects
portable config
portable encrypted secret state
Backup Manifest
compatibility metadata
integrity metadata
```

The artifact is `VERIFIED` only after creation and integrity verification succeed.

Lifecycle/reason examples:

```text
NORMAL
PRE_UPGRADE
MANUAL / PORTABLE USE
```

`portable` means the same canonical artifact is copied/exported for off-device or cross-host use; portable use does not create a second backup format or backup class.

## 13.2 Local repository

```text
macOS → /Library/O3Pilot/backups/repository/
Linux → /opt/O3Pilot/backups/repository/
```

The Local Backup Repository is on the same host as primary data and therefore does not protect against total-host / total-disk loss. R2 or separately retained portable copies may provide off-device resilience.

Local Backup Manifest/reachability is the retention source of truth. R2 is a replica of retained encrypted backup state, not a second independent retention authority.

## 13.3 Retention

Default Local `NORMAL` restore-point density remains:

```text
Daily  → 14 days
Weekly → 8 weeks
Monthly → 12 months
```

Default `PRE_UPGRADE` retention remains:

```text
latest 3 VERIFIED PRE_UPGRADE restore points
```

A restore point or object needed by an active upgrade / rollback / restore / recovery must not be retired merely because a count or age threshold is reached.

User-created portable copies outside the managed Local Repository are not silently deleted by O3Pilot.

Retention is Manifest- and reachability-aware:

```text
retire restore point / manifest
↓
recalculate object reachability
↓
only unreferenced objects become GC candidates
```

Directory/file age alone must not delete objects still referenced by a valid Backup Manifest.

## 13.4 PRE_UPGRADE restore point

Before any operation that can change persistent format, the updater creates and verifies a `PRE_UPGRADE` restore point.

Failure to create or verify it:

```text
ABORT UPGRADE
```

The protected scope covers persistent state that the migration can change.

---

# 14. Upgrade & Rollback

## 14.1 Upgrade compatibility

Application Version and Persistent Format are separate dimensions.

Every target release declares:

```text
persistent_format
minimum_supported_persistent_format
```

Compatibility is determined from the target release declaration and actual current persistent format. There is no blanket contract that all earlier v1 persistent formats are directly upgradable to every future v1 release.

If current persistent state is below the target release’s declared supported window:

```text
UPGRADE NOT COMPATIBLE
→ fail before modifying the instance
```

If the target supports the current format, the migration runner applies the required ordered migrations.

Skipped application versions are acceptable only when the declared persistent-format compatibility and migration chain support that path.

Downgrade is not an `update.sh` capability. Same-version update exits successfully without modification. Draft / Pre-release does not enter the normal Stable update path.

## 14.2 Canonical upgrade flow

```text
Validate existing active instance
↓
Read current application version + persistent format
↓
Resolve compatible target Stable release
↓
Compatibility preflight
↓
No new compatible release → exit successfully
↓
Capacity + persistent-state integrity preflight
↓
Download + verify artifact
↓
Quiesce runtime through the approved lifecycle
↓
Acquire exclusive instance lock
↓
Create + verify PRE_UPGRADE restore point
↓
Install new release
↓
Run required migrations
↓
Verify migrated persistent state
↓
Switch app/current
↓
Start through platform service manager
↓
Readiness + post-upgrade validation
↓
Recompute release-protection references
↓
Mark upgrade succeeded
```

Persistent instance directories are never replaced by application-release installation.

A migration must have an auditable, crash-aware execution model and a defined way to resume or safely fail.

## 14.3 Rollback

If persistent migration has not changed state incompatibly, rollback may switch back to the previous protected release.

If persistent migration has changed state that the previous release cannot understand:

```text
Stop target release
↓
Restore verified PRE_UPGRADE restore point
↓
Switch app/current to previous protected release
↓
Start
↓
Verify
```

Generic reverse migration is not the rollback foundation.

Automatic crash-loop rollback is not allowed.

---

# 15. Restore & Cross-host Restore

## 15.1 Canonical Restore operation

DEPLOYMENT owns one operational Restore model. `o3pilotctl restore` is its CLI projection; there is not a second independent Restore flow.

Canonical operation:

```text
Resolve requested Backup Artifact / restore source
↓
Capacity preflight
↓
Verify Manifest / artifact integrity
↓
Verify application + persistent-format compatibility
↓
Display exact restore source and target
↓
Satisfy SECURITY-defined Restore authorization gate
↓
Require explicit destructive confirmation from a real terminal
↓
Stop O3Pilot through the approved lifecycle
↓
Acquire exclusive Restore / Instance lock
↓
Restore into isolated staging
↓
Materialize required DB / Raw / Config / portable encrypted secret state
↓
Verify staged SQLite / object integrity
↓
Preserve any SECURITY-required pre-replacement recoverable point
↓
Reset runtime-specific state
↓
Apply supported migration when required
↓
Promote atomically
↓
Apply SECURITY-defined post-restore session / secret / temp-data requirements
↓
Start
↓
Readiness check
```

The authorization proof mechanism, Recent Authentication semantics, Recovery Key role, cross-host secret rewrap, decrypted-temporary-material protection and session invalidation semantics remain owned by `SECURITY.md` under B0-C17.

The security authorization requirement must be satisfied before the destructive transition. DEPLOYMENT does not redefine what proves authorization.

Restore never streams a Backup directly over the live production database and must fail closed on integrity / compatibility failure.

There is no production escape hatch equivalent to:

```text
--force-without-verification
--ignore-integrity
--ignore-compatibility
```

## 15.2 Cross-host Restore

The same canonical Backup Artifact may be transferred to another supported/qualified target host and restored there.

A portable restore source must not depend on source-host runtime-only state such as:

```text
absolute Application path
virtualenv path
PID / UID
launchd / systemd runtime state
cloudflared credential / connector runtime state
temporary directory
browser Session
```

Target-host operational requirements:

```text
verify target host capabilities
establish target O3Pilot runtime identity / filesystem layout
satisfy SECURITY-defined target-host Owner/recovery/bootstrap precondition
restore the canonical Backup Artifact
rebuild target-host runtime-specific state
re-establish cloudflared separately if remote access is desired
```

Cross-host secret re-protection and handling of Owner credential state are SECURITY-owned requirements and are not redefined here.

## 15.3 Full Purge is not a recovery path

Full Purge removes the Local Backup Repository under `O3PILOT_ROOT`. Recovery after Full Purge is possible only if a verified restore source survives outside the purged local instance, such as a user-retained portable copy or an available off-device replica.

---

# 16. Logs, Temporary Files & Redaction

O3Pilot-owned persistent logs:

```text
<O3PILOT_ROOT>/logs/
```

Temporary / staging state:

```text
<O3PILOT_ROOT>/state/tmp/
<O3PILOT_ROOT>/state/staging/
```

No second persistent O3Pilot log directory is created in `/var/log`, `/Library/Logs` or user Home.

Default runtime-log policy remains:

```text
Retention: 30 days
Maximum total O3Pilot runtime log storage: 1 GiB
Rotate: daily OR 20 MiB per active log file
```

Structured Job / Backup / Import / Alert / Sync / Migration / Security / Operation history stored in persistent data is not a runtime log and is not deleted by runtime-log retention.

Sensitive-data terminology and canonical redaction obligations are owned by `SECURITY.md`. DEPLOYMENT references that vocabulary instead of maintaining a duplicate generic secret list.

---

# 17. Capacity, Disk & SQLite Operations

## 17.1 Capacity preflight

No fixed CPU / RAM / free-disk support boundary is frozen without release evidence.

Capacity preflight is required before operations whose temporary or persistent working set can exceed available capacity, including:

```text
Fresh Install
Startup
Backup
Upgrade
Restore
Large Import
```

Upgrade estimates at least:

```text
downloaded artifact / transfer object
extracted release
PRE_UPGRADE restore-point requirement
migration workspace
safety reserve
```

Backup, Restore and Large Import estimate their own working set before high-risk writes.

Insufficient capacity must fail before the operation can fill the disk or partially modify persistent state.

## 17.2 Runtime disk pressure

Disk health monitors:

```text
free disk
SQLite size
Raw Store size
Backup Repository size
log size
temporary usage
```

Warning / critical thresholds are operational policy derived from measured workload, configured safety reserve and the capacity required by protected operations; this document does not freeze unevidenced universal GiB thresholds.

At critical disk pressure, high-growth/high-risk writes may be paused while retaining, where safe:

```text
UI
Authentication
Diagnostics
existing data reads
cleanup / maintenance
```

Disk pressure must not trigger automatic deletion of:

```text
SQLite business data
IRREPLACEABLE Raw
valid restore points
objects referenced by active Backup Manifests
protected application releases
```

## 17.3 SQLite safety

Backup consistency is an `ARCHITECTURE.md` invariant. DEPLOYMENT implements it with a SQLite-supported consistent snapshot mechanism; ordinary file copy of a live database is not a formal backup.

Routine operations use Migration / Maintenance / Recovery / Reprocess capabilities rather than ad-hoc production SQL mutation.

---

# 18. Health & Diagnostics

`health` and `diagnostics` are distinct operator surfaces.

`health` provides lightweight current state, including as applicable:

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

External Ozon / R2 / Internet / Cloudflare failure does not automatically mean the local O3Pilot process is not ready.

Readiness consumes the runtime/integrity contract from `ARCHITECTURE.md` and includes the local conditions required to safely operate.

`diagnostics` performs deeper operator inspection of host, application, persistent state, service, permissions, backups, logs and integrations.

Neither health nor diagnostics may bypass integrity/security checks or become a dangerous repair shell.

---

# 19. Uninstall & Full Purge

## 19.1 Application Uninstall

Default:

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash
```

Outcome:

```text
Stop O3Pilot
↓
Disable / unregister platform service
↓
Remove service definition
↓
Remove replaceable application releases + launcher
↓
Preserve persistent instance state
↓
Mark instance APPLICATION_UNINSTALLED / RETAINED
```

Preserved state includes:

```text
data/
backups/
config/
secrets/
required instance / compatibility state
dedicated runtime identity required to preserve ownership/reinstallability
```

Reinstall uses `install.sh` and follows the Retained Instance Reinstall path.

## 19.2 Full Purge

Explicit command:

```bash
curl -fsSL https://o3pilot.app/uninstall.sh | sudo bash -s -- --purge
```

Full Purge is destructive and local-only.

Before deletion:

```text
detect and validate exact local instance
display exact deletion scope
warn that local backups under O3PILOT_ROOT will be destroyed
recommend creating/verifying an external surviving Backup Artifact if recovery is desired
require explicit destructive confirmation from /dev/tty or equivalent real terminal
stop O3Pilot
ensure instance lock is released
```

Confirmation must not be satisfied by normal piped stdin, timeout default, `yes`, or other non-interactive input.

Allowed local deletion boundary:

```text
macOS:
  /Library/O3Pilot/
  /Library/LaunchDaemons/com.o3pilot.app.plist
  dedicated O3Pilot runtime identity

Linux:
  /opt/O3Pilot/
  /etc/systemd/system/o3pilot.service
  dedicated O3Pilot runtime identity
```

Linux performs the required `systemd daemon-reload` after removing the service definition.

If the exact target path, service identity, instance identity or deletion boundary is ambiguous:

```text
ABORT PURGE
```

Full Purge must not follow a symlink/path traversal outside the validated local instance.

## 19.3 External-resource boundary

Uninstall and Full Purge do not delete or mutate external resources such as:

```text
Cloudflare Tunnel
Cloudflare DNS
Cloudflare R2 Bucket
R2 Backup Replica
Cloudflare Access configuration
```

They also do not modify Ozon Webhook subscriptions, products, prices, inventory, orders, promotions or any other Ozon configuration.

Ozon read-only remains an upstream `ARCHITECTURE.md` + `SECURITY.md` invariant.

---

# 20. Operator CLI

Stable entry point:

```text
macOS → /Library/O3Pilot/bin/o3pilotctl
Linux → /opt/O3Pilot/bin/o3pilotctl
```

Lifecycle scripts own install/update/uninstall. `o3pilotctl` owns installed-instance operations.

Stable surface:

```text
o3pilotctl service status
o3pilotctl service start
o3pilotctl service stop
o3pilotctl service restart

o3pilotctl health
o3pilotctl version
o3pilotctl backup create
o3pilotctl backup create --portable
o3pilotctl backup list
o3pilotctl backup verify <backup-id>
o3pilotctl restore <backup-id>
o3pilotctl restore --from <backup-artifact>
o3pilotctl logs
o3pilotctl logs --follow
o3pilotctl logs --since <duration>
o3pilotctl diagnostics
```

`backup create --portable` creates/exports the same canonical Backup Artifact for portable use; it does not create a second backup format/class.

Backup create performs capacity preflight, consistent snapshot and integrity verification. `backup verify` does not modify the instance.

Restore commands are a projection of §15 and must not define an alternative sequence.

Core local operator commands remain available without Internet / GitHub / Cloudflare / Ozon except for checks that inherently require an external service.

Operator commands may acquire administrator privilege for bounded service/backup/restore/permission operations; they must not run the long-lived O3Pilot runtime as root.

Production CLI does not expose:

```text
arbitrary SQL shell
arbitrary application shell
raw-delete
force-schema-version
unsafe config mutation
force unlock bypassing active-instance safety
integrity bypass
compatibility bypass
```

A future exceptional recovery capability requires its own approved contract.

---

# 21. Platform Acceptance

Production support is evidence-based.

For each release and platform / architecture combination:

```text
Officially Tested
→ required platform acceptance evidence exists and passes

Not Qualified
→ prerequisites may be satisfied, but complete production acceptance evidence is absent

Unsupported
→ concrete technical incompatibility is documented
```

A release’s Platform Acceptance records, at minimum where applicable:

```text
Fresh Install
Retained Instance Reinstall
Start / Stop
Upgrade
Rollback
Backup
Restore
Cross-host Restore
Tunnel deployment
Uninstall
Full Purge
Operator CLI
```

A platform name in documentation is not, by itself, proof that the current release is `Officially Tested`.

---

# 22. Deployment / Release Acceptance Gate

A Production Release consumes the relevant upstream Architecture and Security PASS results and must validate the Deployment contract that changed or is exercised by the release.

Required coverage includes, where applicable:

```text
artifact identity + integrity
runtime-consumed compatibility metadata
canonical release source
no undeclared host dependency
Fresh Install
Retained Instance Reinstall
Upgrade compatibility preflight
PRE_UPGRADE restore point
migration + readiness
rollback
Backup Artifact create / verify / retention
Restore canonical flow
Cross-host Restore
single-primary OS lock implementation
service crash / fail-closed behavior
capacity / disk failure paths
Cloudflare Tunnel failure isolation
R2 outage without local-runtime failure
log / temp lifecycle
health + diagnostics
Application Uninstall
Full Purge destructive boundary
external-resource non-deletion
Operator CLI contract
Ozon read-only upstream controls
```

Acceptance must not reintroduce deleted contracts.

In particular:

```text
Oldest/all-v1 direct-upgrade blanket test = prohibited
fixed GitHub workflow identity as runtime acceptance = prohibited
unevidenced fixed resource-minimum test = prohibited
second Restore flow = prohibited
second §35-style normative contract = prohibited
```

Compatibility-path acceptance validates the **declared** compatibility window of the target release, not a permanent “from every v1 state” promise.

Critical Deployment failure blocks the release.

---

# 23. Implementation Boundary

This document defines the intended v1 Deployment Contract. Installer, updater, backup/restore implementation, migrations, operator CLI and platform adapters may still be under implementation.

Required user-facing lifecycle entrypoints remain:

```text
install.sh
update.sh
uninstall.sh
<O3PILOT_ROOT>/bin/o3pilotctl
```

Historical O3Pilot / oPanel implementation details may be used as engineering reference only when they do not contradict this contract or the Frozen B.0.1 authority boundary.

No retained rule in this document is justified merely because it existed in the previous baseline. Every surviving rule must have a clear operational reason, one authoritative owner and a validation value. `KEEP_LOCAL` is not a synonym for “not reviewed”.
