# Packet 07: Orders, Immutable Snapshots, History, And Ownership

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packet 06 Approved; decision `P2-ORDER-001` |
| Layer Ownership | Orders own Order/OrderItem/OrderStatusHistory and legal transitions |
| Risk Review | **Pre-code and post-test commerce, security, and database approval required** |
| Produces | Pending order creation/history/detail/migration/tests, `evidence/07-completion.md` |

## Objective

Implement idempotent pending-order creation from an approved checkout, immutable commercial/address snapshots, controlled status history, customer ownership/admin permission, and bounded order list/detail APIs.

## Read First And Prerequisites

- Approved Phase 0/1 identity/API/audit/data contracts.
- Packet 01 state/idempotency contracts and Packet 06 checkout evidence.
- Phase 2 roadmap sections 3-7, 10, 12-15, and 18.
- Approved admin permission evolution/change process.

## In Scope

- Order, OrderItem, OrderStatusHistory entities/invariants/mappings/migration.
- Unique order number and checkout-to-order relationship.
- Internal idempotent `CreatePendingOrderFromCheckout` application command.
- Immutable product/variant/SKU/name/quantity/Money/discount/tax/line totals and address snapshots.
- Legal status transition service/history/audit.
- `GET /api/v1/orders` and `GET /api/v1/orders/{orderId}` with customer ownership and approved admin permission.

## Out Of Scope

- Payment initiation/settlement, shipment/fulfillment, cancel/refund/return implementation beyond approved pre-payment state, admin order-management UI, invoice/receipt, or order edits after placement.

## Likely File Areas To Inspect

- Core Orders entities/value objects/commands/queries/services/transitions/events.
- Checkout/Catalog snapshot contracts.
- Infrastructure Order mappings/query/persistence/transactions/DI/migration.
- API order read endpoints/DTOs/OpenAPI and unit/integration/API/security tests.

## Architecture And Prohibited Dependencies

- Orders own historical business truth and transitions, not live Catalog/Inventory/Payment state.
- Create only from an approved own CheckoutSession; no client-supplied snapshots/totals/status/customer.
- Core has no EF/HTTP/provider types.
- API read endpoints use application queries and ownership/permission filters, not direct DbContext.

## Data And State Contract

| Area | Rule |
| --- | --- |
| Creation | One Order per CheckoutSession; idempotent replay returns same order. |
| Initial status | `PendingPayment`; no Paid state without verified settlement. |
| Items | Immutable copies of product/variant labels/SKU/quantity and all Money components. |
| Address | Link/copy approved checkout snapshots; later profile edits cannot change order. |
| History | Every transition records from/to/reason/time/actor/correlation. |
| Transition | Central policy only; illegal/repeated transition deterministic. |
| Number | Unique, non-secret, not authorization; public identifier remains opaque/unguessable enough per contract. |

## API Contract

| Method/Route | Authorization | Behavior |
| --- | --- | --- |
| `GET /api/v1/orders` | Customer | Own orders only; bounded newest-first pagination and minimal summary. |
| `GET /api/v1/orders/{orderId}` | Customer owner or approved admin permission | Immutable item/totals/status history and minimized address snapshot. |

Pending order creation is internal during checkout flow. If a public create route is later approved, it requires customer ownership and idempotency; do not expose it by default.

## Database Changes And Migrations

Create only Order/OrderItem/OrderStatusHistory relationships, snapshots, uniqueness, indexes, version, delete restrictions, and link to CheckoutSession/AddressSnapshot. Order/history/item deletion must not cascade from mutable catalog/profile data.

## Security, Logging, And Failure Rules

- Query ownership in data access; use privacy-safe 404 for another customer's order.
- Admin read requires explicit permission introduced through approved Phase 1 change process and is audited.
- Never bind entity/snapshot/status from client.
- Do not log full order/address/items/payment details; log order/customer/status/total category/correlation IDs.
- Creation/history/outbox intent must be transactionally consistent; Packet 10 adds durable outbox integration.
- Order-number collision retries are bounded and safe.

## Implementation Steps

1. Obtain pre-code approval for `P2-ORDER-001`, order numbering, transitions, admin permission, address visibility, transaction/idempotency.
2. Define Orders entities/invariants/transition policy/internal create/query contracts.
3. Add mappings/constraints/indexes/delete restrictions and reviewed migration.
4. Implement idempotent pending-order creation from persisted own checkout snapshots/totals/reservations.
5. Implement central transition/history service with legal/duplicate behavior.
6. Implement customer history/detail queries and approved admin read/audit path.
7. Add thin APIs/DTOs/pagination/Problem Details/OpenAPI.
8. Add unit, migration, snapshot immutability, duplicate/concurrency, ownership/admin, status/history, and PII tests.
9. Run checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Create | Valid checkout; wrong owner/status/expired; duplicate/concurrent one order; number collision. |
| Snapshot | Source copied exactly; later Product/Profile changes do not alter order; client cannot override. |
| State | Legal transitions/history; illegal/repeated safe conflict/idempotent outcome. |
| List/detail | Own only; pagination; another customer denied/hidden; admin permission allowed/audited. |
| Persistence | Unique checkout/order number, immutable/delete restrictions, migration fresh apply. |
| Logging | No full address/items/payment/PII. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "OrderStatus|OrderStatusHistory|OrderNumber|CustomerProfileId|Address|UnitPrice|TotalAmount|DbContext" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- Exactly one pending order is created per valid checkout with immutable snapshots.
- Status changes use central legal transitions and complete history.
- Customer ownership/admin permission and privacy-safe DTOs are tested.
- Migration/constraints/delete restrictions are reviewed.
- No paid/payment settlement/shipping/refund/admin UI/edit behavior was added.
- Pre-code and post-test approvals/evidence exist.

## Security Stop Conditions

Stop on client-provided snapshots/totals/status/owner, duplicate orders, mutable historical items/address, bypassed transition/history, BOLA/IDOR, admin read without permission/audit, unbounded list, unsafe cascade, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Never delete orders/history to undo logic after shared use. Before test-only use, reviewed migration rollback is possible; afterward forward-fix statuses/snapshots only through auditable correction design. Hand pending-order/Money/status/ownership contracts to Payment Packet 08 and Outbox Packet 10.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 07 from docs/implementation-guides/phase-2/07-orders-snapshots-history-and-ownership.md only after PHASE0/PHASE1 gates, Packet 06, P2-ORDER-001, and pre-code commerce/security/database approval. Implement only Order/OrderItem/History, scoped migration, idempotent pending creation from trusted checkout snapshots, legal transitions, owned/admin-permission read APIs, and tests. Add evidence/07-completion.md, run checks, then obtain post-test approval. Stop on client snapshots/status, duplicate/mutable history, BOLA/admin audit, cascade, transaction, or migration concerns.
```
