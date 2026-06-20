# Packet 05: Reviews, Ratings, And Moderation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packets 01-02 Approved; `P3-REVIEW-001`; Phase 2 verified-purchase projection |
| Layer Ownership | Reviews owns review lifecycle and approved aggregates; reads Catalog and Order verification |
| Risk Review | Mandatory pre-code and post-test human review by security, moderation, commerce, legal/content, and UX owners |
| Produces | Verified-review and moderation vertical slice, `evidence/05-completion.md` |

## Objective

Deliver trustworthy product ratings and reviews with verified-buyer eligibility, explicit moderation, abuse controls, owner-safe editing, and approved-only public aggregates.

## Read First And Prerequisites

- Approved Phase 1 ownership/permissions/audit/rate-limit contracts and Phase 2 Order/Product read contracts.
- Packets 01-02 and resolved `P3-REVIEW-001` policy decisions.
- Phase 3 roadmap review, authorization, security, failure, logging, testing, and UI sections.

## In Scope

- Review text/lifecycle and rating value, ownership, verified-purchase evidence reference, moderation reason, timestamps, concurrency, and optional soft delete.
- Server eligibility through a minimal Order read projection; one approved uniqueness policy per customer/product or order line.
- Public approved review list and aggregate, customer create/edit/delete, admin/support moderation if explicitly permitted.
- Simple local abuse controls: length/character rules, rate limits, duplicate prevention, moderation queue, and audit.

## Out Of Scope

- AI moderation, sentiment analysis, review incentives, media uploads, comments/voting, anonymous reviews, vendor replies, external anti-spam services, or model-generated review text.

## Ownership And Read-Only Contracts

- **Owns:** Review/Rating content, owner, product/order evidence reference, moderation state/reason, aggregate read model, edit/delete lifecycle.
- **Reads:** public Product identity/visibility and a boolean/minimal verified-purchase projection from Orders.
- **Must not own/mutate:** Product, Order/Payment state, CustomerProfile, Notification delivery, or admin permissions.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/products/{productId}/reviews` | Anonymous | Paginated approved reviews and approved aggregate only. |
| `POST /api/v1/products/{productId}/reviews` | Customer | Validates verified purchase, uniqueness, content/rating, and rate limit. |
| `PUT /api/v1/me/reviews/{reviewId}` | Owner | Edits under approved window; re-enters moderation when policy requires. |
| `DELETE /api/v1/me/reviews/{reviewId}` | Owner | Idempotent logical removal under retention policy. |
| `GET /api/v1/admin/reviews` | `reviews.moderate` | Filtered/paginated moderation queue without unrelated customer data. |
| `POST /api/v1/admin/reviews/{reviewId}/moderation` | `reviews.moderate` | Approve/reject/hide with reason, concurrency, and audit. |

Use Phase 0 Problem Details, pagination/filter/sort allowlists, and approved concealed ownership behavior. Never expose internal moderation notes publicly.

## UI Acceptance States

- Product page: aggregate/list loading, no reviews, paginated success, stale aggregate marker if approved, dependency failure.
- Customer: ineligible explanation, form validation, pending moderation, approved/rejected state, edit/delete confirmation, rate limit, stale conflict, session expired.
- Moderation: queue empty/loading/filter, detail, approve/reject/hide reason, stale decision, forbidden, audit failure, and retry-safe result.
- Rating inputs are keyboard accessible, labelled as values 1-5, and errors are announced near fields.

## Likely File Areas To Inspect

- Core Review/Rating entities/services/policies, Order verification and Catalog read interfaces.
- Infrastructure mappings, owner/aggregate/moderation queries, migration.
- API public/customer/admin DTOs/endpoints/OpenAPI and Web product/account/moderation views.
- Unit, integration, API/BOLA/rate-limit, UI, accessibility, aggregate, and audit tests.

## Architecture And Prohibited Dependencies

- Reviews asks Orders whether the authenticated customer has an eligible finalized purchase; it never queries Payment tables or changes Orders.
- Public aggregate includes only approved, active ratings and is updated transactionally or via an explicitly consistent reviewed read model.
- Controllers/views do not calculate aggregates, decide eligibility, query DbContext, or return raw entities.

## Database Changes And Migrations

Add/reconcile Review and Rating as one aggregate unless the roadmap's separate Rating entity has a proven need. Store customer/product, eligible order/order-item evidence, value 1-5, bounded title/body, moderation status/reason/actor/time, version, audit timestamps, and deletion state. Enforce the approved uniqueness rule and index product/status/date, owner/date, and moderation queue. Document aggregate rebuild behavior.

## Security, Logging, And Failure Rules

- Verified buyer is a server Order projection; no client flag. Product/order/customer relationship is checked within the command.
- Normalize and safely encode displayed text; reject markup unless a separately reviewed sanitizer exists. Apply rate limits and bounded content.
- Moderation requires permission, anti-forgery for MVC, reason, concurrency, and mandatory audit in the reviewed transaction boundary.
- Log IDs/status/result/rate-limit/audit/correlation; never full review bodies, email/profile, order details, tokens, cookies, or request bodies.
- If moderation/audit transaction fails, state does not change. Aggregate failure marks result stale/unavailable rather than inventing a rating.

## Implementation Steps

1. Obtain mandatory pre-code approval for eligibility/order states, uniqueness, edit window, moderation statuses/roles/reasons, aggregate consistency, text rules, rate limits, exact routes/UI.
2. Define domain lifecycle, eligibility/aggregate interfaces, commands/queries, and DTOs.
3. Add reviewed mapping, constraints/indexes, and migration.
4. Implement verified-purchase read projection, owner commands, moderation/audit transaction, and aggregate behavior.
5. Implement exact APIs and accessible customer/public/moderation UI.
6. Add abuse, ownership, concurrency, content encoding, aggregate, audit-failure, API/UI/accessibility tests.
7. Run checks, create evidence, and obtain mandatory post-test human approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Eligibility | Eligible finalized purchase; unpaid/cancelled/refunded policy; wrong customer/product; client flag ignored. |
| Lifecycle | Pending/approved/rejected/hidden; edit requeue; delete; illegal transition; stale moderation. |
| Ownership | Anonymous denied; customer A cannot alter B; moderator permission; concealed IDs. |
| Abuse/content | Duplicate; rate limit; bounds; markup encoded; invalid rating; repeated attempts. |
| Aggregate | Approved active only; edit/delete/moderation recalculation; empty/stale/failure. |
| Audit/UI | Mandatory audit rollback; all UI states; keyboard rating/focus/errors; migration. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Review|Rating|VerifiedPurchase|reviews.moderate|Moderation|OrderItem" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Only authenticated customers with server-verified eligible purchases can create a review under the approved uniqueness rule.
- Public output and aggregates include approved active reviews only; moderation notes and Order/private data never leak.
- Owner/moderator permissions, rate/content controls, concurrency, audit atomicity, exact API/UI states, and accessibility tests pass.
- Reviews consume Catalog/Order read contracts without mutating commerce data.
- Mandatory pre-code and post-test human approvals and evidence exist.

## Security Stop Conditions

Stop for missing human review, client-controlled verified status/owner, Order/Payment writes, cross-owner access, public moderation note, raw HTML rendering, absent rate/content limits, moderation without permission/reason/audit/concurrency, non-atomic audit, private logging, or destructive migration.

## Rollback / Forward-Fix And Handoff

Disable write/moderation routes while retaining review evidence. Forward-fix aggregates from authoritative review rows; do not hard-delete disputed/moderated records without retention approval. Hand approved aggregate/read contracts to Packets 09-11 and Notification events only after their contracts are approved.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 05 only after all prerequisite gates, Packets 01-02, P3-REVIEW-001, an approved Phase 2 verified-purchase projection, and mandatory pre-code human review. Implement only verified review/rating lifecycle, owner/moderator APIs, approved-only aggregate, accessible UI, migration, audit, abuse/security and failure tests. Never trust client eligibility, mutate Order/Payment, expose notes/private data, render unsafe markup, or add AI moderation. Add evidence/05-completion.md, run checks, and obtain mandatory post-test review.
```
