# Packet 08: Admin Order Management Experience

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packet 01 Approved; Phase 2 Order transitions/concurrency/audit Approved; `P3-ORDER-001` |
| Layer Ownership | Orders owns state; Phase 3 adds authorized admin query/command presentation only |
| Risk Review | Mandatory pre-code and post-test human review by commerce, security, operations, audit, and UX owners |
| Produces | Privileged order-management vertical slice, `evidence/08-completion.md` |

## Objective

Give authorized staff safe order search/detail/status-management screens that reuse Phase 2 Order services and legal transitions without creating a second order workflow or editing payment/inventory truth.

## Read First And Prerequisites

- Approved Phase 1 permissions/admin/audit contracts and complete Phase 2 Order/Payment/Inventory/Outbox transition evidence.
- Packet 01 and resolved `P3-ORDER-001` transitions/reasons/roles/concurrency decisions.
- Phase 3 roadmap admin order, authorization, security, logging, failure, testing, and UI sections.

## In Scope

- Paginated/filterable admin order list and privacy-minimized detail projection.
- Explicit permitted staff transitions through the existing Phase 2 Order application service, with expected version, reason, authorization, mandatory audit, and existing events/outbox.
- MVC/Razor admin list/detail/transition workflow and negative authorization/concurrency tests.
- Support read-only lookup through the approved support-safe projection; broader transitions require separate permission.

## Out Of Scope

- Direct status/database edits, payment confirmation/refund/capture, inventory quantity/reservation edits, total/address/order-item edits, shipment integration, cancellation compensation redesign, bulk mutation, or customer impersonation.

## Ownership And Read-Only Contracts

- **Owns:** Phase 2 Orders module remains sole owner of Order/OrderStatusHistory and legal transition commands.
- **Reads:** Payment summary, Inventory/fulfillment-safe summary, customer-safe contact projection only through approved interfaces and permissions.
- **Must not own/mutate:** Payment, Inventory, CustomerProfile, Cart/Checkout, or Outbox directly; Phase 3 UI owns no business state.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/admin/orders` | `orders.read` | Allowlisted filters/sorts, bounded pagination, privacy-minimized summaries. |
| `GET /api/v1/admin/orders/{orderId}` | `orders.read` | Approved detail projection; sensitive fields permission-gated/masked. |
| `POST /api/v1/admin/orders/{orderId}/status-transitions` | `orders.manage` | Requested legal transition, reason, expected version, idempotency key where Phase 2 requires it. |

No generic `PATCH order`. Return validation Problem Details, `401`, `403/404` per policy, `409` for illegal/stale transitions, and dependency-safe `503` only when truth cannot be established.

## UI Acceptance States

- List: loading, empty, paginated/filter/sort success, invalid filter, partial masked data, dependency failure, forbidden.
- Detail: loading, not found/concealed, current history, allowed next actions, no actions for read-only role, stale refresh, dependency failure.
- Transition: confirmation, reason, expected state/version, pending, success, illegal transition, stale conflict, audit failure, duplicate retry-safe result.
- Keyboard/focus/dialog/error behavior is accessible; destructive or consequential actions are never single-click or color-only.

## Likely File Areas To Inspect

- Approved Phase 2 Order application services/transitions/read interfaces, Phase 1 permission/audit services.
- Infrastructure optimized admin queries/index evidence only; no alternate write repository.
- API admin order DTOs/endpoints/OpenAPI and Web admin controllers/view models/views.
- Unit transition regression, integration, API authorization, concurrency/idempotency/audit, UI/accessibility, and privacy tests.

## Architecture And Prohibited Dependencies

- All writes call the existing Order transition service. Controller/view/SQL cannot assign status.
- Read composition uses application query interfaces; sensitive fields require a documented business need and permission.
- Existing Phase 2 transaction, compensation, idempotency, status history, audit, and Outbox contracts remain authoritative.

## Database Changes And Migrations

No new business table is expected. Add an index only after measured query evidence and owning Order review. Any schema/status change returns to Phase 2 contract review and migration policy; Phase 3 must not silently extend the state machine.

## Security, Logging, And Failure Rules

- Separate `orders.read` from `orders.manage`; SupportAgent receives only approved support lookup unless explicitly elevated.
- Require anti-forgery for MVC writes, expected version, reason, idempotency where applicable, least privilege, and mandatory audit in the transition boundary.
- Mask/minimize address/contact/payment provider references; never display secrets/card data or expose another privileged role's fields.
- Log actor/order/from/to/result/reason code/idempotency outcome/correlation, not full reason text, addresses, contact data, payment detail, tokens, or request bodies.
- Audit/outbox/transaction failure means no apparent transition; UI refreshes authoritative state after uncertain failure.

## Implementation Steps

1. Obtain mandatory pre-code approval for staff transition matrix, reasons, permission split, sensitive fields/masking, query limits/index budget, exact route/UI and idempotency behavior.
2. Add approved admin read projections and allowlisted query contracts.
3. Expose only existing transition commands with authorization/version/reason/idempotency/audit.
4. Implement exact APIs and accessible list/detail/confirmation UI.
5. Add negative permission, direct-write guard, transition regression, stale/duplicate/audit/outbox/failure/privacy/performance/UI/accessibility tests.
6. Run checks, create evidence, and obtain mandatory post-test human approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Authorization | Customer denied; read-only staff cannot write; support scope; admin/super-admin explicit permissions. |
| Transitions | Every approved/illegal transition; reason; expected version; duplicate/idempotent retry; history/event. |
| Integrity | No total/item/address/payment/inventory edit; existing Phase 2 compensation/outbox preserved. |
| Privacy | Masked/minimized list/detail; permission-gated fields; safe logs/audit. |
| Failure | Stale, database, audit, outbox, dependency uncertainty; authoritative refresh. |
| UI/performance | All states/accessibility; bounded filters/pages; query budget/index evidence. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "orders.read|orders.manage|OrderStatus|StatusHistory|Idempotency|AuditLog" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Admin screens use one Phase 2 Order state machine and application service; no direct status or commerce-data writes exist.
- Read/manage permission split, legal transitions, version/reason/idempotency, mandatory audit, and Outbox behavior are tested.
- Sensitive detail is minimized/masked and support access stays within its approved projection.
- Exact API/UI states, failure recovery, performance budget, and accessibility tests pass.
- Mandatory pre-code and post-test human approvals and evidence exist.

## Security Stop Conditions

Stop for missing human review, generic order patch, controller/SQL status assignment, new state without Phase 2 review, payment/inventory/total/address mutation, permission conflation, support overreach, absent reason/version/idempotency/audit, unsafe sensitive display/logging, unbounded query, or destructive migration.

## Rollback / Forward-Fix And Handoff

Disable admin write UI/route while retaining read access and audit/history. Never reverse a completed commerce transition by database edit; invoke an approved compensating transition or forward-fix. Hand privacy-safe admin projections to Packet 09 and integrated privileged flows to Packet 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 08 only after all prerequisite gates, Packet 01, complete approved Phase 2 Order evidence, P3-ORDER-001, and mandatory pre-code human review. Implement only bounded admin Order queries and accessible screens plus exact calls to existing legal transition services with permissions, reason, version, idempotency, audit, outbox, tests, and evidence. Never add generic patches, direct status/SQL writes, new states, payment/inventory/total/address edits, or broad support access. Add evidence/08-completion.md, run checks, and obtain mandatory post-test review.
```
