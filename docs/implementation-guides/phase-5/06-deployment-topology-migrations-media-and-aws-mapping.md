# Packet 06: Deployment Topology, Migrations, Media, And AWS Mapping

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packets 01-02 and 05 Design Approved; `P5-DB-001`, `P5-MIG-001`, `P5-DEPLOY-001`, `P5-AWS-001` |
| Ownership | Release/operations own topology; database owns migrations; media/module owners own data behavior |
| Manual Review | Mandatory topology, migration, secrets, IAM, payment, storage, feature-flag, rollback and release review |
| Produces | Deployment/migration/rollout design and `evidence/06-completion.md`; no deployment code |

## Objective

Design Web/API/worker deployment, configuration/secrets injection, EF migration execution, media handling, compatibility, feature flags, rolling/blue-green rollout, rollback/forward-fix, and future AWS mapping without provisioning or changing deployment state.

## Read First And Prerequisites

- Packet 01 inventory/environments/owners, Packet 02 health, Packet 05 artifact/pipeline contract.
- Approved database engine/DbContext/migrations, local media abstraction, Outbox/index workers, payment webhook/provider, data protection key and Phase 4 flags.
- Future AWS target architecture as a mapping only.

## In Scope

- Logical topology for Web/API/worker processes, ports/routes, scaling/statefulness, health/drain, deployment unit/version, database/storage/provider/config/secret/data-protection dependencies.
- Migration classification/review/generated script/backup/lock/backfill/expand-contract/compatibility/maintenance/forward-fix and execution authority.
- Static/public media and future private access, local-to-S3 migration boundary, checksums/lifecycle/deletion/rollback.
- Feature flag owner/default/expiry/audit/failure/rollback, rolling versus blue/green decision criteria, previous/current version compatibility and safe traffic shift.
- Future mapping to ECS/ALB/RDS/S3/CloudFront/WAF/SQS/EventBridge/CloudWatch/Secrets Manager/IAM, with local equivalent and later prerequisites.

## Out Of Scope

- Dockerfile, compose, workflow, IaC, ECS/RDS/S3/CloudFront/WAF/IAM resource or policy, DNS/certificate/network, secret, production SQL execution, container build/deploy, traffic shift, or AWS account action.

## Ownership And Operational Contracts

- **Owns:** logical deployable/topology/version/compatibility/rollout/migration/media/config/flag/rollback design.
- **Reads:** immutable artifact, health/security/observability, module data/provider contracts.
- **Must not own/mutate:** business state, migration source ownership, secret values, AWS resources/IAM, payment semantics, or go-live authority.

Every deployable records artifact digest/version, replicas/process, startup/live/ready/drain, state/storage, configuration and secret keys by class, data-protection key requirements, dependency/timeout/retry, scaling, worker singleton/lease behavior, and rollback compatibility.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Topology/data-flow/trust diagrams, migration/media/flag/rollout/rollback plans, AWS mapping and named approvals. |
| Local Verification Evidence | Process separation/config validation, migration fresh/upgrade/script review, media abstraction, worker idempotency, old/new compatibility tests. |
| Staging Evidence | Same artifact topology, migration rehearsal/timing/locks/backfill, rolling/blue-green/flag/drain/smoke/rollback drill. |
| Production Approval Evidence | Exact artifact/migration/backup/window/owners/health/monitoring/rollback/IAM/secrets/topology signed before execution. |

## API And UI Contract

- Public routes/domains/media URLs/webhooks must remain compatible during rollout; old/new app versions and schema coexist for approved window.
- Feature-disabled UI/API returns safe approved behavior and cannot bypass security/migrations.
- Payment webhook URL/validation/idempotency remains stable; topology/rate/routing change requires payment owner test/approval.
- No internal topology/secret/config detail in health/errors/media responses.

## Likely File Areas To Inspect

- Projects/Program/config templates/DbContexts/migrations/storage/outbox/workers/data protection/payment/webhook/feature settings and tests.
- Documentation only now; no container/workflow/IaC/deployment/cloud files.

## Architecture And Prohibited Dependencies

- Core remains deployment/cloud unaware. Infrastructure adapters enable future service swaps.
- Do not run unreviewed migrations at app startup. One controlled migration job/authority; applications verify compatibility and fail safely.
- Expand/contract supports old/new coexistence; destructive cleanup is a later separately approved release.
- Workers are horizontally safe through leases/idempotency; no duplicate payment/outbox/index effect.

## Database Changes And Migrations

This packet creates no migration. Define review template: owner/purpose/schema/data size, generated SQL, destructive/lock/rewrite/backfill risk, compatibility matrix, estimated/staging duration, transaction/batch, backup/restore, forward-fix, stop thresholds, execution identity, verification and audit. Manual DB approval is mandatory before every staging/production apply.

## Security, Logging, Audit, And Failure Rules

- Config and secret keys are separated; future task identity uses least privilege and no static AWS keys. Exact IAM is later manually reviewed.
- Feature flags are config, not authorization; high-risk changes are audited, default safe, time-bound and removed.
- Migration/media/deployment logs use IDs/version/status/timing, never SQL data/connection credentials/private URLs/PII/payment payloads.
- Failed migration/health/smoke/traffic shift stops rollout. Application rollback only when schema compatible; otherwise forward-fix under incident control.

## Implementation Steps

1. Resolve database/topology/rollout/media/flag/data-protection/AWS-account assumptions with owners.
2. Draw local/staging/future AWS topology and trust/data/dependency flows.
3. Define deployable process/health/config/secret/scaling/drain/worker behavior and compatibility matrix.
4. Define migration classification/script/review/backup/execute/verify/forward-fix/stop process.
5. Define media migration/checksum/access/cache/deletion/rollback and feature-flag lifecycle.
6. Define rolling/blue-green staging drill and production evidence template.
7. Run local compatibility/migration/media/worker tests only when executable; obtain all manual reviews.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Topology | Web/API/worker start/stop/scale/drain, dependency loss, singleton/lease/idempotency. |
| Compatibility | Old app/new schema, new app/old-expand schema, mixed rollout, flag on/off, API/media/webhook stability. |
| Migration | Fresh/upgrade, additive/backfill/destructive detection, lock/time/transaction/failure/forward-fix and backup dependency. |
| Media | Upload/read/delete/checksum/orphan/cache/old-new URL, local failure and future adapter contract. |
| Rollout | Health/smoke fail before/during/after shift, app rollback versus forward-fix decision. |
| Security | No secret/static key/internal detail; least-privilege future IAM matrix reviewed, no provider semantics change. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
dotnet ef migrations list --project ECommerce.Infrastructure --startup-project ECommerce.API
rg -n "Migrate\(|EnsureCreated|BackgroundService|Outbox|FileStorage|Feature|DataProtection|Webhook" ECommerce.API ECommerce.Web ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

Run only commands valid for the inspected repository; record `not run` with reason rather than editing code to force a planning command.

## Acceptance Criteria

- Web/API/worker topology, compatibility, health/drain, config/secrets, media, flags, migrations, rollout and rollback are explicit and owned.
- Migration process forbids startup auto-apply and includes generated script/manual review/backup/locks/backfill/forward-fix/verification.
- Future AWS mapping has local equivalent, trigger, IAM/secret/cost/operations prerequisites and no implementation.
- Design/local evidence pass; no deployment/cloud/migration execution or staging/production claim occurs.

## Manual Approval Gates

Manual approval is mandatory for topology, container/artifact, database engine/migration/script/apply, secrets/data-protection, future IAM, payment/webhook routing, media access, feature flags, rollout/traffic shift, rollback/forward-fix and production execution.

## Stop Conditions

Stop for request to deploy/provision/write deployment code, unresolved DB/topology/IAM owner, startup auto-migration, destructive/unreviewed migration, no backup/forward-fix, old-new incompatibility, duplicate worker/payment effect, secret/static AWS key, flag bypassing security, unsafe media access, or design marked as staging/production proof.

## Rollback / Forward-Fix And Handoff

This packet changes no deployment. Future rollout keeps last artifact, backup and compatible schema; stop traffic shift on gate failure. Roll back code only when compatible, otherwise incident-controlled forward-fix. Hand topology/migration/media dependencies to Packets 07-10/12.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 06 as documentation/local verification only after prior gates, Packets 01/02/05, P5-DB-001/P5-MIG-001/P5-DEPLOY-001, and mandatory reviews. Update only Phase 5 topology, compatibility, migration, media, flag, rollout, rollback and future AWS mapping docs/evidence; run approved local tests only. Do not write Docker/workflow/IaC/deployment code, provision AWS, add IAM/secret, apply staging/production migration, change payment behavior, or shift traffic. Separate evidence tiers, stop on unreviewed/destructive migration or incompatibility, and add evidence/06-completion.md.
```
