# Packet 02: Public Catalog Browse And Detail Reads

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packet 01 Approved; `P2-CATALOG-001` resolved |
| Layer Ownership | Catalog owns source data; Core read contracts; Infrastructure query; API mapping |
| Risk Review | Catalog owner and security/API reviewer |
| Produces | Trusted public product browse/detail reads, `evidence/02-completion.md` |

## Objective

Expose bounded public catalog list/detail reads that provide trusted active Product/ProductVariant/price data for customers and downstream cart/checkout services without moving catalog ownership into commerce modules.

## Read First And Prerequisites

- Approved Phase 0 domain/API/data contracts and Phase 1 API/security foundation.
- `P2-CATALOG-001` evidence identifying the catalog source model and owner.
- Phase 2 roadmap sections 3-5 and 15-18.
- Current repository catalog types/migrations/data, if any.

## In Scope

- Framework-neutral catalog read query/DTO contracts.
- Active/published product and active/sellable variant visibility.
- Trusted current price/currency read boundary.
- `GET /api/v1/products` pagination/filter/sort and `GET /api/v1/products/{productId}`.
- Allowlisted category/min/max price/basic text filters and sorts.
- Query projection/no-tracking/performance rules, minimal seed/test data if approved, API/security tests.

## Out Of Scope

- Catalog admin CRUD, images upload, product variants authoring, semantic search, recommendations, inventory quantities/reservations, discounts, cache implementation, or silently creating an unapproved catalog source schema.

## Likely File Areas To Inspect

- Approved Catalog Core entities/read interfaces/DTOs.
- Infrastructure Catalog query implementations and EF projections.
- API product endpoints/mappers/OpenAPI.
- Catalog read/API/query tests and approved seed fixtures.

## Architecture And Prohibited Dependencies

- Catalog remains source of Product/ProductVariant/public price visibility.
- Cart/Checkout consume read interfaces; they do not query catalog tables directly through shared DbContext from controllers.
- Core read contracts have no EF/HTTP types.
- Inventory availability is not returned as trusted stock unless an approved Inventory query supplies it.

## API Contract

| Method/Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/products` | Anonymous | Page 1/20, max 100; allowlisted filters/sorts; active/published only. |
| `GET /api/v1/products/{productId}` | Anonymous | Active/published product and active variants only; privacy-safe 404 otherwise. |

Responses include opaque IDs, name, public description/category, active variants/SKUs if public, trusted current Money, and public image metadata only if baseline exists. They exclude cost/internal notes/draft flags/stock source/audit/admin data.

## Database Changes And Migrations

No commerce migration. If no approved catalog schema exists, stop and resolve `P2-CATALOG-001`; do not create it casually in this packet. Minimal test seed data must be fake and idempotent.

## Security, Logging, And Failure Rules

- Allowlist filters/sorts; bound page size, search length, price ranges, and query complexity.
- EF parameterization/projection; no raw user-built SQL/order expression.
- Hide inactive/draft/deleted products/variants.
- Logs contain route/filter category/result count/duration/correlation, not full query PII or internal product data.
- Database failure uses Phase 1 safe errors; do not serve stale/unapproved data silently.

## Implementation Steps

1. Resolve `P2-CATALOG-001` with approved catalog owner/model/evidence.
2. Inspect existing model/query patterns; do not duplicate Product/ProductVariant.
3. Define Core browse/detail query contracts and safe DTO projections.
4. Implement Infrastructure no-tracking, bounded, allowlisted projections.
5. Implement list endpoint with pagination/filter/sort validation.
6. Implement detail endpoint with active/public visibility.
7. Add OpenAPI and fake deterministic test data as approved.
8. Add unit/query/API/security/performance-plan tests.
9. Run build/tests/query/log/scope checks, write evidence, request review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| List | Defaults/max; category/price/text filters; allowed/invalid sorts; empty results. |
| Visibility | Draft/inactive/deleted product hidden; inactive variant hidden. |
| Detail | Active found; hidden/missing safe 404; only public fields. |
| Money | Trusted amount/currency projection; mixed/invalid data rejected or flagged. |
| Query | Parameterized projection/no tracking; bounded query count/no obvious N+1. |
| Security | Oversized page/filter rejected; internal fields absent. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "FromSqlRaw|OrderBy\(|Include\(|PageSize|IsActive|IsPublished" ECommerce.Infrastructure ECommerce.API
git diff --check
git diff --stat
git status --short
```

Inspect query matches manually for allowlists and performance.

## Acceptance Criteria

- Catalog baseline/owner is approved rather than invented.
- Browse/detail APIs are bounded, public-only, and use trusted price reads.
- Draft/inactive/deleted records and internal fields are not exposed.
- Query/API/security tests pass without obvious N+1/unbounded behavior.
- No inventory/cart/admin/semantic-search/cache implementation or commerce migration was added.
- Evidence and catalog/security/API approval exist.

## Security Stop Conditions

Stop on missing catalog baseline, duplicate source model, internal/draft exposure, dynamic unsafe filter/sort, unbounded query, client-controlled price, direct EF controller access, or migration outside approved prerequisite.

## Rollback / Forward-Fix And Handoff

Revert this packet before consumers if rejected. After Cart/Checkout depend on the read contract, use additive/versioned forward fixes. Handoff trusted product/variant/price visibility contracts to Packets 03 and 06.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 02 from docs/implementation-guides/phase-2/02-catalog-browse-and-detail-reads.md only after PHASE0/PHASE1 gates, Packet 01, and P2-CATALOG-001 approval. Inspect the approved catalog baseline, implement only framework-neutral read contracts, bounded no-tracking projections, public list/detail /api/v1 endpoints, OpenAPI, and tests. Add evidence/02-completion.md, run checks, and stop on missing/duplicate catalog ownership, hidden-data exposure, unsafe/unbounded queries, client price, migration, or direct EF controller concerns.
```
