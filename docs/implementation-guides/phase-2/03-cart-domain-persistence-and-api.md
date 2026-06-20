# Packet 03: Cart Domain, Persistence, And API

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 01-02 Approved |
| Layer Ownership | Cart owns Cart/CartItem; Core rules/services; Infrastructure persistence; API mapping |
| Risk Review | Commerce architect before migration; security/API reviewer after tests |
| Produces | Authenticated cart behavior/API/migration/tests, `evidence/03-completion.md` |

## Objective

Implement the Cart and CartItem aggregate, persistence, authenticated ownership, concurrency, expiry/status behavior, and bounded cart APIs while treating cart contents as customer intent rather than trusted price or stock truth.

## Read First And Prerequisites

- Approved Phase 0/1 contracts, Packet 01 commerce contracts, Packet 02 catalog reads.
- Phase 2 roadmap sections 3-5, 7, 12-15.
- Phase 1 current-user/auth/audit/API/migration evidence.

## In Scope

- Cart/CartItem entities/invariants/status/version/timestamps.
- One active cart per authenticated customer, add/combine/update/remove/read behavior.
- Product/variant active validation through Catalog read contract.
- Configurable max distinct items and per-line quantity.
- Optimistic concurrency and repeat-safe commands.
- `GET /api/v1/cart`, `POST /api/v1/cart/items`, `PATCH /api/v1/cart/items/{cartItemId}`, `DELETE /api/v1/cart/items/{cartItemId}` for authenticated customers.
- EF mappings/migration, audit/log rules, unit/integration/API/security tests.

## Out Of Scope

- Guest cookie/key and merge (Packet 04), inventory reservation, checkout, trusted totals, discounts, orders, payments, saved cart/wishlist, promotions, or cart analytics.

## Likely File Areas To Inspect

- Core Cart entities/value objects/commands/queries/services/contracts.
- Infrastructure Cart mappings/repositories/query/transaction/DI and migration.
- API cart endpoints/DTOs/mappers/OpenAPI.
- Unit/integration/API/security/concurrency tests.

## Architecture And Prohibited Dependencies

- Cart owns intent and quantities, not Product price/visibility source or Inventory stock.
- Core calls Catalog abstraction; no EF/HTTP/current-user framework types.
- API derives customer ID from authenticated subject; request never chooses owner/cart ID.
- Controllers do not access DbContext/Catalog tables directly.
- Do not create a cross-module generic repository or god commerce service.

## Data Contract And Invariants

| Area | Rule |
| --- | --- |
| Ownership | Exactly one owner mode; authenticated behavior uses CustomerProfileId. Guest mode fields may be mapped for Packet 04 but unused here. |
| Active cart | At most one active cart per customer with DB constraint/transaction-safe creation. |
| Item | Unique active Cart+Variant; add same variant combines quantity within limit. |
| Quantity | Positive and bounded; zero uses remove endpoint/command, not ambiguous update. |
| Status | Active permits writes; Merged/CheckedOut/Expired/Abandoned reject mutation. |
| Version | Concurrency token on mutable cart/item or aggregate-level approved strategy. |
| Price/stock | No trusted final price, discount, shipping, tax, total, or stock availability stored from client. |

## API Contract

| Method/Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/cart` | Customer | Return/create own active cart and current catalog display summary. |
| `POST /api/v1/cart/items` | Customer | Add/combine active variant and bounded quantity. |
| `PATCH /api/v1/cart/items/{cartItemId}` | Customer owner | Set bounded positive quantity with concurrency/conflict behavior. |
| `DELETE /api/v1/cart/items/{cartItemId}` | Customer owner | Remove own item; repeat behavior explicitly defined. |

Responses may show current informational price from Catalog but label/recalculate at checkout. Do not accept price/stock/owner fields.

## Database Changes And Migrations

Create only Cart/CartItem schema, ownership/status/expiry/timestamp/version fields, FK/index/unique constraints. If including guest-key column for the approved aggregate shape, document why; Packet 04 owns behavior. Review filtered/partial unique-index portability for the approved provider.

## Security, Logging, And Failure Rules

- Ownership is enforced in query/update predicates, not after broad load.
- Use privacy-safe 404 for another customer's item when contract requires.
- Validate product/variant active status but do not disclose internal draft data.
- Log cart/customer/item IDs, action/result/correlation and safe counts; no full cart body/profile/token.
- Database/concurrency failure returns safe 409/500 without partial mutation.
- Catalog unavailable fails safely; do not add unvalidated item.

## Implementation Steps

1. Approve aggregate/invariants/limits/concurrency/repeat behavior.
2. Define framework-neutral Cart/CartItem model and service contracts.
3. Add EF mappings/constraints/indexes and reviewed migration.
4. Implement transaction-safe get-or-create active customer cart.
5. Implement add/combine/update/remove with Catalog validation and ownership predicates.
6. Implement thin APIs/DTOs/Problem Details/OpenAPI.
7. Add logs/audit where ownership-sensitive and correlation propagation.
8. Add unit, mapping, migration, API, ownership, duplicate, and concurrency tests.
9. Run build/tests/migration/diff checks, create evidence, obtain reviews.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Get/create | Existing active; first create; concurrent create leaves one active. |
| Add | Active variant; inactive/missing; duplicate combine; max line/items; client price ignored. |
| Update/remove | Valid; zero/negative/over-limit; missing; another owner; repeat remove policy. |
| Status | Non-active cart rejects writes. |
| Concurrency | Stale version conflict without lost update. |
| Persistence | Constraints/indexes/migration fresh apply. |
| API/security | 401 anonymous; ownership-safe 404/403; no internal/client-trusted fields. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Price|Discount|Shipping|Tax|Stock|CustomerProfileId|DbContext" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect matches for trusted-value/ownership violations.

## Acceptance Criteria

- Authenticated cart CRUD/combine is owned, bounded, concurrency-safe, and tested.
- Cart does not become trusted price/stock source.
- Catalog validation and one-active-cart constraint are enforced.
- Migration is scoped/reviewed/reproducible.
- No guest/checkout/inventory/order/payment/later feature was added.
- Evidence and pre/post commerce/security/API/database approval exist.

## Security Stop Conditions

Stop on client owner/cart selection, cross-user access, client-trusted price/stock, unbounded cart, lost update, duplicate active carts, direct EF controller access, unreviewed migration, or catalog ownership duplication.

## Rollback / Forward-Fix And Handoff

Back up local DB before migration. Before shared use, rollback reviewed migration if safe; after dependent data exists, prefer forward fix. Hand aggregate/API/ownership/version contracts to Packet 04 and checkout Packet 06.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 03 from docs/implementation-guides/phase-2/03-cart-domain-persistence-and-api.md only after PHASE0/PHASE1 gates and Packets 01-02 approval. Obtain pre-code review, inspect repository, implement only authenticated Cart/CartItem aggregate, scoped migration, Catalog-validated owned/concurrency-safe APIs and tests. Never trust/store client price/stock/owner. Add evidence/03-completion.md, run checks, then obtain post-test review. Stop on ownership, bounds, concurrency, migration, catalog, or controller-layer concerns.
```

