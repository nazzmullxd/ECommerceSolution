# Packet 11: Compensation, Recovery, And Reconciliation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 05-10 Approved |
| Layer Ownership | Owning modules compensate their state; application recovery coordinator uses approved interfaces |
| Risk Review | **Pre-code and post-test payment-security, commerce, database, and operations approval required** |
| Produces | Recovery/reconciliation policies, commands, failure tests, `evidence/11-completion.md` |

## Objective

Implement explicit, idempotent compensation and reconciliation for partial failures across Checkout, Inventory, Order, Payment, webhook settlement, and Outbox without double charging, overselling, deleting evidence, or inventing refunds.

## Read First And Prerequisites

- Approved Phase 0/1 audit/failure/operations contracts.
- Packet 01 state/transaction/idempotency rules and Packets 05-10 evidence.
- Phase 2 roadmap sections 6-15 and approval/open-question sections.

## In Scope

- Failure-mode matrix and owner/action for each scenario.
- `RequiresReview`/Unknown/reconciliation state use and safe internal commands.
- Reservation release/expiry, payment/order state convergence, provider-status query through Packet 08 adapter, outbox retry/dead-letter recovery.
- Idempotent replay/reconciliation, operator-safe evidence/log/audit, bounded automated recovery versus manual review.
- Failure injection at transaction/provider/worker boundaries and convergence tests.

## Out Of Scope

- Refund/void/chargeback implementation, fulfillment/shipment, customer compensation, real operator UI, production alerts/runbooks, direct DB edits, or automatic destructive correction.

## Likely File Areas To Inspect

- Core Checkout/Inventory/Orders/Payments/Outbox recovery commands/results/policies.
- Infrastructure transaction/provider-query/outbox/dead-letter implementations.
- Internal/admin-authorized recovery entry point only if approved; otherwise service/test harness.
- Failure-injection, reconciliation, audit, concurrency, idempotency tests.

## Architecture And Prohibited Dependencies

- Each module owns its invariant/transition; recovery coordinator invokes interfaces and never edits another module's tables directly.
- Core recovery policies have no EF/HTTP/provider SDK types.
- No distributed transaction, direct SQL repair, or catch-all status setter.
- Automated retries act only on idempotent operations with durable evidence.

## Failure And Compensation Matrix

At minimum define and implement/test:

| Scenario | Required Safe Outcome |
| --- | --- |
| Price/product changes before checkout | Recalculate/show new total or fail before payment; no stale trust. |
| Reservation creation partial/DB failure | Whole transaction rolls back; no active partial reservation/session. |
| Reservation expires before payment | Release once; checkout/order expired/incompatible; restart required. |
| Payment initiation timeout/unknown | Query deterministic provider; no blind second charge intent. |
| Provider success, local settlement transient failure | Provider retries or reconciliation replays same event; exactly one local settlement. |
| Verified success after reservation expiry | Preserve Payment/RequiresReview evidence; no unsafe fulfillment/false failure. |
| Stock confirmation failure after verified success | Payment/order RequiresReview; no double charge; manual resolution path. |
| Duplicate/delayed callback | One effect; duplicate success response; delayed compatibility/review rule. |
| Order created, notification fails | Order remains valid; outbox retry/dead-letter. |
| Outbox handler succeeds, mark fails | Redelivery produces one idempotent effect. |
| Migration/database outage | No partial commit; safe error/retry evidence; no manual silent mutation. |

## API Contract

No public recovery endpoint. If an internal/admin command is approved, require dedicated permission, reason, correlation, audit, idempotency, narrow allowed transitions, and no raw provider/PII response. Customer sees safe current status/support guidance only.

## Database Changes And Migrations

None expected. Any missing status/reference/reconciliation field indicates an owning-packet schema gap and returns to that packet's migration review.

## Security, Logging, And Failure Rules

- Never compensate by charging again, editing totals, inventing provider result, restoring expired reservation after payment without review, or deleting event/history.
- No raw SQL/operator table edits in normal procedure.
- Logs/audit use IDs, old/new safe statuses, action/reason/result/correlation; no card/provider secret/raw payload/address/full order.
- Recovery commands are idempotent, permissioned if exposed, and concurrency-safe.
- Unknown or contradictory external truth defaults to RequiresReview, not confident success/failure.

## Implementation Steps

1. Obtain pre-code approval for complete failure matrix, automation boundaries, reconciliation permissions, and manual-review states.
2. Map each scenario to owning module, trigger, idempotency key, transaction, retry limit, resulting state, audit, and operator evidence.
3. Implement narrow module compensation/reconciliation commands using existing transition services.
4. Implement provider-status reconciliation for Unknown/RequiresReview without refund/cancel/second charge.
5. Integrate reservation expiry, webhook replay, payment review, and outbox dead-letter recovery paths.
6. Add permission/audit entry point only if explicitly approved; otherwise keep internal/test-driven.
7. Add fault-injection seams that cannot activate in production accidentally.
8. Test every scenario, duplicate/concurrent recovery, crash points, and eventual convergence.
9. Verify no sensitive logs/history deletion, run checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Reservation | Partial failure rollback; expiry once; confirm/expiry race. |
| Payment | Initiation unknown; callback transient retry; success after expiry; stock confirmation failure. |
| Duplicate | Replayed callback/reconciliation/recovery command one effect. |
| Outbox | Retry/dead-letter; handler-success/mark-failure dedupe. |
| Concurrency | Two recovery workers/commands converge safely. |
| Authorization | Internal/admin path denied without permission/reason; audited. |
| Evidence | History/audit/provider references retained; no sensitive data. |
| Fault injection | Disabled outside test; each injected point leaves valid state. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "RequiresReview|Unknown|Compensat|Reconcile|Retry|DeadLetter|SetStatus|ExecuteSql" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

Inspect for unrestricted status setters/direct SQL/blind retry.

## Acceptance Criteria

- Every required failure scenario has one owner, deterministic safe outcome, audit/evidence, and tests.
- Recovery is idempotent/concurrency-safe and preserves provider/business history.
- Unknown/contradictory states require review rather than unsafe automation.
- No duplicate charge, oversell, double release/confirm/notification, or silent lost event occurs in tests.
- No refund/fulfillment/operator UI/direct DB edit/schema/later feature was added.
- Pre-code and post-test approvals/evidence exist.

## Security Stop Conditions

Stop on blind payment retry, direct status/table mutation, deleted evidence/history, automated fulfillment after expired stock, unpermissioned recovery, non-idempotent compensation, sensitive logs, production-enabled fault injection, or schema patch outside owner review.

## Rollback / Forward-Fix And Handoff

Do not roll back by deleting real-like payment/order/inventory/outbox history. Disable unsafe recovery entry points and forward-fix policies with versioned/audited corrections. Hand the failure matrix and convergence evidence to Packet 12 and Phase 5 runbook planning.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 11 from docs/implementation-guides/phase-2/11-compensation-recovery-and-reconciliation.md only after PHASE0/PHASE1 gates, Packets 05-10, and pre-code payment/security/commerce/database/operations approval. Implement only narrow idempotent module-owned recovery/reconciliation and test-only fault injection for the required failure matrix, preserving history and using RequiresReview for uncertainty. Add evidence/11-completion.md, run convergence/duplicate/concurrency checks, then obtain post-test approval. Stop on blind charges, direct status/SQL, deleted evidence, unsafe fulfillment, sensitive logs, unpermissioned or non-idempotent recovery.
```

