# Packet 02: Product Variants, Attributes, And Detail Experience

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packet 01 Approved; `P3-CATALOG-001`, `P3-VARIANT-001` |
| Layer Ownership | Catalog/Product owns variants/options/attributes/price presentation; reads Inventory availability |
| Risk Review | Pre-code and post-test Catalog, commerce, security, UX, and database review |
| Produces | Variant/attribute admin and public detail behavior, `evidence/02-completion.md` |

## Objective

Refine the single approved Catalog model for sellable ProductVariant combinations and public ProductAttribute facts, connect variants to Phase 2 Inventory/Cart/Checkout contracts, and deliver public/admin API plus product-detail UI behavior without duplicating price or stock truth.

## Read First And Prerequisites

- Approved Phase 0-2 Catalog/ProductVariant/Inventory/Cart/Checkout contracts/evidence.
- Packet 01 permissions/status/UI conventions.
- Phase 3 roadmap sections 3-8, 12, 15-21, and 24.

## In Scope

- ProductVariant/VariantOption/ProductAttribute refinement, invariants, mappings/migration only if not already present.
- Unique SKU and option-combination/default-variant/active/sort/version rules.
- Variant-specific price adjustment using approved Catalog price service and same currency.
- Read-only Inventory availability state linkage; Cart/Checkout continue using variant ID and server price/stock validation.
- Public list/detail variant APIs and product detail option-selection UI.
- Admin create/update/disable variants and manage attributes with `catalog.manage`, validation, concurrency, audit, and UI.

## Out Of Scope

- A second Product/Variant table, direct stock edits, client-calculated checkout price, image upload (Packet 03), advanced pricing/promotions, semantic search, bulk import, or full catalog admin redesign.

## Ownership And Read-Only Contracts

- **Owns:** variant SKU/options/display/default/active/sort/price-adjustment metadata and attributes.
- **Reads:** Product source, Inventory sellable availability projection, image metadata later.
- **Must not own/mutate:** Inventory quantities/reservations, Cart items, Order snapshots, Payment state.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/products/{productId}/variants` | Anonymous | Active/public variants; unavailable state may show without purchasability. |
| `GET /api/v1/products/{productId}/variants/{variantId}` | Anonymous | Matching product/variant public detail only. |
| Admin variant create/update/disable routes | `catalog.manage` | Explicit DTOs, option/SKU/default/version validation, audit. |
| Admin attribute create/update/delete routes | `catalog.manage` | Public/filterable/type/value validation, audit. |

Avoid ambiguous combined `POST/PUT/DELETE` route descriptions: document exact routes/methods before coding.

## UI Acceptance States

- Product detail: loading, no variants, default selection, selectable combination, impossible combination disabled, selected price/availability/image, unavailable selection, validation/dependency failure.
- Add-to-cart control enabled only for complete active/sellable selection; server still revalidates.
- Admin: list/edit form, duplicate SKU/combination, stale version conflict, disable confirmation, default reassignment, forbidden, audit result.
- Keyboard-operable option controls with labels, selected/disabled semantics, focus retention, and screen-reader announcement of price/availability changes.

## Likely File Areas To Inspect

- Existing approved Catalog Core entities/services/read contracts and Phase 2 checkout/cart interfaces.
- Infrastructure Catalog mappings/query/configuration/migration.
- API public/admin endpoints/DTOs/OpenAPI.
- Web product detail/admin views/view models/controllers/styles/scripts and tests.

## Architecture And Prohibited Dependencies

- Refine existing Catalog model; no duplicate source.
- Inventory is read through interface; variant admin cannot write stock.
- Price adjustment is applied by server Catalog pricing; Web/JS display is not checkout truth.
- Core has no EF/MVC/HTTP dependency; controllers/views do not query DbContext.

## Database Changes And Migrations

Only approved refinement of ProductVariant/VariantOption/ProductAttribute constraints/indexes/version/default/filter metadata. Stop if Phase 2 baseline differs; amend owning migration/contracts rather than duplicate tables. Review unique option-combination strategy and provider portability.

## Security, Logging, And Failure Rules

- Public APIs hide internal/draft/private attributes and inactive variants according to approved visibility.
- Admin changes require permission, anti-forgery for MVC writes, concurrency, reason/audit where sensitive.
- Logs include IDs/SKU/status/result/correlation, not full product internals/request bodies.
- Inconsistent variant/inventory link disables purchase safely and signals admin; it never fabricates stock.

## Implementation Steps

1. Obtain pre-code approval for baseline reconciliation, option uniqueness, default, adjustment/currency, visibility, inventory projection, exact routes/UI.
2. Refine framework-neutral entities/invariants/services without duplicating Phase 2 model.
3. Add approved mappings/constraints/indexes/migration.
4. Implement public projections and availability read composition.
5. Implement admin commands/policies/concurrency/audit and exact APIs.
6. Implement product detail selector and admin MVC/Razor states/accessibility.
7. Verify Cart/Checkout continue to revalidate variant/price/stock server-side.
8. Add unit/migration/API/security/UI/accessibility/concurrency tests.
9. Run checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Invariants | Unique SKU/options; one default; active/disabled; valid adjustment/currency. |
| Public | Active/public only; product/variant relationship; private attributes hidden. |
| Inventory | Availability read only; inconsistent/missing link disables purchase. |
| Admin | Permission/anti-forgery; create/update/disable; duplicate; stale; audit. |
| Commerce | Disabled/incomplete selection cannot cart; server revalidation still authoritative. |
| UI | Keyboard/focus/labels/disabled/price/availability/empty/error states. |
| Migration | No duplicate tables; fresh apply/constraints/indexes. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "ProductVariant|VariantOption|ProductAttribute|AvailableQuantity|ReservedQuantity|PriceAdjustment|catalog.manage" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- One Catalog variant model supports unique options/default/SKU/price adjustment and public attributes.
- Inventory linkage is read-only and Cart/Checkout authority is preserved.
- Public/admin APIs and MVC UI cover required states/accessibility/security.
- Migration is reconciled/scoped/reviewed; audit/concurrency tests pass.
- No image upload, stock mutation, client checkout price, advanced pricing, or semantic search added.
- Pre-code and post-test approvals/evidence exist.

## Security Stop Conditions

Stop on missing Catalog baseline, duplicate variant source, direct stock write, client-authoritative price/availability, private attribute leak, admin permission/anti-forgery/audit bypass, inaccessible selector, destructive migration, or Phase 2 contract change without review.

## Rollback / Forward-Fix And Handoff

Back up local DB before migration. After variants are referenced by Cart/Order, never delete/renumber to roll back; disable and forward-fix while preserving IDs/snapshots. Hand variant/attribute/detail contracts to Images, Wishlist, Reviews, Search, and integrated UI packets.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 02 from docs/implementation-guides/phase-3/02-product-variants-attributes-and-detail-experience.md only after all prerequisite gates, Packet 01, P3-CATALOG-001/P3-VARIANT-001, and pre-code review. Refine only the approved Catalog variant/attribute model, scoped migration, read-only Inventory availability composition, public/admin APIs, accessible MVC product-detail/admin states and tests. Preserve server Cart/Checkout price/stock validation, add evidence/02-completion.md, run checks, then obtain post-test review. Stop on duplicate catalog, stock writes, client truth, private leak, permission/audit/accessibility, or migration concerns.
```
