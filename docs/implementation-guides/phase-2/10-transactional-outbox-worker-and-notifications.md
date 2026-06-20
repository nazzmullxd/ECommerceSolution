# Packet 10: Transactional Outbox, Local Worker, And Notifications

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 07-09 Approved; decision `P2-OUTBOX-001` |
| Layer Ownership | Operations owns Outbox storage/dispatch; producer modules own event facts; consumers own idempotent effects |
| Risk Review | **Pre-code and post-test commerce, database, security, and operations approval required** |
| Produces | Transactional outbox/local worker/mock notification lifecycle, `evidence/10-completion.md` |

## Objective

Implement durable outbox writes in the same business transaction, a concurrency-safe local worker with retries/backoff/lease/dead-letter behavior, idempotent handlers, and mock notification handoff without requiring SQS/EventBridge or real email.

## Read First And Prerequisites

- Approved Phase 0/1 audit/logging/worker/database contracts.
- Packet 01 event/idempotency contract and Packets 07-09 producer events/transactions.
- Phase 2 roadmap sections 3, 6, 9, 11-15, and 18.

## In Scope

- OutboxMessage entity/status/mapping/migration and minimal versioned payload envelope.
- Producer integration for approved Order/Payment/Inventory state changes in same DB transaction.
- Local background worker polling/claiming/leasing/batching, bounded retry/backoff, stuck-processing recovery, dead-letter.
- Idempotent handler/receipt strategy and mock notification handler.
- Correlation/event IDs, safe payload schema, operational logs/metrics-ready fields, failure/concurrency tests.

## Out Of Scope

- SQS/EventBridge, real email/SMS/push, admin dead-letter UI, production scheduler/alerts, analytics, arbitrary reflection event dispatch, or including sensitive snapshots in payloads.

## Likely File Areas To Inspect

- Core integration event/envelope/handler contracts.
- Infrastructure Outbox mappings/store/worker/clock/backoff/DI/migration and mock notification handler.
- Producer transactions in Orders/Payments/Inventory.
- Worker/idempotency/failure/concurrency/redaction tests.

## Architecture And Prohibited Dependencies

- Domain/application producers create minimal event facts; Infrastructure serializes/persists/dispatches.
- Business transaction and outbox insert share approved DbContext transaction.
- Worker calls idempotent handlers; it does not decide Order/Payment/Inventory state.
- Core has no hosted-service, EF, JSON serializer provider, SQS/EventBridge, or email SDK dependency.
- Outbox is durable publication source, not business source of truth.

## Outbox And Handler Contract

| Area | Rule |
| --- | --- |
| Identity | Globally unique message/event ID; versioned event type; aggregate ID; occurred time; correlation/causation as approved. |
| Payload | Allowlisted minimal IDs/status/categories; no address, card, secret, token, raw provider payload, or unnecessary PII. |
| Write | Same transaction as state change; rollback together. |
| Claim | Atomic claim/lease with owner/expiry; concurrent workers cannot both process as active owner. |
| Retry | Bounded backoff (jitter if approved), next-attempt time, retry count, safe error category. |
| Dead letter | Terminal after limit; no infinite hot loop; manual review evidence. |
| Handler | At-least-once delivery expected; idempotent effect/receipt keyed by message/event ID. |

## API Contract

No public endpoint. No customer/admin ability to replay or inspect payloads in Phase 2. Notification effect is local mock/evidence only.

## Database Changes And Migrations

Create only OutboxMessage plus an approved handler receipt/deduplication table if required. Include status/available/lease/retry/aggregate/correlation indexes and bounded error/payload fields. Review payload storage classification.

## Security, Logging, And Failure Rules

- Never serialize full entity/request/provider payload by default.
- Logs use message/event/aggregate IDs, type, attempt/result/duration/correlation; safe truncated error category only.
- Handler success then mark failure can cause redelivery; idempotent consumer prevents duplicate effect.
- Worker crash/lease expiry permits recovery without concurrent double claim.
- Poison messages dead-letter; one bad message does not block unrelated batch.
- Notification failure never rolls back valid Order/Payment after transaction committed.

## Implementation Steps

1. Obtain pre-code approval for event envelope/payload allowlists, `P2-OUTBOX-001`, lease/batch/retry/backoff/dead-letter/dedupe behavior.
2. Define framework-neutral event/handler contracts and versioned Phase 2 events.
3. Add Outbox mapping/constraints/indexes and reviewed migration.
4. Integrate outbox inserts into approved Order/Payment/Inventory transactions.
5. Implement atomic polling/claim/lease/batch and stuck-processing recovery.
6. Implement bounded retry/dead-letter and safe operational logging.
7. Implement idempotent mock notification handler/receipt.
8. Add transaction, duplicate, crash, lease, retry, poison, concurrency, redaction, migration tests.
9. Document future adapter boundary to SQS/EventBridge without adding packages/services.
10. Run checks/failure injection, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Transaction | Business state+outbox commit/rollback together at each producer. |
| Claim | Two workers; one claim; lease expiry recovery; batch limits. |
| Delivery | Success; handler failure; retry schedule; max -> dead-letter; poison isolation. |
| Idempotency | Redelivery after handler success/mark failure produces one notification/effect. |
| Crash | Before/after claim/handler/mark; eventual safe convergence. |
| Payload | Required metadata/version; prohibited sensitive fields absent/bounded. |
| Migration | Indexes/constraints/fresh apply. |
| No cloud | No AWS/email SDK/service/credential requirement. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Outbox|PayloadJson|RetryCount|Lease|DeadLetter|BackgroundService|SQS|EventBridge|Email" ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

Inspect payload/logging and cloud/provider dependency boundaries.

## Acceptance Criteria

- Approved producers persist outbox in same business transaction.
- Worker claim/lease/retry/dead-letter and handler idempotency survive concurrency/crash tests.
- Payload/logs exclude sensitive data and are versioned/bounded.
- Notification failure does not invalidate committed commerce state.
- Migration is scoped/reviewed; no paid cloud/real notification/UI added.
- Pre-code and post-test commerce/database/security/operations approvals/evidence exist.

## Security Stop Conditions

Stop on non-transactional producer write, full entity/provider/address/payment payload, unbounded errors, duplicate handler effect, concurrent double claim, infinite retry/hot loop, poison batch blockage, business decisions in worker, cloud SDK/credential, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Do not delete pending/dead-letter messages or receipts to hide failures. Before shared use, reviewed migration rollback may be possible; afterward preserve messages and forward-fix handlers/envelopes with versioning. Handoff failure/dead-letter/duplicate scenarios to Packet 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 10 from docs/implementation-guides/phase-2/10-transactional-outbox-worker-and-notifications.md only after PHASE0/PHASE1 gates, Packets 07-09, P2-OUTBOX-001, and pre-code commerce/database/security/operations approval. Implement only versioned minimal OutboxMessage, same-transaction producer writes, concurrency-safe local claim/lease/retry/dead-letter worker, idempotent mock notification handler and tests. Add evidence/10-completion.md, run crash/duplicate/failure checks, then obtain post-test approval. Stop on non-atomic writes, sensitive payloads, duplicate effects, worker/poison/retry issues, cloud dependencies, or migration concerns.
```
