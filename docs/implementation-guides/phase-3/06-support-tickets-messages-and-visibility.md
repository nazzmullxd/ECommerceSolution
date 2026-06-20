# Packet 06: Support Tickets, Messages, And Visibility

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packet 01 Approved; `P3-ACCESS-001`, `P3-SUPPORT-001`; approved limited Order support projection |
| Layer Ownership | Support owns tickets/messages/assignment/escalation; reads identity and limited Order summaries |
| Risk Review | Mandatory pre-code and post-test human review by security, privacy, support operations, authorization, and UX owners |
| Produces | Customer/support/admin ticket vertical slice, `evidence/06-completion.md` |

## Objective

Provide a privacy-safe support workflow whose customer messages, internal notes, assignments, status transitions, and optional Order context are isolated by ownership and least privilege.

## Read First And Prerequisites

- Approved Phase 1 roles/permissions/ownership/audit and Phase 2 Order access/read contracts.
- Packet 01 and resolved `P3-ACCESS-001`/`P3-SUPPORT-001` decisions.
- Phase 3 roadmap support, authorization, security, failure, logging, testing, and UI sections.

## In Scope

- SupportTicket and TicketMessage lifecycle, customer ownership, category/priority, assignment, escalation, closure/reopen policy, concurrency, and audit.
- Distinct customer-visible replies and internal staff notes, enforced in storage queries and DTOs.
- Customer create/list/detail/reply; support queue/assignment/reply/internal note/status; admin escalation/oversight.
- Optional Order reference validated through a minimum support projection, never full payment/address data.

## Out Of Scope

- File attachments, live chat, AI-generated replies, external helpdesk integration, SLA automation, refund/payment decisions, or unrestricted Order/customer search.

## Ownership And Read-Only Contracts

- **Owns:** ticket number, customer owner, category/priority/status, subject, assignment, escalation, messages, internal-note visibility, timestamps, and version.
- **Reads:** authenticated user/staff identity and a minimal authorized Order support summary such as order number, owner match, date, public status, and support-safe totals if approved.
- **Must not own/mutate:** ApplicationUser/CustomerProfile, Order/Payment/Inventory state, refunds, Notification delivery, or permissions.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `POST /api/v1/me/support-tickets` | Customer | Creates caller-owned ticket; optional order must belong to caller. |
| `GET /api/v1/me/support-tickets` | Customer | Paginated caller-owned summaries only. |
| `GET /api/v1/me/support-tickets/{ticketId}` | Customer owner | Customer-visible messages only. |
| `POST /api/v1/me/support-tickets/{ticketId}/messages` | Customer owner | Adds customer-visible reply if status permits. |
| `GET /api/v1/support/tickets` | `support.tickets.read` | Allowlisted queue filters/sorts and assigned/unassigned scope. |
| `GET /api/v1/support/tickets/{ticketId}` | `support.tickets.read` | Staff projection including internal notes only when permitted. |
| `POST /api/v1/support/tickets/{ticketId}/messages` | `support.tickets.reply` | Explicit `visibility=Customer` or `Internal`; server validates permission. |
| `POST /api/v1/support/tickets/{ticketId}/assignment` | `support.tickets.assign` | Assign/unassign with version and audit. |
| `POST /api/v1/support/tickets/{ticketId}/status` | `support.tickets.manage` | Legal transition with reason/version/audit. |
| `POST /api/v1/admin/support-tickets/{ticketId}/escalation` | `support.tickets.escalate` | Admin escalation decision and audit. |

Do not reuse a staff DTO for customers. Conceal cross-customer identifiers according to the Phase 1 ownership convention.

## UI Acceptance States

- Customer: create validation, list empty/loading, detail/replies, closed state, order mismatch, session expiry, concealed not-found, stale conflict, dependency failure.
- Support: queue empty/loading/filter, assignment, customer reply versus clearly distinct internal note composer, status transition/reason, stale assignment, forbidden, audit failure.
- Admin: escalated queue, decision/reason, conflict, and audited result.
- Internal notes are visually and semantically distinct and cannot appear in customer HTML, JSON, notification, or accessibility tree.

## Likely File Areas To Inspect

- Core Support entities/lifecycle/policies/current-user/Order support interfaces.
- Infrastructure mappings, owner/staff projections, migration, audit transaction.
- API customer/support/admin DTOs/endpoints/OpenAPI and Web portals/views/view models.
- Unit, integration, API/BOLA, visibility, audit, UI, accessibility, and failure tests.

## Architecture And Prohibited Dependencies

- Separate customer and staff query paths/DTOs. Filtering internal notes in a view is insufficient.
- Support consumes a minimal Order application query; it does not query Order/Payment tables directly or invoke privileged status/refund commands.
- Core has no EF/MVC/HTTP dependency; API/Web contain no authorization business rules or DbContext queries.

## Database Changes And Migrations

Add/reconcile SupportTicket and TicketMessage with opaque ticket ID plus human-safe ticket number, owner, optional order reference, category, priority, status, assignee, escalation, message author/type/visibility, bounded content, timestamps, version, and retention state. Index customer/date, status/priority/date, assignee/status, ticket/message order, and ticket-number uniqueness.

## Security, Logging, And Failure Rules

- Apply owner/staff scope in database queries; validate every message/status/assignment against current role/permission and ticket state.
- Internal visibility is an enum/invariant, not a user-entered label. Customer notifications can consume customer-visible message events only.
- UI warns users not to submit passwords/card details. Bound and safely encode content; no raw HTML in MVP.
- Log ticket/message IDs, actor ID, operation/status/visibility/result, and correlation ID; never message text, full Order/address/profile, tokens, cookies, or request bodies.
- Mandatory audit failure rolls back privileged assignment/status/escalation/internal-note changes. Reply delivery failure does not fabricate success.

## Implementation Steps

1. Obtain mandatory pre-code approval for roles/permissions, assignment, statuses, reopen/escalation, internal-note model, Order projection, retention, rate/content limits, exact routes/UI.
2. Define lifecycle, entities, separate customer/staff projections, and authorization policies.
3. Add reviewed mapping, constraints/indexes, and migration.
4. Implement owner-scoped customer commands/queries and limited Order validation.
5. Implement staff assignment/reply/internal note/status/escalation with mandatory audit.
6. Implement exact APIs and accessible customer/support/admin UI with unmistakable visibility state.
7. Add BOLA/internal-note leak/audit/concurrency/content/rate/failure/UI/accessibility tests.
8. Run checks, record evidence, and obtain mandatory post-test human approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Ownership | Customer A cannot list/read/reply to B; guessed IDs concealed; order belongs to caller. |
| Staff scope | Support/admin permissions; assignment scope; unauthorized escalation/internal note/status denied. |
| Visibility | Internal note absent from customer API/HTML/notification/accessibility tree; separate DTO/query. |
| Lifecycle | Legal/illegal transitions; closed reply/reopen; assignment/escalation; stale version. |
| Content/privacy | Bounds/rate limit/encoding; no sensitive body logging; audit rollback. |
| UI/migration | All states and keyboard/error behavior; fresh migration/indexes/uniqueness. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "SupportTicket|TicketMessage|Internal|support.tickets|OrderSupport|MessageBody" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Customers can access only their tickets and customer-visible messages; internal notes cannot enter any customer surface.
- Support/admin actions use approved least-privilege policies, legal transitions, concurrency, reason, and atomic audit.
- Order context is minimal/read-only and no support action mutates commerce truth.
- Exact API/UI states, privacy-safe logging, content/rate controls, BOLA, failure, migration, and accessibility tests pass.
- Mandatory pre-code and post-test human approvals and evidence exist.

## Security Stop Conditions

Stop for missing human review, unresolved SupportAgent permissions, owner filtering after materialization, shared customer/staff DTO, possible internal-note leakage, raw HTML, unbounded/rate-unlimited messages, broad Order/Payment access, support refund/status writes, missing mandatory audit, sensitive logs, or destructive migration.

## Rollback / Forward-Fix And Handoff

Disable new writes while preserving tickets/messages/audit evidence. Forward-fix visibility or assignment defects; never hard-delete a potentially exposed record before incident review. Hand customer-visible reply/status events only to Packet 07 and integrated role flows to Packet 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 06 only after all prerequisite gates, Packet 01, P3-ACCESS-001/P3-SUPPORT-001, an approved minimal Order support projection, and mandatory pre-code human review. Implement only owner-safe tickets/messages, separate customer/staff projections, internal-note isolation, least-privilege assignment/status/escalation, exact APIs, accessible portals, migration, mandatory audit, and tests. Never expose note text, query Order/Payment directly, mutate commerce, accept owner from client, or add attachments/AI. Add evidence/06-completion.md, run checks, and obtain mandatory post-test review.
```
