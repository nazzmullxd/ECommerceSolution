# Packet 08: Audit Service And Sensitive Transactions

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 04-07 Approved; decision `P1-AUDIT-001` |
| Layer Ownership | Core audit contract/policy; Infrastructure persistence/redaction; application services invoke it |
| Produces | Mandatory/redacted audit service and tests, `evidence/08-completion.md` |
| Human Review | Security reviewer, solution architect, database reviewer |

## Objective

Implement a structured, redacted audit service and explicit fail-closed/fail-open transaction rules for sensitive identity, session, bootstrap, role, permission, and profile actions.

## Read First And Prerequisites

- Approved Phase 0 audit/security/data contracts.
- Packet 03 logging/error behavior, Packet 04 AuditLog mapping, Packets 05-07 audit call sites.
- Phase 1 roadmap sections 5, 13, 16-19.

## In Scope

- Framework-neutral audit event/result contract.
- Allowlisted action names and bounded/redacted metadata.
- Actor/target/result/reason/correlation/time fields.
- Required audit catalog and sensitive-action transaction semantics.
- Infrastructure persistence and safe operational failure logging.
- Integration of audit calls into already implemented Phase 1 services.

## Out Of Scope

- Audit-read/export API/UI, analytics, CloudWatch/S3 export, immutable external ledger, full before/after PII snapshots, request-body capture, or later-phase business audit events.

## Likely File Areas To Inspect

- Core audit contracts/actions/policy.
- Infrastructure audit persistence/redaction/DI and DbContext transaction support.
- Identity/token/access services from Packets 05-07.
- API correlation/current-user abstraction only if needed; endpoints remain later.
- Audit/failure/redaction tests.

## Architecture And Prohibited Dependencies

- Core audit contract cannot depend on EF/logging providers/HTTP context.
- Infrastructure persists; application service supplies semantic actor/action/target/result.
- General debug logs do not substitute for audit records.
- Do not create a generic dictionary dump that accepts arbitrary sensitive objects.
- Current-user/correlation abstractions must not make Core depend on HTTP.

## Audit Contract

At minimum audit: registration, login success, repeated login failure/lockout, logout, refresh reuse, account disable/security change, role assignment/removal, permission change, super-admin bootstrap, denied admin action, and profile update (field names only).

| Action Class | Failure Rule |
| --- | --- |
| Privilege/bootstrap/security-state write | Business change and audit commit together or operation fails/rolls back. |
| Token reuse detection | Security revocation must proceed safely; audit failure generates high-severity operational signal without restoring compromised session. |
| Low-risk authentication/read telemetry | Best effort if approved; operational failure logged safely. |

Finalize these nuances in `P1-AUDIT-001`; do not use one blanket rule that weakens security.

## API And UI Contracts

No audit read endpoint/UI. Audit event creation is internal. Future admin audit access requires separate permission/data-minimization design.

## Security, Logging, And Failure Rules

- Never store passwords, tokens/hashes, cookies, auth headers, reset codes, signing keys, full addresses/profiles, sensitive request bodies, or full old/new values.
- Hash/truncate network/user-agent metadata only if approved and documented; do not assume it is anonymous.
- Audit writes are append-oriented; callers cannot update/delete history through normal services.
- Audit failure logging contains action ID/category and correlation, not failed sensitive payload.
- Correlation ID is trace metadata, not trust/authorization.

## Implementation Steps

1. Approve audit action catalog, metadata allowlists, retention class, and `P1-AUDIT-001` failure matrix.
2. Define Core audit event/result/interface and stable action constants.
3. Implement bounded redaction/serialization and Infrastructure persistence.
4. Define transaction coordination for sensitive writes and audit using approved DbContext boundary.
5. Integrate audit calls into Packets 05-07 services without duplicating events in controllers.
6. Add operational error logging for audit failure without recursive/sensitive logging.
7. Restrict normal update/delete access to audit history.
8. Add unit/integration/failure/redaction/concurrency tests.
9. Search persisted/logged metadata for prohibited fields, run checks, record evidence, request security review.

## Database Changes And Migrations

No expected schema change beyond Packet 04. Any new field/index requires Packet 04-style migration review. No deletion/retention job in Phase 1.

## API Contract

None.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Catalog | Every required action has stable name/classification/failure rule. |
| Persistence | Required fields/index behavior; append-oriented normal API. |
| Redaction | Prohibited secrets/PII absent from record and logs. |
| Transaction | Privilege write audit success; audit failure rolls back where required. |
| Token reuse | Session family remains revoked even if audit persistence fails, with safe signal. |
| Duplicate | Retry/idempotent service behavior does not create misleading duplicate business effects; audit semantics documented. |
| Architecture | Core has no EF/HTTP/log-provider dependency. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Password|Token|Authorization|Cookie|ConnectionString|Request.Body|MetadataJson" ECommerce.Core ECommerce.Infrastructure ECommerce.API
git diff --check
git diff --stat
git status --short
```

Inspect matches and test fixtures for actual sensitive values.

## Acceptance Criteria

- Audit action catalog and failure matrix are approved.
- Sensitive state changes have correct transaction/audit behavior.
- Token-reuse revocation cannot be undone by audit failure.
- Records/logs exclude prohibited data and metadata is bounded.
- Core remains infrastructure/HTTP-neutral.
- No audit read API, cloud export, retention job, or later feature was added.
- Evidence and security/database approval exist.

## Security Stop Conditions

Stop on raw sensitive metadata, request-body capture, privilege change without mandatory audit, audit failure that restores compromised sessions, unbounded metadata, HTTP/EF in Core, or unreviewed schema change.

## Rollback And Handoff

Preserve existing audit history. Revert only new service/integration behavior under reviewer direction; never delete audit records to make tests pass. Hand audited service contracts to API Packets 09-11.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 08 from docs/implementation-guides/phase-1/08-audit-service-and-sensitive-transactions.md only after PHASE0-GATE, Packets 04-07, and P1-AUDIT-001 approval. Implement only the framework-neutral audit contract, bounded redacted persistence, approved sensitive transaction/failure rules, and service integrations/tests. Add evidence/08-completion.md, run checks, preserve audit history, and stop on sensitive metadata, fail-open privilege changes, compromised-session restoration, schema, or dependency concerns. Do not add audit APIs.
```
