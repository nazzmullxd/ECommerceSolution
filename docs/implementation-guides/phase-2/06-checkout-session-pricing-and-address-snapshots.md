# Packet 06: Checkout Session, Trusted Pricing, And Address Snapshots

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 02-05 Approved; decisions `P2-MONEY-001`, `P2-IDEMP-001`, `P2-CHECKOUT-001` |
| Layer Ownership | Checkout owns session/orchestration/snapshots; source modules retain price/stock/profile ownership |
| Risk Review | **Pre-code and post-test commerce, security, and database approval required** |
| Produces | Trusted checkout/reservation/address workflow, `evidence/06-completion.md` |

## Objective

Implement authenticated CheckoutSession and AddressSnapshot creation with server-side repricing, explicit discount/shipping/tax placeholders, cart/address ownership, atomic inventory reservation coordination, expiry, and durable idempotency.

## Read First And Prerequisites

- Approved Phase 0/1 API/data/identity/audit contracts.
- Packets 01-05 evidence and Phase 1 DbContext/current-user/idempotency/error behavior.
- Phase 2 roadmap sections 3-8, 12-15, and 18.

## In Scope

- CheckoutSession/AddressSnapshot entities/status/expiry/version/mappings/migration.
- `POST /api/v1/checkout/sessions` authenticated only.
- `PUT /api/v1/checkout/sessions/{id}/address` if address is not finalized during creation.
- Cart/customer/address ownership validation.
- Server-side Catalog price/currency/activity reload, quantity validation, discount placeholder validation, shipping/tax placeholder calculation, totals/rounding.
- Inventory all-line reservation, idempotency key/fingerprint/result replay, concurrency/failure behavior, audit/logging/tests.

## Out Of Scope

- Guest checkout, real promotions/coupons, shipping carrier/tax service, order/payment creation, saved-address CRUD, card/payment data, or frontend checkout UI.

## Likely File Areas To Inspect

- Core Checkout entities/value objects/commands/services/results/calculators.
- Catalog/Cart/User/Inventory application contracts.
- Infrastructure Checkout mappings/idempotency persistence/transactions/DI/migration.
- API checkout endpoints/DTOs/OpenAPI and unit/integration/API/security tests.

## Architecture And Prohibited Dependencies

- Checkout orchestrates approved application interfaces; it does not own Catalog price, Cart intent, User saved address, or Inventory stock.
- Core calculation/orchestration has no EF/HTTP/payment/provider dependency.
- API does not calculate or accept trusted totals/stock/owner.
- Transaction strategy must align with Phase 1 one-database/DbContext decision; no distributed transaction.

## Trusted Calculation Contract

1. Resolve authenticated customer and own active cart.
2. Load each active ProductVariant and current Money from Catalog.
3. Validate quantities/limits and one currency.
4. Validate discount code server-side; if unsupported, reject or explicit zero per `P2-CHECKOUT-001`.
5. Calculate server-side shipping/tax placeholders and statuses; never accept client amounts.
6. Calculate subtotal/discount/shipping/tax/total using approved rounding.
7. Resolve own saved address or validated one-time input and copy immutable snapshot.
8. Reserve all inventory lines atomically with session expiry.
9. Persist CheckoutSession, snapshot, idempotency result, reservation references, and required audit in approved transaction order.

## API Contract

| Method/Route | Auth/Idempotency | Behavior |
| --- | --- | --- |
| `POST /api/v1/checkout/sessions` | Customer; required `Idempotency-Key` | Own cart/address identifiers only; server computes all values and reserves stock. |
| `PUT /api/v1/checkout/sessions/{id}/address` | Customer owner; approved repeat/concurrency rule | Replace snapshot only while allowed and before payment/order lock. |

Response includes session ID/status/expiry, server-calculated totals/statuses, sanitized snapshot summary, changed-price/unavailable-item details safe for customer, and reservation outcome. It never echoes trusted client totals because none are accepted.

## Database Changes And Migrations

Create CheckoutSession/AddressSnapshot and approved idempotency uniqueness/index/version/expiry relationships only. Define whether one or two snapshots are modeled; roadmap examples imply shipping and billing, so cardinality/type must be approved before migration.

## Security, Logging, And Failure Rules

- Authenticated customer only; guest must merge/login first.
- Query-level ownership for cart/address/session; privacy-safe 404 for others.
- Explicit request DTO excludes price/discount amount/shipping/tax/total/stock fields.
- Do not log full address/phone/body; log IDs, safe total/currency/status, change categories, correlation.
- Failed reserve/calculation/persistence leaves no active partial reservation/session, or compensation is explicit/tested.
- Same idempotency key/same fingerprint returns original session; different fingerprint 409.

## Implementation Steps

1. Obtain pre-code approval for placeholders, snapshot cardinality, transaction sequence, expiry, idempotency retention, recalculation response.
2. Define CheckoutSession/AddressSnapshot model/invariants/calculation results in Core.
3. Implement pure pricing/total calculator with one-currency/rounding and placeholder statuses.
4. Add mappings/constraints/indexes and reviewed migration.
5. Implement ownership-safe cart/address loading and Catalog repricing.
6. Implement all-line reservation coordination and atomic/failure behavior.
7. Implement durable idempotent create/replay/conflict.
8. Implement thin endpoints/DTOs/Problem Details/OpenAPI/audit/logging.
9. Add unit, integration, API, ownership, price-tamper, idempotency, reservation-failure, concurrency, and PII-redaction tests.
10. Run checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Auth/ownership | Anonymous; wrong cart/address/session; own valid. |
| Reprice | Client cannot submit trusted amounts; changed price reflected; inactive/missing variant fails. |
| Totals | Rounding/currency; quantity; placeholder discount/shipping/tax status and zero behavior. |
| Snapshot | Saved/one-time allowed; validation; immutable copy; no log/response excess. |
| Reserve | Success all lines; insufficient one line rolls back; concurrency conflict safe. |
| Idempotency | Same/same replay; same/different 409; concurrent duplicate one session/reservation. |
| Expiry/status | Legal updates only; expired session cannot proceed. |
| Migration/logging | Constraints/indexes; no full address/body/client total fields. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "UnitPrice|DiscountAmount|ShippingAmount|TaxAmount|TotalAmount|Address|Phone|Idempotency" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect API requests/logging for client-trusted or private fields.

## Acceptance Criteria

- Checkout requires authenticated ownership and durable idempotency.
- All price/discount/shipping/tax/total/stock decisions are server-side.
- Snapshot is validated, immutable for the session/order flow, and privacy-safe.
- Reservation/session persistence is atomic or explicitly compensated/tested.
- Migration is scoped/reviewed/reproducible.
- No order/payment/real promotion/shipping/tax/guest checkout/UI was added.
- Pre-code and post-test approvals/evidence exist.

## Security Stop Conditions

Stop on request trusted amounts/stock, mixed currency ambiguity, cross-user cart/address, full address logs, duplicate reservation/session, non-atomic failure, missing idempotency fingerprint, unsupported placeholder ambiguity, unreviewed snapshot/migration, or direct EF controller logic.

## Rollback / Forward-Fix And Handoff

Release active test reservations before rollback. Back up local DB before migration. After shared checkouts exist, use forward-fix and explicit reservation compensation. Hand approved session/totals/snapshots/reservation/idempotency contracts to Orders Packet 07.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 06 from docs/implementation-guides/phase-2/06-checkout-session-pricing-and-address-snapshots.md only after PHASE0/PHASE1 gates, Packets 02-05, required decisions, and pre-code commerce/security/database approval. Implement only CheckoutSession/AddressSnapshot, scoped migration, server-only repricing/placeholders, owned cart/address validation, atomic inventory reservation, durable idempotency, thin APIs and tests. Add evidence/06-completion.md, run checks, then obtain post-test approval. Stop on client amounts/stock, PII logs, ownership, currency, idempotency, atomicity, or migration concerns.
```

