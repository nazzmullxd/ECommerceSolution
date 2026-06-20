# Packet 01: Experience Contracts, Permissions, And UI Conventions

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Approved Phase 0-2 contracts/evidence |
| Layer Ownership | Core framework-neutral statuses/permissions/read contracts; API/Web conventions |
| Risk Review | Security, UX, commerce, and solution-architecture review |
| Produces | Phase 3 contract baseline/tests, `evidence/01-completion.md` |

## Objective

Define Phase 3 statuses, permissions, ownership/read-only matrix, cross-module projections, canonical API/UI states, and MVC/Razor accessibility conventions before any experience entity or privileged screen is implemented.

## Read First And Prerequisites

- Approved Phase 0-2 ADRs/contracts/evidence.
- Phase 3 roadmap sections 2-7, 15-21, and 24.
- Phase 1 permission/current-user/audit contracts and Phase 2 Catalog/Inventory/Order/Outbox contracts.
- Existing Web/API conventions and this package README.

## In Scope

- Status/transition contracts for Review, SupportTicket, TicketMessage visibility, Notification, ProductImage, and optional DashboardMetric.
- Permission constants/policy requirements and SupportAgent role evolution through approved access process.
- Ownership matrix and minimal cross-module read projection contracts.
- API list/error/pagination/filter/sort conventions inherited from prior phases.
- MVC/Razor page/form/component state and accessibility conventions.
- Architecture/security tests for forbidden dependencies and ownership metadata.

## Out Of Scope

- Entities/migrations/business endpoints/pages, role assignment to real users, seeded credentials, CSS redesign, image storage, reports, or feature implementation.

## Ownership And Read-Only Contracts

| Module | Owns | Reads Through Approved Interface | Must Not Own/Mutate |
| --- | --- | --- | --- |
| Product Experience | Variant/options/attributes/images presentation rules | Catalog, Inventory availability projection | Inventory quantity, Cart, Order, Payment |
| Wishlist | Wishlist/items/privacy | Public Catalog visibility | Product/price/stock source |
| Reviews | Review/rating/moderation | Customer identity, Product, verified Order projection | Order/payment truth |
| Support | Ticket/message/assignment/visibility | Customer and limited authorized Order summary | Order/payment mutation |
| Notifications | Recipient record/read/delivery status | Outbox event facts | Business source state |
| Dashboard/Reports | Derived aggregate projections | Catalog/Inventory/Order/Support/Review queries | Transactional truth |

## Permission Contract

At minimum review/define stable codes: `catalog.manage`, `reviews.moderate`, `support.tickets.read`, `support.tickets.assign`, `support.tickets.update`, `support.internal-notes.write`, `orders.read`, `orders.manage`, `dashboard.read`, and `reports.read`. Add through approved Phase 1 permission seed/version/invalidation process; do not hardcode role shortcuts.

## API Contract

- Preserve `/api/v1`, camelCase, Problem Details, correlation, max page 100, allowlisted filters/sorts, ownership-safe 403/404.
- Define standard paged result and UI-facing safe summary projections without duplicating source entities.
- No route is implemented in this packet.

## UI Acceptance States

Define reusable MVC/Razor conventions for loading/processing, empty, success, field validation, unauthorized login redirect/API 401, forbidden/privacy-safe not found, stale conflict, dependency unavailable, pagination, destructive confirmation, audit reason, and screen-reader status.

## Likely File Areas To Inspect

- Core status/permission/read-contract conventions.
- API/Web shared models/helpers/views/layout/styles only if a minimal convention scaffold is approved.
- Architecture/unit tests; Phase 1 permission seed/change evidence.

## Architecture And Prohibited Dependencies

- Core has no MVC/Razor/EF/HTTP/Identity framework dependency.
- Web/API consume application services; no direct DbContext/source-table reporting.
- Specialized roles use ApplicationUser+roles/permissions, not separate staff entities.
- Read projections cannot be used for writes or checkout/payment truth.

## Database Changes And Migrations

None. Permission metadata seed changes require approved Phase 1 access-control change process and may be executed in owning feature packets.

## Security, Logging, And Failure Rules

- Deny by default; ownership/permission applied in server query/command.
- UI hidden controls are not authorization.
- Logs contain IDs/status/action/correlation, never private bodies/files/addresses/tokens/payment data.
- Unknown dependency/derived freshness returns safe unavailable/stale state, not fabricated data.

## Implementation Steps

1. Verify all prerequisite gates and existing conventions.
2. Approve module names, statuses/transitions, ownership/read matrix, and permission codes.
3. Define framework-neutral permission/status/read projection contracts.
4. Define API metadata conventions for every Phase 3 endpoint.
5. Define MVC/Razor page/form state and accessibility checklist consistent with existing Web stack.
6. Define audit reason/confirmation/concurrency conventions for privileged actions.
7. Add architecture/unit tests for status transitions, permission metadata, forbidden dependencies.
8. Record future packet mapping and unresolved decisions.
9. Run build/tests/diff checks, create evidence, request approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Statuses | Legal/illegal transitions deterministic. |
| Permissions | Stable unique codes; deny-by-default metadata; no role shortcut. |
| Ownership | Every Phase 3 resource has owner/access rule. |
| Read contracts | Minimal, framework-neutral, no mutation/source duplication. |
| API conventions | Error/pagination/filter/sort/auth metadata defined. |
| UI conventions | Keyboard/focus/labels/errors/status/empty/forbidden/stale states defined. |
| Architecture | Core contains no outer framework dependencies. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "reviews.moderate|support.tickets|orders.manage|dashboard.read|reports.read|SupportAgent" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- Ownership/read-only matrix, statuses, permissions, API and UI conventions are approved/tested.
- Specialized staff remains role/permission on ApplicationUser.
- UI states/accessibility expectations are implementation-ready.
- No entity/migration/endpoint/feature/later-scope behavior was added.
- Evidence and security/UX/architecture/commerce approval exist.

## Security Stop Conditions

Stop on missing prerequisite gate, role shortcut/fail-open policy, separate staff identity, mutable cross-module projection, UI-only authorization, private-data logging, unclear ownership, or Phase 2 rule redefinition.

## Rollback / Forward-Fix And Handoff

Before feature consumption, revert rejected contracts. After use, change statuses/permission/read contracts additively with version/change review. Handoff approved baseline to Packets 02-10.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 01 from docs/implementation-guides/phase-3/01-experience-contracts-permissions-and-ui-conventions.md only after PHASE0/PHASE1/PHASE2 gates. Implement only framework-neutral Phase 3 statuses, permissions, ownership/read projections, shared API/UI/accessibility conventions and architecture/unit tests. Preserve ApplicationUser roles and Phase 2 truth, add evidence/01-completion.md, run checks, and stop on role shortcuts, unclear ownership, mutable projections, UI-only authorization, private logging, or outer dependencies. Do not add entities, migrations, routes, or feature pages.
```

