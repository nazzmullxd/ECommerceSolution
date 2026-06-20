# Packet 04: Wishlist Ownership And Experience

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packets 01-02 Approved; `P3-WISHLIST-001` |
| Layer Ownership | Wishlist owns each customer's private collection; reads Catalog projections |
| Risk Review | Pre-code and post-test identity, privacy, Catalog, API, and UX review |
| Produces | Authenticated wishlist vertical slice, `evidence/04-completion.md` |

## Objective

Deliver one private, authenticated-customer wishlist with deterministic duplicate handling and safe behavior when products or variants become unavailable.

## Read First And Prerequisites

- Approved Phase 1 current-user/ownership/Problem Details contracts and Phase 2 Catalog identifiers.
- Packets 01-02 and resolved `P3-WISHLIST-001` product-versus-variant decision.
- Phase 3 roadmap wishlist, authorization, failure, logging, testing, and UX sections.

## In Scope

- Wishlist and WishlistItem ownership, uniqueness, timestamps, concurrency, and persistence.
- Authenticated read/add/remove APIs and MVC/Razor product-detail plus wishlist experience.
- Product/variant summaries through a public Catalog read contract; unavailable entries remain visible but cannot be purchased.
- Idempotent duplicate add and remove behavior, bounded pagination if item count can exceed the approved limit.

## Out Of Scope

- Guest wishlist, sharing/public lists, multiple named lists, price-drop/restock delivery, bulk move-to-cart, personalized ranking, or Wishlist ownership of product/price/stock.

## Ownership And Read-Only Contracts

- **Owns:** Wishlist, WishlistItem, customer ID, selected product/optional variant ID, added timestamp, and item lifecycle.
- **Reads:** active/public Catalog title, slug, primary image, displayed price, and availability projection.
- **Must not own/mutate:** Product/Variant, price, Inventory, Cart, Order, Notification, or CustomerProfile data.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/me/wishlist` | Customer | Returns caller-owned items and current Catalog display projection. |
| `POST /api/v1/me/wishlist/items` | Customer | Adds product/approved variant idempotently; body contains identifiers only. |
| `DELETE /api/v1/me/wishlist/items/{itemId}` | Customer owner | Idempotent removal; never accepts another customer ID. |

Use current-user identity from the server, never request bodies. Follow Phase 0 response/Problem Details and conceal cross-owner identifiers consistently.

## UI Acceptance States

- Product detail: signed-out sign-in path, add, already saved, remove, loading, validation, forbidden/session-expired, conflict, dependency failure.
- Wishlist page: loading, empty, success, unavailable/disabled product, missing image, remove confirmation/undo only if approved, stale conflict, and Catalog temporarily unavailable.
- Controls are keyboard operable, announce saved state, preserve focus, and do not rely only on color/icons.

## Likely File Areas To Inspect

- Core Wishlist entities/services/repositories and current-user/Catalog read interfaces.
- Infrastructure EF mapping/query composition/migration.
- API DTOs/endpoints/OpenAPI and Web controllers/view models/views/styles/scripts.
- Unit, integration, API authorization/BOLA, UI, and accessibility tests.

## Architecture And Prohibited Dependencies

- Wishlist composes Catalog summaries through an interface; it does not copy authoritative price/stock.
- API/Web do not query DbContext, trust customer IDs, or expose internal deletion/visibility state.
- Add-to-cart remains a Phase 2 command and revalidates product, variant, price, and stock server-side.

## Database Changes And Migrations

Add/reconcile Wishlist and WishlistItem with customer ownership, product ID, optional variant ID, timestamps, version, and soft-delete only if the approved retention policy requires it. Enforce one wishlist per customer and one active item per approved product/variant identity. Index customer/item ordering and product references.

## Security, Logging, And Failure Rules

- Derive owner from authenticated subject; apply ownership in the database query, not after materialization.
- Apply normal API rate limits and anti-forgery for MVC writes. No anonymous persistent identifier is introduced.
- Log operation/result, wishlist/item/product IDs, and correlation ID; never email, full profile, cookies, tokens, or response contents.
- Missing/hidden products render unavailable; Catalog failure returns safe partial/failure behavior and does not delete the item.

## Implementation Steps

1. Approve product-versus-variant identity, duplicate semantics, item limit/pagination, unavailable behavior, exact routes, and UI states.
2. Define entities/invariants/application commands/queries and Catalog read DTO.
3. Add reviewed mapping, uniqueness/indexes, and migration.
4. Implement owner-scoped repository/query and idempotent add/remove.
5. Implement exact APIs and product-detail/wishlist MVC UI states.
6. Add ownership, concurrency, unavailable dependency, UI, and accessibility tests.
7. Run checks, record evidence, and obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Domain | One list/customer; approved identity; duplicate add; idempotent remove; item limit. |
| Ownership | Anonymous denied; customer A cannot read/remove B; no customer ID injection/BOLA. |
| Catalog | Active, hidden, deleted, variant mismatch, unavailable, read failure. |
| Commerce | Wishlist data cannot set Cart price/stock; add-to-cart revalidates. |
| API/UI | All loading/empty/success/error/conflict/session states and accessibility behavior. |
| Migration | Fresh apply; uniqueness/indexes; no Catalog duplication. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Wishlist|WishlistItem|CurrentUser|CustomerId|/api/v1/me/wishlist" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Authenticated customers can operate only their private wishlist; guest wishlist is absent.
- Duplicate behavior, product/variant identity, unavailable rendering, and ownership are deterministic and tested.
- Catalog is consumed read-only and Cart remains authoritative for purchasability/price/stock.
- Exact API/UI states, accessibility, privacy-safe logs, and migration tests pass.
- Completion evidence and post-test review exist.

## Security Stop Conditions

Stop for a client-supplied owner ID, repository query without owner predicate, public/shared wishlist leakage, hidden Catalog data leakage, copied authoritative price/stock, direct Wishlist-to-Inventory writes, guest tracking, missing anti-forgery/rate limit, or destructive migration.

## Rollback / Forward-Fix And Handoff

Disable UI/routes before schema removal. Preserve customer items and forward-fix uniqueness or projection failures; do not cascade-delete products. Hand the authenticated wishlist projection to Packet 11 and reserve future price/restock notifications for a separately approved Notification event contract.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 04 only after all prerequisite gates, Packets 01-02, P3-WISHLIST-001, and pre-code review. Implement one authenticated private wishlist, owner-scoped persistence, exact APIs, accessible MVC states, read-only Catalog composition, migration, and tests. Keep guest/shared lists and notification automation out of scope; never accept customer ownership from the client or copy authoritative price/stock. Add evidence/04-completion.md, run checks, and obtain post-test privacy/UX review.
```
