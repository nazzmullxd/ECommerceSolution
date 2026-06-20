# Packet 05: Inventory Reservations, Concurrency, And Expiry

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packet 01 Approved; decision `P2-INV-001` |
| Layer Ownership | Inventory owns stock/reservations/transactions; Core rules; Infrastructure persistence/worker |
| Risk Review | **Pre-code and post-test commerce, database, and security approval required** |
| Produces | Overselling-safe inventory reservation lifecycle, `evidence/05-completion.md` |

## Objective

Implement InventoryItem, append-oriented InventoryTransaction, and StockReservation with all-or-nothing multi-line reservation, optimistic concurrency/transactions, bounded retries, expiry/release/confirm behavior, and overselling tests.

## Read First And Prerequisites

- Approved Phase 0/1 data/database/audit/worker contracts.
- Packet 01 status/transaction/event contracts.
- Phase 2 roadmap sections 3, 5, 6, 8, 12-15, and 18.
- Approved local provider/DbContext transaction/concurrency behavior from Phase 1.

## In Scope

- InventoryItem, InventoryTransaction, StockReservation entities/invariants/mappings/migration.
- Reserve all cart/checkout lines atomically, release, confirm, expire, and query reservation result.
- Available/reserved quantity semantics, safety stock, version/concurrency token.
- Bounded optimistic retry and deterministic lock/update ordering.
- Local expiry worker/command with concurrency-safe leasing/batching/idempotency.
- Safe audit/log/event intent and comprehensive unit/integration/concurrency/failure tests.

## Out Of Scope

- Warehouses/transfers, procurement, stock admin UI, stock forecasting, distributed locks, cache-as-truth, customer inventory mutation API, payment/order decisions, or AWS workers/queues.

## Likely File Areas To Inspect

- Core Inventory entities/value objects/commands/results/services/events.
- Infrastructure Inventory EF mappings/repositories/transactions/worker/clock/DI and migration.
- Integration/concurrency/failure/worker tests.
- Audit/outbox contracts only; full outbox is Packet 10.

## Architecture And Prohibited Dependencies

- Inventory owns quantities/reservation lifecycle; Catalog owns product description; Checkout requests reserve; Payment does not mutate stock directly outside approved settlement orchestration.
- Core has no EF/hosted-service/provider types.
- Infrastructure transaction/retry logic follows approved database semantics; no process-local lock as correctness mechanism.
- Worker invokes idempotent Inventory application service; no duplicated stock arithmetic in worker.

## Data And Invariant Contract

| Area | Rule |
| --- | --- |
| InventoryItem | Unique ProductVariant; Available >= 0; Reserved >= 0; SafetyStock >= 0; versioned. |
| Reserve | Check `Available - requested >= SafetyStock` per approved policy; decrease Available, increase Reserved. |
| Release/expire | Active only; decrease Reserved, increase Available exactly once. |
| Confirm | Active only; decrease Reserved exactly once; Available remains already reduced. |
| Transaction | Append record for reserve/release/expire/confirm/adjust with reference/reason/time. |
| Batch | All requested lines succeed or none mutate. |
| Reservation | Unique active CheckoutSession+InventoryItem; expiry/status/version indexed. |

## API Contract

No customer endpoint for reserve/release/confirm. Core service contracts are internal and idempotent. Any inventory availability shown publicly is informational and not checkout truth.

## Database Changes And Migrations

Create only InventoryItem, InventoryTransaction, StockReservation schema/constraints/indexes/concurrency fields. Review provider-specific concurrency token strategy, check constraints, delete behavior, and expiry query index. No warehouse table.

## Security, Logging, And Failure Rules

- Inventory mutation requires internal authorized application path; customer cannot supply trusted Available/Reserved values.
- Log item/reservation/reference IDs, quantity delta, result, retry count, correlation; no full cart/customer/address/payment data.
- Concurrency retry is bounded and only repeats the entire safe transaction; exhaustion returns conflict/insufficient result.
- Worker claims small batches and tolerates concurrent instances without double release.
- Database failure rolls back quantity, transaction record, and reservation together.

## Implementation Steps

1. Obtain pre-code approval for quantity semantics, safety stock, duration, concurrency token, retry count, expiry batch/lease, transaction boundary.
2. Define Inventory entities/invariants/status transitions and service contracts in Core.
3. Add mappings/constraints/indexes and reviewed migration.
4. Implement all-or-nothing multi-line reserve with deterministic ordering and concurrency handling.
5. Implement idempotent release, expire, and confirm with InventoryTransaction records.
6. Implement expiry query/worker invoking the same service with safe batch/lease behavior.
7. Add audit/event intent hooks without implementing external publishing.
8. Add unit, migration, transaction, parallel checkout, expiry race, duplicate, and failure tests.
9. Run stress/concurrency repetitions locally, build/test/diff checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Reserve | Enough/exact/insufficient/safety stock; multi-line all-or-none. |
| Concurrency | Parallel last-unit requests allow only valid quantity; bounded retry/exhaustion. |
| Release | Active once; duplicate safe; confirmed/released/expired invalid/no double increment. |
| Confirm | Active once; duplicate safe; Reserved decreases; Available not restored. |
| Expiry | Due/not due; worker rerun; concurrent workers; payment/confirm race. |
| Transaction | Quantity/reservation/history commit/rollback together. |
| Migration | Fresh apply; unique/index/check/concurrency behavior. |
| Logging/security | No customer/payment payload; no public mutation endpoint. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "AvailableQuantity|ReservedQuantity|StockReservation|InventoryTransaction|Concurrency|Retry" ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- Overselling is prevented under tested parallel reservations.
- Reserve/release/confirm/expire invariants are atomic and idempotent.
- Worker cannot double release and retries are bounded.
- Migration/constraints/indexes are reviewed and reproducible.
- No public mutation API, warehouse, payment/order decision, cache truth, or paid service was added.
- Pre-code and post-test human approvals/evidence exist.

## Security Stop Conditions

Stop on negative stock, process lock correctness, partial multi-line reservation, unbounded retry, double release/confirm, worker race, client-trusted stock, transaction/history mismatch, provider-unsupported concurrency, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Back up local DB before migration. Never repair stock by deleting history. Before shared data, reviewed rollback may be possible; afterward use auditable forward adjustments/fixes. Hand reservation API/status/expiry/transaction contracts to Checkout Packet 06 and settlement Packet 09.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 05 from docs/implementation-guides/phase-2/05-inventory-reservations-and-expiry.md only after PHASE0/PHASE1 gates, Packet 01, P2-INV-001, and pre-code commerce/database/security approval. Implement only InventoryItem/Transaction/StockReservation, scoped migration, atomic all-line reserve, idempotent release/confirm/expire, bounded optimistic concurrency and safe expiry worker with tests. Add evidence/05-completion.md, run repeated concurrency checks, then obtain post-test approval. Stop on oversell, partial/double mutation, worker/retry/transaction/migration concerns.
```

