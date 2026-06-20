# Packet 11: Data Protection, Access, Secrets, And Compliance Evidence

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packets 01, 03-04 and 07 Design Approved; `P5-LOG-001`, `P5-PRIVACY-001`, `P5-AWS-001` |
| Ownership | Privacy/security own policy; data/module owners implement; operations controls production access/secrets |
| Manual Review | Mandatory PII/retention/deletion/legal hold, audit, secrets/rotation, IAM/access, backup, payment, support, AI and compliance-owner review |
| Produces | Data/access/secret/control evidence package and `evidence/11-completion.md` |

## Objective

Define and later verify data minimization/masking/retention/deletion, production access review, audit protection, local and future secret handling/rotation, least-privilege IAM planning, and compliance evidence ownership without storing secrets or claiming certification.

## Read First And Prerequisites

- Packet 01 data/config/secret/owner/environment inventory, Packet 03 telemetry/audit, Packet 04 security, Packet 07 backup.
- Approved Phase 0 data classifications and Phase 1-4 entity/ownership/log/AI retention contracts.
- Applicable market/privacy/legal/accounting/payment record obligations from named owner; do not infer jurisdiction.

## In Scope

- Data inventory/flow/classification/purpose/legal or business basis/owner, field minimization/masking/encryption/access/export/retention/deletion/backup/search/cache/log/audit behavior.
- Required categories: identity/profile/address, cart/session, checkout snapshots, orders/items/history, payment/webhook metadata, inventory, uploads/media, wishlist/reviews/support/messages/notifications, audit/outbox, logs/traces, AI index/retrieval/conversation/feedback/evaluation, backups.
- Production human/service access roles, least privilege/separation of duties/JIT/time-bound/break-glass/review/revoke/audit; future IAM trust/action/resource/condition matrix only.
- Secret inventory by key name/purpose/service/environment/source/consumer/rotation/revocation/owner; local user-secrets/env and future Secrets Manager/IAM boundary.
- Retention/deletion/legal hold/account deletion/data request process, audit integrity/access/export, compliance evidence register/control owner/expiry.

## Out Of Scope

- Secret value, production credential/account/access, IAM policy/resource, KMS/Secrets Manager, compliance certification/legal advice, production deletion, customer data export, encryption implementation, real rotation, or copying production data.

## Ownership And Operational Contracts

- **Owns:** cross-system data/retention/access/secret/control matrices, review/revocation/rotation/deletion evidence process.
- **Reads:** module fields/purposes/ownership, environment/topology/backup/log/audit and legal owner decisions.
- **Must not own/mutate:** source data, legal obligation, Identity/IAM, secret stores, backups, audit records, or production access.

Least privilege matrix records actor/service, environment, purpose, resource/data class, allowed actions/conditions/duration, approver, authentication/MFA, audit, review/revoke and break-glass. No wildcard action/resource default.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Signed data/retention/deletion/access/secret/IAM/audit/compliance control matrices and owners. |
| Local Verification Evidence | Schema/DTO/log/config/repo scans, masking/ownership/deletion-selection/secret-validation tests using synthetic data. |
| Staging Evidence | Masked/synthetic data only, role/access/MFA/secrets injection/rotation rehearsal/audit/retention job and deletion/export dry-run evidence. |
| Production Approval Evidence | Access roster/review, service IAM/secrets/rotation, retention/deletion/backup/audit protection and privacy/security sign-off for exact environment. |

## API And UI Contract

- APIs/UI return minimum fields by purpose/permission; masking is not authorization. Exports, admin support context, audit/retrieval-log views and deletion requests have explicit permission, reason, limits, audit and safe async/failure behavior.
- Error/log/evidence must not disclose existence/content beyond approved policy.
- No real data subject request endpoint is invented here; document workflow and ownership before implementation.

## Likely File Areas To Inspect

- Entities/DTOs/mappings/config/options/logging/audit/storage/cache/search/AI/backups/admin/support exports and tests.
- Repository/history/attachments must be scanned for secrets safely; do not print values into evidence.

## Architecture And Prohibited Dependencies

- Data minimization occurs at source/DTO/query, not only UI masking.
- Derived copies/indexes/caches/logs/backups inherit source retention/classification; deletion propagates or is documented as retained/legal hold.
- Audit is append-oriented/tamper-evident within approved capability and access is audited; it cannot contain unnecessary payload.
- Applications use secret references/injected values; no secret in Core, source, docs, image, logs or artifact.

## Database Changes And Migrations

No migration. Retention/deletion fields/jobs/schema changes return to owning module with privacy/DB/migration review, backup/restore impact and dry-run. Hard delete never runs from this packet.

## Security, Logging, Audit, And Failure Rules

- Never record secret values, credentials, keys, tokens, cookies, auth headers, connection strings, private URLs, full PII/support/payment/AI content in docs/evidence.
- Secret missing/invalid fails startup/feature closed; rotation supports overlap where provider/protocol requires, revocation and rollback without logging values.
- Access denied/privileged view/export/role/IAM/secret/retention/deletion/legal hold/audit export changes are audited.
- Retention/deletion job failure stops destructive continuation, records safe counts/IDs and escalates; legal hold overrides deletion with audit.

## Implementation Steps

1. Resolve jurisdiction/record obligations/purpose/owners/retention/legal hold/data-request expectations.
2. Build field-level data inventory/flow/copy matrix across DB/media/cache/index/log/audit/backup/export.
3. Define minimization/masking/access/encryption classification and retention/delete propagation for every category.
4. Define human/service access, separation, JIT/break-glass/MFA/review/revoke/audit and future IAM matrix.
5. Define secret inventory/source/consumer/rotation/revoke/overlap/failure/evidence with names only.
6. Define compliance control/evidence register and ownership/refresh/exception process without certification claim.
7. Run local schema/DTO/log/repo/config/ownership/masking/deletion-selection tests with synthetic data; obtain manual reviews.

## Test And Review Matrix

| Review | Required Cases |
| --- | --- |
| Minimization | Public/customer/support/admin/audit/report/AI DTOs expose only purpose fields; BOLA negatives. |
| Copies | Cache/index/log/audit/backup/export inherit classification/retention/delete behavior; stale copies removed/rebuilt. |
| Retention/delete | Eligible/not eligible/legal hold/dependency/partial failure/dry-run/idempotent retry and audit. |
| Access | Least privilege, role removal, expired/JIT/break-glass, MFA, service separation, denied access/audit. |
| Secrets | Missing/invalid/rotation overlap/revoke/rollback; no value in source/log/error/artifact/evidence. |
| Compliance evidence | Control owner/source/version/date/expiry/exception; no unsupported certification claim. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Password|Secret|ApiKey|Token|ConnectionString|Authorization|Cookie|Address|Phone|Email|Retention|DeletedAt|Audit" ECommerce.API ECommerce.Web ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

Do not paste matching secret-like values into evidence. Use safe scanner summaries and rotate any genuinely exposed value through the approved owner process.

## Acceptance Criteria

- Every data field/copy has purpose/classification/owner/access/retention/delete/backup/log/export behavior and no unresolved private-data path.
- Human/service access and future IAM are least-privilege, reviewed, revocable, MFA/audited and free of wildcard defaults.
- Secret lifecycle uses names/references only, fails safely and has reviewed rotation/revocation/rollback.
- Local scans/tests pass; staging/production access/secrets/retention evidence remain pending until actual controls exist.
- No certification/legal claim, secret/value, cloud policy/resource or production data action is added.

## Manual Approval Gates

Manual approval is mandatory for jurisdiction/retention/deletion/legal hold, production access/JIT/break-glass/MFA, audit access/retention/export, secrets source/rotation/revocation, IAM trust/actions/resources, encryption/key dependency, production data masking/export and compliance evidence acceptance.

## Stop Conditions

Stop for unknown legal/privacy owner, secret value/private URL in source/docs/log/evidence, wildcard/broad access, production data in lower environment, untracked derived copy, hard delete without dry-run/legal hold/backup/audit, audit tampering/unrestricted access, unreviewed IAM/rotation, unsupported compliance claim, or design marked as implemented control.

## Rollback / Forward-Fix And Handoff

This packet changes no production data/access/secret. Future access/retention/rotation rollout keeps emergency revoke/disable and audited rollback; destructive deletion cannot be rolled back except restore and requires stronger approval. Failed control blocks launch and returns to owner. Hand accepted controls/risks to Packet 12/90/99.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 11 only after prior gates, Packets 01/03/04/07, P5-PRIVACY-001/P5-LOG-001, and mandatory privacy/security review. Update only Phase 5 data/retention/deletion/access/secret/future-IAM/audit/compliance matrices and approved synthetic local scans/tests. Never include secret values/private URLs/customer data, create IAM/Secrets Manager/KMS/cloud resources, rotate real secrets, grant production access, execute deletion/export, or claim certification. Separate evidence tiers, leave staging/production pending, stop on unknown obligation/broad access/untracked copy/destructive ambiguity, and add evidence/11-completion.md.
```
