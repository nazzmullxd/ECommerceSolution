# Packet 07: Backup, Restore, RTO/RPO, And Drills

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packets 01 and 06 Design Approved; `P5-DB-001`, `P5-BACKUP-001`, `P5-PRIVACY-001` |
| Ownership | Database/media/module owners own recoverability; operations owns drill coordination |
| Manual Review | Mandatory backup/restore, RTO/RPO, retention, encryption, access, migration, accidental deletion and drill witness approval |
| Produces | Backup catalog, restore procedures/drills/evidence and `evidence/07-completion.md` |

## Objective

Define every recoverable asset, approved RTO/RPO and retention, backup integrity/access, isolated restore and application verification, accidental-deletion recovery, and future AWS backup mapping, then require witnessed local and staging restore evidence before launch.

## Read First And Prerequisites

- Packet 01 data/workload/classification/owner inventory and Packet 06 database/media/topology/migration plan.
- Approved retention/privacy/legal hold and encryption/key-management direction.
- Module invariants for Identity, Catalog/media, Inventory/reservations, Orders/Payments/webhooks, Outbox, Audit, Support, and AI derived data.

## In Scope

- Backup inventory for relational database, media/files, data-protection keys/required secret references, configuration templates, runbooks, release/migration metadata and audit evidence.
- RTO/RPO per critical asset/journey, frequency/type/retention/location/immutability/encryption/checksum/access/monitoring/owner and dependency order.
- Safe local backup/restore into a new isolated database/storage path with synthetic data; integrity/security/business verification and measured RTO/RPO.
- Staging restore rehearsal/point-in-time/migration/media/outbox/payment/audit verification plan, accidental deletion selective-recovery decision, future RDS automated backup/PITR/snapshot, S3 versioning/lifecycle/AWS Backup mapping.

## Out Of Scope

- Production backup/restore, destructive deletion of current workspace/database, AWS configuration, KMS/IAM key/policy, real customer data copy, cross-region DR, production failover, or claiming RTO/RPO from estimates only.

## Ownership And Operational Contracts

- **Owns:** backup catalog/schedule/retention/integrity/access, restore order/procedure/verification, drill/evidence and recovery decision matrix.
- **Reads:** asset schemas/versions/invariants and environment/topology/security contracts.
- **Must not own/mutate:** business records outside approved restore, production keys/secrets, AWS resources, legal retention, or incident/go-live authority.

Each asset entry records owner, classification, source, backup mechanism, frequency/RPO, restore dependency/order, RTO, retention, encryption/key owner, access, checksum/verification, location/separation, monitoring, last successful backup/restore drill, and secure deletion/legal hold interaction.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Approved asset/RTO/RPO/frequency/retention/access/encryption/restore/verification/AWS mapping and drill plan. |
| Local Verification Evidence | Actual synthetic backup plus isolated restore, checksums, schema/version/business invariants, measured times and witness. |
| Staging Evidence | Production-like backup/PITR/media restore to isolated staging recovery target, security/consistency/smoke and measured objectives. |
| Production Approval Evidence | Backup jobs/access/retention/monitoring/restore authority and successful recent staging drill accepted before go-live. |

## API And UI Contract

No public backup API/UI. Restore verification uses approved test/smoke surfaces with synthetic/dedicated identities. Operational status reveals only aggregate backup ID/time/state to authorized operators, never data contents, locations, credentials or encryption keys.

## Likely File Areas To Inspect

- DbContexts/migrations, storage abstraction/media, data-protection configuration, Outbox/workers, module tests/invariants, config templates/runbooks/docs.
- Backup outputs remain outside repository and must not be committed.

## Architecture And Prohibited Dependencies

- AI search documents/embeddings are rebuildable, but source knowledge/version/config/evaluation evidence must be recoverable.
- Database and media restore points must be coordinated enough to avoid references to missing media; document accepted inconsistency and repair.
- Payment/webhook/order/inventory/outbox/audit consistency is verified after restore; do not replay blindly.
- Backups are separated from primary failure domain and access, with least privilege/immutability in future design.

## Database Changes And Migrations

No migration. Restore verifies expected migration history and compatibility. Before risky future migration, create and verify the approved backup/snapshot and forward-fix plan; backup existence alone does not guarantee timely rollback.

## Security, Logging, Audit, And Failure Rules

- Backups inherit highest contained classification; encrypt and restrict access. Never store secrets in source-controlled backup/config evidence.
- Use synthetic data locally. Production data restoration to non-production requires explicit masking/privacy/security approval and is out of default scope.
- Log backup/restore IDs, asset/version, timestamps/duration/bytes/checksum/status/error code/witness, not contents/credentials/PII.
- Failed/partial/corrupt/missing/decryption/checksum/schema/media restore is a failed drill; preserve evidence and remediate before approval.

## Implementation Steps

1. Resolve asset owners, database engine, data-protection/key dependencies, RTO/RPO/retention/legal requirements.
2. Create backup catalog and restore dependency/order/business-verification matrix.
3. Write safe local procedure that backs up synthetic data/media, restores to a new isolated target, never overwrites source, and cleans up only reviewed temporary paths.
4. Define checksums, migration/schema, auth, catalog/media, inventory, order/payment, Outbox/audit, support and AI rebuild verification.
5. Execute witnessed local drill when implementation exists; measure actual backup/restore/recovery times and achieved RPO.
6. Define staging PITR/full/media/selective deletion/failed migration restore drills and evidence template.
7. Map future RDS/S3/AWS Backup controls and required IAM/KMS/cost/monitoring approvals without configuration.
8. Obtain owners' design/local review; staging/production remain pending.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Integrity | Valid/corrupt/incomplete/wrong version/checksum/decryption/space failure and retry. |
| Restore | New isolated target, dependency order, no source overwrite, schema/migration/config compatibility. |
| Business | Login, catalog/media, inventory/reservation, order/payment/webhook, outbox dedupe, audit, support; AI rebuild. |
| Accidental deletion | Stop writes, restore separate, compare/selective repair versus full restore, audit/approval. |
| Security | Access denied, no secrets/production data in lower tier/evidence, retention/legal hold/deletion behavior. |
| Objectives | Measured backup/restore/application recovery and achieved RPO compared with approved RTO/RPO. |

## Verification Commands

Commands depend on the approved database/storage and must be written after inspection. Always include:

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
git diff --check
git status --short
```

Record exact database/media backup/restore commands in protected evidence without credentials or private paths. Never execute an unreviewed destructive command.

## Acceptance Criteria

- Every critical asset has approved owner/classification/RTO/RPO/frequency/retention/integrity/access/restore/monitoring/deletion behavior.
- Local synthetic backup restores to an isolated target and passes schema/security/business consistency with measured witness evidence.
- Staging and production tiers remain pending until real drills/configuration exist; proposed 4h/1h targets are not claimed achieved.
- Future AWS mapping includes IAM/encryption/cost/monitoring/restore prerequisites but no resource.

## Manual Approval Gates

Manual approval and witnessed sign-off are mandatory for RTO/RPO/retention, backup access/encryption/key dependencies, restore commands/target, risky migration backup, staging/PITR/media/selective recovery drills, production restore authority, and go-live recency requirement.

## Stop Conditions

Stop for unknown asset/owner/key dependency, real production data in local/staging without approval, destructive overwrite/delete, credential/path in repo/evidence, unencrypted/unrestricted sensitive backup, failed/corrupt/partial restore, blind event/payment replay, unmeasured RTO/RPO claimed achieved, or design used as production backup proof.

## Rollback / Forward-Fix And Handoff

This packet does not alter data. During future drill, stop before source overwrite and restore only to approved isolated target. A failed drill blocks launch; fix backup/procedure/config, create a new backup, rerun and re-sign. Hand measured recovery dependencies to Packets 08, 10-12 and Packet 99.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 07 only after prior gates, Packets 01/06, P5-BACKUP-001/P5-PRIVACY-001, and mandatory review. Update backup/restore documentation and, only when approved, perform a synthetic local backup plus restore to a new isolated target using inspected provider commands. Never overwrite/delete source data, use production data/secrets, commit backup/private paths, provision AWS, or claim RTO/RPO without measurements. Verify schema and business invariants, record witnessed design/local evidence separately, leave staging/production pending, stop on any partial/corrupt/security failure, and add evidence/07-completion.md.
```
