# Packet 05: Multi-Warehouse Inventory, Allocation, And Recovery

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packet 01 Approved; stable Phase 2 Inventory/Checkout/Order; `P6-WH-001` |
| Decision Type | Planning first; future Inventory-module extension in monolith only |
| Manual Review | Inventory/warehouse/fulfillment, commerce/order/payment, DB/concurrency, security/audit, operations/recovery, finance/cost and UX review |
| Produces | Warehouse inventory/allocation/transfer/recovery plan and `evidence/05-completion.md` |

## Objective

Extend the Inventory design to warehouse-specific stock, allocation, reservations, transfers and fulfillment while preserving transactional overselling protection and explicitly bounding eventual consistency to derived views/events.

## Read First And Prerequisites

- Approved InventoryItem/Transaction/Reservation concurrency/expiry/settlement/compensation and Checkout/Order snapshots.
- Actual warehouse count/locations/ownership/cutoffs/capabilities/shipping/cost/legal/residency and operational processes.
- Packet 01 measured business need and module/data/on-call owner.

## In Scope

- Warehouse, WarehouseStockItem, WarehouseReservation, Transfer/TransferLine, AllocationRule/Decision, FulfillmentRoute and inventory transaction/source concepts.
- Per-warehouse available/reserved/on-hand/safety/damaged/in-transit semantics, transactions/concurrency, allocation candidates/tie-break/split/no-split, reservation expiry/release/confirm/reallocate.
- Transfer request/approve/ship/receive/cancel/reconcile, loss/damage/partial receive and audit.
- Checkout/Order/Fulfillment snapshots/events, eventual-consistency boundaries, reconciliation, recovery, phased rollout from current stock model.

## Out Of Scope

- Microservice, distributed reservation, active-active stock writes, external WMS/carrier, geolocation profiling, unapproved split shipment/cost promise, direct admin stock edit, eventual final reservation, AWS service.

## Ownership And Boundaries

- **Owns:** warehouse metadata/capability, stock rows, reservations, allocation, transfers, inventory ledger/transactions and reconciliation.
- **Reads:** ProductVariant identity, Checkout demand, Order/payment result and approved shipping inputs.
- **Must not own/mutate:** Catalog description/price, Payment truth, Order status, customer address beyond minimum allocation input or reporting truth.
- Inventory remains one monolith module and single authoritative write boundary initially.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable single-stock reservation/settlement/concurrency/recovery and warehouse business owner. |
| Measured Need Evidence | Warehouse count/order distribution/allocation/stock contention/fulfillment/cost target. |
| Design Evidence | Stock semantics/allocation/transfer/transactions/events/security/migration/test/rollout/recovery. |
| Monolith-First Evidence | Future concurrency/oversell/reconciliation/warehouse canary and production measurements. |
| Extraction/Cloud Evidence | Not applicable; remain monolith until Packet 10 proves otherwise. |

## Data And Contract Design

Define invariant equations per warehouse and variant: on-hand, available, reserved, unavailable/damaged and in-transit cannot be double-counted; every quantity change has immutable InventoryTransaction/reference/idempotency/audit. Reservation binds warehouse/checkout line/version/expiry. Allocation decision stores rule/version/candidates/result/reason snapshot.

Define transfer source decrement/reserve and destination in-transit/receive semantics to avoid stock existing in both warehouses. Reconciliation compares ledger and projections.

## API And UI Contract

- Checkout requests quantity/variant and receives reservation ID/warehouse/expiry or safe failure; client cannot select stock truth unless an approved fulfillment choice exists.
- Admin warehouse/stock/transfer/allocation commands require granular permissions, reason, expected version, idempotency and audit; no generic quantity patch.
- Customer sees approved fulfillment promise, not internal stock/cost/warehouse details unless business chooses pickup/location.

## Likely File Areas To Inspect

- Core Inventory/Checkout/Order/Payment contracts, Infrastructure transactions/mappings/workers, API/Web inventory/admin/checkout, Tests concurrency/failure.
- Planning only now.

## Architecture And Prohibited Dependencies

- Final reservation/allocation is transactionally consistent within Inventory database boundary; events/reporting may lag.
- Do not use distributed lock/cache/read model as stock truth.
- Allocation strategy is versioned/typed and deterministic for same snapshot; external cost/geo input must be bounded/authorized.
- Checkout cannot reserve across separate services in this phase.

## Database Changes And Migrations

No migration now. Future plan must safely migrate current InventoryItem to default warehouse without double availability, with unique warehouse+variant, concurrency, ledger/reference/idempotency/index constraints, backfill verification, maintenance/feature flag, backup and rollback/forward-fix.

## Security, Logging, Audit, And Failure Rules

- Protect warehouse/internal stock/cost/address data and prevent BOLA/privilege misuse/negative stock/replay/concurrent last-stock/transfer fraud.
- Log warehouse/stock/reservation/transfer/reference IDs, quantity delta/status/rule/reason code/correlation; not full address/PII.
- Audit adjustments/transfers/allocation rule/warehouse/status/manual release/reconcile.
- Payment success after reservation expiry, partial transfer/receive, worker outage and reconciliation drift enter explicit review/compensation, never silent oversell.

## Planning Or Implementation Steps

1. Resolve stock/warehouse/fulfillment/allocation/split/cost/address/cutoff/transfer/recovery semantics.
2. Define entities/ledger/invariants/lifecycles/concurrency/idempotency and authorization/audit.
3. Define deterministic allocation and reservation/reallocation/settlement transaction boundaries.
4. Define transfer state machine, in-transit accounting and reconciliation.
5. Define Checkout/Order/Payment/Fulfillment API/event/snapshot/failure contracts.
6. Design default-warehouse migration, shadow comparison, canary warehouse/variants, rollback and tests.
7. Obtain planning approval before separate bounded monolith implementation.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Invariants | On-hand/available/reserved/damaged/in-transit equations and immutable ledger. |
| Concurrency | Same last item, multiple warehouses, retries, expiry/confirm/release/reallocate, no oversell. |
| Allocation | Priority/distance/cost/capability/ties/split/no-split/missing input/version determinism. |
| Transfers | Approve/ship/partial receive/loss/damage/cancel/retry/concurrent adjustment/reconcile. |
| Commerce | Price/checkout/order/payment success/failure/delayed callback/reservation expiry/fulfillment snapshot. |
| Migration/rollout | Default warehouse backfill, no double stock, shadow/canary/flag off/rollback/forward-fix. |

## Verification Commands

```powershell
rg -n "InventoryItem|InventoryTransaction|StockReservation|Checkout|Payment|Order|Concurrency|Warehouse|Transfer" docs/roadmap docs/implementation-guides/phase-2 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Warehouse stock/reservation/transfer/allocation semantics and invariants prevent overselling transactionally.
- Eventual consistency is limited to reports/integrations, never final stock reservation.
- Checkout/Order/Payment/fulfillment/recovery and default-warehouse migration are safe/idempotent/audited/testable.
- Design remains in Inventory monolith module with no service/cloud/multi-region writes or code now.

## Manual Approval Gates

Manual approval is mandatory for warehouse/stock semantics, allocation/split/fulfillment promise, address/privacy, transfers/adjustments, concurrency/transactions, payment/expiry compensation, migration/backfill, canary/rollback and any extraction review.

## Stop Conditions

Stop for unstable single-stock baseline, unknown warehouse process/owner, cache/eventual final stock, distributed reservation, negative/double stock ambiguity, client quantity/allocation truth, direct generic stock patch, missing payment compensation/reconciliation, service/cloud/multi-region or implementation without approval.

## Rollback / Forward-Fix And Handoff

Planning-only. Future rollout uses default warehouse and shadow allocation first, canaries scoped variants/orders, and disables new allocation on defect while preserving reservations/ledger. Never collapse warehouse stock without reconciliation. Hand events to Packet 08, reports to 06, access to 07 and risks to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 05 as planning only after prior gates, Packet 01 and P6-WH-001. Update only warehouse stock/reservation/allocation/transfer/fulfillment/concurrency/ledger/reconciliation/API/event/migration/test/canary/rollback design and evidence. Do not write code/migrations, use cache/events as stock truth, distribute reservations, trust client stock/allocation, expose private warehouse/address data, create services/cloud/multi-region, or self-approve. Keep Inventory authoritative in the monolith and preserve oversell prevention. Add evidence/05-completion.md with commerce/DB/operations review.
```
